# File I/O in UNIX (open, read, write, lseek, close)

## 1. How It Is Used in Programs
In UNIX, programs often need to work with files—to read from them or write to them. To do this, they use five basic functions:

- **open()** → Opens a file (like opening a book to read or write).  
- **read()** → Reads data from the file (like reading words from a book).  
- **write()** → Writes data to the file (like writing notes in a book).  
- **lseek()** → Moves the "cursor" in the file (like flipping pages to a specific part).  
- **close()** → Closes the file (like closing the book when done).  

These functions talk directly to the computer's kernel (the core part of the OS), so they are called "unbuffered I/O" (no extra memory is used to hold data before reading/writing).

---

## 2. Applications (Where These Are Used)
These functions are used in many programs, such as:

- **Text Editors** (like Vim or Nano) → To open, edit, and save files.  
- **File Copiers** → To read one file and write to another.  
- **Databases** → To store and retrieve data from files.  
- **System Tools** (like `cat`, `grep`) → To read and process files.  

---

## 3. Important Concepts
- **Atomic Operations** → Ensures an action (like writing) happens completely without interruption (important when multiple programs use the same file).  
- **File Sharing** → Allows many programs to read/write the same file, with UNIX safely controlling access.  
- **Extra Functions** →  
  - **dup()** → Duplicates file descriptors.  
  - **fcntl()** → Performs file control operations.  
  - **sync()** → Forces writes to disk.

---

# Section 3.2 File Descriptors in UNIX

## 1. What is a File Descriptor?
- A file descriptor is just a number (like 0, 1, 2, 3, ...) that the UNIX kernel uses to keep track of open files.  
- When a program opens a file, the kernel gives it a file descriptor (like a ticket number).  
- The program uses this number to read, write, or close the file.

---

## 2. How It Is Used in Programs

### Example:
```c
int fd = open("example.txt", O_RDONLY); // Open file, get descriptor (e.g., 3)
read(fd, buffer, 100);                  // Read 100 bytes using descriptor 3
close(fd);                              // Close the file
```

### Standard File Descriptors (Always Open):
- **0 (STDIN_FILENO)** → Input (like keyboard).  
- **1 (STDOUT_FILENO)** → Output (like screen).  
- **2 (STDERR_FILENO)** → Error messages (also screen).  

---

## 3. Applications (Where File Descriptors Are Used)
- **Shell Commands** (e.g., `cat`, `grep`, `echo`) → Use 0 (input), 1 (output), 2 (error).  
- **Servers & Databases** → Handle many open files (sockets, logs, data files).  
- **Pipes & Redirections** (e.g., `ls > files.txt`) → Change where 1 (stdout) goes.  

---

## 4. Important Notes
- **Numbers vs. Names** → It’s better to use `STDIN_FILENO` instead of `0` for readability.  
- **Limits** → Older UNIX allowed only ~20 open files, but modern systems allow much more (depends on system memory).  

---

## Summary
- File descriptors are numbers representing open files.  
- **0, 1, 2** are special (stdin, stdout, stderr).  
- Programs use them to read/write files without needing filenames every time.  
- File descriptors are used in almost every UNIX program (shells, servers, scripts).

## Summary
These five functions (**open, read, write, lseek, close**) are the basic tools for file operations in UNIX. They are fast (no extra buffering) and are used in many programs that work with files.

# Section 3.3 `open()` and `openat()` Functions

## 1. What Do `open()` and `openat()` Do?
These functions open or create files in UNIX. They return a **file descriptor** (a number) that the program uses to read/write the file.

- **`open()`** → Opens a file at a given path (e.g., `open("file.txt", O_RDWR)`).  
- **`openat()`** → Opens a file relative to a directory (useful for threads & security).  

---

## 2. How They Are Used in Programs
```c
#include <fcntl.h>  

// Open a file for reading  
int fd = open("example.txt", O_RDONLY);  

// Open (or create) a file for writing  
int fd2 = open("newfile.txt", O_WRONLY | O_CREAT, 0644);  

// Open a file relative to a directory (using openat)  
int dir_fd = open("/some/directory", O_RDONLY);  
int file_fd = openat(dir_fd, "file.txt", O_RDWR);  
```

### Common Flags:
- **`O_RDONLY`** → Open for reading only.  
- **`O_WRONLY`** → Open for writing only.  
- **`O_RDWR`** → Open for both reading and writing.  
- **`O_CREAT`** → Create the file if it doesn’t exist (needs file permissions like `0644`).  
- **`O_APPEND`** → Always write at the end of the file (good for logs).  
- **`O_TRUNC`** → Delete all data in the file if it already exists.  

