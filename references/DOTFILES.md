# Dotfiles and Personalization

**Reference for:** Personalizing dev containers with dotfiles, shell configurations, and custom tool setups.

## Overview

Dotfiles are configuration files (named with a leading dot, like `.bashrc`, `.zshrc`) that customize your shell, editor, and command-line tools. The Dev Containers extension provides built-in support for automatically applying your dotfiles to any container.

**Benefits:**
- **Consistent environment** across all dev containers
- **Shell customization** (aliases, functions, prompts)
- **Tool configuration** (git, vim, tmux, etc.)
- **Personal productivity setup** without modifying project devcontainer.json

## How DEV Containers Dotfiles Support Works

VS Code can automatically clone a dotfiles repository and run an installation script when a container is created.

### Built-in Dotfiles Properties

Three user settings control dotfile integration:

1. **dotfiles.repository** - GitHub repository URL containing your dotfiles
2. **dotfiles.targetPath** - Where to clone the repository in the container
3. **dotfiles.installCommand** - Script to run after cloning

### Configuration

**settings.json (User scope):**

```json
{
  "dotfiles.repository": "https://github.com/yourusername/dotfiles",
  "dotfiles.targetPath": "~/dotfiles",
  "dotfiles.installCommand": "~/dotfiles/install.sh"
}
```

Or via VS Code Settings UI:

1. **Open Settings:** `Cmd/Ctrl + ,`
2. **Search:** "dotfiles"
3. **Configure:**
   - **Repository:** `yourusername/dotfiles` or `https://github.com/yourusername/dotfiles`
   - **Target Path:** `~/dotfiles`
   - **Install Command:** `install.sh` or `~/dotfiles/install.sh`

**From this point forward, dotfiles are applied to all containers you create.**

## Creating a Dotfiles Repository

### Option 1: Minimal Dotfiles Repo

Create a simple repository with essential configs:

```
dotfiles/
├── .bashrc
├── .zshrc
├── .gitconfig
├── .vimrc
└── install.sh
```

**install.sh:**
```bash
#!/bin/bash
set -e

# Symlink dotfiles
ln -sf ~/dotfiles/.bashrc ~/.bashrc
ln -sf ~/dotfiles/.zshrc ~/.zshrc
ln -sf ~/dotfiles/.gitconfig ~/.gitconfig
ln -sf ~/dotfiles/.vimrc ~/.vimrc

echo "Dotfiles installed!"
```

Make it executable:
```bash
chmod +x install.sh
```

### Option 2: Bootstrap-style Repository

Use a more sophisticated setup script:

```
dotfiles/
├── bash/
│   ├── .bashrc
│   └── .bash_aliases
├── zsh/
│   ├── .zshrc
│   └── .zsh_aliases
├── git/
│   └── .gitconfig
├── vim/
│   └── .vimrc
└── bootstrap.sh
```

**bootstrap.sh:**
```bash
#!/bin/bash
set -e

DOTFILES_DIR="$HOME/dotfiles"

# Detect shell
if [ -n "$ZSH_VERSION" ]; then
  SHELL_TYPE="zsh"
elif [ -n "$BASH_VERSION" ]; then
  SHELL_TYPE="bash"
else
  SHELL_TYPE="bash"
fi

# Link shell configs
ln -sf "$DOTFILES_DIR/$SHELL_TYPE/.${SHELL_TYPE}rc" "$HOME/.${SHELL_TYPE}rc"
ln -sf "$DOTFILES_DIR/$SHELL_TYPE/.${SHELL_TYPE}_aliases" "$HOME/.${SHELL_TYPE}_aliases"

# Link other configs
ln -sf "$DOTFILES_DIR/git/.gitconfig" "$HOME/.gitconfig"
ln -sf "$DOTFILES_DIR/vim/.vimrc" "$HOME/.vimrc"

# Install tools
if ! command -v fzf &> /dev/null; then
  echo "Installing fzf..."
  git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
  ~/.fzf/install --all --no-bash --no-zsh
fi

echo "✓ Dotfiles bootstrapped for $SHELL_TYPE"
```

