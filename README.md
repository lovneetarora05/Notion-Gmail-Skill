# Notion & Gmail Summary Digest

A Cowork skill for Cleo Product Managers. Runs on a schedule to scan Notion and Gmail for activity relevant to you, automatically creates Jira tickets for action items, and delivers a single digest to your Slack DM — so you don't have to manually check Notion comments, planning docs, and your inbox separately.

---

## What it does

Each run covers three areas:

**Notion — Your Docs**
Scans pages where you are the author or a contributor for meaningful changes (new sections, status updates, decisions, new comments) within the look-back window. Ignores trivial edits like typo fixes. Uses topic-based searches across your configured work areas so that existing docs (created weeks or months ago) are included — not just newly created pages.

**Notion — @Mentions**
Finds any page where you've been @mentioned in a comment. Summarises what's being asked and automatically creates a Jira ticket in your project for each new mention (with duplicate prevention across runs).

**Squad Planning Docs**
Checks the T2-26 Team Plans + Check-ins database for any squad planning docs updated in the window — surfacing meaningful changes like new decisions, status changes, or timeline updates.

**Gmail**
Scans your inbox for emails from Cleo colleagues (@meetcleo.com), external partners (Plaid, Mastercard, Equifax), and relevant Notion/Slack tool notifications. Excludes meeting invites, marketing emails, and unrelated third-party senders. Creates Jira tickets for any clear action items or direct questions.

Everything gets bundled into a single Slack DM delivered at the end of each run.

> **Important — Notion date filtering:** The Notion search tool's `created_date_range` filter works on page *creation* date, not last-edited date. This skill uses broad date ranges and filters by the `timestamp` field in results (which reflects last-edit time) to correctly surface existing documents that were edited in the look-back window. If you customise the skill, preserve this approach or you will miss updates to long-lived docs.

---

## Look-back window

- **Daytime runs (08:00–20:00 BST):** 2-hour look-back
- **Overnight runs (20:00–08:00 BST):** 6-hour look-back

---

## Setup

### 1. Install the skill

In the Cowork app, install `notion-gmail-summary-digest.skill`.

### 2. Connect the required MCPs

Make sure the following connectors are active in Cowork — and crucially, enabled for **scheduled tasks** (not just the chat interface):

| Connector | Used for |
|---|---|
| Notion | Reading pages, comments, and planning docs |
| Gmail | Scanning your inbox |
| Slack | Sending the digest DM |
| Jira / Atlassian | Creating and deduplicating LP tickets |

> **Note on Slack:** If Slack DMs stop arriving after working initially, the likely cause is the Slack connector being available only "in chat" but not in the cloud task runner. Fix by disconnecting and reconnecting Slack from Cowork Settings — this re-registers it with the scheduler.

### 3. Fill in your config

At the top of `SKILL.md`, replace the placeholder values with your own:

| Variable | How to find it |
|---|---|
| `YOUR_EMAIL` | Your Cleo email address |
| `YOUR_NOTION_USER_ID` | Ask Claude: "what is my Notion user ID?" with Notion connected |
| `YOUR_SLACK_USER_ID` | Ask Claude: "what is my Slack user ID?" with Slack connected |
| `YOUR_JIRA_PROJECT_KEY` | Your team's Jira project key (e.g. `LP`) |
| `SQUAD_PLANNING_DB_URL` | URL of the T2-26 Team Plans + Check-ins Notion database |

### 4. Schedule it

Set up a recurring scheduled task in Cowork pointing at this skill. Recommended cron:

```
0 8,10,12,14,16,18,20 * * *
```

This runs every 2 hours during UK working hours. Add a `0 6 * * *` run if you want overnight coverage (it will automatically use the 6-hour look-back).

---

## Files

```
notion-gmail-summary-digest.skill   — installable Cowork skill package
README.md                           — this file
```

---

## Built with

- [Cowork](https://claude.ai) by Anthropic
- Notion MCP · Gmail MCP · Slack MCP · Atlassian MCP
