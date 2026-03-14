# apt-up

A comprehensive update script for Debian-based systems, designed to streamline system maintenance.
It can be run as an interactive one-shot or install itself with a config file, logging directory, etc.
**apt-up** handles kernel firmware updates, system package upgrades, Flatpak updates, old kernel & header cleanup, and cache management.
Built with flexibility and extensibility in mind, it supports colored output, interactive and non-interactive modes, custom pre/post run hooks, and detailed logging.

## Features
- **Firmware Updates:** Pulls the latest from linux-firmware via Git (default: git.kernel.org; GitLab upstream available in config for bleeding-edge hardware support).

- **System Updates:** Runs aptitude full-upgrade for packages.

- **Distribution Upgrades:** Optional major version upgrade using apt full-upgrade (per Debian upgrade guide).

- **Dry-Run Mode:** Preview all changes without modifying the system.

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

  - `ionice` (for lower I/O priority in non-interactive mode).

## Installation
Download:
```bash
wget https://raw.githubusercontent.com/cwadge/apt-up/main/apt-up -O apt-up
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
--no-dist-upgrade    Skip distribution upgrade
--no-flatpak         Skip Flatpak updates
--no-kernel-cleanup  Skip removal of old kernels
--no-cache-clean     Skip cleaning package caches
--no-hooks           Skip running hook scripts
--interactive        Run the script in interactive mode
--update-firmware    Update system firmware
--update-system      Update system packages
--dist-upgrade       Perform distribution upgrade (uses apt instead of aptitude)
--update-flatpak     Perform Flatpak updates
--kernel-cleanup     Remove old kernels and headers
--cache-clean        Clean all apt package caches
--run-hooks          Run scripts in hook directories
--dry-run            Show what would be done without making changes
--dry-run-hooks      Run hook scripts during --dry-run, passing IS_DRY_RUN=true (default)
--no-dry-run-hooks   List hooks during --dry-run without executing them
--install            Create configuration files and hook directories
--help               Display this help message
```
### Configuration

Edit `/etc/apt-up.conf` (after optional `sudo apt-up --install`).

## Dry-Run Mode

Preview all changes before committing to them with `--dry-run`. This mode shows what would happen without modifying your system.

**Important:** Dry-run mode **does** download package lists (via `apt update` / `aptitude update`) to show accurate information about available updates. This is necessary to provide meaningful previews. However, no packages are installed, removed, or upgraded.

### What Gets Checked

- **System Updates:** Downloads package lists, then shows what would be upgraded (using `aptitude -s`)
- **Flatpak Updates:** Shows available Flatpak updates (using `flatpak remote-ls --updates`)
- **Firmware:** Checks if new firmware is available without fetching
- **Kernel Cleanup:** Lists old kernels and headers that would be removed with size estimates
- **Cache Cleaning:** Shows current cache sizes that would be freed
- **Distribution Upgrades:** Downloads package lists, then simulates the upgrade process (using `apt -s`)

### Usage

Basic dry-run:
```bash
sudo apt-up --dry-run
```

Dry-run for specific operations:
```bash
# Preview distribution upgrade
sudo apt-up --dry-run --dist-upgrade

# Check kernel cleanup only
sudo apt-up --dry-run --no-update-system --no-firmware --no-flatpak
```

### Hook Scripts

By default, hook scripts **are executed** during dry-run mode. Before any hooks run, apt-up exports `IS_DRY_RUN=true` into the environment so that hook scripts can detect dry-run mode and skip their own side effects:

```bash
#!/bin/bash
# Example hook with IS_DRY_RUN awareness

if [ "${IS_DRY_RUN:-false}" = "true" ]; then
    echo "[DRY-RUN] Would perform custom action"
    exit 0
fi

# Normal execution
echo "Performing custom action..."
```

If a hook runs during dry-run but contains no reference to `IS_DRY_RUN`, apt-up prints a warning to flag that it may not be dry-run-aware:

```
[WARNING] Hook '50-legacy-hook' does not check IS_DRY_RUN and may make unintended changes
```

To skip hook execution entirely during dry-run (listing hooks instead), use `--no-dry-run-hooks` on the command line or set `DRY_RUN_HOOKS=false` in `/etc/apt-up.conf`:

```bash
# Skip hook execution during --dry-run; only list what would run
sudo apt-up --dry-run --no-dry-run-hooks
```

```
[INFO] Running pre hooks
[DRY-RUN] Would run hook: 00-example
[DRY-RUN] Would run hook: 50-ge-proton
```

### Example Output

```bash
$ sudo apt-up --dry-run

╔════════════════════════════════════════════════════════════════╗
║                         DRY-RUN MODE                           ║
║            No changes will be made to the system               ║
╚════════════════════════════════════════════════════════════════╝

[DRY-RUN] Would check for package updates
[DRY-RUN] Packages that would be upgraded:
Inst firefox-esr [115.8.0esr-1~deb12u1] (115.9.0esr-1~deb12u1)
Inst linux-image-6.1.0-18-amd64 [6.1.76-1] (6.1.82-1)
... and 23 more packages

[INFO] Running pre hooks
[INFO] Running hook: 00-example
[DRY-RUN] Would perform custom action

[DRY-RUN] No Flatpak updates available

[DRY-RUN] Would remove these packages:
[DRY-RUN]   - linux-image-6.1.0-17-amd64
[DRY-RUN]   - linux-headers-6.1.0-17-amd64
[DRY-RUN] Would free approximately: 287.4 MB

[DRY-RUN] New firmware available:
[DRY-RUN]   Current: a8c5f23
[DRY-RUN]   Remote:  d4e9b67
[DRY-RUN] Would update firmware files in /lib/firmware

[DRY-RUN] Would clean apt cache (~1.3 GB)

[INFO] Running post hooks
[INFO] Running hook: 50-ge-proton
[DRY-RUN] Would update GE-Proton: GE-Proton10-29 → GE-Proton10-30
```

## Distribution Upgrades

**apt-up** can perform major distribution version upgrades using the `--dist-upgrade` option. This feature uses `apt full-upgrade` instead of `aptitude full-upgrade`, per the Debian upgrade guide.

### Important Notes

- Distribution upgrades are **disabled by default** for safety
- This feature works with any Debian-based distribution (Debian, Ubuntu, Mint, etc.)
- **Always update `/etc/apt/sources.list` to point to the new release before running**
- The upgrade process can take considerable time and may require manual intervention

### Preparation Steps

Before performing a distribution upgrade:

1. **Backup your system and important data**
2. **Update your sources.list:**
   ```bash
   sudo nano /etc/apt/sources.list
   # Change release codenames (e.g., bookworm → trixie, jammy → noble)
   ```
3. **Review the release notes for your distribution**
4. **Ensure adequate disk space** (check with `df -h`)
5. **Consider running from a terminal multiplexer** (tmux or screen) for stability

### Usage

Interactive mode (recommended):
```bash
sudo apt-up --dist-upgrade
```

Non-interactive mode (for automation):
```bash
sudo apt-up --dist-upgrade --no-interactive
```

Enable in config file:
```bash
# In /etc/apt-up.conf, uncomment:
ENABLE_DIST_UPGRADE=true
```

### What It Does

The distribution upgrade process:
1. Validates sources.list accessibility
2. Provides warnings and pre-upgrade checklist
3. Updates package lists from new repositories
4. Performs minimal upgrade (`apt upgrade`)
5. Performs full upgrade (`apt full-upgrade`)
6. Provides post-upgrade recommendations

### Post-Upgrade

After a successful upgrade:
- Reboot the system when convenient
- Review system logs for errors
- Remove obsolete packages: `sudo apt autoremove`
- Verify critical services are running