Configuration:
```json
{
  "dotfiles.repository": "yourusername/dotfiles",
  "dotfiles.targetPath": "~/dotfiles",
  "dotfiles.installCommand": "~/dotfiles/bootstrap.sh"
}
```

## Common Dotfiles Content

### Bash Configuration (.bashrc)

```bash
# ~/.bashrc

# Aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias g='git'
alias dc='docker compose'
alias k='kubectl'

# Custom prompt
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# Environmental variables
export EDITOR=vim
export VISUAL=vim

# Functions
mkcd() {
  mkdir -p "$1" && cd "$1"
}

# Git shortcuts
alias gs='git status'
alias gp='git pull'
alias gc='git commit'
alias gco='git checkout'
```

### Zsh Configuration (.zshrc)

```bash
# ~/.zshrc

# Oh My Zsh (if using)
export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="robbyrussell"
plugins=(git docker kubectl)
source $ZSH/oh-my-zsh.sh

# Aliases
alias ll='ls -alF'
alias dc='docker compose'
alias k='kubectl'

# Custom functions
mkcd() {
  mkdir -p "$1" && cd "$1"
}

# History
HISTSIZE=10000
SAVEHIST=10000
setopt HIST_IGNORE_ALL_DUPS
setopt HIST_FIND_NO_DUPS
```

### Git Configuration (.gitconfig)

```ini
[user]
  name = Your Name
  email = your.email@example.com

[core]
  editor = vim
  autocrlf = input

[alias]
  co = checkout
  br = branch
  ci = commit
  st = status
  unstage = reset HEAD --
  last = log -1 HEAD
  lg = log --oneline --graph --decorate

[pull]
  rebase = true

[init]
  defaultBranch = main
```

### Vim Configuration (.vimrc)

```vim
" ~/.vimrc

" Basic settings
set number          " Line numbers
set relativenumber  " Relative line numbers
set expandtab       " Use spaces instead of tabs
set tabstop=2       " Tab width
set shiftwidth=2    " Indent width
set autoindent      " Auto-indent
set smartindent     " Smart indent

" Search
set ignorecase      " Case-insensitive search
set smartcase       " Case-sensitive if uppercase used
set hlsearch        " Highlight search results
set incsearch       " Incremental search

" Colors
syntax on
set background=dark
```

## Advanced Patterns

### Conditional Configuration (OS-specific)

```bash
# .bashrc

# Detect OS
if [[ "$OSTYPE" == "darwin"* ]]; then
  # macOS-specific
  alias ls='ls -G'
  export BROWSER=open
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
  # Linux-specific
  alias ls='ls --color=auto'
  export BROWSER=xdg-open
fi
```

### Tool Installation

```bash
# install.sh

# Install starship prompt
if ! command -v starship &> /dev/null; then
  curl -sS https://starship.rs/install.sh | sh -s -- -y
  echo 'eval "$(starship init bash)"' >> ~/.bashrc
fi

# Install fzf (fuzzy finder)
if [ ! -d ~/.fzf ]; then
  git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
  ~/.fzf/install --all
fi

# Install exa (modern ls)
if ! command -v exa &> /dev/null && command -v cargo &> /dev/null; then
  cargo install exa
fi
```

### Private Dotfiles (Secrets)

For configs with secrets, use a private repository:

```json
{
  "dotfiles.repository": "git@github.com:yourusername/private-dotfiles.git",
  "dotfiles.targetPath": "~/dotfiles",
  "dotfiles.installCommand": "~/dotfiles/install.sh"
}
```

Ensure SSH keys are available in the container (see [SECURITY.md](./SECURITY.md)).

## Popular Dotfiles Frameworks

### Oh My Zsh

Zsh configuration framework with themes and plugins:

