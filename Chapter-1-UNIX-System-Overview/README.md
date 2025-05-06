# UNIX System Introduction and Architecture

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

# 1.3 Logging In & Shells

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


# 1.5 Input and Output

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


## Run Examples:

- `./a.out > data`: Saves typed input to `data`.
- `./a.out < infile > outfile`: Copies `infile` to `outfile`.


## Standard I/O

**What is it?**  
Easier functions with automatic buffering (the computer manages data chunks for you).

**Key functions:**  
`printf()`, `fgets()`, `getc()`, `putc()`


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


## Buffered vs. Unbuffered I/O

| Unbuffered I/O                 | Standard I/O                                |
|-------------------------------|---------------------------------------------|
| Manual control (you pick buffer size). | Automatic buffering (computer manages it). |
| Uses `read()` / `write()`.    | Uses `getc()` / `putc()`, `printf()`, etc.  |
| Faster for big files.         | Easier for text (reads lines, handles errors). |


## Why This Matters

- File descriptors help programs work with files.
- Redirection lets you control where input/output goes (files, screens, etc.).
- Standard I/O is simpler for most tasks (like reading text).


# 1.6 Programs and Processes

## **Program**  
- A **program** is a file saved on your computer (like `a.out` after compiling code).  
- Example: A recipe saved in a book.  


## **Process**  
- A **process** is when the program is **running** (like cooking using the recipe).  
- Each process has a unique **Process ID (PID)** (e.g., `851`, `854`).  

### **Example Code**  
This program prints its own PID:  
```c
#include "apue.h"

int main(void) {
  printf("hello world from process ID %ld\n", (long)getpid());
  exit(0);
}

$ ./a.out  
hello world from process ID 851  
$ ./a.out  
hello world from process ID 854  
```

## `getpid()`

- `getpid()` gets the PID (Process ID).
- Every time you run the program, it gets a new PID.


## Process Control

**3 key functions control processes:**

- `fork()`: Creates a copy of the current process (parent → child).
  - Parent gets the child’s PID.
  - Child gets `0`.

- `exec()`: Replaces the current process with a new program (e.g., runs `ls`, `date`).

- `waitpid()`: Makes the parent wait for the child to finish.


## Example: Simple Shell Program

This code acts like a basic shell (reads commands and runs them):

```C
#include "apue.h"
#include <sys/wait.h>

int main(void) {
  char buf[MAXLINE];
  pid_t pid;
  int status;

  printf("%% "); // Show prompt (e.g., % )
  while (fgets(buf, MAXLINE, stdin) != NULL) {
    buf[strlen(buf) - 1] = 0; // Remove newline
    if ((pid = fork()) < 0) {
      err_sys("fork error");
    } else if (pid == 0) { // Child process
      execlp(buf, buf, (char *)0); // Run command (e.g., "date")
      err_ret("error: %s", buf);
      exit(127);
    }
    // Parent waits for child
    if (waitpid(pid, &status, 0) < 0)
      err_sys("waitpid error");
    printf("%% ");
  }
  exit(0);
}
```

## How it works:

- Reads a command (e.g., `date`).
- `fork()` creates a child process.
- Child uses `execlp()` to run the command.
- Parent waits (`waitpid()`) for the child to finish.

**Limitation:** Can’t pass arguments (e.g., `ls -l` won’t work).


## Threads

A thread is like a worker inside a process.  
**1 process = 1+ threads.** Threads share memory (e.g., variables, files).

- **Thread ID (TID):** Unique within a process (not across the whole system).


### Key Points:

- Threads are faster to create than processes.
- Threads need synchronization (to avoid messing up shared data).
- **Example:** Multiple threads in a web browser can load images and text at the same time.


## Why This Matters

- **Processes** let you run multiple programs at once (e.g., browser + music player).
- **Threads** make programs faster by doing many tasks at once (e.g., downloading files while scrolling).


