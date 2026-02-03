# Bash Pattern Matching and Text Processing: A Complete Beginner's Guide

---

## Introduction: Why Text Processing Matters in Linux

You've probably heard the phrase **"everything in Linux is a file."** This isn't just a philosophy—it's literally how Linux works. System logs, configuration files, datasets, network streams, and even hardware devices are represented as files containing text.

This means that **manipulating text is one of the most important skills** you can develop as a Linux user. Instead of clicking through GUIs, you'll:

- Search through thousands of log lines for error messages
- Extract specific columns from CSV files
- Transform configuration files automatically
- Analyze data from command output

### The Unix Philosophy: Small Tools, Big Power

Linux doesn't give you one massive program that does everything. Instead, it provides **small, focused tools** that each do one thing well. You combine them using **pipes** (`|`) to build powerful workflows.

```
Tool A → | → Tool B → | → Tool C → Final Output
```

Each tool:
1. Reads from **standard input** (stdin)
2. Transforms the text
3. Writes to **standard output** (stdout)

This is called a **pipeline**, and it's the foundation of Linux text processing.

---

## Part 1: Bash Wildcards (Globbing)

### What is Globbing?

**Globbing** is the shell's way of expanding patterns into matching filenames. When you type `*.txt`, the shell finds all files ending in `.txt` and replaces your pattern with the actual filenames before the command runs.

### Why Does Globbing Exist?

Without globbing, you'd have to type every filename individually:
```bash
# Without globbing (tedious!)
rm file1.txt file2.txt file3.txt file4.txt file5.txt

# With globbing (easy!)
rm *.txt
```

### The Wildcards

#### The Asterisk (`*`) — Zero or More Characters

The `*` matches **any number of characters** (including zero).

| Pattern | Matches | Does NOT Match |
|---------|---------|----------------|
| `*.txt` | `file.txt`, `a.txt`, `.txt` | `file.csv`, `txt` |
| `project*` | `project`, `project1`, `project_final` | `my_project` |
| `*data*` | `data`, `mydata`, `data.csv`, `old_data_backup` | (matches almost anything with "data") |

**Practical Examples:**

```bash
# List all Python files
ls *.py

# Delete all backup files
rm *.bak

# Copy all images to a folder
cp *.jpg *.png images/

# List files starting with "report"
ls report*
```

#### The Question Mark (`?`) — Exactly One Character

The `?` matches **exactly one character**—no more, no less.

| Pattern | Matches | Does NOT Match |
|---------|---------|----------------|
| `file?.txt` | `file1.txt`, `fileA.txt` | `file.txt`, `file10.txt` |
| `???.log` | `app.log`, `sys.log` | `a.log`, `application.log` |

**Practical Examples:**

```bash
# List files like file1.txt, file2.txt, but not file10.txt
ls file?.txt

# Match 3-letter filenames
ls ???

# Match files with single-digit numbers
ls report?.pdf
```

#### Square Brackets (`[]`) — Character Sets

Square brackets match **any one character** from the set inside.

| Pattern | Matches | Does NOT Match |
|---------|---------|----------------|
| `file[123].txt` | `file1.txt`, `file2.txt`, `file3.txt` | `file4.txt`, `file12.txt` |
| `[abc].txt` | `a.txt`, `b.txt`, `c.txt` | `d.txt`, `ab.txt` |
| `[a-z].txt` | Any single lowercase letter `.txt` | `A.txt`, `1.txt` |
| `[A-Za-z0-9]` | Any single alphanumeric character | `_`, `-`, `.` |

**Special syntax inside brackets:**

| Syntax | Meaning |
|--------|---------|
| `[a-z]` | Range: any character from a to z |
| `[0-9]` | Range: any digit |
| `[!abc]` or `[^abc]` | Negation: any character EXCEPT a, b, or c |

**Practical Examples:**

```bash
# List image1.png through image5.png
ls image[1-5].png

# List files NOT starting with a vowel
ls [!aeiou]*

# Match uppercase or lowercase
ls [Rr]eport.txt
```

