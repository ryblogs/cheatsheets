# fd Cheatsheet

A fast and user-friendly alternative to `find`. Uses regex by default, respects .gitignore, and has sane defaults.

## 🎯 Core Concepts

**Default behavior:**
- Uses **regex patterns** (not glob patterns)
- **Ignores hidden files** (dotfiles)
- **Respects .gitignore** in git repos
- Case-insensitive if pattern is lowercase, sensitive if uppercase

## 🔍 Basic Usage

```bash
# Simple search (finds "test" anywhere in filename)
fd test

# Search in specific directory
fd test /path/to/dir

# Search with full path output
fd test --full-path
fd test -p
```

## 🌟 Pattern Matching

### Regex Mode (Default)
```bash
# Match substring anywhere
fd config              # Finds: config.yml, .config, myconfig.txt

# Match at start (^ = beginning)
fd '^test'             # Finds: test.txt, test_file.py (not mytest.txt)

# Match at end ($ = end)
fd '\.txt$'            # Finds: file.txt, test.txt (not txt.old)

# Match with regex patterns
fd 'test[0-9]+'        # Finds: test1, test123 (not test)
fd '(foo|bar)'         # Finds files with foo OR bar
```

### Glob Mode (With --glob or -g)
```bash
# Use glob patterns
fd '*.txt' --glob      # All .txt files
fd '*test*' -g         # Glob-style wildcards

# Specific glob patterns
fd 'test*' -g          # Starts with test
fd '*test' -g          # Ends with test
fd '**/*.py' -g        # All .py files recursively
```

## 🔑 Essential Flags

### Show Hidden/Ignored Files
```bash
fd -H                  # Include hidden files (.dotfiles)
fd --hidden

fd -I                  # Include ignored files (.gitignore)
fd --no-ignore

fd -HI                 # Show EVERYTHING (like find)
fd --hidden --no-ignore
```

### File Type Filters
```bash
fd -t f                # Files only
fd -t d                # Directories only
fd -t l                # Symlinks only
fd -t x                # Executable files only

# Combine with pattern
fd -t f 'test'         # Only files named test
fd -t d 'config'       # Only directories named config
```

### Extension Filter
```bash
fd -e txt              # All .txt files (no dot!)
fd -e py -e js         # Multiple extensions
fd config -e yml       # config files with .yml extension
```

### Case Sensitivity
```bash
# Smart case (default)
fd test                # Case-insensitive (finds: test, Test, TEST)
fd Test                # Case-sensitive (has uppercase)

# Force modes
fd -i TEST             # Force case-insensitive
fd -s test             # Force case-sensitive
```

## 📁 Common Patterns

### Find Configuration Files
```bash
fd -H '^\..*rc$'       # Dotfiles ending in rc (.bashrc, .vimrc)
fd -H config           # Any file with config (including hidden)
fd -e yml -e yaml config  # Config files with specific extensions
```

### Find by Extension
```bash
fd -e md               # All markdown files
fd -e py               # All Python files
fd -e js -e ts         # JavaScript and TypeScript
```

### Find in Specific Locations
```bash
fd test src/           # Only in src directory
fd -e py ~/projects    # All .py files in projects
fd . /etc -d 1         # Max depth 1 in /etc
```

### Exclude Patterns
```bash
fd -E node_modules     # Exclude node_modules
fd -E '*.tmp' -E '*.log'  # Exclude multiple patterns
fd test -E test_*      # Find test but exclude test_*
```

## 🚀 Advanced Usage

### Depth Control
```bash
fd -d 1                # Only current directory (depth 1)
fd --max-depth 3       # Max 3 levels deep
fd --min-depth 2       # Skip current directory
```

### Execute Commands
```bash
fd -e txt -x cat {}    # Cat each .txt file
fd -e py -x wc -l      # Line count for all .py files
fd -e sh -x chmod +x   # Make all .sh files executable

# With placeholder
fd test -x echo "Found: {}"
fd -e jpg -x convert {} {.}.png  # {.} = filename without extension
```

