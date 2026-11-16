
# Part 04: Case Statements, Arrays and Expansions

## 1. Case Statements

A `case` statement is useful when a variable needs to be compared against multiple possible patterns.

It is often cleaner than writing several `if-elif` conditions.

### Syntax

```bash
case "$variable" in
    pattern1)
        commands
        ;;
    pattern2)
        commands
        ;;
    *)
        default_commands
        ;;
esac
```

### Example: Menu-driven program

```bash
read -r -p "Enter your choice: " choice

case "$choice" in
    1)
        echo "Starting application"
        ;;
    2)
        echo "Opening settings"
        ;;
    3)
        echo "Exiting"
        ;;
    *)
        echo "Invalid choice"
        ;;
esac
```

### Multiple patterns

Use `|` to match multiple patterns.

```bash
read -r -p "Enter an alphabet: " letter

case "$letter" in
    a|e|i|o|u|A|E|I|O|U)
        echo "Vowel"
        ;;
    *)
        echo "Not a vowel"
        ;;
esac
```

### Pattern matching

Bash case patterns support wildcard matching.

```bash
case "$filename" in
    *.txt)
        echo "Text file"
        ;;
    *.sh)
        echo "Shell script"
        ;;
    *.md)
        echo "Markdown file"
        ;;
    *)
        echo "Unknown file type"
        ;;
esac
```

### Important terminators

| Terminator | Behaviour |
|---|---|
| `;;` | Stop processing the case statement |
| `;&` | Execute the next case body without testing its pattern |
| `;;&` | Continue testing subsequent patterns |

Example of `;;&`:

```bash
value="apple"

case "$value" in
    a*)
        echo "Starts with a"
        ;;&
    *e)
        echo "Ends with e"
        ;;&
    *)
        echo "Processed"
        ;;
esac
```

Output:

```text
Starts with a
Ends with e
Processed
```

---

## 2. Indexed Arrays

Arrays store multiple values under one variable name.

Bash supports indexed and associative arrays.

### Creating an indexed array

```bash
languages=("Bash" "Python" "Java" "C++")
```

Access an element:

```bash
echo "${languages[0]}"
```

Output:

```text
Bash
```

Important: Bash indexed arrays start at index `0`.

### Adding elements

```bash
languages+=("JavaScript")
```

### Updating elements

```bash
languages[1]="Go"
```

### Accessing all elements

```bash
echo "${languages[@]}"
```

Recommended when passing all elements as separate arguments:

```bash
printf '<%s>\n' "${languages[@]}"
```

### Array length

```bash
echo "${#languages[@]}"
```

### Accessing array indices

```bash
echo "${!languages[@]}"
```

This prints the indices that actually exist.

### Removing an element

```bash
unset 'languages[1]'
```

The index is removed, but other elements do not automatically shift.

### Sparse arrays

```bash
arr=()

arr[0]="A"
arr[5]="B"

echo "${#arr[@]}"    # 2
echo "${!arr[@]}"    # 0 5
```

The number of existing elements is not necessarily the highest index plus one.

### Iterating through an array

```bash
for language in "${languages[@]}"; do
    echo "$language"
done
```

### Iterating through indices

```bash
for index in "${!languages[@]}"; do
    echo "$index: ${languages[index]}"
done
```

### Important distinction: `[@]` vs `[*]`

Inside double quotes:

```bash
"${array[@]}"
```

Expands each element as a separate word.

```bash
"${array[*]}"
```

Expands all elements as one word, joined using the first character of IFS.

Example:

```bash
arr=("Hello World" "Bash")

printf '<%s>\n' "${arr[@]}"
```

Output:

```text
<Hello World>
<Bash>
```

But:

```bash
printf '<%s>\n' "${arr[*]}"
```

Produces one argument:

```text
<Hello World Bash>
```

---

## 3. Associative Arrays

Associative arrays use strings as keys instead of numeric indices.

They require Bash 4.0 or later.

### Declaration

```bash
declare -A student
```

### Assigning values

```bash
student[name]="Prabhath"
student[course]="Software Engineering"
student[university]="MANIT"
```

### Accessing values

```bash
echo "${student[name]}"
echo "${student[course]}"
```

### Declaration and initialization

```bash
declare -A student=(
    [name]="Prabhath"
    [course]="Software Engineering"
    [university]="MANIT"
)
```

### Getting all keys

```bash
echo "${!student[@]}"
```

### Getting all values

```bash
echo "${student[@]}"
```

### Getting number of elements

```bash
echo "${#student[@]}"
```

### Iterating

```bash
for key in "${!student[@]}"; do
    echo "$key: ${student[$key]}"
done
```

Important: Associative array iteration order should not be relied upon.

### Removing an entry

```bash
unset 'student[course]'
```

### Important edge case

Declare an associative array explicitly:

```bash
declare -A config
```

Do not assume that assigning string keys to an ordinary indexed array will convert it into an associative array.

---

## 4. IFS: Internal Field Separator

IFS is a special Bash variable that determines how Bash splits unquoted expansion results and input fields in contexts such as `read`.

Default value:

```bash
IFS=$' \t\n'
```

It contains:
- Space
- Tab
- Newline

### Example

```bash
data="apple banana orange"

for item in $data; do
    echo "$item"
done
```

Output:

```text
apple
banana
orange
```

