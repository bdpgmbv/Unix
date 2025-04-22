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
