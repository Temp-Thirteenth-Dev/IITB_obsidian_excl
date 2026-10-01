# CS 699 — Lecture 3 Study Notes
## Version Control, Git & GitHub

> **Source:** CS 699 – Lec 3, Om Damani, CSE IIT Bombay  
> **Purpose:** Lecture revision + quiz preparation + lab-test/practical preparation

---

# 1. What You Should Be Able to Do

After this lecture, you should be able to:

- Explain what a **Version Control System (VCS)** is and why it is useful.
- Distinguish **Local VCS, Centralised VCS (CVCS), and Distributed VCS (DVCS)**.
- Explain why Git is a **distributed** version control system.
- Create a Git repository and make commits.
- Understand the basic Git flow:
  **working directory → staging area → repository**
- Inspect changes using `git status`, `git diff`, and `git diff --staged`.
- Create, switch, compare, merge, and delete branches.
- Explain and resolve a **merge conflict**.
- Understand a **push rejection** and why `git push --force` is dangerous.
- Use `.gitignore`.
- Explain the difference between **Git, GitHub, GitLab, and Sourcetree**.
- Connect a local repository to GitHub and push commits.
- Work collaboratively using branches and Pull Requests.
- Understand the basic Git object model: **blobs, trees, commits**.
- Perform the kinds of Git workflows required by the lecture assignments.

---

# 2. Version Control

## Definition

A **Version Control System (VCS)** is a tool that records changes made to files over time.

It allows you to:

- Track changes
- Compare versions
- Restore previous versions
- Manage different versions of a project
- Recall a specific version later
- See who changed what
- Undo mistakes

### Why do we need it?

Without version control, collaboration can turn into:

```text
report_final
report_final_v2
report_final_NEW
report_final_ACTUAL
report_final_ACTUAL_v3
```

Git provides a systematic history instead.

---

# 3. Types of Version Control Systems

| Type | Main idea | Example |
|---|---|---|
| **Local VCS** | History stored in a database on one machine | RCS |
| **Centralised VCS (CVCS)** | One server holds the full history | CVS, SVN |
| **Distributed VCS (DVCS)** | Every clone contains the complete repository/history | Git, Mercurial |

## Local VCS

History is stored locally on a single machine.

Example:

- **RCS (1982)** — tracked patch sets per file.

## Centralised VCS

A central server contains the repository history.

Examples:

- CVS (1990)
- SVN / Subversion (2000)

The lecture notes that CVS had no atomic commits, while SVN fixed several CVS flaws.

## Distributed VCS

Every clone is a complete repository containing the history.

Examples:

- Git (2005)
- Mercurial / hg (2005)

### Key point

> **Git is DVCS: a clone is not merely a snapshot; it contains the repository history.**

---

# 4. Why Version Control?

## 4.1 Collaboration without chaos

Multiple people can edit the same project.

Git can merge their work instead of simply overwriting changes.

## 4.2 Accountability and history

Changes are associated with:

- Person
- Timestamp
- Commit message

The message explains why a change was made.

## 4.3 Safe experimentation

Branches allow risky work to happen separately.

```text
main
 |
 +---- feature branch
          |
          +---- experiment
```

The working `main` branch can remain untouched until the experiment is ready.

## 4.4 Safety net

Each commit acts as a restore point.

If a change breaks the project, earlier states can be recovered.

---

# 5. What is Git?

**Git** is a distributed version control system used to track changes in files and manage different versions of a project.

Important facts from the lecture:

- Created by **Linus Torvalds**
- Originally developed in **April 2005**
- Works offline for operations such as:
  - commit
  - branch
  - diff
  - log
- Network is needed for sharing operations such as push/pull.
- Supports branching and merging.
- Multiple developers can work on the same project.
- Git objects are content-addressed by hashes.
- If content changes, its identity/hash changes.

---

# 6. Why Git?

## Distributed

```bash
git clone <repository>
```

gives you the repository history, not merely the latest snapshot.

Many operations work offline:

```bash
git commit
git branch
git diff
git log
```

### Important consequence

There is no need for a server for normal local version-control operations.

A clone contains the repository inside its `.git` directory.

---

## Branching

Branches are inexpensive and allow independent development.

```text
main
 |
 +---- feature/login
 |
 +---- experiment
```

---

## Content-based identity

Git objects are named using hashes of their contents.

This means:

