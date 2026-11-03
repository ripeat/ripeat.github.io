# Account policy: users, groups and admin rights

(This is the file for `ripeat policy`. To limit what ripeat itself may do, see
[Administrator policy](./admin-policy); its `deny_accounts` rule can switch this feature off.)

`ripeat policy` makes sure groups and users exist, and puts users in groups.
Because it changes who can do what on a machine, it is cautious by design.

```yaml
# ripeat.policy.yaml
groups: [docker]            # groups that must exist
users:
  - name: alice
    groups: [docker]        # alice must be in these
    admin: true             # also add alice to sudo / wheel / Administrators
    shell: /bin/bash        # Linux only; default /bin/bash
```

Unknown keys are errors. Names may contain letters, digits, `_` and `-` (up to 32
characters, starting with a letter or `_`); nothing else is accepted, so a name
can never carry a command. Every group a user is put in must exist already or be
listed under `groups:`.

## Commands

```bash
ripeat policy                          # dry run: show the plan, change nothing
ripeat policy --apply                  # make the changes
ripeat policy --apply --allow-admin    # required when any user has admin: true
ripeat policy --file other.policy.yaml
```

The plan lists each thing to create. Memberships that grant admin rights are marked
`GRANTS ADMIN RIGHTS`. Without `--apply` nothing changes; with an admin grant and
without `--allow-admin`, ripeat refuses before changing anything.

Before applying a policy, `ripeat assess --accounts <file>` rates it: creating accounts is MEDIUM, and granting
admin rights is HIGH ([Risk and `assess`](./assess)).

## What it will and will not do

- It only **adds**. A group, user or membership that already exists is left exactly
  as it is; ripeat never changes, removes or demotes it.
- It refuses `root`, `Administrator`, `System`, `nobody`, `guest`, the account running
  ripeat (and `SUDO_USER`), Linux accounts with a uid below 1000, and Windows
  built-in accounts.
- It never sets or stores a password. A new Linux user has a locked password
  until its owner sets one. A new Windows user is created disabled and without a
  password; the owner sets one and enables it.
- Admin rights mean membership of `sudo` (or `wheel` if there is no `sudo`) on
  Linux, and of the Administrators group on Windows. It never edits sudoers, and
  never turns off firewalls, UAC or other protections.
- It asks for admin rights once before applying ([Admin rights](./privileges)).
  Windows needs an elevated terminal.

## Undoing it

Each change is journaled before it is made ([Install, uninstall, rollback](./install-and-rollback)).
`ripeat rollback` removes only what that run **created**, newest first: memberships,
then users, then groups. A user's home folder is kept. A group that still has
members cannot be removed and is reported. If a change fails part-way, the run is
rolled back automatically.

## Platforms

Linux uses `groupadd`, `useradd`, `usermod`, `gpasswd`, `userdel`, `groupdel`.
Windows uses PowerShell's local user and group commands. **macOS is not supported**: on a Mac
`ripeat policy` stops with "not supported on darwin" and changes nothing ([What has been verified](./platforms)).
