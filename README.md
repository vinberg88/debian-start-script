# Debian 13 / 14 WSL2 systemd namespace 

Start any desktop via DEBIAN 13 / 14 and WSL2 and WINDOWS 11 =}

<p align="center">
<a href="https://github.com/vinberg88/debian">
<img width="850" height="250" alt="Debian" src="https://github.com/user-attachments/assets/3e029862-0459-41ec-8ae5-0cc82e181636" />
</p>

---

Minimal start scripts for Debian 13 (Trixie) and Debian 14 (Forky) on WSL2.
They keep the namespace-based systemd startup used by the original
[Ubuntu script](https://github.com/vinberg88/ubuntu24.04-wsl2-systemd-script).

Debian 14 is still under development, so its support should be treated as
provisional until the final release is available.

This is the original Classic compatibility mode. Do not run it together with
WSL's native systemd mode. `/etc/wsl.conf` must contain `systemd=false` under
`[boot]` before the final WSL restart.

## Install

Run inside Debian as a normal user in the `sudo` group:

```bash
sudo apt update
sudo apt install -y git daemonize procps sudo systemd util-linux

cd ~
git clone https://github.com/vinberg88/debian-start-script.git
cd debian-start-script

sudo install -o root -g root -m 0644 start-systemd-namespace /usr/sbin/start-systemd-namespace
sudo install -o root -g root -m 0755 enter-systemd-namespace /usr/sbin/enter-systemd-namespace

sudo tee /etc/sudoers.d/systemd-namespace >/dev/null <<'EOF'
Defaults env_keep += WSLPATH
Defaults env_keep += WSLENV
Defaults env_keep += WSL_INTEROP
Defaults env_keep += WSL_DISTRO_NAME
Defaults env_keep += PRE_NAMESPACE_PATH
Defaults env_keep += PRE_NAMESPACE_PWD
%sudo ALL=(ALL) NOPASSWD: /usr/sbin/enter-systemd-namespace
EOF
sudo chmod 0440 /etc/sudoers.d/systemd-namespace
sudo visudo -cf /etc/sudoers.d/systemd-namespace

if grep -qE '^[[:space:]]*systemd[[:space:]]*=' /etc/wsl.conf 2>/dev/null; then
  sudo sed -i 's/^[[:space:]]*systemd[[:space:]]*=.*/systemd=false/' /etc/wsl.conf
elif grep -q '^\[boot\]' /etc/wsl.conf 2>/dev/null; then
  sudo sed -i '/^\[boot\]/a systemd=false' /etc/wsl.conf
else
  printf '\n[boot]\nsystemd=false\n' | sudo tee -a /etc/wsl.conf >/dev/null
fi

sudo sed -i '/^[[:space:]]*source \/usr\/sbin\/start-systemd-namespace[[:space:]]*$/d' /etc/bash.bashrc
grep -qxF '[ -r /usr/sbin/start-systemd-namespace ] && source /usr/sbin/start-systemd-namespace' /etc/bash.bashrc ||
  printf '\n# Start or enter the systemd namespace in WSL2\n[ -r /usr/sbin/start-systemd-namespace ] && source /usr/sbin/start-systemd-namespace\n' |
  sudo tee -a /etc/bash.bashrc >/dev/null
```

Close Debian, run `wsl --shutdown` in Windows PowerShell, and open Debian again.
Verify with:

```bash
ps -p 1 -o pid,comm,args
```

A working namespace shows `/lib/systemd/systemd --unit=basic.target` as PID 1.

---

<p align="center">
<a href="https://github.com/vinberg88">
<img width="1013" height="243" alt="Github" src="https://github.com/user-attachments/assets/e52d4101-bf08-4cdf-ae06-13cb4890d0b9" />
</p>

---

## Optional Budgie desktop

The smaller Budgie installation used for the verified Debian 13 test was:

```bash
sudo apt update
sudo apt install budgie-desktop
sudo apt install budgie*
```

GNOME Terminal is a reliable optional terminal for this setup:

```bash
sudo apt install gnome-terminal
```

For the complete package selection, including suggested packages (this can use
considerably more disk space):

```bash
sudo apt install --install-suggests budgie-desktop
```

With a Windows X server already running, start the X11 session from the Debian
terminal entered through the systemd namespace:

```bash
export DISPLAY="$(ip route show default | awk '/default/ {print $3; exit}'):0.0"
export XDG_SESSION_TYPE=x11
export GDK_BACKEND=x11
unset WAYLAND_DISPLAY
budgie-desktop
```

Only the two start helpers are included in this repository. Desktop packages
are optional and are never installed by the scripts.

Tested on DEBIAN 13 and works well via BUDIGE desktop for WSL2. 

Look here: https://github.com/vinberg88/debian/blob/main/DEBIAN-13-BUDGIE.txt

<p align="center">
<a href="https://github.com/vinberg88/debian">
<img width="1920" height="1080" alt="Debian-13-BUDGIE" src="https://github.com/user-attachments/assets/e27e669b-a619-443a-a680-19af6c215b05" />
</p>

Find more desktops here for WSL and DEBIAN 13 and DEBAIN 14 

<p align="center">
<a href="https://github.com/vinberg88/debian">
<img width="681" height="181" alt="DEBIAN" src="https://github.com/user-attachments/assets/a1ca75dd-5370-4424-a3e4-0dfca213b03e" />
</p>

Regards,
Mattias Vinberg - StockHolm - Sweden - WSL - Microsoft - Lnux - Debian - 2026