- Identical content can be stored once.
- Changing content changes its hash.
- Undetected corruption/tampering is detectable.

---

## Recovery

The lecture highlights:

```bash
git reflog
```

It records previous states that the repository's `HEAD` has been in.

The lecture notes approximately **90 days** as the usual reflog retention period.

---

# 7. Git's Basic Process Flow

The fundamental mental model is:

```text
Working Directory
       |
       | git add
       v
Staging Area
       |
       | git commit
       v
Git Repository
```

Think of it as:

```text
edit files
   ↓
check changes
   ↓
stage selected changes
   ↓
commit a snapshot
```

## Three important states

### Working directory

Files you are currently editing.

### Staging area

Changes selected for the next commit.

### Repository

Committed history stored by Git.

---

# 8. Installing and Configuring Git

## Install

On Linux:

```bash
sudo apt install git
```

Check installation:

```bash
git --version
```

## Configure identity

Git attaches a name and email to commits.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Set the default branch name:

```bash
git config --global init.defaultBranch main
```

Check configuration:

```bash
git config --list
```

### Lab-test memory

If asked to configure Git, remember:

```text
user.name
user.email
init.defaultBranch
```

---

# 9. Creating a Local Git Repository

Start with a project folder:

```bash
mkdir my-project
cd my-project
```

Initialize Git:

```bash
git init
```

Check repository state:

```bash
git status
```

Create files:

```bash
echo "# My Projects readme file" > README.md
echo "print('hello')" > app.py
```

Check again:

```bash
git status
```

At this stage, the newly created files are **untracked**.

---

# 10. Stage and Commit

## Stage

```bash
git add README.md app.py
```

## Commit

```bash
git commit -m "Initial commit: add README and app"
```

## View history

```bash
git log --oneline
```

The basic sequence is:

```bash
git status
git add <files>
git commit -m "message"
git log --oneline
```

---

# 11. Modifying an Existing File

Suppose:

```bash
echo "print('hello again')" >> app.py
```

Now inspect the modification:

```bash
git diff
```

The lecture uses:

```text
+ added
- removed
```

Then:

```bash
git add app.py
git commit -m "Update greeting in app.py"
```

View compact history:

```bash
git log --oneline --graph
```

---

# 12. `git status` — Extremely Important

Use:

```bash
git status
```

to understand the current state of your repository.

It can show things such as:

- Current branch
- Untracked files
- Modified files
- Staged changes
- Changes ready for commit

### Lab habit

When confused, run:

```bash
git status
```

before doing anything else.

---

# 13. `git diff` vs `git diff --staged`

## Changes not yet staged

```bash
git diff
```

Shows changes in the working directory that have **not** been staged.

## Changes already staged

```bash
git diff --staged
```

Shows what is currently prepared for the next commit.

### Remember

```text
working tree changes → git diff

staged changes       → git diff --staged
```

---

# 14. `.gitignore`

A `.gitignore` file tells Git about files/directories that should be ignored.

Lecture example:

```bash
mkdir "node_modules"
printf "node_modules" > .gitignore
```

Then:

```bash
git add .gitignore
git commit -m "Add gitignore"
```

### Why?

Some generated/dependency files should not be included in version control.

---

# 15. Branches

A branch allows development to happen separately from another line of development.

## Create a branch

```bash
git branch <branch-name>
```

## Create AND switch to a new branch

```bash
git switch -c <branch-name>
```

Example:

```bash
git switch -c feature/new699branch
```

## Switch branches

```bash
git switch main
```

or:

```bash
git switch <branch-name>
```

## List branches

```bash
git branch
```

## Show current branch

```bash
git branch --show-current
```

## List local + remote branches

```bash
git branch -a
```

---

# 16. Branch Workflow

Example:

```bash
git switch -c feature/new699branch
```

Make a change:

```bash
echo "print('add file to the new branch')" >> app.py
```

Commit:

```bash
git commit -am "Add branch-specific line"
```

Then switch back:

```bash
git switch main
```

The branch-specific change does not automatically become part of `main`.

---

# 17. Merging Branches

Suppose we are on `main`:

```bash
git switch main
```

Merge another branch:

```bash
git merge <branch-name>
```

Example:

```bash
git merge feature/new699branch
```

After merging, the changes from that branch are incorporated into `main`.

## Delete a local branch

After merging:

```bash
git branch -d <branch-name>
```

## Inspect the history