#### The Double Asterisk (`**`) — Recursive Matching

The `**` matches **zero or more directories**. It "dives" into subdirectories.

| Pattern | Matches |
|---------|---------|
| `**/*.txt` | `file.txt`, `dir/file.txt`, `dir/subdir/file.txt` |
| `**/config` | `config`, `app/config`, `app/settings/config` |

**⚠️ Important: Must Be Enabled First!**

The `**` glob is disabled by default in Bash. Enable it with:

```bash
shopt -s globstar
```

To make this permanent, add it to your `~/.bashrc` file:

```bash
echo 'shopt -s globstar' >> ~/.bashrc
```

**Practical Examples:**

```bash
# Enable globstar first
shopt -s globstar

# Find all Python files in current directory and all subdirectories
ls **/*.py

# Count lines in all JavaScript files recursively
wc -l **/*.js

# Find all README files anywhere
ls **/README*
```

### What My Professor Didn't Explain

- **Globbing happens BEFORE the command runs.** The shell expands `*.txt` into `file1.txt file2.txt file3.txt`, then passes those to `ls`. The `ls` command never sees `*.txt`.

- **If no files match, behavior varies:**
  - By default, the pattern is passed literally (e.g., `ls *.xyz` shows `*.xyz` if no matches)
  - With `shopt -s nullglob`, unmatched patterns expand to nothing
  - With `shopt -s failglob`, unmatched patterns cause an error

- **Hidden files (starting with `.`) are NOT matched by `*`** unless you explicitly include the dot:
  ```bash
  ls .*        # Hidden files only
  ls * .*      # All files including hidden
  ```

- **Glob patterns are NOT regular expressions.** They're simpler and have different syntax.

### ⚠️ Gotcha: Test Before You Delete!

Always preview what a glob will match before using destructive commands:

```bash
# GOOD: See what would be deleted first
ls *.bak
rm *.bak

# DANGEROUS: Delete without checking
rm *       # Deletes everything!
```

---

## Part 2: Linux Text Processing Tools

### Setting Up Practice Files

Create a directory and save these files to follow along:

```bash
mkdir -p ~/wk3
cd ~/wk3
```

**File 1: `dc.csv`** (DC Comics characters)
```csv
alias,name,age,city,occupation,species
batman,"Bruce Wayne",54,gotham,none,human
superman,"Clark Kent",43,metropolis,reporter,kryptonian
supergirl,"Kara Danvers",28,"national city",reporter,kryptonian
raven,"Rachel Roth",27,"washington dc",teacher,half-demon
icon,"Augustus Freeman",205,dakota,lawyer,terminan
blue beetle,"Jaime Reyes",21,"palmera city",student,Human
wonder woman,"Diana Prince",5000,"washington dc",nurse,demigod
martian manhunter,"John Jones",350,"national city",detective,martian
zatana,"zatana zatara",37,"Washington dc",magician,human
orphan,"cassandra cain",21,gotham,none,human
sandman,"wesley dodds",86,"new york","investment banker",human
flash,"Barry Allen",34,"central city",forensic scientist,human
green lantern,"John Stewart",47,"detroit",architect,human
aquaman,"Arthur Curry",38,"amnesty bay",king,atlantean
cyborg,"Victor Stone",25,detroit,engineer,cyborg
nightwing,"Dick Grayson",29,bludhaven,detective,human
starfire,"Koriand'r",31,"jump city",hero,tamaranean
robin,"Damian Wayne",15,gotham,student,human
harley quinn,"Harleen Quinzel",32,gotham,psychiatrist,human
deathstroke,"Slade Wilson",55,"unknown",mercenary,enhanced human
```

