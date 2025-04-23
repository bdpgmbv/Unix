## File I/O Operations in UNIX

This section introduces file I/O operations in UNIX, focusing on five core functions:

- **open** – Open a file  
- **read** – Read from a file  
- **write** – Write to a file  
- **lseek** – Adjust file position  
- **close** – Close a file  

### Key Points:
- **Unbuffered I/O** – These functions are called "unbuffered" because each operation triggers a direct system call (unlike standard I/O, which uses buffers).  
- **Standards Compliance** – These functions are part of POSIX.1 and the Single UNIX Specification, not ISO C.  
- **Atomic Operations** – Critical for resource sharing among processes (prevents race conditions).  
- **File Sharing & Kernel Data Structures** – Explores how multiple processes interact with files and the underlying kernel mechanisms.  

### Additional Functions:
Later sections cover:  
- **dup** – Duplicate file descriptors  
- **fcntl** – File control operations  
- **sync, fsync** – Force write to disk  
- **ioctl** – Device-specific operations  

This section sets the foundation for understanding low-level file operations and process interactions in UNIX.



# Section 3.2: File Descriptors

## Key Concepts:

### File Descriptors (FDs)
- Non-negative integers used by the kernel to reference open files.  
- Returned by system calls like `open()` and `creat()`.  
- Used in `read()`, `write()`, and other I/O operations.  

---

### Standard File Descriptors (Shell Convention)
- **0 (STDIN_FILENO)** → Standard input (stdin)  
- **1 (STDOUT_FILENO)** → Standard output (stdout)  
- **2 (STDERR_FILENO)** → Standard error (stderr)  

> **Note:** This convention is not enforced by the kernel, but breaking it can cause application issues.

---

### POSIX Compliance & Best Practices
- Use symbolic constants (`STDIN_FILENO`, etc.) instead of magic numbers (`0`, `1`, `2`).  
- Defined in `<unistd.h>`.  

---

### File Descriptor Limits
- **Historically:** Max 20 FDs per process (early UNIX).  
- **Modern Systems** (FreeBSD, Linux, macOS, Solaris): Effectively unlimited, constrained by:  
  - Available memory  
  - Integer size (e.g., 32-bit vs. 64-bit)  
  - System-configured hard/soft limits (e.g., `ulimit -n`)  

---

## Key Takeaway:
File descriptors are the fundamental mechanism for file I/O in UNIX. Understanding their role, standard assignments, and system limits is crucial for system programming.

# Section 3.3: `open` and `openat` Functions

## Key Functions:
```c
#include <fcntl.h>  
int open(const char *path, int oflag, ... /* mode_t mode */);  
int openat(int fd, const char *path, int oflag, ... /* mode_t mode */);  
```
- **Return:** File descriptor (FD) on success, `-1` on error.  
- **`mode` argument:** Required only when creating a file (`O_CREAT`).

---

## `oflag` Argument (File Access Modes)
Exactly one must be specified:
- **`O_RDONLY`** → Open for read-only.  
- **`O_WRONLY`** → Open for write-only.  
- **`O_RDWR`** → Open for read-write.  
- **`O_EXEC`** → Open for execute-only (rare).  
- **`O_SEARCH`** → Open for directory search (not widely supported).  

---

## Optional Flags (Combined via Bitwise OR `|`)

### File Creation & Control:
- **`O_CREAT`** → Create file if it doesn’t exist (requires `mode`).  
- **`O_EXCL`** → Fail if file exists (atomic check + create).  
- **`O_TRUNC`** → Truncate file to length 0 if it exists.  
- **`O_APPEND`** → Always write at end of file.  
- **`O_CLOEXEC`** → Close FD on `exec` (avoid leaks).  

### Filesystem Behavior:
- **`O_DIRECTORY`** → Fail if path is not a directory.  
- **`O_NOFOLLOW`** → Fail if path is a symlink.  
- **`O_NOCTTY`** → Prevent terminal device from becoming controlling terminal.  

### I/O Behavior:
- **`O_NONBLOCK`** → Non-blocking mode (for FIFOs/devices).  
- **Synchronous I/O (Ensure writes hit disk):**
  - **`O_SYNC`** → Wait for data + attributes.  
  - **`O_DSYNC`** → Wait only for data (not metadata).  
  - **`O_RSYNC`** → Sync reads with pending writes.  

---

## `openat` vs. `open`

### `fd` Parameter Usage:
- **Absolute path:** `fd` ignored (like `open`).  
- **Relative path + `fd`:** Resolve path relative to directory FD.  
- **Relative path + `AT_FDCWD`:** Resolve relative to current working directory.  

### Purpose:
- **Thread-safe relative paths:** Avoid shared CWD issues.  
- **Mitigate TOCTTOU (Time-of-Check-to-Time-of-Use) race conditions.**  

---

## Filename Truncation
- **Historical Behavior:** Some systems truncated long filenames silently (e.g., SVR2).  
- **Modern Systems:** Return `ENAMETOOLONG` if `_POSIX_NO_TRUNC` is in effect.  
- **File system-dependent:**
  - **UFS (BSD):** Enforces limits.  
  - **PCFS (DOS-compatible):** Truncates filenames.  

