---
name: notion-gmail-summary-digest
description: >
  Scheduled digest for Product Managers at Cleo. Every run, scans Notion for docs you author or contribute to, pages where you've been @mentioned in comments, and squad planning docs — then checks Gmail for relevant emails from colleagues and partners. Creates Jira tickets automatically for action items and @mentions, and delivers a single summary Slack DM. Use this skill to set up or run a recurring PM digest that keeps you across Notion changes, email actions, and planning doc updates without having to check everything manually.
---

# Notion & Gmail Summary Digest

A scheduled automation for Cleo PMs. Each run scans Notion and Gmail for activity relevant to you, creates Jira tickets for anything actionable, and sends a single digest to your Slack DM.

---

## CONFIGURATION (fill in before first run)

| Variable | Description | Example |
|---|---|---|
| `YOUR_EMAIL` | Your Cleo email address | lovneet@meetcleo.com |
| `YOUR_NOTION_USER_ID` | Your Notion user UUID | be22e733-5bb1-... |
| `YOUR_SLACK_USER_ID` | Your Slack user ID | U06G3UGP3J8 |
| `YOUR_JIRA_PROJECT_KEY` | Your Jira project key | LP |
| `SQUAD_PLANNING_DB_URL` | URL of the T2-26 Team Plans + Check-ins database | https://www.notion.so/meetcleo/3125c63b... |
| `YOUR_WORK_AREAS` | 3–5 search terms covering your squad/product area | "Geo Expansion", "UK EWA", "vulnerable user" |

To find your Notion user ID: ask Claude "what is my Notion user ID?" with the Notion MCP connected.
To find your Slack user ID: ask Claude "what is my Slack user ID?" with the Slack MCP connected.

---

## LOOK-BACK WINDOW

Before doing anything else, run `TZ=Europe/London date '+%H'` in the shell to get the current UK hour. Do not guess — always read it fresh.

- UK hour in **{8–19}** → use a **2-hour** look-back
- UK hour in **{20–7}** → use a **6-hour** look-back (covers overnight activity)

State the computed window (start and end in both UTC and BST) at the top of your working notes.

---

## CRITICAL: How Notion search date filtering works

**The `created_date_range` filter in the Notion search tool filters by page *creation* date — NOT by last-edited date.** If you filter by creation date to find pages edited in the look-back window, you will miss every existing document that was edited but created before the filter date. This is the primary cause of missed updates.

**The correct approach for every Notion search in this skill:**
1. Use a **broad `created_date_range`** (e.g., `start_date: "2025-01-01"`) so that long-lived documents are included in results.
2. After getting results, filter by the **`timestamp` field** in each result — this reflects the page's **last-edited time**, not its creation time. Only include results whose `timestamp` falls within the look-back window.
3. If you are unsure whether a doc was edited in the window, fetch it directly. The "as of [datetime]" shown at the top of the fetch output is the page's last-modified time — use this to confirm.

Apply this approach in every Notion search below.

---

## JOB 1 — NOTION

### 1a. Docs you author or contribute to

Run **multiple targeted searches in parallel** to surface docs you own or actively contribute to. Use a broad `created_date_range` (e.g., `start_date: "2025-01-01"`) and filter results by `timestamp` to identify pages edited within the look-back window.

Recommended parallel searches:
1. Search with `created_by_user_ids: ["YOUR_NOTION_USER_ID"]` — finds pages you created.
2. Search for your name (e.g., `"YOUR_NAME"`) — surfaces pages where you are listed as owner or contributor in page properties or body.
3. Search for each term in `YOUR_WORK_AREAS` (one search per area) — covers active docs in your squad/product area that may not list you by name but are within your ownership scope.

> **Customise `YOUR_WORK_AREAS`** to match your squad. For example, if you work on Geo Expansion and UK EWA, your searches might include "Geo Expansion", "UK EWA", "vulnerable user". Add or remove terms to suit your area.

After getting results, keep only those where `timestamp` is within the look-back window. For each such doc:
- Fetch the full page with `include_discussions: true`.
- Check for **meaningful changes** only — new sections, significant rewrites, deleted content, status changes, structural edits. Ignore trivial edits like typo fixes, whitespace, or minor formatting.
- Check for **new comments** (any commenter).
- Summarise what changed and/or what comments were added.

### 1b. Pages where you are @mentioned in a comment

Search broadly for pages where you have been @mentioned in a comment. Use a broad `created_date_range` and filter by `timestamp` as described above.

Run these searches in parallel:
1. Query: your name (e.g., `"Lovneet"`)
2. Query: your Notion user ID (e.g., `"be22e733"`)

