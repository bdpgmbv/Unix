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
