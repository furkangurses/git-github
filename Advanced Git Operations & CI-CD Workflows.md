# 🛡️ Infrastructure Health Checks & Team Git Workflow

> **Welcome to the `health-checks` repository.** > This repository contains critical Python scripts and automation tools used to monitor the state of our production nodes (e.g., disk usage, pending reboots, network connectivity). 
> 
> *As a DevOps team, maintaining a clean, predictable, and linear Git history is just as important as the code we write. Please review this guide to understand our standard remote repository operations.*



---

## 1. Prerequisites & Authentication

Before interacting with our upstream infrastructure repositories, ensure your environment is configured for seamless authentication. Constantly typing passwords interrupts flow.

**Enable Credential Caching (15-minute default window):**
```bash
git config --global credential.helper cache

```

_Pro-Tip: For persistent environments, consider setting up an SSH key pair and associating your public key with your GitHub Enterprise profile._

----------

## 2. Initial Setup: Cloning the Repo

To begin contributing to the health checks automation, pull down a local copy of the master tree.

Bash

```
# Clone the repository via HTTPS (or SSH)
git clone [https://github.com/redquinoa/health-checks.git](https://github.com/redquinoa/health-checks.git)

# Navigate into the working directory
cd health-checks/

```

Verify the contents. You should see our initial setup:

Bash

```
ls -l
# Output: -rw-rw-r-- 1 user user 62 Jan  6 14:06 README.md

```

----------

## 3. Daily Collaboration Workflow

In a distributed DevOps environment, multiple engineers are pushing infrastructure code simultaneously. Here is how to keep your local state synchronized and push your updates.

### Pushing Your Commits

Once you have modified a script (e.g., adding `check_reboot()`) and committed it locally, push the state upstream:

Bash

```
git commit -a -m "feat(system): add pending reboot check"
git push

```

### Pulling Upstream Changes

Before starting new work, always pull the latest changes from your team to avoid merge conflicts:

Bash

```
git pull

```

> **Behind the Scenes:** `git pull` is a composite command. It executes a `git fetch` (downloads the data) followed immediately by a `git merge origin/master` (integrates it into your current working tree).

----------

## 4. Managing Remote Repositories

Understanding how your local machine tracks our upstream servers is crucial for debugging synchronization issues.

### Inspecting Remotes

By default, Git names the remote server `origin`. To view the exact upstream URLs you are tracking:

Bash

```
git remote -v
# Output:
# origin  [https://github.com/redquinoa/health-checks.git](https://github.com/redquinoa/health-checks.git) (fetch)
# origin  [https://github.com/redquinoa/health-checks.git](https://github.com/redquinoa/health-checks.git) (push)

```

For a deeper dive into the health of your tracking branches:

Bash

```
git remote show origin

```

_This command displays HEAD branches, tracked remote branches, and whether your local refs are out-of-date compared to upstream._

### Safe Synchronization (Fetch + Merge)

If you want to review changes _before_ integrating them into your active workspace, use `fetch` instead of `pull`:

Bash

```
# 1. Download the latest upstream state without modifying your working tree
git fetch

# 2. Inspect the commits pushed by your colleagues
git log origin/master

# 3. Check your standing against upstream
git status
# Output: Your branch is behind 'origin/master' by 1 commit...

# 4. Integrate via fast-forward (if linear)
git merge origin/master

```

----------

## 5. Working with Feature Branches

We isolate experimental features (like aggressive CPU throttling checks) in separate remote branches.

To view all branches existing on the remote server:

Bash

```
git branch -r

```

If an engineer has pushed a branch named `experimental`, you can track it locally by simply checking it out. Git will automatically set up the tracking linkage:

Bash

```
git checkout experimental
# Output: Branch 'experimental' set up to track remote branch 'experimental' from 'origin'.

```

To update your local cache of **all** remote branches without merging:

Bash

```
git remote update

```

----------

## 6. DevOps Git Command Cheat Sheet

**Command**

**Operational Context**

`git clone <url>`

Bootstraps a new local repository from an upstream source.

`git push`

Broadcasts your local, committed infrastructure changes upstream.

`git pull`

Fetches and immediately merges the latest upstream states.

`git fetch`

Safely downloads remote objects and refs without altering the working tree.

`git remote -v`

Audits the URLs mapped to your remote aliases (e.g., `origin`).

`git remote show <name>`

Diagnoses tracking status between local and remote branches.

`git branch -r`

Lists all remote-tracking branches available to you.

`git remote update`

Refreshes all tracked remote branches globally.

----------

## 7. Onboarding Checklist

New engineers to the squad, please complete the following before your first PR:

-   [ ] Clone the `health-checks` repository.
    
-   [ ] Configure your global `credential.helper` or SSH keys.
    
-   [ ] Run `git remote show origin` to verify connectivity.
    