---

## Key Takeaways:
- `open`/`openat` are foundational for file operations, with fine-grained control via `oflag`.  
- `openat` enhances security and thread safety.  
- **Synchronous I/O flags (`O_SYNC`, etc.)** ensure data integrity for critical writes.  
- Always use symbolic constants (e.g., `O_RDWR`) for portability.  

This section is crucial for understanding low-level file handling and avoiding common pitfalls (e.g., race conditions, blocking I/O, etc.).


# (Vyshali's Section) TOCTTOU in `open()` vs `openat()` (With Examples)

## What is TOCTTOU?
**TOCTTOU** (Time-of-Check to Time-of-Use) is a security bug where:
1. A program **checks** a file (e.g., for permissions or existence).  
2. Before the program **uses** the file, an attacker **changes** it.  

This leads to the program acting on the **wrong file**, causing crashes or security risks.

---

## `open()` is Vulnerable to TOCTTOU

### Example: A Privileged Program Backing Up a File
Suppose a backup tool does this:
```c
// Check if the file exists
if (file_exists("/tmp/user_data.txt")) {
    // Open the file
    int fd = open("/tmp/user_data.txt", O_RDONLY);
}
```

### Attack:
1. After the **check**, an attacker replaces `user_data.txt` with a **symlink** to `/etc/passwd`.  
2. When the program opens the file, it ends up reading sensitive system files like `/etc/passwd`.

### Why?
- `open()` uses **absolute paths**, so if the file changes between the check and open, the program is fooled.

---

## `openat()` Fixes TOCTTOU

### Example: Safe File Access with `openat()`
Instead of checking and opening separately:

1. **Open the directory first (lock it down):**
```c
int dir_fd = open("/tmp", O_RDONLY | O_DIRECTORY);
```

2. **Open the file inside that directory (atomically):**
```c
int file_fd = openat(dir_fd, "user_data.txt", O_RDONLY);
```

### Why This is Safe:
- The file is opened **relative to `/tmp`**, so even if an attacker changes `/tmp/user_data.txt`, the program still accesses the **original file**.  
- There is **no time gap** between checking and opening → **No TOCTTOU**!

---

## Key Differences

| Feature           | `open()`                                   | `openat()`                              |
|--------------------|--------------------------------------------|-----------------------------------------|
| **Path Handling**  | Uses full path (e.g., `/tmp/file.txt`)      | Uses a directory file descriptor + relative path |
| **TOCTTOU Risk**   | Vulnerable (file can change after check)   | Safe (no race condition)                |
| **Thread Safety**  | Bad (all threads share working directory)  | Good (each thread can have its own dir) |
| **Use Case**       | Simple file access                        | Secure/safe file operations             |

---

## When to Use Which?

- **Use `open()`** → For simple scripts or non-security-critical tasks.  
- **Use `openat()`** → For security-sensitive programs (e.g., `sudo`, backup tools, servers).

---

## Final Summary
- **`open()`** → Fast but risky (**TOCTTOU possible**).  
- **`openat()`** → Safer, avoids race conditions, better for security.  

**Always use `openat()` when security matters!**


# Sections 3.4: `creat` and 3.5: `close` Functions

## 3.4 `creat` Function
```c
#include <fcntl.h>  
int creat(const char *path, mode_t mode);  
```

### Purpose:
- Creates a new file (obsolete, but still exists for backward compatibility).

### Equivalent to:
```c
open(path, O_WRONLY | O_CREAT | O_TRUNC, mode);  
```

### Limitations:
- Opens the file as **write-only** → Cannot read after creation without reopening it.
- Modern code should use `open()` with `O_RDWR | O_CREAT | O_TRUNC` for **read-write access**.

### Historical Context:
- In early UNIX, `creat` was needed because `open()` lacked `O_CREAT` and `O_TRUNC` flags.

---

## 3.5 `close` Function
```c
#include <unistd.h>  
int close(int fd);  
```

### Purpose:
- Closes an open file descriptor.

### Behavior:
- **Releases file locks** held by the process.
- **Frees kernel resources** associated with the file descriptor.

### Automatic Closure:
- All open files are automatically closed when a process terminates (handled by the kernel).
- Explicitly closing files is still recommended to prevent resource leaks, especially in long-running programs.

---

## Key Takeaways:
- **Avoid `creat`:** Use `open()` with appropriate flags for better flexibility.
- **Always close file descriptors:** While the kernel handles cleanup on process exit, explicit closure prevents leaks in long-running programs.
- **File Locking:** Closing a file releases any locks (important for concurrent access).

These functions are fundamental for file lifecycle management in UNIX systems.


# Section 3.6: `lseek` Function

## Core Concept
- Every open file has a **current file offset**, which tracks the read/write position (default: `0` at the start of the file).  
- `lseek` explicitly sets this offset, enabling **random access** in files.

---

## Function Prototype
```c
#include <unistd.h>  
off_t lseek(int fd, off_t offset, int whence);  
```
- **Returns:** New file offset on success, `-1` on error.

---

