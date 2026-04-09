
# Git Operations and Error Management Guide

> **Document Purpose:** This guide is designed to standardize Git file operations, error recovery strategies, and commit history refinement techniques commonly encountered in CI/CD pipelines, Infrastructure as Code (IaC) management, and automation scripting.

---

## 📑 Table of Contents
- [1. Infrastructure Asset Management](#1-infrastructure-asset-management)
  - [Removing Obsolete Scripts (`git rm`)](#removing-obsolete-scripts-git-rm)
  - [Renaming Assets (`git mv`)](#renaming-assets-git-mv)
  - [Excluding Temporary and System Files (`.gitignore`)](#excluding-temporary-and-system-files-gitignore)
- [2. Error Management & Reverting](#2-error-management--reverting)
  - [Discarding Unstaged Changes](#discarding-unstaged-changes)
  - [Unstaging Accidentally Staged Files](#unstaging-accidentally-staged-files)
- [3. Amending Commit History](#3-amending-commit-history)
  - [Adding Missing Files and Editing Messages (`--amend`)](#adding-missing-files-and-editing-messages---amend)

---

## 🗂️ 1. Infrastructure Asset Management

Over time, server control scripts become obsolete or require refactoring. Keeping the repository clean is a core tenet of DevOps culture.

### 🗑️ Removing Obsolete Scripts (`git rm`)
When files become deprecated, simply deleting them from the disk is insufficient; they must also be removed from Git's tracking.

```bash
# Check repository contents
$ ls -l
total 8
-rw-rw-r-- 1 devops 659 Jul  9 19:28 ebs_volume_check.py
-rw-rw-r-- 1 devops 659 Jul 15 21:43 legacy_deploy.sh

# Remove the legacy deployment script from Git tracking and disk
$ git rm legacy_deploy.sh
rm 'legacy_deploy.sh'

# Verify the status
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    legacy_deploy.sh

# Finalize the change
$ git commit -m "chore: remove deprecated legacy_deploy script"

```

### 🔄 Renaming Assets (`git mv`)

A script may start with a single purpose but expand over time. Similar to the Linux `mv` command, `git mv` is used to move or rename files within Git.

Bash

```
# Rename a specific script to a more generic and inclusive name
$ git mv ebs_volume_check.py check_free_space.py

$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        renamed:    ebs_volume_check.py -> check_free_space.py

$ git commit -m "refactor: rename ebs check script to generic free space checker"

```

### 🙈 Excluding Temporary and System Files (`.gitignore`)

Preventing OS artifacts (`.DS_Store`), log files (`*.log`), or execution outputs (`output.txt`) from entering the repo reduces "noise."

Bash

```
# Add new rules to the ignore list
$ echo ".DS_Store" > .gitignore
$ echo "*.log" >> .gitignore

# Check including hidden files
$ ls -la
total 20
drwxrwxr-x 8 devops devops 4096 Jul 15 21:52 .git
-rw-rw-r-- 1 devops devops   16 Jul 15 22:15 .gitignore
-rw-rw-r-- 1 devops devops  659 Jul  9 19:28 check_free_space.py

# Include the .gitignore file in the repo
$ git add .gitignore 
$ git commit -m "chore: add .gitignore for OS artifacts and logs"

```

----------

## ⏪ 2. Error Management & Reverting

One of the greatest strengths of version control systems is the ability to safely revert errors. Different strategies apply depending on the state of the change.

### 🛑 Discarding Unstaged Changes

Suppose you broke a "Health Check" script while updating and haven't run `add` or `commit` yet. To discard changes and return to the last clean version, use `git checkout` (or `git restore` in newer versions).

Bash

```
# Execute the accidentally broken file
$ ./cluster_health_check.py 
Traceback (most recent call last):
  File "./cluster_health_check.py", line 14, in <module>
NameError: name 'check_reboot' is not defined

# Git status provides a hint
$ git status
On branch master
Changes not staged for commit:
  (use "git checkout -- <file>..." to discard changes in working directory)
        modified:   cluster_health_check.py

# Restore the file to the last committed, working state
$ git checkout cluster_health_check.py

# Test again
$ ./cluster_health_check.py 
Everything ok.

```

### 📦 Unstaging Accidentally Staged Files

Sometimes commands like `git add *` include files that shouldn't be in the repo, such as debug logs. `git reset` is used to clear them from the Staging Area.

Bash

```
# Accidentally added all files
$ git add *
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
        new file:   debug_output.log

# Unstage the log file
$ git reset HEAD debug_output.log
Unstaged changes after reset:
M       debug_output.log

$ git status
On branch master
Untracked files:
        debug_output.log

```

----------

## 🛠️ 3. Amending Commit History

### ✍️ Adding Missing Files and Editing Messages (`--amend`)

If you committed new scripts but forgot one file or realized the message wasn't descriptive enough, use the `--amend` parameter to overwrite the last commit instead of creating commit clutter.

> **⚠️ CRITICAL SECURITY WARNING:** NEVER use `git commit --amend` on commits that have been pushed to a remote/public repository. It rewrites history and causes severe conflicts for other developers. Use it only for **local** commits.

Bash

```
# Created new scripts
$ touch auto_scale.py
$ touch collect_metrics.sh

# Committed only one by mistake
$ git add auto_scale.py
$ git commit -m "feat: add auto scaling and metrics scripts"
[master 9c78761] feat: add auto scaling and metrics scripts
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 auto_scale.py

# Add the forgotten file to the stage
$ git add collect_metrics.sh

# Amend the last commit (Editor will open)
$ git commit --amend

```

**Professional Commit Message Example:**

Plaintext

```
feat: add two new automation scripts

- collect_metrics.sh: Gathers system performance under heavy load.
- auto_scale.py: Triggers ECS tasks based on daily traffic.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
#       new file:   auto_scale.py
#       new file:   collect_metrics.sh

```

----------

## 📌 Quick Command Reference Table

**Need / Situation**

**Git Command**

Delete file from disk and Git

`git rm <file_name>`

Rename / Move file

`git mv <old_name> <new_name>`

Revert broken file to last state

`git checkout <file_name>` or `git restore <file_name>`

Unstage a file

`git reset HEAD <file_name>`

Modify last commit / Add missing file

`git commit --amend`