---

## 3. Why Use `openat()`?
- **Thread Safety** → Different threads can work in different directories.  
- **Security** → Avoids race conditions (**TOCTTOU attacks**) where a file changes between checks.  
- **Relative Paths** → Opens files inside a directory without needing full paths.  

---

## 4. Applications (Where These Are Used)
- **Text Editors** → Open, modify, and save files.  
- **Shell Commands** (`cat`, `echo`, `grep`) → Read/write files.  
- **Servers & Databases** → Open many files safely.  
- **Logging Systems** → Use `O_APPEND` to safely add logs.  

---

## 5. Important Notes
- **File Descriptors** → The smallest available number is given (e.g., if `0` (stdin) is closed, `open()` might return `0`).  
- **Errors** → If the file doesn’t exist, `open()` returns `-1` (check with `errno`).  
- **Filename Limits** → Some old systems truncate long names, but modern systems give an error (`ENAMETOOLONG`).  

---

## Summary
- **`open()`** → Basic way to open files.  
- **`openat()`** → Safer for threads and security.  
- **Used Everywhere** → Editors, shells, servers, and more.  
- **Options Control** → Read/write, create, append, etc.

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


# Section 3.4 - Section 3.5: `creat()` and `close()` Functions

## 1. `creat()` - Create a New File

### What it does?
- **Creates a new file** (or overwrites an existing one).  
- Opens it for **writing only** (you can’t read from it).

### How to use it?
```c
#include <fcntl.h>
int fd = creat("newfile.txt", 0644); // 0644 = file permissions (read/write for owner, read for others)
```
- **Returns:** A file descriptor (`fd`) if successful, `-1` on error.

### Why is it outdated?
- `open()` can do the same thing but better:
```c
int fd = open("newfile.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
```
- `creat()` is just a **legacy function** from old UNIX.

---

## 2. `close()` - Close an Open File

### What it does?
- Closes a file and **frees its file descriptor** for reuse.  
- Releases **locks** on the file (important for multi-process programs).

### How to use it?
```c
#include <unistd.h>
int fd = open("file.txt", O_RDONLY);  
// ... read/write operations ...
close(fd); // Close the file when done!
```
- **Returns:** `0` on success, `-1` on error.

### What happens if you don’t close?
- The OS **automatically closes all open files** when a program exits.  
- However, it’s **bad practice** to rely on this—always close files manually to avoid resource leaks.

---

## Applications (Where These Are Used)

### File Creation:
- **`creat()` (or `open()`)** → Used in programs that need to generate new files (e.g., compilers, loggers).

### File Cleanup:
- **`close()`** → Used in every program that opens files (editors, web servers, databases).

### Temporary Files:
#### Old way:
```c
int fd = creat("temp.txt", 0644);  
close(fd);  
fd = open("temp.txt", O_RDWR);  
```

#### Better way:
```c
int fd = open("temp.txt", O_RDWR | O_CREAT | O_TRUNC, 0644);  
```

---

## Key Takeaways:
-  **`creat()`** → Old way to create and open a file (use `open()` instead today).  
-  **`close()`** → Always close files to free resources.  
-  Modern code should use `open()` with flags (`O_CREAT | O_TRUNC`) instead of `creat()`.

# Section 3.6: `lseek()` Function

## 1. What is `lseek()`?
- Every open file has a **current position** (like a cursor in a text file).  
- `lseek()` moves this position to **read/write** at different places in the file.

---

## 2. How to Use `lseek()`?
```c
#include <unistd.h>  
off_t new_pos = lseek(int fd, off_t offset, int whence);  
```

### Parameters:
- **`fd`** → File descriptor (from `open()`).  
- **`offset`** → How many bytes to move.  
- **`whence`** → Where to start moving:  
  - **`SEEK_SET`** → Start from beginning of file.  
  - **`SEEK_CUR`** → Start from current position.  
  - **`SEEK_END`** → Start from end of file.  

---

## 3. Examples

### Jump to byte 100:
```c
lseek(fd, 100, SEEK_SET);  
```

### Move 50 bytes forward from current position:
```c
lseek(fd, 50, SEEK_CUR);  
```

### Go to 10 bytes before the end:
```c
lseek(fd, -10, SEEK_END);  
```

### Find current position:
```c
off_t pos = lseek(fd, 0, SEEK_CUR);  
```

---

## 4. Applications (Where It’s Used)

### Reading/Writing at Specific Positions
- **Databases** → Jump to a record.  
- **Video Players** → Skip to a timestamp.  

