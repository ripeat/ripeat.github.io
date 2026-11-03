# Install, uninstall and rollback

## The rules ripeat keeps

1. **Never remove what it did not install.** A tool whose `check` already succeeds
   is recorded as *pre-existing*. Uninstall and rollback only forget it, and install never changes it: if you asked for a
   version that differs from the one already there, ripeat says so and leaves it ([Risk and `assess`](./assess)).
2. **Journal before acting.** Each step is written to `state.json` as `pending`
   before it runs, then `done` or `failed`. That is what lets a failed or crashed
   run be undone.
3. **Honest about what cannot be undone.** Anything ripeat could not undo is listed.
4. **No OS protections are ever disabled.**

## `ripeat install [tool[@version] ...]`

0. **A name with no recipe** (at a keyboard, not with `--dry-run`): ripeat says so and asks `Look for "opera" and draft a recipe? [y/N]`.
   On yes it does what `ripeat add` does ([Install by name](./find-and-add)): looks the name up, lets you pick when it is found in more than one
   place, shows the drafted recipe, and saves it when you agree. Then the install carries on from step 1 with the new recipe, which is kept
   for next time. Answer no, or find nothing, and nothing is installed. `--yes` never answers this question, because a recipe made from
   something found on the internet is for you to read first. With no terminal (a script, an AI assistant) or with `--dry-run`, ripeat does
   not offer: it fails with `no recipe named ...` and the `ripeat add` hint, as before. With several such names it asks once for all of them, then shows
   every draft before one question to save them ([Install by name](./find-and-add)).
1. Resolves the list and its dependencies ([Recipes](./recipes)), applies
   `ripeat.lock` if present ([The lockfile](./lockfile)), and prints the plan:
   tool, method, `user` or `admin`, and why it is there.
2. Rates the plan's risk ([Risk and `assess`](./assess)); a HIGH-risk plan is not run unless you confirm or pass `--accept-risk`.
3. **Asks before it changes anything:** `Install 3 tools as shown above? [y/N]`. Answer `y` to go ahead. `--yes` (or `-y`)
   answers for you. With no terminal to ask in (a script, an AI assistant) and no `--yes`, ripeat refuses and changes
   nothing. A plan with nothing to install asks nothing.
   Then it asks for admin rights once if any step needs them ([Admin rights](./privileges)).
4. Runs the steps in dependency order. Tools already managed are skipped;
   tools already on the machine are recorded and left alone. A tool whose recipe is `manual` ([Recipes](./recipes)) and that is not there is **left for you**: ripeat does
   not install it, carries on with the rest, and ends with a "Left for you to install by hand" list.
5. If a step makes the machine need a restart, the run pauses; see [Restarts and `resume`](./reboots).
6. If a step fails, ripeat says why it probably failed ([When something fails](./failures)) and the whole run is undone, newest step first, so the machine
   returns to how it was. `--no-rollback` turns this off; you can then run
   `ripeat rollback` yourself.

`ripeat plan` (or `--dry-run`) prints the plan and the commands that would run,
and writes nothing.

## `ripeat uninstall tool ...`

- Shows what it will remove (and which tools it will only forget), then asks `Go ahead? [y/N]`; `--yes` answers for you,
  and with no terminal it refuses without it.
- Refuses to remove a tool that another managed tool still requires, unless `--force`.
- Removes in an order where dependents go before what they need.
- A pre-existing tool is only forgotten, not removed.
- Afterwards it names dependencies that were installed only for the removed tool
  and are no longer needed. It does not remove them for you.
- `remove` is an alias.

## `ripeat rollback`

Undoes the **most recent** run that is not already undone; run it again for older
runs. It lists what it will undo and asks `Undo this run as shown above? [y/N]`; `--yes` answers
for you, and with no terminal and no `--yes` it refuses. What "undo" means per kind of step:

| Step | Undone by |
|---|---|
| install | uninstall (never for pre-existing tools) |
| uninstall | reinstall (at the recorded version, or the latest if it was unpinned). A tool that is already back is not reinstalled: only the record is restored, and winget's "already installed, nothing to upgrade" answer counts as done |
| upgrade | **not undone**: the old version is not kept; rollback says so and leaves it |
| policy (users, groups) | removing what that run created ([Users and groups](./policy)) |

Each `rollback` undoes **one** run. If older runs can still be undone, it says so (`An older run can still be undone (...)`),
and running it again undoes the next one: undoing an `uninstall` run reinstalls the tool, so check what it lists before you answer.
`ripeat history` lists the runs.

Rollback is best effort: one failure does not stop the rest, everything that could
not be undone is listed, and the run stays rollbackable so it can be retried.
A run that was interrupted or crashed is undone from its journal.
`--dry-run` lists what would be undone.

## Reading the state

- `ripeat list`: each managed tool, its version (detected, else requested), method,
  and whether it was already there.
- `ripeat history`: the last 20 runs, with status (`in_progress`, `complete`,
  `failed`, `rolled_back`, `needs_reboot`).

## Limits

- Rollback cannot restore data a tool wrote itself (config, volumes, caches).
- Rollback removes what ripeat installed, not packages the system manager pulled in
  as dependencies (for example `apt` installs `git` with `npm`; `apt autoremove`
  cleans those). `pacman` removal is the exception: it also drops helper packages.
- A reinstall during rollback fetches the recorded version, or the latest if the
  install was unpinned.
- Only one ripeat command that writes state runs at a time. A second one fails
  immediately, naming `state.json.lock`; the lock is released automatically if
  ripeat crashes. Dry runs and read-only commands (`list`, `history`, `recipes`,
  `outdated`, `lock`) do not take it.