**File 2: `log`** (Web server access log)
```
2026-01-15 09:12:01 INFO  192.168.1.10 GET /api/login 200 120ms
2026-01-15 09:12:03 WARN  192.168.1.23 POST /api/login 401 95ms
2026-01-15 09:12:04 INFO  192.168.1.10 GET /api/profile 200 45ms
2026-01-15 09:12:07 INFO  192.168.1.15 GET /api/products 200 310ms
2026-01-15 09:12:09 ERROR 192.168.1.23 POST /api/login 500 820ms
2026-01-15 09:12:10 INFO  192.168.1.18 GET /api/products 200 290ms
2026-01-15 09:12:11 INFO  192.168.1.10 POST /api/logout 200 30ms
2026-01-15 09:12:14 WARN  192.168.1.42 GET /api/products 404 15ms
2026-01-15 09:12:18 INFO  192.168.1.15 GET /api/cart 200 110ms
2026-01-15 09:12:22 INFO  192.168.1.18 POST /api/cart 200 220ms
2026-01-15 09:12:25 ERROR 192.168.1.99 GET /api/products 503 1500ms
2026-01-15 09:12:30 INFO  192.168.1.10 GET /api/profile 200 50ms
2026-01-15 09:12:33 WARN  192.168.1.23 GET /api/profile 403 20ms
2026-01-15 09:12:36 INFO  192.168.1.15 POST /api/order 201 450ms
2026-01-15 09:12:40 INFO  192.168.1.18 GET /api/order 200 130ms
2026-01-15 09:12:45 ERROR 192.168.1.42 POST /api/order 500 980ms
2026-01-15 09:12:48 INFO  192.168.1.10 GET /api/products 200 300ms
2026-01-15 09:12:52 INFO  192.168.1.15 GET /api/products 200 280ms
2026-01-15 09:12:55 WARN  192.168.1.99 GET /api/login 429 10ms
2026-01-15 09:12:59 INFO  192.168.1.18 GET /api/health 200 5ms
```

**File 3: `config`** (SSH configuration)
```
# Global SSH configuration
Host *
    ForwardAgent no
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes

# Production server
Host prod-server
    HostName prod.example.com
    User deploy
    Port 22
    IdentityFile ~/.ssh/id_rsa_prod

# Staging environment
Host staging-server
    HostName staging.example.com
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_rsa_staging

# Legacy server (to be retired)
Host legacy-server
    HostName 192.168.10.50
    User admin
    Port 22
    PubkeyAuthentication no
```

---

### `cut` — Extract Columns from Text

#### Purpose

`cut` extracts specific **columns** (fields) or **character positions** from each line of text. It's perfect for structured data like CSV files or log files.

#### Why It Exists

When you have a file with columns of data, you often only need certain columns. `cut` lets you slice out exactly what you need without loading the file into a spreadsheet.

#### Syntax

```bash
cut [options] <file>
```

#### Key Options

| Option | Meaning | Example |
|--------|---------|---------|
| `-d` | **Delimiter** — what separates fields | `-d','` (comma), `-d' '` (space) |
| `-f` | **Fields** — which columns to extract | `-f1`, `-f2,3`, `-f1-5` |
| `-c` | **Characters** — extract by character position | `-c1-10` |
| `-b` | **Bytes** — extract by byte position | `-b1-4` |

#### Practical Examples

**Extract the names column (field 2) from the CSV:**
```bash
cut -d',' -f2 dc.csv
```

**Output:**
```
name
"Bruce Wayne"
"Clark Kent"
"Kara Danvers"
...
```

**Remove the header row using `tail`:**
```bash
cut -d',' -f2 dc.csv | tail -n +2
```

The `tail -n +2` means "start from line 2" (skip line 1).

**Extract multiple fields (names and ages):**
```bash
cut -d',' -f2,3 dc.csv
```

**Output:**
```
name,age
"Bruce Wayne",54
"Clark Kent",43
...
```

**Extract IP addresses from the log file:**
```bash
cut -d' ' -f4 log
```

**Output:**
```
192.168.1.10
192.168.1.23
192.168.1.10
...
```

**Extract the year (first 4 characters) from each line:**
```bash
cut -c1-4 log
```

