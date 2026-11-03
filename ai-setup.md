# Set up ripeat with your AI assistant

Copy the prompt below and paste it into your AI assistant (Claude Code, Codex, or another that can run commands on your computer). The assistant reads the setup instructions on this page, installs ripeat, and connects itself to it. It asks you before making changes.

## The prompt

````text
Read https://ripeat.github.io/ai-setup.md and follow the agent setup instructions to install ripeat and connect it to this assistant. Ask me before making changes.
````

## Agent setup instructions

ripeat sets up a machine from a list and can undo its changes. Follow these steps when the person asks you to install ripeat and connect it to their assistant:

1. Before each command, say what it does and wait for the person to agree. Never add `--yes`, `--accept-risk` or `--allow-admin`, and never pipe a download into a shell.
2. Check whether ripeat is already installed. If it is missing, use Homebrew on macOS or Linux (`brew install ripeat/tap/ripeat`), or Scoop on Windows (`scoop install git`, `scoop bucket add ripeat https://github.com/ripeat/scoop-bucket`, then `scoop install ripeat`). If the appropriate package manager is missing, stop and tell the person; do not install it for them.
3. Check the installation with `ripeat version`.
4. Connect the assistant using `ripeat mcp setup --client claude-code`, `claude-desktop`, or `codex`, as appropriate. Show what will change and let the setup command ask. Do not use `--allow-run`. If the assistant is not supported, explain that and stop rather than guessing its configuration.
5. Stop and tell the person to restart the assistant or app to pick up the connection. Do not install their other tools unless they make a separate request.

## What it can and cannot do

- **It can** install ripeat through Homebrew or Scoop, check the version, and register ripeat's MCP server with your assistant using [`ripeat mcp setup`](./mcp#connecting), which shows the change, keeps a backup of any file it edits and asks first.
- **It cannot** install your tools (the prompt tells the assistant not to, and without `--allow-run` ripeat's server offers no install tool at all), accept a risk for you, or add `--yes`.
- The prompt lives on this site over HTTPS, so what you paste is what is published here. An assistant is an instruction follower, not a lock: the real protections are ripeat's own confirmations and the
  [administrator policy](./admin-policy), not the wording of the prompt.

An assistant that only has a shell can also use [the skill](./agent-skill).

After the restart, ask the assistant something like "Set up git, node and ripgrep on this machine." It looks each one up, builds the plan and its risk rating, and gives you the `ripeat install` command to read and run yourself.
To let it run the install after you confirm each run in its own app, see [ripeat as an MCP server](./mcp#letting-the-assistant-install-ripeat-mcp---allow-run-off-by-default).
