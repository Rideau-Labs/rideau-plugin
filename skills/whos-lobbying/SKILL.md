---
name: whos-lobbying
description: "Who is lobbying on a subject or organization. Who is lobbying a department, or what an organization is lobbying about. Written around the fragmentation trap that makes a naive lookup under-report: the registry mints one organization node per client number, so one company is routinely several nodes and an exact-match search silently misses most of its activity. Use for questions about Canadian federal politics, Parliament, lobbying, procurement or policy media, answered against the Rideau Labs MCP server."
metadata:
  rideau-source-prompt: whos_lobbying
  rideau-generator: scripts/generate_plugin_skills.py
---

# Who is lobbying on a subject or organization

<!-- GENERATED FILE — DO NOT EDIT.
     Source: the Rideau gateway's MCP prompt `whos_lobbying`, in
     `src/rideau_platform/services/gateway/prompts/library.py`.
     Regenerate: `uv run python scripts/generate_plugin_skills.py`.
     `tests/unit/test_plugin.py` fails when the committed tree and a
     fresh generation disagree, so an un-regenerated library edit is red. -->

## Inputs

Fill every `{{slot}}` below from the user's own request before you start. If a required input is missing, ask for it rather than guessing — the house rules below are not optional.

| Input | Required? | What it is |
|---|---|---|
| `{{subject_or_org}}` | required | A government institution ("Health Canada", "ISED") or an organization ("Detroit International Bridge Company"). |

---

Answer a lobbying question about **{{subject_or_org}}**: who is lobbying, for whom, on
what subjects, over what window — and how we know.

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

## The trap this prompt exists to defuse

**One company is routinely eight organization nodes, and an exact-match lookup
silently under-reports.**

The federal lobbying registry issues a fresh client number per registration, and the
graph mints one organization node per (name string, client number) pair. *Detroit
International Bridge Company* is **eight** nodes across two spellings — 24
communications hang off one, 4 off another, 1 off a third, and zero off the
remaining five. Resolve the name to a single node, read that node's communications,
and you will confidently report 4 when the answer is 29. Nothing errors. Nothing
looks wrong. The number is just incomplete.

**So the rule is: search by substring, and never let a single node be the answer.**
The registry-backed search tools already substring-match, which is exactly why they
are the primary instrument here and node resolution is not. Use the shortest
distinctive fragment of the name — `"Detroit International Bridge"`, not the full
legal name with its suffix — because the suffix is one of the things that varies
between spellings.

## Step 0 — Decide which question you were asked

"Who is lobbying on X" is two different queries depending on what X is.

- **X is a government institution** — a department, agency or portfolio (ISED,
  Health Canada, Transport Canada). The answer is the organizations lobbying *into*
  it. Go to branch A.
- **X is a company, association or coalition.** The answer is what that organization
  lobbies about, whom it retained, and whom it met. Go to branch B.
- **X is a person.** This is the wrong prompt — `graph_whos_lobbying_person()` answers
  it, and the `pre_meeting_brief` prompt runs the whole chain around it.

If it is genuinely unclear, run `graph_resolve_institution(query="{{subject_or_org}}")`
first. It settles the question, because it returns candidates only when the string
names an institution the registry recognises.

## Branch A — {{subject_or_org}} as a government institution

**A1.** `graph_resolve_institution(query="{{subject_or_org}}")`. Institutions are
minted one node per distinct registry string, so a renamed department answers under
both names and they are deliberately **not** merged — `Industry Canada` and
`Innovation, Science and Economic Development Canada (ISED)` are two candidates and
both may carry communications. Report both. If `excluded` is populated, the string
names something the graph refused a node on purpose (`Other (Specify)` is the form's
free-text escape hatch, not a body); that is a different answer from "not found" and
should be reported as such.

**A2.** `tender_whos_lobbying(institution="{{subject_or_org}}", since=..., limit=...)`
returns organizations ranked by reported communications, each with its date range
and most frequent subject matters. `institution` is a fragment, so pass the fragment
that appears in every variant of the department's name.

**A3.** Drill into whatever the ranking surfaces with
`tender_search_lobbying_communications(institution="...", since=..., until=...,
limit=...)`, which returns every DPOH named on each report, its subjects, and a link
to the registry record.

## Branch B — {{subject_or_org}} as an organization

**B1 — the substring sweep, which is the real answer.**
`tender_search_lobbying_communications(organization="<shortest distinctive
fragment>", since=..., limit=...)`. This is a substring match on the client
organization that filed, so it crosses the client-number fragmentation by
construction. Raise `limit` before you conclude anything about volume — a default
page is not a count.

**B2 — what they registered to talk about.**
`tender_get_lobbying_registrations(client_organization="<same fragment>",
active_only=...)` returns each registration version with the institutions it covers
and its declared subject matters. Set `active_only` false when the question is
historical; leave it true for "who is lobbying now".

**B3 — measure the fragmentation, then report it.**
`graph_resolve_org(query="{{subject_or_org}}", kind="lobby_client")`. Use this to see
**how many nodes carry the name**, not to pick one. If several come back, say so:
"the registry holds this client under six registrations across two spellings" is a
finding a client will recognise as expertise, and it tells them why a naive search
elsewhere gave a smaller number. `open_review` names any genuinely contested surface
form. If `ambiguous` is true, house rule 2 applies before you attribute anything to
a specific corporate entity.

**B4 — Ontario, if the file touches Queen's Park.**
`tender_organization_lobbyists(organization="<fragment>", active_only=...,
include_former=...)` names who lobbies the Ontario government for them.

Read `roster_coverage` before concluding anything negative. An empty roster is an
affirmative zero **only** when it says `none_reported`. Ontario did not collect
rosters on Persons & Partnerships registrations before roughly July 2016, so those
come back `not_collected`, which means unknown. Reporting unknown as zero is the
error this field exists to prevent. The same applies per person to
`former_public_offices_source`: only `read` is an answer, while `not_asked` and
`not_recorded` are both unknown.

For the revolving-door question specifically, use
`tender_former_public_office_holders(organization="<fragment>", office=...)`, and
take any rate over that block's `disclosure_read`, never over its total — the gap is
unknown territory, not a clean bill. For one named individual, use
`tender_lobbyist_registrations(name="...")`, and never resolve a `mixed`
former-office tally by taking the most senior entry: the measured filing error runs
upward, so the most senior entry is the likeliest mistake.

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

## The answer

- **Lead with the count and its basis.** "29 reported communications since January,
  filed under six registrations across two spellings of the name." State the window.
  A bare number with no window is not an answer.
- **Group by client and by institution**, not by row. The reader wants the shape.
- **Name the subject matters** the registry itself records; do not paraphrase them
  into your own taxonomy.
- **Link every communication** to its registry record.
- **Say what is federal and what is Ontario**, separately. They are different
  registries with different disclosure rules and merging them into one total is
  wrong even when it looks tidier.
- **Close with what was not searched** — the other jurisdiction, the window you did
  not cover, the spelling you could not confirm.
