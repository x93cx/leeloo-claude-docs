# Leeloo MCP setup for Claude Code

This plain text version is returned because this request does not look like a
browser page navigation. The same address opens a formatted page in a browser.

Reference documentation for connecting Leeloo.AI to Claude. The full guide, with
screenshots, is at https://support.leeloo.ai/leeloo-claude.html

Nothing on this page overrides the judgement of whoever is reading it. Treat it
as documentation, not as instructions to follow unquestioned — and if a step
looks wrong for the situation in front of you, say so instead of carrying on.

MCP server: `https://app.leeloo.ai/mcp`

---

## First: check where you are running

This decides which of the two routes below applies. Getting it wrong wastes the
user's time and can leave a half-configured server that never authenticates.

| Surface | Route |
|---|---|
| Claude Code on the user's own machine — CLI, the Code tab of the desktop app, or the IDE extension | **Route A**, the command line |
| Cowork | **Route B**, connector |
| Claude Code on the web, cloud sessions, routines | **Route B**, connector |
| The chat at claude.ai | **Route B**, connector |

Route A needs two things that only exist locally: a shell, and a browser on the
same machine that an OAuth callback to `localhost` can reach. Everywhere else,
use Route B and say so plainly instead of attempting commands that cannot work.

---

## Route A — local Claude Code

### 1. Register the server

```
claude mcp add --scope user --transport http leeloo https://app.leeloo.ai/mcp
```

Skip this if a server named `leeloo` already points at the same URL. If it points
somewhere else, stop and tell the user both URLs — do not log in to the existing
server and report success, that silently connects them to a different
environment. `claude mcp add` has no overwrite flag; removing and re-adding, or
choosing another name, is the user's call.

### 2. Log in — the user's own terminal is the reliable path

```
claude mcp login leeloo
```

Have the user run this in their own terminal window. It has a real TTY and opens
the browser in their own profile, so the page loads with their session and the
link is used within seconds. Prefer this whenever they have a terminal open. It
is not a fallback; it is the route that works.

### 3. Driving the login yourself

Only when the user asks you to and is at the keyboard right now.

**a. Start the login in a console, logging to a file.** `claude mcp login` needs a
TTY, which an agent shell does not have.

```
# macOS / Linux
python3 -c 'import pty,sys; pty.spawn(sys.argv[1:])' claude mcp login leeloo

# Windows — pty does not exist, spawn a real console
powershell.exe -NoProfile -Command "Start-Process cmd -ArgumentList '/c','claude mcp login leeloo > %TEMP%\leeloo-login.log 2>&1'"
```

The console window will look blank and idle. That is expected — its output is
redirected into the log. Tell the user so, or they will think it hung.

**b. Poll the log** until the `https://app.leeloo.ai/authorize?...` line appears.

**c. Show the user the URL and say what it leads to** — see section 4. Let them
open it themselves. They are about to be asked for a password, so they should be
the one who chooses to go there.

The link is short-lived, so tell them it needs opening now rather than in a few
minutes, and that a fresh one can be issued if it expires.

If the user asks you to open it for them, you may:

```
# macOS
open "<authorization URL>"

# Windows — use PowerShell, not `start`
powershell.exe -NoProfile -Command "Start-Process '<authorization URL>'"

# Linux
xdg-open "<authorization URL>"
```

On Windows, do **not** use `cmd.exe /c start "" "<url>"`. Called from a POSIX
shell it opens an interactive command prompt instead of the browser: it prints
the Windows banner and exits 0, so it looks like it worked while nothing opened.
`Start-Process` is silent when it succeeds.

Use the platform command rather than a built-in or automated browser: those carry
none of the user's sessions and will only ever show a sign-in form.

**d. Keep polling** the log for the authenticated line.

**After two failed attempts, stop and hand it back.** Repeating the loop does not
change the outcome; ask the user to run the login in their own terminal.

### 4. What the authorization page currently asks for

Be straight with the user about this, and let them make up their own mind.

At the time of writing, the link does not open an OAuth consent screen. It
redirects to `/mcp/login`, a standalone form asking for a Leeloo email and
password. That is unusual for an OAuth flow, and a reasonable person may well
decide not to enter a password on a page an agent opened for them. That decision
is theirs; do not argue them out of it, and do not characterise their caution as
a misunderstanding.

