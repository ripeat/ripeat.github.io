# Platforms and what has been verified

"Works" below means it has run for real, not only compiled or passed unit tests.

## Linux

| Area | Status |
|---|---|
| `apt`, `dnf`, `pacman`: install, uninstall, rollback | Verified: Ubuntu 24.04 (arm64), Fedora 41 (arm64), Arch (amd64), in containers |
| `shell` (`typescript`), dependency chains | Verified on the same three |
| `github_release` (`ripgrep`), checksum check | Verified on the same three |
| `outdated`, `upgrade` | Verified with a real older `git` on Ubuntu (`apt`) and Fedora (`dnf`). `pacman` upgrades are unit-tested only: Arch keeps no old versions to test with |
| `lock`, install from `ripeat.lock` | Verified on the same three |
| `ripeat mcp` over real standard input and output, including `--allow-run` (the confirmation dialog, a real `github_release` install and its rollback, the refusal of a step that needs administrator rights) | Tried by hand in an Ubuntu 24.04 container (the assistant test notes); no CI job. The unit tests cover the same flow with an in-memory client |
| Recipe packs (`ripeat tap`) | Unit-tested against a fake GitHub, and tried by hand on macOS against a **real private GitHub repository** with a token: add (two recipes, one shell recipe refused, the commit pinned), `plan` and `assess` (the risk reason names the pack), a change pushed to the repository, then `update` (the new commit, an added recipe, a line-by-line diff of a changed one, the pin moving only after the answer). Not tried on Windows or Linux |
| `policy` (users, groups, admin) | Verified on the same three |
| `validate`, `schema` | Verified on the same three |
| `find` and `add` (`apt`, `dnf`, `pacman` lookups; GitHub release drafts that download and inspect the real file) | Verified on the same three |
| Administrator policy (`/etc/ripeat/policy.yaml`): blocking, fallbacks, never blocking removal, refusing a file that is not root-owned | Verified on the same three containers. The "not owned by root" refusal is only exercised by the Ubuntu VM and macOS CI jobs, and counts as verified only while those pass |
| Restart detection, pause and `resume` (via `/var/run/reboot-required`) | Verified on the same three; `needs-restarting -r` not exercised |
| ripeat running as a normal user through `sudo` | Covered by the Ubuntu VM CI job (`scripts/linux-vm-test.sh`, passwordless `sudo`); the container tests run as root, where `sudo` is never used |
| The interactive `sudo` password prompt | Not verified: no CI can type a password |
| `deb_url` (the `vscode` recipe), `snap` (`hello`), `flatpak` (GNOME Calculator) | Covered by the same VM job. Treat as verified only while that job passes; a missing `snap` is reported as skipped. `deb_url` for `chrome` is not exercised (Chrome is preinstalled on the runner) |

## Windows

The administrator policy file (`%ProgramData%\ripeat\policy.yaml`) is read and enforced the same way, but
ripeat does not check its ACL, and nothing tests it on Windows.

Everything compiles and the logic is unit-tested. A CI job also runs
`scripts/windows-test.ps1` on a GitHub Windows runner (the test notes): real
`choco`, `scoop` and `winget` installs, the GitHub download with a lockfile, the PowerShell
user and group commands, and restart detection and `resume`.

Checked by hand on a real Windows 11 ARM64 machine (a VMware virtual machine; steps in
the Windows test notes), 2 November 2026:

