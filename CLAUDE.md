# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
make          # Build the kilo binary (uses cc with -Wall -W -pedantic -std=c99)
make clean    # Remove the compiled binary
./kilo <filename>  # Run the editor
```

There is no test suite, linter, or formatter configured for this project.

## Architecture

Kilo is a minimal terminal text editor written in a single C file (`kilo.c`, ~1330 lines). It has zero external dependencies — no curses/ncurses — and uses VT100 escape sequences directly for terminal control.

**Global state**: All editor state lives in a single `struct editorConfig E` global, holding cursor position, screen dimensions, scroll offsets, the file's row array, dirty flag, filename, and syntax context.

**Key data structures**:
- `erow` — one row of text, storing both raw chars and a rendered version (with TAB expansion), plus a per-character syntax highlight type array
- `abuf` — append buffer used to batch terminal writes for efficient screen refresh
- `editorSyntax` / `HLDB[]` — syntax highlighting definitions for C/C++, Python, Java, Go

**Control flow** (`main` at line 1310): init editor → open file → enable raw terminal mode → enter loop of `editorRefreshScreen()` + `editorProcessKeypress()`.

**Key keybindings**: Ctrl-S save, Ctrl-Q quit, Ctrl-F find, Ctrl-U/D/R/L cursor movement (also arrow keys).

**Platform**: POSIX (Linux, macOS, BSD). Requires `termios.h` for raw mode terminal control and handles `SIGWINCH` for terminal resize.
