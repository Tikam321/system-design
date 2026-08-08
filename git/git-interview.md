
# Git Interview Questions & Answers

A complete guide to Git interview questions — covering fundamentals, branching, merging vs rebasing, undoing changes, collaboration workflows, and troubleshooting.

---

## Table of Contents
1. [Git Basics](#git-basics)
2. [Branching & Merging](#branching--merging)
3. [Rebasing](#rebasing)
4. [Undoing Changes](#undoing-changes)
5. [Remote Repositories & Collaboration](#remote-repositories--collaboration)
6. [Stashing & Cleaning](#stashing--cleaning)
7. [History & Logs](#history--logs)
8. [Tags & Releases](#tags--releases)
9. [Conflict Resolution](#conflict-resolution)
10. [Advanced / Internals](#advanced--internals)
11. [Git Workflows](#git-workflows)
12. [Troubleshooting & Scenario-Based](#troubleshooting--scenario-based)

---

## Git Basics

### 1. What is Git? How is it different from a centralized VCS like SVN?
Git is a **distributed version control system (DVCS)** — every developer has a full copy of the entire repository history locally, not just the latest snapshot. This means you can commit, branch, and view history entirely offline, and there's no single point of failure (unlike centralized systems like SVN, where the central server holds the only full history).

### 2. What is the difference between Git and GitHub?
**Git** is the version control tool/software itself — it works locally and doesn't require any online service. **GitHub** (along with GitLab, Bitbucket) is a cloud-based hosting platform for Git repositories, adding collaboration features on top: pull requests, issue tracking, CI/CD, code review, access control.

### 3. What are the three main states of a file in Git?
- **Modified** — the file has been changed but not yet staged
- **Staged** — the change is marked to be included in the next commit (via `git add`)
- **Committed** — the change is safely stored in the local `.git` repository history

### 4. What is the staging area (index)?
An intermediate area between your working directory and the repository history — it lets you selectively choose exactly which changes go into the next commit, rather than committing every modified file at once.

### 5. What is a `.gitignore` file used for?
Specifies intentionally untracked files/patterns (build artifacts, dependencies like `node_modules`, secrets, IDE config, `.env` files) that Git should never track or stage, keeping the repository clean and preventing accidental commits of sensitive or generated files.

### 6. What is the difference between `git init` and `git clone`?
- **`git init`** — creates a brand-new, empty Git repository in the current directory
- **`git clone <url>`** — copies an **existing** remote repository (full history included) to your local machine, and automatically sets up the remote (`origin`) connection

### 7. What is a commit hash (SHA)?
A unique 40-character SHA-1 hash (or SHA-256 in newer Git versions) that identifies a specific commit, computed from the commit's content, metadata, author, timestamp, and parent commit(s). Since it's content-based, even a tiny change produces a completely different hash — this is also what makes Git tamper-evident.

---

## Branching & Merging

### 8. What is a branch in Git?
A lightweight, movable pointer to a specific commit. Creating a branch doesn't copy any files — it just creates a new named reference, making branching in Git extremely fast and cheap compared to older VCS tools.

### 9. What is `HEAD` in Git?
A pointer to the **current branch's latest commit** — essentially "where you are right now" in the repository. Most of the time `HEAD` points to a branch (e.g., `main`), which in turn points to a commit; in a **detached HEAD** state, `HEAD` points directly to a specific commit instead of a branch.

### 10. What is the difference between `git merge` and `git rebase`?
Covered in detail in the [Rebasing](#rebasing) section — but at a high level:
- **`merge`** combines two branch histories by creating a new "merge commit" with two parents, preserving the exact history of both branches.
- **`rebase`** replays your branch's commits on top of another branch, creating a linear history without a merge commit — but it rewrites commit hashes.

### 11. What are the different types of merges in Git?
- **Fast-forward merge**: happens when the target branch has no new commits since the feature branch diverged — Git simply moves the branch pointer forward, no merge commit needed.
- **Three-way merge**: happens when both branches have diverged with new commits — Git creates a new merge commit with two parent commits, using the common ancestor to determine changes from each side.
- **Squash merge**: combines all commits from a feature branch into a **single** commit on the target branch, discarding individual commit history — common for keeping `main`'s history clean.

### 12. What is a fast-forward merge? When does it NOT happen?
A fast-forward merge occurs when the current branch's pointer can simply move forward to the target branch's latest commit, because there's a direct linear path between them (no divergent commits). It does **not** happen if the current branch has its own new commits since diverging — in that case, Git must create a real merge commit (or you use `rebase` first to make it linear again).

### 13. How do you delete a branch, locally and remotely?
```bash
git branch -d branch-name          # delete local branch (safe — only if merged)
git branch -D branch-name          # force delete local branch (even if unmerged)
git push origin --delete branch-name   # delete the branch on the remote
```

### 14. What is the difference between `git branch` and `git checkout -b`?
- **`git branch <name>`** — creates a new branch, but stays on your current branch
- **`git checkout -b <name>`** (or `git switch -c <name>` in modern Git) — creates a new branch **and** switches to it in one command

---

## Rebasing

### 15. Explain `git rebase` with an example.
Rebase takes the commits from your current branch and "replays" them one by one on top of another branch's latest commit, effectively rewriting your branch's history to look like it was built starting from the newer base — resulting in a clean, linear commit history.

```bash
git checkout feature-branch
git rebase main
# your feature commits are now replayed on top of main's latest commit
```

### 16. What is the key difference between merge and rebase in terms of history?
- **Merge** preserves the true, non-linear history — you can see exactly when and how branches diverged and came back together (via the merge commit), but the log can get cluttered with many merge commits in active repos.
- **Rebase** creates a clean, linear history as if all work happened sequentially — easier to read (`git log --oneline` looks like a straight line) — but it **rewrites commit hashes**, which is dangerous on shared/public branches.

### 17. Why is rebasing a shared/public branch dangerous?
Rebase creates **entirely new commits** with new SHA hashes (even though the content is identical) — so if others have already pulled the original commits and you rebase and force-push, their local history now diverges from the remote, causing confusing conflicts and duplicate commits when they try to pull/push. **Golden rule: never rebase commits that have already been pushed and shared with others.**

### 18. What is an interactive rebase (`git rebase -i`)? What can you do with it?
Opens an editable list of commits, letting you rewrite history before it's finalized:
- **`pick`** — keep the commit as-is
- **`reword`** — keep the commit's changes but edit the commit message
- **`squash`/`fixup`** — combine a commit into the previous one (squash keeps both messages, fixup discards the squashed commit's message)
- **`drop`** — remove a commit entirely
- **`edit`** — pause at that commit to amend it or split it into multiple commits
- Reordering lines in the list also **reorders the commits**

```bash
git rebase -i HEAD~5   # interactively edit the last 5 commits
```

### 19. What is a rebase conflict, and how do you resolve it?
Since rebase replays commits one at a time, a conflict can occur at any individual commit being replayed (not just once, like in a merge). Resolve each conflict, then:
```bash
git add <resolved-files>
git rebase --continue
# or, to abandon the whole rebase and go back to the pre-rebase state:
git rebase --abort
# or, to skip the current commit entirely:
git rebase --skip
```

---

## Undoing Changes

### 20. What is the difference between `git reset`, `git revert`, and `git checkout` (for undoing changes)?

| Command | Effect | Rewrites history? | Safe on shared branches? |
|---|---|---|---|
| `git reset` | Moves the branch pointer backward (optionally modifying working directory/staging) | Yes | No |
| `git revert` | Creates a **new commit** that undoes a previous commit's changes | No (adds a new commit) | Yes |
| `git checkout <commit> -- <file>` | Restores a specific file's content from a given commit, into the working directory | No | Yes |

### 21. What are the three modes of `git reset`?
```bash
git reset --soft <commit>    # moves HEAD/branch pointer only; changes stay staged
git reset --mixed <commit>   # (default) moves HEAD + unstages changes; files keep their edits
git reset --hard <commit>    # moves HEAD + unstages + discards all working directory changes — destructive!
```

### 22. When would you use `git revert` instead of `git reset`?
Use `revert` when the commit you want to undo has **already been pushed/shared** — since it creates a new "undo" commit rather than rewriting history, it's safe for everyone else who already has the original commit. `reset` is fine for **local, unpushed** commits you want to clean up before sharing.

### 23. How do you undo the last commit but keep the changes in your working directory?
```bash
git reset --soft HEAD~1     # keeps changes staged
# or
git reset HEAD~1            # (mixed, default) keeps changes unstaged
```

### 24. How do you completely discard uncommitted local changes to a file?
```bash
git checkout -- filename
# modern Git equivalent:
git restore filename
```

### 25. How do you amend the last commit (message or content)?
```bash
git commit --amend -m "new message"
# to add forgotten files to the last commit:
git add forgotten-file.txt
git commit --amend --no-edit
```
**Caution**: like rebase, `--amend` rewrites the last commit's hash — avoid amending commits already pushed and shared, unless you're prepared to force-push and coordinate with your team.

---

## Remote Repositories & Collaboration

### 26. What is the difference between `git fetch` and `git pull`?
- **`git fetch`** — downloads new commits/branches from the remote into your local repo's remote-tracking branches (e.g., `origin/main`), but does **not** merge them into your current working branch. Safe, non-destructive — lets you review changes before integrating.
- **`git pull`** — is essentially `git fetch` **+** `git merge` (or `+ rebase`, if configured) in one step — immediately integrates the fetched changes into your current branch.

### 27. What is the difference between `origin` and `upstream`?
- **`origin`** — the default name Git gives to the remote repository you cloned from (usually your own fork or the main team repo).
- **`upstream`** — a conventional name (not automatic — you add it manually) for the **original** repository, typically used in a fork-based workflow where `origin` is your personal fork and `upstream` is the original project you forked from, so you can pull in the latest changes from the source.

```bash
git remote add upstream https://github.com/original-owner/repo.git
git fetch upstream
git merge upstream/main
```

### 28. What is a pull request (PR) / merge request (MR)?
A request (a GitHub/GitLab/Bitbucket platform feature, not a core Git concept) to merge changes from one branch (often a fork or feature branch) into another, typically triggering code review, automated CI checks, and discussion before the merge is approved and completed.

### 29. What happens if you `git push` and someone else has already pushed changes you don't have?
Git **rejects the push** with a "non-fast-forward" or "updates were rejected" error, since your local branch is now behind the remote. You need to `git pull` (or `fetch` + `rebase`/`merge`) first to integrate the remote changes, resolve any conflicts, and then push again.

### 30. What is `git push --force` vs `git push --force-with-lease`? Why is the latter safer?
- **`--force`** — overwrites the remote branch with your local branch unconditionally, potentially **destroying commits** that someone else pushed in the meantime (if you didn't have them locally when you force-pushed).
- **`--force-with-lease`** — only force-pushes if the remote branch is still in the state you last saw it (i.e., no one else has pushed since your last fetch) — otherwise it fails safely, protecting against accidentally overwriting a teammate's work.

### 31. How do you check which remote branches exist?
```bash
git branch -r          # list remote-tracking branches
git branch -a          # list both local and remote branches
git remote -v          # list configured remotes and their URLs
```

---

## Stashing & Cleaning

### 32. What is `git stash`? When would you use it?
Temporarily saves your uncommitted changes (both staged and unstaged, by default not untracked files) onto a stack, reverting your working directory to match `HEAD` — useful when you need to quickly switch branches (e.g., to fix an urgent bug) without committing half-finished work.

```bash
git stash                    # save current changes
git stash list                # view all stashed entries
git stash pop                 # apply the most recent stash AND remove it from the stack
git stash apply               # apply the most recent stash but KEEP it in the stack
git stash drop                # delete a stash without applying it
git stash -u                  # also stash untracked files
```

### 33. What is the difference between `git stash pop` and `git stash apply`?
`pop` applies the stashed changes to your working directory **and removes** that stash from the stash list. `apply` does the same but **keeps** the stash in the list, useful if you want to apply the same stash to multiple branches.

### 34. What does `git clean` do?
Removes **untracked files** from the working directory (files that are not staged/committed and not covered by `.gitignore` in the way you specify). Commonly used to clean up build artifacts or accidental stray files.

```bash
git clean -n     # dry run — shows what WOULD be deleted, without deleting
git clean -f     # actually deletes untracked files
git clean -fd    # also removes untracked directories
```

---

## History & Logs

### 35. How do you view commit history? What are useful `git log` flags?
```bash
git log                          # full commit history
git log --oneline                # condensed, one line per commit
git log --graph --oneline --all  # visual branch/merge graph
git log -p                       # show diffs for each commit
git log --author="name"          # filter by author
git log --since="2 weeks ago"    # filter by date
git log -- filename              # history of changes to a specific file
```

### 36. What is the difference between `git diff` and `git diff --staged`?
- **`git diff`** — shows changes in the working directory that are **not yet staged**
- **`git diff --staged`** (or `--cached`) — shows changes that **are staged**, about to be included in the next commit

### 37. What is `git blame` used for?
Shows, line by line, which commit and author last modified each line of a file — useful for tracking down when/why a specific line of code was introduced, often as a starting point for investigating a bug's origin.

### 38. What is `git bisect`? How does it work?
A binary search tool to find which specific commit introduced a bug. You mark a known-good commit and a known-bad commit, and Git checks out commits in between, asking you to mark each as good/bad, narrowing down the culprit commit in O(log n) steps instead of checking every commit one by one.

```bash
git bisect start
git bisect bad                 # current commit is broken
git bisect good v1.0           # this earlier commit was fine
# Git checks out a midpoint commit; you test it and mark:
git bisect good   # or
git bisect bad
# ...repeat until Git identifies the exact bad commit
git bisect reset  # end the session, return to original branch
```

---

## Tags & Releases

### 39. What is a Git tag? What's the difference between lightweight and annotated tags?
A tag marks a specific commit as significant, typically used for release versions (`v1.0.0`).
- **Lightweight tag** — just a named pointer to a commit, like a branch that doesn't move; no extra metadata.
- **Annotated tag** — a full Git object with its own metadata (tagger name, date, message), and can be GPG-signed — recommended for actual releases.

```bash
git tag v1.0.0                              # lightweight
git tag -a v1.0.0 -m "Release version 1.0"  # annotated
git push origin v1.0.0                      # tags aren't pushed automatically — must push explicitly
git push origin --tags                      # push all tags
```

---

## Conflict Resolution

### 40. What causes a merge conflict?
Occurs when Git can't automatically reconcile changes because **the same lines of a file** were modified differently on both branches being merged (or one branch modified a file the other deleted, etc.) — Git needs a human decision on which change to keep.

### 41. How do you resolve a merge conflict?
1. Git marks the conflicting sections in the file with conflict markers:
```
<<<<<<< HEAD
your current branch's version
=======
the incoming branch's version
>>>>>>> feature-branch
```
2. Manually edit the file to keep the correct content, removing the conflict markers
3. Stage the resolved file: `git add <file>`
4. Complete the merge: `git commit` (for a merge) or `git rebase --continue` (for a rebase)

### 42. What tools can help resolve merge conflicts more easily?
- `git mergetool` — launches a configured visual diff/merge tool (VS Code, Meld, KDiff3, Beyond Compare)
- IDE-integrated conflict resolution (VS Code, IntelliJ show conflicts inline with "Accept Current/Incoming/Both" buttons)
- `git diff` to review conflicting hunks before deciding

---

## Advanced / Internals

### 43. What are the main Git object types?
- **Blob** — stores raw file content (no filename/metadata, just content, addressed by content hash)
- **Tree** — represents a directory; a list of blobs and other trees (subdirectories), each with a filename and permissions
- **Commit** — points to a single tree (a full snapshot), references its parent commit(s), and includes author, timestamp, and message
- **Tag** — (for annotated tags) a reference object pointing to a commit, with extra metadata

### 44. How does Git store changes internally — as diffs or full snapshots?
As **full snapshots**, not diffs (a common misconception). Every commit points to a tree representing the complete state of the project at that point. Git is efficient about storage because unchanged files between commits are stored as the same blob (referenced by the same content hash) rather than duplicated — so unchanged files cost effectively nothing extra, even though conceptually each commit is a full snapshot.

### 45. What is the difference between `git merge --no-ff` and a regular merge?
By default, Git performs a fast-forward merge when possible (no merge commit created). `--no-ff` **forces** a merge commit to be created even when a fast-forward would have been possible — useful for preserving a clear record that a feature branch existed and was merged, rather than the history looking like the work happened directly on the main branch.

### 46. What is a detached HEAD state? Is it dangerous?
Occurs when you check out a specific commit (not a branch) directly — `HEAD` now points straight to that commit instead of to a branch reference. It's not inherently dangerous, but **any new commits made in this state aren't attached to any branch** — if you switch away without creating a branch first, those commits become unreachable and eventually get garbage collected. To keep new work, create a branch from the detached state: `git checkout -b new-branch-name`.

### 47. What is `git reflog`? How can it save you from a "lost" commit?
A log of every place `HEAD` has pointed to, including commits from resets, rebases, amends, and even deleted branches — even ones no longer reachable from any branch. It's a safety net for recovering "lost" work.

```bash
git reflog                      # view the full HEAD history
git checkout <sha-from-reflog>  # go back to a specific prior state
git branch recovered-work <sha> # create a branch to save recovered commits
```

### 48. What is the difference between `.git` folder and `.gitignore` file?
- **`.git`** — a hidden directory containing the **entire actual repository**: all commits, branches, tags, config, object database. Deleting it destroys all local Git history (though the working files themselves remain).
- **`.gitignore`** — a plain text file listing patterns of files/folders Git should never track — has nothing to do with the repository's internal storage.

### 49. What are Git hooks?
Scripts that Git automatically runs at specific points in the workflow (e.g., `pre-commit`, `pre-push`, `post-merge`), stored in `.git/hooks/`. Commonly used for linting/formatting before a commit, running tests before a push, or enforcing commit message conventions. Since `.git/hooks` isn't versioned/shared by default, teams often use tools like **Husky** (JS) or **pre-commit** (Python) to manage and share hook configuration via the repo itself.

### 50. What is `git cherry-pick`?
Applies the changes from a **specific commit** (from any branch) onto your current branch, without merging the entire branch — useful for pulling in a single bug fix or feature from another branch without bringing in unrelated history.

```bash
git cherry-pick <commit-sha>
```

---

## Git Workflows

### 51. What is Git Flow? Briefly describe its branch structure.
A branching model with defined long-lived and short-lived branches:
- **`main`/`master`** — always reflects production-ready code
- **`develop`** — integration branch for ongoing development
- **`feature/*`** — individual feature branches, branched from and merged back into `develop`
- **`release/*`** — prepares a release (final testing/bugfixes) before merging into both `main` and `develop`
- **`hotfix/*`** — urgent production fixes, branched from `main`, merged into both `main` and `develop`

### 52. What is trunk-based development? How does it differ from Git Flow?
Trunk-based development uses a **single long-lived branch** (`main`/`trunk`), with developers committing small, frequent changes directly to it (or via very short-lived feature branches merged quickly) — often paired with **feature flags** to hide incomplete work in production. It favors continuous integration and fast iteration over Git Flow's more structured, longer-lived branch hierarchy — increasingly the preferred approach for teams practicing CI/CD.

### 53. What is a fork, and how is it different from a branch?
A **fork** is a full copy of an entire repository under a different owner's account (typically on a platform like GitHub) — common in open-source contribution workflows where you don't have write access to the original repo. A **branch** exists within a single repository and doesn't require copying ownership — used for parallel work within a team that shares repo access.

### 54. What is a typical open-source contribution workflow using forks?
1. Fork the original repository on GitHub
2. Clone your fork locally
3. Add the original repo as an `upstream` remote
4. Create a feature branch, make changes, commit
5. Push to your fork (`origin`)
6. Open a pull request from your fork's branch to the original repo's `main`/`develop`
7. Periodically sync your fork with `upstream` to stay current

---

## Troubleshooting & Scenario-Based

### 55. You accidentally committed to the wrong branch. How do you fix it?
```bash
# Move the commit to the correct branch, and remove it from the current one
git branch correct-branch          # create the correct branch at current position
git reset --hard HEAD~1            # remove the commit from the wrong branch
git checkout correct-branch        # switch to the correct branch (commit is now there)
```

### 56. You need to combine the last 3 commits into one. How?
```bash
git rebase -i HEAD~3
# in the editor, change 'pick' to 'squash' (or 's') for the 2nd and 3rd commits
```

### 57. How do you recover a deleted branch?
```bash
git reflog                          # find the SHA of the branch's last commit
git branch recovered-branch <sha>   # recreate the branch pointing to that commit
```

### 58. How do you find out which commit introduced a specific line of code (or deleted a specific line)?
```bash
git log -S "specific code string" -- path/to/file    # find commits that added/removed this exact string
git log --follow -- path/to/file                     # track a file's history even across renames
```

### 59. Your `.gitignore` isn't working for a file that's already tracked — why, and how do you fix it?
`.gitignore` only prevents **untracked** files from being added — if a file was already committed before being added to `.gitignore`, Git continues tracking it regardless. Fix by removing it from tracking (without deleting the actual file):
```bash
git rm --cached path/to/file
git commit -m "Stop tracking file"
```

### 60. How would you split a large, messy commit into multiple smaller, cleaner commits?
```bash
git rebase -i HEAD~1     # or wherever the messy commit is
# mark it as 'edit' in the interactive rebase list
git reset HEAD~1          # unstage the commit's changes, keeping them in working directory
git add <specific files>  # stage and commit in logical chunks
git commit -m "First logical piece"
git add <other files>
git commit -m "Second logical piece"
git rebase --continue
```

### 61. What's the difference between resolving a conflict during a `merge` vs during a `rebase`, in terms of workflow?
- **Merge conflict**: resolve once, then a single `git commit` finishes the merge.
- **Rebase conflict**: may need to resolve conflicts **multiple times** — once per commit being replayed — using `git rebase --continue` after each resolution, since each original commit is reapplied individually.

### 62. How do you compare two branches to see what's different?
```bash
git diff branch1..branch2              # full diff of changes between branches
git log branch1..branch2 --oneline     # commits in branch2 not in branch1
git diff --stat branch1 branch2        # summary of changed files only
```

### 63. You need to work on a hotfix urgently but have uncommitted changes for an unrelated feature. What do you do?
```bash
git stash                     # save current unrelated work
git checkout main
git checkout -b hotfix/urgent-fix
# ... fix the issue, commit, push, PR ...
git checkout original-feature-branch
git stash pop                 # resume where you left off
```

---

## Quick Cheat-Sheet Summary

| Concept | Key Point |
|---|---|
| `fetch` vs `pull` | Fetch downloads only; pull = fetch + merge/rebase |
| `merge` vs `rebase` | Merge preserves branch history; rebase creates linear history but rewrites hashes |
| `reset` vs `revert` | Reset rewrites history (local only); revert adds a safe new "undo" commit |
| `--force` vs `--force-with-lease` | Lease version fails safely if remote has new commits you don't have |
| Rebase golden rule | Never rebase commits already pushed/shared |
| `stash` vs `commit` | Stash = temporary, uncommitted save; use for quick context switches |
| `reflog` | Recovers "lost" commits even after reset/rebase/branch deletion |
| Git objects | Blob (content), Tree (directory), Commit (snapshot + metadata) |
| Detached HEAD | New commits are unreachable unless you branch from that point |
| Git Flow vs Trunk-based | Structured long-lived branches vs single branch + short-lived features/flags |

---

*Good luck with your interview! A very common practical test: interviewers describe a messy git history scenario (wrong branch, needs squashing, lost commit) and ask you to talk through the exact commands to fix it — practice narrating the "why" behind each command, not just memorizing syntax.*
