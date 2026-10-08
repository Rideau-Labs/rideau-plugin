# Rideau

Canadian legislative and policy intelligence, in the assistant you already work in.

Rideau is one knowledge graph over what government **says** (Hansard and committee
proceedings, ahead of the official record), who is **lobbying** whom (the federal
Registry of Lobbyists), what government **buys** (federal and provincial procurement),
and what the **press** reports. This plugin connects your assistant to it and adds the
13 government-relations workflows the platform ships as skills.

## Home screen

In a host that supports Plugin Extensions, Rideau opens a home with a request field,
workflow starters, Discover stories, your Playbooks and recent Files. Browse the full
13-workflow catalogue, read a sourced story, or continue work in chat. Account reads
use the same Rideau permissions as the rest of the connector. Other clients retain
the skills and ordinary MCP tools. Chat handoff and scheduling depend on the host.

## What is in here

| | |
|---|---|
| **One MCP server** | `https://mcp.rideaulabs.com/mcp`: search, look up, watch, and read the corpora |
| **13 skills** | the workflows below, each a procedure rather than a description |

| Skill | What it does |
|---|---|
| `business-development` | Find prospective clients |
| `issue-research` | Research an issue |
| `brief-me` | Brief me |
| `files-and-alerts` | Manage Files and alerts |
| `playbook` | Build or continue a Playbook |
| `write-document` | Write or improve a document |
| `people-search` | Find people and office staff |
| `procurement-search` | Search tenders and contracts |
| `meeting-prep` | Prepare for a meeting |
| `committee-prep` | Prepare for a committee |
| `hearing-brief` | Brief me on a hearing |
| `lobbying-brief` | Analyze lobbying |
| `routine` | Run this as a routine |

The gateway retains older MCP prompt names for compatibility. The plugin catalogue
uses these 13 skills. Routines depend on a supported host scheduler; live hearing
briefs are snapshots unless the host provides a continuous capability.

## Install

**Claude Code**

```
/plugin marketplace add rideau-labs/rideau-plugin
/plugin install rideau@rideau-labs
```

or from a local checkout:

```
claude plugin marketplace add ./rideau-plugin
claude plugin install rideau@rideau-labs
```

**Anything that reads Agent Plugins 1.0**: point it at this repository. `plugin.json`
and `mcp.json` at the root are the 1.0.0 manifests.

## Signing in

The plugin declares a bare MCP URL and no credentials. On first use your client runs
the OAuth handshake itself: the server answers an unauthenticated call with a `401`
naming its protected-resource metadata, your client registers dynamically, and you
approve the connection in a browser. Nothing is stored in this repository, and this
plugin grants no access on its own: every account is authorised at the server.

## Access

Rideau is a commercial service. An account is required, and what you can reach is
granted per corpus. Talk to us at [rideaulabs.com](https://rideaulabs.com).

## Licence

Proprietary. You may install this plugin and use it to reach the Rideau service; the
skill documents are not licensed for copying, modification, redistribution, or use in a
competing product. See [LICENSE](LICENSE).

## About these skills

The 13 skills are **generated** from the same prompt library the MCP server serves
over `prompts/list`, so the procedure you get as a skill and the procedure you get as a
slash command are the same text. They are regenerated and diffed in CI; they are not
edited by hand.
