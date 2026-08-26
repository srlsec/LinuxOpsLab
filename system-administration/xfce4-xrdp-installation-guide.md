# Installing XFCE4 Desktop + XRDP on Ubuntu Server (with Troubleshooting)

This guide documents the full process of adding a graphical desktop (XFCE4) and remote desktop access (XRDP) to a headless Ubuntu Server install, including the real issues encountered and how they were fixed.

**Environment:** Ubuntu Server (headless), hostname `server`, user `examadmin`

---

## Part 1: Base Installation

### 1. Bring the network interface up (if not already configured)
```bash
sudo ip link set enp1s0 up
sudo dhclient enp1s0
ip addr    # confirm you got an IP
```

### 2. Install XFCE4 desktop environment
```bash
sudo apt install xfce4 -y
```

### 3. Install XFCE4 goodies, LightDM (display manager), and XRDP together
```bash
sudo apt install xfce4 xfce4-goodies lightdm xrdp -y
```
> Installing `lightdm` here is important — without a display manager, there's no graphical login screen.

### 4. Enable and start XRDP
```bash
sudo systemctl enable --now xrdp
```

### 5. Set the system to boot into graphical mode instead of console
```bash
sudo systemctl set-default graphical.target
```

### 6. Reboot
```bash
sudo reboot
```

---

## Part 2: The Problem — "Failed to start session"

After reboot, logging in as `examadmin` at the graphical login screen (LightDM/unity-greeter) failed with:

```
examadmin
Failed to start session
```

### Diagnosis steps taken

**1. Confirmed the display manager in use**
```bash
journalctl -b -u gdm      # No entries — GDM not in use
```
This ruled out GDM and confirmed LightDM was the active display manager.

**2. Checked for basic filesystem/session error clues**
```bash
ls -la /home/examadmin/
cat ~/.xsession-errors     # No such file — session never got far enough to log anything
```

**3. Checked the actual LightDM log (this is where the real answer was)**
```bash
sudo tail -50 /var/log/lightdm/lightdm.log
```

Found the key lines:
```
Greeter requests session ubuntu
Seat seat0: Failed to find session configuration ubuntu
Seat seat0: Can't find session 'ubuntu'
```

**Root cause identified:** LightDM was trying to launch a session called `ubuntu` (GNOME/Unity), but only `xfce4` had been installed — there was no `ubuntu` session definition available.

**4. Confirmed available session files**
```bash
ls /usr/share/xsessions/
# Output: xfce.desktop
```
Only `xfce.desktop` existed — confirming the correct session name was `xfce`, not `ubuntu`.

---

## Part 3: The Fix

### Fix attempt 1 — update `~/.dmrc` (partial fix, not sufficient alone)
```bash
sed -i 's/Session=ubuntu/Session=xfce/' ~/.dmrc
cat ~/.dmrc
```
Result:
```
[Desktop]
Session=xfce
```

This alone **did not** resolve the issue — the greeter kept requesting `ubuntu` again on the next login attempt.

### Mistake to avoid — do not manually run `startxfce4` from a TTY
While troubleshooting, running:
```bash
startxfce4
```
from an SSH/TTY session while LightDM's X server was still running on `:0` produced:
```
Fatal server error:
Server is already active for display 0
```
This happens because LightDM already owns the display — a second X server can't start on top of it. This is **not** the fix; login must go through the LightDM greeter itself, not a manually started X session.

Cleanup after this mistake:
```bash
sudo rm -f /tmp/.X0-lock
sudo systemctl restart lightdm
```

### Real root cause — AccountsService overrides `.dmrc`

On Ubuntu's `unity-greeter`, the **last selected session is actually stored and read from AccountsService**, not `.dmrc`. This is why `.dmrc` alone didn't fix it.

Checked:
```bash
sudo cat /var/lib/AccountsService/users/examadmin
```
Output showed:
```
[User]
Session=
XSession=ubuntu
Icon=/home/examadmin/.face
SystemAccount=false
```

**This was the actual fix — update `XSession` here:**
```bash
sudo sed -i 's/XSession=ubuntu/XSession=xfce/' /var/lib/AccountsService/users/examadmin
```

Verify:
```bash
sudo cat /var/lib/AccountsService/users/examadmin
```
Should now show:
```
[User]
Session=
XSession=xfce
Icon=/home/examadmin/.face
SystemAccount=false
```

### Apply the fix
```bash
sudo systemctl restart accounts-daemon
sudo systemctl restart lightdm
```

> Note: restarting `lightdm` resets the active login screen — run this from an SSH session, not from the console you're trying to fix.

### Result
Logging in as `examadmin` at the physical console now succeeded, launching the XFCE desktop correctly. Confirmed in the log:
```bash
sudo tail -20 /var/log/lightdm/lightdm.log
```
showed the greeter requesting the `xfce` session instead of `ubuntu`, with no further errors.

---

## Part 4: Alternative Access — XRDP (Remote Desktop)

Since XRDP was already installed and enabled in Part 1, the desktop can also be reached remotely without touching the physical console:

- **Windows:** `Win + R` → `mstsc` → connect to the server's IP (e.g. `192.168.1.15`) → log in as `examadmin`.
- **macOS:** Use "Microsoft Remote Desktop" from the App Store → add PC by IP → connect.
- **Linux:** Use **Remmina** or any RDP client → connect via RDP on port `3389`.

XRDP launches sessions via `/etc/xrdp/startwm.sh`, independent of the local LightDM greeter, so it's a useful fallback if the local session ever breaks again.

---

## Summary of Root Cause & Fix

| Issue | Cause | Fix |
|---|---|---|
| "Failed to start session" | LightDM/unity-greeter requested a nonexistent `ubuntu` session; only `xfce4` was installed | Point the session to `xfce` |
| `.dmrc` edit alone didn't work | unity-greeter reads last session from **AccountsService**, not `.dmrc` | Edit `/var/lib/AccountsService/users/<user>` and set `XSession=xfce` |
| "Server already active for display 0" | Manually ran `startxfce4` while LightDM's X server already owned `:0` | Don't manually start X sessions — let LightDM's greeter launch the session; remove stale `/tmp/.X0-lock` if this happens |

**Key files involved:**
- `/usr/share/xsessions/xfce.desktop` — defines the available session
- `~/.dmrc` — per-user session preference (secondary)
- `/var/lib/AccountsService/users/<username>` — authoritative session preference used by unity-greeter
- `/var/log/lightdm/lightdm.log` — primary log for diagnosing LightDM session failures
