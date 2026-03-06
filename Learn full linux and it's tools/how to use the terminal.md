Here are the most essential terminal navigation shortcuts:

**Moving the cursor**

- `Ctrl+A` — go to beginning of line
- `Ctrl+E` — go to end of line
- `Ctrl+←` / `Ctrl+→` — jump word by word
- `←` / `→` — move one character at a time

**History**

- `↑` / `↓` — scroll through previous commands
- `Ctrl+R` — search command history (type to search, press `Ctrl+R` again to cycle matches)

**Editing**

- `Ctrl+U` — delete everything before cursor
- `Ctrl+K` — delete everything after cursor
- `Ctrl+W` — delete the word before cursor
- `Ctrl+L` — clear the screen (same as `clear`)

**Copy/Paste in terminal**

- `Ctrl+Shift+C` — copy selected text
- `Ctrl+Shift+V` — paste
- You can also just **highlight text with mouse** to select, then middle-click to paste

**Canceling**

- `Ctrl+C` — cancel current command
- `Ctrl+D` — exit/logout current shell

**Scroll up in terminal**

- `Shift+PageUp` / `Shift+PageDown` — scroll through terminal output
- Once you've scrolled to the text, just **click and drag with mouse** to highlight it, then `Ctrl+Shift+C` to copy

**If you're in a terminal with a scrollbar** you can just scroll with the mouse wheel too.

**The cleanest option — pipe to `less`** If you're running a command that outputs a lot, pipe it through `less` so you can scroll freely:

```bash
your_command | less
```

Then use `↑`/`↓` or `PageUp`/`PageDown` to navigate, and `q` to quit.

---

For your current situation (finding output from a past command), `Shift+PageUp` to scroll back is your quickest bet. Most terminal emulators support it out of the box.


me : i want to use terminal like neovim. i want to use home row more so that i don't have to lift my entire hand.

AI : 

You want **tmux** + a shell with Vi mode!

**Step 1: Enable Vi mode in your shell**

If you're using zsh (which OmarchyOS uses by default), add this to your `~/.zshrc`:

```bash
bindkey -v
```

Now in the terminal:

- Press `Esc` to enter normal mode
- `h` `j` `k` `l` to move
- `b` / `w` to jump words
- `0` / `$` for line start/end
- `i` to go back to insert mode

**Step 2: Install tmux for pane/window navigation**

```bash
sudo pacman -S tmux
```

Tmux lets you split terminals, switch windows, all from keyboard. The rabbit hole goes deep here.

---

**Honestly though**, for your immediate use case (scrolling back through output), the biggest bang for your buck is just:

1. `bindkey -v` in zshrc — gives you vi motion keys for editing commands
2. `Ctrl+R` — fuzzy search history without scrolling at all

If you want to go full home-row, look into **tmux** properly later. It's powerful but has a learning curve. Start with vi mode first and get comfortable with that.



To select and copy in tmux:

1. `Ctrl+B [` — enter scroll/copy mode
2. `↑`/`↓` or `Ctrl+U`/`Ctrl+D` to scroll
3. `Space` to start selection, move to highlight
4. `Enter` to copy

Then `Ctrl+B ]` to paste.

---

Now back to Limine — did `ls /mnt/windows-efi/EFI/` show a `Microsoft` folder?