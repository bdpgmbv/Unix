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


# File Operations

## **3.4 creat Function**  
- **What it does**: Creates a new file.  
- **Syntax**:  
  ```c
  int creat("filename", permissions);
  ```

  - Returns a **file descriptor** (number) for writing.

**Note:**  
`creat` is the same as `open` with specific flags:

```c
open(path, O_WRONLY | O_CREAT | O_TRUNC, mode);
```

## Problem

- `creat` only opens a file for **writing**.
- To read after creating, you must **close and reopen** the file.
- **Better option:** Use `open()` with the `O_RDWR` flag for **read/write** access.

---

## 3.5 `close` Function

**What it does:**  
Closes an open file.

**Syntax:**
```c
close(file_descriptor);
```

## Key Points

- Always **close files** when done to free resources.
- If a program **crashes or exits**, the OS automatically closes all open files.

---

## 3.6 `lseek` Function

**What it does:**  
Moves the "cursor" (file offset) in a file to read/write at specific positions.

**Syntax:**
```c
off_t lseek(fd, offset, whence);  
```

### `whence` can be:

- `SEEK_SET`: Start from the beginning of the file.
- `SEEK_CUR`: Start from the current position.
- `SEEK_END`: Start from the end of the file.

---

### Examples:

**Get current position:**
```c
lseek(fd, 0, SEEK_CUR);
```
**Jump to byte 100:**
```c
lseek(fd, 100, SEEK_SET);  
```
**Jump 50 bytes backward:**
```c
lseek(fd, -50, SEEK_CUR);
```

## File Holes

- If you `lseek` past the end of a file and write, it creates a **"hole"** (unused space that takes no disk storage).

### Example:
```c
write(fd, "start", 5);   // Write "start" at position 0  
lseek(fd, 1000, SEEK_SET); // Jump to position 1000  
write(fd, "end", 3);     // Write "end" at position 1000
```

- The file size is 1003 bytes, but the middle 995 bytes are empty (saves disk space!).

---

## File Offset Sizes

- `off_t` is the data type for file offsets.

### System Differences:
- **32-bit systems:** Max file size = 2 GB (`2^31 - 1` bytes).
- **64-bit systems:** Max file size = Much larger (depends on OS/filesystem).

### Enabling Large Files:
- Use `#define _FILE_OFFSET_BITS 64` in your code to enable large file support (if supported by system and compiler).

---

## Why This Matters

- `creat`: Rarely used now; `open()` is more flexible and preferred.
- `close`: Prevents **resource leaks** (open file descriptors are limited).
- `lseek`: Lets you **read/write anywhere** in a file — useful for **databases**, **logs**, and **binary formats**.
- **Holes**: Efficient use of storage when writing sparse data.

