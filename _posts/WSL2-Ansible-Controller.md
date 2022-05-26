---
layout: post
author: ndt
---

# Ansible Controller in WSL2

Ansible cannot be run directly on Windows (who would ever want to do that either? ;-)
But apart from a VM you can run ansible it in WSL2.
You can even also connect to a SSH agent on Windows.

Prerequisites:
- running password manager, e.g. KeePassXC
- running PuTTY Pageant as SSH agent

Start your WSL2 Terminal and run the following command:


# Update system and install packages
```
sudo apt update && sudo apt upgrade && sudo apt install wget socat git python3-pip sshpass
```

# Prepare WSL with pageant socket
As bridge between windows pageant and wsl2 download wsl2-ssh-pageant.exe from https://github.com/BlackReloaded/wsl2-ssh-pageant/.
```
mkdir -p $HOME/.ssh
wget -O $HOME/.ssh/wsl2-ssh-pageant.exe https://github.com/BlackReloaded/wsl2-ssh-pageant/releases/download/v1.2.0/wsl2-ssh-pageant.exe
chmod +x $HOME/.ssh/wsl2-ssh-pageant.exe
```


# Extend profile to start pageant socket on WSL login
```
cat >> $HOME/.profile <<'EOF'

export SSH_AUTH_SOCK="$HOME/.ssh/agent.sock"
if ! ss -a | grep -q "$SSH_AUTH_SOCK"; then
  rm -f "$SSH_AUTH_SOCK"
  wsl2_ssh_pageant_bin="$HOME/.ssh/wsl2-ssh-pageant.exe"
  if test -x "$wsl2_ssh_pageant_bin"; then
    (setsid nohup socat UNIX-LISTEN:"$SSH_AUTH_SOCK,fork" EXEC:"$wsl2_ssh_pageant_bin" >/dev/null 2>&1 &)
  else
    echo >&2 "WARNING: $wsl2_ssh_pageant_bin is not executable."
  fi
  unset wsl2_ssh_pageant_bin
fi

EOF
```

# Activate new profile
```
source $HOME/.profile
```

# Install ansible and friends as python user packages with pip3
```
pip3 install --user ansible ansible-lint molecule
```
