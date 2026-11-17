---
name: ad-performance
description: "Reviews paid ad performance from the ad accounts connected to Reeve (Meta, Google Ads, TikTok): lists campaigns and pulls spend, impressions, clicks, conversions and other metrics for a date window, for the account, a campaign, an ad set or an ad. Use when someone asks how their ads are doing, what they spent on ads, which campaign performs best, or for a weekly or monthly ad report."
---

# Ad performance through Reeve

Reeve reads the ad accounts the user has connected in Reeve. This workflow
only reads; it changes no campaign.

## Tools

| Tool | What it returns |
| :- | :- |
| `ads_list_providers` | The ad platforms connected for this organization and the reads each supports |
| `ads_list_campaigns` | The campaigns on one platform (`provider`), optionally filtered by `status` |
| `ads_list_insights` | Performance metrics on one platform for a date window, at a chosen `level` |

`provider` is one of `meta_ads`, `google_ads` or `tiktok_ads`.

## Workflow

1. **Find the platforms.** Call `ads_list_providers`. If the user named a
   platform, use it; otherwise review each connected one. If none is
   connected, tell the user to connect an ad account in the Reeve app.
2. **List the campaigns** with `ads_list_campaigns` when the user asks about
   a campaign by name or wants a per-campaign view. Pass `status: "ACTIVE"`
   for running campaigns only. TikTok needs `advertiser_id`.
3. **Pull the metrics** with `ads_list_insights`:
   - the window as `date_preset` ("last_7d", "last_30d", "this_month") or as
     `since` and `until` (YYYY-MM-DD);
   - `level` ("account", "campaign", "adset" or "ad") for the breakdown;
   - `campaign_id` to narrow to one campaign;
   - for Google Ads, `report` ("keyword", "ad", "device" or
     "conversion_action") for a drilldown instead of `level`.
4. **Summarize.** Lead with spend and results for the window, then the best
   and worst performers by cost per result. Name the window and platform for
   every number, and compare platforms only on metrics they both report.
