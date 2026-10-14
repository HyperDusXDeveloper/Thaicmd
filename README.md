# ThaiCMD

**English** · [ภาษาไทย](README.th.md)

A Windows terminal that types and displays Thai correctly. Under the hood it runs the real `cmd.exe` through ConPTY, so every command works the same as in Command Prompt, including full-screen programs like **Claude Code CLI**.

ThaiCMD also explains errors in Thai and has a step-by-step Python debugger with Thai output, so beginners can see what went wrong and how to fix it.

![Thai text rendered as whole words, with vowels and tone marks in the right place](docs/thai-rendering.png)

## Why

Thai has vowels and tone marks that stack above and below consonants (e.g. `น้ำ`, `ผู้หญิง`). The classic Windows console and many terminals place each of these in its own cell, which leads to:

- floating vowels and tone marks
- a cursor that drifts and deletes the wrong character
- Thai input that doesn't register in some full-screen apps

These problems are especially noticeable when chatting with Claude Code in Thai. ThaiCMD fixes them at the rendering and input level while keeping the real Windows console underneath.

## Features

**Thai support**
- Type Thai directly into the terminal, at the CMD prompt and inside Claude Code
- Thai words are drawn as whole words, so vowels and tone marks are shaped correctly
- Bundled condensed Thai font (Noto Sans Thai at 80% width) so Thai words fit the terminal grid
- Automatically matches how the console measures Thai combining marks (see [How Thai rendering works](#how-thai-rendering-works))

**Built for Claude Code**
- `Shift+Enter` inserts a new line without sending
- The mouse wheel scrolls Claude's view (sends PgUp/PgDn instead of arrow keys, which would cycle through prompt history)
- Synchronized output, so the screen doesn't flicker while Claude streams a reply
- Drag a file onto the window to paste its path

**Thai-language help for errors**
- An error card appears automatically when a Python, JavaScript or CMD error shows up, explaining the cause and how to fix it in Thai
- `อธิบาย` (`explain-th`) explains any error message you copy or type
- `ดีบัก file.py` (`debug-th`) runs a Python file line by line and shows every variable change, function call and return value in Thai

**A full terminal**
- Tabs and split panes
- Search, clickable links (Ctrl+click), and rectangular selection (Alt+drag)
- Text reflows when you resize the window
- GPU rendering (WebGL), with a software fallback
- Profiles for CMD, Windows PowerShell, PowerShell 7, WSL and Git Bash
- Explorer context menu: "Open ThaiCMD here" for folders, "Run in ThaiCMD" for `.bat` `.cmd` `.py` `.ps1` `.js`
- Taskbar progress (OSC 9;4), bell, OSC 52 clipboard, inline images (Sixel/iTerm), screen reader mode
- Black (Campbell, like CMD) and white themes, adjustable opacity, font, size and cursor style

| Claude Code | Error card | Thai debugger |
|---|---|---|
| ![Claude Code running in ThaiCMD](docs/claude-cli.png) | ![Thai error explanation card](docs/error-card.png) | ![Step-by-step Python debugger with Thai output](docs/thai-debugger.png) |

## Install

Requires Windows 10 version 1809 or later, or Windows 11 (64-bit).

Download from the [Releases](../../releases) page. There are two options:

| File | Size | What it does |
|---|---|---|
| `ThaiCMD-2.0.0-Setup.zip` | ~2 MB | Extract and run `Install-ThaiCMD.bat`. It downloads the Electron runtime (~150 MB) from the official Electron GitHub release, verifies its SHA-256, and installs to `%LOCALAPPDATA%\Programs\ThaiCMD`. No admin rights needed. Adds Start Menu and Desktop shortcuts and an uninstall entry in Settings → Apps. |
| `ThaiCMD-2.0-win-x64.zip` | ~140 MB | Portable. Extract anywhere and run `ThaiCMD.exe`. |

The app is not code-signed yet. If Windows SmartScreen shows "Windows protected your PC", click **More info → Run anyway**.

Python is only needed for the `ดีบัก` command, since it debugs Python code.

## Usage

### With Claude Code

```
claude
```

Then type in Thai directly. `Shift+Enter` adds a new line, and the mouse wheel or PgUp/PgDn scrolls the conversation.

### Thai commands

These work inside any CMD tab. Each has an English alias.

| Command | Alias | Description |
|---|---|---|
| `ดีบัก file.py [steps]` | `debug-th` | Run a Python file line by line with Thai explanations |
| `อธิบาย [error text]` | `explain-th` | Explain an error in Thai. With no text, reads the clipboard |
| `ช่วย` | `help-th` | List commands and shortcuts |

### Keyboard shortcuts

| Keys | Action |
|---|---|
| `Ctrl+C` | Copy if text is selected, otherwise send Ctrl+C to the program |
| `Ctrl+V`, `Shift+Insert` | Paste (asks first when pasting multiple lines into CMD) |
| Right-click | Copy if text is selected, otherwise paste (like CMD) |
| `Alt` + drag | Rectangular selection |
| `Ctrl+Shift+A` | Select all |
| `Ctrl+Shift+F` | Find (`Ctrl+F` also works in CMD) |
| `Ctrl` + click | Open link |
| `Ctrl+Shift+T` | New tab; `Ctrl+Shift+1…9` opens a specific profile |
| `Ctrl+Tab` | Next tab; `Ctrl+Alt+1…9` jumps to a tab |
| `Alt+Shift+=` / `Alt+Shift+-` | Split side by side / top and bottom; `Alt+Shift+D` splits automatically |
| `Alt` + arrows | Move between panes; `Alt+Shift` + arrows resizes |
| `Ctrl+Shift+W` | Close pane or tab |
| `Ctrl+Shift+N` | New window |
| `F11`, `Alt+Enter` | Full screen |
| `Ctrl` + wheel, `Ctrl+=` / `Ctrl+-` / `Ctrl+0` | Zoom in / out / reset |
| `Shift+PgUp` / `Shift+PgDn` | Scroll by page |
| `Ctrl+Shift+K` | Clear screen and scrollback |
| `Ctrl+Break` | Send Ctrl+Break |

### Command-line options

```
ThaiCMD.exe --cwd "D:\work"           open in a folder
ThaiCMD.exe --run "D:\work\test.py"   run a file
ThaiCMD.exe --profile powershell      open with a profile (cmd, powershell, pwsh, wsl, gitbash)
```

Launching ThaiCMD again while it's running opens a new window in the same process.

### Settings

All settings are in the **⋯** menu in the top-right corner. They are stored in `%APPDATA%\ThaiCMD\settings.json`.

## How Thai rendering works

A terminal is a grid of cells, and the program running in it (through ConPTY) decides which cell each character goes in. Any disagreement about character width shows up as misplaced text and a drifting cursor. ThaiCMD handles this in three layers:

1. **Width model.** A custom xterm.js Unicode provider (based on xterm.js's Unicode 11 tables) treats Thai above/below vowels and tone marks as zero-width, joining them to the preceding consonant. Older Windows consoles count them as one cell each, so on startup ThaiCMD measures the console once and uses a matching "cell" mode for plain CMD output. Full-screen and bracketed-paste apps such as Claude Code always use the zero-width model, because that's what they assume.
2. **Shaping.** A character joiner groups each run of Thai letters so the WebGL renderer draws it as one shaped string. This keeps `น้ำ` and `ผู้หญิง` correct, while the starting cell of each word stays exactly where the program put it.
3. **Font metrics.** Regular Thai glyphs are wider than a monospace cell, so they overflow into the next word. ThaiCMD bundles "ThaiCMD Thai", a static instance of Noto Sans Thai at 80% width generated with fontTools, so an average Thai letter fits inside one Consolas cell.

## Limitations

- **Default terminal:** ThaiCMD can't be set as the Windows default terminal application. Windows only allows MSIX-packaged apps that register a handoff COM server. Double-clicking a `.bat` file still opens the classic console; use "Run in ThaiCMD" from the context menu instead.
- **`start cmd`:** opens a classic console window.
- **Without WebGL:** if WebGL is disabled entirely, the DOM renderer is used, and it doesn't support character joiners, so Thai looks letter-spaced.
- **Size:** about 325 MB on disk, because it ships the Electron runtime.

## Build from source

Requires Node.js 22 or later.

```
npm install
npm start        # build and run
npm run dist     # build out/ThaiCMD (portable), out/ThaiCMD-win-x64.zip and out/ThaiCMD-<version>-Setup.zip
```

`npm run dist` can run on Linux: node-pty ships prebuilt Windows binaries, and the exe icon and version info are set with [resedit](https://github.com/jet2jet/resedit-js), so Wine isn't needed.

### Project structure

```
src/main/          Electron main process: windows, ConPTY sessions with batching and flow control,
                   settings, menus, profiles, Explorer integration
src/renderer/      Terminal UI: xterm.js, Thai Unicode provider, tabs, split panes, search, error card
src/shared/        thai-errors.js: Thai explanations for Python, JavaScript and CMD errors
src/tools/         explain / help commands (run with ELECTRON_RUN_AS_NODE, no Node install needed)
tools/             debug_runner.py: the Thai step-by-step Python debugger
installer/         PowerShell installer and uninstaller
scripts/           build (esbuild) and packaging
```

## Tech stack

- [Electron](https://www.electronjs.org/)
- [xterm.js](https://xtermjs.org/) with the WebGL, search, web-links, clipboard and image addons
- [node-pty](https://github.com/microsoft/node-pty) with the bundled ConPTY (`conpty.dll` + `OpenConsole.exe` from Windows Terminal)
- [esbuild](https://esbuild.github.io/), [@electron/packager](https://github.com/electron/packager)

## License

MIT. See [LICENSE.txt](LICENSE.txt).

Third-party components and fonts are listed in [THIRD_PARTY_NOTICES.txt](THIRD_PARTY_NOTICES.txt): xterm.js, node-pty and Electron (MIT), Windows Terminal's ConPTY (MIT), and Noto Sans Thai, Poppins and JetBrains Mono (SIL Open Font License 1.1).
