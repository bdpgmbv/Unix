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

## 🔴 `open()` is Vulnerable to TOCTTOU

### Example: A Privileged Program Backing Up a File
Suppose a backup tool does this:
```c
// Check if the file exists
if (file_exists("/tmp/user_data.txt")) {
    // Open the file
    int fd = open("/tmp/user_data.txt", O_RDONLY);
}
```

### 💥 Attack:
1. After the **check**, an attacker replaces `user_data.txt` with a **symlink** to `/etc/passwd`.  
2. When the program opens the file, it ends up reading sensitive system files like `/etc/passwd`.

### Why?
- `open()` uses **absolute paths**, so if the file changes between the check and open, the program is fooled.

---

## ✅ `openat()` Fixes TOCTTOU

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

### ✅ Why This is Safe:
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
