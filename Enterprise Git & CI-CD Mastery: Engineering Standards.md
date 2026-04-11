# 🚀 Advanced Git Collaborative Workflow & PR Management
> "A clean commit history is the foundation of reliable CI/CD pipelines and effective Infrastructure as Code (IaC) management." — DevOps Engineering Principles

Welcome to the internal engineering guide on contributing to our infrastructure repositories. This document outlines the standard operating procedures (SOP) for forking, branching, submitting Pull Requests (PRs), and maintaining a clean git history through interactive rebasing.

---

## 1. Forking & Local Environment Setup

Directly committing to upstream infrastructure repositories (e.g., our `k8s-manifest-validator`) is restricted. We utilize a **Fork-and-Pull** model.

1. Navigate to the upstream repo and click **Fork**.
2. Clone your forked remote to your local development environment:

```bash
git clone [https://github.com/your-username/k8s-manifest-validator.git](https://github.com/your-username/k8s-manifest-validator.git)
cd k8s-manifest-validator
ls -l

```

**Expected Output:**

Plaintext

```
total 20
-rw-rw-r-- 1 devops devops 11357 Jan 7 09:42 LICENSE
-rw-rw-r-- 1 devops devops   211 Jan 7 09:42 validator.py
-rw-rw-r-- 1 devops devops   762 Jan 7 09:42 validator_test.py

```

Verify the commit history to ensure synchronization with upstream:

Bash

```
git log --oneline -3

```

> **Note:** Always keep your fork synced with the upstream `master` or `main` branch before starting new work.

----------

## 2. Branching & Feature Implementation

Never work directly on the `master` branch. Create a feature branch with a descriptive, ticket-based naming convention (e.g., `feature/INFRA-102-add-readme`).

Bash

```
git checkout -b feature/add-readme

```

Let's author a `README.md` file for the validator tool using your preferred IDE (e.g., VSCode, Vim, or Atom):

Markdown

```
# Kubernetes Manifest Validator
===============================

This module parses and validates YAML manifests against our cluster's security policies.

```

Stage and commit the initial implementation:

Bash

```
git add README.md
git commit -m 'docs: Add initial README for k8s validator'

```

----------

## 3. The Pull Request (PR) Lifecycle

Push your local branch to your forked remote repository:

Bash

```
git push -u origin feature/add-readme

```

### 📝 PR Creation Best Practices

When opening a PR in the GitHub UI, ensure your description provides maximum context for the reviewer:

-   **Why?** Explain the business or technical value (e.g., "Onboarding new engineers requires documentation for the validator").
    
-   **How?** Explain the testing methodology (e.g., "Tested locally against `deployment.yaml` and verified markdown rendering").
    

----------

## 4. Addressing Code Review Feedback

It is standard procedure to receive feedback from Senior Maintainers. Suppose a reviewer requests an example usage block in the README.

Update your file accordingly:

Markdown

```
# Example Usage

Calling the script via CLI:
`python validator.py --target ./manifests/production.yaml`
Returns: `[PASS] Security Context Validated.`

```

Commit and push the updates:

Bash

```
git commit -a -m 'docs: append CLI usage example to README'
git push

```

GitHub automatically appends this new commit to your open Pull Request. However, this creates a fragmented commit history.

----------

## 5. Advanced: Interactive Rebase & Squashing

To prevent polluting the upstream master branch with incremental commits (e.g., "fix typo", "add example"), maintainers will often request that you **squash** your commits before merging.

Since PR branches are generally isolated to your environment, it is safe to rewrite their history. We achieve this using `git rebase -i` (interactive rebase) against the target branch (`master`).

Bash

```
git rebase -i master

```

This opens your default terminal editor. Change the action of the second commit from `pick` to `squash`:

Plaintext

```
pick 736d754 docs: Add initial README for k8s validator
squash 01231b0 docs: append CLI usage example to README

# Rebase 367a127..01231b0 onto 367a127 (2 commands)
#
# Commands:
# p, pick = use commit
# s, squash = use commit, but meld into previous commit

```

