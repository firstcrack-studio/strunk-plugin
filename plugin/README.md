# Strunk plugin

The install-once bundle of Strunk. Claude Code reads `.claude-plugin/plugin.json` and `.mcp.json`.
Hermes Agent, VS Code and other Agent Plugins hosts read `plugin.json` and `mcp.json`. Both point at
the same server and the same skills.

- **The Strunk remote MCP** (`.mcp.json`, `mcp.json`), served over Streamable HTTP at `https://strunk.io/mcp` and
  protected by WorkOS/AuthKit OAuth. It carries the atomic transport tools: `publish_doc`,
  `pick_google_doc`, `pull_comments`, `apply_revisions`, `reply_comment`, `resolve_comment`,
  `inspect_doc`, `get_preferences`, `set_preferences`, `get_account`, `get_version`,
  `get_review_workflow` and `get_writing_rules`.
- **Slash commands** (`commands/`, Claude Code only): `/strunk:pull-comments`, `/strunk:publish`,
  `/strunk:link-doc`.
- **Two skills.** `skills/strunk-review/` makes a session reach for Strunk when the author mentions a
  doc that came back with comments. `skills/strunk-setup/` walks the two connections Strunk needs.
- **`SETUP.md`**, followed on install, which ends at the author's first review rather than at
  "connected".

**Every one of these is a launcher.** The review workflow, the triage tiers and the author's writing
rules are not bundled here. `get_review_workflow` and `get_writing_rules` serve them live from the
Strunk backend, so they stay current without a plugin update and work on any harness that connects to
the MCP. The non-negotiable invariants ride in the MCP `instructions` field, present the moment the
server connects and before any tool call.

That split is deliberate: a bundled copy of the judgment drifts from the server that owns it, which is
why the original bundled skill was removed. Commands and skills here point at the server. They never
restate it.

## Hermes Agent

Needs Hermes 0.23.1 or later.

```bash
hermes plugins install strunk
hermes mcp add strunk --url https://strunk.io/mcp --auth oauth
```

The second command signs you in to Strunk. Your browser opens; if it does not, run
`hermes mcp login strunk`. Then start a new session, or run `/reload-mcp`, so the tools load. The
first time you publish or open a doc, Strunk asks you to connect Google Docs.

What to know before installing:

- **Network.** Every tool call goes to `https://strunk.io/mcp`. Publishing sends the draft you
  publish. Pulling comments returns reviewers' comments, names and the spans they anchor to. That
  content also reaches your model provider through the session, like anything else the agent reads.
- **Google access.** Strunk uses Google's `drive.file` scope, which covers only the docs Strunk
  creates and the files you pick for it.
- **Write tools change your Google Docs.** Publishing creates or updates a doc. Revising edits it.
  Replying and resolving act on comment threads under your Google account.
- **Approvals.** The package marks the `strunk` server untrusted, so Hermes asks before each write
  call. The sign-in command above writes its own `strunk` entry to `~/.hermes/config.yaml`, which
  replaces the package's, trust included. To keep the approval prompt, add `trust: untrusted` under
  `mcp_servers.strunk` in that file.
- **Credentials.** Hermes stores and refreshes the Strunk OAuth token. Your Google refresh token is
  held by WorkOS, Strunk's sign-in provider, and never reaches your machine. Nothing in this package
  reads either one.
- **No local code.** The package is two manifests and Markdown. The tool list comes from Strunk's
  server at run time, so the pinned commit does not cover it.

Privacy policy: https://strunk.io/privacy

Install and usage on other tools: see the [repo README](../README.md).
