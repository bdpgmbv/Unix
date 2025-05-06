# 1.1 Introduction
- **Operating Systems (OS)** help programs run by providing services like:
  - Starting a new program.
  - Opening/reading files.
  - Managing memory.
  - Checking the time.
- This book focuses on **UNIX OS** and its services.
- Explaining UNIX in order (without skipping ahead) is hard. So, this chapter gives a quick overview.
- It introduces key ideas and examples for programmers new to UNIX.

---

# 1.2 UNIX Architecture
- **What is an OS?**  
  Strictly, it is software that:
  - Controls computer hardware (like CPU, memory).
  - Lets programs run.  
  This core part is called the **kernel**.

- **System Calls**:  
  - Basic functions that programs use to talk to the kernel (e.g., open a file).
  - Libraries (pre-written code for common tasks) are built on top of system calls.
  - Programs can use both system calls and libraries.

- **Shell**:  
  A special program that lets users run other programs (like typing commands in a terminal).

- **Broad Definition of OS**:  
  Includes the **kernel** + other tools like:
  - System utilities (tools for managing the computer).
  - Applications (programs you use).
  - Shells and libraries.

- **Example**:  
  - **Linux** is a kernel.
  - **GNU** is an OS that uses Linux as its kernel.
  - Together, they are called **GNU/Linux**, but most people say “Linux”.


# 1.3 Logging In

## **Login Name**
- To use a UNIX system, you **log in** with:
  - **Login name** (username).
  - **Password** (hidden for security).
- The system checks your login name in the **password file** (`/etc/passwd`).  
  Example entry:  
  `sar:x:205:105:Stephen Rago:/home/sar:/bin/ksh`  
  This has **7 parts** separated by colons `:`:
  1. Login name (`sar`).
  2. Password (`x` means password is stored in another file).
  3. User ID (`205` – unique number for the user).
  4. Group ID (`105` – group the user belongs to).
  5. Comment (e.g., user’s real name).
  6. Home folder (`/home/sar` – user’s personal files).
  7. Shell program (`/bin/ksh` – the shell the user uses).

> Modern systems store the encrypted password in a different file (like `/etc/shadow`).  
> (We’ll learn more about password files in Chapter 6.)


## **Shells**
- After logging in, you see a **shell** (a program that takes your commands and runs them).  
  - Example: typing `ls` in the shell lists files in your folder.
  - Shells can be **interactive** (you type commands) or **scripts** (run commands from a file).

### **Types of Shells**
1. **Bourne Shell** (`sh`):  
   - Oldest shell (used since 1970s).  
   - Simple, works on almost all UNIX systems.

2. **C Shell** (`csh`):  
   - Adds features like command history and job control.  
   - Syntax similar to C programming language.

3. **Korn Shell** (`ksh`):  
   - Combines features of Bourne and C shells.  
   - Better for scripting and interactive use.

4. **Bourne-Again Shell** (`bash`):  
   - Default on Linux and Mac OS.  
   - Modern, user-friendly, and powerful.

5. **TENEX C Shell** (`tcsh`):  
   - Improved C shell with auto-complete and other extras.

### **Which Shell Do You Use?**
- The system picks your shell based on the **7th field** in the password file (e.g., `/bin/ksh`).
- Different systems use different default shells:
  - **Linux**: Often `bash` or `dash` (simpler/faster).
  - **Mac OS**: `bash`.
  - **FreeBSD/Solaris**: Mix of shells like `sh`, `csh`, `bash`.


**Note**:  
- This book uses examples from **Bourne-like shells** (Bourne, Korn, `bash`) for consistency.
- Shells help you control the computer, run programs, and automate tasks.

# 1.4 Files and Directories

## **File System**
- UNIX files are organized like a tree:
  - Starts at the **root** directory (`/`).
  - Folders (**directories**) can contain files and other directories.
- **Directory** = A special file that holds info about other files (names, types, sizes, permissions, etc.).
  - Use `stat` or `fstat` to see file details (Chapter 4).


## **Filename**
- **Rules**:
  - Can’t use `/` or `null` (empty) characters.
  - Keep it simple: use letters, numbers, `.`, `-`, or `_`.
- Every directory has:
  - `.` (current directory).
  - `..` (parent directory).  
    Example: In `/home/user`, `..` points to `/home`.