Save and exit. Git will prompt you to combine the commit messages. Clean them up into a single, highly descriptive commit message:

Plaintext

```
docs: Create comprehensive README with CLI examples

This commit adds the initial README and includes security 
context validation examples for the k8s-manifest-validator.

```

### ⚠️ The Force Push

Because you have rewritten history, your local branch and the remote branch have diverged. Standard push will be rejected. You must **force push**:

Bash

```
git push -f

```

> **Warning:** Only force push to your isolated PR branches. Never force push to shared/mainline branches!

Check your beautiful, linear commit history:

Bash

```
git log --graph --oneline --all -4

```

----------

## 6. PR Merge Strategies

When a PR is approved, maintainers have several options to merge the code into the upstream repository. Understanding these helps you format your commits correctly:

**Merge Strategy**

**Description**

**Best Used For**

**Merge Commit** (`--no-ff`)

Retains all individual commits from the feature branch and creates a new "Merge" commit joining the two histories.

Large, multi-developer features where granular history is needed.

**Squash and Merge**

Condenses all PR commits into a single new commit on the base branch. The feature branch history is flattened.

Small features, bug fixes, or documentation updates. Prevents clutter.

**Rebase and Merge**

Replays all feature branch commits directly on top of the base branch without creating a merge commit.

Maintaining a perfectly linear history without "merge bubbles".

----------

## ✅ Pre-Merge Checklist

Before requesting a final review, ensure you have completed the following:

-   [x] Code passes all automated CI pipeline checks.
    
-   [x] Commits are logically grouped and squashed if necessary.
    
-   [x] PR description references the Jira/Issue ticket (e.g., `Closes #42`).
    
-   [x] Secrets/Tokens are **NOT** hardcoded in any configuration files.


---

## 7. Issue Tracking & Work Coordination

In a distributed DevOps environment, decentralized work without tracking leads to critical infrastructure gaps. We utilize GitHub's integrated **Issue Tracker** to coordinate tasks, report bugs, and assign responsibilities.

### 🐛 Issue Lifecycle Management
When creating an issue (e.g., "Implement syslog critical error parser"), provide comprehensive context:
* **Current State:** What is failing or missing?
* **Desired State:** What should the module do? (e.g., *Scan `/var/log/syslog` for `CRITICAL` flags and alert.*)
* **Acceptance Criteria:** How do we test it?

### 🔗 Auto-Closing Issues via Commits
To maintain a clean tracking history, developers must link their PRs and commits to open issues. GitHub automatically resolves issues when specific keywords are merged into the `master` branch.

**Example Implementation:**
If you are assigned to **Issue #42** (Missing documentation for syslog auditor), update the file and commit with the closure syntax:

```bash
git commit -a -m "docs: update syslog auditor configuration and usage

Includes detailed parameter flags for log parsing.
Closes #42"

```

Once merged, GitHub will automatically transition Issue #42 to `Closed`, streamlining our Agile board.

----------

## 8. Continuous Integration & Deployment (CI/CD)

"It worked on my machine" is not an acceptable engineering standard. We enforce strict **CI/CD pipelines** to automate the build, test, and deployment of our infrastructure code.

### 🔄 The CI/CD Pipeline Architecture

Every push or Pull Request triggers our CI system (e.g., Jenkins, Travis CI). The pipeline executes a defined set of steps:

1.  **Linting:** Enforces coding standards.
    
2.  **Testing:** Runs automated unit and integration tests.
    
3.  **Build:** Compiles code or builds container images.
    
4.  **Artifact Generation:** Produces downloadable logs, compiled binaries, or OS-specific packages.
    

### 🔒 Security Mandate: Secrets Management

When configuring CI/CD YAML files, **never** hardcode API tokens, SSH keys, or passwords.

