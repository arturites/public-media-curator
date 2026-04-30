---
name: public_media_curator
description: On-demand German public-media documentary picks filtered against a personal profile, sent via Telegram
metadata:
  openclaw:
    os: ["linux"]
    requires:
      bins: ["curl", "python3"]
---

# public_media_curator

## Preconditions

Before running, verify:

- `profile.md` exists in the workspace root. If missing, halt and instruct the user to copy `profile.example.md` to `profile.md` and personalize it.
- `format.md` exists in the workspace root. If missing, halt — there is no fallback.
- Telegram is configured in OpenClaw Settings (bot token + chat ID).

## Data Source

Run the following commands to generate the input JSON:

```bash
curl -O https://liste.mediathekview.de/Filmliste-akt.xz
python3 scripts/parse_filmliste.py Filmliste-akt.xz --limit 1337
```

The output is passed directly into the prompt. Each entry contains:

- `title` — title of the content
- `channel` — broadcaster (ARD, ZDF or ARTE)
- `date` — broadcast date
- `duration` — duration of the content
- `description` — description of the content
- `website` — link to the media library page

This JSON is the single source of truth. Do not use web search, browser tools, or any other method to find documentaries. Do not invent titles, descriptions, or links.

## Inputs

Read the following files before proceeding:

- `profile.md` — the user's interests and preferred themes. Use this to understand what topics to prioritize.
- `format.md` — the output template and extraction rules. Use this to format the final recommendations.

## Candidate Selection

Treat all entries in the input JSON as the candidate pool.

Remove duplicates (same title appearing multiple times).

Exclude:
- news segments
- magazine clips
- trailers
- talk shows
- purely promotional content

Prefer:
- entries with an informative description
- content that matches the user's interests in `profile.md`
- investigative, scientific, historical, philosophical, or cultural documentaries
- full documentary productions

## Recommendations

Select **4 recommendations**:

- 3 aligned with the user's interests in `profile.md`
- 1 exploratory pick outside the user's usual interests to encourage discovery

The exploratory pick should still be intellectually interesting, visually impressive, or culturally valuable. Avoid trivial entertainment-only content.

All recommendations must be:
- thoughtful and informative
- linked to the official media library page via the `website` field
- currently streamable if possible

Prefer documentaries released within the last 5 years. Older documentaries may be recommended if particularly insightful or relevant.

## Verification

Use the `website` field from each entry as the recommendation link. Do not construct or guess URLs. If no URL is present, omit the link entirely.

## Output

- Write the final recommendations in **German**
- Use the template defined in `format.md`
- Send via Telegram through the configured OpenClaw channel

## Error Handling

| Situation | Action |
|---|---|
| `python3` not found | Instruct: `docker exec -it openclaw apt install python3` |
| Download fails | Retry once. If still failing, abort and report the error. |
| Parser returns empty JSON | Report no results. Do not fall back to web search or invent entries. |
| `profile.md` missing | Halt. Instruct user to copy `profile.example.md` → `profile.md`. |
| `format.md` missing | Halt. No fallback exists. |
| Telegram delivery fails | Check that the OpenClaw Telegram channel is configured with a valid bot token and chat ID. |