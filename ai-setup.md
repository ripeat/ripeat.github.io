# Set up ripeat with your AI assistant

Copy the prompt below and paste it into your AI assistant (Claude Code, Codex, or another that can run commands on your computer). The assistant installs the ripeat skill, then follows it to install ripeat and connect itself. It asks you before changing the machine.

## The prompt

````text
Install the ripeat skill with `npx skills add ripeat/ripeat.github.io --skill ripeat`, then follow it to install ripeat and connect this assistant. If `npx` is not available, do not install Node.js: read the whole of https://ripeat.github.io/skills/ripeat/SKILL.md with a shell command such as curl, not a tool that summarizes it, and follow that. Ask me before making changes.
````

The skills installer needs Node.js. It asks where to install the skill; if Node.js or the installer is unavailable, use [the manual skill instructions](./agent-skill). The setup command supports Claude Code, Claude Desktop and Codex; the skill stops with an explanation for other clients rather than guessing their configuration. An older ripeat may need an update before it can connect the assistant.

## What it can and cannot do

- **It can** install ripeat through Homebrew or Scoop, check the version, and register ripeat's MCP server with your assistant using [`ripeat mcp setup`](./mcp#connecting), which shows the change, keeps a backup of any file it edits and asks first.
- **It cannot** install your tools (the prompt tells the assistant not to, and without `--allow-run` ripeat's server offers no install tool at all), accept a risk for you, or add `--yes`.
- The prompt lives on this site over HTTPS, so what you paste is what is published here. An assistant is an instruction follower, not a lock: the real protections are ripeat's own confirmations and the
  [administrator policy](./admin-policy), not the wording of the prompt.

An assistant that only has a shell can also use [the skill](./agent-skill).

After the restart, ask the assistant something like "Set up git, node and ripgrep on this machine." It looks each one up, builds the plan and its risk rating, and gives you the `ripeat install` command to read and run yourself.
To let it run the install after you confirm each run in its own app, see [ripeat as an MCP server](./mcp#letting-the-assistant-install-ripeat-mcp---allow-run-off-by-default).
