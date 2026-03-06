That's a genuinely smart approach. You're not learning three things separately — you're learning one integrated workflow where each piece reinforces the others. Bandit forces you to actually _use_ the terminal to progress, so nothing is abstract.

---

### How They Reinforce Each Other

- **Bandit** forces real terminal usage — you're not doing fake exercises, you need to actually solve puzzles to move forward. Pressure makes it stick faster.
- **Neovim** will come up naturally in Bandit — reading files, editing, navigating. You'll be using motions for a real reason, not just drilling.
- **Tmux** ties it together — you'll naturally want multiple panes open (one for reading hints, one for working, one for notes) and that's exactly what tmux is for.

---

### One Thing to Know About Tmux + Vim

There's one gotcha — **Ctrl+b** is tmux's default prefix key (like Space is leader in Vim). So every tmux command is `Ctrl+b` then something. Most people remap it to `Ctrl+a` or `Ctrl+Space` because `Ctrl+b` is awkward.

You'll want to sort that early before you build muscle memory on the default.

---

### Suggested Starting Order

Since you're starting fresh:

1. **Start Bandit today** — Level 0 just teaches you to SSH in. Immediately familiar territory.
2. **Use tmux from day one** — just two panes, nothing fancy yet. Left pane for work, right pane for notes in Neovim.
3. **Don't rush Bandit levels** — read every man page it points you to. That's where the real Linux learning is.

---

### Realistic Expectation

The first week will feel slow because you're context switching between three new things. That's normal. By week two your brain starts bundling them together as one skill rather than three. By week three it starts feeling natural.

It's a solid plan — go for it.






Yes, you've spotted a real thing — but it's not quite a problem in practice.

---

### How Space as Leader Actually Works

When you press `Space`, Vim **waits** for the next key before doing anything. It doesn't immediately trigger whatever Space alone does. So there's no race against a timer — it just sits there patiently until you press the next key.

What you're probably seeing is **which-key** — a plugin (comes default in LazyVim) that shows a popup menu of available bindings after you press Space. That popup appearing is actually intentional and helpful, not a mistake. It's just showing you your options.

So `Space` → pause → `d` works perfectly fine. No rush.

---

### About `Shift`

Yes, Shift is heavily used in Vim. It's basically a **modifier that gives capital versions of commands**, and almost every letter has a shifted version that does something related but bigger/opposite:

|Normal|Shift version|Difference|
|---|---|---|
|`w` next word|`W`|next word (ignoring punctuation)|
|`b` prev word|`B`|prev word (ignoring punctuation)|
|`d` delete operator|`D`|delete to end of line|
|`c` change operator|`C`|change to end of line|
|`y` yank operator|`Y`|yank whole line|
|`p` paste after|`P`|paste before cursor|
|`o` new line below|`O`|new line above|
|`a` insert after|`A`|insert at end of line|
|`i` insert before|`I`|insert at line start|
|`v` visual mode|`V`|visual line mode|
|`g` prefix|`G`|go to bottom of file|
|`j` move down|`J`|join line below to current|
|`x` delete char|`X`|delete char before cursor|
|`f` jump to char|`F`|jump backward to char|
|`t` jump before char|`T`|jump backward before char|
|`r` replace char|`R`|enter replace mode|
|`u` undo|`U`|undo all changes on line|
|`n` next search|`N`|previous search result|
|`s` substitute char|`S`|substitute whole line|

---

### The Pattern to Notice

Shift in Vim almost always means either:

- **"Do the same thing but bigger"** — like `d` deletes a motion, `D` deletes to end of line
- **"Do the opposite direction"** — like `f` searches forward, `F` searches backward

Once you internalize that pattern, you don't need to memorize shifted commands separately — you can often guess them correctly.




You navigate in **Normal mode** (press `Esc` / your remapped Caps Lock to get there).

---

### `f`, `;` and `,` — Actually Very Useful Bindings

These are **already taken and genuinely useful:**

|Key|What it does|
|---|---|
|`f{char}`|Jump forward to next occurrence of that character on the line|
|`F{char}`|Jump backward to that character on the line|
|`;`|Repeat the last `f` or `F` jump forward|
|`,`|Repeat the last `f` or `F` jump backward|
|`t{char}`|Jump to just _before_ the character (instead of on it)|
|`T{char}`|Same but backward|

**Example:** if your line is `hello world foo bar` and cursor is at the start, pressing `fw` jumps straight to the `w` in `world`. Then `;` jumps to the next `w` in whatever follows. It's surprisingly powerful for line-level navigation.

---

### `d` — Also Already Taken, Very Important

`d` is the **delete operator** and it's one of the most used keys in Vim:

|Key|What it does|
|---|---|
|`dd`|Delete whole line|
|`dw`|Delete to next word|
|`db`|Delete previous word|
|`d$`|Delete to end of line|
|`d0`|Delete to start of line|
|`diw`|Delete inner word (wherever cursor is)|
|`di"`|Delete inside quotes|
|`di(`|Delete inside parentheses|

