# Homebrew tap for the Canvas connector

Installs [canvas-mcp-for-claude](https://github.com/brockcraft/canvas-mcp-for-claude), a local MCP server that gives Claude access to the Canvas LMS REST API. Python dependencies live in a private virtual environment, so nothing touches your system Python.

## Install

```bash
brew tap brockcraft/canvas-mcp
brew install canvas-mcp
```

## Set up

**1. Create a Canvas access token.** In Canvas: Account → Settings → Approved Integrations → **+ New Access Token**. Set an expiration.

**2. Store it in macOS Keychain** (input is hidden). Use your own Canvas host in place of `canvas.uw.edu`:

```bash
security add-generic-password -s canvas-api -a canvas.uw.edu -w
```

On Linux, save the token to `~/.canvas/token` and run `chmod 600 ~/.canvas/token`.

**3. Register the connector with Claude Desktop.** Find the full path to the installed command:

```bash
echo "$(brew --prefix)/bin/canvas-mcp"
```

Claude Desktop doesn't use your shell `PATH`, so the config needs that full path. Open `~/Library/Application Support/Claude/claude_desktop_config.json` (or Settings → Developer → Edit Config) and add the entry below under `mcpServers`, keeping any servers already there. Apple Silicon is shown; on Intel Macs the path is `/usr/local/bin/canvas-mcp`.

```json
{
  "mcpServers": {
    "canvas": {
      "command": "/opt/homebrew/bin/canvas-mcp"
    }
  }
}
```

If your Canvas host isn't `canvas.uw.edu`, add the host:

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

The token is never put in this file; the connector reads it from Keychain.

**4. Install the skill (recommended).** In Claude, open Settings → Customize → Skills, upload a skill, and choose:

```bash
echo "$(brew --prefix canvas-mcp)/share/canvas-mcp/skill/canvas-api/SKILL.md"
```

**5. Quit Claude (Cmd+Q) and reopen it.**

## Claude Code

```bash
claude mcp add canvas -- "$(brew --prefix)/bin/canvas-mcp"
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
