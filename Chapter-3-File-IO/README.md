# 3.1 & 3.2: File I/O Basics & File Descriptors Explained Simply  

## **3.1 File I/O Basics**  
- **What is File I/O?**  
  It’s how programs **read/write files** on a computer.  
  - 5 key functions:  
    1. `open()` – Open a file.  
    2. `read()` – Read data from a file.  
    3. `write()` – Write data to a file.  
    4. `lseek()` – Move the "cursor" in a file (like skipping parts).  
    5. `close()` – Close a file when done.  

- **Unbuffered I/O**:  
  - Every `read()` or `write()` talks directly to the **computer’s core (kernel)**.  
  - No extra layers (simple but less efficient for tiny operations).  

- **Atomic Operations**:  
  - Actions that **cannot be interrupted** (e.g., creating a file in one step to avoid errors).  


## **3.2 File Descriptors**  
- **What is a File Descriptor?**  
  A **number** the computer uses to track open files.  
  - Example: Open a file → get number `3`. Use `3` to read/write later.  

- **Standard File Descriptors**:  
  | Number | Name              | Purpose                     |  
  |--------|-------------------|-----------------------------|  
  | `0`    | `STDIN_FILENO`  | Input (like keyboard).      |  
  | `1`    | `STDOUT_FILENO` | Output (like screen).       |  
  | `2`    | `STDERR_FILENO` | Error messages (screen).    |  

- **Key Notes**:  
  - **Always use names** (e.g., `STDIN_FILENO`) instead of `0`, `1`, `2` for clarity.  
  - File descriptors start at `0`.  
  - Old systems allowed ~20 open files. **Modern systems** allow thousands (limited by memory).  


## **Why This Matters**  
- **File I/O** lets programs save/load data (e.g., text files, images).  
- **File descriptors** help the computer manage many open files at once.  

# 3.3 open & openat Functions Explained Simply

## **What They Do**  
- **`open()`** and **`openat()`** are used to **open** or **create** files.  
- Return a **file descriptor** (a number) if successful, `-1` if error.  


## **Parameters**  
1. **`path`**: File name/path to open/create.  
2. **`oflag`**: Options (see below).  
3. **`mode`**: Required only if creating a new file (sets permissions).  


## **Key Options (`oflag`)**  
**Must pick one of these first:**  
- `O_RDONLY`: Open for **reading only**.  
- `O_WRONLY`: Open for **writing only**.  
- `O_RDWR`: Open for **read + write**.  
- `O_EXEC`: Open for **executing** (like running a program).  
- `O_SEARCH`: Open for **searching** (for directories).  

**Optional Flags (combine with `|`):**  
- `O_APPEND`: Add new data to the **end** of the file.  
- `O_CREAT`: **Create** the file if it doesn’t exist.  
- `O_EXCL`: Error if `O_CREAT` is used and file **already exists**.  
- `O_TRUNC`: **Empty** the file if it exists.  
- `O_NONBLOCK`: Don’t wait for slow devices (like keyboards).  
- `O_SYNC`: Wait until data is **safely written** to disk.  


## **`openat()` vs `open()`**  
- **`openat()`** adds a `fd` (file descriptor) parameter:  
  1. If `path` is **absolute** (e.g., `/home/file.txt`), `fd` is ignored.  
  2. If `path` is **relative** (e.g., `docs/file.txt`), `fd` is the directory to start from.  
  3. If `fd = AT_FDCWD`, use the **current directory**.  

**Why use `openat()`?**  
- Lets threads work in **different folders** safely.  
- Avoids security bugs (TOCTTOU errors).  


## **Errors to Know**  
- If the filename is **too long**:  
  - Some systems **truncate** it (cut off extra letters).  
  - Others return `ENAMETOOLONG` error.  


## **Example Code**  
```c
#include <fcntl.h>

int main() {
  int fd = open("test.txt", O_RDWR | O_CREAT, 0644); 
  // Opens (or creates) "test.txt" for read/write
  // 0644 = permissions: user=read/write, others=read-only
}
```

## Why This Matters

- `open()` is the basic way to access files.
- `openat()` adds flexibility for security and multi-threaded apps.