## `whence` Parameters
- **`SEEK_SET`** → Set offset to `offset` bytes from the **start** of the file.  
- **`SEEK_CUR`** → Adjust offset by `±offset` bytes from the **current position**.  
- **`SEEK_END`** → Set offset to `offset` bytes from the **end** of the file.  

---

## Key Features

### Determine Current Offset
```c
off_t pos = lseek(fd, 0, SEEK_CUR);  
```
- Returns the current file offset.  
- **Returns `-1` (with `errno=ESPIPE`)** for non-seekable files (e.g., pipes, FIFOs, sockets).

---

### File Holes
- **Holes** are unallocated disk space created by seeking beyond EOF and writing data.  
- Reads in the hole return **zeros**, but storage isn’t used (saves disk space).  

#### Example:
```c
write(fd, "abc", 3);       // Writes 3 bytes (offset=3).  
lseek(fd, 16384, SEEK_SET); // Skips to offset 16384.  
write(fd, "XYZ", 3);       // Writes 3 bytes (offset=16387).  
```
- File size: **16,387 bytes**, but only **6 bytes** consume disk blocks.

---

### Negative Offsets
- Rarely allowed (device-dependent).  
- For regular files, offsets must be **≥0**.

---

### 64-bit Support
- `off_t` size varies (32-bit vs. 64-bit). Use `_FILE_OFFSET_BITS=64` for 64-bit offsets.  
- Check system support via `sysconf` (e.g., `_SC_V7_LP64_OFF64`).

---

## Example Use Cases
1. **Random Access:** Read/write at arbitrary positions.  
2. **Append Data:** Use `lseek(fd, 0, SEEK_END)` to write at EOF.  
3. **Detect Seekability:** Test if FD supports `lseek` (e.g., stdin in a pipeline fails).

---

## Platform Notes
- **Solaris:** `off_t` is **8 bytes** by default (64-bit).  
- **Legacy Systems:** Hardcoded values for `whence`:
  - `0` → `SEEK_SET`  
  - `1` → `SEEK_CUR`  
  - `2` → `SEEK_END`  

---

## Key Takeaway
- `lseek` enables **efficient file manipulation** (holes, random access).  
- It is critical for handling **large files (>2GB)** via 64-bit offsets.  
- Always check for **seekability** and use modern constants (`SEEK_*`) for portability.


# Section 3.7: `read` Function

## Core Functionality
```c
#include <unistd.h>  
ssize_t read(int fd, void *buf, size_t nbytes);  
```

### Purpose:
- Reads data from an open file descriptor (`fd`) into a buffer (`buf`).

### Returns:
- **`> 0`** → Number of bytes successfully read.  
- **`0`** → End-of-file (EOF) reached.  
- **`-1`** → Error (check `errno`).

---

## Key Behaviors

### Partial Reads (`bytes read < nbytes`):
- **Regular files:** EOF reached before filling the buffer.  
- **Terminals:** Typically reads one line at a time.  
- **Pipes/FIFOs:** Returns only the available data.  
- **Networks:** Buffering may limit returned bytes.  
- **Signal interruption:** Returns partial data if interrupted (see Section 10.5).

### Offset Handling:
- Reads start at the file’s **current offset**, which advances by the bytes read.

### Special Cases:
- **Record-oriented devices** (e.g., tapes): May return one record per call.

---

## POSIX.1 Updates

### Argument Types:
- **`void *buf`** → Generic pointer (ISO C compliant).  
- **`ssize_t` return** → Signed, accommodates `-1` for errors.  
- **`size_t nbytes`** → Unsigned, ensures large buffer sizes.

### Legacy Prototype:
```c
int read(int fd, char *buf, unsigned nbytes);  // Pre-POSIX.  
```

---

## Practical Notes

### Looping:
- Always handle partial reads by checking return values in **loops**.

### Non-blocking I/O:
- Returns immediately if no data is available (with `O_NONBLOCK`).

### Thread Safety:
- **Atomic** for regular files; behavior varies for other FD types.

---

## Example Usage
```c
char buffer[1024];  
ssize_t bytes_read = read(fd, buffer, sizeof(buffer));  
if (bytes_read == -1) { /* error */ }  
else if (bytes_read == 0) { /* EOF */ }  
else { /* process buffer[0..bytes_read-1] */ }  
```

---

## Key Takeaway
`read` is the foundation of file/device input in UNIX. Robust programs must handle:
- **Partial reads** → Always loop!  
- **EOF and errors** → Check return values.  
- **Offset management** → Implicit for sequential access.

### Why It's Critical:
This function is essential for **low-level I/O**, with nuances based on file type and system constraints.

# Section 3.8: `write()` Function Overview

## Function Prototype
```c
#include <unistd.h>  
ssize_t write(int fd, const void *buf, size_t nbytes);  
```

## Return Value
- **On Success:**  
  - Returns the number of bytes written (usually equal to `nbytes`).  
- **On Failure:**  
  - Returns `-1` (check `errno` for the error cause).  

---

## Common Errors
- **Disk full:** No space left on the disk.  
- **Exceeding process file size limit:** If the write exceeds the allowed file size for the process.  

---

## Key Behaviors
1. **File Offset Management:**
   - For **regular files**, writing starts at the **current file offset**.  
   - If the file was opened with **`O_APPEND`**, the offset is set to the **end of the file** before each write.  

