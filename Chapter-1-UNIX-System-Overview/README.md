# Simple Explanation of UNIX System Introduction and Architecture

# 1.1 Introduction
- **What is an Operating System (OS)?**  
  An OS gives tools (services) to programs. Examples:  
  - Run a new program  
  - Open/read files  
  - Use memory  
  - Check time  

- **This Book’s Focus**  
  Explains how **UNIX OS** (different versions) gives these services.  

- **Why This Chapter?**  
  UNIX is hard to explain step-by-step without skipping ahead. This chapter gives a quick summary of UNIX for programmers. Details come later in the book.


# 1.2 UNIX Architecture  
### Key Parts of UNIX:  
1. **Kernel**  
   - The "core" of the OS.  
   - Controls computer hardware (like memory, CPU).  
   - Makes sure programs run smoothly.  

2. **System Calls**  
   - Acts as a bridge between programs and the kernel.  
   - Programs ask the kernel for help via system calls (e.g., "open a file").  

3. **Libraries**  
   - Pre-made code for common tasks (e.g., math functions).  
   - Built on top of system calls to make coding easier.  

4. **Shell**  
   - A special program that lets users run other programs.  
   - Example: typing commands in a terminal.  

5. **Other Software**  
   - Tools (like file managers), apps, and utilities.  

### What is an "Operating System"?  
- **Strict Sense**: Just the **kernel**.  
- **Broad Sense**: Kernel + all tools/apps (this gives the OS its "personality").  

### Example: Linux vs. GNU/Linux  
- **Linux** is just the kernel.  
- **GNU/Linux** = Linux kernel + GNU tools/apps.  
- Most people call it "Linux" for short, even though it’s technically the whole system.  

# 1.3 Logging In & Shells Explained Simply

## Logging In
- **Login Name & Password**:  
  When you log into a UNIX system, you type your **login name** and **password**.  
  The system checks these in a file (`/etc/passwd`).  

- **Password File Example**:  
  Your entry in this file has 7 parts, separated by colons (`:`):  
  `sar:x:205:105:Stephen Rago:/home/sar:/bin/ksh`  
  1. Login name (`sar`)  
  2. Hidden password (`x` – stored in another file now)  
  3. User number ID (`205`)  
  4. Group number ID (`105`)  
  5. Comment/Full name (`Stephen Rago`)  
  6. Home folder (`/home/sar`)  
  7. Default shell (`/bin/ksh` – the program you use to type commands).  

## Shells
A **shell** is like a tool that lets you talk to the computer by typing commands.  
The system picks your shell based on Part 7 of the password file entry.

### Types of Shells:
1. **Bourne Shell**:  
   - Old but works on almost all UNIX systems.  
   - Made by Steve Bourne.  

2. **C Shell**:  
   - Made by Bill Joy.  
   - Adds cool features like command history.  
   - Inspired by the C programming language.  

3. **Korn Shell**:  
   - Upgraded Bourne shell.  
   - Made by David Korn.  
   - Adds job control and command editing.  

4. **Bourne-Again Shell (Bash)**:  
   - Default in Linux and Mac OS.  
   - Mixes features from Bourne, C, and Korn shells.  

5. **TENEX C Shell (tcsh)**:  
   - Better version of the C shell.  
   - Adds auto-complete for commands.  

### Default Shells in Different Systems:
- **Linux**: Uses **Bash** or **Dash** (simpler/faster).  
- **FreeBSD/Mac OS**: Uses shells based on **Bash**.  
- **Solaris**: Includes all shells above.  

## Why This Matters?
- Shells let you control the computer.  
- Different shells have extra features (like command history).  
- The password file tells the system where your files are and which shell to use.  

# 1.4 Files and Directories

### **File System**
- **What is it?**  
  UNIX files are organized **like a tree** starting from the **root** (`/`).  
  - **Root** (`/`): The top folder. Everything starts here.  
  - **Directory**: A folder that holds files or other folders.  

- **Directory Entries**  
  Each folder has items with:  
  - Filename (e.g., `notes.txt`).  
  - File details (type, size, owner, permissions, etc.).  
  - Use `stat` or `fstat` to get these details.  

**Note**: File details (like size) are *not stored directly in the folder entry* (to avoid issues with hard links).  


### **Filename Rules**
- **Allowed Characters**:  
  - Letters (`a-z`, `A-Z`), numbers (`0-9`), `.`, `-`, `_`.  
  - **Avoid**: `/` (used to separate folders) and special symbols (like `*` or `$`).  

- **Special Names**:  
  - `.` (dot) = Current folder.  
  - `..` (dot-dot) = Parent folder.  
  - In the root folder (`/`), `.` and `..` are the same.  

- **History**:  
  Old UNIX limited filenames to 14 letters. Now, most allow **255 letters**.  


