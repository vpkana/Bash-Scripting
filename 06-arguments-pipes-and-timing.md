
# Part 06: Bash Arguments, Pipe Status and Timing

## 1. Command-Line Arguments

Bash scripts can receive arguments when executed.

Example:

```bash
./script.sh Prabhath 23 MANIT
```

Inside the script, these arguments are accessible through positional parameters.

### Positional parameters

| Parameter | Meaning |
|---|---|
| `$0` | Script name |
| `$1` | First argument |
| `$2` | Second argument |
| `$3` | Third argument |
| `${10}` | Tenth argument |
| `$#` | Number of arguments |
| `$@` | All positional arguments |
| `$*` | All positional arguments |

Example:

```bash
#!/usr/bin/env bash

echo "Script: $0"
echo "First: $1"
echo "Second: $2"
echo "Total arguments: $#"
```

Execute:

```bash
./script.sh Alice 25
```

### Accessing arguments beyond 9

Use braces.

```bash
echo "${10}"
echo "${11}"
```

Without braces, `$10` is interpreted as `$1` followed by the literal character `0`.

---

## 2. Understanding `$@` and `$*`

Both represent positional arguments, but their behaviour differs when quoted.

### Unquoted `$@`

```bash
"$@" 
```

The recommended quoted form is:

```bash
printf '<%s>\n' "$@"
```

Each argument is preserved as a separate word.

### Quoted `$*`

```bash
printf '<%s>\n' "$*"
```

All arguments are joined into one word using the first character of IFS.

### Demonstration

Suppose:

```bash
./script.sh "Hello World" Bash Linux
```

Inside the script:

```bash
printf '<%s>\n' "$@"
```

Output:

```text
<Hello World>
<Bash>
<Linux>
```

But:

```bash
printf '<%s>\n' "$*"
```

Output:

```text
<Hello World Bash Linux>
```

### Golden rule

Use:

```bash
"$@"
```

when forwarding arguments to another command or function.

Example:

```bash
run_command() {
    command "$@"
}

run_command ls -l
```

This preserves argument boundaries.

---

## 3. Argument Validation

A script should validate its input instead of blindly trusting arguments.

Example:

```bash
#!/usr/bin/env bash

if (( $# != 2 )); then
    echo "Usage: $0 <number1> <number2>" >&2
    exit 1
fi

a=$1
b=$2

if [[ ! "$a" =~ ^-?[0-9]+$ || ! "$b" =~ ^-?[0-9]+$ ]]; then
    echo "Error: Both arguments must be integers." >&2
    exit 1
fi

echo "Sum: $((a + b))"
```

### Important considerations

- Check the expected number of arguments.
- Validate the data type.
- Handle missing arguments.
- Quote variable expansions.
- Print errors to stderr.
- Return meaningful exit statuses.

---

## 4. `shift`: Moving Positional Parameters

The `shift` builtin removes positional parameters from the beginning of the argument list.

Example:

```bash
#!/usr/bin/env bash

echo "$1"

shift

echo "$1"
```

Execute:

```bash
./script.sh Alice Bob
```

Output:

```text
Alice
Bob
```

### Shifting multiple positions

```bash
shift 2
```

Removes the first two positional parameters.

### Processing arguments using shift

```bash
#!/usr/bin/env bash

while (( $# > 0 )); do
    echo "Argument: $1"
    shift
done
```

This is useful for implementing simple command-line argument parsers.

Important: `shift` modifies the positional parameters of the current shell or function.

---

## 5. Understanding Exit Status

Every command returns an exit status.

Conventionally:

| Status | Meaning |
|---|---|
| 0 | Success |
| Non-zero | Failure or another condition |

### `$?`

Stores the exit status of the most recently executed command.

```bash
ls

echo "$?"
```

If `ls` succeeds, the output is normally:

```text
0
```

### Example

```bash
mkdir test

if (( $? == 0 )); then
    echo "Directory created"
else
    echo "Creation failed"
fi
```

Better approach:

```bash
if mkdir test; then
    echo "Directory created"
else
    echo "Creation failed"
fi
```

Avoid unnecessary use of `$?` when a command can be placed directly in an `if` statement.

### Exit status of a script

Use `exit` to terminate a script with a specific status.

```bash
exit 0
```

```bash
exit 1
```

### Important distinction

`return` exits a function with an exit status.

`exit` terminates the current shell or script.

---

## 6. Understanding Pipes

A pipe passes the standard output of one command to the standard input of another.

Syntax:

```bash
command1 | command2
```

Example:

```bash
ls -l | grep ".txt"
```

Execution flow:

```text
ls -l
   |
   v
stdout
   |
   v
grep ".txt"
   |
   v
Matching output
```

### Multiple pipes

```bash
cat application.log | grep ERROR | wc -l
```

A cleaner version:

```bash
grep ERROR application.log | wc -l
```

Each pipe connects one process to another.

---

