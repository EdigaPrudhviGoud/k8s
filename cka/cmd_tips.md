echo $SHELL
If the output is something like /bin/bash, you're using Bash, and your configuration file is likely ~/.bashrc.
If the output is /bin/zsh, you're using Zsh, and your configuration file is ~/.zshrc.

alias k='kubectl'

Reload after adding alias:
source ~/.bashrc   # For Bash
source ~/.zshrc    # For Zsh
