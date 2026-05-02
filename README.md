# public-media-curator

A self-hosted content curator for German public-service media (e. g. ARD, ZDF or Arte).

There's a lot of great content buried across the various Mediatheken — documentaries, reports, deep dives — but discovering it means clicking through multiple apps and websites. This project fixes that: it downloads the official MediathekView film list, filters it against your interests, and sends you a compact list of recommendations. **Mediathek highlights without the endless search.**

> **Note:** This project is community-driven and not affiliated with any public broadcaster.

## What This Is

This is an **[OpenClaw](https://openclaw.ai) skill** — a self-contained pipeline definition that runs inside OpenClaw and delivers recommendations via the configured output channel. Support for other agentic frameworks is planned.

## How It Works

```
MediathekView (Filmliste)  →  Python Parser  →  OpenClaw  →  Telegram
      download                  filter            curate       deliver
```

1. The official [MediathekView](https://mediathekview.de/) film list is downloaded directly — a compressed snapshot of all available content across German public broadcasters.
2. A Python script unpacks and parses the list, filters by channel (ARD, ZDF, ARTE) and date (last 7 days), and outputs clean JSON.
3. OpenClaw reads the JSON, matches it against your interest profile, and picks the best recommendations.
4. Results are delivered to your phone via Telegram.

### Why This Setup

* **No external services.** No FreshRSS, no RSS feeds, no API tokens. Just a direct download and a Python script.
* **Stateless by design.** Each run is fully independent. No read-state management, no persistent containers, no deduplication overhead.
* **Token-efficient.** The parser outputs only the six fields the LLM actually needs — keeping context usage low even with large candidate pools.

## Repository Structure

| File | Purpose |
|---|---|
| `SKILL.md` | OpenClaw skill definition. Documents the full pipeline, triggered via `/public_media_curator`. |
| `format.md` | Output template for Telegram messages. Referenced by the skill. |
| `profile.template.md` | Template interest profile. Copy to `profile.md` and personalize. |
| `scripts/parse_filmliste.py` | Downloads and parses the MediathekView film list. |

The skill loads two files at runtime: `profile.md` (your interests) and `format.md` (the output format). You only need to customize `profile.md` — everything else works out of the box.

## Prerequisites

* A running [OpenClaw](https://openclaw.ai) instance with Python 3 available in the container
* A configured output channel connected to OpenClaw
* The film list added to `.gitignore` (it's ~70 MB and should not be committed):
  ```
  Filmliste-akt.xz
  ```

## Usage

Install the skill and run it on demand:

```bash
openclaw skills install public-media-curator
```

```
/public_media_curator
```

Each run will:

1. Download the current MediathekView film list
2. Parse and filter it by channel and date
3. Match against `profile.md`
4. Send recommendations via Telegram

## Personalizing Your Profile

Copy the template profile and edit it to describe what you're looking for:

```bash
cp profile.template.md profile.md
```

OpenClaw uses this profile to score and filter candidates. Adjust it anytime — no restart needed.

## Troubleshooting

**Python not found in container**
Verify with `docker exec -it openclaw python3 --version`. If missing, install via `apt install python3` inside the container.

**No results from the parser**
Check that the download succeeded and the file is not corrupted. Re-run the parser manually with `--limit 10` to verify output.

## Models

This skill works with any LLM supported by OpenClaw. Larger cloud models (e.g. from OpenAI, Anthropic, or Google) tend to produce the most reliable results when filtering and ranking against a nuanced interest profile.

## Background

This started as a personal project to stop wasting time browsing through Mediatheken to find interesting documentaries.

The generic framework behind this is topic-agnostic — it works with any structured content source. This repo is the concrete instance for German public-service media.

## License

MIT