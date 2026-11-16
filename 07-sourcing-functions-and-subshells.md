
# Part 07: Sourcing Code, Subshells and Return vs Output

## 1. Introduction

As Bash scripts become larger, maintaining all functionality in a single file becomes difficult.

A better approach is to divide code into reusable files and functions.

Bash provides the `source` builtin to execute another file within the current shell environment.

This allows us to reuse:
- Functions
- Variables
- Configuration
- Common utilities
- Shared logic

---

## 2. Executing a Script vs Sourcing a Script

There are two common ways to execute another Bash file.

### Method 1: Execute normally

```bash
./script.sh
```

Or:

```bash
bash script.sh
```

This executes the script in a separate process environment.

Changes to ordinary shell variables do not affect the parent shell.

Example:

`child.sh`

```bash
#!/usr/bin/env bash

name="Prabhath"

echo "Inside child: $name"
```

Parent shell:

```bash
name="Alice"

bash child.sh

echo "Inside parent: $name"
```

Output:

```text
Inside child: Prabhath
Inside parent: Alice
```

### Method 2: Source the file

```bash
source script.sh
```

Short form:

```bash
. script.sh
```

This executes the file in the current shell environment.

Example:

```bash
name="Alice"

source child.sh

echo "Inside parent: $name"
```

Output:

```text
Inside child: Prabhath
```

The variable changed because the sourced file executes in the current shell.

### Important distinction

| Feature | Normal execution | Sourcing |
|---|---|---|
| Execution environment | Separate process | Current shell |
| Variable changes | Isolated from parent | Affect current shell |
| Function definitions | Not imported automatically | Available afterward |
| Working directory changes | Do not affect parent | Affect current shell |
| Script needs executable permission | For direct `./script.sh` execution | No |
| Shebang | Used for direct execution | Not used to select interpreter |

Sourcing should be done carefully because commands such as `cd`, `exit`, `unset` and variable assignments can affect your current shell.

---

## 3. Reusing Functions Across Files

Suppose we have this structure:

```text
project/
│
├── main.sh
└── functions.sh
```

### functions.sh

```bash
greet() {
    echo "Hello, $1"
}

add() {
    echo "$(($1 + $2))"
}
```

### main.sh

```bash
#!/usr/bin/env bash

source ./functions.sh

greet "Prabhath"

result=$(add 10 20)

echo "Sum: $result"
```

Execute:

```bash
bash main.sh
```

Output:

```text
Hello, Prabhath
Sum: 30
```

The functions become available to `main.sh` after sourcing `functions.sh`.

---

## 4. The Dot Operator

The dot (`.`) is an alternative to `source`.

Both perform the same operation in Bash.

```bash
source functions.sh
```

Equivalent to:

```bash
. functions.sh
```

The dot form is specified by POSIX and is useful when writing portable shell scripts.

However, the file must still contain syntax supported by the shell doing the sourcing.

---

## 5. Relative Paths and File Location

One common problem is sourcing files using relative paths.

Example:

```bash
source ./functions.sh
```

This searches relative to the current working directory, not necessarily the directory containing the script.

If a script is executed from another directory, it may fail.

### More reliable approach

Use `BASH_SOURCE` to identify the script's own location.

```bash
#!/usr/bin/env bash

SCRIPT_DIR="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)"

source "$SCRIPT_DIR/functions.sh"
```

Explanation:

- `${BASH_SOURCE[0]}` identifies the current script source.
- `dirname` extracts its directory.
- `cd` changes into that directory inside a subshell.
- `pwd` obtains its absolute path.
- `SCRIPT_DIR` stores the result.

This is more reliable for ordinary scripts launched through paths or symlinks, although complex symlink layouts may require additional resolution logic.

---

## 6. Avoiding Repeated Sourcing

Sometimes the same file may be sourced more than once.

This can lead to repeated initialization or unexpected side effects.

A common solution is an include guard.

```bash
# functions.sh

if [[ -n "${FUNCTIONS_LOADED:-}" ]]; then
    return 0
fi

FUNCTIONS_LOADED=1

greet() {
    echo "Hello, $1"
}
```

Now the file's main definitions execute only once per shell environment.

### Understanding the condition

```bash
${FUNCTIONS_LOADED:-}
```

