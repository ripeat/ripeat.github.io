# The lockfile: `ripeat.lock`

Reproduces the same versions on another machine or later.

```yaml
version: 1
tools:
  git:
    linux/arm64:
      method: apt
      version: 1:2.43.0-1ubuntu7.4
  ripgrep:
    linux/arm64:
      method: github_release
      version: 15.2.0
      url: https://github.com/.../ripgrep-15.2.0-aarch64-unknown-linux-musl.tar.gz
      sha256: 800b1e72...
```

Entries are per OS and CPU (`linux/arm64`). Commit the file.

## `ripeat lock [tool ...]`

Writes `./ripeat.lock` from what ripeat has installed on **this** machine (all
managed tools, or the ones named). Entries for other platforms already in the file
are kept, so you can lock on Linux and on Windows into one file.

| Tool | Locked as |
|---|---|
| installed with `apt`, `dnf`, `pacman`, `brew`, `brew_cask` | the package manager's installed version |
| installed from a release or URL | the version, plus the URL and SHA-256 |
| pre-existing, or installed with `shell`, `winget`, `scoop`, `choco`, ... | not locked; ripeat says why |

It fails if nothing could be locked.

## Installing with a lock

`ripeat install` uses `./ripeat.lock` automatically and says "Using ripeat.lock".

- The pinned version is used, unless you typed a version on the command line, which wins.
- A download whose SHA-256 no longer matches the lock is refused.
- A pin made with another method (locked with `apt`, this machine uses `dnf`) is
  ignored, with a note.
- A tool missing from the lock installs unpinned, with a note.
- `--no-lock` ignores the file.

## Limits

- `pacman`, `brew` and `brew_cask` cannot install a chosen version; their pins are recorded
  but not enforced (ripeat warns).
- A package manager that no longer offers the locked version makes the install fail.