-   [ ] Review the `check_disk_full` logic in `all_checks.py`.
    
-   [ ] Create a feature branch for your first assigned Jira ticket.

---


# 🚀 Advanced Git Operations & CI/CD Workflows

> **Infrastructure Team Handbook: Part 2**
> This repository documents the advanced Git workflows required for contributing to our core infrastructure scripts. 
> 
> *Objective: Ensure high availability of our systems while maintaining a clean, linear, and predictable version control history.*



---

## 1. 🛑 Handling Upstream Divergence (Merge Conflicts)

In a highly collaborative DevOps environment, multiple engineers modify the same configuration files concurrently. If your `git push` is rejected, it means the upstream remote contains state changes you lack locally.

### Resolution Protocol:
1. **Pull Upstream Changes:** Fetch and attempt to merge the remote state.
   ```bash
   git pull
   # If there are overlapping changes, Git will pause and flag a CONFLICT.

   ```

2.  **Audit the State:** Visualize the divergent paths.
    
    Bash
    
    ```
    git log --graph --oneline --all
    
    ```
    
3.  **Resolve Manually:** Open the conflicted file (e.g., `system_diagnostics.py`). Look for the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`). Reconcile the logic—for instance, combining a variable rename (`min_gb`) with a newly enforced conditional execution order.
    
4.  **Finalize the Merge:**
    
    Bash
    
    ```
    git add system_diagnostics.py
    git commit -m "fix(diagnostics): resolve merge conflict in storage evaluation logic"
    git push
    
    ```
    

----------

## 2. 🌿 Feature Branching Strategy

**Never commit experimental infrastructure changes directly to `master`.** To ensure system stability, all refactoring and feature additions must be isolated in dedicated branches. This allows us to patch critical production bugs on `master` without deploying half-finished features.

### Creating an Isolated Environment

Bash

```
# Create and immediately switch to a new infrastructure branch
git checkout -b refactor/diagnostic-wrappers

```

### Iterative Commits

Make self-contained, logical commits. For example, when refactoring our diagnostic loop to catch multiple system failures:

Bash

```
git commit -a -m "refactor(core): implement wrapper functions for storage checks"
git commit -a -m "feat(core): enable multi-error logging via boolean state flags"

```

### Pushing the Feature Upstream

To allow code review (via Pull Requests) and CI pipeline testing, publish your branch to the remote origin:

Bash

```
git push -u origin refactor/diagnostic-wrappers

```

----------

## 3. 🔄 The Rebase Workflow (Linear History)

While standard merges are acceptable, our team strictly prefers **Rebasing** feature branches before merging them. Rebasing rewrites the commit history to make it look as if your changes were written _after_ the latest upstream updates, eliminating messy "three-way merge" commits.

### The Standard Rebase Procedure:

1.  **Update Local Master:** Ensure you have the absolute latest production state.
    
    Bash
    
    ```
    git checkout master
    git pull
    
    ```
    
2.  **Rebase the Feature Branch:** Replay your feature commits on top of the new `master` tip.
    
    Bash
    
    ```
    git checkout refactor/diagnostic-wrappers
    git rebase master
    
    ```
    
    > _Note: If conflicts occur during rebase, resolve them, `git add` the files, and run `git rebase --continue`._
    
3.  **Fast-Forward Merge:** Integrate the rebased feature back into production.
    
    Bash
    
    ```
    git checkout master
    git merge refactor/diagnostic-wrappers
    
    ```
    

----------

## 4. 🧹 Branch Lifecycle & Cleanup

Once a feature is successfully integrated and deployed, its corresponding branches are considered obsolete and must be purged to keep our repository clean.

Bash

```
# 1. Delete the remote tracking branch from upstream
git push --delete origin refactor/diagnostic-wrappers

# 2. Delete the local development branch
git branch -d refactor/diagnostic-wrappers

