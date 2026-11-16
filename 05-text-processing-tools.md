
# Part 05: Text Processing Tools

## 1. Introduction

Unix provides several powerful utilities for processing text.

These tools are especially useful in:
- Log analysis
- Data extraction
- Text transformation
- Automation
- System administration
- Processing command output

The major tools covered here are:

| Tool | Primary purpose |
|---|---|
| `cut` | Extract columns or character positions |
| `tr` | Translate or delete characters |
| `grep` | Search matching lines |
| `sed` | Stream editing |
| `awk` | Field-based text processing |
| `find` | Search filesystem entries |

---

## 2. The `cut` Command

`cut` extracts specific portions of each line.

### Extracting characters

```bash
echo "BASH" | cut -c 1-2
```

Output:

```text
BA
```

Other examples:

```bash
cut -c 1 file.txt
cut -c 1-5 file.txt
cut -c 3- file.txt
cut -c -4 file.txt
```

### Extracting fields

Consider:

```text
Prabhath:MANIT:Bhopal
```

Extract the first field:

```bash
echo "Prabhath:MANIT:Bhopal" | cut -d ':' -f 1
```

Output:

```text
Prabhath
```

Extract multiple fields:

```bash
echo "Prabhath:MANIT:Bhopal" | cut -d ':' -f 1,3
```

Output:

```text
Prabhath:Bhopal
```

### Useful options

| Option | Meaning |
|---|---|
| `-c` | Select characters |
| `-d` | Specify delimiter |
| `-f` | Select fields |
| `-b` | Select bytes |
| `--complement` | Select everything except specified positions/fields |
| `-s` | Suppress lines without delimiters |

### Example with a file

`students.txt`:

```text
101,Prabhath,MANIT
102,Alice,IIT
103,Bob,NIT
```

Extract names:

```bash
cut -d ',' -f 2 students.txt
```

Output:

```text
Prabhath
Alice
Bob
```

### Important limitations

- `cut` is not a full CSV parser.
- It cannot correctly handle arbitrary quoted delimiters.
- `-c` and `-b` are not necessarily equivalent for multibyte text.
- Delimiter matching is based on characters, not arbitrary multi-character strings.

---

## 3. The `tr` Command

`tr` translates or deletes characters from standard input.

It does not normally accept filenames as operands. Use input redirection when processing a file.

### Convert lowercase to uppercase

```bash
echo "hello world" | tr 'a-z' 'A-Z'
```

Output:

```text
HELLO WORLD
```

### Convert uppercase to lowercase

```bash
echo "HELLO" | tr 'A-Z' 'a-z'
```

### Delete characters

```bash
echo "hello123" | tr -d '0-9'
```

Output:

```text
hello
```

### Squeeze repeated characters

```bash
echo "hellooo     world" | tr -s ' o'
```

`-s` squeezes consecutive repeated characters from the specified set.

### Replace spaces with underscores

```bash
echo "bash scripting" | tr ' ' '_'
```

Output:

```text
bash_scripting
```

### Character classes

```bash
echo "Hello 123!" | tr '[:lower:]' '[:upper:]'
```

Common character classes:

| Class | Meaning |
|---|---|
| `[:lower:]` | Lowercase |
| `[:upper:]` | Uppercase |
| `[:digit:]` | Digits |
| `[:alpha:]` | Alphabetic characters |
| `[:alnum:]` | Alphanumeric |
| `[:space:]` | Whitespace |
| `[:punct:]` | Punctuation |

### Removing newline characters

```bash
tr -d '\n' < file.txt
```

### Important limitations

`tr` works at the character level, not at the word or field level.

For complex transformations, use `sed` or `awk`.

---

## 4. The `grep` Command

`grep` searches for lines matching a pattern.

### Basic search

```bash
grep "error" application.log
```

### Case-insensitive search

```bash
grep -i "error" application.log
```

### Display line numbers

```bash
grep -n "error" application.log
```

### Count matching lines

```bash
grep -c "error" application.log
```

### Invert matching

```bash
grep -v "error" application.log
```

Displays lines that do not match.

### Whole-word matching

```bash
grep -w "cat" file.txt
```

Avoids matching words such as `category`.

### Recursive search

```bash
grep -r "TODO" .
```

### Search multiple patterns

```bash
grep -E "error|warning" application.log
```

### Extended regular expressions

`grep -E` enables extended regular expression syntax.

Examples:

```bash
grep -E 'cat|dog' file.txt
grep -E '[0-9]+' file.txt
grep -E '^ERROR' file.txt
grep -E 'done$' file.txt
```

### Fixed-string search

```bash
grep -F 'a.b' file.txt
```

Here, `.` is treated literally rather than as a regular expression wildcard.

### Useful options

| Option | Meaning |
|---|---|
| `-i` | Ignore case |
| `-n` | Line numbers |
| `-v` | Invert match |
| `-c` | Count matching lines |
| `-r` | Recursive search |
| `-w` | Whole-word match |
| `-E` | Extended regular expressions |
| `-F` | Fixed-string matching |
| `-l` | Print matching filenames |
| `-o` | Print only matched portions |
| `-q` | Quiet mode |

### Important: grep exit status

| Status | Meaning |
|---|---|
| 0 | At least one match |
| 1 | No match |
| 2 | Error |

Example:

```bash
if grep -q "ERROR" application.log; then
    echo "Errors found"
else
    echo "No errors found"
fi
```

A status of `1` does not mean grep encountered an execution error.

---

## 5. The `sed` Command

`sed` is a stream editor used to transform text.

### Basic syntax

```bash
sed 'command' file.txt
```

### Substitution

