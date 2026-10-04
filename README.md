# Homebrew tap for the Canvas connector

Tested on Apple Silicon Macs. Intel Macs (no precompiled packages from Homebrew) and Linux are untested.

Installs [canvas-mcp-for-claude](https://github.com/brockcraft/canvas-mcp-for-claude), a local MCP server that gives Claude access to the Canvas LMS REST API. Python dependencies live in a private virtual environment, so nothing touches your system Python.

## Install

```bash
brew install brockcraft/canvas-mcp/canvas-mcp
```

That one command adds the tap and installs the formula. Homebrew 6 and later won't load formulae from third-party taps until you trust them; installing by the full name trusts only this one formula, not the whole tap. If you tap it first (`brew tap brockcraft/canvas-mcp`) and then see "Refusing to load formula from untrusted tap", run `brew trust --formula brockcraft/canvas-mcp/canvas-mcp`.

## Set up

**1. Find your Canvas host.** It's the address you sign in to Canvas at, without `https://` (for example `canvas.example.edu`). **There is no default; the connector won't work until you set it.**

**2. Create a Canvas access token.** In Canvas: Account → Settings → Approved Integrations → **+ New Access Token**. Set an expiration.

**3. Store the token in macOS Keychain** (input is hidden). Use your own host in place of `canvas.example.edu`:

```bash
security add-generic-password -s canvas-api -a canvas.example.edu -w
```

**4. Register the connector with Claude Desktop.** Find the full path to the installed command:

```bash
echo "$(brew --prefix)/bin/canvas-mcp"
```

Claude Desktop doesn't use your shell `PATH`, so the config needs that full path. Open `~/Library/Application Support/Claude/claude_desktop_config.json` (or Settings → Developer → Edit Config) and add the entry below under `mcpServers`, keeping any servers already there. Replace `canvas.example.edu` with your host. Apple Silicon is shown; on Intel Macs the path is `/usr/local/bin/canvas-mcp`.

```json
{
  "mcpServers": {
    "canvas": {
      "command": "/opt/homebrew/bin/canvas-mcp",
      "env": { "CANVAS_HOST": "canvas.example.edu" }
    }
  }
}
```

If you leave out the host, every Canvas tool call fails with a message telling you to set it. You can also set `host = "canvas.example.edu"` in `~/.config/canvas-mcp/config.toml` instead of the `env` entry.

The token is never put in this file; the connector reads it from Keychain.

**5. Install the skill (recommended).** In Claude, open Settings → Customize → Skills, upload a skill, and choose:

```bash
echo "$(brew --prefix canvas-mcp)/share/canvas-mcp/skill/canvas-api/SKILL.md"
```

**6. Quit Claude (Cmd+Q) and reopen it.**

## Claude Code

```bash
claude mcp add canvas -e CANVAS_HOST=canvas.example.edu -- "$(brew --prefix)/bin/canvas-mcp"
```

## Upgrade and uninstall

```bash
brew update && brew upgrade canvas-mcp
```

```bash
brew uninstall canvas-mcp
brew untap brockcraft/canvas-mcp
security delete-generic-password -s canvas-api
```

Also remove the `canvas` entry from your Claude config. See the [main repository](https://github.com/brockcraft/canvas-mcp-for-claude) for the course guard, configuration and audit log.