# 1.7 Error Handling
## **What Happens When Errors Occur?**  
- When a UNIX function fails (e.g., opening a file), it returns a special value (like `-1` or `NULL`).  
- The computer sets a global variable **`errno`** to a number explaining why the error happened.  
  - Example: `EACCES` means "Permission denied".  


## **Understanding `errno`**  
- **`errno`** is like an error code book:  
  - Defined in `<errno.h>`.  
  - Every error has a name starting with `E` (e.g., `ENOENT` = "File not found").  
- **Rules for `errno`**:  
  1. Only check `errno` **after** a function fails (it keeps old values otherwise).  
  2. `errno` is never `0` (so `0` means no error).  


## **Error Messages**  
Two ways to get readable error messages:  

1. **`strerror()`**:  
```c
char *strerror(int errno); // Turns error code to text
```
**Example:**

```c
printf("Error: %s\n", strerror(EACCES)); // Prints "Permission denied"
```

2. **`perror()`**: 
```c
void perror(const char *msg); // Prints message + error
```

**Example:**
```c
perror("my_program"); // Prints "my_program: No such file"
```

**Example code:**

```c
#include "apue.h"
#include <errno.h>

int main() {
  fprintf(stderr, "EACCES: %s\n", strerror(EACCES)); // Show EACCES message
  errno = ENOENT; // Set error to "File not found"
  perror("my_program"); // Prints "my_program: No such file"
  exit(0);
}
```

**Output:**

```
EACCES: Permission denied
my_program: No such file or directory
```

## Types of Errors

### Fatal Errors:
- **No fix.**  
  - **Example:** Out of memory.
- **What to do:** Print error → Exit program.

### Non-Fatal Errors:
- **Can be fixed by retrying.**  
  - **Examples:**
    - `EAGAIN`: Resource busy.
    - `ENOSPC`: Disk full.
- **What to do:** Wait → Try again later.


## Why This Matters

- `errno` tells you why something failed.
- `perror()` / `strerror()` turn codes into human-readable messages.
- Handle errors to make programs **stable** (don’t crash easily).


## Tips

- Always check function return values for errors.
- Use `perror()` to include your program’s name in error messages (helps debugging).
- For threads, `errno` is **thread-safe** (each thread has its own copy).


# 1.8 User Identification

## **User ID (UID)**  
- **What is it?**  
  Every user has a **unique number** (like a secret code) called a **User ID**.  
  - Assigned by the system admin.  
  - **Root user** (superuser) has UID **0** → has full control over the system.  

- **Why it matters?**  
  The system uses UID to check if you can do things (like open files).  


## **Group ID (GID)**  
- **What is it?**  
  A **Group ID** is a number for a "team" of users.  
  - Lets team members share files.  
  - Example: All developers in a group can edit the same code.  

- **How it works?**  
  - Stored in `/etc/passwd` (user info) and `/etc/group` (group info).  
  - The system uses numbers (not names) because they’re faster to process.  

### **Example Code**  
This program prints your UID and GID:  
```c
#include "apue.h"

int main() {
  printf("uid = %d, gid = %d\n", getuid(), getgid());
  exit(0);
}

```

## Output:

`uid = 205, gid = 105`

- `getuid()` → Your User ID.
- `getgid()` → Your Group ID.


## Supplementary Groups

## What are they?

- You can belong to extra groups (like being in multiple clubs).
- Started with BSD systems (up to 16 groups).
- Modern systems allow 8+ groups (often 16).

## Why use them?

- Lets you access files from different teams without changing your main group.

## Key Files

- `/etc/passwd`:  
  Stores user info (name, UID, GID, home folder, shell).

- `/etc/group`:  
  Stores group info (group name, GID, members).

## Why This Matters

- `UID`/`GID` control file access (who can read/write).
- `root` (UID 0) can do anything (be careful!).
- Groups make sharing easier (no need to set permissions for each user).