## **Pathname**
- **Pathname** = Directions to find a file (e.g., `/home/user/file.txt`).
  - **Absolute path**: Starts with `/` (e.g., `/home/user`).
  - **Relative path**: Starts from your current folder (e.g., `docs/notes.txt`).


## **Example: Listing Files**
A simple program to list files in a directory:
```c
#include "apue.h"
#include <dirent.h>

int main(int argc, char *argv[]) {
  DIR *dp;
  struct dirent *dirp;
  
  if (argc != 2)
    err_quit("Usage: ls directory_name");
  
  if ((dp = opendir(argv[1])) == NULL)
    err_sys("can’t open %s", argv[1]);
  
  while ((dirp = readdir(dp)) != NULL)
    printf("%s\n", dirp->d_name);
  
  closedir(dp);
  exit(0);
}
```

## How it works

- Takes a directory name as input.
- Opens the directory and reads each file’s name.
- Prints all filenames (unsorted).

## Errors

- **Permission Denied**  
  If you try to open a restricted folder (e.g., `/etc/ssl/private`):  
  `can’t open /etc/ssl/private: Permission denied.`

- **Not a Directory**  
  If you try to open a file (not a directory):  
  `can’t open /dev/tty: Not a directory.`

## Working Directory

- **Working Directory** = The folder you’re “in” right now.
- Use `chdir` to change your working directory.
- Example: `doc/memo/joe` looks for `joe` in `memo`, inside `doc` (from your current folder).

## Home Directory

- **Home Directory** = Your personal folder when you log in (e.g., `/home/sar`).
- Set in the password file (see Section 1.3).
- Starts as your working directory when you log in.

## Key Notes

- File names are **case-sensitive** (e.g., `File.txt` ≠ `file.txt`).
- Use `man 1 ls` to read the manual for the `ls` command.
- Older UNIX systems had shorter filename limits (14 characters), but modern ones allow **255+**.

# 1.5 Input and Output

## **File Descriptors**
- **What are they?**  
  Small numbers (like 0, 1, 2) used by the system to track open files.
  - Example: When you open a file, the system gives you a number (file descriptor) to use for reading/writing.


## **Standard Input, Output, Error**
- Every program gets 3 default "channels":
  1. **Standard Input (0)**: Where the program reads data (usually keyboard).
  2. **Standard Output (1)**: Where the program writes results (usually screen).
  3. **Standard Error (2)**: Where error messages go (usually screen).

- **Example**:  
  - `ls` uses all 3 channels (shows files on screen).
  - `ls > file.list` redirects output to a file (`file.list`).


## **Unbuffered I/O**
- **Basic functions**: `open`, `read`, `write`, `close`.
- Works directly with file descriptors (no automatic buffering).

### **Example: Copy a File**
```c
#include "apue.h"
#define BUFFSIZE 4096

int main(void) {
  int n;
  char buf[BUFFSIZE];
  
  // Read from stdin (0), write to stdout (1)
  while ((n = read(STDIN_FILENO, buf, BUFFSIZE)) > 0) {
    if (write(STDOUT_FILENO, buf, n) != n)
      err_sys("write error");
  }
  
  if (n < 0)
    err_sys("read error");
  
  exit(0);
}
```

## How it works

- Reads data in chunks (`BUFFSIZE` bytes at a time).
- Writes data to output.
- Stops when `read` returns `0` (end of file) or `-1` (error).

## Run it

```bash
./a.out < infile > outfile
```

## Copies infile to outfile

## Standard I/O (Buffered)

- Easier functions: `printf`, `fgets`, `getc`, etc.
- Automatically handles buffering (faster for small reads/writes).

### Example

Copy a file with buffering.

```c
#include "apue.h"

int main(void) {
  int c;
  
  // Read one character at a time
  while ((c = getc(stdin)) != EOF) {
    if (putc(c, stdout) == EOF)
      err_sys("output error");
  }
  
  if (ferror(stdin))
    err_sys("input error");
  
  exit(0);
}
```

## How it works

- `getc` reads one character from input.
- `putc` writes it to output.
- Stops at `EOF` (end-of-file, like pressing `Ctrl+D`).

## Key Notes

- **Unbuffered I/O** is low-level (you control everything).
- **Standard I/O** is simpler (handles buffering for you).
- Use `<unistd.h>` for unbuffered functions, `<stdio.h>` for buffered.