**Output:**
```
2026
2026
2026
...
```

#### Understanding `-c` vs `-b`

| Option | Counts | Best For |
|--------|--------|----------|
| `-c` | Characters | Human-readable text, Unicode |
| `-b` | Bytes | Binary data, fixed-width formats |

**Why this matters with Unicode:**
```bash
echo "café" | cut -c1-4    # Output: café (4 characters)
echo "café" | cut -b1-4    # Output: caf� (4 bytes, but é is 2 bytes!)
```

The `é` character takes 2 bytes in UTF-8, so `-b1-4` cuts it in half, producing garbage.

**Rule of thumb:**
- Use `-c` for text you'll read
- Use `-b` for binary/fixed-width data

#### What My Professor Didn't Explain

- **`cut` cannot handle quoted CSV fields properly.** If a field contains the delimiter inside quotes (like `"Washington, DC"`), `cut` will split it incorrectly. For complex CSVs, use `awk` or a proper CSV parser.

- **Field numbers start at 1**, not 0.

- **You can specify ranges:**
  ```bash
  cut -f1-3 file    # Fields 1, 2, and 3
  cut -f3- file     # Field 3 to the end
  cut -f-3 file     # Beginning to field 3
  ```

#### Common Combinations

```bash
# Extract column, sort, remove duplicates
cut -d',' -f4 dc.csv | sort | uniq

# Count occurrences of each city
cut -d',' -f4 dc.csv | sort | uniq -c | sort -rn
```

---

### `sed` — Stream Editor

#### Purpose

`sed` performs **text transformations** on a stream of text. It can search and replace, delete lines, insert text, and much more—all without opening a text editor.

#### Why It Exists

Imagine you need to change a configuration value in 100 files, or replace every occurrence of "http://" with "https://" in a codebase. Opening each file manually would take forever. `sed` automates these transformations.

#### The Name

"sed" stands for **S**tream **ED**itor. It reads text line by line, applies transformations, and outputs the result.

#### Basic Syntax

```bash
sed 'command' file
```

The most common command is **substitution**:

```bash
sed 's/old/new/' file       # Replace first occurrence per line
sed 's/old/new/g' file      # Replace ALL occurrences (global)
```

#### Anatomy of a Substitution Command

```
s/pattern/replacement/flags
│ │       │           │
│ │       │           └── g = global (all matches), i = case-insensitive
│ │       └── What to replace it with
│ └── What to search for (can be regex)
└── s = substitute command
```

#### Key Options

| Option | Meaning |
|--------|---------|
| `-i` | Edit file **in-place** (modify the original file) |
| `-E` | Use **extended** regular expressions (more portable) |
| `-n` | **Suppress** automatic printing (use with `p` command) |
| `-f` | Read commands from a **file** |

#### Practical Examples

**Simple replacement (output to screen):**
```bash
sed 's/human/HUMAN/' dc.csv
```

This replaces the first "human" on each line with "HUMAN".

**Global replacement (all occurrences):**
```bash
sed 's/human/HUMAN/g' dc.csv
```

**Case-insensitive replacement:**
```bash
sed 's/human/HUMAN/gi' dc.csv
```

This matches "human", "Human", "HUMAN", etc.

**Save to a new file:**
```bash
sed 's/human/HUMAN/g' dc.csv > dc_modified.csv
```

**Edit the original file in-place:**
```bash
sed -i 's/human/HUMAN/g' dc.csv
```

⚠️ **Warning:** `-i` modifies the file permanently. There's no undo!

**Real-world example: Update Debian version**

You've already used this! When upgrading Debian:
```bash
sudo sed -i 's/trixie/forky/g' /etc/apt/sources.list
```

This replaces every "trixie" with "forky" in the sources list.

#### Advanced Examples

**Convert quoted strings to lowercase:**
```bash
sed 's/\("[^"]*"\)/\L\1/g' dc.csv > dc_lowercase.csv
```

Let's break this down:

