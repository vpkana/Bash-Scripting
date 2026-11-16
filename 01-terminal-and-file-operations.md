
# Part 01: Terminal and File Operations

## 1. Introduction to the Terminal

A terminal is a text-based interface that allows us to interact with the operating system by executing commands.

Unlike a graphical interface, where we interact using buttons and menus, a terminal allows us to perform operations by entering commands.

### Important terminology

| Term | Description |
|---|---|
| Terminal | Interface used to interact with a shell |
| Shell | Program that interprets commands |
| Bash | Bourne Again SHell, a popular Unix shell |
| Command | Instruction given to the shell |
| Directory | Another name for a folder |
| Path | Location of a file or directory |

### Terminal vs Shell

The terminal is the interface, whereas the shell is the program running inside it.

For example:

```bash
echo "Hello World"
```

Here:
- Terminal: Where we enter the command.
- Bash: Interprets the command.
- `echo`: Command being executed.

---

## 2. Understanding the Current Directory

### `pwd`

Print Working Directory displays the current directory.

```bash
pwd
```

Example output:

```text
/home/user/projects
```

### `cd`

Change Directory allows us to navigate between directories.

```bash
cd /home/user
```

Useful variations:

```bash
cd          # Navigate to the home directory
cd ~        # Navigate to the home directory
cd ..       # Move one directory backward
cd ../..    # Move two directories backward
cd -        # Return to the previous directory
cd /        # Navigate to the root directory
```

### Absolute vs Relative Paths

**Absolute path:** Complete path starting from the root directory.

```bash
cd /home/user/projects/bash
```

**Relative path:** Path specified relative to the current directory.

```bash
cd projects/bash
```

Special path symbols:

| Symbol | Meaning |
|---|---|
| `/` | Root directory |
| `.` | Current directory |
| `..` | Parent directory |
| `~` | Home directory |
| `-` | Previous working directory |

---

## 3. Listing Files and Directories

### `ls`

Lists files and directories.

```bash
ls
```

Useful options:

```bash
ls -l       # Detailed listing
ls -a       # Include hidden files
ls -la      # Detailed listing including hidden files
ls -h       # Human-readable sizes (usually combined with -l)
ls -lh      # Detailed listing with readable sizes
ls -R       # Recursively list directories
ls -lt      # Sort by modification time
ls -ltr     # Reverse modification-time order
```

Example:

```bash
ls -lah
```

Possible output:

```text
drwxr-xr-x  2 user user 4.0K Nov 16 10:00 .
-rw-r--r--  1 user user  120 Nov 16 10:00 notes.md
-rwxr-xr-x  1 user user  250 Nov 16 10:00 script.sh
```

### Understanding the output of `ls -l`

| Field | Meaning |
|---|---|
| `drwxr-xr-x` | Permissions and file type |
| `2` | Hard link count |
| `user` | Owner |
| `user` | Group |
| `4.0K` | Size |
| `Nov 16` | Modification date |
| `notes.md` | Filename |

### Hidden files

In Linux and Unix-like systems, filenames beginning with `.` are treated as hidden by convention.

Examples:

```text
.bashrc
.profile
.gitignore
.env
```

List hidden files:

```bash
ls -a
```

Important: Hidden does not mean protected or secure. These files can still be accessed directly.

---

## 4. Creating Files and Directories

### `touch`

Creates an empty file if it does not exist.

```bash
touch notes.txt
```

If the file already exists, its access and modification timestamps are updated (subject to filesystem behaviour and permissions).

Create multiple files:

```bash
touch file1.txt file2.txt file3.txt
```

### `mkdir`

Creates a directory.

```bash
mkdir projects
```

Create nested directories:

```bash
mkdir -p projects/bash/notes
```

The `-p` option creates missing parent directories without reporting an error if the target directory already exists.

---

## 5. Copying, Moving and Renaming

### `cp`

Copies files.

```bash
cp source.txt destination.txt
```

Copy into a directory:

```bash
cp notes.txt backup/
```

Copy directories recursively:

```bash
cp -r source_dir destination_dir
```

Useful options:

```bash
cp -i file.txt backup.txt   # Prompt before overwriting
cp -v file.txt backup.txt   # Show operation
cp -n file.txt backup.txt   # Do not overwrite existing destination
```

### `mv`

Moves or renames files.

Rename:

```bash
mv old.txt new.txt
```

Move:

```bash
mv notes.txt ~/Documents/
```

