
# Part 02: Manual Pages, Commands and Variables

## 1. Understanding Manual Pages

Linux provides documentation through manual pages, commonly called man pages.

### `man`

```bash
man ls
```

Displays documentation for the `ls` command.

Other examples:

```bash
man grep
man bash
man chmod
```

Useful navigation:

| Key | Action |
|---|---|
| Space | Next page |
| b | Previous page |
| /word | Search |
| n | Next match |
| N | Previous match |
| q | Quit |

### Manual page sections

Manual pages are divided into sections.

| Section | Description |
|---|---|
| 1 | User commands |
| 2 | System calls |
| 3 | Library functions |
| 4 | Special files |
| 5 | File formats and conventions |
| 6 | Games |
| 7 | Miscellaneous |
| 8 | System administration commands |

Example:

```bash
man 1 printf
man 3 printf
```

These may describe different implementations of `printf`.

### `whatis`

Displays a short description.

```bash
whatis grep
```

### `apropos`

Searches manual page descriptions.

```bash
apropos network
```

### `command --help`

Many commands also support:

```bash
ls --help
grep --help
```

---

## 2. Understanding Commands

A command may be:

1. A shell builtin.
2. An external executable.
3. A shell function.
4. An alias.

### `type`

Identifies how Bash interprets a command.

```bash
type cd
type ls
type echo
```

Example:

```text
cd is a shell builtin
```

### `command`

Runs a command while bypassing shell function lookup.

```bash
command ls
```

It can also be used to avoid an alias or function with the same name.

### `which`

Finds the executable path of a command in `PATH`.

```bash
which python
```

Note: `which` is not guaranteed to identify shell builtins, aliases or functions reliably. Prefer `type` when investigating Bash command resolution.

### `type -a`

Displays all matching definitions.

```bash
type -a ls
```

This is useful when multiple commands, aliases or functions share a name.

---

## 3. Understanding the PATH Variable

`PATH` is an environment variable containing directories where the shell searches for executable programs.

```bash
echo "$PATH"
```

Example:

```text
/usr/local/bin:/usr/bin:/bin
```

Directories are separated using colons (`:`).

### Adding a directory to PATH

```bash
export PATH="$PATH:$HOME/bin"
```

This adds `$HOME/bin` to the end of PATH.

Important:
- This affects the current shell and subsequently started child processes.
- It does not permanently modify PATH unless placed in a shell startup file.
- Adding a directory to PATH can create security risks if it contains untrusted executables.

---

## 4. Introduction to Bash Variables

Variables store values that can be reused.

### Creating a variable

```bash
name="Prabhath"
age=23
```

Important: There must be no spaces around `=`.

Correct:

```bash
name="Prabhath"
```

Incorrect:

```bash
name = "Prabhath"
```

Bash interprets the incorrect example as a command named `name`.

### Accessing variables

Use `$` to expand a variable.

```bash
name="Prabhath"

echo "$name"
```

Output:

```text
Prabhath
```

Recommended:

```bash
echo "$name"
```

Instead of:

```bash
echo $name
```

Quoting prevents unwanted word splitting and pathname expansion.

### Curly braces

Braces clearly define variable boundaries.

```bash
name="Prabhath"

echo "${name}_developer"
```

Output:

```text
Prabhath_developer
```

Without braces:

```bash
echo "$name_developer"
```

Bash looks for a variable named `name_developer`.

---

## 5. Variable Types and Quoting

Bash variables are fundamentally strings, although Bash supports arithmetic contexts and indexed/associative arrays.

### Single quotes

Prevent variable expansion.

```bash
name="Prabhath"

echo '$name'
```

Output:

```text
$name
```

### Double quotes

Allow variable and command expansion.

```bash
echo "$name"
```

Output:

```text
Prabhath
```

### Unquoted expansion

```bash
echo $name
```

Unquoted expansion can cause:
- Word splitting.
- Filename expansion (globbing).

Example:

```bash
files="one file.txt"

printf '<%s>\n' $files
```

