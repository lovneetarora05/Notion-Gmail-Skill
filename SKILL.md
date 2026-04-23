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

To find your Notion user ID: ask Claude "what is my Notion user ID?" with the Notion MCP connected.
To find your Slack user ID: ask Claude "what is my Slack user ID?" with the Slack MCP connected.

---

## LOOK-BACK WINDOW

Before doing anything else, run `TZ=Europe/London date '+%H'` in the shell to get the current UK hour. Do not guess — always read it fresh.

- UK hour in **{8–19}** → use a **2-hour** look-back
- UK hour in **{20–7}** → use a **6-hour** look-back (covers overnight activity)

State the computed window (start and end in both UTC and BST) at the top of your working notes.

---

## JOB 1 — NOTION

### 1a. Docs you author or contribute to

Search Notion for pages where you are the author or contributor with activity in the look-back window. For each:
- Check for **meaningful changes** only: new sections, significant rewrites, deleted content, status changes, structural edits. Ignore typo fixes, whitespace, or minor formatting.
- Check for **new comments** (any commenter).
- Summarise what changed and/or what comments were added.

### 1b. Pages where you are @mentioned in a comment

Search Notion for pages where you have been @mentioned in a comment within the look-back window. For each result:
- Summarise the comment and what is being asked of you.
- Note the doc name and who tagged you.
- **Create a Jira ticket** in `YOUR_JIRA_PROJECT_KEY`:
  - **Summary**: one-line description of what's being asked
  - **Description**: full comment text, commenter name, doc name, link to the page
  - **Issue type**: Task
  - **No duplicates**: check for existing open tickets referencing the same Notion comment before creating.

---

## JOB 2 — SQUAD PLANNING DOCS

Fetch the squad planning docs database from `SQUAD_PLANNING_DB_URL`.

For each planning doc edited within the look-back window:
- Check for **meaningful changes**: new sections, status changes, updated timelines, decisions added or removed, structural edits. Ignore trivial edits.
- Check for **new comments** on the page.
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
:robot_face: *Notion & Gmail Digest — [time range e.g. 14:00–16:00 BST]*

*:memo: Notion — Docs You Own or Contribute To*
[For each doc with meaningful activity: doc name + summary. If nothing: "No significant updates."]

*:bell: Notion — You Were Tagged*
[For each @mention: doc name, who tagged you, summary of ask, Jira ticket number. If none: "No new tags."]

*:clipboard: Squad Planning Docs*
[For each updated planning doc: name + summary. If nothing: "No significant updates."]

*:email: Email — Cleo / Partners / Tool Notifications*
[For each qualifying email: sender, subject, one-line summary. If none: "No relevant emails."]

*:ticket: New Jira Tickets Created*
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
