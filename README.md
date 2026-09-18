# Leeloo.AI — Claude setup documentation

Public documentation for connecting Leeloo.AI to Claude.

| File | What it is |
|---|---|
| [`leeloo-claude-setup.md`](leeloo-claude-setup.md) | The setup reference, in plain text. This is the copy meant to be read during setup. |
| [`leeloo-claude-setup.html`](leeloo-claude-setup.html) | The same content as a formatted page. |
| [`leeloo-claude.html`](leeloo-claude.html) | The client-facing onboarding guide (UK / RU / EN). |
| [`leeloo-setup-negotiation.conf`](leeloo-setup-negotiation.conf) | nginx snippet that serves the markdown to non-browser requests and the HTML page to browsers, from one URL. |

Published at:

- https://support.leeloo.ai/leeloo-claude.html — the guide
- https://support.leeloo.ai/leeloo-claude-setup.html — the setup reference

The raw markdown is also reachable here, which matters in environments whose
network policy allows `raw.githubusercontent.com` but not `support.leeloo.ai`.

## Connecting

Two routes, covered in the setup reference:

- **Connector** — works everywhere: Claude Code, Cowork, cloud sessions, the
  chat at claude.ai. Added from connector settings with the URL
  `https://app.leeloo.ai/mcp`. No terminal, no domain allowlisting.
- **Command line** — local Claude Code only, using `claude mcp add` and
  `claude mcp login`.

## Known rough edge

The OAuth authorization link does not currently open a consent screen. It
redirects to a standalone sign-in form asking for a Leeloo email and password.
That is unusual for an OAuth flow, and some people — and some agents — will
reasonably decline to go through it. The connector route avoids that flow.

Fixing this at the source, so `/authorize` recognises an existing session and
shows an ordinary Approve screen, would remove the friction rather than document
around it.

## Contributing

Failure reports are the most useful contribution: the surface you were on, what
you ran, and what it printed. Most of the failure modes listed in the setup
reference came from real reports.