This can produce separate arguments:

```text
<one>
<file.txt>
```

Use:

```bash
printf '<%s>\n' "$files"
```

Output:

```text
<one file.txt>
```

---

## 6. Special Variables

Bash provides predefined variables.

| Variable | Meaning |
|---|---|
| `$HOME` | Home directory |
| `$USER` | Current username (commonly set) |
| `$PWD` | Current working directory |
| `$SHELL` | User's configured login shell |
| `$PATH` | Executable search path |
| `$?` | Exit status of previous command |
| `$$` | PID of current shell |
| `$!` | PID of most recently backgrounded job |
| `$0` | Script or shell name |
| `$#` | Number of positional parameters |
| `$@` | All positional parameters |
| `$*` | All positional parameters |

Example:

```bash
echo "Home: $HOME"
echo "Current directory: $PWD"
echo "Exit status: $?"
```

Important: `$SHELL` usually indicates the configured login shell, not necessarily the shell currently executing a script.

---

## 7. Environment Variables vs Shell Variables

A normal shell variable belongs to the current shell.

```bash
name="Prabhath"
```

An environment variable is exported to child processes.

```bash
export name="Prabhath"
```

Alternatively:

```bash
name="Prabhath"
export name
```

Example:

```bash
export APP_ENV="development"

bash -c 'echo "$APP_ENV"'
```

Output:

```text
development
```

Without `export`, the child Bash process normally does not receive the variable.

### Temporary environment variables

```bash
APP_ENV=testing ./application.sh
```

The variable is provided to that command's environment without permanently exporting it in the current shell.

---

## 8. Variable Scope and `unset`

### Removing a variable

```bash
name="Prabhath"
unset name
```

Check whether a variable is set:

```bash
[[ -v name ]] && echo "Set" || echo "Not set"
```

Important distinction:

```bash
name=""
```

The variable exists but contains an empty string.

```bash
unset name
```

The variable no longer exists.

These conditions are different:

```bash
[[ -v name ]]  # Is it set?
[[ -n ${name-} ]]  # Does it contain a non-empty value?
```

---

## 9. File Permissions

Linux permissions determine who can read, write or execute a file.

Three permission categories:

- User (owner)
- Group
- Others

Three permission types:

| Permission | Symbol | Numeric value |
|---|---|---|
| Read | r | 4 |
| Write | w | 2 |
| Execute | x | 1 |

Example:

```text
-rwxr-xr--
```

Interpretation:

| Category | Permissions |
|---|---|
| Owner | rwx |
| Group | r-x |
| Others | r-- |

### `chmod`

Changes permissions.

```bash
chmod +x script.sh
```

Numeric permissions:

```bash
chmod 755 script.sh
chmod 644 notes.txt
```

Meaning of `755`:

- Owner: 7 = 4 + 2 + 1 = rwx
- Group: 5 = 4 + 1 = r-x
- Others: 5 = 4 + 1 = r-x

Meaning of `644`:

- Owner: rw-
- Group: r--
- Others: r--

### `chown`

Changes ownership.

```bash
sudo chown user:group file.txt
```

### Important permission distinction

Execute permission on a script allows it to be executed directly if the interpreter and execution environment are available.

A script can also be run through Bash without the executable bit:

```bash
bash script.sh
```

This requires read permission on the script, but not execute permission on the script itself.

---

## 10. Quick Revision

- `man`: Read manual pages.
- `whatis`: Short command description.
- `apropos`: Search manual descriptions.
- `type`: Identify command type.
- `type -a`: Show all matching definitions.
- `which`: Locate executable paths.
- `PATH`: Search directories for executables.
- `$variable`: Access variable.
- `${variable}`: Explicit variable boundary.
- `export`: Pass variables to child processes.
- `unset`: Remove variables.
- `chmod`: Modify permissions.
- `chown`: Modify ownership.

**Golden rule:** Quote variable expansions unless you intentionally want word splitting or pathname expansion.