**install.sh:**
```bash
#!/bin/bash
# Install Oh My Zsh
if [ ! -d ~/.oh-my-zsh ]; then
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
fi

# Link custom .zshrc
ln -sf ~/dotfiles/.zshrc ~/.zshrc
```

**.zshrc:**
```bash
export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="agnoster"
plugins=(git docker kubectl npm)
source $ZSH/oh-my-zsh.sh
```

### Homebrew Bundle (macOS)

For macOS-specific tools:

**Brewfile:**
```ruby
brew "fzf"
brew "ripgrep"
brew "bat"
brew "exa"
cask "iterm2"
```

**install.sh:**
```bash
if [[ "$OSTYPE" == "darwin"* ]]; then
  brew bundle --file=~/dotfiles/Brewfile
fi
```

### Dotbot

Automated dotfile installation framework:

```yaml
# install.conf.yaml
- clean: ['~']
- link:
    ~/.bashrc: bash/.bashrc
    ~/.zshrc: zsh/.zshrc
    ~/.gitconfig: git/.gitconfig
    ~/.vimrc: vim/.vimrc
- shell:
  - [git clone https://github.com/junegunn/fzf.git ~/.fzf, Installing fzf]
```

## Extension Synchronization

VS Code can sync extensions separately from dotfiles using Settings Sync:

1. **Enable Settings Sync:** Command Palette → `Settings Sync: Turn On...`
2. **Sign in** with GitHub/Microsoft account
3. **Select what to sync:** Extensions, Settings, Keybindings, etc.

**Note:** Settings Sync is separate from dotfiles. Use both for complete environment replication.

## Troubleshooting

**Dotfiles not applied**
- Check settings: `dotfiles.repository`, `dotfiles.targetPath`, `dotfiles.installCommand`
- Rebuild container: `Dev Containers: Rebuild Container`
- Check container logs: `Dev Containers: Show Container Log`

**Install script fails**
- Ensure script is executable: `chmod +x install.sh`
- Check script for errors: Test locally in a container
- Add debugging: `set -x` at top of script

**Permission denied**
- Script may run as wrong user
- Use `postCreateCommand` for additional setup:
  ```json
  {
    "postCreateCommand": "chmod +x ~/.bashrc && source ~/.bashrc"
  }
  ```

**Git clone fails**
- For private repos, ensure SSH keys or tokens are available
- Test: `git ls-remote <your-repo-url>`

## Best Practices

1. **Keep dotfiles public-friendly**
   - Don't commit secrets or API keys
   - Use separate private repo for sensitive configs

2. **Make install script idempotent**
   - Script should be safe to run multiple times
   - Check if tools exist before installing

3. **Test in a clean container**
   ```bash
   docker run -it --rm ubuntu:22.04 bash
   # Test your install script
   ```

4. **Document requirements**
   - Add README to dotfiles repo
   - List what the install script does

5. **Version control everything**
   - Commit all changes to dotfiles repo
   - Tag releases for stability

## Example Dotfiles Repositories

Inspiration from popular dotfiles repos:

- [mathiasbynens/dotfiles](https://github.com/mathiasbynens/dotfiles) - Extensive macOS setup
- [paulirish/dotfiles](https://github.com/paulirish/dotfiles) - Bash/Zsh with tool installations
- [holman/dotfiles](https://github.com/holman/dotfiles) - Modular approach with topics
- [thoughtbot/dotfiles](https://github.com/thoughtbot/dotfiles) - Minimal, well-organized
- [cowboy/dotfiles](https://github.com/cowboy/dotfiles) - OS-aware configuration

## Related Documentation

- [Dotfiles on GitHub](https://dotfiles.github.io/)
- [Personalizing with Dotfiles (VS Code)](https://code.visualstudio.com/docs/devcontainers/containers#_personalizing-with-dotfile-repositories)
- [Settings Sync](https://code.visualstudio.com/docs/editor/settings-sync)
- [Oh My Zsh](https://ohmyz.sh/)