The pattern is **`d` + any motion** — so once you learn motions, delete becomes very expressive.

---

### The Home Row Keys at a Glance

Since you care about home row, here's what every home row key does in Normal mode:

|Key|Action|
|---|---|
|`a`|Enter insert mode _after_ cursor|
|`s`|Delete character and enter insert mode|
|`d`|Delete operator|
|`f`|Jump to character on line|
|`g`|Prefix for many commands (`gg` = top of file, `gc` = comment)|
|`h`|Move left|
|`j`|Move down|
|`k`|Move up|
|`l`|Move right|
|`;`|Repeat last `f/F/t/T` jump|
|`'`|Jump to mark|

Almost every home row key is taken and useful — which is exactly why Vim is so powerful without leaving home row.




## Getting Started with Vim Motions

### What are Vim Motions?

Vim motions are keyboard shortcuts for navigating and editing text _without_ touching your mouse. They're based on a modal editing system — you switch between modes rather than holding modifier keys.

**The 3 core modes:**

- **Normal mode** — navigation & commands (default, press `Esc` to return here)
- **Insert mode** — actually typing text (press `i`)
- **Visual mode** — selecting text (press `v`)

---

### Essential Motions to Learn First

**Navigation (Normal mode):**

|Key|Action|
|---|---|
|`h j k l`|Left, Down, Up, Right|
|`w` / `b`|Next / previous word|
|`e`|End of word|
|`0` / `$`|Start / end of line|
|`gg` / `G`|Top / bottom of file|
|`Ctrl+d` / `Ctrl+u`|Half-page down / up|
|`{` / `}`|Jump between paragraphs|

**Editing (Normal mode):**

|Key|Action|
|---|---|
|`x`|Delete character|
|`dd`|Delete line|
|`yy`|Copy (yank) line|
|`p`|Paste|
|`u`|Undo|
|`Ctrl+r`|Redo|
|`ciw`|Change inner word|
|`di"`|Delete inside quotes|

**Entering Insert mode:**

|Key|Action|
|---|---|
|`i` / `a`|Insert before / after cursor|
|`I` / `A`|Insert at line start / end|
|`o` / `O`|New line below / above|

---

### Vim Motions Across Editors

The **core motions are identical** across all platforms — that's the beauty of the Vim model. The differences are in configuration depth and plugin ecosystem.

#### VSCode — VSCodeVim or VSCode Neovim

- Install the **VSCodeVim** extension (easiest) or **VSCode Neovim** (more powerful, requires Neovim installed)
- VSCodeVim emulates most motions; VSCode Neovim runs an actual Neovim instance in the background
- Configure via `settings.json`; some advanced features (macros, complex remaps) work better with the Neovim extension

#### Sublime Text — NeoVintageous

- Install **NeoVintageous** via Package Control
- Very faithful Vim emulation, supports `.vimrc`-style config
- Covers surround, commentary, and other common plugins

#### Obsidian — Vim Mode (built-in)

- Enable in Settings → Editor → **Vim key bindings**
- More limited than other editors — aimed at note-taking, so some motions (window splits, etc.) don't apply
- The **Vimrc Support** community plugin lets you load a `.vimrc` for remaps

#### JetBrains IDEs (IntelliJ, PyCharm, etc.)

- Install the **IdeaVim** plugin
- Excellent emulation with deep IDE integration; configure via `~/.ideavimrc`

---

### Vim vs Neovim vs LazyVim

These are **actual editors**, not just motion plugins. The motions themselves are 100% identical — the differences are in the editor itself.

||**Vim**|**Neovim**|**LazyVim**|
|---|---|---|---|
|**What it is**|The original|Modern Vim fork|Neovim _distribution_ (preconfigured Neovim)|
|**Config language**|Vimscript|Lua (+ Vimscript)|Lua (built on top of Neovim)|
|**Plugin ecosystem**|VimPlug, Vundle|lazy.nvim, packer|Comes pre-bundled|
|**LSP / Autocomplete**|Manual setup|Built-in LSP client|Pre-configured out of the box|
|**Async support**|Limited|Full async|Full async|
|**Best for**|Purists, servers|Power users who like configuring|People who want a ready-made IDE-like setup|
|**Learning curve**|Medium|Medium–High|Lower (it's pre-configured)|

**The motions are the same.** `ciw` in Vim works identically in Neovim and LazyVim. What changes is the surrounding tooling — how you set up LSPs, file trees, fuzzy finders, themes, etc.

---

### Recommended Learning Path

1. Run `vimtutor` in your terminal — it's a 30-min interactive lesson built into Vim/Neovim
2. Install VSCodeVim if you use VSCode — lowest friction way to practice daily
3. Focus on mastering `h/j/k/l`, `w/b`, `ci`/`di` operators, and `v` visual mode first
4. After a week or two of muscle memory, explore Neovim if you want to go deeper

The investment pays off — once the motions are in muscle memory, they follow you to every editor you ever use.