| Part | Meaning |
|------|---------|
| `s/` | Start substitution |
| `\(` | Start a capture group |
| `"` | Match a literal quote |
| `[^"]*` | Match any characters that aren't quotes |
| `"` | Match the closing quote |
| `\)` | End the capture group |
| `/\L\1/` | Replace with lowercase (`\L`) version of group 1 (`\1`) |
| `g` | Global (all matches on each line) |

**Change a configuration value:**
```bash
sed -i 's/^    ServerAliveInterval 60/    ServerAliveInterval 120/' config
```

The `^` means "start of line" (after the spaces).

**Mask IP addresses in logs:**
```bash
sed -E 's/([0-9]+\.[0-9]+\.[0-9]+)\.[0-9]+/\1.XXX/g' log
```

This captures the first three octets and replaces the fourth with "XXX":
- `192.168.1.10` → `192.168.1.XXX`

**Delete lines containing a pattern:**
```bash
sed '/ERROR/d' log
```

The `d` command deletes matching lines.

**Print only specific lines:**
```bash
sed -n '5,10p' log
```

- `-n` suppresses automatic printing
- `5,10p` prints lines 5 through 10

#### Using sed Script Files

For complex transformations, store commands in a file:

**cleanup.sed:**
```sed
# Remove DEBUG lines
/^DEBUG/d

# Capitalize "warning"
s/warning/WARNING/g

# Replace multiple spaces with comma
s/  */,/g

# Add prefix to every line
s/^/PROCESSED: /
```

**Run the script:**
```bash
sed -f cleanup.sed input.txt
```

#### What My Professor Didn't Explain

- **Delimiters can be changed.** If your pattern contains `/`, use a different delimiter:
  ```bash
  sed 's|/usr/local|/opt|g' file    # Using | instead of /
  sed 's#http://#https://#g' file   # Using #
  ```

- **`-i` works differently on macOS.** On macOS, you need:
  ```bash
  sed -i '' 's/old/new/g' file      # Empty string after -i
  ```

- **sed processes line by line.** It can't easily match patterns spanning multiple lines (use `awk` or `perl` for that).

- **Backup before in-place editing:**
  ```bash
  sed -i.bak 's/old/new/g' file     # Creates file.bak backup
  ```

#### Common sed Commands Summary

| Command | Meaning | Example |
|---------|---------|---------|
| `s/old/new/` | Substitute | `sed 's/foo/bar/'` |
| `d` | Delete line | `sed '/pattern/d'` |
| `p` | Print line | `sed -n '/pattern/p'` |
| `i\text` | Insert before | `sed '1i\Header'` |
| `a\text` | Append after | `sed '$a\Footer'` |

---

### `grep` — Search for Patterns

#### Purpose

`grep` searches for **patterns** in files or input streams and prints matching lines. It's one of the most frequently used Linux commands.

#### Why It Exists

Finding specific information in large files or command output is a constant need:
- Find error messages in logs
- Search for function calls in code
- Filter command output

#### The Name

"grep" stands for **G**lobally search for a **R**egular **E**xpression and **P**rint matching lines.

#### Basic Syntax

```bash
grep [options] 'pattern' file(s)
```

#### Key Options

| Option | Meaning |
|--------|---------|
| `-i` | **Case-insensitive** matching |
| `-v` | **Invert** match (show non-matching lines) |
| `-n` | Show **line numbers** |
| `-c` | **Count** matching lines |
| `-r` | **Recursive** (search directories) |
| `-l` | Show only **filenames** with matches |
| `-E` | **Extended** regex (same as `egrep`) |
| `-w` | Match whole **words** only |
| `-A n` | Show n lines **after** match |
| `-B n` | Show n lines **before** match |
| `-C n` | Show n lines of **context** (before and after) |

#### Practical Examples

**Find all ERROR lines in the log:**
```bash
grep 'ERROR' log
```

