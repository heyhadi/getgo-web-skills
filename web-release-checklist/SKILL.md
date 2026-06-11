name: web-release-checklist
description: >
Use this skill whenever the user wants to generate a web app release checklist (FMS, CMS, ZipZap, or other) from a Git commit ID.
Triggers when the user provides a commit hash and wants to create a Confluence release checklist,
or says things like "create a release checklist from commit X", "generate checklist from [commit hash]",
"make a release page for commits since X", or "release notes from commit [hash] to now".
This skill fetches all commits from the given commit ID to the current HEAD, groups them, and
publishes a formatted release checklist to Confluence using the standard template.

---

# Web Release Checklist Skill

Given a starting Git commit ID, this skill:

1. Lists all commits from that commit (exclusive) to `HEAD` (inclusive)
2. Groups commits into feature areas based on ticket prefixes or branch naming conventions
3. Creates a Confluence release checklist page using the standard template

---

## Step 1 — Gather Inputs

**First**, ask the user which web app this release is for:

> "Which web app is this release for? (FMS / CMS / ZipZap / Other)"
> If the user selects **Other**, ask them to specify the app name (e.g. `Admin Portal`). Store this as `app_type`.
> Then ask for the remaining inputs if not already provided:
> | Input | Description | Example |
> | --------------- | ------------------------------------------- | ------------------------------------------ |
> | `app_type` | Web app being released | `FMS`, `CMS`, `ZipZap`, `Admin Portal` |
> | `start_commit` | The commit ID to start from (exclusive) | `7b17354011b73346c0eb8d6a78bab630a453f067` |
> | `release_name` | Short name for this release | `GetPay 2C`, `FMS v3.1` |
> | `branch` | Branch to inspect (default: current branch) | `2.0/main` |
> | `tag_version` | The version tag to push on release | `v2.6.0` |
> | `prev_build_no` | Previous build number (for rollback plan) | `134` |
> | `deployer` | Name/handle of the person doing deployment | `@Firliandy` |
> | `web_lead` | Web Lead/EM to inform | `@Pramana` |

## | `sre` | SRE to inform | `@Beng Liang` |

## Step 2 — List Commits

Run this command in the repository root to get commits from `start_commit` to `HEAD`:

```bash
git log {start_commit}..HEAD --oneline --no-merges
```

This returns lines in the format:

```
abc1234 FRNT-2854 Fix payment gateway timeout
def5678 FRNT-3670 Add PayNow filter in transactions
...
```

Parse each line to extract:

-   **Commit short hash** (first 7 chars)
-   **Ticket ID** (e.g. `FRNT-XXXX`) — detected by regex `[A-Z]+-\d+`
-   **Commit message** (remainder of line)
    If no ticket prefix is found, group the commit under `General / Misc`.

---

## Step 3 — Group by Feature Area

Group commits by their ticket prefix (e.g. `FRNT`, `PAY`, `CMS`) or by the feature area the user specifies.
If the user provided explicit feature area groupings (e.g. "GetPay 2A = FRNT-2853 to FRNT-2858"), respect those. Otherwise, auto-group by ticket prefix.
**Deduplication:** In the Ticket List section, list each unique ticket ID only once (even if multiple commits reference the same ticket).
**Ticket links:** Always render tickets as ADF `inlineCard` nodes (see Step 5). This makes Confluence resolve and display the ticket title inline as a smart link. Never use plain URLs or markdown hyperlinks for tickets.

---

## Step 4 — Determine Feature Flags

Ask the user:

> "Are there any feature flags that need to be toggled for this release? If so, list them and whether they should be ON or OFF."
> Example:

```
REACT_APP_GETPAY_FLAG → ON
REACT_APP_GETPAY_2B_FLAG → OFF
```

## If the user says none, omit the feature flags row from the checklist table.

## Step 5 — Build the Confluence Page

Use the Atlassian MCP to create a new Confluence page in the **APP space** (spaceId: `196609`).
**Title format:** `{YYYY-MM-DD} {app_type} Release Checklist - {release_name}`
Use today's date for `YYYY-MM-DD`.
**IMPORTANT: Always use `contentFormat: "adf"` when publishing.** Markdown format renders ticket URLs as plain hyperlinks showing the URL text. ADF format with `inlineCard` nodes is required for Confluence to resolve and display the Jira ticket title inline as a smart link.

### ADF structure

The page body must be a valid ADF JSON string with `{"version": 1, "type": "doc", "content": [...]}`.
**Ticket smart link (inlineCard)** — use this ADF node anywhere a Jira ticket link appears (Ticket List bullets and Commits table):

```json
{
	"type": "inlineCard",
	"attrs": { "url": "https://getgo.atlassian.net/browse/FRNT-XXXX" }
}
```

**Example Ticket List section in ADF:**