2. **Offset Advancement:**
   - After a successful write, the file offset advances by the number of bytes written.

---

## Example Usage
```c
#include <unistd.h>
#include <fcntl.h>

int fd = open("example.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
if (fd == -1) {
    perror("open failed");
    return 1;
}

const char *data = "Hello, World!";
ssize_t bytes_written = write(fd, data, 13);

if (bytes_written == -1) {
    perror("write failed");
} else {
    printf("Wrote %zd bytes to the file.\n", bytes_written);
}

close(fd);
```

---

## Applications (Where It’s Used)
- **File Writing:** Save content to files.  
- **Logging Systems:** Append logs to a file using `O_APPEND`.  
- **Inter-Process Communication (IPC):** Write data to pipes or FIFOs.  
- **Network Programming:** Write data to sockets.  

---

## Key Takeaways
- **`write()`** is essential for low-level I/O in Unix-like systems.  
- Always check the **return value** to handle errors or partial writes.  
- Combine with **`O_APPEND`** for safe appending to files.


# Section 3.9: I/O Efficiency (File Copy Example)

## Program Overview
- **Purpose:** Copies data from **stdin** → **stdout** using `read()` and `write()`.  
- **Buffering:** Uses a fixed buffer size (`BUFFSIZE`) for reading and writing.  
- **File Handling:**  
  - Relies on **shell redirection** for input/output files (e.g., `./prog < input.txt > output.txt`).  
  - No explicit file closing—kernel handles descriptor closure on process exit.  
- **File Types:** Works for both **text and binary files** (UNIX treats them the same).  

---

## Key Observations

### 1. Buffer Size (`BUFFSIZE`) Impact
- **Optimal Performance:** Achieved when buffer size matches the file system's **block size** (e.g., `4096 bytes`).  
- **Large Buffers:** Larger than the block size show **diminishing returns**.  
- **Small Buffers:** Even small buffers (e.g., `32 bytes`) perform well due to **read-ahead optimization** (kernel prefetches data).

---

### 2. Caching Effects
- **Disk vs. RAM:** Performance improves after the first run because subsequent reads may come from the **RAM cache** (instead of disk).  
- **Testing Accuracy:** Benchmarks avoided caching by using **different file copies** for each test.

---

### 3. System Considerations
- **Filesystem Block Size:**  
  - On **ext4 (Linux)**, block size directly affects I/O efficiency.  
  - Larger block sizes favor larger buffers.  
- **Synchronous Writes:** Compared with **`stdio` buffering**, discussed in later sections (e.g., 3.14, 5.8).

---

## Takeaways
1. **Buffer Size Matters:** Efficient I/O requires tuning the buffer size to match the **filesystem block size**.  
2. **Avoid Caching Bias:** Isolate tests to avoid skewed benchmarks from **RAM caching**.  
3. **Simplified File Handling:** Shell redirection eliminates the need for explicit `open()` and `close()`.  

> *(Note: "incore" refers to in-memory operations, a term originating from ferrite-core memory systems.)*


# Section 3.10: File Sharing in UNIX Systems

## Key Data Structures for File Sharing
The kernel uses three structures to manage open files across processes:

### 1. **Process-Level File Descriptor Table**
- Each process maintains its own table of **file descriptors** (e.g., `0=stdin`, `1=stdout`).  
- Each entry contains:
  - **File descriptor flags** (e.g., `close-on-exec`).  
  - **Pointer to a file table entry** in the kernel.

---

### 2. **Kernel File Table**
- Shared among **all processes**.  
- Each entry tracks:
  - **File status flags** (e.g., `O_RDONLY`, `O_APPEND`).  
  - **Current file offset** (position for `read`/`write`).  
  - **Pointer to a v-node/i-node entry**.

---

### 3. **v-node/i-node Table**
- Contains **file metadata**, including:
  - File type, size, disk blocks, and permissions.  
- **v-node**: Used in Solaris/BSD for multi-filesystem support (e.g., NFS).  
- **i-node**: Linux uses a unified i-node instead of a v-node + i-node split.  

---

## How File Sharing Works
- Multiple processes can open the **same file**:
  - Each process gets a **separate file table entry** with its **own offset**.  
  - All processes share the **same v-node/i-node** (ensuring file metadata consistency).  

### Example: Two Processes Writing to the Same File
- Each process has **independent offsets**, unless `O_APPEND` is used.  
- Without proper synchronization, writes may overlap or overwrite each other.

---

## Critical Behaviors

### 1. **`write()` Updates**
- Increments the **file table offset** after writing.  
- If the **offset > file size**, the i-node size expands.

---

### 2. **`O_APPEND` Mode**
- Ensures **atomic writes** by forcing all writes to the **end of the file**:  
  - Offset = file size **before each write**.  

---

### 3. **`lseek()`**
- Only modifies the **file table offset** (no actual I/O occurs).  
- Unlike `O_APPEND`, it doesn’t guarantee **atomic appends**.

---

### 4. **Shared File Table Entries**
- Occurs after **`dup()`** or **`fork()`**:
  - Child processes inherit the parent’s descriptors.  
  - Shared file table entries affect offsets.

