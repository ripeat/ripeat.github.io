# Privileges

ripeat asks for admin rights only for the steps that need them, and tells you
before it changes anything.

## Scope per step

Each install option has a `scope` ([Recipes](./recipes)): `user` needs no admin
rights, `system` (the default) does. The plan shows it as `user` or `admin`.

## One up-front request

Before the first change, `install` and `upgrade` list the steps that need admin
rights ("Administrator rights are needed for: ...") and, on Linux and macOS when
you are not root, run `sudo -v` once so the password prompt comes first.
If that fails nothing has been changed. Steps already satisfied (managed or
already on the machine) are not counted. `policy --apply` does the same.

`uninstall` and `rollback` do not ask up front; `sudo` may prompt in the middle.
`sudo` remembers the password for its own timeout (usually 15 minutes), so a long
run can still prompt again.

## macOS

Homebrew refuses to run as root, so `brew` and `brew_cask` steps never need admin rights
and ripeat never runs them through `sudo`. A cask whose installer needs a password
(some `.pkg` apps) asks for it itself, in the middle of the run.

## Windows

A process cannot gain admin rights part-way on Windows, so ripeat cannot elevate
itself. `shell` steps with `scope: system` are refused up front when the terminal
is not elevated (nothing changes). `winget` installers ask for UAC themselves.
`choco` needs an elevated terminal, and ripeat does not check that up front yet.
`policy --apply` requires an elevated terminal.

## `--scope user|system`

On `install` and `plan`, limits which options may be chosen:

| Flag | Chooses only options that |
|---|---|
| `--scope user` | need no admin rights (`scoop`, `github_release`, `winget` with `scope: user`, ...) |
| `--scope system` | need admin rights |
| (none) | any; the first usable option wins |

If a tool has no option in that scope, the plan fails before anything changes and
names the methods the recipe has. For example, `ripeat plan git --scope user` on
Linux fails because `apt`, `dnf` and `pacman` all need admin rights.
