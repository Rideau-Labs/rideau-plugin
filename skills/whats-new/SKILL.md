---
name: whats-new
description: "What changed on a file since a date. The \"I was away, catch me up\" sweep, routed from a saved file. A file's targets are already-resolved graph nodes, so each type goes to the tools that can use it. Named entities remain as a first-run path for someone who has no file yet. Both empty is a refusal — this does not invent a watchlist. Use for questions about Canadian federal politics, Parliament, lobbying, procurement or policy media, answered against the Rideau Labs MCP server."
metadata:
  rideau-source-prompt: whats_new
  rideau-generator: scripts/generate_plugin_skills.py
---

# What changed on a file since a date

<!-- GENERATED FILE — DO NOT EDIT.
     Source: the Rideau gateway's MCP prompt `whats_new`, in
     `src/rideau_platform/services/gateway/prompts/library.py`.
     Regenerate: `uv run python scripts/generate_plugin_skills.py`.
     `tests/unit/test_plugin.py` fails when the committed tree and a
     fresh generation disagree, so an un-regenerated library edit is red. -->

## Inputs

Fill every `{{slot}}` below from the user's own request before you start. If a required input is missing, ask for it rather than guessing — the house rules below are not optional.

| Input | Required? | What it is |
|---|---|---|
| `{{since}}` | required | The date to diff from, ISO format (YYYY-MM-DD). Everything before it is assumed already seen. |
| `{{file}}` | optional | Optional. Name of a saved file on this account. When set, targets come from get_file and entities is ignored. |
| `{{entities}}` | optional | Optional. Comma-separated people, orgs, departments or topics, used only when file is empty. |

---

Answer: **what changed since {{since}}?** You were not given a saved file or
named entities. **Stop. Do not invent a watchlist. Do not sweep the four corpora.**
This platform keeps per-account files and subscriptions; inventing
interests on the caller's behalf produces a briefing about the wrong file.

The tool refuses this call for the same reason, so there is nothing to route
around: `whats_new(since=...)` with neither `file` nor `entities` is an error,
not an empty digest.

Ask for the name of a saved file, or for named people / orgs / bills / issues.
One call is allowed if it helps them pick: `list_files()`. Then stop this
digest.

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
assembles the whole captured transcript into one Markdown file, stores it, and returns
a signed link that opens in a browser and works for seven days. None of it crosses this
conversation, so the cap does not apply and there is no paging to explain. Two things
to say when you hand the link over, because the file travels without you: the link is a
bearer capability, so anyone it is forwarded to can read the document until it expires;
and where the sitting has a published official record, the document indexes it and does
not reproduce it, because that record is licence-quarantined.

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

There is nothing to digest until a file or named entities are supplied. A
short refusal is the product working.

---

## Variation — when `{{file}}` is supplied

Everything above is written for a request that did NOT name `{{file}}`. When the user has supplied it, the instructions below replace whatever they contradict.

Answer: **what changed on the saved file "{{file}}" since {{since}}?**

This is the "I was away, catch me up" question. The reader has already seen
everything before {{since}}, so repetition is the failure mode — a digest that
restates known background has wasted the one thing it was asked for.

## One call. The fan-out is the tool's, not yours.

Call `whats_new(since="{{since}}", file="{{file}}")`.

That single call does all of the routing this prompt used to describe: it reads
the file (targets are already-resolved graph nodes), applies the six-target and
three-query budget, and asks four corpora in the dialect each one speaks —
`find_events()` for proceedings, `find_bills()` for bills, `radar_query_items()`
for media, `tender_search_lobbying_communications()` for the federal registry.
A person reaches the registry as a designated public office holder and an
organization as a client; you do not have to know that, and you must not
second-guess it by calling the granular tools again in parallel.

Add `until=` to close the window and `limit=` to widen or narrow the rows per
corpus. Nothing else is needed for the digest below.

**Do not re-run the corpora by hand.** A second sweep costs the caller their
rate limit, double-counts every row, and can only disagree with the answer you
already have. If a corpus came back `failed`, say so — do not retry it into a
different-looking answer.

## Read `coverage` before you write a word

Every source the tool could have answered from gets one line, and the five
statuses are five different answers:

- **`answered`** — rows came back; they are in `results` under that name.
- **`empty`** — the corpus answered and there is genuinely nothing. **This is the
  only status you may report as "nothing happened".**
- **`unentitled`** — this account holds no grant for that corpus. Say the corpus
  was not covered; do not say it was quiet.
- **`failed`** — the corpus did not answer. Say so plainly. Reporting this as
  silence is the single worst thing this digest can do.
- **`not_requested`** — nothing was searched there, and the line says why.

The `answer` field is the tool's own one-sentence verdict and it already
follows this rule: it says "nothing changed" only when every corpus answered.
Do not upgrade a hedged answer into a clean one.

Two caveats the tool reports rather than hides, and you repeat:

- **the bills corpus has no date filter at all**, so the window was applied to
  the rows after the fact; the coverage line says how many fell outside it and
  how many carried no readable date.