Uses the variable's value if it is set and non-null. Otherwise, it expands to an empty string.

The `return` statement exits the sourced file without terminating the calling shell.

Important: A file intended to be sourced should not blindly use `exit`, because it can terminate the shell that sourced it.

---

## 7. Curly Braces vs Parentheses

Bash provides two important ways to group commands.

### Curly braces `{ }`

Commands inside curly braces execute in the current shell environment.

Syntax:

```bash
{
    command1
    command2
}
```

Example:

```bash
count=10

{
    count=20
}

echo "$count"
```

Output:

```text
20
```

The variable change remains because the commands execute in the current shell.

Important syntax rules:
- A space is required after `{`.
- A command separator is required before `}`.
- The closing `}` must be a separate shell token.

Correct:

```bash
{
    echo "Hello"
    echo "World"
}
```

Incorrect:

```bash
{echo "Hello"}
```

### Parentheses `( )`

Commands inside parentheses execute in a subshell environment.

Syntax:

```bash
(
    command1
    command2
)
```

Example:

```bash
count=10

(
    count=20
)

echo "$count"
```

Output:

```text
10
```

The assignment happened in the subshell and did not change the parent shell variable.

---

## 8. Comparing Curly Braces and Parentheses

| Feature | `{ }` | `( )` |
|---|---|---|
| Execution environment | Current shell | Subshell |
| Variable changes persist | Yes | No |
| Directory changes persist | Yes | No |
| Can group commands | Yes | Yes |
| Shell process isolation | No | Yes |

### Example: Working directory

Using curly braces:

```bash
pwd

{
    cd /tmp
}

pwd
```

The second `pwd` displays `/tmp`.

Using parentheses:

```bash
pwd

(
    cd /tmp
)

pwd
```

The final `pwd` displays the original directory.

### Example: Localizing temporary changes

Subshells are useful when temporary changes should not affect the parent.

```bash
(
    export APP_ENV="testing"
    cd /tmp
    echo "$APP_ENV"
)
```

After the subshell ends, the parent shell retains its original environment and directory.

### Important edge case

A subshell is not a complete isolated operating system environment.

It generally inherits environment and file descriptors from its parent. It can still:
- Read and write accessible files.
- Start processes.
- Modify shared external resources.
- Communicate through files, pipes and other IPC mechanisms.

The isolation mainly concerns shell execution state.

---

## 9. Return vs Output

This is one of the most important concepts in Bash functions.

Bash functions can communicate results in two primary ways:

1. Return an exit status.
2. Print data to stdout.

These are not interchangeable.

### Return: Communicating success or failure

The `return` keyword sets the function's exit status.

```bash
check_number() {
    if (( $1 > 10 )); then
        return 0
    else
        return 1
    fi
}
```

Call it:

```bash
if check_number 15; then
    echo "Number is greater than 10"
else
    echo "Number is not greater than 10"
fi
```

Conventionally:
- `0` means success.
- Non-zero means failure.

Bash function return statuses are limited to 0 through 255.

### Output: Communicating actual data

To send actual data from a function, use stdout.

```bash
add() {
    echo "$(($1 + $2))"
}
```

Capture the result:

```bash
result=$(add 10 20)

echo "$result"
```

Output:

```text
30
```

### Why not use return for normal data?

Consider:

```bash
add() {
    return $(($1 + $2))
}
```

This does not return the number as ordinary data.

It attempts to use the number as an exit status.

Exit statuses are restricted to 0 through 255, so larger numbers cannot be represented as expected.

---

## 10. Capturing Output vs Capturing Exit Status

Consider:

```bash
get_value() {
    echo "42"
    return 0
}
```

Call it:

```bash
result=$(get_value)
status=$?
```

Now:

```bash
echo "$result"
echo "$status"
```

Output:

```text
42
0
```

The two values have different purposes:

- `$result` contains the function's stdout.
- `$status` contains the function's exit status.

### Important: Avoid mixing data and logs on stdout

Bad approach:

```bash
calculate() {
    echo "Calculating..."
    echo "$(($1 + $2))"
}
```

Capturing this:

```bash
result=$(calculate 10 20)
```

produces:

```text
Calculating...
30
```

If the caller expects only the number, this is problematic.

