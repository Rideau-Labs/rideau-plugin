---
name: monitoring-report
description: "Weekly monitoring report from a saved file. The weekly report a junior drafts, over one of this account's saved files. Reads the file via get_file — targets are already-resolved graph nodes — and surfaces any unresolved ids rather than dropping them. Not a free-text entity sweep. Use for questions about Canadian federal politics, Parliament, lobbying, procurement or policy media, answered against the Rideau Labs MCP server."
metadata:
  rideau-source-prompt: monitoring_report
  rideau-generator: scripts/generate_plugin_skills.py
---

# Weekly monitoring report from a saved file

<!-- GENERATED FILE — DO NOT EDIT.
     Source: the Rideau gateway's MCP prompt `monitoring_report`, in
     `src/rideau_platform/services/gateway/prompts/library.py`.
     Regenerate: `uv run python scripts/generate_plugin_skills.py`.
     `tests/unit/test_plugin.py` fails when the committed tree and a
     fresh generation disagree, so an un-regenerated library edit is red. -->

## Inputs

Fill every `{{slot}}` below from the user's own request before you start. If a required input is missing, ask for it rather than guessing — the house rules below are not optional.

| Input | Required? | What it is |
|---|---|---|
| `{{file_name}}` | required | This account's name for the file ("the Vertria file"). |
| `{{since}}` | optional | Optional ISO date (YYYY-MM-DD). Empty means the last seven days. |

---

Draft this week's **monitoring report from the saved file `{{file_name}}`.**

This is the deliverable a junior would draft: dated, sourced, scoped to the
file, readable by a principal who will not open a tool. It is not a catch-up
sweep over a free-text entity list. The file's targets are already-resolved
graph nodes; that is the point.

No `since` was supplied, so use seven days ago (a weekly window) as the window start and say that default out loud in the answer. Do not invent a longer history.

## House rules

These bind the answer, not just the research. They are the whole difference between
a research note and something a consultant can hand to a principal.

**1 — Cite it or drop it.** Every row these tools return carries a link back to the
record that proves it: `deep_link` on a lobbying communication, an official-record
URL or a ParlVU clip on a speech, `legisinfo_url` on a bill, `url` on a media item,
a notice URL on a tender. Carry that link into the answer, attached to the sentence
it supports. A claim you cannot link is a claim you do not make. In government
relations a single bad edge discredits every other line on the page, so an
unsourced sentence costs more than it adds.

**2 — Abstain over infer.** When a resolver returns `ambiguous` true, `resolved` is
null and the graph is telling you it will not choose between two real entities.
Neither will you. List the candidates with whatever distinguishes them — chamber,
party, riding observations, external identifiers, client numbers — and ask which one
is meant. Never take the first candidate because it was first. The same restraint
applies to a thin result: name what is thin and what is missing instead of padding
the section to look complete.

**3 — Attribute, never assert.** Enriched biographical detail is sourced and
date-stamped and has to reach the reader that way. "Per their LinkedIn profile,
retrieved 3 August 2026, they were then Vice-President of Regulatory Affairs at Y"
is a claim about a profile on a date. "They are the VP of Regulatory Affairs at Y"
is a claim about the world, and only the first is what we hold. Keep the source and
the retrieval date on the clause, not in a footnote at the bottom. The registry gets
the same treatment: a lobbying communication report is what a registrant *filed*,
not an independently verified account of what happened in the room.

**4 — Empty is an answer, and often the right one.** The House of Commons is
adjourned for most of the calendar year. `live_events()` returning an empty list and
`upcoming_events()` coming back thin are correct during a recess, not a malfunction.
Say "the House is not sitting; nothing scheduled in this window" and move to the
corpora that keep moving through an adjournment — the lobbying registry, the
procurement feed and the media corpus all do. Never manufacture chamber activity to
fill a heading, and never present a recess as a finding.

**5 — A refused tool is entitlement, not breakage — the exception, not the new-account default.**
Access is granted per corpus. A new account can already reach hansard, tender, radar
and graph: hansard is ungated, and tender, radar and graph are granted automatically
when the account is created. A tool named below may still be absent from your tool
list, or may refuse a call as not entitled — that is a revoked grant or a corpus that
is not yet enabled, not the default for a new account, and it is not breakage. When
it happens: **name the corpus that is unavailable, say which section of the answer it
would have filled, and complete the answer from the corpora you can reach.** Do not
skip a tender, radar or graph step on the assumption that a new account cannot reach
it. Do not silently drop the section, do not substitute a different tool and present
its output as the same thing, and never improvise the content from general knowledge.
If a call actually refuses, "I could not check the lobbying registry — that grant is
not on this account — so the 'who is in their ear' section is missing rather than
empty" is a good answer. Quietly returning four sections where the reader expects
five is not.

