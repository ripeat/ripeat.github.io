# The administrator policy

An administrator can limit what ripeat is allowed to do on a machine. The rules live in
one file that ordinary users cannot change, and ripeat applies them to every install and
upgrade. This is separate from `ripeat policy`, which creates users and groups
([Users and groups](./policy)); this page is about restricting ripeat itself.

## Where the file is

| OS | Path |
|---|---|
| Linux | `/etc/ripeat/policy.yaml` |
| macOS | `/Library/Application Support/ripeat/policy.yaml` |
| Windows | `%ProgramData%\ripeat\policy.yaml` |

No file means no restrictions. The path is fixed: `$RIPEAT_HOME`, flags and environment
variables cannot move it, so a user cannot point ripeat at a different rulebook.

## The rules

```yaml
version: 1                       # required, must be 1
allow_methods: [apt, brew]       # if set, ONLY these install methods
deny_methods: [shell, deb_url]   # these install methods, never
allow_domains: [github.com, "*.example.com"]   # if set, downloads only from these hosts
require_checksums: true          # refuse downloads that are not SHA-256 verified
user_scope_only: true            # refuse anything that needs admin rights
allow_tools: [git, node]         # if set, ONLY these tools can be installed
deny_tools: [docker]             # these tools, never
deny_accounts: true              # no user, group or admin-right changes (ripeat policy)
deny_mcp_run: true               # an AI assistant may never run installs through ripeat mcp --allow-run
deny_taps: true                  # no recipe packs (ripeat tap)
```

Every key is optional except `version`. Unknown keys and unknown install method names
are errors, so a typo cannot silently weaken a rule.

| Rule | What it blocks |
|---|---|
| `allow_methods`, `deny_methods` | install options by method ([Recipes](./recipes)). Deny wins over allow. The `manual` method is exempt: it runs nothing and changes nothing |
| `allow_domains` | `url` and `deb_url` downloads whose host is not listed. `*.example.com` matches subdomains only, not `example.com` itself or a look-alike such as `evilexample.com`. `github_release` counts as `github.com`. Package managers are not affected. Hosts a download is redirected to are not checked |
| `require_checksums` | `deb_url`, whose download cannot be checked. (`url` already requires a SHA-256 and `github_release` refuses to install without one) |
| `user_scope_only` | any option that needs admin rights ([Admin rights](./privileges)) |
| `allow_tools`, `deny_tools` | tools by recipe name, including tools pulled in as dependencies |
| `deny_accounts` | `ripeat policy`, even as a dry run |
| `deny_mcp_run` | the run tools of `ripeat mcp --allow-run` ([ripeat as an MCP server](./mcp)): the server starts without them, and they refuse if called |
| `deny_taps` | `ripeat tap add` and `update`; recipes from taps already added are not loaded ([Recipe packs (taps)](./taps)) |

To stop arbitrary commands, put `shell` in `deny_methods`.

## How it is applied

- **Every plan.** Rules filter a recipe's install options. If a recipe lists `apt` then `scoop`
  and the policy denies `apt`, ripeat uses `scoop`. If every option is blocked, the plan stops
  before anything changes and says what blocked each one:
  `git is blocked by the administrator policy: apt: method "apt" is denied (see `ripeat rules`)`.
- **Installs, upgrades and resume.** An upgrade whose tool is now blocked is skipped with a note.
- **Removing is never blocked.** `uninstall` and `rollback` always work, so tightening the
  rules can never trap anyone with software they cannot remove.
- **It fails closed.** If the file exists but cannot be used (see below), ripeat refuses to
  install, upgrade or change accounts, and changes nothing. It never ignores a broken policy.

## Who is allowed to write it

On Linux and macOS ripeat only uses the file if it is **owned by root and not writable by
group or others**. Otherwise a user who could edit it could lift their own limits, so ripeat
refuses with a message such as `must be owned by root` or `must not be writable by group or
others`. Create it as an administrator:

```bash
sudo mkdir -p /etc/ripeat
sudo tee /etc/ripeat/policy.yaml >/dev/null <<'EOF2'
version: 1
deny_methods: [shell]
EOF2
sudo chown root:root /etc/ripeat/policy.yaml && sudo chmod 644 /etc/ripeat/policy.yaml
```

On Windows ripeat does **not** check the file's permissions yet. Keep `%ProgramData%\ripeat`
writable by administrators only.

## Seeing and checking it

- `ripeat rules` prints the policy in force in plain words, or says there is none. If the file
  cannot be used it says why.
- `ripeat validate` checks the system policy (including who owns it) along with your other files,
  and `ripeat validate <file>` checks any policy file before you install it.
- `ripeat schema admin-policy` prints a JSON schema for editors.

## Risk and the policy

`ripeat assess` ([Risk and `assess`](./assess)) reports a tool the policy forbids as BLOCKED, and `--accept-risk` never overrides
the policy.

## What it covers besides installs

`ripeat find` and `ripeat add` ([Install by name](./find-and-add)) hide or refuse candidates the
policy forbids, so a user is never offered, or able to save, a recipe that could not be installed.

## Limits

- It restricts ripeat, not the machine: a user can still run `apt` themselves.
- A `shell` recipe can do anything, so `deny_methods: [shell]` is the only reliable guard against it.
- Domain checks look at the address a recipe names, not where the download is redirected to.
- Root can change the file; that is the point.