```bash
sed 's/old/new/' file.txt
```

This replaces the first matching occurrence on each line.

### Replace all occurrences on each line

```bash
sed 's/old/new/g' file.txt
```

### Case-insensitive replacement

```bash
sed 's/error/warning/gi' file.txt
```

### Delete lines

```bash
sed '3d' file.txt
```

Deletes line 3.

```bash
sed '1,5d' file.txt
```

Deletes lines 1 through 5.

### Print specific lines

```bash
sed -n '3p' file.txt
```

Prints only line 3.

```bash
sed -n '1,5p' file.txt
```

Prints lines 1 through 5.

### Match using patterns

```bash
sed '/ERROR/d' file.txt
```

Deletes lines containing `ERROR`.

### In-place modification

```bash
sed -i 's/old/new/g' file.txt
```

This modifies the original file.

Safer backup:

```bash
sed -i.bak 's/old/new/g' file.txt
```

Creates a backup with a `.bak` extension on GNU sed.

Note: `sed -i` portability and syntax differ between GNU sed and BSD/macOS sed.

---

## 6. The `awk` Command

`awk` is a programming language designed for processing structured text.

It is particularly useful when data is arranged in rows and columns.

### Basic syntax

```bash
awk 'pattern { action }' file.txt
```

### Default field separator

Consider:

```text
Prabhath 23 MANIT
Alice 22 IIT
Bob 24 NIT
```

```bash
awk '{print $1}' students.txt
```

Output:

```text
Prabhath
Alice
Bob
```

### Important predefined variables

| Variable | Meaning |
|---|---|
| `$0` | Entire current line |
| `$1` | First field |
| `$2` | Second field |
| `$NF` | Last field |
| `NF` | Number of fields |
| `NR` | Current record number |
| `FNR` | Record number in current file |
| `FS` | Input field separator |
| `OFS` | Output field separator |
| `RS` | Input record separator |
| `ORS` | Output record separator |

### Printing multiple fields

```bash
awk '{print $1, $3}' students.txt
```

### Custom delimiter

For comma-separated data:

```bash
awk -F ',' '{print $2}' students.csv
```

### Changing output separator

```bash
awk 'BEGIN { OFS=":" } {print $1, $2}' students.txt
```

### Filtering records

```bash
awk '$2 > 20 {print $1, $2}' students.txt
```

### Calculating totals

Consider:

```text
Apple 10
Banana 20
Orange 30
```

```bash
awk '{sum += $2} END {print sum}' fruits.txt
```

Output:

```text
60
```

### BEGIN and END

`BEGIN` executes before input processing.

`END` executes after all input has been processed.

```bash
awk '
BEGIN {
    print "Student Report"
}
{
    print $1
}
END {
    print "Report completed"
}
' students.txt
```

### Regular expressions

```bash
awk '/ERROR/ {print $0}' application.log
```

### Important limitations

AWK's default field splitting is not equivalent to a full CSV parser. Quoted commas and embedded newlines require a proper CSV-aware tool.

---

## 7. The `find` Command

`find` searches for files and directories.

### Basic syntax

```bash
find starting_path expression
```

### Find by filename

```bash
find . -name "notes.md"
```

### Case-insensitive filename search

```bash
find . -iname "*.txt"
```

### Find only files

```bash
find . -type f
```

### Find only directories

```bash
find . -type d
```

### Find files by extension

```bash
find . -name "*.sh"
```

### Find files by size

```bash
find . -size +10M
```

Find files larger than 10 MB.

```bash
find . -size -1k
```

Find files smaller than 1 KiB.

### Find recently modified files

```bash
find . -mtime -1
```

Find entries modified within the last 24 hours.

### Find by permissions

```bash
find . -perm 644
```

Permission matching behaviour depends on the form of the `-perm` expression. Exact matching is used here.

### Execute commands on results

```bash
find . -type f -name "*.tmp" -exec rm -i {} \;
```

Here:
- `{}` represents the current result.
- `\;` terminates the command.

Another option:

```bash
find . -type f -name "*.txt" -exec wc -l {} +
```

The `+` form batches multiple paths into fewer command invocations.

### Safe handling of filenames

Filenames can contain spaces, tabs and even newline characters.

Use null-delimited processing:

```bash
find . -type f -print0
```

Pass safely to another command:

```bash
find . -type f -print0 | xargs -0 wc -l
```

This avoids breaking filenames containing whitespace.

---

## 8. Combining Text Processing Tools

The real strength of these tools comes from combining them using pipes.

### Example 1: Find all error messages

```bash
grep -i "error" application.log
```

### Example 2: Count errors

```bash
grep -ic "error" application.log
```

### Example 3: Extract usernames

```bash
cut -d ':' -f 1 /etc/passwd
```

### Example 4: Convert text to uppercase

```bash
cat file.txt | tr '[:lower:]' '[:upper:]'
```

The `cat` here is unnecessary. Prefer:

```bash
tr '[:lower:]' '[:upper:]' < file.txt
```

### Example 5: Find the most common log messages

```bash
sort application.log | uniq -c | sort -nr
```

### Example 6: Find all Bash scripts and count them

```bash
find . -type f -name "*.sh" | wc -l
```

---

## 9. Quick Revision

| Tool | Primary purpose |
|---|---|
| `cut` | Extract fields or characters |
| `tr` | Translate or delete characters |
| `grep` | Search matching lines |
| `sed` | Transform text |
| `awk` | Process structured text |
| `find` | Search filesystem |

**Important:** Understand the difference between text processing, field extraction, regular expressions and filesystem searching. These tools complement each other rather than replacing one another.
