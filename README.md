# public-media-curator

A self-hosted content curator for German public-service media (ARD, ZDF, Arte, 3sat, Phoenix, …).

There's a lot of great content buried across the various Mediatheken — documentaries, reports, deep dives — but discovering it means clicking through multiple apps and websites. This project fixes that: it aggregates everything into one feed, filters it against your interests, and sends you a compact list of recommendations. **Mediathek highlights without the endless search.**

> **Note:** This project is community-driven and not affiliated with any public broadcaster.

## How It Works

```
MediathekViewWeb (RSS)  →  FreshRSS  →  OpenClaw  →  Telegram
         feed               aggregate     curate       deliver
```

1. [MediathekViewWeb](https://mediathekviewweb.de/) publishes an RSS feed of all new items across German public broadcasters.
2. [FreshRSS](https://freshrss.org/) aggregates and deduplicates the feed. It acts as the single source of truth — no local databases, no file-based state.
3. [OpenClaw](https://openclaw.com/) reads unread items from FreshRSS, matches them against your interest profile, and picks the best recommendations.
4. Results are delivered to your phone via Telegram.

### Why This Setup

* **No pre-filtering in feeds.** The full MediathekViewWeb feed goes into FreshRSS unfiltered. All curation intelligence lives in the OpenClaw prompt — not in feed queries or filter rules.
* **Self-managing read state.** FreshRSS tracks what has already been recommended. Recommended items get marked as read, so the next run only sees new content.

## Repository Structure

| File | Purpose |
|---|---|
| `TODO.md` | The system prompt for OpenClaw. This is the core of the curation logic. |
| `format.md` | Output template for Telegram messages. Referenced by the prompt (already embedded as an input). |
| `profile.example.md` | Example interest profile. Copy to `profile.md` and personalize. |

The prompt tells OpenClaw to load two files at runtime: `profile.md` (your interests) and `format.md` (the output format). You only need to customize `profile.md` — everything else works out of the box.

## What You Need

* A home server or VPS (this guide assumes [UmbrelOS](https://umbrel.com/))
* FreshRSS (available in the UmbrelOS App Store)
* [OpenClaw](https://openclaw.com/)
* A Telegram Bot ([setup instructions below](#5-create-a-telegram-bot))

## Setup

### 1. Install FreshRSS

On UmbrelOS: App Store → search "FreshRSS" → install. Note the assigned port.

### 2. Add the MediathekViewWeb Feed

In FreshRSS: **Subscription Management** → **Add a feed**:

```
https://mediathekviewweb.de/feed?future=false
```

Assign the category `Dokus`. FreshRSS will start collecting items automatically (default polling interval: 1 hour).

### 3. Create an API User in FreshRSS

Keep API access separate from your admin account:

1. **Administration** → **User Management** → create user `openclaw` (role: User)
2. **Administration** → **System Configuration** → enable **API access**
3. Log in as `openclaw` → **Profile** → **API Management** → set an API password (avoid special characters)

Verify:

```bash
# Check API endpoint
curl -s "http://<freshrss-ip>:<port>/api/"

# Authenticate (must be POST)
curl -s -X POST "http://<freshrss-ip>:<port>/api/greader.php/accounts/ClientLogin" \
  -d "Email=openclaw&Passwd=<API-PASSWORD>"
```

A successful response returns `Auth=openclaw/<token>`.

> **UmbrelOS note:** `umbrel.local` doesn't resolve inside Docker containers. Use the host's local IP (e.g. `192.168.x.x`).

### 4. Set Up OpenClaw

Install the FreshRSS skill:

```bash
openclaw skills install freshrss-reader
```

#### Required Fixes

**Fix authentication method** — the installed skill uses GET, but FreshRSS requires POST. In `scripts/freshrss.sh`, replace:

```bash
# Before
RESPONSE=$(curl -s "${API_BASE}/accounts/ClientLogin?Email=${FRESHRSS_USER}&Passwd=${FRESHRSS_API_PASSWORD}")

# After
RESPONSE=$(curl -s -X POST "${API_BASE}/accounts/ClientLogin" -d "Email=${FRESHRSS_USER}&Passwd=${FRESHRSS_API_PASSWORD}")
```

**Fix credential loading** — create `.env` in the skill root:

```
FRESHRSS_URL=http://<freshrss-ip>:<port>
FRESHRSS_USER=openclaw
FRESHRSS_API_PASSWORD=<your-api-password>
```

Add to `scripts/freshrss.sh` after `set -e`:

```bash
SCRIPT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
[ -f "$SCRIPT_DIR/.env" ] && source "$SCRIPT_DIR/.env"
```

**Install jq** (required for JSON parsing):

```bash
brew install jq  # persists on UmbrelOS via /home/linuxbrew volume
```

#### Verify

```bash
/freshrss-reader categories        # should list "Dokus"
/freshrss-reader headlines --category "Dokus" --unread --count 10
```

### 5. Create a Telegram Bot

1. Open Telegram and search for [@BotFather](https://t.me/BotFather).
2. Send `/newbot` and follow the prompts to choose a name and username.
3. BotFather will reply with your **Bot Token** — save it.
4. To get your **Chat ID**: send any message to your new bot, then open `https://api.telegram.org/bot<YOUR-TOKEN>/getUpdates` in a browser. Your Chat ID is in the `chat.id` field.
5. In OpenClaw, go to **Settings** and add the Bot Token and Chat ID in the Telegram configuration.

### 6. Configure Your Interest Profile

Copy the example profile and personalize it:

```bash
cp profile.example.md profile.md
```

Edit `profile.md` to describe what you're looking for. OpenClaw uses this profile to score and filter items from FreshRSS. Adjust it anytime — no restart needed.

### 7. Schedule It

Set up a cron job in OpenClaw to run the curation on a schedule. The job must be configured as an **isolated job** — otherwise Telegram delivery will fail. Make sure to specify the **channel ID** in the job configuration.

Use the following prompt for the cron job:

```
"Read the file <your-workspace-path>/public-media-curator/TODO.md, follow all instructions contained in it, and execute the task completely."
```

Replace `<your-workspace-path>` with the actual path to your OpenClaw workspace.

Example schedule: every Monday at 18:00 Europe/Berlin.

For details on how to create and configure cron jobs, see the [OpenClaw Scheduled Tasks documentation](https://docs.openclaw.ai/automation/cron-jobs).

Each run will:

1. Fetch unread items from FreshRSS
2. Match against `profile.md`
3. Send recommendations via Telegram
4. Mark all fetched items as read in FreshRSS

## Troubleshooting

**FreshRSS authentication fails**
Make sure you're using the API password (set under Profile → API Management), not the login password. Also verify the request uses POST, not GET.

**`umbrel.local` doesn't resolve**
Inside Docker containers, mDNS names don't work. Use the host's local IP instead (e.g. `192.168.1.x`).

**`jq: command not found`**
Install jq with `brew install jq`. On UmbrelOS, Homebrew installs persist across reboots via the `/home/linuxbrew` volume.

**No results from FreshRSS**
Check that the feed is active and collecting items: log in to FreshRSS and verify unread items exist. New feeds may take up to an hour for the first poll.

## Tested Models

This project has been tested with **GPT-5.4** (OpenAI) via OpenClaw. GPT-5.4 handles the full pipeline reliably — fetching 1000 items from FreshRSS, filtering against the profile, and formatting the output.

**GPT-5.4-mini** does not work well for this use case: the combined input (1000 feed items + prompt + profile + format template) exceeds its context window limits.

If you test with other models, feel free to open an issue with your results.

## Background

This started as a personal project to stop wasting time browsing through Mediatheken to find interesting documentaries.

The generic framework behind this is topic-agnostic — it works with any RSS source. This repo is the concrete instance for German public-service media.

## License

MIT