### **Pathname**
- **What is it?**  
  A path tells how to find a file/folder.  
  - **Absolute Path**: Starts with `/` (e.g., `/home/user/file.txt`).  
  - **Relative Path**: Starts from the current folder (e.g., `docs/report.pdf`).  

**Example**:  
- Absolute: `/usr/bin/ls` → Starts from root.  
- Relative: `pics/photo.jpg` → Starts from current folder.  


### **Example: Simple `ls` Program**
Here’s a basic program to list files in a folder (like `ls` command):  
```c
#include "apue.h"
#include <dirent.h>

int main(int argc, char *argv[]) {
    DIR *dp;
    struct dirent *dirp;

    if (argc != 2)
        err_quit("Usage: ls directory_name");

    if ((dp = opendir(argv[1])) == NULL)
        err_sys("Can’t open %s", argv[1]);

    while ((dirp = readdir(dp)) != NULL)
        printf("%s\n", dirp->d_name);

    closedir(dp);
    exit(0);
}
```

## How it works:

**Input:** Takes a folder name (e.g., `./a.out /dev`).

- Opens the folder and reads all filenames.
- Prints each filename (unsorted).

**Sample Output:**

```
$ ./a.out /dev
.
..
cdrom
stderr
stdin
... (more files)
```

## Errors:

- **Permission denied:** Can’t open folder.
- **Not a directory:** Tried to list a file, not a folder.


## Working Directory

**What is it?**  
The folder your program is "in" right now.

- Use `chdir` to change it.
- Relative paths start here.

**Example:**  
If working directory is `/home/user`, then `docs/file.txt` → `/home/user/docs/file.txt`.


## Home Directory

**What is it?**  
Your personal folder when you log in (e.g., `/home/yourname`).

- Set by the system when you login.

**Example:**  
After login, your working directory is your home.

```bash
$ cd ~  # Shortcut to go home.
```


# 1.5 Input and Output Explained Simply

## **File Descriptors**  
- **What are they?**  
  File descriptors are like "file numbers" the computer uses to track open files.  
  - Example: When you open a file, the computer gives you a number (like 0, 1, 2) to work with it.  

## **Standard Input, Output, and Error**  
By default, every program gets 3 "file numbers":  
1. **Standard Input (0)**: Where the program reads data (usually your keyboard).  
2. **Standard Output (1)**: Where the program writes results (usually your screen).  
3. **Standard Error (2)**: Where errors are shown (also your screen).  

- **Redirection Example**:  
  `ls > file.list`  
  - Runs `ls` but saves output to `file.list` instead of the screen.  

---

## **Unbuffered I/O**  
- **What is it?**  
  Basic functions to read/write files directly (no automatic help from the computer).  
  - Key functions: `open()`, `read()`, `write()`, `close()`.  

### **Example Code**  
This program copies input to output (like the `cat` command):  
```c
#include "apue.h"
#define BUFFSIZE 4096

int main(void) {
  int n;
  char buf[BUFFSIZE];
  
  while ((n = read(0, buf, BUFFSIZE)) > 0) { // Read from input (0)
    if (write(1, buf, n) != n) // Write to output (1)
      err_sys("write error");
  }
  if (n < 0) 
    err_sys("read error");
  exit(0);
}
```

## How it works:

- Reads data in chunks (size `BUFFSIZE`).
- Writes the same data to output.
- Stops when there’s no more input or an error.

---

## Run Examples:

- `./a.out > data`: Saves typed input to `data`.
- `./a.out < infile > outfile`: Copies `infile` to `outfile`.

---

## Standard I/O

**What is it?**  
Easier functions with automatic buffering (the computer manages data chunks for you).

**Key functions:**  
`printf()`, `fgets()`, `getc()`, `putc()`

---

## Example Code

This program also copies input to output, but uses simpler functions:

```C
#include "apue.h"

int main(void) {
  int c;
  
  while ((c = getc(stdin)) != EOF) { // Read 1 character
    if (putc(c, stdout) == EOF)      // Write 1 character
      err_sys("output error");
  }
  if (ferror(stdin))
    err_sys("input error");
  exit(0);
}
```

## How it works:

- Reads one character at a time.
- Writes one character at a time.
- Stops at EOF (like pressing Ctrl+D).

---

## Buffered vs. Unbuffered I/O

| Unbuffered I/O                 | Standard I/O                                |
|-------------------------------|---------------------------------------------|
| Manual control (you pick buffer size). | Automatic buffering (computer manages it). |
| Uses `read()` / `write()`.    | Uses `getc()` / `putc()`, `printf()`, etc.  |
| Faster for big files.         | Easier for text (reads lines, handles errors). |

---

## Why This Matters

- File descriptors help programs work with files.
- Redirection lets you control where input/output goes (files, screens, etc.).
- Standard I/O is simpler for most tasks (like reading text).


