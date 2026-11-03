# Risk: `ripeat assess`

`ripeat assess` says how risky a plan is, before you run it. It is plain rules, not AI: the same plan always
gets the same answer, every reason is written out, and you can disagree with a rule and read it below. It
changes nothing on the machine.

```bash
ripeat assess git node@20 ripgrep      # tools named on the command line
ripeat assess --file ripeat.yaml       # a profile
ripeat assess --accounts ripeat.policy.yaml   # also: users, groups and admin rights ([Users and groups](./policy))
ripeat assess --json ...               # the same report for programs
ripeat assess --fail-on high ...       # exit with an error at or above a level (medium or high), for scripts
```

It resolves the plan exactly as `ripeat plan` would (fallbacks, dependencies, `ripeat.lock`, the administrator
policy, `--scope`), so the report is about what would really run on this machine.

## What a report shows

```
Risk: MEDIUM   1 to install · 2 already managed · 1 left alone

  RISK      TOOL     HOW             WHY
  -         git      already managed by ripeat, skipped
  MEDIUM    ripgrep  github_release  requested
                     - downloads a program from a third party: its SHA-256 is
                       verified, but ripeat cannot vouch for the project itself
  -         docker   already on this machine, left alone

What would run, without administrator rights:
  download the latest BurntSushi/ripgrep release ...

Can it be undone? Yes: `ripeat rollback` removes what this run installs
```

The first line is the whole story: the level, then how many tools would be installed, are already managed, are left alone, or are **for you to install by hand**
(a `manual` recipe whose tool is missing: shown as "by hand", with its note, no risk level, and `"status": "manual"` in `--json`; a tool that needs one is reported as blocked). Long reasons are wrapped under the tool. On a terminal the levels are coloured (green LOW, yellow MEDIUM,
red HIGH and BLOCKED); in a pipe, in a file, for an AI assistant, with `--json`, or when the `NO_COLOR` variable is set,
the text is plain.

If any requested tool is blocked, the report says the full request cannot be installed yet. Other tools are assessed individually and their commands are previews, not a promise that the complete plan can run. Resolve the listed reasons, then assess the profile again; undo information applies only if the available steps are later installed.

- **Risk:** the highest level among the steps that would run: LOW, MEDIUM, HIGH, or BLOCKED.
- **Each tool:** its level (`none` for a tool that is left alone, since there is nothing to rate), install method, why it is in the plan (requested, or a dependency of another tool),
  and the reasons for its level. Tools already installed are listed as left alone and add no risk.
- **Version warnings:** a tool already on the machine is left alone, even when you asked for a different version
  (`node@20` while `24.15.0` is installed). The report says so in the tool's line ("it is 24.15.0, NOT the 20 you asked
  for") and in a note, and `ripeat plan` and `ripeat install` carry the same warning. ripeat does not claim a version is in
  place when it is not. If it cannot detect the installed version, it says nothing either way.
- **The commands:** exactly what would run, split into those that need administrator rights and those that do not.
- **Whether it can be undone** (`ripeat rollback`), and notes such as lockfile pins that were not used.

A recipe that came from a recipe pack ([Recipe packs (taps)](./taps)) is rated like one you wrote yourself, and its reason names the pack ("the recipe comes from the
recipe pack acme, not shipped with ripeat").

## The rules

Each step gets the highest level that applies, and the plan gets the highest of its steps.

| Fact about the step | Level |
|---|---|
| Forbidden by the administrator policy, no recipe, or install method unavailable here | **BLOCKED** (cannot run) |
| A classic-confinement snap (`classic: true`), which has broad system access | **HIGH** |
| A `shell` command in a recipe **you added** (including an approved draft) | **HIGH** |
| A download ripeat cannot verify (`deb_url`, no checksum) in a recipe **you added** | **HIGH** |
| An `installer` that is a `.pkg`, `.msi` or `.exe` (runs the vendor's installer as administrator), or any installer with `unsigned: true` | **HIGH** |
| A policy file that grants administrator rights (`admin: true`) | **HIGH** |
| A `shell` command, or an unverified `deb_url` download, in a recipe **shipped with ripeat** | MEDIUM |
| A third-party program downloaded from GitHub or a URL, SHA-256 verified | MEDIUM |
| An `installer` that is a `.dmg` (copies an app, runs nothing; SHA-256 and signature verified) | MEDIUM |
| A package that needs administrator rights (for example `apt`, `dnf`), in any recipe | MEDIUM |
| Creating users or groups without admin rights | MEDIUM |
| A step that cannot be undone (a `shell` recipe with no `uninstall`) | noted; part of the shell rating |
| A package for your user only, from the system's package manager (`brew`, `scoop`, ...) | LOW |

Why it is shaped like this:

- **Added recipes are judged more strictly** for the things where the recipe itself supplies what runs: a shell
  command, or a download. Recipes shipped with ripeat were written and tested by the project; one you added (or
  an AI drafted and you approved) has not been.
- **A package-manager recipe is not escalated** just because you added it: it only supplies a package name, and
  ripeat builds the command, so `apt install <name>` stays MEDIUM whoever wrote the recipe.
- **A verified download is MEDIUM, not LOW:** the SHA-256 shows the file is the one that was published, not that
  the project behind it is trustworthy.
- **Admin rights make a package MEDIUM,** because the command runs as root.

## What ripeat does with the level

- **`ripeat install`** prints a short "Risk:" summary with the reasons, then:
  - LOW and MEDIUM go to the normal question, "Install N tools as shown above?" ([Install, uninstall, rollback](./install-and-rollback)).
  - **HIGH** runs only if you say so. In a terminal ripeat shows the reasons and asks `[y/N]`. Without a
    terminal (a script, or an AI client) it refuses unless you pass `--accept-risk`. Nothing is changed when it
    refuses. `--yes` answers the normal question but never this one: only `--accept-risk` (or a yes to the HIGH-risk
    question) lets a HIGH plan run, and approving it counts as the install confirmation too. `ripeat resume` does the same, except that a plan you already accepted before a restart is not asked about again.
  - `ripeat plan` and `--dry-run` show the risk but are never blocked.
- **`uninstall`, `rollback` and `upgrade`** are not gated: removing is never blocked, and `upgrade` only touches
  tools ripeat already installed.
- The administrator policy ([Administrator policy](./admin-policy)) is separate and stricter: it forbids outright,
  and nothing, `--accept-risk` included, overrides it.

## Limits

- It judges what a step *does*, not whether a project or a package name is trustworthy. A MEDIUM download is
  verified, not vouched for.
- It does not ask the package manager to simulate the install, so it cannot yet tell that a package would pull
  in or remove other packages.
- `upgrade` is not rated by `assess`: upgrades cannot be rolled back (see [Versions and upgrades](./upgrades)).
- The levels are judgment calls. If one looks wrong to you, the rule and its reason are in the report.
