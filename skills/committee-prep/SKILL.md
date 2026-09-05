---
name: committee-prep
description: "Prep pack for a committee appearance. Prep pack for an appearance: members, the study, past witness Q&A, positions. Resolves the committee on the graph, reads hansard's studies and hearings, and caps witness and member lookups so the pack arrives before the hearing. Use for questions about Canadian federal politics, Parliament, lobbying, procurement or policy media, answered against the Rideau Labs MCP server."
metadata:
  rideau-source-prompt: committee_prep
  rideau-generator: scripts/generate_plugin_skills.py
---

# Prep pack for a committee appearance

<!-- GENERATED FILE — DO NOT EDIT.
     Source: the Rideau gateway's MCP prompt `committee_prep`, in
     `src/rideau_platform/services/gateway/prompts/library.py`.
     Regenerate: `uv run python scripts/generate_plugin_skills.py`.
     `tests/unit/test_plugin.py` fails when the committed tree and a
     fresh generation disagree, so an un-regenerated library edit is red. -->

## Inputs

Fill every `{{slot}}` below from the user's own request before you start. If a required input is missing, ask for it rather than guessing — the house rules below are not optional.

| Input | Required? | What it is |
|---|---|---|
| `{{committee}}` | required | Acronym (FINA, INDU, on:GA) or full name. Queen's Park acronyms (on:…) resolve on the graph; appearance and Q&A go through find_events, not the federal live/upcoming feed. |
| `{{study_or_context}}` | optional | Optional. The study or appearance to lead with, so the pack is not a general committee briefing. |

---

Prepare a **committee appearance pack for {{committee}}.**

This is what a consultant hands a witness or a principal the night before: who
is on the committee, what they have been studying, who testified last, and where
the members have already put themselves on the record. It is a prep pack, not a
transcript dump.

No study or appearance context was supplied, so prep the committee as it sits today and say so in one line at the top rather than inventing a hearing.

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

## Research chain — run it in this order, and stop where it tells you to

**Step 1 — Resolve the committee. Do not proceed on a guess.**
Call `graph_get_committee(query="{{committee}}")`. Add `body` of `ca-commons`,
`ca-senate` or `on-assembly` if you already know which chamber is meant. The
query may be an acronym (FINA, on:GA) or a full name.

- `ambiguous` true → **stop and ask.** The same acronym can be a House committee
  and a Senate committee. Show each candidate with its `body` and `name_en`.
- Exactly one candidate → take `resolved` (or `candidates[0]`). Carry
  `committee_id`, `name_en`, `body`, and the acronym from `aliases` — hansard's
  tools key on the **exact acronym**, not the graph UUID.
- No candidates → if this looks federal, try `list_committees(active_only=true)`
  as a browse, then stop and ask. ⚠️ **`list_committees()` is House and Senate
  only — Ontario never appears on it.** For an `on:` acronym with no graph hit,
  stop and ask. Do not brief a near-miss.

Read `members` correctly: `current` means the membership was still present at
the last complete re-read of the source (`membership_current_as_of`). The graph
never deletes, so a departed member's edge persists and is identified by going
stale. ⚠️ **An empty `members` list on a Senate committee is a known gap in the
graph, not proof the committee has no members.** Say that, then continue — hansard
still has the studies, the hearings and the spoken record.

**Ontario vs federal is a tool split, not a filter.** Graph resolve works for
Queen's Park (`on:GA`, `body` of `on-assembly`). Hansard's `live_events()`,
`upcoming_events()`, `get_committee()`, `list_committees()` and
`search_utterances()` are **federal only** (House and Senate). They return `[]`
during a Queen's Park sitting. Empty from those five is **not collected here**,
not "nothing scheduled" and not "no Q&A". `find_events()` and `get_event()` are
all-jurisdiction and are the Ontario appearance and hearing path. If resolved
`body` is `on-assembly`, or the acronym starts with `on:`, skip the federal
tools in steps 2–5 and use the Ontario branch of each step.

**Step 2 — Studies, chair, and the live roster hansard holds.**
- **Federal** (`ca-commons`, `ca-senate`, acronyms like INDU/FINA):
  `get_committee(committee_acronym="<acronym from step 1>")` returns chair,
  vice-chairs, members, `recent_topics`, and `current_study_titles`. That last
  field is the study hop this pack exists for. If step 1 could not supply an
  acronym, stop rather than guessing one — `get_committee()` matches the acronym
  exactly and a wrong one comes back null, which looks like "no such committee".
- **Ontario** (`on-assembly`, `on:` prefix): **do not** call `get_committee()`.
  It is federal-scoped (0 recent events for `on:GA`). The graph roster from
  step 1 is what you have; hearings and studies come from `find_events()` below.

**Step 3 — What is on right now, and what is coming.**
- **Federal:** `live_events(event_type="committee")` — if this committee is in
  session, say so first; that changes the pack from prep to "you are late".
  `upcoming_events(within_hours=168, event_type="committee",
  committee_acronym="<acronym>")`. Read `is_in_camera`, `status` (cancelled rows
  are still returned), `study_titles`, `expected_attendees` and `provenance`.
  Do not present `officially_scheduled` as settled. Empty during a recess is
  "nothing scheduled".
