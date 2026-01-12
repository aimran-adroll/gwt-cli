# gwt - Git WorkTree Manager

A CLI tool for managing git worktrees with minimal friction. There are many similar ones but this one is mine. Built for developers juggling multiple branches, running parallel AI agents, or reviewing PRs without disrupting their flow.

### AI Disclosure: 
A large portion of the code was authored by Opus 4.5 with exhaustive product guidance from me

## Why gwt?

Git worktrees let you have multiple branches checked out simultaneously. But the vanilla experience has friction:

- Remembering paths and flags
- Managing bare repos manually
- Cleaning up stale worktrees

`gwt` wraps all of this into simple commands:

```bash
gwt clone git@github.com:user/repo.git
cd repo/main

gwt add feat-auth           # new feature branch
gwt switch coworker-branch  # existing branch
gwt agent pr-review-42      # ephemeral worktree

gwt status                  # health check all worktrees
gwt clean                   # nuke temporary worktrees
```

## Installation

### From Git

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/gwt.git ~/.local/share/gwt

# Make scripts executable
chmod +x ~/.local/share/gwt/gwt*

# Add to PATH (choose one)

# Option A: Symlink (recommended)
ln -s ~/.local/share/gwt/gwt ~/.local/bin/gwt

# Option B: Add to PATH in your shell config
# For bash (~/.bashrc):
export PATH="$HOME/.local/share/gwt:$PATH"

# For zsh (~/.zshrc):
export PATH="$HOME/.local/share/gwt:$PATH"

# For fish (~/.config/fish/config.fish):
fish_add_path ~/.local/share/gwt
```

### Verify Installation

```bash
gwt help
```

## Quick Start

### Fresh Clone

```bash
gwt clone git@github.com:user/repo.git
cd repo/main
```

This creates:

```
repo/
├── .git            # pointer to .bare
├── .bare/          # bare git repository
└── main/           # your working directory
```

### Migrate Existing Repo

```bash
cd ~/dev/existing-repo
gwt init
cd main
```

## Commands

| Command | Description |
|---------|-------------|
| `gwt clone <url> [name]` | Clone with bare + worktree structure |
| `gwt init` | Convert existing clone to gwt structure |
| `gwt add <branch> [base]` | Create new branch + worktree |
| `gwt switch <branch>` | Create worktree for existing branch |
| `gwt agent <branch> [base]` | Create ephemeral worktree in ~/.worktrees |
| `gwt ls` | List all worktrees |
| `gwt status` | Show status across all worktrees |
| `gwt rm <branch\|path>` | Remove worktree (and branch if merged) |
| `gwt clean` | Remove all ephemeral worktrees |
| `gwt help [command]` | Show help |

## Workflow

### Feature Development

```bash
# Start a new feature
gwt add feat-auth origin/main
cd ../feat-auth

# ... work work work ...

# Done? Clean up
cd ../main
gwt rm feat-auth
```

### Parallel AI Agents

```bash
# Spin up isolated worktrees for each agent
gwt agent agent-task-1
gwt agent agent-task-2
gwt agent agent-task-3

# Point each Claude/AI session at a different worktree
# ~/.worktrees/repo--agent-task-1/
# ~/.worktrees/repo--agent-task-2/
# ~/.worktrees/repo--agent-task-3/

# When done, clean up all at once
gwt clean
```

### Code Review

```bash
# Check out a coworker's branch without disrupting your work
gwt switch feature-to-review
cd ../feature-to-review

# Review, run tests, etc.

# Clean up
cd ../main
gwt rm feature-to-review
```

### Health Check

```bash
gwt status
```

Shows for each worktree:
- Uncommitted changes (staged/unstaged)
- Untracked files
- Ahead/behind remote

## Best Practices

### Use `origin/*` for base branches

```bash
gwt add feat-x origin/main      # fresh from remote
gwt add feat-x main             # might be stale
```

The scripts fetch before branching, so `origin/*` refs are always current.

### Fetch early, fetch often

`git fetch` is safe and updates remote refs for all worktrees at once. Many gwt commands fetch automatically, but when in doubt:

```bash
git fetch origin
```

### Naming conventions

For ephemeral worktrees, use descriptive prefixes:

```bash
gwt agent pr-review-123
gwt agent claude-refactor-auth
gwt agent experiment-new-api
```

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `GWT_WORKTREE_DIR` | `~/.worktrees` | Where ephemeral worktrees are created |

## Directory Structure

After using gwt, your project looks like:

```
~/dev/
└── myproject/
    ├── .git                # file: "gitdir: .bare"
    ├── .bare/              # bare repository
    ├── main/               # main branch worktree
    ├── feat-auth/          # feature worktree
    └── feat-payments/      # another feature

~/.worktrees/
├── myproject--agent-task-1/    # ephemeral
└── myproject--pr-review-42/    # ephemeral
```

## License

MIT

## Contributing

Issues and PRs welcome. Keep it simple.