## Step 1 — Read the file. Do not invent the roster.

Call `get_file(name="{{file_name}}")`. Reads are scoped to this account; a name
that is not on this account reports the same way as a name that was removed.

- `found` false → call `list_files()` once so you can name what *is* on this
  account, then **stop and ask** which file to use. Do not guess a nearby name
  and draft against it. Do not assemble a roster from the prompt argument.
- `found` true → the roster is `targets[]` (each `node_type` in
  `person | org | institution | bill | committee | issue`, plus canonical
  `node_id` and `name`), plus `queries[]` (saved search strings) and
  `position` (free-text stance). `position` is the account's own words; treat
  it as the lens the report is written through, attributed as such, never as a
  fact about the file.
- ⚠️ **Do not extract entities from `position`.** `position` is the lens/stance,
  not the roster. If there are no `targets` and no `queries`, write the report
  from that stance plus corpus freshness only (`graph_corpus_stats()`,
  `corpus_stats()`, `tender_corpus_stats()`, or a bounded
  `radar_query_items(since=..., until=...)` for media) — or stop and say the
  file has no resolved targets. Do not parse names out of the paragraph and
  run the per-type sweeps against them. Budget "eight of zero" is not a prompt
  to invent a watchlist.
- ⚠️ **If `unresolved` is present, say so in the report.** Those ids are no
  longer in the graph under the type they were saved with. List each one
  (`node_type`, `node_id`) in a dedicated "could not resolve" line. Do not
  silently drop them, do not pretend the target count is the whole file, and
  do not look them up under a different type as a guess.

There is no `delete_file`. A missing name is not something you can undelete.

## Step 2 — Route each target by `node_type`. Budget is the design.

Files may hold up to 100 targets. Sweeping every one across four corpora is how
a weekly report overruns the digest it is meant to be. **Hard budget:**

- Sweep **at most eight** node targets. If the file is larger, pick a diverse
  slice (at least one of each `node_type` present, then prefer targets the
  `position` already names *among those targets* — never new names parsed out
  of the paragraph) and **list the skipped names** at the end rather
  than implying the report covered the whole file.
- **At most two tool calls per swept target.** The primary call below, plus one
  drill if that call is thin. Then move on.
- Sweep **at most three** `queries[]`, each with one
  `radar_query_items(keyword="<query>", since=..., limit=25)` and/or one
  `find_events(query="<query>", since=..., limit=10)` — not both plus lobbying
  plus a bill lookup.
- After the budget is spent, **stop calling tools and write.** Do not start a
  second pass.

