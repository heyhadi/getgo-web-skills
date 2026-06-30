---
name: confluence-release-plan
description: Use when creating a new CMS (or DMS/FMS) release plan document in Confluence. Creates a parent release page with page properties, reviewer checklist, and Jira tickets table, plus a child release checklist page.
---

# Confluence Release Plan Creator

## Overview
Creates a two-page Confluence structure under "2026 Web Releases":
1. **Parent page** — release metadata, reviewers, Jira issues datasource table, notes
2. **Child page** — release checklist with pre-defined deployment tasks

## Required Parameters
If invoking via `args`, pass a natural language string or key=value pairs. Ask the user for any missing values before proceeding.

| Parameter | Description | Example |
|-----------|-------------|---------|
| `date` | Release date, YYYY-MM-DD | `2026-06-22` |
| `version` | Semantic version | `3.0.3` |
| `brand` | Brand(s) for this release | `Popcar` or `GetGo & Popcar` |
| `tickets` | Comma-separated Jira keys | `FRNT-4687, FRNT-4697` |
| `summary` | Release summary bullet(s) | `Bug fixes for Popcar` |
| `app` | App name (default: `CMS`) | `CMS` |

Optional:
- `release_manager_id` — Atlassian account ID, defaults to `627888b3a20bd0006fd8cf34` (Hadi Munawirul)

## Fixed Configuration (do not ask user for these)
- **Cloud ID**: `getgo.atlassian.net`
- **Atlassian Cloud UUID**: `c3808d3d-1f1e-4314-9ece-d425abdf9f8a`
- **Space ID**: `196609` (APP space, key = "APP")
- **Parent Page ID**: `2851898897` ("2026 Web Releases")
- **Reviewers**:
  - Pramana Baharsyah: `712020:a7f10a91-99e7-4962-9aca-675e175c1f34`
  - Firliandy Murdaya Eddy: `61a5cad3d2e64c007157d80b`
- **SRE (for checklist)**: Beng Liang Koh: `633a2cb8409249995eeb6d27`

---

## Step 1 — Create Parent Release Page

**Title**: `{date} {app} v{version} Release`
Example: `2026-06-22 CMS v3.0.3 Release`

**Tool call**: `mcp__atlassian__createConfluencePage`
- `cloudId`: `getgo.atlassian.net`
- `spaceId`: `196609`
- `parentId`: `2851898897`
- `contentFormat`: `html`
- `title`: `{date} {app} v{version} Release`

### Body HTML Template

Build the JQL by stripping all spaces from the ticket list: `issue in (FRNT-XXXX,FRNT-XXXX)`
URL-encode commas as `%2C` for the datasource URL.

For `summary`, if multiple items are provided (separated by `;` or newlines), render each as a `<li>` item. If single, one `<li>` is fine.

