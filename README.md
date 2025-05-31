# apt-up

A comprehensive update script for Debian-based systems, designed to streamline system maintenance.
It can be run as an interactive one-shot or install itself with a config file, logging directory, etc.
**apt-up** handles kernel firmware updates, system package upgrades, Flatpak updates, old kernel & header cleanup, and cache management.
Built with flexibility and extensibility in mind, it supports colored output, interactive and non-interactive modes, custom pre/post run hooks, and detailed logging.

## Features
- **Firmware Updates:** Pulls the latest from linux-firmware via Git (default: GitLab upstream).

- **System Updates:** Runs aptitude full-upgrade for packages.

- **Flatpak Support:** Updates Flatpak apps if installed.

- **Kernel Cleanup:** Removes outdated kernels, keeping the current and previous versions.

- **Cache Cleaning:** Clears apt, apt-get, and aptitude caches.

- **Extensible via Hooks:** Pre, post, and failure hooks in /etc/apt-up.d for customization.

- **Resource Control:** Adjustable nice and ionice for non-interactive runs.

- **Logging:** Optional logs to /var/log/apt-up.log.

## Requirements
- `Debian`-based system (e.g. Debian, Mint, Ubuntu, etc.)

- `Bash` (still basically works with 'sh' though).

- `aptitude` (required, auto-installs if missing).

- `Root` privileges (sudo or direct root).

- **Optional:**

  - `flatpak`

  - `git` (for firmware updates).

  - `ionice` (for I/O priority in non-interactive mode).

## Installation
Download:
```bash
curl -sL https://raw.githubusercontent.com/cwadge/apt-up/main/apt-up -o apt-up
```
Or clone the repo:
```bash
git clone https://github.com/cwadge/apt-up.git
cd apt-up
```

Make executable:
```bash
chmod +x apt-up
```

(Optional but recommended) Install System-Wide:
```bash
sudo mv apt-up /usr/local/sbin/apt-up
```

Or download + install all at once:
```bash
sudo wget https://raw.githubusercontent.com/cwadge/apt-up/main/apt-up -O /usr/local/sbin/apt-up && sudo chmod 755 /usr/local/sbin/apt-up
```

## Usage
Run interactively:
```bash
sudo apt-up
```
Run non-interactively (e.g., cron):
```bash
sudo apt-up --no-interactive
```
### Options
```
--no-interactive     Run without user interaction
--no-firmware        Skip firmware updates
--no-update-system   Skip system package updates
--no-flatpak         Skip Flatpak updates
--no-kernel-cleanup  Skip removal of old kernels
--no-cache-clean     Skip cleaning package caches
--no-hooks           Skip running hook scripts
--update-firmware    Update system firmware
--update-system      Update system packages
--update-flatpak     Perform Flatpak updates
--kernel-cleanup     Remove old kernels and headers
--cache-clean        Clean all apt package caches
--run-hooks          Run scripts in hook directories
--install            Create configuration files and hook directories
--help               Display this help message
```
### Configuration

Edit `/etc/apt-up.conf` (after optional `sudo apt-up --install`).

## Example Run
Here’s what it looks like in action, checking for firmware & system updates, then running a custom hook script:
```
[INFO] Loading configuration from /etc/apt-up.conf
[INFO] Running pre hooks
 ________________________________
/ Exporting global variables...  \
\________________________________/
[INFO] Exporting: IGNORE_CC_MISMATCH=1
[INFO] Exporting: GE_PROTON_TARGET=/home/my_account/.steam/steam/compatibilitytools.d
 ________________________________
/ Checking for updated files...  \
\________________________________/
Hit https://security.debian.org/debian-security bookworm-security InRelease
Hit https://deb.debian.org/debian bookworm InRelease
Hit https://deb.debian.org/debian bookworm-updates InRelease
Hit https://deb.debian.org/debian bookworm-backports InRelease
Hit https://brave-browser-apt-release.s3.brave.com stable InRelease
Hit https://repo.steampowered.com/steam stable InRelease
Hit https://deb.xanmod.org releases InRelease
Hit https://www.deb-multimedia.org bookworm InRelease
Hit https://www.deb-multimedia.org bookworm-backports InRelease
Hit https://download.opensuse.org/repositories/home:/strycore/Debian_12 ./ InRelease

 ________________________________
/ Updating files if necessary... \
\________________________________/
No packages will be installed, upgraded, or removed.
0 packages upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 0 B of archives. After unpacking 0 B will be used.

 ________________________________
/ Updating flatpak packages...   \
\________________________________/
Looking for updates…

Nothing to do.
 ________________________________
/ Updating kernel firmware...    \
\________________________________/
[INFO] Checking for firmware updates...
remote: Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
From https://gitlab.com/kernel-firmware/linux-firmware
 * branch            main       -> FETCH_HEAD
HEAD is now at f2e9c60 Merge branch 'robot/pr-0-1748597148' into 'main'
[INFO] Firmware already up to date.
 ________________________________
/ Cleaning out the apt cache...  \
\________________________________/
[INFO] Finished cleaning cache.
 ________________________________
/ Purge old kernels & headers... \
\________________________________/
[INFO] Current running kernel: 6.14.9-x64v3-xanmod1
[INFO] Latest installed kernel: 6.14.9-x64v3-xanmod1
[INFO] Only one kernel installed, keeping: 6.14.9-x64v3-xanmod1
[INFO] Keeping headers for kernel 6.14.9-x64v3-xanmod1: linux-headers-6.14.9-x64v3-xanmod1
[INFO] No old kernels or headers to remove.
 ________________________________
/ Syncing buffers out to disk... \
\________________________________/
[INFO] Running post hooks
[INFO] Running hook: 50-ge-proton
Checking for GE-Proton updates...
Using overridden Steam directory: /home/my_account/.steam/steam/compatibilitytools.d
Latest version: GE-Proton10-3
GE-Proton10-3 already installed at /home/my_account/.steam/steam/compatibilitytools.d/GE-Proton10-3
No installation needed.
Cleaning up old GE-Proton versions...
No old versions to clean up.
Done!
 ________________________________
/           Finished.            \
\________________________________/
```
(Normally output is color-coded, if the terminal supports it.)

## Hooks
If you want to extend apt-up's functionality, you can add custom scripts to:
- `/etc/apt-up.d/pre.d/` (before updates).

- `/etc/apt-up.d/post.d/` (after updates).

- `/etc/apt-up.d/fail.d/` (on failure).

NOTE: Scripts with '`critical`' in the name (e.g. `00-critical-check.sh`) halt execution if they fail.

Run `sudo apt-up --install` to create these directories (with a sample hook in `pre.d`).

## License

GNU General Public License v2.0 or later ([LICENSE](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)) - contributions must remain open-source.

## Contributing

You know the drill: fork, open an issue or submit a pull request. 

---

_A tool for maintaining Debian-based systems by Chris Wadge_
