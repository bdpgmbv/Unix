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