> **CRITICAL SECURITY DIRECTIVE:** > - **Separation of Privileges:** The service account (API token) authorized to deploy to the Staging/Test environment MUST NOT be the same entity authorized for Production.
> 
> -   **Compromise Recovery:** Always maintain a break-glass recovery plan to revoke pipeline access immediately if a token is leaked.
>     

----------

## 9. Code Review (CR) & Quality Assurance

Code review is our primary defense against technical debt and production incidents. We adhere strictly to the **Google Style Guides** for all supported languages to ensure our massive codebase remains readable and uniform.

### 🔍 Accepted Code Review Strategies

Depending on the complexity of the feature, we utilize different peer review methodologies:

**Strategy**

**Description**

**Best For**

**Tool-Assisted (PRs)**

Asynchronous reviews via GitHub. Leaves audit trails and usage metrics.

Standard workflow, remote teams.

**Over-the-Shoulder**

Author explains the code locally to a peer.

Complex logic requiring immediate feedback.

**Pair Programming (XP)**

Two engineers write the code together simultaneously.

Mentoring juniors, high-risk core infrastructure.

### ⭐ 5 Golden Rules for Pull Request Reviews

When you submit or review a PR, adhere to these standards:

1.  **Be Selective:** Do not tag the entire engineering department. Tag 1-2 subject matter experts.
    
2.  **Timeliness:** Aim to complete reviews within **2 hours** to prevent context-switching delays.
    
3.  **Constructive Feedback:** Explain _why_ a change is needed, not just _what_. Use non-accusatory language.
    
4.  **Detailed Descriptions:** The PR body must explain the delta between the feature branch and `master`, prerequisites, and testing steps.
    
5.  **Interactive Rebasings:** Keep commits clean. Use `git rebase -i` to squash messy WIP commits before requesting a review.
    

----------

## 10. Repository Authentication Policies

To push code or access private repositories, you must authenticate securely. **Basic password authentication over HTTPS is deprecated.**

### Option A: HTTPS with Personal Access Token (PAT)

If using HTTPS, you must generate a PAT from your GitHub Developer Settings.

To avoid entering it constantly, configure the **Git Credential Manager (GCM)**, which supports Multi-Factor Authentication (MFA) and securely stores tokens at the OS level.

_(Alternative: Local `.netrc` file)_

Plaintext

```
machine github.com
    login your-corporate-username
    password your-personal-access-token

```

_Ensure this file has strict permissions (`chmod 600 ~/.netrc`), or Git will reject it._

### Option B: SSH Authentication (Recommended for DevOps)

For seamless, secure CLI operations, utilize SSH key pairs.

1.  Generate an RSA/Ed25519 key pair (`ssh-keygen`).
    
2.  Copy the public key (`id_rsa.pub`).
    
3.  Add the public key to your GitHub account (`Settings > SSH and GPG keys`).
    

----------

## 📚 Appendix: Standard Engineering Terminology

-   **Artifacts:** Compiled binaries, packages, or logs generated by a CI pipeline.
    
-   **Continuous Deployment (CD):** The automated process of deploying incremental code changes to production.
    
-   **Fix up:** A Git rebase command that melds a commit into the previous one but _discards_ its commit message.
    
-   **Squash:** A Git rebase command that melds a commit into the previous one and _combines_ their commit messages.
    
-   **Indirect Merges:** When GitHub automatically marks a PR as merged because its commits were integrated into the base branch via another external route.

---

## 11. Workflow Case Study: Security Patching & Upstream Synchronization

This section documents a real-world engineering scenario: identifying a logic vulnerability in a user-validation module, applying a fix in an isolated feature branch, and contributing the patch back to the upstream production repository.

### 🏗️ Environment Initialization

In an enterprise environment, we never modify the `upstream` source directly. We utilize a forked synchronization model to ensure stability.

1. **Synchronize Remote Tracking:**
   Verify your remote connections to ensure you are receiving updates from the source (`upstream`) and pushing to your workspace (`origin`).