```bash
git log --oneline --graph --all
```

---

# 18. Merge Conflicts

A merge conflict can occur when:

- Two branches modify the **same part of the same file**
- Git cannot determine automatically which version should remain

Git can often merge:

- Different files
- Different parts of the same file

A conflict usually occurs when both branches change overlapping content.

## What Git does

Git:

1. Pauses the merge.
2. Marks the file as conflicted.
3. Places both versions into the file using conflict markers.

You must manually decide the final contents.

---

# 19. Resolving a Merge Conflict

Conceptually:

```text
Branch A change
       \
        +---- conflict ----> manually choose final code
       /
Branch B change
```

The important skill is not memorizing a magic command; it is understanding that **Git cannot decide the desired final content for you**.

Resolution process:

1. Inspect the conflicted file.
2. Decide what the final code should be.
3. Remove/modify the conflict markers.
4. Save the corrected file.
5. Stage the resolved file.
6. Commit the resolution.
7. Continue the workflow.

The lecture also demonstrates resolving conflicts through GitHub.

---

# 20. Push Rejection

A different problem is a **push rejection**.

Typical situation:

```text
Remote repository moved ahead
          ↓
Your local repository is behind
          ↓
Your push could overwrite remote commits
          ↓
Git rejects the push
```

The lecture's example:

```text
! [rejected] main -> main (fetch first)
```

## Why does it happen?

Someone pushed to `main` after your last pull.

Your local history does not contain that new remote update.

## Important warning

> **Do not fix a normal push rejection with `git push --force`.**

The lecture specifically warns that this can delete teammates' work.

---

# 21. Git Cheat Sheet

| Task | Command |
|---|---|
| Check Git version | `git --version` |
| Show config | `git config --list` |
| Create repository | `git init` |
| Check status | `git status` |
| Stage file | `git add <file>` |
| Stage everything | `git add .` |
| Commit | `git commit -m "message"` |
| View history | `git log` |
| Compact history | `git log --oneline` |
| History graph | `git log --graph` |
| Full graph | `git log --oneline --graph --all` |
| View unstaged changes | `git diff` |
| View staged changes | `git diff --staged` |
| Show commit | `git show <commit>` |
| List branches | `git branch` |
| Current branch | `git branch --show-current` |
| All branches | `git branch -a` |
| Create branch | `git branch <name>` |
| Create + switch | `git switch -c <name>` |
| Switch branch | `git switch <name>` |
| Merge | `git merge <name>` |
| Delete local branch | `git branch -d <name>` |
| Fetch remote changes | `git fetch` |
| Download + integrate | `git pull` |
| Upload commits | `git push` |
| Temporarily save changes | `git stash` |
| Restore stash | `git stash pop` |
| Restore/discard file changes | `git restore <file>` |
| Unstage file | `git restore --staged <file>` |
| Reverse an earlier commit | `git revert <commit>` |
| Move HEAD/branch | `git reset <commit>` |
| View previous HEAD states | `git reflog` |
| Show line authorship | `git blame <filename>` |
| Create tag | `git tag <tag-name>` |
| Add remote | `git remote add origin <URL>` |
| Compare branches | `git diff main <branch-name>` |

---

# 22. `revert` vs `reset`

These are easy to confuse.

## `git revert`

```bash
git revert <commit>
```

Creates a **new commit** that reverses an earlier commit.

Think:

```text
A → B → C → reverse-C
```

The old history remains.

## `git reset`

```bash
git reset <commit>
```

Moves the branch/HEAD to another commit.

Think:

```text
A → B → C
    ↑
   reset
```

The lecture lists both commands, but the important distinction is:

> **revert creates a new reversing commit; reset moves HEAD/branch.**

---

# 23. `git stash`

Temporarily save uncommitted changes:

```bash
git stash
```

Restore them:

```bash
git stash pop
```

Useful when you have unfinished work but need to temporarily change branches or work on something else.

---

# 24. Git Object Model

Git internally stores objects.

## Blob

A **blob** stores file contents.

Important:

> A blob does not store the filename/path.

## Tree

A **tree** represents a directory listing.

It maps:

```text
name → blob
```

and can also point to other trees.

## Commit

A commit contains information including:

- One tree
- Parent
- Author
- Message

Simplified:

```text
Commit
 ├── tree
 ├── parent
 ├── author
 └── message
```

## Hashing

Git objects are named by hashes of their contents.

