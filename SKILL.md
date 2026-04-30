---
name: public_media_curator
description: On-demand German public-media documentary picks filtered against a personal profile, sent via Telegram
metadata:
  openclaw:
    os: ["linux"]
    requires:
      bins: ["curl", "python3"]
    slash_commands:
      - name: givemedocs
        description: Generate documentary recommendations from MediathekView right now
---

# public_media_curator

## Trigger

User runs `/givemedocs`.

## Preconditions

Before running, verify:

- `profile.md` exists in the workspace root. If missing, halt and instruct the user to copy `profile.example.md` to `profile.md` and personalize it.
- `format.md` exists in the workspace root. If missing, halt — there is no fallback.
- Telegram is configured in OpenClaw Settings (bot token + chat ID).

## Steps

1. Download the MediathekView film list:
   ```bash
   curl -O https://liste.mediathekview.de/Filmliste-akt.xz
   ```
2. Parse and filter by channel and date:
   ```bash
   python3 scripts/parse_filmliste.py Filmliste-akt.xz --limit 1337
   ```
3. Read `profile.md` (user interests) and `format.md` (output template).
4. Deduplicate entries. Exclude non-documentary content: news clips, magazine segments, trailers, talk shows.
5. Select 4 recommendations:
   - 3 aligned with the interests in `profile.md`
   - 1 exploratory pick outside the user's usual interests
6. Write the output in German using the template defined in `format.md`.
7. Send via Telegram through the configured OpenClaw channel.

## Error Handling

| Situation | Action |
|---|---|
| `python3` not found | Instruct: `docker exec -it openclaw apt install python3` |
| Download fails | Retry once. If still failing, abort and report the error. |
| Parser returns empty JSON | Report no results. Do not fall back to web search or invent entries. |
| `profile.md` missing | Halt. Instruct user to copy `profile.example.md` → `profile.md`. |
| `format.md` missing | Halt. No fallback exists. |
| Telegram delivery fails | Check that the OpenClaw job is configured as an isolated job with the channel ID set. |