### Creating Sparse Files (Files with Holes)
**Example:**
```c
write(fd, "start", 5);  
lseek(fd, 1000, SEEK_CUR); // Skip 1000 bytes  
write(fd, "end", 3);  
```
- Saves disk space → **Holes** (unwritten parts) aren’t stored physically.  

### Checking if a File Supports Seeking
- Pipes, sockets, and FIFOs can’t seek → `lseek()` returns `-1`.

---

## 5. Important Notes
-  Works on regular files, but **not pipes/sockets**.  
-  Negative offsets allowed (but rare).  
-  Can seek past file end → Next `write()` extends the file.  
-  **Holes in files** → Unwritten bytes read as `0` but take no disk space.

---

## Key Takeaways
- **`lseek()`** → Moves the file’s read/write position.  
- Used for **random access** (like skipping parts of a file).  
- Helps create **space-efficient sparse files**.  
- Essential for databases, media players, and file utilities.


# Section 3.7: `read()` Function Overview

## 1. What Does `read()` Do?
- Reads data from an **open file** (or device) into a **buffer** (memory space).  
- **Returns:**  
  - The number of bytes read.  
  - `0` → End-of-file (EOF).  
  - `-1` → Error (check `errno`).

---

## 2. How to Use `read()`?
```c
#include <unistd.h>  
char buffer[100];  
ssize_t bytes_read = read(fd, buffer, sizeof(buffer));  
```

### Parameters:
- **`fd`** → File descriptor (from `open()`).  
- **`buffer`** → Where to store the read data.  
- **`nbytes`** → Maximum bytes to read (e.g., `100`).  

---

## 3. Key Behaviors

-  **Regular Files** → Reads up to `nbytes` (stops at end-of-file).  
-  **Terminals** (keyboard input) → Reads **1 line** at a time by default.  
-  **Pipes/Network** → May return **fewer bytes** than requested (due to buffering).  
-  **Error Handling** → Returns `-1` if interrupted (e.g., by a signal).  

---

## 4. Example: Reading a File
```c
int fd = open("file.txt", O_RDONLY);  
char buf[256];  
ssize_t n = read(fd, buf, sizeof(buf));  

if (n > 0)  write(STDOUT_FILENO, buf, n); // Print what was read  
else if (n == 0) printf("End of file!\n");  
else perror("read failed");  

close(fd);  
```

---

## 5. Applications (Where It’s Used)

- **File Editors** → Load files into memory.  
- **Web Servers** → Read HTML files to send to browsers.  
- **Databases** → Fetch records from disk.  
- **Shell Commands** (`cat`, `grep`) → Process file contents.  
- **Device Drivers** → Read input from hardware (e.g., keyboards).  

---

## 6. Important Notes

-  **Partial Reads** → Always check `bytes_read` (could be less than `nbytes`).  
-  **Blocking vs. Non-blocking** → By default, `read()` waits for data (unless `O_NONBLOCK` is set).  
-  **Thread Safety** → Concurrent `read()` calls on the same file may cause race conditions.  

---

## Summary
- **`read()`** → Gets data from files/devices into memory.  
- Used in almost all programs that process files/input.  
- Always check the **return value** to handle errors and partial reads.


# Section 3.8 - `write` Function

## What Does `write` Do?
The `write` function is used to **save (write) data** into an open file.

---

## Basic Syntax
```c
ssize_t write(int fd, const void *buf, size_t nbytes);
```

### Parameters:
1. **`fd`** → The file descriptor (a number representing the open file).  
2. **`buf`** → The data to write (e.g., text or numbers).  
3. **`nbytes`** → Number of bytes (letters/numbers) to write.  

---

## Return Value
- **If successful:** Returns the **number of bytes written**.  
- **If it fails:** Returns `-1` (check `errno` for the error).  

---

## Common Errors
1. **Disk is full** → No space left to save the data.  
2. **File is too big** → Writing exceeds the file size limit for the process.  

---

## How Writing Works
1. Writing **starts at the current position** in the file.  
2. If the file was opened with **`O_APPEND`**, new data is **always added to the end** of the file.  
3. After writing, the file’s position advances by the number of bytes written.  

---

## Example: Writing to a File
```c
#include <unistd.h>
#include <fcntl.h>

int fd = open("example.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
if (fd == -1) {
    perror("open failed");
    return 1;
}

const char *data = "Hello";
ssize_t bytes_written = write(fd, data, 5); // Write "Hello" (5 bytes).

if (bytes_written == -1) {
    perror("write failed");
} else {
    printf("Wrote %zd bytes to the file.\n", bytes_written);
}

close(fd);
```
### What Happens in the Example:
- It saves `"Hello"` where the file’s pointer is.  
- The pointer moves forward by 5 bytes.  
- If there’s an error (e.g., disk full), it returns `-1`.  