This helps explain:

- Content identity
- Deduplication
- Integrity
- Why changing old history changes subsequent hashes

---

# 25. Git vs GitHub vs GitLab vs Sourcetree

## Git

Git is the actual version-control software.

It runs on your machine and performs:

- commits
- branches
- merges
- history management

Git can work without an internet connection.

## GitHub

GitHub is a hosting/collaboration platform.

It provides things such as:

- Cloud-hosted repositories
- Pull Requests
- Code review
- Issues
- Discussions
- GitHub Actions
- Permissions/branch rules

## GitLab

GitLab is a competing platform.

The lecture highlights:

- Self-hosting
- CI/CD
- Container registry
- Security scanning
- Deployment
- Full lifecycle DevOps platform

## Sourcetree

Sourcetree is a GUI client.

It provides a visual interface to Git.

For example:

```text
git log --graph
```

can be viewed as a visual commit tree.

### Important distinction

> Sourcetree is **not** a version-control system. It is a front-end that runs Git commands.

---

# 26. Why GitHub When We Have Git?

| Git | GitHub |
|---|---|
| Lives on your machine | Hosts a copy online |
| Local version control | Online collaboration |
| No built-in Pull Requests | Pull Requests |
| No built-in issue tracking | Issues/project features |
| No built-in permissions system | Permissions/branch rules |
| Local commits | Shared remote repository |
| No GitHub Actions | GitHub Actions |

A useful mental model:

```text
Git
 ↓
version control

GitHub
 ↓
hosting + collaboration
```

---

# 27. Starting a GitHub Project

## Create a GitHub account

The lecture's workflow:

1. Go to GitHub.
2. Sign up.
3. Choose username.
4. Verify email.
5. Optionally enable 2FA.

## Configure local Git

```bash
git --version

git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main

git config --list
```

---

# 28. Authentication

Two approaches from the lecture:

## Option A — GitHub CLI

```bash
gh auth login
```

Follow the prompts and authenticate through the browser.

## Option B — SSH key

Generate:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Display public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Add the public key through GitHub:

```text
Settings
→ SSH and GPG keys
→ New SSH key
```

Test:

```bash
ssh -T git@github.com
```

The lecture expects a successful authentication message.

---

# 29. Creating a GitHub Repository

On GitHub:

1. Create a new repository.
2. Choose repository name.
3. Choose visibility:
   - Public
   - Private
4. Optionally add README.
5. Optionally add `.gitignore`.

## Clone

Example:

```bash
git clone git@github.com:yourname/my-project.git
cd my-project
```

Check remotes:

```bash
git remote -v
```

---

# 30. Adding a Remote

A remote connects the local repository with a remote repository.

Syntax:

```bash
git remote add origin <URL>
```

Example:

```bash
git remote add origin https://github.com/username/project.git
```

Check:

```bash
git remote -v
```

### Remember

```text
origin = conventional name for the remote repository
```

---

# 31. Pushing to GitHub

After creating/modifying files:

```bash
git status
git add .
git commit -m "added index"
```

Then push:

```bash
git push -u origin main
```

The `-u` sets the upstream relationship for the local branch.

After that, later pushes can generally use:

```bash
git push
```

---

# 32. Pulling from GitHub

To get and integrate remote changes:

```bash
git pull
```

or explicitly:

```bash
git pull origin main
```

Conceptually:

```text
Remote repository
       ↓
     pull
       ↓
Local repository
```

The lecture uses pulling when collaborators have made newer changes.

---

# 33. Collaborative Feature-Branch Workflow

A typical lecture workflow:

```bash
git switch -c feature/branchname
```

Make changes.

```bash
git add .
git commit -m "Add login validation"
```

Push the branch:

```bash
git push -u origin feature/login
```

Then create a Pull Request on GitHub.

---

# 34. Pull Requests

A Pull Request allows a proposed change to be reviewed before merging.

## Repository owner's workflow

1. Open the repository on GitHub.
2. Click **Pull requests**.
3. Open the teammate's Pull Request.
4. Select **Files changed**.
5. Inspect the changes.
6. Add comments if needed.
7. Select **Review changes → Approve**.
8. Click **Merge pull request**.
9. Confirm merge.
10. Optionally delete the feature branch.
11. Update the local repository.

### Key concept

```text
feature branch
      ↓
    push
      ↓
Pull Request
      ↓
review
      ↓
approve
      ↓
merge
      ↓
main
```

