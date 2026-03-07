# Git — A Real-World Guide

> A comprehensive guide to git written from real usage. Every command here was used in practice, not invented for a tutorial. Covers setup, daily workflow, dotfiles, hosting on GitHub and Codeberg, SSH, hygiene habits, and everything you need as a beginner or a refresher.

---

## Table of Contents

1. [What is Git?](#1-what-is-git)
2. [The Mental Model](#2-the-mental-model)
3. [Initial Setup](#3-initial-setup)
4. [SSH Keys — The Right Way to Authenticate](#4-ssh-keys--the-right-way-to-authenticate)
5. [GitHub and Codeberg](#5-github-and-codeberg)
6. [Core Daily Commands](#6-core-daily-commands)
7. [Commit Hygiene](#7-commit-hygiene)
8. [Branching](#8-branching)
9. [Undoing Things](#9-undoing-things)
10. [Viewing History](#10-viewing-history)
11. [Working with Remotes](#11-working-with-remotes)
12. [Pushing to Multiple Remotes](#12-pushing-to-multiple-remotes)
13. [Merge Conflicts](#13-merge-conflicts)
14. [Dotfiles with a Bare Repo](#14-dotfiles-with-a-bare-repo)
15. [What NOT to Commit](#15-what-not-to-commit)
16. [Conventional Commits](#16-conventional-commits)
17. [Team Workflow and Pull Requests](#17-team-workflow-and-pull-requests)
18. [Habits Checklist](#18-habits-checklist)
19. [Command Cheat Sheet](#19-command-cheat-sheet)

---

## 1. What is Git?

Git is a **version control system**. It tracks changes to files over time, lets you travel back to any previous state, and makes it safe to experiment without fear of breaking things permanently.

Git is not:
- A backup tool (use backups for that)
- GitHub (GitHub is a website that hosts git repos — git itself runs entirely on your machine)
- Only for code (it works perfectly for config files, notes, scripts, documentation)

---

## 2. The Mental Model

Before memorizing commands, understand these concepts. They explain why git works the way it does.

### Git is a timeline of snapshots

Every `git commit` saves a snapshot of your project at that moment. Git stores the full timeline.

```
A ── B ── C ── D ── E   ← main branch, moving forward
              ↑
          you are here
```

You can travel to any point on this timeline at any time.

### The three areas

```
Working Directory  →  Staging Area (Index)  →  Repository (History)
   (your files)          (git add)               (git commit)
```

- **Working Directory** — files as they exist on disk right now
- **Staging Area** — changes you've selected for the next commit
- **Repository** — permanent history of all commits

You control what goes into each commit by staging selectively. Changed 10 things but only want to commit 3? Stage just those 3.

### Git stores deltas, not full copies

Git saves only what changed between commits, not a full copy of every file every time. 100 commits on a text file costs almost nothing in disk space. Binary files (images, compiled binaries) are the exception — git can't compress those efficiently.

---

## 3. Initial Setup

Run this once on any new machine before doing anything else.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@yourdomain.com"
git config --global core.editor "nvim"           # or nano, vim, etc.
git config --global init.defaultBranch master
git config --global color.ui auto
```

**Useful extras worth setting:**

```bash
git config --global pull.rebase true             # cleaner history on pull
git config --global push.autoSetupRemote true    # auto-set upstream on push
git config --global diff.algorithm histogram     # clearer diffs
git config --global rerere.enabled true          # remember conflict resolutions
git config --global branch.sort -committerdate   # sort branches by recent activity
```

Verify everything:
```bash
git config --list
```

Your global config lives at `~/.config/git/config` (or `~/.gitconfig` on some systems). It applies to every repo on your machine unless overridden locally.

---

## 4. SSH Keys — The Right Way to Authenticate

When pushing to GitHub or Codeberg, you need to prove who you are. SSH keys are the correct method — more secure than passwords, and you only type your passphrase once per session.

### Generate a key (only if you don't have one)

```bash
ls ~/.ssh/    # check if id_ed25519 already exists
```

If not:
```bash
ssh-keygen -t ed25519 -C "you@yourdomain.com"
# Accept default location
# Set a passphrase — don't skip this
```

This creates two files:
- `~/.ssh/id_ed25519` — your **private key**. Never share this with anyone, ever.
- `~/.ssh/id_ed25519.pub` — your **public key**. This is what you give to GitHub/Codeberg.

### Add your public key to GitHub and Codeberg

```bash
cat ~/.ssh/id_ed25519.pub
# Copy the entire output
```

- **GitHub:** Settings → SSH and GPG keys → New SSH key → paste → title it with your machine name
- **Codeberg:** Settings → SSH/GPG Keys → Add Key → paste

### Verify it works

```bash
ssh -T git@github.com
# Expected: "Hi username! You've successfully authenticated..."

ssh -T git@codeberg.org
# Expected: "Hi there, username! You've successfully authenticated..."
```

The message "does not provide shell access" is normal — it just means git operations work but you can't SSH into their servers as a shell.

### SSH agent — stop typing your passphrase every time

Without this, you type your passphrase on every push. Fix it by adding this to your `~/.zshrc` (or `~/.bashrc`):

```bash
if [ -z "$SSH_AUTH_SOCK" ]; then
  eval "$(ssh-agent -s)" > /dev/null
  ssh-add ~/.ssh/id_ed25519 2>/dev/null
fi
```

Then reload:
```bash
source ~/.zshrc
```

Now you type your passphrase once when your terminal starts. Git pushes work silently after that.

### Email privacy on GitHub

GitHub has a privacy feature that blocks pushes containing your real email address in commit metadata. To use it:

1. Go to github.com/settings/emails
2. Copy your no-reply address — looks like: `123456789+username@users.noreply.github.com`
3. Set it as your email:

```bash
git config --global user.email "123456789+username@users.noreply.github.com"
```

If you already made commits with your real email, amend the last commit:
```bash
git commit --amend --reset-author --no-edit
git push --force   # only safe on brand new repos with no collaborators
```

---

## 5. GitHub and Codeberg

**GitHub** is the most widely known git hosting platform. Owned by Microsoft. Large community, widely recognized by employers.

**Codeberg** is a FOSS alternative run by a German nonprofit. Runs Forgejo (a Gitea fork). No ads, no tracking, no corporate ownership. Less well-known but growing.

You can use both simultaneously — push to GitHub for visibility, push to Codeberg as your principled backup. Git supports multiple remotes natively.

### Creating a repo

On both platforms:
- Click New Repository
- Give it a name
- Set visibility (Public or Private)
- **Do not initialize with README** if you're pushing an existing local repo — it creates a conflict

### Repo visibility
- **Public** — anyone can see it. Use for projects, scripts, notes you want to share.
- **Private** — only you (and collaborators you invite). Use for dotfiles, personal configs, anything sensitive.

---

## 6. Core Daily Commands

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
git add .                    # stage everything (use carefully)
git add -p                   # interactive — choose which chunks to stage
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

Always run `git diff --staged` before committing. Read what you're about to save.

### See history
```bash
git log --oneline            # compact view
git log --oneline --graph    # with branch visualization
git log --oneline -10        # last 10 commits only
```

---

## 7. Commit Hygiene

This is the most important section for long-term sanity.

### Rule 1: One commit = one logical change

Every commit should do exactly one thing. If you can't describe it in one short sentence, it's probably two commits.

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

It also inverts cleanly — the revert of "add X" is obviously "remove X".

### Rule 3: Commit often

The longer you wait, the harder it is to write a clean focused message. Commit every time you reach a logical stopping point.

### Rule 4: Never commit broken state to main

If your change is incomplete or broken, don't commit yet, or commit to a branch. `main`/`master` should always be in a working state.

### Rule 5: Use `git add -p` for precision

When you've changed multiple things and want to make separate clean commits:

```bash
git add -p filename.md
```

Git shows each changed chunk and asks: stage this? `y` for yes, `n` for no, `s` to split the chunk further. This is how you craft exactly the commit you want.

---

## 8. Branching

### Why branch even when working solo

- Keeps `master` always clean and working
- Makes it safe to experiment without touching your stable state
- Builds muscle memory for team workflows

### The workflow

```bash
git checkout -b feat/my-feature    # create and switch to branch

# do work, commit normally
git add file
git commit -m "add: thing"

# when done, merge back
git checkout master
git merge feat/my-feature

# clean up
git branch -d feat/my-feature
```

### Branch naming conventions

```
feat/arduino-setup          # new feature or content
fix/capslock-config         # correcting something wrong
update/browser-notes        # modifying existing content
remove/spotify-entry        # deleting something
experiment/new-structure    # trying something, may be abandoned
```

### View branches

```bash
git branch              # local branches
git branch -a           # local + remote branches
```

---

## 9. Undoing Things

Git makes undoing things safe by design. Don't panic.

### Undo last commit (keep the changes)
```bash
git reset HEAD~1
```
Your changes are still in your working directory. Nothing is lost.

### Undo last commit (discard changes too)
```bash
git reset --hard HEAD~1
```
> ⚠️ Destructive. Changes are gone. Only use when certain.

### Safely revert a commit (the default choice)
```bash
git revert <commit-hash>
```
This adds a new commit that undoes the target. History only moves forward. Nothing is rewritten. This is the command to reach for first.

```
Before:  A ── B ── C (bad) ── D ── E
After:   A ── B ── C (bad) ── D ── E ── F (reverts C)
```

### Find a commit hash
```bash
git log --oneline    # copy the short hash from here
```

### Discard unstaged changes to a file
```bash
git checkout -- filename.md
```
> ⚠️ Destructive. Unsaved changes to that file are gone.

### Unstage a file (keep changes, just remove from staging)
```bash
git restore --staged filename.md
```

### Stash changes temporarily
```bash
git stash           # set aside current changes
git stash pop       # bring them back
```
Useful when you need to switch branches but aren't ready to commit.

### Safe vs destructive at a glance

| Command | Safe? | What it does |
|---|---|---|
| `git revert` | ✅ Always safe | Adds a new undo commit |
| `git reset HEAD~1` | ✅ Safe | Uncommits, keeps changes |
| `git stash` | ✅ Safe | Sets aside changes temporarily |
| `git reset --hard` | ⚠️ Destructive | Uncommits and deletes changes |
| `git checkout -- file` | ⚠️ Destructive | Discards unstaged changes |

---

## 10. Viewing History

```bash
git log                          # full log
git log --oneline                # compact, one line per commit
git log --oneline --graph        # with ASCII branch graph
git log --oneline -10            # last 10 commits only
git log --oneline -- filename    # history of a specific file

git show a3f92c1                 # full diff of a specific commit
git diff main..branch-name       # diff between two branches
git diff HEAD~3 HEAD             # diff between 3 commits ago and now

git show HEAD~3:filename.md      # see a file as it was 3 commits ago
```

---

## 11. Working with Remotes

A remote is another copy of the repo — on GitHub, Codeberg, a VPS, etc.

### Add a remote
```bash
git remote add origin git@github.com:username/repo.git
```

### Push for the first time
```bash
git push -u origin master
```
The `-u` sets upstream tracking so future pushes are just `git push`.

### Regular push/pull
```bash
git push           # send your commits to remote
git pull           # fetch remote commits and merge
```

### Always pull before you push
```bash
git pull
# resolve any conflicts if needed
git push
```

### Clone a repo
```bash
git clone git@github.com:username/repo.git
```

### View remotes
```bash
git remote -v
```

### Remove a remote
```bash
git remote remove origin
```

---

## 12. Pushing to Multiple Remotes

Git supports pushing to multiple remotes simultaneously. This is useful for mirroring between GitHub and Codeberg.

### Setup — push to both with one command

```bash
git remote add origin git@github.com:username/repo.git
git remote set-url --add --push origin git@github.com:username/repo.git
git remote set-url --add --push origin git@codeberg.org:username/repo.git
```

After this, `git push` sends to both at once. `git pull` still only pulls from the primary (fetch) remote.

### Verify your remotes
```bash
git remote -v
```

Expected output — exactly 3 lines, no duplicates:
```
origin  git@github.com:username/repo.git (fetch)
origin  git@github.com:username/repo.git (push)
origin  git@codeberg.org:username/repo.git (push)
```

### Fix duplicate remotes
If you accidentally added a remote twice:
```bash
git remote set-url --delete --push origin git@github.com:username/repo.git
git remote set-url --delete --push origin git@codeberg.org:username/repo.git
# then re-add once each
git remote set-url --add --push origin git@github.com:username/repo.git
git remote set-url --add --push origin git@codeberg.org:username/repo.git
```

### Force push (use carefully)
```bash
git push --force
```
Only use after amending commits on a brand new repo with no collaborators. Never force push to a branch others are working on.

---

## 13. Merge Conflicts

A conflict happens when two different changes were made to the same line from two different directions, and git can't decide which wins. Git stops and asks you to decide.

**This is not an error. It's git asking a question.**

### What a conflict looks like

```
<<<<<<< HEAD
kb_options = compose:ralt
=======
kb_options = compose:ralt,caps:swapescape
>>>>>>> branch-name
```

- Between `<<<<<<< HEAD` and `=======` — your current version
- Between `=======` and `>>>>>>>` — the incoming version
- Delete the markers, keep what you want, save

### Resolving it

```bash
nvim filename.md           # open the conflicted file
# search for <<<< to find conflict markers
# edit to the correct final state
# delete all <<<<, ====, >>>> markers
# save and exit

git add filename.md        # stage the resolved file
git commit                 # complete the merge
```

---

## 14. Dotfiles with a Bare Repo

Dotfiles are config files scattered across your home directory (`~/.config/hypr/`, `~/.zshrc`, etc.). A bare git repo lets you track them without moving them or using symlink tools.

### How it works

A normal git repo has a `.git/` folder inside the project directory. A bare repo stores git data separately and lets you point the working tree anywhere — in this case, your entire home directory.

You use an alias `dot` instead of `git` for all dotfile operations.

### Setup (run once)

```bash
git init --bare ~/.dotfiles

echo "alias dot='git --git-dir=\$HOME/.dotfiles --work-tree=\$HOME'" >> ~/.zshrc
source ~/.zshrc

dot config --local status.showUntrackedFiles no
dot config --local user.name "Your Name"
dot config --local user.email "you@yourdomain.com"
```

`status.showUntrackedFiles no` is critical — without it, `dot status` would list every file in your home directory as untracked. This tells git to only show files you've explicitly added.

### Tracking files

Never use `dot add .` — always add files explicitly by name:

```bash
dot add ~/.config/hypr/input.conf
dot add ~/.config/waybar/config.jsonc
dot add ~/.zshrc
```

Check what's staged:
```bash
dot status
dot diff --staged    # always read this before committing
```

Commit:
```bash
dot commit -m "init: dotfiles — hypr, waybar, zsh"
```

### Daily dotfiles workflow

Any time you change a tracked config:
```bash
dot status                              # see what changed
dot diff                                # see the actual changes
dot add ~/.config/hypr/bindings.conf   # stage the specific file
dot commit -m "add: localsend keybinding"
dot push
```

### Restoring dotfiles on a new machine

```bash
git clone --bare git@github.com:username/dotfiles.git ~/.dotfiles
echo "alias dot='git --git-dir=\$HOME/.dotfiles --work-tree=\$HOME'" >> ~/.zshrc
source ~/.zshrc
dot config --local status.showUntrackedFiles no
dot checkout
```

If checkout fails because files already exist:
```bash
dot checkout 2>&1 | grep "^\s" | awk '{print $1}' | xargs -I{} mv {} {}.bak
dot checkout
```

---

## 15. What NOT to Commit

This section matters more than most. One leaked secret in a public repo is a real security incident.

### Never commit

- **Private keys** — SSH keys (`~/.ssh/id_ed25519`), WireGuard `PrivateKey` values
- **API tokens and passwords** — any line containing a token, key, secret, or password
- **`.env` files** — these almost always contain secrets
- **Database files** — Vaultwarden `.db`, SQLite files
- **Binary files** — images, compiled binaries, archives (bloat the repo)
- **Cache and generated data** — `.cache/`, `node_modules/`, `__pycache__/`
- **Browser profile data** — Chromium, Firefox profile directories

### Before committing any config file, scan it
```bash
grep -i "key\|token\|secret\|password" filename
```

If anything comes back, remove those values before staging.

### Use `.gitignore`

Create a `.gitignore` file in your repo root to permanently exclude patterns:

```
# Secrets
*.env
*.key
*.pem

# Generated
node_modules/
__pycache__/
*.pyc
.cache/

# Binaries
*.exe
*.bin
*.img
```

Git will never track files matching these patterns, even with `git add .`.

### Where secrets DO go

Use encrypted storage instead of git:
- **Vaultwarden/Bitwarden** — secure notes for private keys, tokens, passwords
- **GPG-encrypted files** — `gpg --symmetric secrets.tar.gz`
- **Private git repo + git-crypt** — encrypted files inside a private repo (advanced)

---

## 16. Conventional Commits

A standard format used by serious projects. Adopt it from day one — it makes history readable and searchable.

### Format

```
type: short description in imperative mood

optional body explaining WHY, not what

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
init: dotfiles repo
add: localsend keybinding to hyprland bindings
fix: capslock config missing caps:swapescape
remove: spotify from install list
update: browser section with honest cons for each option
refactor: reorganize waybar modules
revert: add: ollama (hardware too weak)
```

---

## 17. Team Workflow and Pull Requests

Even if you work solo, practice this workflow — it's the industry standard.

```
1. git pull                           # always start from latest
2. git checkout -b feat/my-change     # never work directly on master
3. make changes, commit often
4. git push -u origin feat/my-change  # push the branch
5. open a Pull Request on GitHub/Codeberg
6. team reviews, leaves comments
7. address comments, push more commits to same branch
8. PR approved and merged into master
9. git checkout master && git pull    # sync local master
10. git branch -d feat/my-change      # clean up
```

### Pull Request etiquette

- One PR = one logical change. Don't bundle unrelated things.
- Write a clear description: what changed, why, notes for reviewer.
- Keep PRs small. A 10-line PR gets reviewed in 2 minutes. A 500-line PR gets rubber-stamped.
- Never force-push to a branch others are reviewing.

---

## 18. Habits Checklist

Build these until they're automatic.

### Before starting any work session
- [ ] `git pull` — sync from remote
- [ ] `git status` — confirm clean state before touching anything

### During work
- [ ] Commit at every logical stopping point
- [ ] One commit per concern — use `git add -p` if needed
- [ ] Imperative mood — "add X" not "added X"
- [ ] Branch if the change is experimental or risky

### Before committing
- [ ] `git diff --staged` — read exactly what you're about to commit
- [ ] Does the message describe the what clearly?
- [ ] Is this one logical change, or should it be split?
- [ ] Does this file contain any secrets? (`grep -i "key\|token\|secret\|password"`)

### After finishing
- [ ] `git push` — don't let local commits sit unsynced
- [ ] Delete merged branches — `git branch -d branch-name`

### When something goes wrong
- [ ] `git status` first — understand the state before doing anything
- [ ] `git log --oneline` — find where things went sideways
- [ ] Prefer `git revert` over `git reset --hard` unless certain
- [ ] When in doubt: `git stash` to set aside changes, investigate, then `git stash pop`

---

## 19. Command Cheat Sheet

### Setup
```bash
git config --global user.name "Name"
git config --global user.email "email"
git config --global core.editor "nvim"
git config --global init.defaultBranch master
git config --list
```

### Daily
```bash
git init
git status
git add filename
git add -p filename          # interactive staging
git diff                     # unstaged changes
git diff --staged            # staged changes
git commit -m "type: msg"
git log --oneline
git log --oneline --graph
```

### Remotes
```bash
git remote add origin git@github.com:user/repo.git
git remote -v
git push -u origin master
git push
git pull
git clone git@github.com:user/repo.git
git remote remove origin
```

### Multiple remotes (GitHub + Codeberg)
```bash
git remote set-url --add --push origin git@github.com:user/repo.git
git remote set-url --add --push origin git@codeberg.org:user/repo.git
git remote set-url --delete --push origin <url>   # remove one
```

### Branches
```bash
git checkout -b branch-name
git checkout master
git merge branch-name
git branch -d branch-name
git branch -a
```

### Undoing
```bash
git revert <hash>            # safe — adds new undo commit
git reset HEAD~1             # uncommit, keep changes
git reset --hard HEAD~1      # uncommit, delete changes ⚠️
git restore --staged file    # unstage a file
git checkout -- file         # discard unstaged changes ⚠️
git stash                    # set aside changes
git stash pop                # restore stashed changes
```

### Dotfiles (bare repo)
```bash
dot status
dot add ~/.config/hypr/input.conf
dot diff --staged
dot commit -m "type: msg"
dot push
dot remote -v
dot log --oneline
```

### Inspecting
```bash
git show <hash>
git show HEAD~3:filename     # file at a past commit
git diff main..branch-name
git log --oneline -- filename
```

### SSH
```bash
ssh-keygen -t ed25519 -C "email"
cat ~/.ssh/id_ed25519.pub    # copy this to GitHub/Codeberg
ssh -T git@github.com
ssh -T git@codeberg.org
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---

> "A commit is a letter to your future self. Write it like they're in a hurry and mildly annoyed at you."


## 20. My Repos

| Repo                      | Location                               | Command | Remotes                     |
| ------------------------- | -------------------------------------- | ------- | --------------------------- |
| dotfiles                  | `~/.dotfiles` (bare)                   | `dot`   | GitHub + Codeberg (private) |
| notes                     | `~/Documents/Obsidian Vault`           | `git`   | GitHub + Codeberg (public)  |
| protonvpn-wireguard-linux | `~/projects/protonvpn-wireguard-linux` | `git`   | GitHub + Codeberg (public)  |
| omarchy-webapp-profile    | `~/projects/omarchy-webapp-profile`    | `git`   | GitHub + Codeberg (public)  |