---

## Atomicity & Race Conditions
- **Concurrent writes** without `O_APPEND` risk data corruption due to **offset races**.  
- **Solutions:**
  - Use **atomic operations** (e.g., `O_APPEND`).  
  - Use **file locks** (e.g., `fcntl`) for synchronization.

---

## Implementation Notes
- **v-node** was introduced for supporting multiple filesystems (e.g., NFS).  
- Linux simplifies this with a unified **i-node** structure.

---

## Takeaway
- File sharing in UNIX relies on three kernel structures:  
  - **Process-level descriptors** for isolation.  
  - **Kernel file table & v-node/i-node** for synchronization.  

- **`O_APPEND`** ensures **atomic writes**. Without it, processes may overwrite each other.  
- **Shared file table entries** (via `dup()`/`fork()`) can affect offsets during file operations.


# Section 3.11: Atomic Operations in UNIX: Ensuring Safe Concurrent File Access

---

## 1. Problem: Non-Atomic Appends (Race Condition)

### Scenario:
Two processes (A and B) attempt to append data to the same file **without `O_APPEND`**:

```c
lseek(fd, 0, SEEK_END);  // Move to end
write(fd, buf, 100);     // Write data
```

### Race Condition:
1. Process A calls `lseek` to move to EOF (offset = 1500), then gets preempted.  
2. Process B calls `lseek` to move to EOF (still offset = 1500) and writes 100 bytes (new EOF = 1600).  
3. Process A resumes and writes 100 bytes at its saved offset (1500), **overwriting Process B’s data**.

---

### Solution: `O_APPEND` Flag

#### How it works:
```c
open("file.log", O_WRONLY | O_APPEND);  // Atomic seek+write
```
- The **kernel automatically sets `offset = EOF` before each `write()`**.
- Guarantees **atomicity**: Ensures no interleaving between `lseek` and `write`.

---

## 2. Atomic Seek+I/O: `pread()` and `pwrite()`

### Functions:
```c
ssize_t pread(int fd, void *buf, size_t n, off_t offset);  // Atomic read at offset
ssize_t pwrite(int fd, const void *buf, size_t n, off_t offset);  // Atomic write at offset
```

### Key Features:
1. **Atomic Seek+I/O:** Combines `seek` and `read`/`write` into one atomic step.  
2. **No Race Conditions:** Prevents interference with the file offset.  
3. Does **not** update the file offset (unlike `lseek` + `read`/`write`).

---

### Example:
```c
pwrite(fd, "Atomic!", 7, 100);  // Writes "Atomic!" at offset 100, no interference.
```

---

## 3. Atomic File Creation: `O_EXCL`

### Problem:
Without atomic checks, `open()` + `creat()` risks **TOCTOU (Time-of-Check-to-Time-of-Use) races**:
```c
if (!file_exists) creat(path, mode);  // Another process may create it meanwhile!
```

---

### Solution: Use `O_CREAT | O_EXCL`
```c
fd = open(path, O_WRONLY | O_CREAT | O_EXCL, mode);  // Fails if file exists.
```
- Ensures **atomic check-and-create**: The kernel guarantees no other process creates the file in the meantime.

---

## Why Atomicity Matters

### 1. Log Files:
- Multiple processes safely append data using **`O_APPEND`**.

### 2. Database Writes:
- Use **`pwrite()`** to avoid corrupting records during concurrent writes.

### 3. Temporary Files:
- Use **`O_EXCL`** to prevent accidental overwrites.

---

## Key Takeaways
1. **Non-atomic operations** (`lseek` + `write`) are unsafe with concurrency.  
2. **`O_APPEND`**: Ensures atomic appends.  
3. **`pread/pwrite`**: Provides atomic I/O at fixed offsets.  
4. **`O_EXCL`**: Guarantees atomic file creation (**no TOCTOU races**).  

---

### Rule of Thumb:
Always use **atomic operations** when shared file access is possible!


# Section 3.12: `dup` and `dup2` Functions: Duplicating File Descriptors

## Purpose
- Duplicate an existing **file descriptor (fd)** to create a new one that refers to the **same file table entry**.  
- Commonly used for **redirecting I/O** (e.g., making `stdout` write to a file).

---

## Functions
```c
#include <unistd.h>  
int dup(int fd);          // Returns lowest available fd  
int dup2(int fd, int fd2); // Forces fd2 to be a copy of fd  
```
- **Returns:** New file descriptor (or `-1` on error).

---

## Key Behaviors

### 1. `dup(int fd)`
- Returns the **lowest unused fd** (e.g., if `fd=1` (stdout), `dup(1)` might return `3`).  
- Shares the **same file table entry** (same offset, same open file).  

#### Example:
```c
int newfd = dup(1);  // Assume newfd = 3 (points to stdout)
write(newfd, "Hello", 5);  // Prints "Hello" to stdout!
```

---

### 2. `dup2(int fd, int fd2)`
- Makes `fd2` a copy of `fd` (closes `fd2` first if open).  
- If `fd == fd2`, simply returns `fd2` without closing it.  
- **Atomic:** Safer than `close` + `fcntl`.