**Output:**
```
2026-01-15 09:12:09 ERROR 192.168.1.23 POST /api/login 500 820ms
2026-01-15 09:12:25 ERROR 192.168.1.99 GET /api/products 503 1500ms
2026-01-15 09:12:45 ERROR 192.168.1.42 POST /api/order 500 980ms
```

**Case-insensitive search:**
```bash
grep -i 'error' log
```

**Show line numbers:**
```bash
grep -n 'ERROR' log
```

**Output:**
```
5:2026-01-15 09:12:09 ERROR 192.168.1.23 POST /api/login 500 820ms
11:2026-01-15 09:12:25 ERROR 192.168.1.99 GET /api/products 503 1500ms
16:2026-01-15 09:12:45 ERROR 192.168.1.42 POST /api/order 500 980ms
```

**Count matches:**
```bash
grep -c 'ERROR' log
```

**Output:** `3`

**Invert match (show lines WITHOUT "INFO"):**
```bash
grep -v 'INFO' log
```

**Search recursively in a directory:**
```bash
grep -r 'TODO' ~/projects/
```

**Show only filenames containing matches:**
```bash
grep -rl 'password' /etc/
```

**Show context around matches:**
```bash
grep -C 2 'ERROR' log    # 2 lines before and after each match
```

#### Using grep with Pipes

`grep` is often used to filter output from other commands:

```bash
# Find running Python processes
ps aux | grep python

# Find your IP address
ip addr | grep inet

# Find a specific package
dpkg -l | grep nginx
```

#### Regular Expressions in grep

`grep` supports regular expressions for powerful pattern matching:

| Pattern | Meaning | Example |
|---------|---------|---------|
| `.` | Any single character | `h.t` matches "hat", "hot", "hit" |
| `*` | Zero or more of previous | `ab*c` matches "ac", "abc", "abbc" |
| `^` | Start of line | `^Error` matches lines starting with "Error" |
| `$` | End of line | `\.txt$` matches lines ending with ".txt" |
| `[abc]` | Any character in set | `[aeiou]` matches vowels |
| `[^abc]` | Any character NOT in set | `[^0-9]` matches non-digits |
| `\` | Escape special character | `\.` matches a literal dot |

**Extended regex (`-E` or `egrep`):**

| Pattern | Meaning |
|---------|---------|
| `+` | One or more of previous |
| `?` | Zero or one of previous |
| `\|` | OR (alternation) |
| `()` | Grouping |

**Examples:**

```bash
# Lines starting with a date
grep '^2026' log

# Lines ending with "ms"
grep 'ms$' log

# Find 500 or 503 errors
grep -E '50[03]' log

# Find IP addresses
grep -E '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' log