- **Ontario:** **do not** treat `live_events()` / `upcoming_events()` as the
  appearance source — both use a federal jurisdiction filter and come back
  empty the night before a listed Queen's Park hearing. Route this appearance
  through `find_events(jurisdiction="on-assembly", committee_acronym="<acronym>",
  since=..., until=..., event_type="committee", limit=10)`, then
  `get_event(identifier=..., include_witnesses=True, include_utterances=False)`
  on the sitting that is this appearance. `identifier` is the `id`
  `find_events()` returned (Ontario sittings have no ParlVU id). Empty
  live/upcoming here is **not collected here**, not "nothing scheduled".

**Step 4 — Past hearings and witness Q&A. Cap this.**
`find_events(committee_acronym="<acronym>", event_type="committee",
since=..., limit=10)`. If this is Ontario, add `jurisdiction="on-assembly"`.
If a study was named, add `query="<study>"`. Prefer `has_witnesses=true`
when the question is who testified.

Then, for **at most three** of the most relevant past meetings — the ones on this
study, newest first — call `get_event(identifier=..., include_witnesses=True,
include_utterances=False)`. `identifier` is the `id` `find_events()` returned.
Read the panels: who appeared, for whom, in what role. Pair with
`get_event_summary(parlvu_content_entity_id_or_date=...)` for topics, bills
referenced and key moments — skip that hop for an Ontario sitting with no
ParlVU id; the official moments on `get_event()` are the Q&A you have.

Do **not** pull a transcript for every meeting. If you need Q&A on the record:
- **Federal:** use `search_utterances(query="<study or witness org>",
  committee_acronym="<acronym>", limit=25)` once — leave `text_mode` at its
  default. Reach for `event_utterances(scheduled_event_id=..., limit=200)` only
  on the **one** hearing you will quote, and only one page of it.
- **Ontario:** **do not** call `search_utterances()` — it is federal captured
  transcript only and comes back empty for `on:GA`. Q&A is the official moments
  on `get_event()`.
⚠️ `search_official_record()` has no date filter (`query`, `jurisdiction`, `limit`,
`offset` only); passing `since` is a hard error. Skip it here unless the
captured window is empty, and if you do call
`search_official_record(query="{{committee}}", jurisdiction=..., limit=...)` with
no date argument. For Ontario, `jurisdiction="on-assembly"`.

**Step 5 — Member positions, without one call per member.**
Do not loop `recent_speeches_by()` over the roster. That is unbounded fan-out: a
committee of fifteen becomes fifteen speech dumps before you have written a
line. Instead:

- **Federal:** one `search_utterances(query="<the study or the file>",
  committee_acronym="<acronym>", limit=25)` already ran in step 4 if you had a
  study; reuse it. If you still need the chair or the critic on this file,
  **at most two** `recent_speeches_by(person_id_or_slug=..., days=90, limit=...)`
  calls, named and justified. Hansard's `person_id_or_slug` is hansard's UUID
  or openparl slug, **not** the graph `person_id` on the membership row. Resolve
  the name with `find_persons(query="<display_name>")` and use that id; passing
  a graph UUID into a hansard tool is a silent miss.
- **Ontario:** skip `search_utterances()`. Member positions come from the
  hearings you already opened with `get_event()`, or from
  `find_events(jurisdiction="on-assembly", committee_acronym="<acronym>",
  query="<the study or the file>", since=..., until=..., limit=10)`. At most
  two `recent_speeches_by()` calls still apply if hansard has the person, and
  still via `find_persons()` — but do not treat an empty federal utterance
  search as "they have not spoken".
- ⚠️ **`search_official_record()` still has no `since`.** Bound the spoken record
  with `find_events(since=..., until=...)` or with `recent_speeches_by()`'s `days`.

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

**This is the stop.** After resolve → studies → upcoming (or Ontario
`find_events()`) → ≤3 past hearings → at most two member speech lookups, do not
make another tool call. A prep pack that arrives after the hearing has failed,
however complete the research was.

## The pack

- **The committee** — body, acronym, chair and vice-chairs, current members with
  the `current`/stale reading stated. One line on the study in play.
- **This appearance** — date, in-camera flag, cancelled flag, expected attendees
  if the notice named them. **Federal:** if `live_events()` / `upcoming_events()`
  are empty, say nothing is scheduled and prep against the current study anyway.
  **Ontario:** do not use that sentence. Empty live/upcoming is **not collected
  here**; report what `find_events(jurisdiction="on-assembly", ...)` returned.
- **Who testified last** — organisations and roles from the three hearings, each
  linked, grouped so a repeat witness is visible as a repeat.
- **Where members already sit** — three to five bullets, named, linked, on this
  file. Inference labelled as inference.
- **What we could not establish** — including an empty Senate membership list
  named as a graph gap, and any hearing whose transcript you did not open.

---

## Variation — when `{{study_or_context}}` is supplied

Everything above is written for a request that did NOT name `{{study_or_context}}`. When the user has supplied it, the instructions below replace whatever they contradict.

The appearance or study the consultant named is: {{study_or_context}}. Lead with that file. Filter every later sweep by it rather than briefing the committee in general.
