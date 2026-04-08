## System Maintenance & Debugging: Mastering diff and patch
### Overview
- When collaborating on scripts or debugging system automation tools, comparing files side-by-side with the naked eye is error-prone and inefficient. This repository demonstrates how to use command-line tools like diff and patch to automatically identify file differences, generate context-rich patch files, and apply those fixes seamlessly.
- While modern Version Control Systems (like Git) handle much of this under the hood, understanding diff and patch is a fundamental skill for any DevOps or Software Engineer working in Linux environments.

### 1. Comparing Files with `diff`
- The diff command compares two files line by line and outputs only the differences.

### Standard `diff`
- Let's look at two versions of a script (`rearrange1.py` and `rearrange2.py`).
```bash
cat rearrange1.py
```

```bash
#!/usr/bin/env python3
import re

def rearrange_name(name):
    result = re.search(r"^([\w.]*), ([\w.]*)$", name)
    if result == None:
        return name
    return "{} {}".format(result[2], result[1])
```

- When we run the `diff` command:
```bash
diff rearrange1.py rearrange2.py
```
- Output:
```bash
6c6
<     result = re.search(r"^([\w.]*), ([\w.]*)$", name)
---
>     result = re.search(r"^([\w.-]*), ([\w.-]*)$", name)
```

- `<` indicates a line removed from the first file.

- `>` indicates a line added to the second file.

- `6c6` means line 6 was changed.

### Unified Format: diff -u
- Standard `diff` can be hard to read without context. Using the `-u` (unified) flag provides surrounding lines to help you understand exactly where the change occurred.
```bash
diff -u validations1.py validations2.py
```
- Output:
```bash
--- validations1.py 2019-06-06 14:28:49.639 +0200
+++ validations2.py 2019-06-06 14:30:48.019 +0200
@@ -2,7 +2,8 @@
 
 def validate_user(username, minlen):
-    assert type(username) == str, "username must be a string"
+    if type(username) != str:
+        raise TypeError("username must be a string")
     if minlen < 1:
         raise ValueError("minlen must be at least 1")
@@ -10,5 +11,8 @@
         return False
     if not username.isalnum():
         return False
+    # Usernames can't begin with a number
+    if username[0].isnumeric():
+        return False
     return True
```

### 2. Generating and Applying Patches
- Instead of telling a colleague to manually change lines of code, you can send them a diff file (or patch file). They can then use the patch command to apply your changes automatically.

### Step 1: Create the Patch File
- Redirect the output of the unified diff into a .diff file.
```bash
diff -u cpu_usage.py cpu_usage_fixed.py > cpu_usage.diff
```

### Step 2: Apply the Patch
- Use the patch command, redirecting the .diff file via standard input (<) into the broken file.
```bash
patch cpu_usage.py < cpu_usage.diff
```
- Output:
```bash
patching file cpu_usage.py
```

### 3. Real-World Troubleshooting Scenario: disk_usage.py
- The Problem
- A colleague wrote a script to check for free disk space, but it throws a SyntaxError and miscalculates gigabytes.

- Original Buggy Code (disk_usage.py):
```bash
#!/usr/bin/env python3
import shutil

def check_disk_usage(disk, min_absolute, min_percent):
    """Returns True if there is enough free disk space, false otherwise."""
    du = shutil.disk_usage(disk)
    percent_free = 100 * du.free / du.total
    gigabytes_free = du.free / 2**30
    if percent_free < min_percent or gigabytes_free < min_absolute:
        return False
    return True

# Check for at least 2 GB and 10% free
if not check_disk_usage("/", 2*2**30, 10):
    print("ERROR: Not enough disk space")
    return 1  # BUG 1: Cannot return outside of a function

print("Everything ok")
return 0      # BUG 1: Cannot return outside of a function
```

### The Fixes
- Syntax Fix: Replaced return outside of a function with sys.exit() and imported the sys module.
- Logic Fix: The script was doing gigabyte conversion twice. Changed the function call parameter from 2*2**30 to just 2.