### Size Filters
```bash
fd --size +100k        # Files larger than 100KB
fd --size -1m          # Files smaller than 1MB
fd --size +1g          # Files larger than 1GB
```

### Time Filters
```bash
fd --changed-within 1h      # Modified in last hour
fd --changed-within 7d      # Modified in last 7 days
fd --changed-before 2023-01-01  # Before date
```

### Search by Owner
```bash
fd --owner root        # Files owned by root
fd --owner $USER       # Files owned by current user
```

## 💡 Common Use Cases

### Find and Delete
```bash
fd -e tmp -x rm        # Delete all .tmp files
fd node_modules -t d -x rm -rf  # Delete all node_modules dirs
```

### Find and Edit
```bash
fd config -e yml -x vim  # Open all config.yml files in vim
fd TODO -t f -x code     # Open all files named TODO in VS Code
```

### Count Files
```bash
fd -e py | wc -l       # Count Python files
fd -t f | wc -l        # Count all files
```

### Find Recently Modified
```bash
fd --changed-within 1d # Files changed today
fd --changed-within 2h -e log  # Recent log files
```

### Search Full Path (not just filename)
```bash
fd -p tests.*\.py$     # Match path: tests/test_file.py
fd --full-path 'src/.*config'  # Match path containing pattern
```

## ⚠️ Common Gotchas

1. **Uses regex by default, NOT globs**
   - `fd *.txt` ❌ (breaks, * is regex repetition)
   - `fd '*.txt' -g` ✅ (use --glob for shell patterns)
   - `fd txt` ✅ (regex matches substring)

2. **Hidden files ignored by default**
   - Need `-H` to see dotfiles

3. **Respects .gitignore**
   - Use `-I` to see ignored files
   - Use `-HI` to see everything

4. **Smart case can surprise you**
   - `fd test` finds Test.txt
   - `fd Test` doesn't find test.txt

5. **Extension flag has no dot**
   - `fd -e txt` ✅
   - `fd -e .txt` ❌

## 🆚 Quick Comparison: find vs fd

```bash
# Find all .txt files
find . -name "*.txt"          # find
fd -e txt                     # fd

# Find directories
find . -type d -name config   # find
fd -t d config                # fd

# Case-insensitive search
find . -iname "test*"         # find
fd -i test                    # fd

# Exclude directory
find . -name node_modules -prune -o -name "*.js" -print  # find
fd -e js -E node_modules      # fd (much simpler!)

# Execute command on results
find . -name "*.txt" -exec cat {} \;   # find
fd -e txt -x cat                       # fd
```

## 🔗 Useful Combinations

```bash
# Find large log files from last week
fd -e log --size +10m --changed-within 7d

# Find executable scripts in PATH
fd -t x -t f . /usr/local/bin

# Find config files including hidden ones
fd -H -e yml -e yaml -e json config

# Find all files except test files
fd -t f -E '*test*' -E '__pycache__'

# Search multiple directories
fd config ~/.config ~/projects

# Find broken symlinks
fd -t l -L -x test -e {} \; -print  # Advanced, checks if target exists
```

## 📚 Tips & Best Practices

1. **Use short flags for common tasks**: `-HI` instead of `--hidden --no-ignore`
2. **Combine with other tools**: `fd -e py | xargs grep "TODO"`
3. **Use `.fdignore` file**: Like `.gitignore` but specifically for fd
4. **Set default options**: Add to shell config: `export FD_OPTIONS="--hidden --follow"`
5. **Use `-0` for null-separated output**: `fd -0 | xargs -0 command` (safe for spaces)

## 🛠 Installation

```bash
# Ubuntu/Debian
apt install fd-find

# macOS
brew install fd

# Arch
pacman -S fd

# Cargo
cargo install fd-find
```

## 📖 More Help

```bash
fd --help          # Quick help
fd --help | less   # Full manual
man fd             # Man page (if available)
```

---

**Note**: On some systems (like Ubuntu), the command is `fdfind` instead of `fd` to avoid conflicts.
