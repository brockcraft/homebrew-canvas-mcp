# CLAUDE.md — homebrew-canvas-mcp

**Status: archived (2026-10-03).** The connector now ships as a Claude Desktop extension (`.mcpb`) from the connector repo, which is easier for non-technical users. This tap is no longer maintained; the notes below are kept for reference.

Homebrew tap for [canvas-mcp-for-claude](https://github.com/brockcraft/canvas-mcp-for-claude), the local MCP connector that lets Claude read and edit Canvas LMS courses. Public repo: https://github.com/brockcraft/homebrew-canvas-mcp (default branch `main`).

## Layout

- `Formula/canvas-mcp.rb` — the formula. Builds a private virtualenv (`Language::Python::Virtualenv`), pins every Python dependency as a `resource`, installs `server/canvas_mcp_server.py` into `libexec`, writes a `bin/canvas-mcp` wrapper, and installs the skill to `pkgshare`.
- `README.md` — user-facing install and setup.

Users install with `brew install brockcraft/canvas-mcp/canvas-mcp`. The full name matters: Homebrew 6+ requires trust for third-party taps, and installing by full name trusts only this one formula.

## How the connector behaves (affects the formula)

- Single module, not a pip package, so there is no `pip_install_and_link`.
- Needs `CANVAS_HOST` (or `host` in `~/.config/canvas-mcp/config.toml`); there is no default. The token is read from macOS Keychain (service `canvas-api`) or `~/.canvas/token`.
- The skill (`skill/canvas-api/SKILL.md`) is uploaded into Claude by the user; the server never reads it at runtime.
- Direct Python dependencies: `mcp>=1.10,<2` and `httpx` (plus `tomli` on Python 3.10, not needed on 3.13).

## Releasing a new connector version

1. In the connector repo: update `CHANGELOG.md`, run the tests, merge to `main`, tag `vX.Y.Z` (annotated) and push the tag.
2. Get the tarball hash: `curl -sL https://github.com/brockcraft/canvas-mcp-for-claude/archive/refs/tags/vX.Y.Z.tar.gz | shasum -a 256`
3. In `Formula/canvas-mcp.rb`, update `url` and `sha256`.
4. If dependencies changed, regenerate the `resource` blocks: `brew update-python-resources brockcraft/canvas-mcp/canvas-mcp`.
5. Test and audit (see below), then commit and push to `main`.

## Testing

Use the native Apple Silicon Homebrew at `/opt/homebrew`. Run from `/tmp`, not from a folder under `~/Documents`: Homebrew's sandbox fails there with `getcwd: Operation not permitted`.

```
cd /tmp
export PATH=/opt/homebrew/bin:$PATH
brew install --build-from-source brockcraft/canvas-mcp/canvas-mcp
brew test canvas-mcp
brew audit --strict --new brockcraft/canvas-mcp/canvas-mcp
```

The tap clone Homebrew uses is `$(brew --repository brockcraft/canvas-mcp)`; it only sees pushed commits, so push (or copy the formula there) before testing.

## Known limits

- Verified only on Apple Silicon macOS. Intel Macs get no precompiled packages from Homebrew and compile everything; Linux is untested. The README says so.
- The machine this was developed on also has an old Intel Homebrew at `/usr/local` that is first on `PATH` unless `~/.zprofile` has `eval "$(/opt/homebrew/bin/brew shellenv)"`. Do not use it for testing.
- Resource names must match PyPI names with hyphens (`pydantic-core`, `typing-extensions`); `brew audit --strict` enforces this.
