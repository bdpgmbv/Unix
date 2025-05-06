Simple Explanation of UNIX File I/O
1. Basic File Operations
UNIX uses five main functions for files:
•	open: Start using a file.
•	read: Get data from a file.
•	write: Save data to a file.
•	lseek: Move to a specific spot in a file (like skipping pages in a book).
•	close: Stop using a file.
These are called "unbuffered" because every action (read/write) talks directly to the system. No waiting or saving up data.
 
2. File Descriptors
•	When you open a file, the system gives you a number (like a ticket) called a file descriptor.
•	Example:
o	0 = Standard Input (where you type, like a keyboard).
o	1 = Standard Output (where text appears, like the screen).
o	2 = Standard Error (for error messages).
•	Use names like STDIN_FILENO instead of numbers to make code clearer.
•	Modern systems let you open thousands of files at once.
 
3. Opening Files with open and openat
open Function:
•	Used to open or create a file.
•	Example: open("file.txt", O_RDONLY) opens "file.txt" for reading.
Flags (Options):
•	O_RDONLY: Open for reading.
•	O_WRONLY: Open for writing.
•	O_CREAT: Create the file if it doesn’t exist.
•	O_APPEND: Add new text to the end (like writing in a diary).
•	O_TRUNC: Erase the file’s content when opened.
openat Function:
•	Works like open but lets you specify a directory (helpful for organizing files or security).
•	Example: openat(fd, "file.txt", O_RDWR) opens "file.txt" in the directory linked to fd.
 
4. Filename Length
•	Old systems cut filenames if too long (e.g., "my_long_file.txt" → "my_long_fi").
•	New systems give an error if the name is too long.
•	Most systems now allow filenames up to 255 letters.
 
Key Tips:
•	Always close files when done to free up space.
•	Use O_CREAT with O_EXCL to avoid accidentally overwriting a file.
•	openat is useful for programs that work with many folders or threads.

![image](https://github.com/user-attachments/assets/8ebae440-7d20-4735-89da-1fb78cea7edb)