For each page with a `timestamp` in the look-back window, fetch it with `include_discussions: true` and check whether any comment explicitly @-mentions your Notion user ID (`YOUR_NOTION_USER_ID`). Only surface comments where you are directly tagged — not every comment on a page you happen to own.

For each confirmed @mention in the look-back window:
- Summarise the comment and what is being asked of you.
- Note the doc name and who tagged you.
- **Create a Jira ticket** in `YOUR_JIRA_PROJECT_KEY`:
  - **Summary**: one-line description of what's being asked
  - **Description**: full comment text, commenter name, doc name, link to the page
  - **Issue type**: Task
  - **No duplicates**: check for existing open tickets referencing the same Notion comment before creating.

---

## JOB 2 — SQUAD PLANNING DOCS

Fetch the squad planning docs database from `SQUAD_PLANNING_DB_URL` to get all planning doc URLs.

Then search Notion for recently edited planning docs using a topic query (e.g., **"T2-26 Planning Doc"** or your current term name), with a broad `created_date_range` (e.g., `start_date: "2025-01-01"`). Filter results by `timestamp` to identify docs edited within the look-back window.

For each planning doc with a `timestamp` in the look-back window:
- Fetch the full page with `include_discussions: true`.
- Check for **meaningful changes** — new sections, status changes, updated timelines, decisions added or removed, structural edits. Ignore trivial edits.
- Check for **new comments** (any commenter).
- Summarise: doc name, what changed, any new comments.

If nothing meaningful changed, say "No significant updates."

---

## JOB 3 — GMAIL

Search Gmail for emails received within the look-back window.

**Include:**
- Emails from Cleo colleagues (any @meetcleo.com sender)
- Emails from external partners: Plaid, Mastercard, Equifax
- Tool notification emails from Notion or Slack — only if they are about someone @mentioning you or making changes to a shared doc. Ignore generic product/marketing emails.

**Exclude:**
- Meeting invitations, acceptances, or declines
- Emails from any other third-party domain
- Automated system/marketing emails (newsletters, receipts, unrelated alerts)

For each qualifying email: summarise sender, subject, and key point or ask. Flag any clear action items or direct questions aimed at you.

**Jira tickets for emails:** For each email from a Cleo colleague, Plaid, Mastercard, or Equifax containing a clear action item or direct question, create a Jira task in `YOUR_JIRA_PROJECT_KEY`:
- **Summary**: concise one-line description of the ask
- **Description**: sender name, email subject, the key ask, date received
- **Issue type**: Task
- **No duplicates**: check for existing open tickets referencing the same email thread before creating.

---

## DELIVERY

Send a single Slack DM to `YOUR_SLACK_USER_ID` using this format (use single `*asterisks*` for bold in Slack, not double):

```
*🗂️ Notion & Email Digest — [time range e.g. 14:00–16:00 BST]*

*📝 Notion — Docs You Own or Contribute To*
[For each doc with meaningful activity: doc name + summary. If nothing: "No significant updates."]

*🔔 Notion — You Were Tagged*
[For each @mention: doc name, who tagged you, summary of ask, Jira ticket number. If none: "No new tags."]

*📋 Squad Planning Docs*
[For each updated planning doc: name + summary. If nothing: "No significant updates."]

*📧 Email — Cleo / Partners / Tool Notifications*
[For each qualifying email: sender, subject, one-line summary. If none: "No relevant emails."]

*🎫 New Jira Tickets Created*
[List all tickets created this run with ticket number, title, and source. If none: "None this round."]
```

---

## CONSTRAINTS

- Do NOT mark emails as read, archive them, or modify them.
- Do NOT mark Notion comments as read or resolve discussions.
- Only DM the user — do not post to any Notion pages or Slack channels.
- For Notion: skip trivial/minor edits — only surface meaningful changes.
- For Gmail: do not surface emails from third-party domains other than Plaid, Mastercard, and Equifax (Notion/Slack tool notifications are the exception).

---

## REQUIRED MCPS

This skill requires the following MCP connectors to be active in your Cowork session:

| MCP | Used for |
|---|---|
| Notion | Reading pages, comments, and planning docs |
| Gmail | Scanning your inbox |
| Slack | Sending the digest DM |
| Jira / Atlassian | Creating and deduplicating Jira tickets |

> **Slack tip:** If DMs stop arriving after working initially, the cause is usually the Slack connector being available only "in chat" but not in the cloud task runner. Fix by disconnecting and reconnecting Slack from Cowork Settings.
