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

**4 — Empty is an answer, and often the right one, but say when it ends.** The House
of Commons is adjourned for most of the calendar year. `live_events()` returning an
empty list and `upcoming_events()` coming back thin are correct during a recess, not a
malfunction. Say "the House is not sitting" and move to the corpora that keep moving
through an adjournment: the lobbying registry, the procurement feed and the media
corpus all do. Never manufacture chamber activity to fill a heading, and never present
a recess as a finding.

⚠️ **"Check back closer to the return" is not good enough, because we hold the return
date.** `chamber_calendar()` reads the statutory sitting calendar ourcommons.ca
publishes (a different object from the broadcast schedule the event tools read) and
answers when the House next sits, how far away that is and when it rose. `upcoming()`
already carries it as its `chamber_status` section. So the honest recess sentence has
the shape "the House is adjourned and returns <weekday> <date>, N days away", with
that tool's own `source_url` attached under rule 1. Take the date from the tool on the
day, never from this paragraph or from memory: the calendar is refreshed roughly
yearly, and a remembered sitting date is exactly the error this rule exists to stop.
Two of that tool's five statuses are measurements (`sitting`, `adjourned`);
`outside_coverage`, `no_calendar` and `invalid_date` all mean the calendar could not
answer, and reporting any of them as a recess asserts an adjournment nobody measured.

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

**6. Too big to hand over is not the same as unavailable. Answer the meeting, do not
page it.** One committee sitting runs to most of a megabyte of transcript, and this
gateway refuses to serve more than 200 000 bytes at once. The refusal is the platform
working: a truncated transcript reads exactly like a complete one, and the reader has
no way to tell. So "read me this meeting" is answered **summary first**.
`get_event_summary()` is what happened (the narrative, the topics, the bills and the
key moments) in a few thousand bytes, under one percent of the transcript it covers.
Lead with it, then quote the two or three exchanges that matter, each pulled by one
targeted read. Those reads are filtered in the database and always have been:
`event_utterances()` takes `person_id_or_slug` for one member's turns, `panel_index`
for one witness panel and `language` for one side of a bilingual sitting, and
`get_event(include_utterances=True)` reads the transcript straight through from any
`utterance_offset`. ⚠️ **Both default to a page of 200, and on a real hearing that
page is over the cap and the call is refused, so you get nothing rather than a lot:
bound every one of them.** `limit` around 25 on `event_utterances()`,
`utterance_limit` around 25 on `get_event()`. ⚠️ **And to reach inside a meeting you
have already identified, go to `event_utterances()` on its id, not to
`search_utterances()`.** Search is how you find a meeting or follow a theme across
many, and it answers a narrower question than it looks like it does: it matches on
every word of the query, so a whole topic phrase finds nothing, and its date window
filters the individual turn, which on an evening sitting carries the NEXT day's UTC
timestamp. Offer these reads as what they are, the way a person reads a hearing, and
never as a consolation prize after a failed call. **Never open an answer with a paging
plan.** A reader who asked what happened at a hearing and got a menu of ways to fetch
it in pieces has been handed the work back, and that is the whole defect this rule
exists to stop. And if they then ask for the verbatim record end to end, that IS
something this surface hands back: `event_transcript_document(scheduled_event_id=...)`
assembles the whole captured transcript and gives you links to it. **It comes back in
two formats and the first one, marked `primary`, is the one to hand over: a web page
that opens in a browser**, with every turn anchored, a filter by speaker or by what was
said, and a link to the video beside each turn. The second is the same document as a
Markdown file, for a reader who wants to keep, forward or diff the text. Each link
works for seven days. So offer it as a page somebody opens and reads, not as a file
they download. None of it crosses this conversation, so the cap does not apply and
there is no paging to explain. Two things to say when you hand the link over, because
the document travels without you: the link is a bearer capability, so anyone it is
forwarded to can read it until it expires; and where the sitting has a published
official record, the document indexes it and does not reproduce it, because that record
is licence-quarantined.

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
⚠️ **A key moment tagged `captured` carries no words.** It is a `label`, a
`why_notable` and an `utterance_id`, and no tool on this surface fetches an
utterance by id, so there is nothing to put in quotation marks. Report it as what
it is (the summariser's note on what mattered) and go get the actual language with
a bounded read below. Only a moment tagged `official` arrives with an `excerpt`
already attached.

Do **not** pull a transcript for every meeting. If you need Q&A on the record:
- **Federal:** use `search_utterances(query="<study or witness org>",
  committee_acronym="<acronym>", limit=25)` once — leave `text_mode` at its
  default. Reach into the transcript itself only on the **one** hearing you will
  quote, and read house rule 6 before you do: `event_utterances()` and
  `get_event(include_utterances=True)` both default to a page of 200, and on a busy
  committee sitting that page is over this gateway's response cap and is refused
  outright. Bound it. `event_utterances(scheduled_event_id=..., limit=25)` is the
  safe read, and it filters server-side: `person_id_or_slug` for the one member or
  witness you are quoting is one call, not a compromise.
  ⚠️ **A single speaker can still exceed the cap on a long sitting**, so pass `limit`
  on the filtered read too. On a typical two-hour hearing one member's whole
  contribution is 40 to 70 KB and comes back fine; on the longest sittings a single
  member has been measured at 578 942 bytes against a 200 000-byte cap.
  ⚠️ **`panel_index` is silent when the notice carried no panel times.** It returns
  an empty list, which reads exactly like "that panel said nothing". Half of a
  sampled dozen committee events had no panel times at all. Check
  `panel_start_at` on the panel from `get_event()` before you trust an empty result.
  ⚠️ **When the ask is to read the whole meeting rather than to quote from it,
  none of this paging is the answer.** Hand over
  `event_transcript_document(scheduled_event_id=...)`: it assembles the whole
  captured transcript and returns a link to it as a web page, good for seven
  days. Hand over the entry marked `primary`; the Markdown copy beside it is for
  a reader who wants the text as a file.
- **Ontario:** **do not** call `search_utterances()` — it is federal captured
  transcript only and comes back empty for `on:GA`. Q&A is the official moments
  on `get_event()`.

⚠️ **Two ways `search_utterances()` returns a confident zero on a meeting it holds.**
Both measured against a June 2026 SECU sitting on Bill C-22:

- **The query is an AND over every word.** `query="lawful access to electronic
  information"`, lifted verbatim from that sitting's own summary topics, returned
  eight rows and **not one of them from that sitting**. `query="lawful access"`
  found it. Search two or three words, never a whole topic string.
- **The date window is on the utterance, not on the sitting.** That meeting is
  scheduled 17 June, and its turns carry timestamps on **18 June UTC**, because an
  evening sitting in Ottawa runs into the next UTC day. `since="2026-06-17",
  until="2026-06-18"` therefore returns nothing for it. Give the window a day of
  slack on each side, or drop the dates and filter the rows yourself.

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
