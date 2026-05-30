# Power Automate Skills (paired bundle)

Two Claude Code skills that work together to build, edit, and deploy Microsoft
Power Automate cloud flows by editing the flow definition JSON directly and
deploying with the PAC CLI — no clicking through the designer.

They are a **pair**. Install both; they reference each other.

| Skill | Responsibility |
|---|---|
| **power-automate-pac** | The *workflow and JSON structure*: PAC CLI auth/export/unpack/pack/import, the flow definition schema, triggers/actions/runAfter/scopes, expressions, editing recipes, deployment, gotchas, best practices. |
| **power-automate-connectors** | The *authoritative action catalog*: which `operationId`s and parameter keys actually exist per connector (SharePoint, Outlook, Excel, OneDrive, Forms, Approvals, etc.), sourced from official Microsoft Learn docs. |

## Why two skills

`power-automate-pac` teaches you *how* to assemble and ship a flow.
`power-automate-connectors` is the reference you check so you never write an
action that doesn't exist. The PAC skill contains a HARD RULE that requires
consulting the connectors skill before emitting any connector action — because
a hallucinated `operationId` or parameter key produces a flow that packs and
imports "successfully" but fails at flow-save time with cryptic schema errors.

## Install

Copy both folders into your Claude Code skills directory:

```
~/.claude/skills/power-automate-pac/SKILL.md
~/.claude/skills/power-automate-connectors/SKILL.md
```

On Windows that's:

```
C:\Users\<you>\.claude\skills\power-automate-pac\SKILL.md
C:\Users\<you>\.claude\skills\power-automate-connectors\SKILL.md
```

Restart Claude Code (or start a new session) so the skills are registered.

## Prerequisites

- **PAC CLI** installed (`pac install latest`, or the .NET tool / MSI).
- Access to a **Power Platform environment** you can read/write.
- Flows must live inside a **Dataverse solution** (managed or unmanaged). A
  personal/non-solution flow must first be added to a solution via the maker
  portal (Solutions → + Add existing → Cloud flow → From outside Dataverse).
- `git` recommended for versioning the unpacked solution source.
- `python` or `node` recommended for scripting edits to large flow JSON.

## Usage

Once installed, just ask Claude to work on a Power Automate flow ("export the
X solution and add an approval step to flow Y"). The PAC skill triggers on any
work with `.json` flow definitions / `Workflows/*.json` / `OpenApiConnection`
actions, and it will pull in the connectors skill to validate every action.

## Conventions in the docs

The skill docs use generic placeholders. Substitute your own values:

| Placeholder | Meaning |
|---|---|
| `<prefix>` | your publisher customization prefix (e.g. `contoso`) |
| `<prefix>_SharePointConnRef` | a connection reference logical/schema name |
| `<tenant>.sharepoint.com` | your SharePoint tenant |
| `<site>` | a SharePoint site name |
| `<list-guid>`, `<flow-guid>`, `<import-id>` | the relevant GUIDs |
| `user@contoso.com` | a recipient/approver email |
| `C:\path\...` | wherever you keep the solution zip / source |

## Notes

- Examples use **PowerShell** syntax. `pac` itself is identical cross-platform
  (macOS/Linux via the .NET tool); only path separators and shell line-continuation
  differ (`` ` `` in PowerShell vs `\` in bash).
- These skills cover the **PAC CLI + direct JSON editing** workflow only.
