# CLAUDE.md

This file helps AI assistants understand the gwt codebase.

## What is gwt?

`gwt` (Git WorkTree) is a CLI tool that simplifies git worktree workflows, especially for:
- Parallel AI agent tasks
- Code review isolation
- Multi-branch development

## Architecture

```
gwt                     # Main dispatcher script
gwt-<command>           # Individual command scripts
```

The dispatcher (`gwt`) parses the first argument and `exec`s the corresponding `gwt-<command>` script. All scripts are bash.

## Key Concepts

### Project Structure (after `gwt clone`)

```
project/
├── .git                # File pointing to .bare
├── .bare/              # Bare git repository
└── main/               # Worktree (checked out branch)
    └── .git            # File pointing to ../.bare/worktrees/main
```

### Worktree Types

1. **Persistent** (`gwt add`): Created under project root, for active feature work
2. **Ephemeral** (`gwt agent`): Created in `~/.worktrees/`, for temporary tasks

### Finding Project Root

Scripts detect project root by searching upward for `.bare/` directory:

```bash
find_project_root() {
    local dir=$(pwd)
    while [[ "$dir" != "/" ]]; do
        if [[ -d "$dir/.bare" ]]; then
            echo "$dir"
            return 0
        fi
        dir=$(dirname "$dir")
    done
    # Fallback for traditional (non-bare) layout
    local wt_root
    wt_root=$(git rev-parse --show-toplevel 2>/dev/null) || return 1
    dirname "$wt_root"
}
```

## Commands

| Command | Script | Purpose |
|---------|--------|---------|
| `clone` | gwt-clone | Clone repo with bare + worktree structure |
| `init` | gwt-init | Convert existing clone to gwt structure |
| `add` | gwt-add | Create new branch + persistent worktree |
| `switch` | gwt-switch | Create worktree for existing branch |
| `agent` | gwt-agent | Create ephemeral worktree in ~/.worktrees |
| `ls` | gwt-ls | List all worktrees |
| `status` | gwt-status | Show status across all worktrees |
| `rm` | gwt-rm | Remove worktree and optionally branch |
| `clean` | gwt-clean | Remove all ephemeral worktrees |
| `help` | gwt-help | Show help |

## Code Patterns

### Color Output

All scripts use this pattern for terminal colors:

```bash
if [[ -t 1 ]]; then
    RED=$'\033[0;31m'
    GREEN=$'\033[0;32m'
    # ... etc
else
    RED='' GREEN='' # ... disabled if not a terminal
fi
```

### Error Handling

```bash
echo -e "${RED}error:${NC} message" >&2
echo -e "${DIM}hint:  helpful suggestion${NC}" >&2
exit 1
```

### Progress Output

```bash
echo -e "${BLUE}::${NC} Doing something..."
```

## Testing Changes

After modifying a script, test from a gwt-managed repo:

```bash
cd /tmp
gwt clone <some-repo-url> test-repo
cd test-repo
gwt ls
gwt add test-branch
gwt status
gwt rm test-branch
```

## Environment Variables

- `GWT_WORKTREE_DIR`: Override ephemeral worktree location (default: `~/.worktrees`)