---

# 35. Two-Person Collaborative Workflow

The class workout requires:

### Student 1

- Create GitHub repository.
- Add Student 2 as collaborator.

### Both students

1. Clone repository.
2. Modify/create different files.
3. Stage changes.
4. Commit.
5. Push.
6. Pull latest changes.

### Feature branch

One teammate:

1. Creates feature branch.
2. Makes changes.
3. Pushes branch.
4. Creates Pull Request.

Repository owner:

1. Reviews changes.
2. Accepts/merges Pull Request.

After merge:

> Both students update their local `main` branch.

---

# 36. Creating a Merge Conflict for Practice

The lecture specifically asks students to practice this.

Both students should start from the latest version.

Then:

```text
Student A                 Student B
    |                         |
modify same line         modify same line
    |                         |
   push                      push
    |                         |
    +------ conflict --------+
```

The second student should:

1. Pull the latest changes.
2. Identify the conflict.
3. Manually resolve it.
4. Stage the resolved file.
5. Commit the resolution.
6. Push the resolved version.
7. Verify that both required changes are present.

---

# 37. Git Bundle — Backup Without a Server

Git can bundle an entire repository into one file.

Example:

```bash
git bundle create ../my-project.bundle --all
```

Restore options from the lecture include:

```bash
git bundle unbundle project.bundle
```

or:

```bash
git clone project.bundle my-project
```

This provides a way to move/backup the repository without a Git server.

---

# 38. High-Value Command Flows

## Flow A — New local repository

```bash
mkdir my-project
cd my-project
git init

# create files

git status
git add .
git commit -m "Initial commit"

git log --oneline
```

---

## Flow B — Modify and commit

```bash
# modify file

git status
git diff

git add <file>
git diff --staged

git commit -m "Describe change"
```

---

## Flow C — Branch and merge

```bash
git switch -c feature/test

# modify files

git add .
git commit -m "Add feature"

git switch main
git merge feature/test

git log --oneline --graph --all
```

---

## Flow D — GitHub project

```bash
git clone <URL>
cd <project>

git status
git add .
git commit -m "Update project"

git push -u origin main
```

---

## Flow E — Feature branch + Pull Request

```bash
git pull origin main

git switch -c feature/login

# modify files

git add .
git commit -m "Add login validation"

git push -u origin feature/login
```

Then:

```text
GitHub → Pull Request → Review → Approve → Merge
```

Finally:

```bash
git switch main
git pull origin main
```

---

# 39. Troubleshooting Patterns

## Problem: "Why isn't my file in the commit?"

Check:

```bash
git status
```

Then stage:

```bash
git add <file>
```

Then commit:

```bash
git commit -m "message"
```

---

## Problem: "What exactly did I change?"

Use:

```bash
git diff
```

If already staged:

```bash
git diff --staged
```

---

## Problem: "Which branch am I on?"

```bash
git branch --show-current
```

---

## Problem: "What branches exist?"

```bash
git branch
```

For local + remote:

```bash
git branch -a
```

---

## Problem: "Why did my merge stop?"

Likely a merge conflict.

Inspect the conflicted file, resolve it manually, stage it, and commit the resolution.

---

## Problem: "Why was my push rejected?"

The remote repository may contain commits that your local repository does not have.

Do **not** blindly force-push.

The lecture's recommended learning direction is to understand the push-rejection resolution workflow and update your local history.

---

# 40. Quiz Preparation — Definitions

Be able to answer these without looking:

### Q1. What is a VCS?

A system that records file changes over time so versions can be tracked, compared, restored, and managed.

### Q2. What type of VCS is Git?

Distributed Version Control System (DVCS).

### Q3. What does `git init` do?

Initializes a Git repository in a project directory.

### Q4. What does `git add` do?

Stages changes for the next commit.

### Q5. What does `git commit` do?

Records staged changes as a commit in the repository history.

### Q6. What does `git status` do?

Shows the current state of the working tree/staging area and repository information.

### Q7. What does `git diff` show?

Unstaged changes.

### Q8. What does `git diff --staged` show?

Changes that have been staged.

### Q9. Why use branches?

To develop/experiment separately without directly affecting another branch.

### Q10. What causes a merge conflict?

Git cannot automatically reconcile overlapping changes, commonly when branches modify the same part of a file.

### Q11. What is GitHub?