#### Example: Redirect stdout to a file
```c
int fd = open("log.txt", O_WRONLY);  
dup2(fd, 1);  // Now fd=1 (stdout) writes to "log.txt"
printf("This goes to log.txt!\n");  
```

---

## Key Differences vs `fcntl(F_DUPFD)`

| **Feature**           | **dup/dup2**      | **fcntl(F_DUPFD)** |
|------------------------|-------------------|--------------------|
| **Atomic?**           | Yes (`dup2`)   | No (race risk)  |
| **Closes `fd2`?**     | Yes (`dup2`)   |  No (must close manually) |
| **Error Handling**    | Simpler           | More flexible      |

---

## Why `dup2` is Preferred
1. **Atomicity:** No risk of race conditions (e.g., signals interrupting `close` + `fcntl`).  
2. **Predictable:** Guarantees `fd2` is reused.  

---

## Shared File Table Entry Implications
- **Same offset:** Writing via one fd advances the offset for all duplicates.
```c
int fd1 = open("file.txt", O_RDWR);  
int fd2 = dup(fd1);  
write(fd1, "123", 3);  
write(fd2, "456", 3);  // Appends after "123" (shared offset)  
// File content: "123456"
```

- **Same status flags:** If `fd1` was opened with `O_APPEND`, `fd2` inherits it.

---

## When to Use?

### 1. **I/O Redirection**
- Shells use `dup2` for pipes (e.g., `cmd1 | cmd2`).

### 2. **Reopening FDs Safely**
- Ensure a backup of `stdin`/`stdout` before redirecting.

### 3. **Avoiding Race Conditions**
- Use `dup2` instead of manual `close` + `fcntl`.

---

## Summary
- **`dup`:** Simplest way to clone an fd (uses the lowest available fd).  
- **`dup2`:** Forces a specific fd number (**atomic and safe**).  
- Both share the **same underlying file state** (offset, flags).  

#### Use `dup2` for robust redirection (e.g., in shells, daemons).

---

### 🔍 Try It: Redirect stderr to a file using `dup2`!
```c
int fd = open("errors.log", O_WRONLY);  
dup2(fd, 2);  // Now stderr (fd=2) writes to "errors.log" 
perror("An error occurred!");  // This message goes into "errors.log"
```


# Section 3.13: `sync`, `fsync`, and `fdatasync`: Ensuring Data Persistence on Disk

---

## Purpose
- Force **kernel buffer cache** → disk writes to prevent **data loss** if the system crashes.  
- **Critical** for:
  - Databases  
  - Transactional logs  
  - File integrity  

---

## Key Functions

| **Function**     | **Behavior**                                      | **Use Case**                          |
|-------------------|---------------------------------------------------|---------------------------------------|
| **`sync()`**      | Queues all pending disk writes (**no waiting**).  | System-wide flush (e.g., `sync` command). |
| **`fsync(fd)`**   | Forces writes for **one file** (**waits for disk**). | Databases (ensure commits are safe).  |
| **`fdatasync(fd)`** | Like `fsync`, but skips metadata (e.g., timestamps). | Performance-critical data writes.    |

---

## 1. `sync()`
- **Asynchronous:**  
  - Schedules **all buffered writes** but does **not wait** for them to complete.  
- Called **automatically** every 30 seconds (by the update daemon).  
- **Command Line Example:**  
  ```bash
  $ sync  # Manually triggers a system-wide flush.
  ```

---

## 2. `fsync(int fd)`
- **Synchronous:**  
  - Blocks until **data + metadata** (e.g., file size, timestamps) are safely written to disk.  

### Example: Ensuring a database transaction is saved
```c
write(db_fd, transaction_data, size);  
fsync(db_fd);  // Wait until disk confirms the write.
```

---

## 3. `fdatasync(int fd)`
- **Optimized `fsync`:**  
  - Only flushes **file data**, skipping metadata (e.g., timestamps).  
- **Faster** for apps that don’t need immediate metadata consistency (e.g., logs).  

### Example: `fsync` vs `fdatasync`
```c
write(fd, "Payment: $100", 12);  
fsync(fd);      // Ensures "Payment: $100" + file size/timestamp are saved.  
fdatasync(fd);  // Only ensures "Payment: $100" is saved (faster).  
```

---

## Why This Matters

### 1. **Delayed Write Default**
- The kernel buffers writes to improve performance.  
- **Risk:** A crash before flushing = **data loss**.

### 2. **Metadata Overhead**
- `fsync` updates inodes (file metadata), while `fdatasync` **skips metadata** for better performance.

---

## Portability Note
- **`fdatasync`** is **not supported** on FreeBSD 8.0.  
- Use **`fsync`** as a fallback for better portability.

---

## When to Use?

| **Scenario**                   | **Function**       |
|---------------------------------|--------------------|
| System-wide flush (e.g., before reboot) | **`sync()`**        |
| Critical file writes (e.g., databases)  | **`fsync(fd)`**     |
| High-speed data logging                | **`fdatasync(fd)`** |

---

## 💡 Rule of Thumb:
- Use **`fsync`/`fdatasync`** after a write if you **cannot afford data loss**.

