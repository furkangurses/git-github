
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


---



# 🛠️ Advanced Git Engineering: Branching, Merging & History Management

> **Engineer's Note:** This documentation provides a technical overview of Git operations beyond the basics. It focuses on patch management, safe undo strategies, and complex branching workflows used in high-availability infrastructure projects.

---

## 🔍 1. Inspecting Changes & Patch Management

In an engineering environment, reviewing exactly which lines of code are modified is critical before pushing to a production branch.

### 📝 Viewing Diffs and Patches
Instead of just checking status, we use `diff` and `log -p` to inspect the actual code changes (the "patch").

```bash
# View changes in the working directory vs the index
$ git diff

# View staged changes (ready to be committed)
$ git diff --staged

# View the commit history with actual code changes (patches)
$ git log -p

# Show details of a specific commit or object
$ git show <commit_id>

```

### 🎯 Interactive Staging (`git add -p`)

To maintain clean commits, use the interactive patch mode. This allows you to review and stage specific "hunks" of code rather than the whole file.

Bash

```
# Interactively choose which changes to stage
$ git add -p

```

----------

## ⏪ 2. Undo Strategies & Safety Protocols

As an engineer, knowing how to recover from a bad configuration or a broken script is paramount.

### 🛡️ Safe Reversion vs. Destructive Reset

-   **`git revert`**: Creates a _new_ commit that undoes changes. Safe for public branches.
    
-   **`git reset`**: Moves the branch pointer. Use with caution.
    

Bash

```
# Revert a specific commit safely without rewriting history
$ git revert <commit_id>

# Unstage files without losing local work (Soft Reset)
$ git reset

# TEMPORARY STORAGE: Stash changes before a hard reset or branch switch
$ git stash
$ git stash pop

```

### ✍️ Amending the Last Commit

If you forgot to add a configuration file or made a typo in the message of your _local_ commit:

Bash

```
# Stage the forgotten file
$ git add config.yaml

# Overwrite the previous commit
$ git commit --amend

```

----------

## 🌿 3. Branching & Merging Workflows

Isolation of features and fixes is handled through robust branching strategies.

### 🏗️ Branch Management

Bash

```
# List all local branches
$ git branch

# Create a new feature branch and switch to it immediately
$ git checkout -b feature/api-gateway-auth

# Delete a branch after a successful merge
$ git branch -d feature/api-gateway-auth

# Force delete an unmerged branch (Use only if data loss is acceptable)
$ git branch -D obsolete-experiment

```

### 🤝 Merging and Conflict Resolution

Integrating changes from a feature branch into the `main` branch.

Bash

```
# Merge the feature branch into current branch
$ git merge <feature-branch>

# If a merge goes wrong or conflicts are too complex, abort safely
$ git merge --abort

```

----------

## 📈 4. Visualizing History

Tracking complex merge histories in a microservices architecture requires high-level visualization tools.

Bash

```
# View a condensed, one-line history
$ git log --oneline

# View an ASCII graph of commits, branches, and merges
$ git log --graph --oneline --all

```

----------

## 🚀 Engineering Cheat Sheet

**Command**

**Use Case**

**Risk Level**

`git commit -a`

Stage all modified files and commit in one go.

Medium (Avoid for new files)

`git checkout <file>`

Restore a file to its last committed state.

High (Loses local work)

`git stash`

Shelve changes to work on something else.

Low (Very Safe)

`git revert`

Roll back a production bug with a trace.

Low (Safe for Public)

`git reset --hard`

Force reset to a specific commit.

Critical (Permanent Data Loss)