**Hansard ids are not graph ids.** A file `node_id` is the graph's canonical
UUID. Graph tools take it. Hansard tools (`recent_speeches_by()`,
`person_appearances()`, `find_events()`'s `person_id_or_slug`) take hansard's
UUID or an openparl slug. Passing a graph UUID into a hansard tool is a silent
miss. For a person target, use `node_id` on graph tools and `name` (via
`find_persons(query="<name>")`) on hansard tools.

Route:

- **person** — `graph_whos_lobbying_person(person_id="<node_id>", since=...,
  limit=...)` is the primary call (federal communications in which a registrant
  reported meeting *this person*; `by_registrant` / `on_behalf_of`;
  `predates_our_evidence` is not doubt). Optional drill: `find_persons()` then
  `recent_speeches_by(person_id_or_slug=..., days=..., limit=...)` — convert the
  window to a day count; that tool takes `days`, not a date.
- **org** — `tender_search_lobbying_communications(organization="<shortest
  distinctive fragment of name>", since=..., limit=...)`. Substring, never the
  full legal name: one company is routinely several registry nodes.
- **institution** — `tender_whos_lobbying(institution="<name>", since=...,
  limit=...)`.
- **bill** — parse the number and Parliament from `name` (it is stored like
  `C-36 (45-1) — title`). Then `get_bill(number_code="...", parliament=...,
  session_number=...)`. `find_bills()` / `get_bill()` take `session_number`;
  `graph_get_bill_identity(number_code=..., parliament=..., session=...,
  body=...)` takes `session`. Optional drill:
  `find_events(bill_reference="<number_code>", since=..., limit=10)`.
- **committee** — `graph_get_committee(query="<name>")` to recover the acronym
  from `aliases`, then `find_events(committee_acronym="<acronym>", since=...,
  limit=10)`. Do not guess an acronym.
- **issue** — `targets[].name` is stored like `Mental health (mental-health)`:
  English label, then the issue **code/slug in parentheses**. Extract that
  slug (`mental-health`) and pass **it** as
  `graph_find_by_issue(query="mental-health", limit=25)`. Do **not** pass the
  composite display name — `query="Mental health (mental-health)"` does not
  resolve (the tool matches exact slug, exact normalized label/alias, or an
  ILIKE of the whole string inside a slug or alias; there is no `issue_id`
  parameter). Do not invent `issue_id=`. A thin or empty tagged result is
  **not** "this issue was never discussed"; there has been no corpus-wide
  tagging pass. Optional drill: `find_events(query="Mental health", since=...,
  limit=10)` or `query="mental-health"` — the label or the slug, not the
  composite. Asking about a domain rolls up children (`via_child`); repeat
  `tagged_issue_code`, not the domain.

⚠️ **`search_official_record()` has no date filter** — parameters are `query`,
`jurisdiction`, `limit`, `offset` only. Passing `since` is a hard error. Bound
the official record through `find_events(since=..., until=...)`, or skip it.

If you reach for procurement, `tender_radar_digest(since=..., keywords=[...],
limit_per_stream=10)` clamps a `since` older than 180 days with no error. Read
`since` / `until` / `window_days` on the response and report the window you
actually got.

## What is actually in the corpus

Know this before answering; say it out loud only where it changes the answer.

- **Real-time captured transcript is the federal House and its committees.** For a
  federal person, `recent_speeches_by()` and `person_appearances()` return captured
  proceedings, which is what makes this corpus days-to-weeks ahead of the official
  record.
- **The official record reaches further than the captured corpus.**
  `search_official_record()` takes `jurisdiction` of `ca-commons` or `on-assembly`;
  Ontario rows arrive as official-record entries with an official URL and no clip.
  `find_events()` takes the same `jurisdiction` filter plus `ca-senate`.
- **Do not assert coverage from memory — read it.** `corpus_stats()`,
  `graph_corpus_stats()` and `tender_corpus_stats()` each report their own shape,
  coverage and freshness. Senate and Ontario coverage are still landing, so a claim
  about what is or is not in the corpus should come from one of those calls made just
  now, not from an assumption. The media corpus has no statistics tool on this
  surface: establish its coverage by querying a bounded window with
  `radar_query_items(since=..., until=...)` and reporting what actually came back.
- **Federal lobbying is fetched daily but the registry publishes weekly.** The
  corpus can legitimately sit several days behind lobbycanada.gc.ca and still be
  perfectly healthy. Read the `freshness` block before calling a gap a finding.
- **Ontario lobbying is a separate registry with separate tools.**
  `tender_organization_lobbyists()` and `tender_lobbyist_registrations()` answer
  Ontario; `tender_search_lobbying_communications()` and `tender_whos_lobbying()`
  answer federal. They are not interchangeable and neither covers the other.

## Stop calling tools and write

**This is the stop.** File read, unresolved named, budgeted sweeps done — write
the report. Do not go back for "one more" target. A late monitoring report is
not a monitoring report.

## The report

A junior's weekly, not an appendix.

- **Header** — file name, the window you actually covered per corpus, and the
  `position` quoted as the account's stance if one is set.
- **Unresolved** — if `unresolved` was present, it is a heading of its own, not
  a footnote. If it was absent, do not invent the heading.
- **What moved, by consequence** — each item one dated, linked line: what
  happened, which target it attaches to, why it matters for this file. Group
  nothing by corpus.
- **What did not move** — named, after checking `freshness` on the lobbying
  payload, because the federal registry publishes weekly while we fetch daily.
- **Skipped targets** — names only, if the eight-target cap left any out.
- **What we could not establish** — refused tools, empty captured windows,
  hearings whose transcripts you did not open.

If everything is quiet, say so in a short report and stop. Quiet-on-this-file
is the product working. A padded report teaches the reader to skim, and after
that they miss the week it matters.

---

## Variation — when `{{since}}` is supplied

Everything above is written for a request that did NOT name `{{since}}`. When the user has supplied it, the instructions below replace whatever they contradict.

The window starts at {{since}}. Everything before that is assumed already seen — repetition is the failure mode.