---

## Summary
- **`sync:`** Flush all buffers (**fire-and-forget**).  
- **`fsync:`** Guarantee a file’s **data + metadata** is written to disk.  
- **`fdatasync:`** Like `fsync`, but **faster** (skips metadata).  

---

### Tradeoff: Safety (`fsync`) vs Speed (`fdatasync`)

**Key Takeaway:** If your app handles money or critical data, always use **`fsync`**!


# Section 3.14: `fcntl` Function: Controlling Open File Descriptors

---

## Purpose
The **`fcntl`** (file control) function modifies properties of an already **open file descriptor (fd)**.  
It supports various operations, including:

1. **Duplicating file descriptors** (`F_DUPFD`, `F_DUPFD_CLOEXEC`)  
2. **Managing file descriptor flags** (`F_GETFD`, `F_SETFD`)  
3. **Managing file status flags** (`F_GETFL`, `F_SETFL`)  
4. **Asynchronous I/O ownership** (`F_GETOWN`, `F_SETOWN`)  
5. **Record locking** (`F_GETLK`, `F_SETLK`, `F_SETLKW`)

---

## Key Commands & Examples

### 1. **Duplicating File Descriptors**
- **`F_DUPFD:`** Duplicates `fd` to the lowest available fd ≥ `arg`.  
  ```c
  int newfd = fcntl(fd, F_DUPFD, 3);  // New fd ≥ 3
  ```

- **`F_DUPFD_CLOEXEC:`** Similar to `F_DUPFD`, but sets **`FD_CLOEXEC`** (close-on-exec).  

---

### 2. **File Descriptor Flags**
- **`F_GETFD:`** Retrieves flags (e.g., **`FD_CLOEXEC`**).  
  ```c
  int flags = fcntl(fd, F_GETFD);  
  ```

- **`F_SETFD:`** Sets flags (e.g., enables `FD_CLOEXEC`).  
  ```c
  fcntl(fd, F_SETFD, FD_CLOEXEC);  // Close fd on exec()
  ```

---

### 3. **File Status Flags**
- **`F_GETFL:`** Retrieves file status flags (e.g., `O_RDWR`, `O_APPEND`).  
  ```c
  int flags = fcntl(fd, F_GETFL);  
  if (flags & O_APPEND) printf("Append mode enabled");  
  ```

- **`F_SETFL:`** Updates file status flags (only certain flags like `O_APPEND`, `O_NONBLOCK`, `O_SYNC`).  
  ```c
  fcntl(fd, F_SETFL, flags | O_APPEND);  // Enable append mode  
  ```

---

### 4. **Asynchronous I/O Ownership**
- **`F_GETOWN:`** Retrieves the process/group receiving `SIGIO`/`SIGURG`.  
- **`F_SETOWN:`** Sets the process (arg > 0) or group (arg < 0) to receive signals.  
  ```c
  fcntl(fd, F_SETOWN, getpid());  // Send signals to this process  
  ```

---

## Critical Notes

### 1. **Atomicity**
- `fcntl` operations are **atomic**, unlike manual `close` + `dup`.

### 2. **Flag Management**
- Always **read-modify-write** flags to avoid overwriting existing settings:  
  ```c
  int flags = fcntl(fd, F_GETFL);  
  fcntl(fd, F_SETFL, flags | O_NONBLOCK);  // Add nonblocking mode  
  ```

### 3. **Access Modes**
- Use **`O_ACCMODE`** to check access modes (`O_RDONLY`, `O_WRONLY`, `O_RDWR`):  
  ```c
  switch (flags & O_ACCMODE) {  
    case O_RDONLY: /* Read-only */ break;  
    case O_WRONLY: /* Write-only */ break;  
    case O_RDWR:   /* Read-write */ break;  
  }  
  ```

---

## Practical Use Cases

### 1. Enable Nonblocking I/O
```c
fcntl(socket_fd, F_SETFL, O_NONBLOCK);  
```

### 2. Force Synchronous Writes (`O_SYNC`)
```c
fcntl(fd, F_SETFL, O_SYNC);  // Each write() waits for disk  
```

### 3. Shell Redirection
- Modify the flags of inherited descriptors (e.g., `stdout`).  

---

## Performance Implications

### 1. **`O_SYNC`/`O_DSYNC`**
- Ensures **data integrity** but slows writes (see Section 3.13).  

### 2. **`O_NONBLOCK`**
- Useful for sockets/pipes to avoid blocking operations.  

---

## Summary

- **`fcntl`** is the Swiss Army knife for file descriptor control.  
- Key Uses:
  - Duplicate file descriptors.  
  - Toggle flags (`O_APPEND`, `O_NONBLOCK`).  
  - Manage asynchronous I/O.  

- **Best Practices:**
  - Always use `F_GETFL` before `F_SETFL` to preserve existing flags.  
  - Prefer `fcntl` over `dup2` for advanced descriptor management (e.g., setting `FD_CLOEXEC`).  

---

**Pro Tip:** Use `fcntl` to enable nonblocking mode or ensure atomic flag modifications!


# Section 3.15: `ioctl` Function

---

## Overview