### Changing IFS

```bash
IFS=","
data="apple,banana,orange"

for item in $data; do
    echo "$item"
done
```

### Using IFS with `read`

```bash
IFS="," read -r first second third <<< "apple,banana,orange"

echo "$first"
echo "$second"
echo "$third"
```

### Important: Restore IFS

Avoid permanently changing the global IFS unnecessarily.

Use a local change:

```bash
old_ifs=$IFS
IFS=","

# Operations

IFS=$old_ifs
```

Or use a subshell:

```bash
(
    IFS=","
    # Operations
)
```

### Important edge cases

IFS is not a general-purpose CSV parser.

CSV may contain:
- Quoted commas
- Escaped quotes
- Embedded newlines

For real CSV parsing, use an appropriate CSV parser rather than relying on simple IFS splitting.

---

## 5. Command Substitution

Command substitution captures the output of a command and places it into a variable or another command.

### Syntax

Modern syntax:

```bash
$(command)
```

Older syntax:

```bash
`command`
```

Recommended:

```bash
current_date=$(date)
```

Example:

```bash
username=$(whoami)

echo "Current user: $username"
```

### Nested command substitution

```bash
result=$(echo "Today is $(date +%A)")

echo "$result"
```

### Using directly

```bash
echo "Current directory: $(pwd)"
```

### Important behaviour

Command substitution removes trailing newline characters from command output.

Example:

```bash
result=$(printf 'Hello\n\n')

printf '%s' "$result"
```

The trailing newlines are removed.

Command substitution also executes in a subshell environment.

Therefore, changes to ordinary shell variables inside it do not normally affect the parent shell.

Example:

```bash
count=10

result=$(count=20; echo "$count")

echo "$result"  # 20
echo "$count"   # 10
```

### Common mistake: Losing exit status

```bash
result=$(some_command)
status=$?
```

This captures the exit status of the command substitution.

Do not run another command between these two lines if you need that status.

---

## 6. Arithmetic Expressions

Bash supports integer arithmetic.

### Arithmetic expansion

```bash
a=10
b=20

result=$((a + b))

echo "$result"
```

### Supported operators

| Operator | Meaning |
|---|---|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Integer division |
| `%` | Modulus |
| `**` | Exponentiation |
| `++` | Increment |
| `--` | Decrement |

Examples:

```bash
echo $((10 + 5))
echo $((10 - 5))
echo $((10 * 5))
echo $((10 / 3))
echo $((10 % 3))
echo $((2 ** 3))
```

Output:

```text
15
5
50
3
1
8
```

### Arithmetic evaluation

```bash
((count++))
```

This increments `count`.

### Pre-increment vs post-increment

```bash
count=5

echo $((count++))  # 5
echo "$count"      # 6
```

```bash
count=5

echo $((++count))  # 6
echo "$count"      # 6
```

### Integer division

Bash does not perform floating-point arithmetic natively.

```bash
echo $((10 / 3))
```

Output:

```text
3
```

For decimal arithmetic, use tools such as `awk` or `bc`.

```bash
awk 'BEGIN { print 10 / 3 }'
```

### Division by zero

```bash
echo $((10 / 0))
```

This causes an arithmetic error.

Always validate the divisor before division.

### Arithmetic overflow

Bash uses fixed-width integer arithmetic, and overflow can produce unexpected results.

Do not use Bash arithmetic for arbitrary-precision calculations.

---

## 7. Process Substitution

Process substitution allows the output or input of a command to be treated like a filename.

### Syntax

```bash
<(command)
```

```bash
>(command)
```

### Input process substitution

```bash
diff <(sort file1.txt) <(sort file2.txt)
```

Here:
1. The first command sorts `file1.txt`.
2. The second command sorts `file2.txt`.
3. `diff` compares their outputs.

Without process substitution, temporary files would commonly be required.

### Another example

```bash
wc -l <(find . -type f)
```

This counts the lines in the generated list of filenames.

### Output process substitution

```bash
echo "Hello World" > >(tr '[:lower:]' '[:upper:]')
```

The output is passed to the process running `tr`.

### Process substitution vs command substitution

| Feature | Command substitution | Process substitution |
|---|---|---|
| Syntax | `$(command)` | `<(command)` |
| Result | Captured text | Filename-like reference |
| Typical use | Assign command output | Supply command output as input |
| Example | `x=$(date)` | `diff <(cmd1) <(cmd2)` |

### Important limitations

Process substitution is Bash-specific and is not supported by every POSIX shell.

The filename-like reference often represents a `/dev/fd` entry or a FIFO, depending on the system.

It should not be assumed to be a regular file.

---

## 8. Quick Revision

| Concept | Important syntax |
|---|---|
| Case statement | `case ... in ... esac` |
| Indexed array | `arr=(a b c)` |
| Associative array | `declare -A arr` |
| Array values | `${arr[@]}` |
| Array keys | `${!arr[@]}` |
| Array length | `${#arr[@]}` |
| IFS | `IFS=","` |
| Command substitution | `$(command)` |
| Arithmetic expansion | `$((expression))` |
| Arithmetic evaluation | `((expression))` |
| Input process substitution | `<(command)` |
| Output process substitution | `>(command)` |

**Important:** Always distinguish between an array's element count, its indices, and its expanded values.