| What | Result |
|---|---|
| `scoop install ripeat` from the bucket, `plan`, `assess`, `find` | worked |
| `install` and `rollback` of `ripgrep` (GitHub release, no administrator rights) | worked |
| `install` and `rollback` of `node` through `winget`, from an administrator window | worked; the plan marked it `admin`, MEDIUM risk |
| `policy --apply` and `rollback` (group, disabled user, membership) | worked; rollback removed all three |
| Administrator policy at `%ProgramData%\ripeat\policy.yaml`: a file missing `version` is refused, a denied method falls back to the next one, all methods denied blocks the tool, no file means no limits | worked |
| `ripeat mcp` answers an MCP `initialize` request | worked |
| Confirmation question in 0.5.0 | **failed**: the console was not recognised as a terminal, so `install` refused without `--yes`. Fixed after 0.5.0; the fix was checked by hand on the same machine |
| `ripeat version --check` (0.6.0 on Scoop) | worked: "You have the latest release" |
| `ripeat recipes check` | worked: the `winget` packages and the ripgrep GitHub release were found; `choco` (not installed) and `scoop` (no lookup yet) were reported as not checked, `typescript` (`shell`) was not checked |
| `ripeat profile export` and `import` | worked: file round trip, a tool with no recipe marked, a plain `http://` address refused |
| Restart and `resume` on a real pending restart | not done |
| `ripeat mcp` with Claude Code (plan, assess, save a profile) and with Codex (plan) | worked; see the assistant test notes. Claude Desktop: one conversation (plan, find; no install); it did not use the draft tool because of out-of-date guidance, now fixed |
| `ripeat mcp --allow-run` | not run on Windows |

Treat a Windows feature as verified only while that job passes on the current code. The job
reports a method as skipped when its tool is missing on the runner, and it
runs on a clean server image, not on a real person's machine, so user-account quirks,
antivirus and corporate policy are not covered.

**`product` (the "Add or remove programs" check for installers)** is unit-tested (the name matching, the "is it there" logic, validation) and builds for Windows, but the registry read and reading the product name from an `.msi` have **not been run on a real Windows machine yet**.

**`installer` for `.pkg`** is unit-tested (payload parsing, the allowed-folder and denied-path rules, which paths are owned, signature and receipt parsing, refusing a damaged record), but not run against a real package: no CI job builds one with `pkgbuild` yet ([Vendor installers](./installers)). Never try it on your own Mac.

**`installer` for `.exe`** is unit-tested (registry diff, Windows command-line parsing, uninstall command choice, argument rules, the same-signer rule, refusals) with stand-in installers and a fake registry list; the real registry and a real vendor `.exe` have **not** been run ([Vendor installers](./installers)).

**`installer` for `.msi`** ([Vendor installers](./installers)) was run by hand on a real Windows 11 ARM64 machine with Node.js's official signed MSI (checksum
checked, signer read as OpenJS Foundation, `msiexec` install, `msiexec /x` undo, saved file deleted). No CI job installs an MSI.

**Not yet verified:** the lookups added for `snap`, `flatpak`, `scoop` and the Microsoft Store source of `winget` ([Install by name](./find-and-add)).
They are unit-tested against sample output of each tool, and the Ubuntu VM job (`snap`, `flatpak`) and the Windows job (`scoop`; the Store is
reported, not required) now exercise them for real. Treat each as verified only while those jobs pass, and the Store lookup only after it has been
seen to work on a real Windows machine.

Install by name on Windows: the `choco` lookup, draft, install and rollback ran for real on the Windows CI runner. The `winget`
lookup is exercised by the same job and counts as verified only while that job passes; its table parsing is the part to watch.

## macOS

| Area | Status |
|---|---|
| `brew` (formulas) and `brew_cask` (apps): install, uninstall, rollback, `outdated` | Exercised by `scripts/macos-test.sh` on a GitHub macOS runner (Apple silicon); treat as verified only while that CI job passes. `brew upgrade` is unit-tested only (parsing) |
| `github_release` (`ripgrep`) with a lockfile | Same job |
| `installer` (`.dmg`): download, SHA-256, signature refusal, copy one app, undo, unmount | Unit test (`internal/backend`) builds a real disk image and runs the real mount, copy and undo into a temporary folder; it runs wherever `go test` runs on macOS. Not tried with a real, signed vendor download, and the signature check of a genuinely signed and notarized app is covered only by parsing sample `spctl` output |
| Built-in recipes on macOS | `git`, `node`, `typescript` (via npm), `chrome`, `firefox`, `vscode`, `docker`, `ripgrep`. The CI job checks every Homebrew package named in them exists |
| Users and groups (`ripeat policy`) | **Not supported**: it stops with an error and changes nothing |
| Restart detection | Never reports a restart |
| Intel Macs | Built in releases, never run |
| Signing and notarization | None, by decision: ripeat is a hobby project (release notes for maintainers) |
