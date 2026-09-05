# Rideau

Canadian legislative and policy intelligence, in the assistant you already work in.

Rideau is one knowledge graph over what government **says** (Hansard and committee
proceedings, ahead of the official record), who is **lobbying** whom (the federal
Registry of Lobbyists), what government **buys** (federal and provincial procurement),
and what the **press** reports. This plugin connects your assistant to it and adds the
eight government-relations workflows the platform ships as skills.

## What is in here

| | |
|---|---|
| **One MCP server** | `https://mcp.rideaulabs.com/mcp` — search, look up, watch, and read the corpora |
| **Eight skills** | the workflows below, each a procedure rather than a description |

| Skill | What it does |
|---|---|
| `getting-started` | What Rideau is, and the first questions worth asking |
| `daily-check` | The morning check: what is live, what is scheduled, what happened yesterday |
| `pre-meeting-brief` | One page on a parliamentarian or official, ready to hand to a principal |
| `whos-lobbying` | Who is lobbying a department, or what an organization is lobbying about |
| `issue-landscape` | Who is driving a file: rooms, witnesses, lobbying, press |
| `committee-prep` | Prep pack for a committee appearance |
| `whats-new` | The "I was away, catch me up" digest over a saved file |
| `monitoring-report` | The weekly monitoring report a junior would draft |

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

**Anything that reads Agent Plugins 1.0** — point it at this repository. `plugin.json`
and `mcp.json` at the root are the 1.0.0 manifests.

## Signing in

The plugin declares a bare MCP URL and no credentials. On first use your client runs
the OAuth handshake itself: the server answers an unauthenticated call with a `401`
naming its protected-resource metadata, your client registers dynamically, and you
approve the connection in a browser. Nothing is stored in this repository, and this
plugin grants no access on its own — every account is authorised at the server.

## Access

Rideau is a commercial service. An account is required, and what you can reach is
granted per corpus. Talk to us at [rideaulabs.com](https://rideaulabs.com).

## About these skills

The eight skills are **generated** from the same prompt library the MCP server serves
over `prompts/list`, so the procedure you get as a skill and the procedure you get as a
slash command are the same text. They are regenerated and diffed in CI; they are not
edited by hand.