```html
<div data-type="bodied-extension" data-extension-key="details" data-extension-type="com.atlassian.confluence.macro.core" data-layout="default" data-parameters="{&quot;macroParams&quot;:{},&quot;macroMetadata&quot;:{&quot;macroId&quot;:{&quot;value&quot;:&quot;cms-page-properties-001&quot;},&quot;schemaVersion&quot;:{&quot;value&quot;:&quot;1&quot;},&quot;title&quot;:&quot;Page Properties&quot;}}"><table data-width="760"><thead><tr><th><p><strong>Date</strong></p></th><td><p><time datetime="{{YYYY-MM-DD}}">{{Month DD, YYYY}}</time></p></td></tr></thead><tbody><tr><th><p><strong>App</strong></p></th><td><p>{{app}}</p></td></tr><tr><th><p><strong>Brand</strong></p></th><td><p>{{brand}}</p></td></tr><tr><th><p><strong>Version</strong></p></th><td><p>{{version}}</p></td></tr><tr><th><p><strong>Release Summary</strong></p></th><td><ul>{{SUMMARY_LI_ITEMS}}</ul></td></tr><tr><th><p><strong>Release Manager</strong></p></th><td><p><span data-type="mention" data-user-id="{{release_manager_id}}">@{{release_manager_name}}</span></p></td></tr></tbody></table></div><h2>👥 Reviewer</h2><p>The checklist should be reviewed and signed off by the reviewer before any deployment. No release should proceed without the completion of all checklist items.</p><ul data-type="task-list"><li data-type="task-item"><input type="checkbox"> <span data-type="mention" data-user-id="712020:a7f10a91-99e7-4962-9aca-675e175c1f34">@Pramana Baharsyah</span></li><li data-type="task-item"><input type="checkbox"> <span data-type="mention" data-user-id="61a5cad3d2e64c007157d80b">@Firliandy Murdaya Eddy</span></li></ul><h2>📝 Issues Fixed &amp; Feature Tickets</h2><div data-type="block-card" data-url="https://getgo.atlassian.net/issues/?jql=issue%20in%20({{URL_ENCODED_TICKETS}})" data-datasource="{&quot;id&quot;:&quot;d8b75300-dfda-4519-b6cd-e49abbd50401&quot;,&quot;parameters&quot;:{&quot;cloudId&quot;:&quot;c3808d3d-1f1e-4314-9ece-d425abdf9f8a&quot;,&quot;jql&quot;:&quot;issue in ({{JQL_TICKETS}})&quot;},&quot;views&quot;:[{&quot;type&quot;:&quot;table&quot;,&quot;properties&quot;:{&quot;columns&quot;:[{&quot;key&quot;:&quot;issuetype&quot;},{&quot;key&quot;:&quot;key&quot;},{&quot;key&quot;:&quot;summary&quot;},{&quot;key&quot;:&quot;assignee&quot;},{&quot;key&quot;:&quot;priority&quot;},{&quot;key&quot;:&quot;status&quot;},{&quot;key&quot;:&quot;updated&quot;}]}}]}"><a href="https://getgo.atlassian.net/issues/?jql=issue%20in%20({{URL_ENCODED_TICKETS}})">https://getgo.atlassian.net/issues/?jql=issue%20in%20({{URL_ENCODED_TICKETS}})</a></div><h2>Notes</h2><blockquote><p>Add any additional context, known issues, or post-deployment observations here.</p></blockquote>
```

### Placeholder Reference
| Placeholder | Value |
|-------------|-------|
| `{{YYYY-MM-DD}}` | e.g. `2026-06-22` |
| `{{Month DD, YYYY}}` | e.g. `June 22, 2026` |
| `{{app}}` | e.g. `CMS` |
| `{{brand}}` | e.g. `Popcar` — use `&amp;` for `&` in HTML |
| `{{version}}` | e.g. `3.0.3` |
| `{{SUMMARY_LI_ITEMS}}` | e.g. `<li><p>Bug fixes for Popcar</p></li>` |
| `{{release_manager_id}}` | Atlassian account ID |
| `{{release_manager_name}}` | Display name |
| `{{JQL_TICKETS}}` | e.g. `FRNT-4687,FRNT-4697` (no spaces) |
| `{{URL_ENCODED_TICKETS}}` | e.g. `FRNT-4687%2CFRNT-4697` |

---

## Step 2 — Create Child Release Checklist Page

After Step 1 succeeds, capture the new page's `id` from the response. Use it as `parentId`.

**Title**: `{date} {app} v{version} Release Checklist`
Example: `2026-06-22 CMS v3.0.3 Release Checklist`

**Tool call**: `mcp__atlassian__createConfluencePage`
- `cloudId`: `getgo.atlassian.net`
- `spaceId`: `196609`
- `parentId`: `{id from Step 1 response}`
- `contentFormat`: `html`
- `title`: `{date} {app} v{version} Release Checklist`

### Body HTML Template

Replace `{{version}}` with the version (e.g. `3.0.3`). All task checkboxes start **unchecked**.