# Find "human" or "Human"
grep -i 'human' dc.csv
```

#### What My Professor Didn't Explain

- **`grep` vs `fgrep` vs `egrep`:**
  - `grep` — basic regex
  - `egrep` (or `grep -E`) — extended regex
  - `fgrep` (or `grep -F`) — fixed strings (no regex, faster)

- **Colorized output** is usually enabled by default via alias. To force it:
  ```bash
  grep --color=always 'pattern' file
  ```

- **Exclude directories when searching recursively:**
  ```bash
  grep -r --exclude-dir=node_modules 'pattern' .
  ```

- **The "grep yourself" problem:** When you `ps aux | grep python`, grep itself shows up in the results. Fix:
  ```bash
  ps aux | grep [p]ython    # The [p] trick
  ps aux | grep python | grep -v grep
  ```

---

### `awk` — Pattern Scanning and Processing Language

#### Purpose

`awk` is a **programming language** designed for text processing. It's more powerful than `cut` and `sed` because it can perform calculations, use variables, and apply complex logic.

#### Why It Exists

Sometimes you need more than simple extraction or substitution:
- Calculate averages from data
- Apply conditions (if age > 100, print name)
- Reformat output with custom logic

`awk` fills the gap between simple tools and full programming languages.

#### The Name

Named after its creators: **A**ho, **W**einberger, and **K**ernighan.

#### Basic Syntax

```bash
awk 'pattern { action }' file
```

- **pattern** — when to apply the action (optional)
- **action** — what to do (print, calculate, etc.)

#### Key Concepts

| Concept | Meaning | Example |
|---------|---------|---------|
| `$0` | The entire line | `{ print $0 }` |
| `$1, $2, ...` | Fields (columns) | `{ print $1 }` |
| `NR` | Current line number | `NR > 1` (skip header) |
| `NF` | Number of fields | `$NF` (last field) |
| `-F` | Field separator | `-F','` for CSV |
| `BEGIN` | Run before processing | Setup code |
| `END` | Run after processing | Summary code |

#### Practical Examples

**Print the second column (names) from CSV, skipping header:**
```bash
awk -F',' 'NR > 1 { print $2 }' dc.csv
```

- `-F','` — use comma as delimiter
- `NR > 1` — only lines where line number > 1 (skip header)
- `{ print $2 }` — print the second field

**Find characters older than 100:**
```bash
awk -F',' 'NR > 1 && $3 > 100 { print $2, "(" $3 ")" }' dc.csv
```

**Output:**
```
"Augustus Freeman" ( 205 )
"Diana Prince" ( 5000 )
"John Jones" ( 350 )
```

**Find slow requests (response time > 500ms):**
```bash
awk '$NF+0 > 500 { print $1, $2, $6, $7, $NF }' log
```

- `$NF` — last field (the response time like "820ms")
- `$NF+0` — convert to number (strips "ms")

**Count log entries by HTTP status code:**
```bash
awk '{ count[$7]++ } END { for (c in count) print c, count[c] }' log
```

This creates an associative array counting occurrences of each status code.

**Output:**
```
200 12
201 1
401 1
403 1
404 1
429 1
500 2
503 1
```

**Increment Wonder Woman's age:**
```bash
awk -F',' 'BEGIN { OFS="," }
NR==1 { print; next }
$1=="wonder woman" { $3 = $3 + 1 }
{ print }
' dc.csv
```

- `OFS=","` — Output Field Separator (keep commas)
- `NR==1 { print; next }` — Print header, skip to next line
- `$1=="wonder woman"` — Match the alias
- `$3 = $3 + 1` — Increment age

**Show process memory usage in MB:**
```bash
ps -eo comm,rss | awk 'NR>1 { $2=sprintf("%.2f MB", $2/1024); print }' | column -t
```

#### awk Script Files

For complex processing, use a script file:

**process_scores.awk:**
```awk
BEGIN {
    FS = ","
    print "--- GRADE REPORT ---"
    print "NAME\t\tSTATUS"
    print "--------------------"
}

$2 > 0 {
    total += $2
    count++
    
    if ($2 >= 60) {
        status = "Pass"
    } else {
        status = "FAIL"
    }
    
    printf "%-10s\t%s\n", $1, status
}

END {
    print "--------------------"
    if (count > 0) {
        printf "Class Average: %.2f\n", total / count
    }
}
```

**students.csv:**
```
Hannah,95
Mo,52
Tomoa,88
Sid,40
Raj,33
Janja,91
```

**Run it:**
```bash
awk -f process_scores.awk students.csv
```

**Output:**
```
--- GRADE REPORT ---
NAME		STATUS
--------------------
Hannah    	Pass
Mo        	FAIL
Tomoa     	Pass
Sid       	FAIL
Raj       	FAIL
Janja     	Pass
--------------------
Class Average: 66.50
```

#### What My Professor Didn't Explain

- **Different awk implementations exist:**
  - `mawk` — fast, minimal (default on Debian)
  - `gawk` — GNU awk, most features
  - `nawk` — "new" awk

- **awk handles quoted CSV fields better than cut**, but still not perfectly. For truly complex CSVs, use Python or a dedicated CSV tool.

- **awk is Turing-complete** — you can write entire programs in it!

> **Note:** AWK won't be on the midterm or final exam, but it's incredibly useful for real-world text processing.

---

### Perl — The Swiss Army Chainsaw

#### What It Is

**Perl** is a full programming language created in 1987, famous for its text processing capabilities. It combines features of `sed`, `awk`, and shell scripting into one powerful tool.

#### Why It's Mentioned

Perl was the dominant scripting language for system administration and web development in the 1990s-2000s. While Python has largely replaced it for new projects, you'll still encounter Perl scripts in:
- Legacy systems
- System administration tools
- One-liners for complex text processing

#### Quick Example

```bash
# Replace text like sed, but with more power
perl -pe 's/old/new/g' file

