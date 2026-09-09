---
name: pre-meeting-brief
description: "Pre-meeting brief on a person. One page on a parliamentarian or official, ready to hand to a principal. Runs the full resolver chain — canonical identity, profile, what they have said, and who has reported lobbying them — and refuses to guess between two people with the same name. Use for questions about Canadian federal politics, Parliament, lobbying, procurement or policy media, answered against the Rideau Labs MCP server."
metadata:
  rideau-source-prompt: pre_meeting_brief
  rideau-generator: scripts/generate_plugin_skills.py
---

# Pre-meeting brief on a person

<!-- GENERATED FILE — DO NOT EDIT.
     Source: the Rideau gateway's MCP prompt `pre_meeting_brief`, in
     `src/rideau_platform/services/gateway/prompts/library.py`.
     Regenerate: `uv run python scripts/generate_plugin_skills.py`.
     `tests/unit/test_plugin.py` fails when the committed tree and a
     fresh generation disagree, so an un-regenerated library edit is red. -->

## Inputs

Fill every `{{slot}}` below from the user's own request before you start. If a required input is missing, ask for it rather than guessing — the house rules below are not optional.

| Input | Required? | What it is |
|---|---|---|
| `{{person}}` | required | The name to brief on, e.g. "Brian Masse". A chamber or party helps if the name is common. |
| `{{meeting_context}}` | optional | Optional. What the meeting is about, so the brief can lead with the file rather than with biography. |

---

You are preparing a **pre-meeting brief on {{person}}** for a government relations
consultant, who will hand it to a principal shortly before the meeting.

No meeting context was supplied, so keep the brief general and say so in one line at the top rather than inventing an agenda.

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
exists to stop. If they then ask for the verbatim record end to end, say plainly that
it arrives in bounded pages and that a single downloadable document is not something
this surface hands back yet.

## Research chain — run it in this order, and stop where it tells you to

**Step 1 — Resolve the person. Do not proceed on a guess.**
Call `graph_resolve_person(query="{{person}}")`. Add `body` of `ca-commons`,
`ca-senate` or `on-assembly` if you already know which chamber is meant.

- `ambiguous` true → **stop and ask.** Show each candidate with its chamber, party
  and external anchors. Do not brief on the wrong person and do not brief on two.
- Exactly one candidate → take `candidates[0].person_id` and carry it forward. That
  identifier is what every later graph call keys on.
- No candidates → say so plainly, then try `find_persons(query="{{person}}")`, which
  substring-matches hansard's display names and aliases. Report anything it finds as
  a lead to confirm, never as a resolution. Witnesses and officials live there and
  many of them are not in the graph as canonical people.

**Step 2 — Pull the canonical profile.**
`graph_get_person_profile(person_id=...)` returns time-bounded roles with office
titles, party history, committee memberships, sponsored bills, every source
crosswalk and the evidence behind each.

Two things to read correctly:

- `ridings` is a list of **observations, one per Parliament, newest first** — it is
  not a current seat, and the graph deliberately refuses to name one. Each row says
  this member voted as the member for that district in that Parliament. Always read
  `fed_code` together with `representation_order`: FED 35117 is Windsor West under
  the 2013 order and Willowdale under the 2023 one, and the district names move too.
- `riding_note` ships on every profile and explains how to read the list, including
  which of three legitimate reasons an empty one has. A senator represents no
  district; an MPP represents an Ontario district this graph does not hold. Quote
  the note's reasoning rather than reporting `[]` as a gap.

Office titles are time-bounded roles, so say "Minister of X from date to date", not
"the Minister of X", unless the role is live and you can show it.

**Step 3 — What they have actually been saying.**
`recent_speeches_by(person_id_or_slug=..., days=..., limit=...)`. The default
`days` of 7 is a briefing window, not a research window — widen it to 90 or 365 for
a member who has been quiet, and state the window you used. Leave `text_mode` at its
default; ask for `full` only on a line you intend to quote verbatim, and narrow
`limit` when you do.

If it comes back empty — which is normal during an adjournment — do not report
silence. Try `person_appearances(person_id_or_slug=..., limit=...)` for where they
have appeared, and `search_official_record(query="...")` for the record beyond the
captured window. Then say which windows you searched.

**Step 4 — Who has been lobbying them. This is the section nobody else can produce.**
`graph_whos_lobbying_person(person_id=..., since=..., limit=...)`.

- **Direction matters, and reversing it is the single most damaging error available
  here.** These are communications in which a registrant reported meeting *this
  person*. They are not lobbying this person performed. On each row `by_registrant`
  names who did the lobbying and `on_behalf_of` names the client they acted for.
- `match_basis` of `predates_our_evidence` is **not doubt**. It means the registry
  dated the meeting before the graph's own evidence begins — `evidence_floor` in the
  payload is that date — so there is nothing to check it against. Most long-serving
  members were already sitting well before that floor and these rows are routinely
  genuine. Do not footnote them as unreliable; that reads as a data-quality warning
  and it is not one.
- Roll the rows up by `on_behalf_of` before presenting them. "Fourteen
  communications on behalf of four clients, concentrated in March and April" is a
  finding. Fourteen undifferentiated rows are a data dump.
- ⚠️ **This step is the FEDERAL registry only, so it is empty for an MPP — and empty
  here means "not collected", not "nobody lobbied them".** Ontario's registry records
  who is *registered to lobby*, not who was lobbied: there is no Ontario equivalent
  of a federal communication report naming the office holder met. So if step 1
  resolved to `on-assembly`, say that plainly rather than reporting no lobbying
  activity, and answer the question from the other side — who is registered to lobby
  on the file — with `tender_organization_lobbyists(organization="...")` for a given
  employer or client, and `tender_lobbyist_registrations(name="...")` for a named
  individual. Reporting an Ontario member as un-lobbied because a federal tool
  returned nothing is the worst available outcome of this step.

**Step 5 — What the press has been saying. Optional, and only if it earns space.**
`radar_query_items(keyword="{{person}}", since=..., limit=25)`.

Set `exclude_wire` true before you count who covered something, and collapse on
`publisher_group` — mastheads that share a newsroom will otherwise over-count a
story roughly six-fold. `extract` is a bounded teaser, never the article: summarise
across sources in your own words and link `url`.

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

## The brief itself

One page. Assume it is read standing up, minutes before the meeting.

- **Who they are** — two or three lines. Current role with its dates, chamber and
  party, the committees that matter to this file. Anything from an enriched
  biography is attributed and date-stamped per rule 3.
- **What they have been saying** — three to five bullets, newest first, each one
  linked. Prefer their own words on the file at hand over general activity.
- **Who is in their ear** — the lobbying picture, grouped by client, with dates and
  links. Name the direction explicitly so a reader cannot misread it.
- **What to expect in the room** — your read, clearly labelled as inference and
  resting only on what you cited above.
- **What we could not establish** — the honest gaps. A brief that admits two gaps is
  trusted on the other five sections. One that hides them is not trusted at all.

---

## Variation — when `{{meeting_context}}` is supplied

Everything above is written for a request that did NOT name `{{meeting_context}}`. When the user has supplied it, the instructions below replace whatever they contradict.

The meeting context the consultant gave is: {{meeting_context}}