Move multiple files:

```bash
mv file1.txt file2.txt backup/
```

Important: `mv` generally renames a file within the same filesystem. Moving between filesystems may involve copying and deleting.

### `rm`

Removes files.

```bash
rm notes.txt
```

Remove multiple files:

```bash
rm file1.txt file2.txt
```

Remove directories:

```bash
rm -r directory/
```

Force removal:

```bash
rm -f file.txt
```

Force recursive removal:

```bash
rm -rf directory/
```

**WARNING:** `rm -rf` is destructive. It does not normally move files to a recycle bin.

Never execute commands involving recursive deletion without verifying the target path.

Safer approach:

```bash
pwd
ls -la ./target-directory
rm -ri ./target-directory
```

The `-i` option asks for confirmation.

---

## 6. Viewing File Contents

### `cat`

Displays file contents.

```bash
cat notes.txt
```

Combine multiple files:

```bash
cat file1.txt file2.txt
```

Write output into another file:

```bash
cat file1.txt file2.txt > combined.txt
```

Append output:

```bash
cat file1.txt >> combined.txt
```

### `less`

Useful for reading large files.

```bash
less large-file.txt
```

Important keyboard shortcuts:

| Key | Action |
|---|---|
| Space | Next page |
| b | Previous page |
| Enter | Next line |
| g | Beginning |
| G | End |
| /word | Search forward |
| ?word | Search backward |
| q | Quit |

### `head`

Displays the first 10 lines by default.

```bash
head file.txt
```

Display first 5 lines:

```bash
head -n 5 file.txt
```

### `tail`

Displays the last 10 lines by default.

```bash
tail file.txt
```

Display last 5 lines:

```bash
tail -n 5 file.txt
```

Follow a file as it grows:

```bash
tail -f application.log
```

Useful for monitoring application logs.

---

## 7. Searching Inside Files

### `grep`

Searches for matching text.

```bash
grep "error" application.log
```

Useful options:

```bash
grep -i "error" file.txt      # Case-insensitive
grep -n "error" file.txt      # Display line numbers
grep -v "error" file.txt      # Display non-matching lines
grep -c "error" file.txt      # Count matching lines
grep -r "error" directory/   # Recursive search
grep -w "error" file.txt      # Match whole word
```

Combine options:

```bash
grep -in "error" application.log
```

### Searching for files using `find`

```bash
find . -name "notes.txt"
```

Find all Markdown files:

```bash
find . -name "*.md"
```

Find directories:

```bash
find . -type d
```

Find regular files:

```bash
find . -type f
```

Case-insensitive filename search:

```bash
find . -iname "*.TXT"
```

---

## 8. Important Redirection Operators

| Operator | Meaning |
|---|---|
| `>` | Redirect stdout, overwrite |
| `>>` | Redirect stdout, append |
| `<` | Redirect stdin |
| `2>` | Redirect stderr |
| `2>>` | Append stderr |
| `&>` | Redirect stdout and stderr (Bash) |
| `2>&1` | Redirect stderr to current stdout destination |

Examples:

```bash
echo "Hello" > output.txt
echo "World" >> output.txt

cat < output.txt

ls invalid-directory 2> errors.txt

command > output.txt 2>&1
```

Important distinction:

```bash
command > output.txt 2>&1
```

and

```bash
command 2>&1 > output.txt
```

are not equivalent because redirections are evaluated from left to right.

---

## 9. Useful Practice Exercises

1. Create a directory called `bash-practice`.
2. Inside it, create three files.
3. Write some text into the first file.
4. Copy the first file into a backup directory.
5. Rename the second file.
6. Search for a particular word across all files.
7. Display the first five and last five lines of a file.
8. Create a nested directory structure using `mkdir -p`.
9. Find every `.md` file inside the current directory.
10. Redirect the output and errors of a command into separate files.

---

## 10. Quick Revision

- `pwd`: Current working directory.
- `cd`: Change directory.
- `ls`: List directory contents.
- `mkdir`: Create directory.
- `touch`: Create empty file or update timestamps.
- `cp`: Copy files.
- `mv`: Move or rename files.
- `rm`: Remove files.
- `cat`: Display and concatenate files.
- `less`: Read large files interactively.
- `head`: Display beginning of a file.
- `tail`: Display end of a file.
- `grep`: Search text.
- `find`: Search filesystem entries.

**Important:** Always understand the target path before using destructive commands.