- **Purpose:**  
  The `ioctl` (I/O control) system call is a **versatile** tool used for performing **device-specific operations** that are not possible with standard I/O functions (`read`, `write`, `lseek`, etc.).

- **Usage:**  
  Commonly used for:
  - **Terminal I/O**  
  - Miscellaneous **device operations** (e.g., disks, tapes, sockets, etc.).

- **Status:**  
  Classified as **obsolescent** in SUSv4 (Single UNIX Specification, Version 4).  
  However, it remains widely used in UNIX systems.

---

## Function Prototype

```c
#include <unistd.h>  /* System V */  
#include <sys/ioctl.h>  /* BSD and Linux */  

int ioctl(int fd, int request, ...);  
```

- **Returns:**  
  - `-1` on error.  
  - Implementation-defined value on success.  

- **Arguments:**
  1. **`fd:`** File descriptor for the target device.  
  2. **`request:`** A predefined command (usually a `#define` constant).  
  3. **`...:`** Optional additional arguments (typically a pointer to a struct or variable).  

---

## Key Points

### 1. **Device-Specific Commands**
- Each **driver** defines its own `ioctl` commands.  
- Common categories:
  - **Disk I/O**  
  - **Terminal control**  
  - **Tape operations**  
  - **Socket management**  

- **Example Headers:**
  - `<termios.h>`  
  - `<sys/disklabel.h>`  
  - `<sys/sockio.h>`

---

### 2. **Terminal I/O**
- Historically, terminal operations relied heavily on `ioctl`.  
- **POSIX.1** later replaced many terminal-specific `ioctl` calls with dedicated functions (covered in Chapter 18).

---

### 3. **Variations Across Systems**
- **FreeBSD/MacOS:**  
  Declare the `request` argument as `unsigned long`.  
- **Extended Support:**  
  Some UNIX systems extend `ioctl` to regular files.

---

## Common Use Cases

1. **Tape Operations:**  
   - Rewind tape, write EOF marks.  

2. **Terminal Window Size Manipulation:**  
   - Resize terminal windows (see Section 18.12).  

3. **Pseudo-Terminal Advanced Features:**  
   - Advanced operations for pseudo-terminals (see Section 19.7).  

---

## Example Categories (FreeBSD)

| **Category**         | **Header**          | **Number of Commands** |
|-----------------------|---------------------|-------------------------|
| **Disk Labels**       | `<sys/disklabel.h>` | 4                       |
| **File I/O**          | `<sys/filio.h>`     | 14                      |
| **Magnetic Tape I/O** | `<sys/mtio.h>`      | 11                      |
| **Socket I/O**        | `<sys/sockio.h>`    | 73                      |
| **Terminal I/O**      | `<sys/ttycom.h>`    | 43                      |

---

## Conclusion
- **`ioctl`** remains a **"catch-all"** for non-standard I/O operations.  
  - Gradually being replaced by **POSIX-specific functions**.
- Essential for **low-level device control**, especially in **legacy** and **specialized hardware interfaces**.

---

## Next Steps
1. **Chapter 18:**  
   Explore POSIX terminal functions replacing `ioctl`.  
2. **Section 18.12 & 19.7:**  
   Demonstrates practical `ioctl` usage for terminal and pseudo-terminal operations.

# Section 3.16: `/dev/fd`

---

## Overview

- **Purpose:**  
  `/dev/fd` is a directory containing files named `0`, `1`, `2`, etc., corresponding to **open file descriptors**.

- **Functionality:**  
  Opening `/dev/fd/n` duplicates descriptor `n` (similar to `dup(n)`).

- **Origin:**  
  Introduced in **Research UNIX (8th Edition)** by Tom Duff.

- **Supported Systems:**  
  Available in **FreeBSD, Linux, macOS, Solaris**.  
  *(Not part of POSIX.1.)*

---

## Key Behaviors

### 1. **Opening `/dev/fd/n`**
- **Equivalent to `dup(n)`**, sharing the same file table entry.
- Most systems **ignore the mode** in `open("/dev/fd/n", mode)` and enforce the original descriptor’s mode.
  - Example: If `n` is read-only, the new descriptor will also be read-only.

- **Linux Exception:**  
  `/dev/fd/n` is mapped to **symbolic links** of the actual files.  
  - This means the **new descriptor’s mode** may differ from the original.

---

### 2. **`creat` and `O_CREAT`**
- Works with `/dev/fd` paths (e.g., `creat("/dev/fd/1", ...)`).

- **Linux Warning:**  
  Using `creat` on `/dev/fd` may **truncate the underlying file** due to symlink resolution.  

---

### 3. **Convenience Symlinks**
Some systems provide easier-to-read symlinks for commonly used descriptors:
- `/dev/stdin` → `/dev/fd/0`  
- `/dev/stdout` → `/dev/fd/1`  
- `/dev/stderr` → `/dev/fd/2`  

---

## Primary Use Case: Shell Commands

- Enables programs to treat **stdin/stdout** like **regular files**, avoiding special cases (e.g., `-` for stdin in `cat`).

### Example:

#### Traditional (using `-` for stdin):
```sh
filter file2 | cat file1 - file3 | lpr  
```


