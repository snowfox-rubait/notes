# Git — Reference and Hygiene Guide

> A personal reference for git usage, mental models, and long-term habits.  
> Written to be re-read when rusty, not just when stuck.

---

## Table of Contents

1. [The Mental Model](#1-the-mental-model)
2. [Initial Setup](#2-initial-setup)
3. [Core Daily Commands](#3-core-daily-commands)
4. [Commit Hygiene](#4-commit-hygiene)
5. [Branching Hygiene](#5-branching-hygiene)
6. [Undoing Things](#6-undoing-things)
7. [Viewing History](#7-viewing-history)
8. [Working with Remotes](#8-working-with-remotes)
9. [Merge Conflicts](#9-merge-conflicts)
10. [Team Workflow (Pull Requests)](#10-team-workflow-pull-requests)
11. [Git for Dotfiles and Notes](#11-git-for-dotfiles-and-notes)
12. [Conventional Commits Reference](#12-conventional-commits-reference)
13. [Habits Checklist](#13-habits-checklist)

---

## 1. The Mental Model

Before memorizing commands, get these concepts locked in.

### Git is a timeline, not a backup tool

Every `git commit` is a **snapshot** of your project at that moment. Git stores the full timeline of all snapshots. You can travel to any point in that timeline at any time.

```
A ── B ── C ── D ── E   (main branch, timeline moves forward)
         ↑
     you are here
```

### Git stores deltas, not full copies

Git does not save a full copy of every file on every commit. It saves only **what changed**. So 100 commits on a text file costs almost nothing in disk space. Binary files (images, compiled binaries) are the exception — git can't delta-compress those.

### The three areas

```
Working Directory  →  Staging Area (Index)  →  Repository (History)
   (your files)          (git add)               (git commit)
```

- **Working Directory** — files as they exist on disk right now
- **Staging Area** — changes you've selected to include in the next commit
- **Repository** — the permanent history of all commits

You control what goes into each commit by staging selectively. This is intentional — it lets you make small focused commits even if you changed many things at once.

---

## 2. Initial Setup

Run this once on any new machine before doing anything else.

```bash
git config --global user.name "Rubait"
git config --global user.email "you@rubait.top"
git config --global core.editor "nvim"
git config --global init.defaultBranch main
git config --global color.ui auto
```

Verify:
```bash
git config --list
```

---

## 3. Core Daily Commands

### Start a new repo
```bash
git init
```

### Check status (run this constantly)
```bash
git status
```

### Stage changes
```bash
git add filename.md          # stage a specific file
git add .                    # stage everything in current directory
git add -p                   # interactive — choose which chunks to stage (powerful)
```

### Commit
```bash
git commit -m "add: localsend keybinding"
```

### See what changed before staging
```bash
git diff                     # unstaged changes
git diff --staged            # staged changes (what will go into next commit)
```

### See history
```bash
git log --oneline            # compact view
git log --oneline --graph    # with branch visualization
```

---

## 4. Commit Hygiene

This is the most important section for long-term sanity.

### Rule 1: One commit = one logical change

Every commit should do **exactly one thing**. If you can't describe it in one short sentence, it's probably two commits.

```
BAD:  "fixed arduino config and also updated browser notes and removed spotify"
GOOD: "fix: arduino exec flags for wayland"
      "update: browser section with cons for each option"
      "remove: spotify from install list"
```

Small commits mean surgical reverts. Big commits mean painful ones.

### Rule 2: Imperative mood in commit messages

A commit message completes the sentence: *"If applied, this commit will ___"*

```
GOOD: "add localsend keybinding"
BAD:  "added localsend keybinding"
BAD:  "adding localsend keybinding"
```

It also inverts cleanly:
```
"add localsend keybinding"   →  revert = "remove localsend keybinding"   ✅
"added localsend keybinding" →  revert = "removed added localsend..." ??  ❌
```

### Rule 3: Commit often, don't let changes pile up

The longer you wait, the harder it is to write a clean focused message. Commit every time you reach a logical stopping point — even if it's small.

### Rule 4: Never commit broken state to main

If your change is incomplete or broken, either don't commit yet, or commit to a branch. `main` should always be in a working state.

### Rule 5: Use `git add -p` for precision

When you've changed multiple things in a session and want to make separate clean commits:

```bash
git add -p filename.md
```

Git shows you each changed chunk and asks: stage this? (`y`/`n`/`s` to split further). Lets you craft exactly the commit you want.

---

## 5. Branching Hygiene

### Why branch even when working solo

- Keeps `main` always clean and working
- Makes it easy to abandon an experiment without messing up your stable state
- Builds the exact muscle memory required for team work

### The workflow

```bash
# start new work
git checkout -b type/short-description    # create and switch to branch

# do your work and commit normally
git add file
git commit -m "add: thing"

# when done, merge back into main
git checkout main
git merge type/short-description

# clean up
git branch -d type/short-description
```

### Branch naming conventions

```
feat/arduino-setup          # new feature or content
fix/capslock-config         # correcting something wrong
update/browser-notes        # modifying existing content
remove/spotify-entry        # deleting something
experiment/new-structure    # trying something out, may be abandoned
```

### View all branches

```bash
git branch              # local branches
git branch -a           # local + remote branches
```

---

## 6. Undoing Things

This is where most people panic. Don't. Git makes undoing things safe by design.

### Undo last commit (keep the changes, just uncommit)
```bash
git reset HEAD~1
```
Your changes are still in your working directory. Nothing is lost.

### Undo last commit completely (discard changes too)
```bash
git reset --hard HEAD~1
```
> ⚠️ This one is destructive. The changes are gone. Only use when you're certain.

### Safely revert a commit (the right way)
```bash
git revert <commit-hash>
```
This **adds a new commit** that undoes the target commit's changes. Your timeline only moves forward. Nothing in history is touched. This is the command to default to.

```
Before:  A ── B ── C (bad) ── D ── E
After:   A ── B ── C (bad) ── D ── E ── F (reverts C)
```

### Revert a commit from the middle of history

Same command. Git surgically undoes just that commit regardless of where it is in the timeline.

```bash
git log --oneline           # find the hash of the bad commit
git revert a3f92c1          # revert just that one
```

If a later commit touched the same lines, git will raise a conflict. Resolve it manually (see section 9), then `git add` and `git commit`.

### Discard unstaged changes to a file
```bash
git checkout -- filename.md
```
> ⚠️ Destructive. Unsaved changes to that file are gone.

### Unstage a file (keep changes, just remove from staging)
```bash
git restore --staged filename.md
```

### See what a file looked like at any past commit
```bash
git show HEAD~3:filename.md           # 3 commits ago
git show a3f92c1:filename.md          # at a specific commit
```

### Safe vs destructive commands at a glance

| Command | Safe? | What it does |
|---|---|---|
| `git revert` | ✅ Always safe | Adds a new undo commit |
| `git reset HEAD~1` | ✅ Safe | Uncommits, keeps changes |
| `git reset --hard` | ⚠️ Destructive | Uncommits, deletes changes |
| `git checkout -- file` | ⚠️ Destructive | Discards unstaged changes |

---

## 7. Viewing History

```bash
git log                          # full log
git log --oneline                # compact, one line per commit
git log --oneline --graph        # with ASCII branch graph
git log --oneline -10            # last 10 commits only
git log --oneline -- filename    # history of a specific file

git show a3f92c1                 # full diff of a specific commit
git diff main..branch-name       # diff between two branches
git diff HEAD~3 HEAD             # diff between 3 commits ago and now
```

---

## 8. Working with Remotes

A remote is just another copy of the repo — on GitHub, Gitea, a VPS, etc.

### Add a remote
```bash
git remote add origin https://github.com/you/repo.git
```

### Push for the first time
```bash
git push -u origin main
```
The `-u` sets upstream tracking so future pushes are just `git push`.

### Regular push/pull
```bash
git push           # send your commits to remote
git pull           # fetch remote commits and merge into current branch
```

### Pull before you push (always)
```bash
git pull
# resolve any conflicts if needed
git push
```

### Clone an existing repo
```bash
git clone https://github.com/you/repo.git
```

### View remotes
```bash
git remote -v
```

---

## 9. Merge Conflicts

A conflict happens when two different changes were made to the **same line** from two different directions, and git doesn't know which one wins. Git stops and asks you to decide manually.

**This is not an error. It's git asking a question.**

### What a conflict looks like in the file

```
<<<<<<< HEAD
kb_options = compose:ralt
=======
kb_options = compose:ralt,caps:swapescape
>>>>>>> branch-name
```

- Everything between `<<<<<<< HEAD` and `=======` is your current version
- Everything between `=======` and `>>>>>>>` is the incoming version
- You delete the markers, keep what you want, save the file

### Resolving it

```bash
# 1. open the conflicted file in nvim
nvim filename.md

# 2. find all conflict markers (search: /<<<<)
# 3. edit the file to the correct final state
# 4. delete all <<<<, ====, >>>> markers
# 5. save and exit

# 6. stage the resolved file
git add filename.md

# 7. complete the merge
git commit
```

### How to avoid conflicts

- **Solo work:** conflicts are rare. They only happen if you revert a commit whose lines were later edited.
- **Team work:** always `git pull` before starting new work. Small focused commits on separate concerns reduce overlap.
- Before reverting a specific commit, inspect it first: `git show <hash>` — check if those lines were touched again later.

---

## 10. Team Workflow (Pull Requests)

This is the standard industry workflow. Practice it even solo to build the habit.

```
1. git pull                          # always start from latest main
2. git checkout -b feat/my-change    # never work directly on main
3. make changes, commit often
4. git push -u origin feat/my-change # push the branch to remote
5. open a Pull Request on GitHub/Gitea
6. team reviews, leaves comments
7. you address comments, push more commits to same branch
8. PR gets approved and merged into main
9. git checkout main && git pull     # sync your local main
10. git branch -d feat/my-change     # clean up local branch
```

### Pull Request etiquette

- One PR = one logical change. Don't bundle unrelated things.
- Write a clear PR description: what changed, why, any notes for reviewer.
- Keep PRs small. A 10-line PR gets reviewed in 2 minutes. A 500-line PR gets rubber-stamped or ignored.
- Don't force-push to a branch that others are reviewing — it rewrites history they've already read.

---

## 11. Git for Dotfiles and Notes

Git works perfectly for config files and markdown notes. This is highly recommended.

### Suggested structure

```
~/notes/
├── post-install.md
├── git-reference.md
├── projects/
│   └── wow-bridge-notes.md
└── ...

~/.dotfiles/
├── .config/hypr/
├── .config/waybar/
├── .config/nvim/
└── ...
```

### Init your notes repo

```bash
cd ~/notes
git init
git add .
git commit -m "init: personal notes repo"
```

### Good commit habits for notes/dotfiles

```bash
# after editing a config
git add .config/hypr/input.conf
git commit -m "fix: capslock swap to compose:ralt,caps:swapescape"

# after a big session
git add -p     # selectively stage only what changed meaningfully
git commit -m "update: post-install checklist to v5"
```

### Multi-machine sync (no GitHub needed)

If you have a VPS or self-hosted Gitea, push there. Then on each machine:

```bash
git pull    # before editing
git push    # after committing
```

Syncthing is a simpler alternative if you don't want to think about git across machines, but you lose history. Git is better.

---

## 12. Conventional Commits Reference

A standard format used by serious projects. Adopt it from day one.

### Format

```
type: short description

optional longer body explaining WHY, not what

optional footer (e.g. "Closes #42")
```

### Types

| Type | When to use |
|---|---|
| `init:` | First commit in a repo |
| `add:` | New content, file, or feature |
| `remove:` | Deleted content or file |
| `update:` | Modified existing content |
| `fix:` | Corrected something wrong |
| `refactor:` | Restructured without changing meaning |
| `docs:` | Documentation only |
| `chore:` | Housekeeping (rename, move files, etc.) |
| `revert:` | Undoing a previous commit |

### Examples

```
add: arduino-ide desktop exec flags for wayland
fix: capslock config — was missing caps:swapescape
remove: spotify from install list
update: browser section with honest cons for each option
refactor: reorganize install packages by category
revert: add: ollama (hardware too weak to use it)
```

---

## 13. Habits Checklist

Build these until they're automatic.

### Before starting any work session
- [ ] `git pull` — sync from remote if applicable
- [ ] `git status` — confirm clean state before touching anything

### During work
- [ ] Commit at every logical stopping point, don't let changes pile up
- [ ] One commit per concern — stage selectively with `git add -p` if needed
- [ ] Imperative mood in message — "add X" not "added X"
- [ ] Use a branch if the change is experimental or risky

### Before committing
- [ ] `git diff --staged` — read exactly what you're about to commit
- [ ] Does the message describe the *what* clearly?
- [ ] Is this one logical change, or should it be split?

### After finishing
- [ ] `git push` — don't let local commits sit unsynced for days
- [ ] Delete merged branches — `git branch -d branch-name`

### When something goes wrong
- [ ] `git status` first — understand the state before doing anything
- [ ] `git log --oneline` — find where things went sideways
- [ ] Prefer `git revert` over `git reset --hard` unless you're certain
- [ ] When in doubt: `git stash` to set aside current changes safely, investigate, then `git stash pop`

---

> "A commit is a letter to your future self. Write it like they're in a hurry and mildly annoyed at you."
