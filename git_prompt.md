# Bash Git Prompt


on rhel (part of git)
```
cp /usr/share/git-core/contrib/completion/git-prompt.sh ~/.git-prompt.sh
source ~/.git-prompt.sh

export GIT_PS1_SHOWDIRTYSTATE=1

export PS1='\[\e[0;32m\]\u@\h\[\e[m\] \[\e[0;34m\]\w\[\e[m\]\[\e[0;31m\]$(__git_ps1)\[\e[m\] $ '

```
on others, google it.
