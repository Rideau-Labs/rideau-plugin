---
name: issue-landscape
description: "Who is driving this file. A map of who is driving an issue or a bill: rooms, witnesses, lobbying, press. Starts at the issue lens, then events and witnesses, then the federal registry, then the media corpus. A thin tagged-with result is a successful answer, not evidence the issue was never discussed. Use for questions about Canadian federal politics, Parliament, lobbying, procurement or policy media, answered against the Rideau Labs MCP server."
metadata:
  rideau-source-prompt: issue_landscape
  rideau-generator: scripts/generate_plugin_skills.py
---

# Who is driving this file

<!-- GENERATED FILE — DO NOT EDIT.
     Source: the Rideau gateway's MCP prompt `issue_landscape`, in
     `src/rideau_platform/services/gateway/prompts/library.py`.
     Regenerate: `uv run python scripts/generate_plugin_skills.py`.
     `tests/unit/test_plugin.py` fails when the committed tree and a
     fresh generation disagree, so an un-regenerated library edit is red. -->

## Inputs

Fill every `{{slot}}` below from the user's own request before you start. If a required input is missing, ask for it rather than guessing — the house rules below are not optional.

| Input | Required? | What it is |
|---|---|---|
| `{{topic}}` | required | An issue ("carbon pricing", "health-safety") or a bill number ("C-27"). A bill number is resolved in time context first. |
| `{{since}}` | optional | Optional ISO date (YYYY-MM-DD). Window start for events, lobbying and media. Empty means the last 90 days. |

---

Map **who is driving this file: {{topic}}.**

This is not a literature review. It is a landscape: which bills, which rooms,
which witnesses, which registrants, which newsrooms. The reader wants the map,
not every row that mentions the term.

No `since` was supplied, so use the last 90 days as the window start and say that default out loud in the answer. Do not invent a longer history.

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

**Step 0 — Decide whether this is an issue, a bill, or both.**
A string like `C-27`, `C-36`, `S-201` is a bill number first. Bare numbers recycle
every session — C-36 is a 2022 supply bill in the 44th Parliament and a 2026
privacy bill in the 45th — so a bare number is never identity. Resolve it with
`graph_get_bill_identity(number_code="{{topic}}", parliament=..., session=...,
body=...)` if you already know the Parliament, or omit parliament/session and
read `collision`. Then `get_bill(number_code="...", parliament=...,
session_number=...)` for the milestone journey. Note `find_bills()` and
`get_bill()` take `session_number` while `graph_get_bill_identity()` takes
`session`. A collision is house rule 2: stop and ask which bill is meant.

If it is not a bill number, or once the bill is identified, continue. An issue
landscape on a bill still needs the issue lens, the rooms, the lobbying and the
press — the bill identity is the start, not the whole map.

**Step 1 — The issue lens. Thin is a successful answer.**
`graph_find_by_issue(query="{{topic}}", limit=25)`. An empty `query` lists the 14
L1 policy domains — do that if the string looks like a guess at a slug rather
than a term the vocabulary carries.

- `ambiguous` true → **stop and ask.** Pick a `code` from `candidates` and call
  again. Do not union the candidates into one landscape.
- Asking about a domain rolls up children. Each row names the issue it was
  actually tagged with in `tagged_issue_code`, and `via_child` marks the
  rolled-up ones. Repeat the specific issue, not the domain: the graph knows
  "tagged with Mental health, which sits under Health and safety" and does not
  know "tagged with Health and safety".
- ⚠️ **A thin or empty tagged result is a successful answer, not "this issue was
  never discussed".** Tags are admitted under a conservative floor and there has
  been no corpus-wide tagging pass, so most of the graph carries no issue tags
  at all. Absence of a tag is not evidence of absence in the chamber, the
  registry or the press. Attribute every tag (`attribution`, `evidence_deep_link`)
  and never assert it as a fact about the world. Then continue the chain — the
  events, lobbying and media sweeps are how you find what the tags do not yet
  cover.

