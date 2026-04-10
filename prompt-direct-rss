# Task

Create a weekly set of documentary recommendations.

The recommendations should come from public media libraries
such as ARD, ZDF, ARTE or similar public broadcasters.

Focus on high-quality, informative and meaningful documentaries.

---

# Inputs

Use the following files:

profile.md

Contains the human's interests and preferred themes.

Use this file to understand:

- what topics the human enjoys
- what themes are worth exploring further

format.md

Contains the output template and extraction rules.

Use this file to format the final recommendations.

---

# Previous Recommendations

Before searching for new documentaries, read all files in:

weekly/

These files contain previously recommended documentaries.

Extract the documentary titles from these files and treat them
as already recommended content.

Do not recommend the same documentary again.

If a documentary appears to be part of a series, avoid recommending
the exact same episode again, but other episodes may still be allowed
if they clearly provide different content.

If a previously recommended documentary is no longer available
in the media library, it should still be treated as already
recommended and must not be suggested again.

---

# Data Source

## Broadcaster Scope

Use the following public broadcasters as data sources:

- ARD
- ZDF
- ARTE.DE

## Feed Generation

Generate RSS feed URLs dynamically on every run.

Read `profile.md` and derive search queries that match
the human's interests. Then construct MediathekViewWeb
RSS feed URLs using those queries.

Base URL format:

https://mediathekviewweb.de/feed?query=<QUERY>&future=false

The `query` parameter uses MediathekViewWeb search syntax.
The `&future=false` parameter excludes future entries.

Each feed returns up to 50 results sorted by date.

## Search Syntax Reference

MediathekViewWeb supports the following selectors
inside the `query` parameter:

| Selector | Field            | Example       |
|----------|------------------|---------------|
| `!`      | Sender (channel) | `!ARD`        |
| `#`      | Thema (topic)    | `#Doku`       |
| `+`      | Titel (title)    | `+Klimawandel`|
| `*`      | Beschreibung     | `*Ozean`      |
| `>`      | Länger als (min) | `>42`         |
| `<`      | Kürzer als (min) | `<90`         |
| (none)   | Thema + Titel    | `Universum`   |

Rules:

- Same selector used multiple times → OR logic.
  Example: `!ARD !ZDF` → ARD or ZDF.
- Different selectors → AND logic.
  Example: `!ARD #Doku` → channel is ARD and topic contains Doku.
- Comma inside a selector value → AND logic.
  Example: `#Klima,Wandel` → topic contains Klima and Wandel.
- Search is case-insensitive.
- Spaces in the URL must be percent-encoded as `%20`.

## Feed Strategy

Always include **one broad feed per broadcaster** as a baseline:

https://mediathekviewweb.de/feed?query=!ARD%20%3E42&future=false
https://mediathekviewweb.de/feed?query=!ZDF%20%3E42&future=false
https://mediathekviewweb.de/feed?query=!ARTE.DE%20%3E42&future=false

These three feeds ensure a wide pool of candidates
with a minimum duration of 42 minutes.

In addition, generate **topic-specific feeds** based on
the human's interests from `profile.md`.

Use the selectors `#` (topic) and `+` (title) to create
targeted queries. Combine them with `!` (channel) and
`>42` (minimum duration) as needed.

## Query Creativity

Do **not** copy search terms directly from `profile.md`.

Instead, read `profile.md`, understand the underlying interests,
and derive your own search queries — including synonyms,
related terms, broader categories, and adjacent topics
that the human might not have listed explicitly.

Example: If `profile.md` mentions "Künstliche Intelligenz",
do not just search for `#Künstliche,Intelligenz`.
Also consider: `#Algorithmen`, `#Digitalisierung`,
`#Roboter`, `#Automatisierung`, `+KI`, `+Machine,Learning`.

Example: If `profile.md` mentions "Geopolitik",
also consider: `#Außenpolitik`, `#Diplomatie`,
`#Konflikte`, `#Weltordnung`, `+NATO`, `+Sicherheitspolitik`.

Vary your queries between runs. The goal is to surface
documentaries that the human would not find on their own.

Examples of dynamically generated feeds:

- Interest "Wissenschaft & Forschung":
  `!ARD !ZDF !ARTE.DE #Forschung >42`
  → `https://mediathekviewweb.de/feed?query=!ARD%20!ZDF%20!ARTE.DE%20%23Forschung%20%3E42&future=false`

