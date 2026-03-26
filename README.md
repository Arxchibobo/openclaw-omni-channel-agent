# openclaw-omni-channel-agent

An omni-channel trend intelligence pipeline for [OpenClaw](https://openclaw.ai), monitoring social media, SEO, and advertising channels to surface emerging content trends and actionable insights.

## Overview

This skill aggregates trend signals across three data channels:

- **Social** — KOL radar, TikTok, Instagram, YouTube, Reddit, and Google Trends
- **SEO** — Semrush keyword research across 11 competitor domains, deduplicated against your existing sitemap and Notion content database
- **Ads** — Facebook Ads Library intelligence across four query scenarios

It is designed to run as a scheduled or on-demand pipeline, outputting Slack-formatted reports and structured JSON files for downstream use.

## Features

- **KOL Radar** — Monitors 11 curated accounts across three tiers (Trend Leaders, Fast Adopters, Professional Radars). Aggregates audio usage signals: when 2+ KOLs use the same music, it flags a trend.
- **AI Fitness Scoring** — Scores each KOL post 0–100 based on transmission power, engagement rate, AI-related keywords, audio signals, and share rate.
- **Multi-source social scraping** — TikTok, Instagram, YouTube, and Reddit via Apify; Google Trends via `pytrends` (free).
- **SEO gap analysis** — Filters Semrush keywords by volume (≥500) and keyword difficulty (≤65), checks coverage against your sitemap and Notion bot database.
- **Facebook Ads intelligence** — Extracts creative formats, active ads, and multi-platform usage across four predefined query scenarios.
- **Slack-ready output** — Reports formatted with localized number display (万/亿) for team distribution.
- **Flexible execution** — Run all channels together or each independently; supports region selection (US/EU/ASIA) and test mode.

## Requirements

**Python**: 3.8+

**Environment variables:**

| Variable | Required | Purpose |
|---|---|---|
| `APIFY_TOKEN` | Yes | Apify API token (all social scraping) |
| `SEMRUSH_API_KEY` | SEO channel | Semrush keyword API |
| `NOTION_IMAGE_BOT_TOKEN` | Recommended | Notion dedup for SEO |
| `TWITTER_TOKEN` | Optional | Twitter API v2 Bearer token |

**Python dependencies:**

- `pytrends` — auto-installed by `google_trends.py` if missing
- All other integrations use Python stdlib (`urllib`, `json`)

**Apify actors used:**

| Platform | Actor |
|---|---|
| TikTok | `clockworks/tiktok-scraper` |
| Instagram | `apify/instagram-scraper` |
| YouTube | `streamers/youtube-scraper` |
| Reddit | `trudax/reddit-scraper-lite` |
| Facebook Ads | `apify/facebook-ads-scraper` |

## Installation

```bash
git clone https://github.com/Arxchibobo/openclaw-omni-channel-agent.git
cd openclaw-omni-channel-agent

# Set required environment variables
export APIFY_TOKEN=your_token
export SEMRUSH_API_KEY=your_key
export NOTION_IMAGE_BOT_TOKEN=your_token
```

No `pip install` step is required for the core pipeline. `pytrends` is installed automatically on first use of the Google Trends source.

## Usage

### Full pipeline (all three channels)

```bash
python3 run_pipeline.py --query "ai filter"
```

### Single channel

```bash
python3 run_pipeline.py --channel social --query "ai filter"
python3 run_pipeline.py --channel seo
python3 run_pipeline.py --channel ads --query "ai photo generator"
```

### Region selection

```bash
python3 run_pipeline.py --region EU --query "ai filter"
python3 run_pipeline.py --region ASIA --query "ai filter"
```

### Test mode (reduced data, faster)

```bash
python3 run_pipeline.py --test
```

### Multi-scenario execution

Runs five predefined scenarios (AI Filter, Dance Challenge, Viral Videos, KOL Content, Viral Prediction):

```bash
python3 run_multi_query.py              # All 5 scenarios
python3 run_multi_query.py --scenario 1 # AI Filter scenario only
```

### Individual social source

```bash
python3 run_all.py --query "ai filter" --source tiktok
python3 run_all.py --query "ai filter" --skip "twitter,reddit"
```

## Output

Each run writes to the `output/` directory:

| File | Contents |
|---|---|
| `full_report_YYYYMMDD_HHMM.txt` | Slack-formatted combined report |
| `social_YYYYMMDD_HHMM.json` | KOL radar, audio trends, TikTok, Instagram, YouTube, Reddit, Google Trends |
| `seo_YYYYMMDD_HHMM.json` | SEO keywords with coverage and priority scores |
| `ads_YYYYMMDD_HHMM.json` | Facebook Ads data by scenario |
| `all_data_YYYYMMDD_HHMM.json` | Combined output from all channels |

The social JSON includes:
- `kol_radar` — posts grouped by tier
- `audio_trends` — music aggregation with KOL usage counts
- `kol_high_potential` — top 20 KOL posts by AI fitness score

## KOL Radar Tiers

| Tier | Role | Accounts |
|---|---|---|
| Tier 1 — Trend Leaders | Create trends; daily monitoring | `@cyber0318`, `@hoaa.hanassii`, `@thybui.__` |
| Tier 2 — Fast Adopters | Confirm trend signals | `@angelinazhq`, `@lena_foxxx`, `@caroline_xdc`, `@upminaa.cos` |
| Tier 3 — Professional Radars | Technical data and benchmarks | `@cellow111`, `@emmawhatstwo`, `@voulezjj`, `@sawamura_kirari` |

**Audio signal thresholds:**
- 3+ KOLs using the same music → Strong trend
- 2 KOLs → Moderate trend
- 1 KOL → Single source

## Project Structure

```
openclaw-omni-channel-agent/
├── run_pipeline.py          # Master orchestrator (all channels)
├── run_all.py               # Social-only pipeline runner
├── run_multi_query.py       # Multi-scenario batch runner
├── apify_client.py          # Apify REST client (stdlib only)
├── sources/
│   ├── kol_radar.py         # KOL monitoring + AI fitness scoring
│   ├── tiktok.py
│   ├── instagram.py
│   ├── youtube.py
│   ├── reddit.py
│   ├── twitter.py
│   ├── google_trends.py
│   ├── facebook_ads.py
│   ├── ads_pipeline.py
│   ├── seo_pipeline.py
│   └── seo_full_research.py
├── formatters/
│   └── slack_formatter.py   # Slack report formatting
├── SKILL.md                 # OpenClaw skill documentation
└── TREND_METHODOLOGY.md     # KOL monitoring methodology
```

## Notes

- Twitter/X requires an official API v2 Bearer token; OpenTwitter JWT is not compatible.
- Google Trends may return HTTP 429 under high-frequency use — add delays between runs.
- The Semrush sitemap target is hardcoded to `art.myshell.ai`; modify `seo_pipeline.py` to change the domain.
- The Notion dedup database ID is hardcoded in `seo_pipeline.py`.
- Fetching all 11 KOL accounts in a single Apify batch call is more cost-efficient than individual calls.