**Step 2 — What government said, including who testified.**
- `find_events(query="{{topic}}", since=..., event_type="committee", limit=10)` for
  the rooms. Add `jurisdiction` to narrow, or leave it off. `has_witnesses=true`
  when you specifically want hearings. Bound dates live here, not on the
  official record.
- ⚠️ **`search_official_record()` has no date filter.** Its only parameters are
  `query`, `jurisdiction`, `limit` and `offset`. Passing it a `since` is a hard
  error rather than an ignored argument. Call
  `search_official_record(query="{{topic}}", limit=...)` only as a supplement, and
  filter its rows yourself on the dates they carry — or skip it and stay on
  `find_events(since=..., until=...)`.
- Witnesses live on the event, not on the issue. For **at most three** of the
  most relevant hearings, call `get_event(identifier=..., include_witnesses=True,
  include_utterances=False)` — `identifier` is the `id` `find_events()` returned
  (Ontario sittings have no ParlVU id). Read the panels. Do not request
  transcripts across the set; that is how a landscape turns into a megabyte.
  Quote from `get_event_summary(parlvu_content_entity_id_or_date=...)` or from
  one bounded page of `event_utterances(scheduled_event_id=..., limit=25)` on the
  **one** hearing you will actually write about. ⚠️ That tool's own default is a
  page of 200, which on a busy sitting is over this gateway's response cap and is
  refused outright, so always name a `limit`.
  If the reader wants to READ that hearing rather than have you quote from it,
  do not page it at all: `event_transcript_document(scheduled_event_id=...)`
  hands back the complete transcript as a web page, and none of it crosses this
  conversation or the cap. Give them the link marked `primary`, which is the
  page; a Markdown copy of the same document comes back beside it.

**Step 3 — Who has been lobbying on it.**
`tender_search_lobbying_communications(subject="{{topic}}", since=..., limit=...)`.
`subject` matches a subject-matter code, its description, or free-text detail —
raise `limit` before you conclude volume from one page. Roll up by
`organization` (the client) and by institution, not by row. Link every
communication. This is the federal registry; Ontario answers who is *registered
to lobby*, not who was met — if the file is Queen's Park, say that and use
`tender_organization_lobbyists(organization="...", active_only=...)` only when
a named employer is already in play, not as a fishing expedition.

If step 0 resolved a department rather than a topic, switch to
`tender_whos_lobbying(institution="...", since=..., limit=...)` and do not also
dump every communication.

**Step 4 — What the press has been saying.**
`radar_query_items(keyword="{{topic}}", since=..., limit=25)`. Set `exclude_wire`
true and collapse on `publisher_group` before you count who covered it. `extract`
is a teaser; summarise in your own words and link `url`. `radar_get_item(item_id=...)` only
for the one or two stories you will actually write about.

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

**This is the stop.** After the four sweeps above — issue lens, events/witnesses
(≤3 hearings), lobbying, media — do not make another tool call. Do not paginate
to exhaust a result. Do not open a fifth sweep. Write the map from what you have.

## The map

- **The file, in one paragraph** — the issue or bill as resolved, with its
  identity and the window you actually covered.
- **Who is in the room** — committees and studies, then the witness organisations
  that keep appearing, each linked. A one-off witness is a footnote, not a
  heading.
- **Who is lobbying** — grouped by client, with dates and registry links. Name
  the direction: these are communication reports, not independently verified
  meetings.
- **Who is covering it** — outlets, not headlines, collapsed on newsroom.
- **What the tags do and do not say** — attributed, and explicit that a thin
  tagged-with result is not "never discussed".
- **What we could not establish** — the honest gaps, including any corpus you
  could not reach.

---

## Variation — when `{{since}}` is supplied

Everything above is written for a request that did NOT name `{{since}}`. When the user has supplied it, the instructions below replace whatever they contradict.

The window starts at {{since}}. Everything before that is assumed already seen — repetition is the failure mode.