# In-place editing with backup
perl -i.bak -pe 's/old/new/g' file
```

> **Note:** Perl won't be on the midterm or final exam.

---

## Finding Files: `find` and `grep`

Two essential tools for locating files:

| Tool | Searches By | Example |
|------|-------------|---------|
| `find` | File **metadata** (name, size, date, permissions) | `find . -name "*.py"` |
| `grep` | File **contents** (text patterns) | `grep -r "TODO" .` |

### Quick `find` Examples

```bash
# Find all Python files
find . -name "*.py"

# Find files modified in the last hour
find . -mmin -60

# Find files larger than 100MB
find . -size +100M

# Find and delete all .tmp files
find . -name "*.tmp" -delete
```

### Combining find and grep

```bash
# Find Python files containing "import requests"
find . -name "*.py" -exec grep -l "import requests" {} \;
```

---

## Tool Comparison Summary

| Task | Best Tool | Example |
|------|-----------|---------|
| Extract columns from CSV | `cut` | `cut -d',' -f2 file.csv` |
| Simple search/replace | `sed` | `sed 's/old/new/g' file` |
| Search for patterns | `grep` | `grep 'ERROR' log` |
| Complex logic/calculations | `awk` | `awk '$3 > 100 { print $1 }'` |
| Find files by name/date/size | `find` | `find . -name "*.log"` |

---

## Quick Reference Card

### Globbing
```bash
*           # Zero or more characters
?           # Exactly one character
[abc]       # One of a, b, or c
[a-z]       # Range: a through z
[!abc]      # NOT a, b, or c
**          # Recursive (needs: shopt -s globstar)
```

### cut
```bash
cut -d',' -f2 file      # Field 2, comma-delimited
cut -d',' -f1,3 file    # Fields 1 and 3
cut -c1-10 file         # Characters 1-10
```

### sed
```bash
sed 's/old/new/' file       # Replace first per line
sed 's/old/new/g' file      # Replace all
sed -i 's/old/new/g' file   # Edit in place
sed '/pattern/d' file       # Delete matching lines
sed -n '5,10p' file         # Print lines 5-10
```

### grep
```bash
grep 'pattern' file         # Find pattern
grep -i 'pattern' file      # Case-insensitive
grep -v 'pattern' file      # Invert (non-matching)
grep -n 'pattern' file      # Show line numbers
grep -r 'pattern' dir/      # Recursive search
grep -c 'pattern' file      # Count matches
```

### awk
```bash
awk '{ print $1 }' file                 # Print first field
awk -F',' '{ print $2 }' file           # CSV second field
awk 'NR > 1 { print }' file             # Skip header
awk '$3 > 100 { print $1 }' file        # Conditional
```

---

## Additional Resources

- [Bash Reference Manual: Pattern Matching](https://www.gnu.org/software/bash/manual/bash.html#Pattern-Matching-1)
- [GNU sed Manual](https://www.gnu.org/software/sed/manual/sed.html)
- [Effective AWK Programming](https://www.gnu.org/software/gawk/manual/gawk.html)
- [DigitalOcean: Introduction to Regular Expressions](https://www.digitalocean.com/community/tutorials/an-introduction-to-regular-expressions)
- [Regular Expressions Cheat Sheet](https://cheatography.com/davechild/cheat-sheets/regular-expressions/)
- [ExplainShell](https://explainshell.com/) — Paste any command to see what each part does