```

----------

## 5. 🛠️ Command Reference Table

**Operation**

**Command Syntax**

**Description**

**Visualize Tree**

`git log --graph --all`

Displays a topological graph of all branches and commits.

**Branch Creation**

`git checkout -b <name>`

Instantiates and switches to a new isolated branch.

**Upstream Link**

`git push -u origin <name>`

Pushes a new local branch and sets upstream tracking.

**Rebasing**

`git rebase <base_branch>`

Rewinds and replays current branch commits atop `<base_branch>`.

**Prune Remote**

`git push --delete origin <name>`

Removes the specified branch from the remote server.

----------

## 6. ✅ Pre-Merge Checklist

Before executing a `git merge` into `master`, verify the following:

-   [ ] All diagnostic scripts have been tested locally (`sys.exit(0)` is returned on healthy runs).
    
-   [ ] You have pulled the latest `master` and successfully rebased your branch.
    
-   [ ] No conflict markers (`<<<<<<<`) remain in the codebase.
    
-   [ ] Commit messages follow the conventional semantic format (e.g., `feat:`, `fix:`, `refactor:`).


---

# 🚀 Advanced Git Workflows: Maintaining Linear History in Distributed Teams

> **Operational Standard for Infrastructure Monitoring Repositories**
> This guide outlines the "Fetch-Rebase-Push" workflow, a professional DevOps standard designed to maintain a clean, linear commit history. This approach avoids unnecessary "merge bubbles" and simplifies the auditing and debugging process in high-velocity production environments.

---

## 📋 Table of Contents
1. [The Linear History Mandate](#1-the-linear-history-mandate)
2. [Case Study: Implementing Network Connectivity Checks](#2-case-study-implementing-network-connectivity-checks)
3. [The "Fetch-Rebase" Protocol](#3-the-fetch-rebase-protocol)
4. [Conflict Resolution: A Step-by-Step Guide](#4-conflict-resolution-a-step-by-step-guide)
5. [Collaboration Best Practices](#5-collaboration-best-practices)
6. [Engineer’s Glossary & Resource Kit](#6-engineers-glossary--resource-kit)

---

## 1. The Linear History Mandate

In modern CI/CD pipelines, a complex, branching history makes it difficult to track down when a bug was introduced. By using **Rebasing** instead of traditional **Merging**, we ensure that all changes appear in a single, straight line.

> **Why Rebase on Master?**
> When you and a teammate modify the same code simultaneously, rebasing "replays" your work on top of their latest commit. This eliminates the "Three-Way Merge" commit and keeps the log readable.

---

## 2. Case Study: Implementing Network Connectivity Checks

To broaden our health checks, we are introducing a socket-based network validation.

### Implementation Logic:
```python
import socket

def check_no_network():
    """Returns True if it fails to resolve Google's URL, False otherwise."""    
    try:
        # Attempt DNS resolution
        socket.gethostbyname("[www.google.com](https://www.google.com)")
        return False
    except socket.error:
        return True

```

### Commit Procedure:

Bash

```
# Add the check to our execution list and commit locally
git commit -a -m 'feat(network): add simple connectivity check via DNS resolution'

```

----------

## 3. The "Fetch-Rebase" Protocol

Instead of a standard `git pull`, which may create a merge commit, follow this professional sequence:

### Step 1: Synchronize Upstream State

Bash

```
git fetch

```

_This updates your `origin/master` branch without touching your working files, allowing you to see if your teammates have pushed new changes._

### Step 2: Rebase Local Work

Bash

```
git rebase origin/master

```

_If a teammate has renamed files or added logic (e.g., a CPU usage check), Git will attempt to stitch your changes onto their new file structure._

----------

## 4. Conflict Resolution: A Step-by-Step Guide

During a rebase, conflicts are common if a file was renamed or modified in the same region.

### The DevOps Resolution Workflow:

1.  **Identify the Conflict:** Git will stop the rebase and flag files (e.g., `health_checks.py`).
    
2.  **Audit & Merge Logic:** Open the file, remove the conflict markers (`<<<<<<<` and `>>>>>>>`), and ensure both functions (CPU Check + Network Check) are integrated.
    
3.  **Validate Dependencies:** In our case, ensure `psutil` and `socket` are both imported.
    
4.  **Continue the Operation:**
    
    Bash
    
    ```
    # Mark the conflict as resolved
    git add health_checks.py 
    
    # Finalize the rebase
    git rebase --continue
    
    ```
    
5.  **Verify the Graph:**
    
    Bash
    
    ```
    git log --graph --oneline
    # You should now see a straight vertical line of commits.
    
    ```
    

----------

## 5. Collaboration Best Practices

**Rule**

**Description**

**Sync Early & Often**

Pull/Fetch before starting work to minimize the conflict window.

**Atomic Commits**

Keep commits small and self-contained. Don't mix a "variable rename" with a "new feature."

**No Public Rebasing**

**Warning:** Never rebase commits that have already been pushed to a shared remote. It rewrites history and breaks your teammates' environments.

**Semantic Messages**

Use clear context (e.g., `fix:`, `feat:`, `refactor:`) to help others understand the "Why" behind the "What."

----------

## 6. Engineer’s Glossary & Resource Kit

### Core Concepts

-   **Distributed VCS:** Every developer maintains a full local copy of the repository history.
    
-   **Rebasing:** Changing the base commit of a branch to keep history linear.
    
-   **SSH (Secure Shell):** The standard protocol for secure, key-based authentication with GitHub/GitLab.
    
-   **API Key:** An authentication token used to identify and authorize programmatic access.
    

### Quick Command Reference

Bash

```
git remote update          # Refresh all remote branches
git log --graph --all      # Visualize the entire project topology
git push --delete origin X # Prune stale remote features branches
git branch -d X            # Clean up local merged branches
```