```json
{"type": "heading", "attrs": {"level": 2}, "content": [{"type": "text", "text": "Ticket List"}]},
{"type": "paragraph", "content": [{"type": "text", "text": "FRNT", "marks": [{"type": "strong"}]}]},
{"type": "bulletList", "content": [
  {"type": "listItem", "content": [{"type": "paragraph", "content": [
    {"type": "inlineCard", "attrs": {"url": "https://getgo.atlassian.net/browse/FRNT-2854"}}
  ]}]},
  {"type": "listItem", "content": [{"type": "paragraph", "content": [
    {"type": "inlineCard", "attrs": {"url": "https://getgo.atlassian.net/browse/FRNT-2855"}}
  ]}]}
]},
{"type": "paragraph", "content": [{"type": "text", "text": "General / Misc", "marks": [{"type": "strong"}]}]},
{"type": "bulletList", "content": [
  {"type": "listItem", "content": [{"type": "paragraph", "content": [{"type": "text", "text": "feat: Update CI config"}]}]}
]}
```

---

## Step 5b — Write Preview File

Before asking for confirmation, write the page content to **`release-checklist-preview.md`** at the repo root so the user can review it in their editor.
The preview file includes the Ticket List, Release Checklist, and Notes sections only — **do NOT include the Commits in this Release table** in the preview file.
The preview uses plain URLs (not ADF) since it's just a local markdown file for review:

```markdown
# {YYYY-MM-DD} {app_type} Release Checklist - {release_name}

---

## Ticket List

**{Feature Area}**

-   https://getgo.atlassian.net/browse/TICKET-XXXX
-   https://getgo.atlassian.net/browse/TICKET-XXXX

---

## Release Checklist

| **Activity**                      | **Owner**  |
| --------------------------------- | ---------- |
| Review code changes to `{branch}` | {deployer} |

## ...

## Notes

> Add any additional context, known issues, or post-deployment observations here.
```

After writing, tell the user:

> "Preview written to `release-checklist-preview.md` — open it to review before I publish."

---

## Step 6 — Confirm & Publish

Before publishing, show the user a **preview summary**:

```
🖥️  App: FMS
📋 Release: GetPay 2C — 2026-05-15
📦 Commits: 9 total (FRNT: 7, Misc: 2)
🏷️  Tag: v2.6.0
🔀 Branch: 2.0/main
🚩 Feature flags: REACT_APP_GETPAY_FLAG ON, REACT_APP_GETPAY_2B_FLAG OFF
👤 Deployer: @Firliandy | Web Lead: @Pramana | SRE: @Beng Liang
```

Ask: "Looks good? I'll publish this to Confluence."
Once confirmed, create the page using the Atlassian MCP tool `createConfluencePage` with:

-   `cloudId`: `getgo.atlassian.net`
-   `spaceId`: `196609`
-   `contentFormat`: `adf`
-   `status`: `current`
    **Parent folder selection** — All web release checklists live under the **Web releases** folder (ID `1763278904`). Inside that folder there are two subfolders, each with quarterly sub-subfolders:
    | App | Subfolder | Subfolder ID |
    | --- | --- | --- |
    | FMS | FMS - Release Checklist | `1763311663` |
    | CMS | CMS Release Checklist | _(unknown — look up via CQL)_ |
    Within each subfolder there is a quarterly folder named `{YYYY} {Q#} - {App}` (e.g. `2026 Q2 - FMS`). Determine the correct quarter from today's date:
    | Month | Quarter |
    | --- | --- |
    | Jan – Mar | Q1 |
    | Apr – Jun | Q2 |
    | Jul – Sep | Q3 |
    | Oct – Dec | Q4 |
    **Known quarterly folder IDs (use directly — no CQL needed if the quarter matches):**
    | App | Quarter | Folder name | Folder ID |
    | --- | --- | --- | --- |
    | FMS | Q2 2026 | 2026 Q2 - FMS | `2729181218` |
    **How to find the target folder ID at publish time:**

1. If the quarter and app match a known ID above, use it directly as `parentId` — skip the CQL lookup.
2. Otherwise search CQL: `ancestor = 1763311663 AND title ~ "Q{N} {YYYY}" AND space.key = "APP"` (use the subfolder ID for the app, e.g. `1763311663` for FMS).
3. If the quarterly folder does not exist yet, use the app subfolder ID as `parentId` and create the quarter folder first if needed. Quarterly folder naming convention: `{YYYY} Q{N} - {App}` (e.g. `2026 Q3 - FMS`).
   If the exact quarterly folder ID cannot be resolved, fall back to `parentId: 1763278904` (Web releases root) — the page will still land in the correct space and can be moved manually.

---

## Error Handling

-   **No commits found**: Tell the user and ask them to verify the commit hash exists on the current branch.
-   **Commit not on branch**: Suggest running `git log --all --oneline | grep {short_hash}` to find which branch it's on.
-   **Atlassian auth failure**: Ask the user to check that the Atlassian connector is connected in Claude settings.
-   **No ticket IDs detected**: Proceed with grouping all commits under "General / Misc" and inform the user.