- Derived alternative for the same interest:
  `!ARD !ZDF !ARTE.DE #Experiment >42`
  → `https://mediathekviewweb.de/feed?query=!ARD%20!ZDF%20!ARTE.DE%20%23Experiment%20%3E42&future=false`

Generate as many topic-specific feeds as needed to cover
the human's interests well.
Keep queries focused — narrow beats broad.

If a feed is unreachable, skip it and continue
with the remaining feeds.

## Important Constraints

- The RSS feeds described above are the **only** data source.
  Do not use web search, browser tools, or any other method
  to find documentaries. All candidates must come from the
  RSS feed results.
- If the RSS feeds return few or no relevant results,
  work with what is available. Do not fall back to web search.
- Do not hardcode topic-specific queries in this file.
  Always derive them from `profile.md` at runtime.
- The three broad baseline feeds above are the only
  fixed URLs. Everything else is generated.
- URL-encode all special characters in the query parameter.
  In particular: `#` → `%23`, `>` → `%3E`, `<` → `%3C`,
  space → `%20`, `!` can remain unencoded or be encoded as `%21`.

---

# Candidate Selection

Merge all RSS feed results into a single candidate pool.

Remove duplicates (same title appearing in multiple feeds).

## Duplicate Detection

Before finalizing recommendations, compare each candidate
against **all** previously recommended titles from weekly/.

A candidate is a duplicate if:
- its title exactly matches a previous recommendation
- its title is a minor variation of a previous recommendation
  (e.g. with or without episode number, subtitle, or punctuation)
- it links to the same media library URL as a previous recommendation

After selecting the final 4 recommendations, verify once more
that no two of them are the same documentary. If they are,
replace the duplicate with the next best candidate.

Exclude:
- entries that match a previously recommended documentary (see above)
- content that does not meet the criteria in "# Documentary Quality"
  (e.g. news segments, magazine clips, talk shows)
- entries containing "Audiodeskription" or "(AD)" in the title,
  unless no non-AD version of the same documentary exists in the pool

Prefer:
- entries with an informative description
- content that matches the human's interests in profile.md
- entries returned by topic-specific feeds over broad baseline feeds

---

# Documentary Quality

Prefer documentaries that are:

- investigative
- scientific
- historical
- philosophical
- cultural
- produced as full documentary productions

Avoid:

- short news segments
- magazine clips
- trailers
- talk shows
- purely promotional content

---

# Recommendations

Create **4 recommendations**:

3 recommendations aligned with the human's interests.

1 recommendation outside the human's usual interests
to encourage discovery and broaden perspective.

The exploration recommendation should still be:

- intellectually interesting
- visually impressive
- culturally valuable

Avoid trivial entertainment-only content.

All recommendations must be:

- thoughtful
- informative
- currently streamable if possible
- linked to the official media library page

Prefer documentaries released within the last 5 years
when possible.

Older documentaries may still be recommended
if they are particularly insightful or relevant.

---

# Verification

Each RSS entry from MediathekViewWeb contains two URLs:

1. A direct MP4 or CDN link (e.g. akamaihd.net) — this is the stream URL.
2. A final URL at the end of the entry — this is the official media library page.

Always use the **last URL** in the RSS entry as the recommendation link.
This URL points directly to the official media library page
(e.g. ardmediathek.de, zdf.de, arte.tv, phoenix.de).

Do not use the MP4 or CDN link as the recommendation link.
Do not construct or guess media library URLs manually.

If no final URL is present in the RSS entry, omit the link entirely.

Only use the browser tool if the duration or availability date
of a candidate cannot be determined from the RSS entry itself.

Do not use web search to find or discover documentaries.
Do not invent titles, descriptions or links.

---

# Output Language

Write the final recommendations in **German**.

---

# Output Location

Save the result in:

tasks/public-media-curator/weekly/

Filename format:

YYYY-MM-DD.md

Example:

tasks/public-media-curator/weekly/2026-03-11.md

---

# Chat Output

After saving the file, also display the full content
directly in the chat.

The chat output must be identical to the saved file
so the human can immediately read the recommendations.

---

# Output Format

Use the template defined in `format.md`.

Follow the extraction rules described in that file.
