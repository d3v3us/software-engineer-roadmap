# Shell Scripting Patterns Deep Dive - Complete Understanding

## Table of Contents
1. [What is Shell Scripting?](#what-is-shell-scripting)
2. [Why Shell Scripting Matters](#why-shell-scripting-matters)
3. [Common Patterns](#common-patterns)
4. [File Operations](#file-operations)
5. [Text Processing](#text-processing)
6. [Error Handling](#error-handling)
7. [Best Practices](#best-practices)

---

## What is Shell Scripting?

### Definition

**Shell Scripting**: Writing scripts using shell commands for automation.

**Key Characteristics:**
- **Automation**: Task automation
- **System administration**: System administration
- **Text processing**: Text processing
- **Command chaining**: Command chaining

### Real-World Analogy

**Shell Scripting = Automation:**
- **Tasks**: Repetitive tasks
- **Scripts**: Automation scripts
- **Efficiency**: Increased efficiency
- **Consistency**: Consistent execution

**System Administration:**
- **Scripts**: Shell scripts
- **Automation**: Task automation
- **Tools**: System tools
- **Productivity**: Increased productivity

---

## Why Shell Scripting Matters?

### Benefits

**1. Automation:**
```
Shell scripting
  ↓
Task automation
  ↓
Increased efficiency
```

**2. System Administration:**
```
Shell scripting
  ↓
System administration
  ↓
Efficient management
```

**3. Text Processing:**
```
Shell scripting
  ↓
Text processing
  ↓
Data manipulation
```

---

## Common Patterns

### Batch File Renaming

**Pattern:**
```bash
#!/bin/bash

# Rename files: aa-1.1.txt -> aa.txt
for file in *-*.txt; do
    # Extract base name (before first dash and version)
    newname=$(echo "$file" | sed 's/-[0-9.]*\.txt$/.txt/')
    mv "$file" "$newname"
done
```

**Advanced Pattern:**
```bash
#!/bin/bash

# Rename: aa-1.1.txt -> aa.txt, ab-1.2.3.4.txt -> ab.txt
for file in *-*.txt; do
    # Remove version pattern (digits and dots before .txt)
    newname=$(echo "$file" | sed -E 's/-[0-9]+(\.[0-9]+)*\.txt$/.txt/')
    if [ "$file" != "$newname" ]; then
        mv "$file" "$newname"
    fi
done
```

### Word Frequency Analysis

**Pattern:**
```bash
#!/bin/bash

# Read file and find n most frequent words
file="$1"
n="${2:-10}"

# Extract words, convert to lowercase, count, sort, get top n
cat "$file" | \
    tr -s '[:space:]' '\n' | \
    tr '[:upper:]' '[:lower:]' | \
    sed 's/[^a-z0-9]//g' | \
    grep -v '^$' | \
    sort | \
    uniq -c | \
    sort -rn | \
    head -n "$n" | \
    awk '{print $2, $1}'
```

### File Operations

**Copy to Remote Server:**
```bash
#!/bin/bash

# Methods to copy file to remote server

# 1. SCP
scp file.txt user@remote:/path/to/destination/

# 2. RSYNC
rsync -avz file.txt user@remote:/path/to/destination/

# 3. SFTP
sftp user@remote <<EOF
put file.txt /path/to/destination/
quit
EOF

# 4. SSH with cat
cat file.txt | ssh user@remote "cat > /path/to/destination/file.txt"

# 5. SSH with tar (for directories)
tar czf - directory/ | ssh user@remote "tar xzf - -C /path/to/destination/"
```

---

## File Operations

### File Processing

**Process Files:**
```bash
#!/bin/bash

# Process all files in directory
for file in *.txt; do
    echo "Processing $file"
    # Process file
    process_file "$file"
done
```

**Recursive Processing:**
```bash
#!/bin/bash

# Process files recursively
find . -name "*.txt" -type f | while read file; do
    echo "Processing $file"
    process_file "$file"
done
```

### File Validation

**Check File Exists:**
```bash
#!/bin/bash

if [ -f "$file" ]; then
    echo "File exists"
else
    echo "File does not exist"
fi
```

**Check Directory:**
```bash
#!/bin/bash

if [ -d "$dir" ]; then
    echo "Directory exists"
else
    echo "Directory does not exist"
fi
```

---

## Text Processing

### awk Patterns

**Common awk Usage:**
```bash
# Print specific column
awk '{print $1}' file.txt

# Print lines matching pattern
awk '/pattern/ {print}' file.txt

# Sum column
awk '{sum += $1} END {print sum}' file.txt

# Group and count
awk '{count[$1]++} END {for (word in count) print word, count[word]}' file.txt
```

### sed Patterns

**Common sed Usage:**
```bash
# Replace text
sed 's/old/new/g' file.txt

# Delete lines
sed '/pattern/d' file.txt

# Print specific lines
sed -n '10,20p' file.txt

# In-place edit
sed -i 's/old/new/g' file.txt
```

### grep Patterns

**Common grep Usage:**
```bash
# Search pattern
grep "pattern" file.txt

# Case insensitive
grep -i "pattern" file.txt

# Recursive
grep -r "pattern" directory/

# Count matches
grep -c "pattern" file.txt

# Show context
grep -C 3 "pattern" file.txt
```

---

## Error Handling

### Error Checking

**Check Command Success:**
```bash
#!/bin/bash

command
if [ $? -ne 0 ]; then
    echo "Command failed"
    exit 1
fi
```

**set -e:**
```bash
#!/bin/bash

set -e  # Exit on error
command1
command2  # Won't execute if command1 fails
```

**set -u:**
```bash
#!/bin/bash

set -u  # Exit on undefined variable
echo "$undefined_var"  # Will exit
```

**set -o pipefail:**
```bash
#!/bin/bash

set -o pipefail  # Exit on pipe failure
command1 | command2 | command3
```

### Error Handling Pattern

**Comprehensive Error Handling:**
```bash
#!/bin/bash

set -euo pipefail

# Error handler
error_exit() {
    echo "Error: $1" >&2
    exit 1
}

# Trap errors
trap 'error_exit "Command failed at line $LINENO"' ERR

# Your script
command1 || error_exit "command1 failed"
command2 || error_exit "command2 failed"
```

---

## Best Practices

### 1. Use Shebang

**Why:**
- **Interpreter**: Specify interpreter
- **Portability**: Better portability
- **Clarity**: Clear intent

**Guidelines:**
```bash
#!/bin/bash
# or
#!/usr/bin/env bash
```

### 2. Quote Variables

**Why:**
- **Spaces**: Handle spaces
- **Special characters**: Handle special characters
- **Safety**: Safer scripts

**Guidelines:**
```bash
# Good
echo "$variable"

# Bad
echo $variable
```

### 3. Use Functions

**Why:**
- **Reusability**: Reusable code
- **Organization**: Better organization
- **Maintainability**: Easier maintenance

**Guidelines:**
```bash
function process_file() {
    local file="$1"
    # Process file
}
```

### 4. Validate Input

**Why:**
- **Safety**: Safer scripts
- **Error handling**: Better error handling
- **User experience**: Better UX

**Guidelines:**
```bash
if [ -z "$1" ]; then
    echo "Usage: $0 <file>"
    exit 1
fi
```

### 5. Use Meaningful Names

**Why:**
- **Readability**: More readable
- **Maintainability**: Easier maintenance
- **Understanding**: Better understanding

**Guidelines:**
```bash
# Good
backup_file="/path/to/backup"

# Bad
bf="/path/to/backup"
```

---

## Summary

Shell scripting patterns enable efficient automation and system administration. Understanding common patterns, file operations, text processing, error handling, and best practices is crucial for effective shell scripting.

**Key Takeaways:**
- **Shell scripting**: Writing scripts for automation (automation, system administration, text processing, command chaining)
- **Common patterns**: Batch file renaming, word frequency analysis, file operations
- **File operations**: File processing, recursive processing, file validation
- **Text processing**: awk patterns, sed patterns, grep patterns
- **Error handling**: Error checking (check command success, set -e, set -u, set -o pipefail), error handling pattern (comprehensive error handling, trap errors)
- **Best practices**: Use shebang, quote variables, use functions, validate input, use meaningful names

**Shell Scripting:**
- **Patterns**: Common patterns
- **Operations**: File operations
- **Processing**: Text processing
- **Error handling**: Robust error handling

**Best Practices:**
- Use shebang
- Quote variables
- Use functions
- Validate input
- Use meaningful names

**Next Steps:**
- Learn patterns
- Practice scripting
- Build scripts
- Share knowledge

