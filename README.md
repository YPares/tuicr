# tuicr

Terminal UI for Code Reviews - A Rust TUI for reviewing changes made by coding agents.

## Overview

`tuicr` is a terminal-based code review tool designed for reviewing changes made by AI coding agents like Claude, Codex, or similar tools. It provides a GitHub-style diff viewing experience with vim keybindings, allowing you to:

- View all changed files in a continuous, scrollable side-by-side diff
- Leave comments at the file or line level
- Mark files as reviewed
- Export your review as structured Markdown to feed back to the coding agent

## Features

- **Infinite scroll diff view** - All changed files in one continuous scroll (GitHub-style)
- **Side-by-side diffs** - See old and new versions side by side
- **Vim keybindings** - Navigate with `j/k`, `Ctrl-d/u`, `gg/G`, etc.
- **Comments** - Add file-level or line-level comments with types (Note, Suggestion, Issue, Praise)
- **Review tracking** - Mark files as reviewed, persist progress to disk
- **Markdown export** - Generate structured output optimized for LLM consumption

## Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/tuicr.git
cd tuicr

# Build and install
cargo install --path .
```

## Usage

Run `tuicr` in any git repository with uncommitted changes:

```bash
cd /path/to/your/repo
tuicr
```

### Keybindings

#### Navigation
| Key | Action |
|-----|--------|
| `j/k` | Scroll down/up |
| `Ctrl-d/u` | Half page down/up |
| `gg/G` | Go to first/last file |
| `{/}` | Jump to previous/next file |
| `[[/]]` | Jump to previous/next hunk |
| `Tab` | Toggle focus between file list and diff |

#### Actions
| Key | Action |
|-----|--------|
| `r` | Toggle file reviewed |
| `c` | Add line comment |
| `C` | Add file comment |
| `e` | Edit comment |
| `dd` | Delete comment |

#### Session
| Key | Action |
|-----|--------|
| `:w` | Save session |
| `:e` | Export to markdown |
| `:q` | Quit |
| `?` | Show help |

## Review Output

When you export your review, `tuicr` generates structured Markdown that can be fed directly to a coding agent:

```markdown
# Code Review: myproject

**Files Reviewed:** 3/5

## Action Items

1. **`src/auth.rs`:42** - Magic number should be a named constant
2. **`src/lib.rs`:15** - Consider adding error handling here
```

## Development

```bash
# Run in development
cargo run

# Run tests
cargo test

# Format code
cargo fmt
```