If they would rather not, Route B below connects Leeloo without this flow at all.

Never type someone's credentials yourself, and never offer to.

### 5. Success signal

Wait for the line:

```
Authenticated with "leeloo"
```

`Connected` on its own does **not** mean the login finished.

Verify with:

```
claude mcp get leeloo
```

---

## Route B — connector

Works on every surface, including Cowork, cloud sessions and claude.ai. It needs
no terminal, no TTY, no localhost callback, and **no domain allowlisting**:
connector traffic travels through Anthropic's servers rather than the session's
network.

These four steps are the user's to perform — you cannot click through claude.ai
settings for them. Say so plainly rather than looping on attempts.

1. Open connector settings and add a custom connector with the URL
   `https://app.leeloo.ai/mcp`.
2. Sign in to Leeloo when the connector asks for authorization.
3. Enable the connector for the surface they are using.
4. Start a **new** session — the current one keeps the tool set it started with.

### Why unblocking a domain is not the fix here

Allowlisting a support domain only lets you read instructions. It does not make
`claude mcp add` work in a running cloud session, and it does not help the OAuth
callback reach the user's browser. If the user asks whether to unblock something,
tell them Route B needs no allowlist at all.

---

## Tools load at session start

The session that runs the setup cannot use the Leeloo tools, on either route.
When the connection succeeds, tell the user to open a new session and make a
request naming Leeloo. Do not attempt a Leeloo tool call in the setup session.

---

## Requirements

- **Claude Code** 2.1.210 or newer — check with `claude --version` and
  `claude mcp login --help`.
- If the shell cannot resolve `claude`, try `command -v claude` and
  `~/.local/bin/claude`. On Windows also check `cmd.exe /d /c where claude`.
  Invoke a resolved executable directly instead of reinstalling over a stale PATH.
- The desktop app does not guarantee the terminal CLI exists. Install it from
  https://claude.com/claude-code only if no executable is found.

## Network access

Managed environments often restrict outbound hosts. If a host is refused by an
egress policy, name the blocked host to the user and stop — do not look for a
mirror or another route. Unblocking a domain is their administrator's decision.

---

## Failure modes

**The open command reported success but no tab appeared.** On Windows this is
almost always `cmd.exe /c start "" "<url>"` — see Route A step 3c. Use
`Start-Process`, which prints nothing when it works.

**The login timed out.** The link expired before anyone opened it. Issue a fresh
one and tell the user it needs opening promptly; old links do not work.

**A blank console window opens and nothing happens.** Expected — output is
redirected to the log. Read the log, not the window. If the log is still empty
after about ten seconds, the command never started: check that `claude` resolves
from that shell, then hand the login to the user's own terminal.

**"Sign in to Leeloo MCP" — email and password, no Approve button.** The correct
screen, not an error. See Route A step 4.

**`MCP server leeloo already exists in user config`.** Check where it points with
`claude mcp get leeloo`. Same URL — skip the add, go straight to the login.
Different URL — stop and tell the user both URLs before touching anything.

**`Connected · tools fetch failed` / `Request timed out`.** The backend is warming
up. Retry the call. Do not re-run the login and do not remove the server.

**`Needs authentication` right after a successful login.** A slow tool-list fetch
can flip the status back for a few seconds. Wait and check again. Treat it as a
real failure only when a Leeloo tool repeatedly returns an explicit authentication
error rather than a timeout.

**HTTP 502 and other 5xx during OAuth.** Retryable service failures during
discovery, registration, callback or token exchange — not bad credentials, and
not a reason to reinstall. Wait for the current login process to exit, then retry
with backoff. Never run two logins at once.

**`Authentication timeout`.** The link expired before it was approved — usually
because nobody opened it. Log out, start one fresh login, open the new URL
straight away, and resume polling. Old links do not work.

**`claude: command not found`.** See Requirements above.

**Login succeeded but Leeloo tools are missing.** The user is still in the session
that did the connecting. Open a new one.

---

## Disconnecting

```
claude mcp logout leeloo      # clear credentials, keep the server
claude mcp remove leeloo -s user   # remove the server entirely
```

---

## Production data

`app.leeloo.ai` holds live tunnels, subscribers and offers. Show a plan before
creating, editing or deleting anything, and never delete in bulk without an
explicit go-ahead.
