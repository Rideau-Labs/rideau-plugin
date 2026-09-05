---
name: daily-check
description: "The morning check. What is live now, what is scheduled, and what happened yesterday. Short by design — one screen, read before the day starts. Honest about a recess rather than padding around it. Use for questions about Canadian federal politics, Parliament, lobbying, procurement or policy media, answered against the Rideau Labs MCP server."
metadata:
  rideau-source-prompt: daily_check
  rideau-generator: scripts/generate_plugin_skills.py
---

# The morning check

<!-- GENERATED FILE — DO NOT EDIT.
     Source: the Rideau gateway's MCP prompt `daily_check`, in
     `src/rideau_platform/services/gateway/prompts/library.py`.
     Regenerate: `uv run python scripts/generate_plugin_skills.py`.
     `tests/unit/test_plugin.py` fails when the committed tree and a
     fresh generation disagree, so an un-regenerated library edit is red. -->

## Inputs

Fill every `{{slot}}` below from the user's own request before you start. If a required input is missing, ask for it rather than guessing — the house rules below are not optional.

| Input | Required? | What it is |
|---|---|---|
| `{{entities}}` | optional | Optional. Comma-separated people, organizations or files to scope the check to. Left empty, it sweeps the corpus generally rather than inventing a watchlist. |

---

The 8:40am check. **What is happening today, and what happened yesterday that the
reader has not seen?**

No entities were named, so run the general sweep: what is happening today across the corpus, unfiltered. Do not invent a watchlist — this platform keeps per-account watches, and inventing interests on the caller's behalf produces a briefing about the wrong file.

Be fast and be short. This is read with a coffee in one hand before the day starts.
If it runs past a screen it has failed, however good the research was.

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

## The sweep

**1 — What is live right now.**
`live_events()`. Optionally narrow with `event_type` of `house_sitting`, `committee`,
`press_conference` or `other`. Each row carries the identifier that
`event_now()` and `current_speaker()` need, so if something relevant is under way you
can go straight to what is being said this minute — that is the wedge, and no
competitor's product answers it.

An empty list is the normal state. The House is adjourned for most of the calendar
year. Report it in one clause and move on; do not treat it as an outage and do not
pad around it.

**2 — What is coming.**
`upcoming_events(within_hours=24)`, widened to 168 for the week ahead or further
during an adjournment, when the look-ahead is what carries the value. Each row
carries expected attendees, bills referenced and study titles. Two fields decide
whether an item belongs in the note at all:

- `is_in_camera` true means closed to the public — worth knowing it is happening,
  pointless to plan coverage of. `null` means the notice did not say, which is not
  the same as public.
- `status` may be `cancelled`; cancelled events are still returned and labelled.
  Reporting one as upcoming is the exact error that makes a reader distrust the feed.
- `provenance` separates `officially_scheduled` (on the schedule, not yet confirmed
  by ParlVU) from `parlvu_confirmed`. Do not present the first as settled.

**3 — Yesterday, in summary form.**
`get_event_summary(parlvu_content_entity_id_or_date="YYYY-MM-DD")` accepts a plain
ISO date and returns that day's most recent ended event summary — meta description,
summary, topics, bills referenced and key moments. That is the cheap first hop.

To sweep a whole day rather than one event, use
`find_events(since="...", until="...", limit=...)` and take summaries from the ids it
returns. Each key moment is tagged `captured` or `official`; an official moment
arrives with its excerpt, speaker, subject and official URL already attached, while a
captured one resolves only through `get_event(identifier=..., include_utterances=True)`
— the flag defaults to false and the transcript is the thing it withholds, so a plain
`get_event()` will not carry the clip. Ask for it on the **one** event you actually
need: the transcript comes back in pages of 200 (`utterance_limit`,
`next_utterance_offset`), and requesting it across a day's events to find one clip is
how a morning note turns into a megabyte. The official record has no timestamps, so
there is no clip for it at all — do not promise the reader one.

**4 — Anything moving off the chamber.**
Only if the day is quiet or the scope calls for it:
`tender_radar_digest(since="<yesterday>", limit_per_stream=10)` for procurement, and
`radar_query_items(since="<yesterday>", limit=25)` for the press. Both keep moving
through an adjournment, which is what makes them the right fallback on a day when the
chamber sweep is empty.

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

## The note

Four headings at most, and drop any heading with nothing under it rather than
writing "nothing to report" four times.

- **Live now** — only if something is. Otherwise one clause: the House is not sitting.
- **Today and tomorrow** — what is scheduled, with the in-camera and cancelled flags
  respected.
- **Since yesterday** — what actually happened, one line each, linked.
- **Worth a look** — at most two items, each with the reason it is here.

If everything is empty, say so in a sentence and stop. A short honest note is the
product working. A padded one teaches the reader to skim, and after that they miss
the day it matters.

---

## Variation — when `{{entities}}` is supplied

Everything above is written for a request that did NOT name `{{entities}}`. When the user has supplied it, the instructions below replace whatever they contradict.

Scope it to these entities, treated as a comma-separated list: {{entities}}. Filter every sweep below by them and say so.
