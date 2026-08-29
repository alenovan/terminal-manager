# Terminal Manager

**English** · [Bahasa Indonesia](README.id.md)

A terminal profile launcher for Windows that runs the terminals **inside its own window**, like the
terminal panel of an IDE. Save your shells as profiles, group them, and open them as a grid of live
panes instead of a scatter of separate windows.

No runtime to install, no dependencies. One `.exe`.

```
+-- Local ---------+---------------+
| PS C:\> npm run  | $ git status  |
| ready on :3000   | On branch main|
| _                | _             |
+------------------+---------------+
| wsl:~$ htop      | ssh prod      |
| CPU  32%         | root@prod:~#  |
| _                | _             |
+------------------+---------------+
```

The terminals are real. Terminal Manager talks to the Windows pseudo console (ConPTY), so ANSI
colour, cursor control, resizing, and full-screen TUI programs such as `vim`, `htop` or an `ssh`
password prompt all behave exactly as they would in a normal console.

---

## Install

1. Download `TerminalManagerSetup.exe` from the
   [latest release](https://github.com/alenovan/terminal-manager/releases/latest).
2. Run it.

That is the whole distribution — a single installer, with the application embedded inside it.

By default it installs **per user**, into `%LOCALAPPDATA%\Programs\TerminalManager`, so Windows does
not raise a UAC prompt. Tick **Install for all users** to put it in `Program Files` instead; the
installer then asks for administrator rights and runs an elevated copy of itself.

The installer creates optional Desktop and Start Menu shortcuts and registers the application in
**Add or remove programs**, so it uninstalls the way any other program does.

Your `profiles.json` does not ship with the installer. It is created on first run, seeded with a few
sample profiles you can rename or delete.

### Requirements

| | |
|---|---|
| Windows | 10 version 1809 or newer, for the pseudo console |
| .NET | Framework 4.x — already present on every Windows 10 and 11 |

On older Windows the application still runs, but profiles fall back to opening in a separate window.

### Silent install

```
TerminalManagerSetup.exe /S                        install with the defaults
TerminalManagerSetup.exe /S /allusers              install into Program Files (needs admin)
TerminalManagerSetup.exe /S /dir:"C:\path"         choose the folder
TerminalManagerSetup.exe /S /nodesktop /nostartmenu
```

### Uninstall

Use **Add or remove programs**, or run `uninstall.exe` from the install folder. Your profiles are
kept unless you tick the box to delete them.

```
uninstall.exe /uninstall /S            remove silently, keep profiles
uninstall.exe /uninstall /S /purge     remove silently, delete profiles too
```

---

## Usage

### Groups and profiles

The sidebar holds **groups** — plain folders such as `Local`, `Dev`, `Servers` — and each group holds
**profiles**. A profile is one terminal configuration: the program to run, its arguments, working
directory, environment variables, and how to open it.

Everything is written to `profiles.json` the moment it changes. It is readable JSON; edit it by hand
if you prefer.

### Opening terminals

Press **Launch** (or `Ctrl+Enter`, or double-click a profile in the sidebar). The terminal opens
**inside the window**, in a tab of its own — clicking a profile shows that terminal and nothing else.

Terminals gather by group: **one tab per group**. Opening a second profile from the same group puts
it beside the first as another pane, while a profile from a different group opens its own tab. So a
group's terminals stay together and visible, and reaching for another group never hides them.

The rest is opt-in:

- **Add pane** (`Ctrl+Shift+Enter`) forces the selected profile into whichever tab is in front,
  even one belonging to another group. Use it to build a grid across groups on purpose.
- **Launch whole group** opens every in-app profile of a group at once as one tab of panes — two
  profiles side by side, four as a 2x2.
- `Alt+Shift+D` and `Alt+Shift+E` split the focused pane directly.

### Panes

| | |
|---|---|
| Split right / down | `Alt+Shift+D` / `Alt+Shift+E` |
| Move between panes | `Alt`+arrow keys |
| Resize | drag the divider between panes |
| Rearrange | drag a pane by its header onto another pane |
| Move a pane to its own tab | the button in the pane header, or `Alt+Shift+T` |
| Fold a pane back into the previous tab | `Alt+Shift+G` |
| Close a pane | the cross in its header, or `Ctrl+Shift+W` |

Dragging a pane by its header moves it within the grid. Drop it on the outer quarter of another
pane's edge to land on that side; drop it in the middle to swap the two panes. The shape of the
split follows from where you let go.

Moving a pane, whether by dragging or between tabs, does not restart it. The control is re-parented;
the shell keeps running and keeps its scrollback.

Closing a pane or a tab ends the process **and everything it started**, because each session runs
inside a job object. A `npm run dev` started in a pane cannot survive as an orphan.

### Switching between a grid and tabs

One toolbar button flips the current tab either way, and its caption says which:

- **To tabs** — the tab holds several panes, so each one moves out into a tab of its own.
- **To grid** — the tab holds a single pane, so every other terminal tab is pulled back in beside it.

Individual panes still move one at a time with the button in the pane header (`Alt+Shift+T`) and
`Alt+Shift+G`.

### Groups, subgroups and locking

Groups nest. Right-click a group for **New subgroup**, and the sidebar indents each level. The count
on the right of a group counts its profiles and its subgroups together.

Right-click also offers **Lock group**. A locked group cannot be renamed or deleted, no profile can
be dragged into or out of it, and nothing inside it can be renamed or deleted either — the lock
applies to every descendant. It is marked with a padlock. Unlocking is the same menu item.

Drag a profile onto another group to move it there; drop it on a profile and it joins that profile's
group. Whole groups can be dragged into other groups the same way. A group cannot be dropped inside
itself, and locked groups refuse both directions.

### Launch modes

| Mode | Where the terminal opens |
|---|---|
| `Inline` | inside this window |
| `Auto` | inside this window |
| `Windows Terminal` | a separate Windows Terminal window |
| `Console` | a separate console window |

In-app is the default. Opening outside is an explicit choice.

Two cases fall back to a separate window on their own, and the status bar says why:

- **Run as Administrator** profiles, unless Terminal Manager is itself elevated. A medium-integrity
  process cannot hand a pseudo console to a high-integrity child, because UAC elevation goes through
  `ShellExecute`, which takes no attribute list. Use **Restart as admin** in the toolbar to reopen
  the whole application elevated — every terminal inside it is then elevated too.
- Windows older than 10 version 1809, where the pseudo console does not exist.

### Saving your session

A live shell cannot be carried across a restart — Windows has no way to move a running process and
its pseudo console into a new session. What can be saved is the arrangement, or the text on screen.

**Reopen as you left it.** The open tabs and panes are written to `session.json` when the application
closes: which profile, in which position, at which divider ratio. On the next start the arrangement
is rebuilt and the shells are started **fresh** — a clean prompt, not the previous output. A profile
deleted in the meantime does not break the restore; its leaf collapses and its sibling takes the
space.

**Named workspaces.** **Save layout** (`Ctrl+Shift+L`) stores the current tab's grid under a name.
Workspaces appear in the sidebar above the groups, marked with a 2x2 icon and their pane count.
Double-click to open one. Deleting a workspace does not delete the profiles it points at.

**Save the output.** **Save output** (`Ctrl+Shift+S`) writes the focused pane's whole scrollback — up
to 5000 lines — to a `.log` or `.txt` file as plain text, with ANSI colour dropped.

### What each terminal costs

The status bar shows the focused terminal's memory and CPU — for example `pwsh   148 MB  3%  x4` —
sampled every two seconds. When a tab holds more than one pane, the same figures also appear in each
pane header, so a grid can be read at a glance.

The numbers cover the whole process tree, not just the shell, because every session runs inside its
own job object: a pane running `npm run dev` reports the dev server too. The trailing count appears
when more than one process is alive in that session. Hidden tabs are not sampled at all.

For comparing them side by side, **Resources** in the toolbar (`Ctrl+Shift+U`) opens a monitor
listing every running terminal with a bar for memory and one for CPU, its group and process id
underneath, and a total at the foot. Click a column heading to sort by it. The window samples every
session while it is open, including the ones on hidden tabs, and closes with `Esc`.

```
Resources
3 running terminal(s)

  Terminal                Memory        CPU
  Memory hog              77 MB         0%
  Test - pid 26416        ##########    ..........
  Busy loop               77 MB         8%
  Test - pid 25932        ##########    #.........
  Idle shell              4 MB          0%
  Test - pid 21736        #.........    ..........

  Total                   157 MB        8%
```

### The office

**Office** in the toolbar, or `Ctrl+Shift+O`, draws the group as a pixel-art office. **One group is
one office, one profile is one character.** Every profile in the group keeps its desk whether or not
a terminal is running, so the floor plan stays put instead of rearranging itself.

What each character does is driven by its session's real CPU, not by a script:

| On screen | Meaning |
|---|---|
| Typing fast, sweating | over 40% CPU |
| Typing | 8% and up |
| Sitting, hands on the desk | some activity, under 8% |
| Typing fast, flushed, sweating, heat rising off them | over 40% CPU - and they keep going |
| Walks to the water cooler, drinks, walks back | quiet for about six seconds |
| Walks into the coffee room and stays a while | quiet, and it felt like coffee instead |
| Playing a game at the desk, the monitor showing it | quiet, and it felt like a game instead |
| Stretching, arms up | quiet, and it felt like a stretch instead |
| Asleep at the desk, `z` drifting up | **an hour** with nothing to do |
| Empty chair, dark monitor | the process has exited, or was never started |
| The boss walks over with an exclamation mark | somebody has been gaming too long |

A quiet character picks between water, coffee, a game, and a stretch, so an idle office looks like an
office rather than a row of statues. The ones going for water or coffee actually walk there: out of
the pod, along the corridor, through the coffee room's doorway, and back again, facing the way they
are going. Only an hour of complete quiet puts somebody to sleep at their desk.

Each character has its own shirt colour and hair, so two profiles running the same shell are still
told apart at a glance.

The floor is a single office seen from above: tiled floor, walls carrying a whiteboard, windows onto
the night and a wall clock whose hands are set from your system time; desks with monitors, keyboards,
mice and mugs; a coffee room behind a partition with its own doorway, counter, coffee machine and a
table with two stools; and an open lounge with a water cooler, its gallon bottle, and plants.

There is a **boss office** down the right-hand side, behind its own wall and door: a wide desk with a
lamp, a big chair, a bookshelf and a plant. The boss sits in it and watches. Let a character keep a
game running too long and he gets up, walks over, stands at that desk with an exclamation mark over
his head, and sends them back to work - that one will not risk another game for about a minute.

**The office keeps your hours.** After six in the evening and before six in the morning the whole
floor goes dark: a cold wash over the room, warm pools under the ceiling lights, the monitors
glowing, and night behind the windows. Set `TM_OFFICE_NIGHT` to `1` or `0` to pin it either way
and see the other look without waiting.

The window can be maximised, from the button in its header or by double-clicking the header. The room
is drawn at the largest whole-number scale that fits, so it grows with the window instead of
stretching. The scene is composed at one unit per pixel and scaled up
with nearest-neighbour, so the pixels are real rather than smooth shapes pretending. It is a window
to watch: nothing in it is clickable, and `Esc` closes it.

### Scrolling

The wheel scrolls the pane the pointer is over, whether or not it holds focus. `Shift+PgUp` and
`Shift+PgDn` move a screen at a time, `Ctrl+Shift+Home` and `Ctrl+Shift+End` jump to either end,
and the slim bar on the right can be dragged. Scrollback holds 5000 lines.

Programs that take over the screen — `vim`, `htop`, `less` — run on the alternate buffer, which
has no scrollback by design; there the wheel sends arrow keys instead, so the program scrolls itself.

### Administrator

When Terminal Manager starts without administrator rights it says so once and offers to restart
elevated. Declining is fine — the title bar then carries a **no admin** badge, and the footer explains
that profiles marked Run as Administrator will open in separate windows. Tick **Do not ask again** to
silence the prompt; the badge stays either way.

### Selecting and copying

Drag to select, double-click for a word, triple-click for a line. `Ctrl+Shift+C` copies,
`Ctrl+Shift+V` pastes, and so does a right-click. Bracketed paste is supported, so shells that
enable it receive multi-line pastes safely.

Every other key goes to the shell. `Ctrl+C`, `Ctrl+S`, `Ctrl+N` and the rest reach the process, not
the application.

---

## Profile fields

| Field | What it does |
|---|---|
| Profile name | The name in the sidebar, and the default tab title |
| Group | Moves the profile to another group |
| Shell preset | Fills in Executable and Arguments. Pick `Custom` to write them yourself |
| Launch mode | See the table above |
| Executable | The program to run, e.g. `pwsh.exe` |
| Arguments | Raw arguments, e.g. `-NoLogo -NoExit` |
| Working directory | Starting folder. Environment variables are expanded: `%USERPROFILE%` |
| Environment variables | `KEY=VALUE`, one per line. Lines starting with `#` are ignored |
| Tab title | Tab title for Windows Terminal |
| Windows Terminal profile | The WT profile to use (`-p`), to inherit its colours and font |
| Startup commands | Typed into the shell on launch, one per line. Lines starting with `#` are ignored |
| Run as Administrator | Run elevated (raises a UAC prompt) |

Presets are included for PowerShell 5.1, PowerShell 7, Command Prompt, WSL, Git Bash, SSH, Node and
Python.

---

## Keyboard shortcuts

### Application

| Key | Action |
|---|---|
| `Ctrl+Enter` | Launch the selected profile |
| `Ctrl+Shift+Enter` | Open it as a pane beside the current tab |
| `Ctrl+S` | Save the profile form |
| `Ctrl+F` | Focus the search box |
| `Ctrl+N` | New profile |
| `F2` | Rename |
| `Ctrl+Tab` / `Ctrl+Shift+Tab` | Switch tabs |
| `Ctrl+Shift+L` | Save the current tab as a workspace |
| `Ctrl+Shift+S` | Save the focused pane's output to a file |
| `Ctrl+Shift+U` | Open the resource monitor |
| `Ctrl+Shift+O` | Open the office view |

### Panes

| Key | Action |
|---|---|
| `Alt+Shift+D` / `Alt+Shift+E` | Split right / down |
| `Alt`+arrows | Move between panes |
| `Alt+Shift+T` | Move the pane to its own tab |
| `Alt+Shift+G` | Fold the pane into the previous tab |
| `Ctrl+Shift+W` | Close the pane |

### Inside a terminal

| Key | Action |
|---|---|
| `Ctrl+Shift+C` / `Ctrl+Shift+V` | Copy / paste (right-click also pastes) |
| `Ctrl+Shift+A` | Select all |
| `Shift+PgUp` / `Shift+PgDn` | Scroll the scrollback |
| `Ctrl+Shift+Home` / `Ctrl+Shift+End` | Jump to the top / bottom |
| `Ctrl+=` / `Ctrl+-` / `Ctrl+0` | Zoom in / out / reset |
| `Ctrl+R` | Restart a session whose process has ended |

## Technical notes

- **The terminal.** `CreatePseudoConsole` plus `CreateProcess` with
  `PROC_THREAD_ATTRIBUTE_PSEUDOCONSOLE`. Output is read on a background thread and drained by a
  16 ms UI timer with a per-tick byte budget, so a flood of output cannot freeze the interface.
- **The renderer.** Cells are drawn with `ExtTextOut` and an explicit advance array, so columns line
  up exactly instead of drifting over a long line. Runs of identical styling are drawn in one call.
- **The VT emulator** is a practical subset, not a complete xterm: SGR including 256-colour and
  truecolour, cursor movement, scroll regions, the alternate buffer, bracketed paste, and mouse
  reporting. Resizing does not reflow text that has already been printed — columns stay where they
  were, like conhost before reflow existed.
- **Process trees.** Each session is assigned to a job object with kill-on-close, so closing a pane
  takes down the whole tree.
- **Environment variables** are handed to ConPTY as a real environment block. The temporary `.cmd`
  wrapper is only needed for external launches, where `ShellExecute` — required for UAC — cannot
  carry them.
- **Portable data.** `profiles.json` lives next to the executable when that folder is writable, and
  falls back to `%APPDATA%\TerminalManager` when it is not, such as under `Program Files`. The active
  path is always shown in the status bar.

---

## License

[MIT](LICENSE)
