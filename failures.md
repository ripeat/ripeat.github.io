# When something fails

If a step of an install, uninstall, upgrade or account change fails, ripeat prints the plain
error as before, and when it recognises the failure it adds a likely cause and something to try:

```
Installing git failed: exit status 100
Why it probably failed: The package manager does not know this package name here.
  What to try: Refresh its package list (for example `sudo apt-get update`), then check the name with `ripeat find <tool>`.
Rolling back this run...
```

It is a fixed list of rules, not AI: the same failure always gets the same words, and a failure no rule
recognises is simply left as the plain error. ripeat never makes up a cause. The error itself
and the exit code are unchanged, so scripts that read them keep working.

## How it decides

When a command fails, ripeat keeps the last few kilobytes of what it printed (it still shows all of it as
it happens). The rules look for known phrases in that text and in the error. Up to three explanations are
shown, most specific first; extra ones start with "It may also be".

## What it recognises

| Cause shown | Typical signs |
|---|---|
| The administrator policy forbids it | `blocked by the administrator policy` ([Administrator policy](./admin-policy)) |
| `sudo` wanted a password and had no terminal | `a password is required`, `no tty present` |
| Homebrew refuses to run as root | `Don't run this as root` |
| Administrator rights are missing (not shown when ripeat is already root or administrator) | `permission denied`, `access is denied`, `operation not permitted` |
| The package manager does not know the name | `Unable to locate package`, `No match for argument`, `target not found`, `No package found`, `No available formula or cask` |
| Another package manager is running | `Could not get lock`, `unable to lock database`, `database is locked` |
| GitHub is limiting requests | `rate limit` (set `GITHUB_TOKEN`) |
| The download's SHA-256 is wrong (nothing was installed) | `checksum mismatch`, `changed since ripeat.lock` |
| The machine does not trust the server's certificate | `x509:`, `certificate verify failed`, `unable to get local issuer certificate` |
| The server could not be reached | `Could not resolve host`, `connection timed out`, `i/o timeout`, `dial tcp` |
| The disk is full | `No space left on device` |
| A company or system policy blocks it | `blocked by group policy`, `AppLocker`, `SmartScreen` |
| macOS Gatekeeper blocked an unsigned program | `cannot be opened because the developer cannot be verified` |
| Dependencies cannot be satisfied | `unmet dependencies`, `held broken packages`, `conflicts with` |
| A program the step needs is missing | `command not found`, `: not found`, `is not recognized as an internal or external command` |
| The file is no longer where the recipe says | `404 Not Found` (not shown when the package manager simply did not know the name) |

The advice differs by system where it matters: on Windows it says to open a terminal as Administrator, and
elsewhere to run `sudo -v` first. Both mention `--scope user` for tools that need no admin rights
([Admin rights](./privileges)).

## Limits

- Only failures during a run are explained. Mistakes found before anything runs (a bad recipe, a blocked
  plan, an unknown tool) already stop with a specific message, and `ripeat validate` finds file mistakes earlier.
- It matches English phrases. A tool printing in another language may go unrecognised, and then the failure is
  left as the plain error.
- The cause is the likeliest one, not proof. The tool's own output, shown above it, is the evidence.
- A failed rollback step is listed as before; it does not get an explanation.