```html
<h1>⭐ Legend</h1><ul><li><p>Completed?: date when task is completed</p></li><li><p>Task: the item that must have been done before any release</p></li><li><p>To Do: checklists to complete the task</p></li><li><p>Information: documentation or any source link related to the task such as a Jira link, Coda link</p></li><li><p>PIC: the team in charge of testing</p></li><li><p>Remarks: the remarks indicate the task status has been completed</p></li></ul><table data-width="1800"><thead><tr><th><p><strong>Completed</strong></p></th><th><p><strong>Task</strong></p></th><th><p><strong>To do</strong></p></th><th><p><strong>Information</strong></p></th><th><p><strong>PIC</strong></p></th></tr></thead><tbody><tr><td><p></p></td><td><p>Review code changes to <code>2.0/main</code></p></td><td><ul data-type="task-list"><li data-type="task-item"><input type="checkbox"> Code review completed</li></ul></td><td><p></p></td><td><p><span data-type="mention" data-user-id="61a5cad3d2e64c007157d80b">@Firliandy Murdaya Eddy</span></p></td></tr><tr><td><p></p></td><td><p>Inform <strong>Web Lead/EM</strong> of deployment</p></td><td><ul data-type="task-list"><li data-type="task-item"><input type="checkbox"> Web Lead/EM notified</li></ul></td><td><p></p></td><td><p><span data-type="mention" data-user-id="712020:a7f10a91-99e7-4962-9aca-675e175c1f34">@Pramana Baharsyah</span> <span data-type="mention" data-user-id="61a5cad3d2e64c007157d80b">@Firliandy Murdaya Eddy</span></p></td></tr><tr><td><p></p></td><td><p>Inform <strong>SRE</strong> of deployment</p></td><td><ul data-type="task-list"><li data-type="task-item"><input type="checkbox"> SRE notified</li></ul></td><td><p></p></td><td><p><span data-type="mention" data-user-id="633a2cb8409249995eeb6d27">@Beng Liang Koh</span></p></td></tr><tr><td><p></p></td><td><p>Merge changes and push tag</p></td><td><ul data-type="task-list"><li data-type="task-item"><input type="checkbox"> Merge changes to <code>2.0/main</code></li><li data-type="task-item"><input type="checkbox"> Push tag version <code>v{{version}}</code></li><li data-type="task-item"><input type="checkbox"> Deployment complete</li></ul></td><td><p></p></td><td><p><span data-type="mention" data-user-id="61a5cad3d2e64c007157d80b">@Firliandy Murdaya Eddy</span></p></td></tr><tr><td><p></p></td><td><p>Rollback Plan (if required)</p></td><td><ul data-type="task-list"><li data-type="task-item"><input type="checkbox"> Redeploy previous build if necessary</li></ul></td><td><p></p></td><td><p><span data-type="mention" data-user-id="61a5cad3d2e64c007157d80b">@Firliandy Murdaya Eddy</span></p></td></tr></tbody></table>
```

---

## Step 3 — Report Back

After both pages are created, report:
- Parent page URL
- Child checklist URL

Example output:
```
✅ Release plan created:
- Release page: https://getgo.atlassian.net/wiki/spaces/APP/pages/XXXXXXXXX/2026-06-22+CMS+v3.0.3+Release
- Checklist: https://getgo.atlassian.net/wiki/spaces/APP/pages/YYYYYYYYY/2026-06-22+CMS+v3.0.3+Release+Checklist
```

## Example Invocation

```
/confluence-release-plan date=2026-06-22 version=3.0.3 brand=Popcar tickets=FRNT-4687,FRNT-4697 summary="Bug fixes for Popcar"
```

Or pass a natural description:
```
/confluence-release-plan Create a release plan for CMS v3.0.3 on 2026-06-22 for Popcar, tickets FRNT-4687 and FRNT-4697
```