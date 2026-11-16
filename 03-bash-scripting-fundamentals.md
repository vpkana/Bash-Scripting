
# Part 03: Bash Scripting Fundamentals

## 1. What is a Bash Script?

A Bash script is a file containing a sequence of commands that Bash executes.

Instead of manually executing commands one by one, we can store them in a file and execute the entire sequence.

Example:

```bash
#!/usr/bin/env bash

echo "Hello World"
echo "Welcome to Bash scripting!"
```

Save it as:

```text
hello.sh
```

Execute it:

```bash
bash hello.sh
```

Alternatively:

```bash
chmod +x hello.sh
./hello.sh
```

---

## 2. Understanding the Shebang

The first line:

```bash
#!/usr/bin/env bash
```

is called the shebang.

It tells the operating system which interpreter should execute the script when it is run directly.

Two common forms:

```bash
#!/bin/bash
```

```bash
#!/usr/bin/env bash
```

The second form searches for Bash using the environment's PATH.

The shebang is particularly important when executing a script directly:

```bash
./script.sh
```

When explicitly running:

```bash
bash script.sh
```

the Bash executable specified in the command is used instead.

---

## 3. Reading User Input

### `read`

The `read` builtin accepts input from the user.

```bash
#!/usr/bin/env bash

echo "Enter your name:"
read name

echo "Hello, $name"
```

A shorter approach:

```bash
read -p "Enter your name: " name
echo "Hello, $name"
```

### Reading multiple values

```bash
read -p "Enter first and last name: " first last

echo "First: $first"
echo "Last: $last"
```

By default, Bash uses IFS characters to split the input into variables. The final variable receives the remaining text.

### Useful options

```bash
read -r name       # Do not interpret backslashes as escapes
read -s password   # Silent input
read -t 10 name    # Timeout after 10 seconds
read -n 1 choice   # Read one character
```

Recommended:

```bash
read -r -p "Enter your name: " name
```

### Handling empty input

```bash
read -r -p "Enter your name: " name

if [[ -z "$name" ]]; then
    echo "Name cannot be empty"
else
    echo "Hello, $name"
fi
```

### Reading input safely

Use `read -r` when you want literal backslashes.

Without `-r`, backslash sequences can receive special treatment.

---

## 4. Functions

Functions allow us to group reusable commands.

### Syntax

```bash
function_name() {
    commands
}
```

Example:

```bash
greet() {
    echo "Hello, Bash!"
}

greet
```

### Functions with arguments

```bash
greet() {
    echo "Hello, $1"
}

greet "Prabhath"
```

Inside the function:

| Variable | Meaning |
|---|---|
| `$1` | First argument |
| `$2` | Second argument |
| `$#` | Number of arguments |
| `$@` | All arguments |
| `$0` | Script name, not function name |

Example:

```bash
add() {
    echo "$(($1 + $2))"
}

add 10 20
```

Output:

```text
30
```

### Local variables

Use `local` to restrict a variable to a function's scope.

```bash
greet() {
    local name="Prabhath"
    echo "$name"
}

greet
```

Without `local`, assigning to a variable inside a function can modify a variable in the surrounding shell.

### Returning values

Bash functions can return an exit status.

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
    echo "Greater than 10"
else
    echo "Not greater than 10"
fi
```

For actual data, use stdout:

```bash
add() {
    echo "$(($1 + $2))"
}

result=$(add 10 20)
echo "$result"
```

---

## 5. Conditional Statements

Conditional statements execute code depending on whether a condition is true or false.

### `if`

```bash
if [[ condition ]]; then
    commands
fi
```

Example:

```bash
age=20

if (( age >= 18 )); then
    echo "Eligible"
fi
```

### `if-else`

```bash
age=16

if (( age >= 18 )); then
    echo "Adult"
else
    echo "Minor"
fi
```

### `if-elif-else`

```bash
marks=85

if (( marks >= 90 )); then
    echo "Grade A"
elif (( marks >= 75 )); then
    echo "Grade B"
elif (( marks >= 60 )); then
    echo "Grade C"
else
    echo "Fail"
fi
```

### String comparisons

```bash
name="Prabhath"

if [[ "$name" == "Prabhath" ]]; then
    echo "Matched"
fi
```

| Operator | Meaning |
|---|---|
| `==` | Equal |
| `!=` | Not equal |
| `-z` | Empty string |
| `-n` | Non-empty string |

Examples:

```bash
[[ -z "$name" ]]
[[ -n "$name" ]]
```

### Numeric comparisons

Using `[[ ]]`:

| Operator | Meaning |
|---|---|
| `-eq` | Equal |
| `-ne` | Not equal |
| `-gt` | Greater than |
| `-ge` | Greater than or equal |
| `-lt` | Less than |
| `-le` | Less than or equal |

Example:

```bash
if [[ "$age" -ge 18 ]]; then
    echo "Adult"
fi
```

Using arithmetic evaluation:

```bash
if (( age >= 18 )); then
    echo "Adult"