### Generating the Fix & Patching:
```bash
# 1. Create a copy to work on
cp disk_usage.py disk_usage_fixed.py

# (Edit disk_usage_fixed.py in your preferred editor like nano or vim)

# 2. Test the fixed script
./disk_usage_fixed.py
# Output: Everything ok

# 3. Generate the patch file
diff -u disk_usage_original.py disk_usage_fixed.py > disk_usage.diff

# 4. View the patch file
cat disk_usage.diff

# 5. Apply the patch to the original file
patch disk_usage.py < disk_usage.diff

# 6. Verify the fix
./disk_usage.py
# Output: Everything ok
```

---


## Git Fundamentals: Lifecycle, Configuration, and Best Practices
### Overview
- This section covers the core mechanics of Git. It explains how to initialize a repository, the lifecycle of a file within Git (Working Tree ➔ Staging Area ➔ Git Directory), and the industry standards for writing clean, professional commit messages.

### 1. Initial Configuration
- Before tracking any files, Git needs to know who is making the changes. This information is attached to every commit you make.
```bash
# Set your global username and email
git config --global user.name "Your Name"
git config --global user.email "me@example.com"

# Verify your configuration
git config -l
```

### 2. Initializing a Repository
- To start using Git in a project, you must initialize it. This creates a hidden directory where Git stores its database, history, and metadata.
```bash
# Create a new project directory and move into it
mkdir scripts
cd scripts

# Initialize the Git repository
git init
```
- Output:
```bash
Initialized empty Git repository in /home/user/scripts/.git/
```

### What is the .git directory?
- You can view the hidden .git folder using ls -la. This folder is the actual "Version Control System." You should never edit the files in here manually; Git commands handle that for you.
```bash
ls -l .git/
```
- Output:
```bash
drwxrwxr-x 2 user user 4096 Jul  9 18:16 branches
-rw-rw-r-- 1 user user   92 Jul  9 18:16 config
-rw-rw-r-- 1 user user   73 Jul  9 18:16 description
-rw-rw-r-- 1 user user   23 Jul  9 18:16 HEAD
drwxrwxr-x 2 user user 4096 Jul  9 18:16 hooks
drwxrwxr-x 2 user user 4096 Jul  9 18:16 info
drwxrwxr-x 4 user user 4096 Jul  9 18:16 objects
drwxrwxr-x 4 user user 4096 Jul  9 18:16 refs
```


## 📝 Git & System Maintenance Cheat Sheet

This section provides a quick reference for the most commonly used commands in this course.

### 🔍 System Maintenance (Diff & Patch)
| Command | Description |
| :--- | :--- |
| `diff file1 file2` | Compares two files and shows differences. |
| `diff -u file1 file2` | Shows differences in **Unified Format** (with context). |
| `diff -u old_file new_file > change.diff` | Generates a patch file. |
| `patch original_file < change.diff` | Applies a patch to the original file. |

### ⚙️ Configuration
| Command | Description |
| :--- | :--- |
| `git config --global user.name "Name"` | Sets your commit username. |
| `git config --global user.email "email@example.com"` | Sets your commit email. |
| `git config -l` | Lists all current Git configurations. |

### 🛠️ Basic Workflow (Local)
| Command | Description |
| :--- | :--- |
| `git init` | Initializes a new local Git repository. |
| `git status` | Shows the status of the working tree and staging area. |
| `git add <file>` | Adds a file to the **Staging Area** (Tracks the file). |
| `git add .` | Adds all modified/new files to the staging area. |
| `git commit` | Opens editor to write a commit message for staged changes. |
| `git commit -m "Message"` | Commits staged changes with a short message. |
| `git log` | Displays the commit history (Snapshots). |

### 📂 File Management
| Command | Description |
| :--- | :--- |
| `git rm <file>` | Removes a file from the working tree and stages the deletion. |
| `git mv <old> <new>` | Renames/Moves a file and stages the change. |
| `ls -la .git/` | Inspects the internal Git database directory. |

### ⏪ Undoing Changes
| Command | Description |
| :--- | :--- |
| `git checkout <file>` | Reverts changes in a file to the last committed state. |
| `git reset HEAD <file>` | Unstages a file (removes it from the staging area). |
| `git commit --amend` | Overwrites the last commit with current staged changes. |

---
*Note: Use `git <command> --help` to see the full manual for any specific command.*
