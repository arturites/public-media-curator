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

- **No pre-filtering in feeds.** The full MediathekViewWeb feed goes into FreshRSS unfiltered. All curation intelligence lives in the OpenClaw prompt — not in feed queries or filter rules.
- **Self-managing read state.** FreshRSS tracks what has already been recommended. Recommended items get marked as read, so the next run only sees new content.

## What You Need

- A home server or VPS (this guide assumes [UmbrelOS](https://umbrel.com/))
- FreshRSS (available in the UmbrelOS App Store)
- [OpenClaw](https://openclaw.com/)
- A Telegram Bot (Bot Token + Chat ID)

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

### 5. Configure Your Interest Profile

Create a `profile.md` that tells OpenClaw what you're looking for. This is the only file you need to personalize. Example:

```markdown
## Interests

- AI and machine learning (practical applications, not hype)
- Investigative journalism and political deep dives
- Science documentaries (physics, biology, space)
- History (20th century, Cold War, German reunification)

## Avoid

- Daily news recaps
- Talk shows
- Sports
```

OpenClaw uses this profile to score and filter items from FreshRSS. Adjust it anytime — no restart needed.

### 6. Schedule It

Set up a cron job in OpenClaw to run the curation (e.g. every Monday at 18:00 Europe/Berlin). OpenClaw will:

1. Fetch unread items from FreshRSS (category `Dokus`)
2. Match against `profile.md`
3. Send recommendations via Telegram
4. Mark recommended items as read in FreshRSS

## Background

This started as a personal project to stop wasting time browsing through Mediatheken to find interesting documentaries.

The generic framework behind this — topic-agnostic, works with any RSS source. This repo is the concrete instance for German public-service media.

## License

MIT