- **a bill or a committee has no route into the lobbying registry** — they are
  not parties to a communication report — so they are named in the coverage
  line rather than guessed at.

Also speak, before researching anything else:

- `unresolved` — ids on the file that are no longer in the graph. Name every
  row (`node_type` + `node_id`). Dropping them is how a digest pretends the
  watchlist is smaller than it is.
- `watchlist.skipped` — targets past the budget. Say how many were skipped.
- `position` is the file's stance. Use it to order consequence; it is not a
  query and you do not search it.

## The digest

- **Order by consequence, not by corpus.** Use `position` when the file has
  one. The reader does not care which pipeline found it. Lead with the thing
  that changes what they should do this week.
- **Every item is one line, dated, linked.** Date, what happened, why it
  matters, the link. If it needs a paragraph it is a briefing note, not a
  digest item.
- **Say what did not move.** "No lobbying activity reported on this file in
  the window" is a real finding — but only after checking `freshness`, because
  the federal registry publishes weekly while we fetch daily.
- **Separate a recess from silence.** If the House was adjourned across the
  window, say so once, at the top, and do not repeat it per target.
- **Close with the window you actually covered**, per corpus, and with the
  budget you hit: targets skipped, queries skipped, calls used. The reader
  should know the edges of the claim.
- **Close with `unresolved`** when the file carried any. Dropping them is how
  a digest pretends the watchlist is smaller than it is.

---

## Variation — when `{{entities}}` is supplied

Everything above is written for a request that did NOT name `{{entities}}`. When the user has supplied it, the instructions below replace whatever they contradict.

Answer: **what changed on {{entities}} since {{since}}?**

This is the "I was away, catch me up" question. The reader has already seen
everything before {{since}}, so repetition is the failure mode — a digest that
restates known background has wasted the one thing it was asked for.

## One call. The fan-out is the tool's, not yours.

Call `whats_new(since="{{since}}", entities="{{entities}}")`.

That single call does all of the routing this prompt used to describe: it reads
the file (targets are already-resolved graph nodes), applies the six-target and
three-query budget, and asks four corpora in the dialect each one speaks —
`find_events()` for proceedings, `find_bills()` for bills, `radar_query_items()`
for media, `tender_search_lobbying_communications()` for the federal registry.
A person reaches the registry as a designated public office holder and an
organization as a client; you do not have to know that, and you must not
second-guess it by calling the granular tools again in parallel.

Add `until=` to close the window and `limit=` to widen or narrow the rows per
corpus. Nothing else is needed for the digest below.

**Do not re-run the corpora by hand.** A second sweep costs the caller their
rate limit, double-counts every row, and can only disagree with the answer you
already have. If a corpus came back `failed`, say so — do not retry it into a
different-looking answer.

## Read `coverage` before you write a word

Every source the tool could have answered from gets one line, and the five
statuses are five different answers:

- **`answered`** — rows came back; they are in `results` under that name.
- **`empty`** — the corpus answered and there is genuinely nothing. **This is the
  only status you may report as "nothing happened".**
- **`unentitled`** — this account holds no grant for that corpus. Say the corpus
  was not covered; do not say it was quiet.
- **`failed`** — the corpus did not answer. Say so plainly. Reporting this as
  silence is the single worst thing this digest can do.
- **`not_requested`** — nothing was searched there, and the line says why.

The `answer` field is the tool's own one-sentence verdict and it already
follows this rule: it says "nothing changed" only when every corpus answered.
Do not upgrade a hedged answer into a clean one.

Two caveats the tool reports rather than hides, and you repeat:

- **the bills corpus has no date filter at all**, so the window was applied to
  the rows after the fact; the coverage line says how many fell outside it and
  how many carried no readable date.
- **a bill or a committee has no route into the lobbying registry** — they are
  not parties to a communication report — so they are named in the coverage
  line rather than guessed at.

Also speak, before researching anything else:

- `unresolved` — ids on the file that are no longer in the graph. Name every
  row (`node_type` + `node_id`). Dropping them is how a digest pretends the
  watchlist is smaller than it is.
- `watchlist.skipped` — targets past the budget. Say how many were skipped.
- `position` is the file's stance. Use it to order consequence; it is not a
  query and you do not search it.

## The digest

- **Order by consequence, not by corpus.** Use `position` when the file has
  one. The reader does not care which pipeline found it. Lead with the thing
  that changes what they should do this week.
- **Every item is one line, dated, linked.** Date, what happened, why it
  matters, the link. If it needs a paragraph it is a briefing note, not a
  digest item.
- **Say what did not move.** "No lobbying activity reported on this file in
  the window" is a real finding — but only after checking `freshness`, because
  the federal registry publishes weekly while we fetch daily.
- **Separate a recess from silence.** If the House was adjourned across the
  window, say so once, at the top, and do not repeat it per target.
- **Close with the window you actually covered**, per corpus, and with the
  budget you hit: targets skipped, queries skipped, calls used. The reader
  should know the edges of the claim.
- **Close with `unresolved`** when the file carried any. Dropping them is how
  a digest pretends the watchlist is smaller than it is.