```bash
# Check current remotes
git remote -v

# Add the primary engineering repo as upstream
git remote add upstream [https://github.com/google/it-cert-automation-practice.git](https://github.com/google/it-cert-automation-practice.git)

# Verify multi-remote configuration
git remote -v

```

**Expected Output:**

Plaintext

```
origin    [https://github.com/your-username/it-cert-automation-practice.git](https://github.com/your-username/it-cert-automation-practice.git) (fetch)
origin    [https://github.com/your-username/it-cert-automation-practice.git](https://github.com/your-username/it-cert-automation-practice.git) (push)
upstream  [https://github.com/google/it-cert-automation-practice.git](https://github.com/google/it-cert-automation-practice.git) (fetch)
upstream  [https://github.com/google/it-cert-automation-practice.git](https://github.com/google/it-cert-automation-practice.git) (push)

```

----------

## 12. Feature Branching & Vulnerability Assessment

Standard Operating Procedure (SOP) requires creating a dedicated branch for every bug fix to prevent `master` branch pollution.

Bash

```
git checkout -b fix/IAM-username-validator
cd ~/it-cert-automation-practice/Course3/Lab4

```

### 🔍 Identifying the Logic Bug

The module `validations.py` is intended to enforce strict username policies. However, current unit tests reveal a failure in edge-case handling where usernames starting with symbols are incorrectly marked as `True`.

**Initial State (`validations.py`):**

Python

```
def validate_user(username, minlen):
    if not re.match('^[a-z0-9._]*$', username):
        return False
    # BUG: Only checks if the first char is numeric, ignores symbols
    if username[0].isnumeric():
        return False
    return True

```

**Test Execution Results:**

**Input**

**Expected**

**Actual**

**Status**

`blue.kale`

True

True

✅

`.blue.kale`

**False**

**True**

❌ **FAIL**

`_red_quinoa`

**False**

**True**

❌ **FAIL**

----------

## 13. Engineering the Fix: Implementation & Verification

To harden the validation engine, we refactor the first-character check to ensure it _only_ accepts alphabetic characters, blocking dots, underscores, and numbers at index `0`.

### 🛠️ The Solution

Update the logic to use `.isalpha()` or a more restrictive regex:

Python

```
def validate_user(username, minlen):
    # ... previous length and type checks ...
    
    # Enforce alphabetic start character
    if not username[0].isalpha():
        return False
    return True

```

**Verified Output:**

Bash

```
python3 validations.py
# Output:
# True
# False
# True
# False

```

_Validation logic now satisfies security requirements._

----------

## 14. Standardized Commit & Remote Integration

When the fix is verified, we move to the staging and commit phase. To automate our project management, we use the `Closes` keyword to link the commit to the internal issue tracker.

Bash

```
git add validations.py
git status

# Commit using Professional Engineering standards
git commit -m "fix(auth): restrict usernames to start with alphabetic characters only

- Refactored validate_user to block symbols/numbers at index 0.
- Updated unit tests to include edge cases for leading dots/underscores.
Closes: #1"

```

### 🚀 Pushing to Origin (MFA/PAT Required)

Since GitHub deprecated password authentication, utilize your **Personal Access Token (PAT)** or **SSH Key** to push the feature branch:

Bash

```
git push origin fix/IAM-username-validator

```

----------

## 15. Pull Request (PR) & Peer Review

Upon pushing, the terminal provides a direct link to generate a Pull Request.

### ✅ Final PR Checklist for Engineers:

1.  **Base Repository:** `google/it-cert-automation-practice` | **Base Branch:** `master`
    
2.  **Head Repository:** `your-username/it-cert-automation-practice` | **Compare Branch:** `fix/IAM-username-validator`
    
3.  **Continuous Integration:** Ensure the "Checks" section shows green before tagging reviewers.
    
4.  **Documentation:** Ensure the PR description explicitly mentions that `PR won't be merged on master` if it is a training/sandbox exercise.
