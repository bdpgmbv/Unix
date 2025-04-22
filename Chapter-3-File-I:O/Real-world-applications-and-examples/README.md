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

## Summary
These five functions (**open, read, write, lseek, close**) are the basic tools for file operations in UNIX. They are fast (no extra buffering) and are used in many programs that work with files.