## 7. The Pipe Status Problem

By default, the exit status of a pipeline is the exit status of its last command.

Example:

```bash
false | true

echo "$?"
```

Output:

```text
0
```

Why?

- `false` returns 1.
- `true` returns 0.
- The pipeline returns the status of the last command, which is `true`.

This can hide failures in earlier pipeline commands.

### `PIPESTATUS`

Bash provides a special array called `PIPESTATUS`.

It stores the exit status of each command in the most recently executed foreground pipeline.

Example:

```bash
false | true | false

printf '%s\n' "${PIPESTATUS[@]}"
```

Output:

```text
1
0
1
```

### Important edge case

`PIPESTATUS` is updated after commands execute.

Capture it immediately:

```bash
command1 | command2 | command3
statuses=("${PIPESTATUS[@]}")
```

Do not execute another command before capturing it.

For example, this is problematic:

```bash
command1 | command2
echo "Finished"
echo "${PIPESTATUS[@]}"
```

The previous pipeline's status information has already been replaced.

---

## 8. `set -o pipefail`

Bash provides `pipefail` to make pipeline failures easier to detect.

```bash
set -o pipefail
```

With this option, a pipeline returns:
- Zero if all commands succeed.
- The status of the rightmost command that failed, if any command fails.

Example:

```bash
set -o pipefail

false | true

echo "$?"
```

Output:

```text
1
```

### Combining with `set -e`

A commonly used strict-mode configuration:

```bash
set -euo pipefail
```

Meaning:

| Option | Behaviour |
|---|---|
| `-e` | Exit on certain unhandled command failures |
| `-u` | Treat unset variables as errors in many contexts |
| `-o pipefail` | Detect failures within pipelines |

Important: `set -e` has exceptions and context-dependent behaviour, particularly with conditionals, functions and compound commands. It is not a substitute for explicit error handling.

---

## 9. Timing Commands

Bash provides the `time` keyword for measuring command execution time.

### Basic usage

```bash
time sleep 2
```

Possible output:

```text
real    0m2.003s
user    0m0.001s
sys     0m0.002s
```

### Understanding the three values

| Metric | Meaning |
|---|---|
| real | Actual elapsed wall-clock time |
| user | CPU time spent executing user-space code |
| sys | CPU time spent executing kernel code |

### Real vs User vs System time

**Real time:** How much time actually passed.

**User time:** CPU time spent executing application instructions.

**System time:** CPU time spent performing operating-system operations on behalf of the process.

For example, a program waiting for a network response may have high real time but relatively low CPU time.

### Timing a pipeline

```bash
time cat large.txt | grep ERROR | wc -l
```

This measures the pipeline.

### Bash `TIMEFORMAT`

Bash allows customization of the time output.

```bash
TIMEFORMAT='Elapsed: %3R seconds'

time sleep 2
```

Useful format specifiers:

| Specifier | Meaning |
|---|---|
| `%R` | Real time |
| `%U` | User time |
| `%S` | System time |
| `%P` | CPU percentage |

Example:

```bash
TIMEFORMAT='Real: %R | User: %U | System: %S | CPU: %P'

time sleep 1
```

---

## 10. Practical Example: A Command-Line Calculator

```bash
#!/usr/bin/env bash

set -o pipefail

if (( $# != 3 )); then
    echo "Usage: $0 <number1> <operator> <number2>" >&2
    exit 1
fi

a=$1
operator=$2
b=$3

if [[ ! "$a" =~ ^-?[0-9]+$ || ! "$b" =~ ^-?[0-9]+$ ]]; then
    echo "Both operands must be integers." >&2
    exit 1
fi

case "$operator" in
    +)
        result=$((a + b))
        ;;
    -)
        result=$((a - b))
        ;;
    x|\*)
        result=$((a * b))
        ;;
    /)
        if (( b == 0 )); then
            echo "Cannot divide by zero." >&2
            exit 1
        fi
        result=$((a / b))
        ;;
    *)
        echo "Unsupported operator." >&2
        exit 1
        ;;
esac

echo "Result: $result"
```

Execute:

```bash
./calculator.sh 10 + 20
./calculator.sh 10 x 20
./calculator.sh 10 / 3
```

Note: Division uses integer arithmetic.

---

## 11. Quick Revision

| Concept | Syntax |
|---|---|
| First argument | `$1` |
| Number of arguments | `$#` |
| All arguments separately | `"$@"` |
| All arguments as one word | `"$*"` |
| Tenth argument | `${10}` |
| Shift arguments | `shift` |
| Previous exit status | `$?` |
| Exit script | `exit` |
| Return from function | `return` |
| Pipe | `\|` |
| Pipeline statuses | `${PIPESTATUS[@]}` |
| Enable pipefail | `set -o pipefail` |
| Measure execution time | `time command` |

**Important:** Always capture `PIPESTATUS` immediately after the pipeline if you need individual command statuses.