---

## Key Takeaways
- **`write`** saves data into files and moves the file pointer forward.  
- Always check the **return value** to ensure the operation succeeded.  
- Use **`O_APPEND`** to always add data to the end of the file.  

This is how you save data to a file in C!

# Section 3.9: I/O Efficiency

This program copies data from **input** (like a file or keyboard) to **output** (like a file or screen).

---

## How It Works:
1. It reads data in **chunks (blocks)** using `read()`.  
2. **`BUFFSIZE`** decides how much data to read at once (e.g., `4096 bytes`).  
3. It writes the same data using `write()`.  
4. It keeps repeating until there’s no more data (`read()` returns `0`).  
5. If there’s an error, it stops and shows a message.  

---

## Important Points:

### ✔ Uses Shell Redirection
- The program **doesn’t open files itself**.  
- Instead, it reads from **standard input (`STDIN_FILENO`)** and writes to **standard output (`STDOUT_FILENO`)**.  
- The shell (command line) handles redirection, e.g.,  
  ```bash
  ./program < input.txt > output.txt
  ```

---

### ✔ No Need to Close Files
- The system **automatically closes all open files** when the program ends.

---

### ✔ Works for Text & Binary Files
- **UNIX treats all files the same,** so it works for both text and binary files.

---

## Why Does `BUFFSIZE` Matter?

- Reading data in **small chunks** (e.g., `32 bytes`) is slow because it makes **many system calls**.  
- Reading in **bigger chunks** (e.g., `4096 bytes`) is faster because:  
  1. The system **reads ahead** (loads extra data in memory for future use).  
  2. It matches the **file system’s block size** (e.g., `4096 bytes` on Linux).  
- **Too large buffers** (e.g., `1MB`) don’t help much.

---

## Testing Performance:
1. The **first run** is slower because the file is read from **disk**.  
2. **Later runs** are faster because the file is **cached in memory**.  
3. To test fairly, each test used a **different file copy** to avoid cache effects.

---

## Simple Summary:
- **Read big chunks** (like `4096 bytes`) for better speed.  
- Let the **shell handle file redirection** (`< input > output`).  
- The system **caches files**, so repeated runs are faster.  

This is how you efficiently copy files in C!


# Section 3.10: File Sharing in UNIX

In UNIX, multiple programs (processes) can share the same open file. Here's how the system manages this:

---

## 3 Key Data Structures in the Kernel

The kernel (core part of the OS) uses **three tables** to keep track of open files:

### 1. **Process File Descriptor Table**
- Each process has its own list of **file descriptors** (e.g., `0=stdin`, `1=stdout`, `3=some_file`).  
- Each entry contains:
  - **File descriptor flags** (e.g., `close-on-exec`).  
  - A **pointer to a File Table Entry**.

---

### 2. **File Table (Shared Across Processes)**
- Stores information about **how the file is opened**, including:
  - **File status flags** (e.g., `O_RDONLY`, `O_WRONLY`, `O_APPEND`).  
  - **Current file offset** (where the next `read`/`write` happens).  
  - A **pointer to the v-node** (file metadata).

---

### 3. **v-node Table (File Metadata)**
- Stores permanent details about the file, such as:
  - File type, owner, size, and disk location.  
- Same for **all processes** opening the same file.  
- *(Note: Linux uses **i-nodes** instead of v-nodes, but the concept is similar.)*

---

## How File Sharing Works
When two processes open the same file, they get:
1. **Different file descriptors** (e.g., `fd=3` in Process A, `fd=4` in Process B).  
2. **Different File Table Entries** (each process has its own file offset).  
3. **The same v-node:** Both processes refer to the same file on disk.

---

## Important Behaviors

### ✔ Writing to a File
- Each `write()` updates the **file offset** in the File Table.  
- If `O_APPEND` is used, the OS ensures all writes happen **at the end of the file** (even if multiple processes are writing).  

---

### ✔ `lseek()` (Changing File Position)
- Updates the **current offset** in the File Table (no actual disk I/O).

---

### ✔ `fork()` (Creating Child Processes)
- The **child process** shares the same file descriptors as the parent.  
- This means they also share the same File Table entries, including file offsets.

---

### ✔ `dup()` (Duplicating File Descriptors)
- Creates a new file descriptor that shares the **same File Table entry**.  
- Both descriptors will have the same file offset.

