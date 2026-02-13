# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Meld

Meld is a visual diff and merge tool for GNOME, written in Python using GTK+ 3 and GtkSourceView 4. It supports 2/3-way file comparison, directory comparison, version control integration (Git, Mercurial, SVN, Bazaar, CVS, Darcs), and image comparison.

## Build and Run

```bash
# Run directly without building (development mode)
bin/meld

# Build with Meson
meson setup _build
ninja -C _build
ninja -C _build install
```

## Testing and Linting

```bash
# Run all tests
pytest

# Run a single test file
pytest test/test_filediff.py

# Run a single test
pytest test/test_filediff.py::TestClassName::test_name

# Lint (uses ruff via pre-commit)
pre-commit run --all-files

# Install dev dependencies
pip install -r dev-requirements.txt
```

## Architecture

### Entry Point

`bin/meld` — handles frozen app detection, development vs installed mode, resource compilation, requirements checking, and launches `MeldApp`.

### Core Document Model

All comparison types inherit from `MeldDoc` (`meld/melddoc.py`), which provides signal-based state management (`close`, `create-diff`, `file-changed`, `tab-state-changed`, `move-diff`), action groups, and async scheduling.

- **FileDiff** (`meld/filediff.py`) — 2/3-way file comparison with syntax highlighting, undo/redo, chunk-based diff navigation, and merge operations
- **DirDiff** (`meld/dirdiff.py`) — 2/3-way directory comparison with tree view, file/folder filtering, and VC synchronization
- **VcView** (`meld/vcview.py`) — version control browser with commit/push dialogs
- **ImageDiff** (`meld/imagediff.py`) — side-by-side image comparison

### Application Layer

- **MeldApp** (`meld/meldapp.py`) — `Gtk.Application` subclass handling CLI args and window management
- **MeldWindow** (`meld/meldwindow.py`) — main window with notebook tabs; inserts active document's action group

### Key Subsystems

- `meld/matchers/` — diff algorithms (Myers in `myers.py`, merge logic in `merge.py`, utilities in `diffutil.py`)
- `meld/vc/` — version control backends; all inherit from `_vc.py` base class
- `meld/ui/` — reusable GTK widgets (find bar, cell renderers, VC dialogs, status bar)
- `meld/task.py` — `FifoScheduler`/`LifoScheduler` for non-blocking async operations
- `meld/settings.py` — GSettings integration via `org.gnome.meld` schema
- `meld/resources/` — GResource XML, UI definitions, CSS, icons (compiled at build or runtime)

### Patterns

- **Signal-based communication**: documents emit GObject signals; the window listens and responds
- **Async scheduling**: diff computation uses `FifoScheduler` to avoid blocking the UI
- **Action groups**: each document defines a `view_action_group` that gets inserted into the window when its tab is active
- **Uninstalled mode**: `bin/meld` detects running from source and compiles resources/schemas on-the-fly

## Code Style

- 4-space indentation for Python, 2-space for Meson files
- Ruff for linting (rules: E4, E7, E9, F, I) and import sorting
- UTF-8 encoding, LF line endings, no trailing whitespace
- See `.editorconfig` and `pyproject.toml` [tool.ruff] for details

## Dependencies

- **Runtime**: Python 3.10+, PyGObject 3.38+, pycairo, GTK+ 3.24+, GtkSourceView 4.0+, GLib 2.66+
- **Build**: Meson 1.2+, Ninja, gettext, glib-compile-resources, glib-compile-schemas
- **Dev**: pytest 8.4, ruff 0.14, pre-commit 4.3
