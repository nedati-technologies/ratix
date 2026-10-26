# ratix plugin for Claude Code

Connects a Claude Code session to Ratix as one of your agent identities
("Suraj's Agent (Otter)"), so what it does in Ratix is credited to that agent.

## Install

```sh
/plugin marketplace add nedati-technologies/ratix
/plugin install ratix@ratix
```

## Start a session

Ratix is required wherever the plugin is enabled: until the session is
connected, every other tool call is held back and Claude is sent to
`ratix_connect` first. You don't run anything yourself:

- **A repo already linked, with your login on this machine:** the session
  connects as Claude Code starts, with an identity picked for you.
- **Anything else:** `ratix_connect` opens ratix.app in your browser. Approve
  the code there (a linked repo's org is preselected), confirm the form in
  Claude Code, and the session connects. A repo that wasn't linked now has
  `.ratix/org.json` at its root. It holds no secrets: commit it, so
  worktrees and teammates share it.

Your login stays in `~/.ratix/contexts/`, one per repo. The home folder (and
anything above it) is never linked.

## Identities

A session gets the identity its repo last used on this machine, if it is
free; else your first free one; else a new one with a generated name. To
switch, ask Claude to call `ratix_switch_identity`, which shows a form, or
run `! ratix use <name>` (`--create <name>` for a new one).

Once connected, the tools are `ratix_status`, `ratix_projects`,
`ratix_switch_identity`, the issue tools (`ratix_issues`, `ratix_issue`,
`ratix_issue_create`, `ratix_issue_update`, `ratix_comment`, `ratix_labels`,
`ratix_members`) and the document tools below. Issues are addressed by key
(`RTX-12`). Everything an agent writes is credited to its identity.

## Documents

The org's Markdown pages, addressed by key (`DOC-12`):

- `ratix_docs` lists them as a tree, or searches them with `query`.
- `ratix_doc` reads one. A text over 40,000 characters comes back as its
  outline; read one part with `section` (a heading, or a path such as
  `Setup > Payments`), or all of it with `full: true`. `revision` reads an
  earlier revision.
- `ratix_doc_create` and `ratix_doc_update` write them. Updates prefer
  `edits`: replace a text that occurs exactly once, append, or rewrite a
  section. A whole-text `content` needs the `baseVersion` it was read at.
- `ratix_doc_history` lists the revisions; `ratix_doc_restore` goes back to
  one as a new revision.
- `ratix_doc_upload` uploads a file from this session's folder or the system
  temp folder (never from a hidden folder) and returns the Markdown that shows
  it. Create the document first, upload, then place the Markdown with
  `ratix_doc_update`.

## Upgrades

Update with `/plugin marketplace update ratix`, then `/plugin` →
Installed → ratix → Update now, then `/reload-plugins`. Each session tells
you once when a newer release is out.

On first use, a new release brings what older ones left behind up to date
on its own:

- `.ratix/project.json` becomes `.ratix/org.json`, staged with `git mv`.
  Commit it once every teammate runs 0.4.0 or later: older plugins only
  read `project.json`.
- Logins in `~/.ratix/contexts/` move to the new field names.
- An installed login service is refreshed.

## Escape hatch

`RATIX_ENFORCE=off` in the environment Claude Code starts with lets every
tool run unconnected: for an outage, CI, or a headless run. When Ratix
can't be reached at all, the session isn't held back either.

## Files

- `dist/ratix.mjs`: the `ratix` command line tool, bundled.
- `.mcp.json`: starts `ratix mcp` for each session.
- `hooks/hooks.json`: `SessionStart` reports the connection; `PreToolUse`
  calls `ratix_gate` on the plugin's own MCP server before every tool.
