# public-media-curator

A self-hosted content curator for German public-service media (ARD, ZDF, Arte).

There's a lot of great content buried across the various Mediatheken — documentaries, reports, deep dives — but discovering it means clicking through multiple apps and websites. This project fixes that: it downloads the official MediathekView film list, filters it against your interests, and sends you a compact list of recommendations. **Mediathek highlights without the endless search.**

> **Note:** This project is community-driven and not affiliated with any public broadcaster.

## How It Works

```
MediathekView (Filmliste)  →  Python Parser  →  OpenClaw  →  Telegram
      download                  filter            curate       deliver
```

1. The official [MediathekView](https://mediathekview.de/) film list is downloaded directly — a compressed snapshot of all available content across German public broadcasters.
2. A Python script unpacks and parses the list, filters by channel (ARD, ZDF, ARTE) and date (last 7 days), and outputs clean JSON.
3. [OpenClaw](https://openclaw.com/) reads the JSON, matches it against your interest profile, and picks the best recommendations.
4. Results are delivered to your phone via Telegram.

### Why This Setup

* **No external services.** No FreshRSS, no RSS feeds, no API tokens. Just a direct download and a Python script.
* **Stateless by design.** Each run is fully independent. No read-state management, no persistent containers, no deduplication overhead.
* **Token-efficient.** The parser outputs only the six fields the LLM actually needs — keeping context usage low even with large candidate pools.

## Repository Structure

| File | Purpose |
|---|---|
| `SKILL.md` | OpenClaw skill definition. Registers `/givemedocs` and documents the pipeline. |
| `TODO.md` | The system prompt for OpenClaw. This is the core of the curation logic. |
| `format.md` | Output template for Telegram messages. Referenced by the prompt. |
| `profile.example.md` | Example interest profile. Copy to `profile.md` and personalize. |
| `scripts/parse_filmliste.py` | Downloads and parses the MediathekView film list. |

The prompt tells OpenClaw to run the parser, then load two files at runtime: `profile.md` (your interests) and `format.md` (the output format). You only need to customize `profile.md` — everything else works out of the box.

## What You Need

* A home server or VPS (this guide assumes [UmbrelOS](https://umbrel.com/))
* [OpenClaw](https://openclaw.com/) with Python 3 available in the container
* A Telegram Bot

## Setup

### 1. Set Up OpenClaw

Install OpenClaw and make sure Python 3 is available in the container:

```bash
docker exec -it openclaw python3 --version
```

### 2. Configure Your Interest Profile

Copy the example profile and personalize it:

```bash
cp profile.example.md profile.md
```

Edit `profile.md` to describe what you're looking for. OpenClaw uses this profile to score and filter candidates. Adjust it anytime — no restart needed.

### 3. Add the Film List to .gitignore

The downloaded film list is ~70 MB and should not be committed:

```
Filmliste-akt.xz
```

### 4. Configure Telegram

Follow the official OpenClaw documentation:
https://docs.openclaw.ai/channels/telegram

### 5. Install

```bash
openclaw skills install public-media-curator
```

### Usage

Run on demand in OpenClaw:

```
/givemedocs
```

Each run will:

1. Download the current MediathekView film list
2. Parse and filter it by channel and date
3. Match against `profile.md`
4. Send recommendations via Telegram

## Troubleshooting

**Python not found in container**
Verify with `docker exec -it openclaw python3 --version`. If missing, install via `apt install python3` inside the container.

**No results from the parser**
Check that the download succeeded and the file is not corrupted. Re-run the parser manually with `--limit 10` to verify output.

## Tested Models

This project has been tested with **GPT-5.4** (OpenAI) via OpenClaw. GPT-5.4 handles the full pipeline reliably — parsing 1000 items, filtering against the profile, and formatting the output. Context usage stays well below 25% with the JSON output format.

If you test with other models, feel free to open an issue with your results.

## Background

This started as a personal project to stop wasting time browsing through Mediatheken to find interesting documentaries.

The generic framework behind this is topic-agnostic — it works with any structured content source. This repo is the concrete instance for German public-service media.

## License

MIT