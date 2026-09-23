# Local Events Finder

Finds a few local events you would actually turn up to, picked from your own
calendar rather than a questionnaire.

## The point

Discovery is solved. Partiful, Luma and Eventbrite publish more events than
anyone can read, and nobody is short of things to do because they could not
find a list. The drop-off is later: people RSVP to three things in a month and
attend none.

So this is built around whether you **go**, not whether you **find**:

- **It reads what you already do** instead of asking. Words that recur across
  your calendar become interest keywords. Subscribed calendars count too, and
  are often the strongest signal you have — nobody thinks to type "I follow the
  Japan Society calendar" into an interests form.
- **Three events, not thirty.** Five maybes on a Thursday means you attend zero.
- **Repeating beats one-off.** A weekly walk club outranks a one-time gala even
  when the gala matches your interests better, because seeing the same people
  again is the actual point.
- **It refuses to suggest anything unconnected to you.** Without that rule the
  top picks are always tech meetups — they are reliably free, small and
  recurring, so they win on structure alone.

It also drops anything sold out, too far away, clashing with something already
in your calendar, or that you turned down once before.

## Running it

```bash
cd scripts
python find_events.py --region sf
```

| Flag | |
|---|---|
| `--region` | `sf`, `nyc`, `la`, `dc` (default `sf`) |
| `--window-days` | how far ahead to look (default 10) |
| `--max-distance-miles` | default 12; only applies if you give a home coordinate |
| `--count` | how many to surface (default 3) |
| `--home-latitude` / `--home-longitude` | turns on the distance filter |
| `--json` | full output, including what was rejected and why |

Example output:

```
Stylized Figure Drawing: 2 Day Workshop
  Sun Oct 4, 11:00am | Inner Mission
  https://partiful.com/e/HMfkVLDffK4PK9GR5pV6
  - Matches what you already do: market
  - Small enough room that you would actually get talked to
  - About 1 miles away
```

## What it needs

Read access to your Google Calendar, reached through `latchkey`. Nothing else
— no payment, no other accounts. Both event sources are read without any login.

## Where the data comes from

| Source | Covers | Role |
|---|---|---|
| [Partiful](https://partiful.com/explore) | `sf`, `nyc`, `la`, `dc` | Primary. Its feeds are explicitly social — walk clubs, craft nights, book clubs |
| [Luma](https://lu.ma) | most major cities | Secondary. Heavily skewed to tech meetups; in SF roughly 3 in 4 listings are pitch nights |

Only public listings are read. Every response is kept on disk under
`data/.skills/local-events-finder/`, so changing how results are processed never
needs a refetch.

## Honest limits

- **Partiful covers four cities.** Outside them only Luma answers, and Luma
  alone is too professional a mix to serve this purpose.
- **Matching is shallow.** It is word overlap. It connects "coffee" to a coffee
  festival, but it cannot connect a weekly language lesson booked under a
  tutor's name to a cultural event in that language. Closing that gap means
  putting a language model in the matching step, at a per-run cost.
- **Inference is past-tense.** Mining a calendar recommends more of what you
  already do, which for a lonely person is how they got there. Worth reserving
  one slot for something outside the pattern.
- **Cold start is the real weakness.** Someone new to a city has almost no
  calendar to read, and is exactly who this is for.
- **Motivation is the bottleneck and software may not move it.** No
  recommendation makes anyone leave the house. The honest test is attendance
  over several weeks, not how good the suggestions look.

`references/why-this-exists.md` has the full reasoning, including five things
that broke the first time this met a real calendar.

## Tests

```bash
cd scripts
pytest .
```

42 tests, all offline. Do not add tests that hit the live sources — their
inventory changes hourly, so any assertion about what is on this week is flaky
by construction.