A hosting/collaboration platform built around Git repositories.

### Q12. Is Sourcetree a VCS?

No. It is a GUI client/front-end for Git.

### Q13. What does `git pull` do?

Downloads remote changes and integrates them into the local repository.

### Q14. What does `git push` do?

Uploads local commits to a remote repository.

### Q15. What is `origin`?

The conventional name used for a remote repository.

---

# 41. Quiz Preparation — "Differentiate"

## Git vs GitHub

```text
Git
= version-control software

GitHub
= online hosting + collaboration platform
```

## Git vs Sourcetree

```text
Git
= version-control system

Sourcetree
= GUI client that runs Git
```

## GitHub vs GitLab

Both provide repository hosting/collaboration features.

The lecture specifically highlights:

```text
GitHub → Pull Requests, Actions, collaboration

GitLab → self-hosting + broader DevOps lifecycle features
```

## `git revert` vs `git reset`

```text
revert → creates a new commit reversing an earlier commit

reset  → moves HEAD/branch to another commit
```

## `git diff` vs `git diff --staged`

```text
diff        → unstaged changes

diff --staged → staged changes
```

## `git fetch` vs `git pull`

```text
fetch → download remote changes

pull  → download + integrate
```

---

# 42. Lab-Test Must-Know Commands

If you have only a few minutes before a practical, memorize this block:

```bash
git init
git status

git add .
git commit -m "message"

git log --oneline
git log --oneline --graph --all

git diff
git diff --staged

git branch
git branch --show-current
git branch -a

git switch -c feature/name
git switch main
git merge feature/name

git remote -v
git remote add origin <URL>

git push -u origin main
git pull origin main

git push -u origin feature/name

git stash
git stash pop

git restore <file>
git restore --staged <file>

git revert <commit>
git reset <commit>

git reflog
```

---

# 43. Lab-Test Practice Task 1 — Basic Git

Create a project:

```bash
mkdir git-practice
cd git-practice
git init
```

Create:

```text
README.md
app.py
```

Then:

1. Check status.
2. Stage both files.
3. Commit them.
4. Display commit history.
5. Modify `app.py`.
6. Display the difference.
7. Stage the modification.
8. Display staged difference.
9. Commit again.
10. Display compact history.

### Commands you should be able to produce

```bash
git status
git add .
git commit -m "Initial commit"
git log --oneline

git diff
git add app.py
git diff --staged
git commit -m "Update app"
git log --oneline
```

---

# 44. Lab-Test Practice Task 2 — Branching

Starting from a repository:

1. Create a feature branch.
2. Switch to it.
3. Modify a file.
4. Commit the change.
5. Switch back to `main`.
6. Observe the difference.
7. Merge the feature branch.
8. Display the graph.
9. Delete the feature branch.

Expected workflow:

```bash
git switch -c feature/test

# edit

git add .
git commit -m "Feature change"

git switch main
git merge feature/test

git log --oneline --graph --all

git branch -d feature/test
```

---

# 45. Lab-Test Practice Task 3 — GitHub

Starting from a GitHub repository:

1. Clone it.
2. Enter the directory.
3. Check the remote.
4. Modify/create a file.
5. Commit.
6. Push to `main`.

Core commands:

```bash
git clone <URL>
cd <project>

git remote -v

git add .
git commit -m "Update project"
git push -u origin main
```

---

# 46. Lab-Test Practice Task 4 — Feature Branch + PR

```bash
git pull origin main
git switch -c feature/test

# modify files

git add .
git commit -m "Add feature"
git push -u origin feature/test
```

Then on GitHub:

```text
feature/test
     ↓
Pull Request
     ↓
Review
     ↓
Approve
     ↓
Merge
```

Then locally:

```bash
git switch main
git pull origin main
```

---

# 47. Lab-Test Practice Task 5 — Merge Conflict

Two people modify the same line.

After one person pushes, the other should:

```bash
git pull origin main
```

Then:

1. Open the conflicted file.
2. Identify Git's conflict markers.
3. Decide the final desired contents.
4. Remove the conflict markers.
5. Save the file.
6. Stage it.
7. Commit the resolution.
8. Push.

Core commands after manual editing:

```bash
git add <resolved-file>
git commit -m "Resolve merge conflict"
git push
```

---

# 48. Assignment 1 — Website Version Control

The lecture asks you to:

1. Create a Git repository for your HTML website from Assignment 1.
2. Create the project folder using the shell.
3. Initialize it as a Git repository.
4. Add HTML files.
5. Create an initial commit.
6. Modify the HTML page.
7. Create another commit.
8. Create a new branch for an alternative version.
9. Make different changes on that branch.
10. Commit the branch changes.
11. Switch between branches.
12. Observe different webpage versions.
13. Display Git commit history.

### Minimum skills being tested

```text
init
add
commit
branch
switch
log
```

---

# 49. Assignment 2 — Version Control of a Project

Create a new project and add the CSV/text files from Lecture 1.

You should be able to:

1. Initialize the repository.
2. Add files.
3. Make an initial commit.
4. Make further changes and commits.
5. Create a new branch.
6. Make different changes on both branches.
7. Add files/folders on the new branch.
8. Switch between branches.
9. Compare files/folders.
10. Compare commits/changes.
11. Merge the branches.
12. Observe whether Git merges automatically or produces a conflict.
13. Display the final history.

---

# 50. Rapid Revision — One Page

## Core idea

```text
VCS
 ↓
tracks versions/history

Git
 ↓
distributed VCS

GitHub
 ↓
hosting + collaboration
```

## Core workflow

```text
EDIT
 ↓
git status
 ↓
git diff
 ↓
git add
 ↓
git diff --staged
 ↓
git commit
 ↓
git log
```

## Branch workflow

```text
git switch -c feature/x
 ↓
edit
 ↓
git add .
 ↓
git commit
 ↓
git switch main
 ↓
git merge feature/x
```

## Remote workflow

```text
clone
 ↓
edit
 ↓
add
 ↓
commit
 ↓
push
```

## Collaboration

```text
main
 ↓
feature branch
 ↓
push
 ↓
Pull Request
 ↓
review
 ↓
merge
 ↓
pull updated main
```

## Conflict

```text
same part changed
       ↓
merge/pull
       ↓
conflict
       ↓
manual resolution
       ↓
git add
       ↓
commit
       ↓
push
```

---

# 51. Final Self-Check

Before the quiz/lab test, make sure you can explain each of these **without notes**:

- [ ] What is version control?
- [ ] Local vs centralised vs distributed VCS
- [ ] Why Git is distributed
- [ ] Working directory vs staging area vs repository
- [ ] `git init`
- [ ] `git status`
- [ ] `git add`
- [ ] `git commit`
- [ ] `git diff`
- [ ] `git diff --staged`
- [ ] `.gitignore`
- [ ] Creating/switching branches
- [ ] Merging
- [ ] Merge conflicts
- [ ] Push rejection
- [ ] Why not blindly use `git push --force`
- [ ] `git fetch` vs `git pull`
- [ ] `git push`
- [ ] `git stash`
- [ ] `git restore`
- [ ] `git revert` vs `git reset`
- [ ] `git reflog`
- [ ] Git object model: blob/tree/commit
- [ ] Git vs GitHub
- [ ] GitHub vs GitLab
- [ ] Sourcetree
- [ ] Remote `origin`
- [ ] GitHub authentication
- [ ] `git clone`
- [ ] `git remote add origin`
- [ ] `git push -u origin main`
- [ ] Feature branches
- [ ] Pull Requests
- [ ] Collaborative workflow
- [ ] Manual merge-conflict resolution

---

# 52. Ultra-Short Memory Map

```text
                    VERSION CONTROL
                          |
                         Git
                          |
          +---------------+---------------+
          |               |               |
       History         Branches        Collaboration
          |               |               |
       commit           switch          GitHub
       log              merge              |
       diff             conflict            |
                                      Pull Request
                                      review/merge


Git local workflow:
working directory
       ↓ add
staging area
       ↓ commit
repository


Remote workflow:
local repository
       ↓ push
GitHub
       ↓ pull
local repository
```

---

## Source / Further Reading

The lecture references:

- **Pro Git** — Scott Chacon and Ben Straub
- **Version Control with Git** — Prem Kumar Ponuthurai and Jon Loeliger
- Git book: `https://git-scm.com/book/en/v2`
- GitHub documentation: `https://docs.github.com/`

---

## Note on Scope

These notes intentionally follow the **L3 lecture material** and its terminology. Commands and explanations are organized to emphasize what is likely to matter for **quiz recall and hands-on lab execution**, rather than adding unrelated Git features.