# 1.9 Signals 

## **What Are Signals?**  
Signals are like "alarms" sent to a program to tell it something happened.  
- Examples:  
  - Dividing by zero → `SIGFPE` (math error signal).  
  - Pressing **Ctrl+C** → `SIGINT` (interrupt signal).  

## **What Can a Program Do with a Signal?**  
1. **Ignore It** (not safe for serious errors like crashes).  
2. **Let Default Action Happen** (e.g., program stops).  
3. **Catch It** (run a special function to handle the signal).  

## **Example: Catching Ctrl+C**  
In the simple shell program (from Section 1.6), pressing **Ctrl+C** would normally exit.  
But we can **catch** the signal to keep running:  

```c
#include "apue.h"
#include <sys/wait.h>

static void sig_int(int); // Signal handler

int main() {
  // Tell the system to call sig_int() when Ctrl+C is pressed
  if (signal(SIGINT, sig_int) == SIG_ERR)
    err_sys("signal error");

  printf("%% "); // Show prompt (%)
  // ... rest of the shell code ...
}

// Function to handle Ctrl+C
void sig_int(int signo) {
  printf("\ninterrupt\n%% "); // Print message, keep running
}
```

## What Changed?

- Added `signal(SIGINT, sig_int)` → Catches Ctrl+C.
- `sig_int()` prints `"interrupt"` and keeps the shell running.

## How Signals Are Generated

- **Hardware Errors:** e.g., invalid math operation.
- **Terminal Keys:** e.g., Ctrl+C, Ctrl+\
- **`kill` Command:** Send signals to other programs (requires permission).

## Why This Matters

- Signals let programs handle unexpected events (like user interruptions).
- **Example:** Saving data before exiting, or ignoring accidental Ctrl+C.

# 1.10 Time Values & 1.11 System Calls
## **1.10 Time Values**  

### **Two Types of Time in UNIX**:  
1. **Calendar Time**:  
   - Counts seconds since **January 1, 1970** (like a computer’s clock).  
   - Used for file timestamps (e.g., "last modified").  
   - Stored in `time_t` (a special number type).  

2. **Process Time (CPU Time)**:  
   - Measures how much **CPU power** a program uses.  
   - Stored in `clock_t` (counts "clock ticks").  

---

### **Three Time Measurements for Programs**:  
| Type              | What It Means                                  |  
|--------------------|-----------------------------------------------|  
| **Clock Time**     | Total time taken (like a stopwatch).          |  
| **User CPU Time**  | Time spent running your code.                 |  
| **System CPU Time**| Time spent on OS tasks (e.g., reading files). |  

**Example**:  
```bash
$ time -p ls  
real 0.05s  # Total time  
user 0.01s  # Your code time  
sys  0.03s  # System time  
```

# 1.11 System Calls vs. Library Functions

### System Calls:
**What:** Direct requests to the OS kernel (e.g., open a file).

- **Examples:** `read()`, `write()`, `fork()`
- **Key Point:** You can’t change them – they’re part of the OS.

### Library Functions:
**What:** Helper tools in C libraries (e.g., `printf()`).

- **Examples:** `malloc()`, `printf()`, `strcpy()`
- **Key Point:** You can replace these (e.g., write your own `malloc()`).

---

## How They Work Together

- **Example 1:** `malloc()` (library) uses `sbrk()` (system call) to get memory.
- **Example 2:** `printf()` (library) uses `write()` (system call) to print text.

---

## Comparison

| System Calls              | Library Functions                 |
|---------------------------|-----------------------------------|
| Minimal, direct to OS     | Add extra features                |
| Hard to replace           | Easy to replace/customize         |

---

## Why This Matters

- **Time values** help track performance (e.g., why a program is slow).
- **System Calls** are the backbone – they let programs interact with the OS safely.
- **Library Functions** make coding easier (you don’t have to write everything from scratch).
