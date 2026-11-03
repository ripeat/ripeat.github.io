# Versions, `outdated` and `upgrade`

## Detected versions

After installing a tool, ripeat runs its recipe's `version` command (or `check`) and
stores the first dotted number it prints (`2.43.0`). Pre-existing tools are
detected the same way. `ripeat list` shows it.

## Is there a newer ripeat? `ripeat version --check`

```bash
ripeat version            # just the version
ripeat version --check    # also asks GitHub whether a newer release is out
```

It reads the latest release of the public releases repository and compares numbers (`0.10.0` is newer than `0.9.0`). If there is a newer
one it prints the upgrade command that fits how ripeat was installed: `brew update && brew upgrade ripeat/tap/ripeat` for
Homebrew, `scoop update ripeat` for Scoop, or the releases page for a download. It never downloads or replaces anything itself. A build made
from source (`0.1.0-dev`) has nothing to compare. Built-in recipes are part of the release, so upgrading ripeat is also how they are updated.

## `ripeat outdated`

For each tool ripeat installed, asks its package manager whether a newer version
exists, and prints `installed -> newer` or `(up to date)`.

- Works for `apt`, `dnf`, `pacman`, `brew` and `brew_cask`. Other methods print "cannot check yet".
- It reads the package manager's **current** index and does not refresh it. Refresh
  first (for example `sudo apt-get update`) to see the newest versions.
- Pre-existing tools are listed as not managed by ripeat; they are never upgraded.
- Versions shown are the package manager's (`1:2.43.0-1ubuntu7.4`), which is also
  what `ripeat.lock` records.

## `ripeat upgrade [tool ...]`

Upgrades the outdated tools (all of them, or the ones named).

- Shows the list, then asks `Upgrade N tools as shown above? [y/N]` (`--yes` answers for you; with no terminal it refuses without it).
- Asks for admin rights once up front ([Admin rights](./privileges)).
- Recorded as one run in the journal, one step per tool.
- `--dry-run` shows the commands.
- **Upgrades cannot be rolled back**: ripeat does not keep the old version.
  `ripeat rollback` on an upgrade run says so and leaves the tools upgraded;
  the next `rollback` then reaches the run underneath.
- If one tool fails, earlier upgrades in that run stay in place.
- `pacman` upgrades a single package without a full system upgrade, which Arch
  discourages; prefer `pacman -Syu` yourself on Arch.
