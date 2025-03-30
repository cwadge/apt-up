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
- `Debian`-based system (e.g., Debian, Mint, Ubuntu, etc.)

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

Make Executable:
```bash
chmod +x apt-up
```

(Optional but recommended) Install System-Wide:
```bash
sudo mv apt-up /usr/local/sbin/apt-up
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
--no-interactive     Run without prompts
--no-firmware        Skip firmware updates
--no-system-update   Skip system package updates
--no-flatpak         Skip Flatpak updates
--no-kernel-cleanup  Skip old kernel removal
--no-cache-clean     Skip cache cleaning
--install            Create sample config and hooks
--help               Show help message
```
### Configuration

Edit `/etc/apt-up.conf` (optional, see sample at `/etc/apt-up.conf.sample` after `sudo apt-up --install`).

## Example Run
Here’s what it looks like in action, cleaning up old kernels:
```
[INFO] No configuration file found at /etc/apt-up.conf, using defaults
 ________________________________ 
/ Exporting global variables...  \ 
\________________________________/ 
[INFO] Exporting: IGNORE_CC_MISMATCH=1
 ________________________________ 
/ Updating kernel firmware...    \ 
\________________________________/ 
[INFO] Checking for firmware updates...
...
[INFO] Firmware already up to date.
 ________________________________ 
/ Checking for updated files...  \ 
\________________________________/ 
...
                                    
 ________________________________ 
/ Updating files if necessary... \ 
\________________________________/ 
...
                                         
 ________________________________ 
/ Updating flatpak packages...   \ 
\________________________________/ 
...
 ________________________________ 
/ Cleaning out the apt cache...  \ 
\________________________________/ 
[INFO] Finished cleaning cache.
 ________________________________ 
/ Purge old kernels & headers... \ 
\________________________________/ 
[INFO] Found some old kernels and headers to remove.
The following packages will be REMOVED:  
  linux-image-6.13.4-x64v3-xanmod1  linux-image-6.13.5-x64v3-xanmod1  linux-image-6.13.6-x64v3-xanmod1
 ________________________________ 
/ Syncing buffers out to disk... \ 
\________________________________/ 
[INFO] Synced.
 ________________________________ 
/           Finished.            \ 
\________________________________/ 

```
Normally output is color-coded, if the terminal supports it.

## Hooks
If you want to extend apt-up's functionality, etc. add custom scripts to:
- `/etc/apt-up.d/pre.d/` (before updates).

- `/etc/apt-up.d/post.d/` (after updates).

- `/etc/apt-up.d/fail.d/` (on failure).

Scripts with critical in the name (e.g. `00-critical-check.sh`) halt execution if they fail.

Run `sudo apt-up --install` to create these directories with a sample hook.

## License

GNU General Public License v2.0 or later ([LICENSE](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)) - contributions must remain open-source.

## Contributing

You know the drill: fork, open an issue or submit a pull request. 

---

_A tool for maintaining Debian-based systems by Chris Wadge_