## Example Run
Here's what it looks like in action, checking for firmware & system updates, then running a custom hook script:
```
[INFO] Loading configuration from /etc/apt-up.conf
╔════════════════════════════════════════════════════════════════╗
║              Exporting global variables...                     ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Exporting: IGNORE_CC_MISMATCH=1
[INFO] Exporting: GE_PROTON_TARGET=/home/gamer_account/.steam/steam/compatibilitytools.d
╔════════════════════════════════════════════════════════════════╗
║              Running pre-update scripts...                     ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Running pre hooks
╔════════════════════════════════════════════════════════════════╗
║              Checking for updated files...                     ║
╚════════════════════════════════════════════════════════════════╝
Hit http://deb.xanmod.org releases InRelease
Hit https://security.debian.org/debian-security bookworm-security InRelease
Hit https://deb.debian.org/debian bookworm InRelease
Hit https://deb.debian.org/debian bookworm-updates InRelease
Hit https://brave-browser-apt-release.s3.brave.com stable InRelease
Hit https://deb.debian.org/debian bookworm-backports InRelease
Hit https://repo.steampowered.com/steam stable InRelease
Hit https://www.deb-multimedia.org bookworm InRelease
Hit https://www.deb-multimedia.org bookworm-backports InRelease
Hit https://download.opensuse.org/repositories/home:/strycore/Debian_12 ./ InRelease

╔════════════════════════════════════════════════════════════════╗
║              Updating files if necessary...                    ║
╚════════════════════════════════════════════════════════════════╝
No packages will be installed, upgraded, or removed.
0 packages upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 0 B of archives. After unpacking 0 B will be used.

╔════════════════════════════════════════════════════════════════╗
║              Updating flatpak packages...                      ║
╚════════════════════════════════════════════════════════════════╝
Looking for updates…

Nothing to do.
╔════════════════════════════════════════════════════════════════╗
║              Cleaning out the apt cache...                     ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Finished cleaning cache.
╔════════════════════════════════════════════════════════════════╗
║              Purge old kernels & headers...                    ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Current running kernel: 6.15.5-x64v3-xanmod1
[INFO] Latest installed kernel: 6.15.5-x64v3-xanmod1
[INFO] Current kernel is latest, keeping: 6.15.5-x64v3-xanmod1 and previous: 6.15.4-x64v3-xanmod1
[INFO] Keeping headers for kernel 6.15.4-x64v3-xanmod1: linux-headers-6.15.4-x64v3-xanmod1
[INFO] Keeping headers for kernel 6.15.5-x64v3-xanmod1: linux-headers-6.15.5-x64v3-xanmod1
[INFO] No old kernels or headers to remove.
╔════════════════════════════════════════════════════════════════╗
║              Updating kernel firmware...                       ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Checking for firmware updates...
remote: Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
From https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git
 * branch            main       -> FETCH_HEAD
HEAD is now at 2208e9f Merge branch 'intel/fan_control_8086_e20b_8086_1100' into 'main'
[INFO] Firmware already up to date.
╔════════════════════════════════════════════════════════════════╗
║              Syncing buffers out to disk...                    ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Disk sync completed successfully.
╔════════════════════════════════════════════════════════════════╗
║              Running post-update scripts...                    ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Running post hooks
[INFO] Running hook: 50-ge-proton
Checking for GE-Proton updates...
Using overridden Steam directory: /home/gamer_account/.steam/steam/compatibilitytools.d
Latest version: GE-Proton10-8
GE-Proton10-8 already installed at /home/gamer_account/.steam/steam/compatibilitytools.d/GE-Proton10-8
No installation needed.
Cleaning up old GE-Proton versions...
No old versions to clean up.
Done!
╔════════════════════════════════════════════════════════════════╗
║                       Finished.                                ║
╚════════════════════════════════════════════════════════════════╝
```
(Normally output is color-coded, if the terminal supports it.)

## Hooks
If you want to extend apt-up's functionality, you can add custom scripts to:
- `/etc/apt-up.d/pre.d/` (before updates).

- `/etc/apt-up.d/post.d/` (after updates).

- `/etc/apt-up.d/fail.d/` (on failure).

NOTE: Scripts with '`critical`' in the name (e.g. `00-critical-check.sh`) halt execution if they fail.

Hooks run in dry-run mode by default with `IS_DRY_RUN=true` exported — implement it in your hook to skip side effects gracefully. See [Dry-Run Mode > Hook Scripts](#hook-scripts) for full details.

Run `sudo apt-up --install` to create these directories (with a sample hook in `pre.d`).

## License

GNU General Public License v2.0 or later ([LICENSE](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)) - contributions must remain open-source.

## Contributing

You know the drill: fork, open an issue or submit a pull request. 

---

_A tool for maintaining Debian-based systems by Chris Wadge_
