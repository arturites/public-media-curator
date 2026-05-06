# public-media-curator

A self-hosted content curator for German public-service media (e. g. ARD, ZDF or Arte).

There's a lot of great content buried across the various Mediatheken — documentaries, reports, deep dives — but discovering it means clicking through multiple apps and websites. This project fixes that: it downloads the official MediathekView film list, filters it against your interests, and sends you a compact list of recommendations. **Mediathek highlights without the endless search.**

> **Note:** This project is community-driven and not affiliated with any public broadcaster.

## What This Is

This is an **[OpenClaw](https://openclaw.ai) skill** — a self-contained pipeline definition that runs inside OpenClaw and delivers recommendations via the configured output channel. Support for other agentic frameworks is planned.

## How It Works

```
MediathekView (Filmliste)  →  Python Parser  →  OpenClaw  →  Output Channel
      download                  filter            curate         deliver
```

1. The official [MediathekView](https://mediathekview.de/) film list is downloaded directly — a compressed snapshot of all available content across German public broadcasters.
2. A Python script unpacks and parses the list, filters by channel (ARD, ZDF, ARTE) and date (last 7 days), and outputs clean JSON.
3. OpenClaw reads the JSON, matches it against your interest profile, and picks the best recommendations.
4. Results are delivered via your configured OpenClaw output channel.

## Prerequisites

* A running [OpenClaw](https://openclaw.ai) instance with Python 3 available in the container
* A configured output channel connected to OpenClaw

## Usage

```bash
openclaw skills install public-media-curator
```

```
/public_media_curator
```

## Models

This skill works with any LLM supported by OpenClaw. Larger cloud models (e.g. from OpenAI, Anthropic, or Google) tend to produce the most reliable results when filtering and ranking against a nuanced interest profile.

## Background

This started as a personal project to stop wasting time browsing through Mediatheken to find interesting documentaries.

The generic framework behind this is topic-agnostic — it works with any structured content source. This repo is the concrete instance for German public-service media.

## License

MIT