fi
```

Prefer `(( ))` for arithmetic comparisons.

### File conditions

| Operator | Meaning |
|---|---|
| `-e` | Path exists |
| `-f` | Regular file exists |
| `-d` | Directory exists |
| `-r` | Readable |
| `-w` | Writable |
| `-x` | Executable |
| `-s` | Exists and is non-empty |

Example:

```bash
file="notes.txt"

if [[ -f "$file" ]]; then
    echo "File exists"
else
    echo "File does not exist"
fi
```

---

## 6. Logical Operators

### AND

Both conditions must be true.

```bash
if [[ "$age" -ge 18 && "$age" -le 60 ]]; then
    echo "Eligible"
fi
```

### OR

At least one condition must be true.

```bash
if [[ "$role" == "admin" || "$role" == "manager" ]]; then
    echo "Authorized"
fi
```

### NOT

Negates a condition.

```bash
if [[ ! -f "notes.txt" ]]; then
    echo "File missing"
fi
```

### Short-circuit execution

```bash
[[ -f "notes.txt" ]] && echo "Found"
[[ -f "notes.txt" ]] || echo "Missing"
```

The right-hand command of `&&` executes only if the left-hand command succeeds.

The right-hand command of `||` executes only if the left-hand command fails.

---

## 7. Loops

Loops execute commands repeatedly.

### `for` loop

```bash
for name in Alice Bob Charlie; do
    echo "Hello $name"
done
```

### C-style for loop

```bash
for (( i=1; i<=5; i++ )); do
    echo "$i"
done
```

### `while` loop

```bash
count=1

while (( count <= 5 )); do
    echo "$count"
    ((count++))
done
```

### `until` loop

Executes until the condition becomes true.

```bash
count=1

until (( count > 5 )); do
    echo "$count"
    ((count++))
done
```

### Infinite loop

```bash
while true; do
    echo "Running..."
    sleep 1
done
```

Stop with `Ctrl+C`.

### `break`

Terminates the current loop.

```bash
for (( i=1; i<=10; i++ )); do
    if (( i == 5 )); then
        break
    fi

    echo "$i"
done
```

### `continue`

Skips the current iteration.

```bash
for (( i=1; i<=5; i++ )); do
    if (( i == 3 )); then
        continue
    fi

    echo "$i"
done
```

---

## 8. Input and Output

### Standard input, output and error

Every process normally has three standard streams:

| Stream | File Descriptor | Purpose |
|---|---:|---|
| stdin | 0 | Input |
| stdout | 1 | Normal output |
| stderr | 2 | Error output |

### Output redirection

```bash
echo "Hello" > output.txt
```

Overwrites the file.

```bash
echo "Another line" >> output.txt
```

Appends to the file.

### Error redirection

```bash
ls invalid-directory 2> error.txt
```

### Separate output and error

```bash
command > output.txt 2> error.txt
```

### Discard output

```bash
command > /dev/null
```

Discard errors:

```bash
command 2> /dev/null
```

Discard both:

```bash
command > /dev/null 2>&1
```

### Pipes

Passes stdout of one command as stdin to another.

```bash
ls -l | grep ".txt"
```

---

## 9. Complete Practice Script

A simple calculator:

```bash
#!/usr/bin/env bash

read -r -p "Enter first number: " a
read -r -p "Enter second number: " b

if [[ ! "$a" =~ ^-?[0-9]+$ || ! "$b" =~ ^-?[0-9]+$ ]]; then
    echo "Please enter valid integers."
    exit 1
fi

echo "1. Addition"
echo "2. Subtraction"
echo "3. Multiplication"
echo "4. Division"

read -r -p "Choose operation: " choice

case "$choice" in
    1) echo "Result: $((a + b))" ;;
    2) echo "Result: $((a - b))" ;;
    3) echo "Result: $((a * b))" ;;
    4)
        if (( b == 0 )); then
            echo "Division by zero is not allowed."
            exit 1
        fi
        echo "Result: $((a / b))"
        ;;
    *)
        echo "Invalid choice."
        exit 1
        ;;
esac
```

Note: Bash arithmetic uses integer arithmetic here. Division truncates toward zero.

---

## 10. Quick Revision

- `#!/usr/bin/env bash`: Shebang.
- `read`: Accept user input.
- `function_name()`: Define a function.
- `local`: Declare a function-local variable.
- `if`: Conditional execution.
- `elif`: Additional condition.
- `else`: Fallback branch.
- `[[ ]]`: Conditional expression.
- `(( ))`: Arithmetic evaluation.
- `for`: Iterate over values.
- `while`: Repeat while condition succeeds.
- `until`: Repeat until condition succeeds.
- `break`: Exit loop.
- `continue`: Skip iteration.
- `exit`: Terminate script with an exit status.
- `return`: Return from a function with an exit status.

**Important:** A command's exit status determines success or failure. Conventionally, `0` means success and non-zero means failure.
