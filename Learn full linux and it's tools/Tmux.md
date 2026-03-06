**Start a tmux session:**

```bash
tmux
```

---

**Two terminals side by side (panes):**

| Action                          | Shortcut           |
| ------------------------------- | ------------------ |
| Split vertically (side by side) | `Ctrl+b %`         |
| Split horizontally (top/bottom) | `Ctrl+b "`         |
| Switch between panes            | `Ctrl+b o`         |
| Close current pane              | `exit` or `Ctrl+d` |

---

**Or use separate windows (like browser tabs):**

| Action                    | Shortcut                     |
| ------------------------- | ---------------------------- |
| New window                | `Ctrl+b c`                   |
| Switch to next window     | `Ctrl+b n`                   |
| Switch to previous window | `Ctrl+b p`                   |
| Switch by number          | `Ctrl+b 0`, `Ctrl+b 1`, etc. |

---

**The pattern:** every tmux shortcut starts with the **prefix** `Ctrl+b`, then you press the key. So `Ctrl+b %` means: hold Ctrl+b, release, then press `%`.

For just toggling between two terminals, panes (`Ctrl+b %` then `Ctrl+b o`) is the most natural. Windows are better when you want more separation between tasks.



**Enter scroll mode (copy mode):**

```
Ctrl+b [
```

Now you can navigate freely. Exit with `q`.

---

**Navigation keys inside copy mode:**

| Action            | Key                         |
| ----------------- | --------------------------- |
| Scroll up         | `Ctrl+u` or arrow keys      |
| Scroll down       | `Ctrl+d` or arrow keys      |
| Page up           | `Ctrl+b` (inside copy mode) |
| Page down         | `Ctrl+f`                    |
| Go to top         | `g`                         |
| Go to bottom      | `G`                         |
| Search forward    | `/` then type               |
| Search backward   | `?` then type               |
| Next search match | `n`                         |

These are vim-style bindings — feels natural once you're used to Neovim.

---

**Selecting and copying text:**

| Action          | Key        |
| --------------- | ---------- |
| Start selection | `Space`    |
| Copy selection  | `Enter`    |
| Paste           | `Ctrl+b ]` |

---

**One config worth adding** — enables vim keys in copy mode properly:

```bash
# ~/.tmux.conf
set-window-option -g mode-keys vi
```

Then reload with `tmux source ~/.tmux.conf`. After this, `hjkl` navigation and `v` for visual select work exactly like Vim, which is what you want given your workflow.