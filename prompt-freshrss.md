# Task

Create a weekly set of documentary recommendations.

The recommendations should come from the content available
in FreshRSS. The feeds configured there define the scope.

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

# Data Source

FreshRSS is the single source of truth for all incoming content.

All feeds are configured and managed inside FreshRSS.
This prompt does not contain feed URLs, query syntax,
or any feed configuration. Adding or changing sources
happens exclusively in FreshRSS.

## Fetching Candidates

Use the FreshRSS skill to fetch **1000 unread** items.

Fetch all unread items regardless of category.
Do not filter by category at this stage — curation
intelligence lives in this prompt, not in FreshRSS.

## Important Constraints

- FreshRSS is the **only** data source.
  Do not use web search, browser tools, or any other method
  to find documentaries. All candidates must come from
  unread FreshRSS entries.
- If FreshRSS returns few or no unread results,
  work with what is available. Do not fall back to web search.
- Do not invent titles, descriptions or links.

---

# Read State Management

After completing the run, collect the IDs of every item
returned in the fetch response. Then mark exactly these
items as read via the FreshRSS skill:

```
freshrss.sh mark-read <id1> <id2> <id3> ...
```

Mark all fetched items — not just the recommended ones.
Items that were fetched but not recommended must also
be marked as read so they are not re-evaluated in the
next run.

Only mark items whose IDs appeared in the fetch response.
Do not use a blanket mark-all-as-read operation, as this
could affect items that arrived after the fetch and were
never seen by this run.

This ensures:

- Recommended items are not suggested again
- Rejected items are not re-evaluated in the next run
- Each run works exclusively with fresh, unseen content
- No items are silently skipped without being evaluated

The read state in FreshRSS replaces any file-based
deduplication. There is no need to cross-reference
previous output files.

---

# Candidate Selection

Treat all fetched unread items as the candidate pool.

Remove duplicates (same title appearing multiple times).

Exclude:

- content that does not meet the criteria in "# Documentary Quality"
  (e.g. news segments, magazine clips, talk shows)
- entries containing "Audiodeskription" or "(AD)" in the title,
  unless no non-AD version of the same documentary exists in the pool

Prefer:

- entries with an informative description
- content that matches the human's interests in profile.md

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

Each FreshRSS entry contains a URL.

Use this URL as the recommendation link.

Do not construct or guess URLs manually.

If no URL is present in the entry, omit the link entirely.

Do not use web search to find or discover documentaries.
Do not invent titles, descriptions or links.

---

# Output Language

Write the final recommendations in **German**.

---

# Output Location

Save the result in:

tasks/open-media-curator/

Filename format:

YYYY-MM-DD.md

Example:

tasks/open-media-curator/2026-04-05.md

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