---

## Problems with Multiple Writers
- If two processes write at the same time **without `O_APPEND`**, their writes can **overlap or corrupt data**.  
- **Solution:** Use atomic operations like `O_APPEND` or synchronization tools (e.g., locks) to avoid conflicts.

---

## Simple Summary
- Each process has its own **file descriptors**.  
- Multiple processes can **read/write** the same file.  
- **`O_APPEND`** ensures safe writes to the end of the file.  
- **`fork()` and `dup()`** share File Table entries and offsets.  

This is how UNIX allows multiple programs to work on the same file efficiently!


# (Vyshali's Notes): File Sharing in UNIX: Examples & Deep Dive

## 1. Basic File Descriptor Sharing
### Scenario:
Two processes open the **same file** independently.

```c
// Process A
int fd1 = open("file.txt", O_RDWR);  
read(fd1, buf, 100);  // Reads first 100 bytes  

// Process B  
int fd2 = open("file.txt", O_RDWR);  
read(fd2, buf, 50);   // Also reads first 50 bytes (independent offset)  
```

### Explanation:
- Each process has its **own file table entry** → separate offsets.  
- Shared **v-node** ensures they read the same file data.

---

## 2. Concurrent Writes & Race Conditions
### Problem:
Without synchronization, processes **overwrite each other**.

```c
// Process A  
write(fd1, "AAAAA", 5);  // Writes at offset 0  

// Process B  
write(fd2, "BBBBB", 5);  // Also writes at offset 0 (race!)  
```

### Result:
- Depending on timing, the file could end up as:
  - `"AAAAA"`  
  - `"BBBBB"`  
  - A mix like `"AABBB"` (if interleaved).

### Solution:
Use **`O_APPEND`** for atomic writes.

```c
open("file.txt", O_RDWR | O_APPEND);  // Forces writes to end  
```

### How It Works:
- Before each `write()`, the kernel sets `offset = file size`.  
- Ensures **no overlap** (but doesn’t guarantee write order).

---

## 3. `fork()` & Shared File Descriptors
### Behavior:
A child process inherits the **parent’s file descriptors**, sharing the same file table entry.

```c
int fd = open("file.txt", O_RDWR);  
if (fork() == 0) {  
    // Child  
    write(fd, "CHILD", 5);  // Advances parent’s offset too!  
} else {  
    // Parent  
    write(fd, "PARENT", 6);  
}  
```

### Result:
- Writes are **interleaved** (e.g., `"PARENTCHILD"` or `"CHILDPARENT"`).  
- Shared offset prevents races, but write order is **non-deterministic**.

---

## 4. `dup()` & Shared Offsets
### Behavior:
`dup()` creates a **new file descriptor** pointing to the same **file table entry**.

```c
int fd1 = open("file.txt", O_RDWR);  
int fd2 = dup(fd1);  

write(fd1, "HELLO", 5);  
write(fd2, "WORLD", 5);  // Appends after "HELLO" (same offset!)  
```

### Result:
- File content: `"HELLOWORLD"`.  
- **Shared offset** ensures writes don’t overlap.

---

## 5. Non-Atomic `lseek()` + `write()`
### Problem:
Manually performing `lseek` + `write` is **not atomic**, leading to races.

```c
// Process A  
off_t end = lseek(fd1, 0, SEEK_END);  // Get file size  
write(fd1, "PROC_A", 6);               // May not be at end if Process B writes!  

// Process B (runs between lseek and write of A)  
write(fd2, "PROC_B", 6);  
```

### Possible Result:
- `"PROC_BPROC_A"` (Process B wrote first).  

### Fix:
- Use **`O_APPEND`** for atomic seek+write.  
- Alternatively, use **file locks** (`fcntl`) to synchronize.

---

## Key Takeaways

### **Reads**
- Independent offsets → Safe for multiple readers.

---

### **Writes**
- **Without `O_APPEND`:** Risk of races/overwrites.  
- **With `O_APPEND`:** Atomic writes (but order is still non-deterministic).

---

### **`fork()`/`dup()`**
- **Shared offset** → Writes can interleave, but no overlap.

---

### **Atomicity**
- **`O_APPEND`:** Atomic seek+write.  
- **Manual `lseek` + `write`:** Not atomic.

---

## When to Care?
- **Log Files:**  
  Use `O_APPEND` to safely append logs from multiple processes.

- **Databases:**  
  Use **file locks** (`fcntl`) for critical sections.

- **Temp Files:**  
  Use `O_EXCL` during creation to prevent races.
