# Set up ripeat with your AI assistant

Paste the text below into your AI assistant (Claude Code, Codex, or another that can run commands on your computer). It installs ripeat and connects the assistant to it. Nothing
happens without your yes at each step, and the assistant gets no power to install software for you.

## The prompt

````text
Install ripeat on this computer and connect it to you, step by step. ripeat is a tool that sets up a machine from a list and can undo it; its website is https://ripeat.github.io.

Rules you must follow:
- Before each command, say what it does and wait for me to agree. Never add --yes, --accept-risk or --allow-admin, and never pipe a download into a shell.
- Install ripeat only with Homebrew (macOS or Linux: brew install ripeat/tap/ripeat) or with Scoop (Windows: scoop install git; scoop bucket add ripeat https://github.com/ripeat/scoop-bucket; scoop install ripeat). If neither is installed, stop and tell me; do not install them for me.
- Check it with: ripeat version
- Then connect yourself with ripeat's own setup command, for the assistant you are: ripeat mcp setup --client claude-code, or claude-desktop, or codex. Show me what it will change and let it ask me. Do not use --allow-run.
- Then stop. Do not use ripeat to install any of my tools. That is a separate request, and I will ask for it. Tell me to restart you (or the app) so the connection is picked up.
````

## What it can and cannot do

- **It can** install ripeat through Homebrew or Scoop, check the version, and register ripeat's MCP server with your assistant using [`ripeat mcp setup`](./mcp#connecting), which shows the change, keeps a backup of any file it edits and asks first.
- **It cannot** install your tools (the prompt tells the assistant not to, and without `--allow-run` ripeat's server offers no install tool at all), accept a risk for you, or add `--yes`.
- The prompt lives on this site over HTTPS, so what you paste is what is published here. An assistant is an instruction follower, not a lock: the real protections are ripeat's own confirmations and the
  [administrator policy](./admin-policy), not the wording of the prompt.

An assistant that only has a shell can also use [the skill](./agent-skill).

After the restart, ask the assistant something like "Set up git, node and ripgrep on this machine." It looks each one up, builds the plan and its risk rating, and gives you the `ripeat install` command to read and run yourself.
To let it run the install after you confirm each run in its own app, see [ripeat as an MCP server](./mcp#letting-the-assistant-install-ripeat-mcp---allow-run-off-by-default).
