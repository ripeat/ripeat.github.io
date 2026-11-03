# Restarts: pausing and `resume`

Some installs need the machine to restart before the next step can work. ripeat
notices, stops cleanly, and lets you carry on afterwards. It does not restart the
machine and does not set anything up to run by itself after a restart.

## How it notices

After each install step ripeat asks the OS whether a restart is now pending.
Only a restart that **appeared during the run** counts; one that was already
waiting before ripeat started is not ripeat's doing and does not stop anything.

| OS | A restart is pending when |
|---|---|
| Linux | `/var/run/reboot-required` exists (Debian, Ubuntu), or `needs-restarting -r` reports it (Fedora family, when that tool is installed) |
| Windows | the Component Based Servicing `RebootPending` key, the Windows Update `RebootRequired` key, or `PendingFileRenameOperations` is set |
| macOS | never reported (not supported yet) |

## What happens during an install

- **A restart appears and more work remains:** the run stops after that step,
  its status becomes `needs_reboot` (see `ripeat history`), and ripeat tells you to
  restart and run `ripeat resume`. Nothing is undone, and the steps done so far
  stay recorded.
- **A restart appears after the last step:** the run completes and ripeat says a
  restart is needed to finish.
- **`upgrade`:** just says so at the end; it never pauses.

## `ripeat resume [--force] [--dry-run]`

Carries on the paused install with the same tools, versions and `--scope`
(and `ripeat.lock`, if present). Steps that already happened are skipped, because
they are managed by now.

- If a restart is still pending, resume refuses and keeps the pause. `--force`
  carries on anyway.
- If you already said yes to the plan's HIGH risk ([Risk and `assess`](./assess)) before the pause, resume does not ask again.
- With nothing paused it prints "Nothing to resume."
- If the resumed install fails it is rolled back like any install, and the pause
  stays recorded so you can retry.
- `--dry-run` shows what it would install.

## Rolling back

A paused run can be rolled back like any other (`ripeat rollback`), and after a
resume the two runs are undone newest first ([Install, uninstall, rollback](./install-and-rollback)).

## Limits

- Resume is manual. Boxstarter-style automatic continuation after the restart would
  need a scheduled task or service, which is a standing change to your machine, so
  ripeat does not create one.
- The Windows detection has never run on a real machine ([What has been verified](./platforms)).
  `PendingFileRenameOperations` is also set on some machines for unrelated reasons;
  because only newly appeared restarts count, this does not stop a run.
