# Debian 13/14 WSL2 systemd namespace

Minimal start scripts for Debian 13 (Trixie) and Debian 14 (Forky) on WSL2.
They keep the namespace-based systemd startup used by the original
[Ubuntu script](https://github.com/vinberg88/ubuntu24.04-wsl2-systemd-script).

Debian 14 is still under development, so its support should be treated as
provisional until the final release is available.

## Install

Run inside Debian as a normal user in the `sudo` group:

```bash
sudo apt update
sudo apt install -y daemonize procps sudo systemd util-linux

sudo install -o root -g root -m 0644 start-systemd-namespace /usr/sbin/start-systemd-namespace
sudo install -o root -g root -m 0755 enter-systemd-namespace /usr/sbin/enter-systemd-namespace

printf '%%sudo ALL=(ALL) NOPASSWD: /usr/sbin/enter-systemd-namespace\n' |
  sudo tee /etc/sudoers.d/systemd-namespace >/dev/null
sudo chmod 0440 /etc/sudoers.d/systemd-namespace
sudo visudo -cf /etc/sudoers.d/systemd-namespace

grep -qxF 'source /usr/sbin/start-systemd-namespace' /etc/bash.bashrc ||
  printf '\n# Start or enter the systemd namespace in WSL2\nsource /usr/sbin/start-systemd-namespace\n' |
  sudo tee -a /etc/bash.bashrc >/dev/null
```

Close Debian, run `wsl --shutdown` in Windows PowerShell, and open Debian again.
Verify with:

```bash
systemctl is-system-running
```

Only the two start helpers are included; desktop installation and launch
commands intentionally stay outside this repository.