Better:

```bash
calculate() {
    echo "Calculating..." >&2
    echo "$(($1 + $2))"
}
```

Now:
- Informational messages go to stderr.
- Actual result goes to stdout.

This makes functions easier to compose.

---

## 11. Combining Return Status and Output

A function can provide both an exit status and output.

Example:

```bash
divide() {
    local numerator=$1
    local denominator=$2

    if (( denominator == 0 )); then
        echo "Division by zero" >&2
        return 1
    fi

    echo "$((numerator / denominator))"
    return 0
}
```

Call it:

```bash
if result=$(divide 20 4); then
    echo "Result: $result"
else
    echo "Calculation failed"
fi
```

Output:

```text
Result: 5
```

This is a clean pattern for Bash functions.

---

## 12. Command Substitution and Subshell Side Effects

One important consequence of command substitution is that it normally runs in a subshell environment.

Example:

```bash
count=10

result=$(count=20; echo "$count")

echo "$result"
echo "$count"
```

Output:

```text
20
10
```

The child environment changed, but the parent variable remained unchanged.

### Common mistake

```bash
count=0

increment() {
    ((count++))
}

increment

echo "$count"
```

Output:

```text
1
```

But:

```bash
count=0

result=$(increment)

echo "$count"
```

The parent variable does not receive the change because the function ran through command substitution.

### Practical lesson

If a function must modify variables in the current shell, call it directly.

If you only need its output, command substitution is appropriate.

---

## 13. Complete Example: Modular Bash Project

Project structure:

```text
calculator/
│
├── main.sh
├── math.sh
└── utils.sh
```

### math.sh

```bash
add() {
    echo "$(($1 + $2))"
}

subtract() {
    echo "$(($1 - $2))"
}

multiply() {
    echo "$(($1 * $2))"
}

divide() {
    if (( $2 == 0 )); then
        echo "Division by zero" >&2
        return 1
    fi

    echo "$(($1 / $2))"
}
```

### utils.sh

```bash
print_header() {
    echo "======================"
    echo "    Bash Calculator"
    echo "======================"
}
```

### main.sh

```bash
#!/usr/bin/env bash

SCRIPT_DIR="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)"

source "$SCRIPT_DIR/math.sh"
source "$SCRIPT_DIR/utils.sh"

print_header

read -r -p "Enter first number: " a
read -r -p "Enter second number: " b

if [[ ! "$a" =~ ^-?[0-9]+$ || ! "$b" =~ ^-?[0-9]+$ ]]; then
    echo "Invalid input" >&2
    exit 1
fi

echo "Addition: $(add "$a" "$b")"
echo "Subtraction: $(subtract "$a" "$b")"
echo "Multiplication: $(multiply "$a" "$b")"

if result=$(divide "$a" "$b"); then
    echo "Division: $result"
else
    echo "Division failed"
fi
```

Run:

```bash
bash main.sh
```

This example demonstrates:
- Modular programming.
- Sourcing external files.
- Reusable functions.
- Command substitution.
- Error handling.
- Return status.
- Output handling.
- Reliable file paths.

---

## 14. Quick Revision

| Concept | Meaning |
|---|---|
| `source file.sh` | Execute file in current shell |
| `. file.sh` | POSIX-compatible sourcing syntax |
| `bash file.sh` | Execute in another Bash process |
| `${BASH_SOURCE[0]}` | Current script source path |
| `{ commands; }` | Execute grouped commands in current shell |
| `( commands )` | Execute grouped commands in subshell |
| `return 0` | Return success status |
| `return 1` | Return failure status |
| `echo` | Send data to stdout |
| `>&2` | Send output to stderr |
| `$(command)` | Capture command output |

### Most Important Takeaways

1. Sourcing executes code in the current shell.
2. Normal script execution does not automatically modify the parent shell's variables.
3. Curly braces group commands in the current shell.
4. Parentheses execute commands in a subshell.
5. `return` communicates an exit status, not arbitrary data.
6. Stdout is appropriate for returning actual data.
7. Stderr is appropriate for errors and diagnostic messages.
8. Command substitution captures stdout and normally runs in a subshell.

**Golden rule:** Use return statuses to communicate whether an operation succeeded, and stdout to communicate the result of an operation.
