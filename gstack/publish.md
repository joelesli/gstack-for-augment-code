---
description: "Publishes a gstack artifact (investigate.md) to Confluence with a Jira macro header. Creates '<KEY> — Root Cause & Solution' page under a specified parent."
argument-hint: [JIRA-KEY] repo=[path] confluence-parent=[page-id]
---

You are publishing a gstack investigation artifact to Confluence for Jira ticket **$ARGUMENTS**.

Parse `$ARGUMENTS`:
- First token: Jira key (e.g. `MEO-37305`)
- `repo=<path>`: absolute path to the source repository
- `confluence-parent=<id>`: Confluence parent page ID

## Step 1 — Read the artifact

Read `<repo>/.augment/features/<KEY>/investigate.md`.

If the file does not exist: output `ERROR: <repo>/.augment/features/<KEY>/investigate.md not found. Run /gstack:investigate <KEY> first.` and stop.

## Step 2 — Fetch the Jira issue

Call `jira_get_issue` for `<KEY>` (fields: `summary,status,priority,issuetype`). Extract the one-line summary for the page header.

## Step 3 — Build the Confluence page content

Produce the page in Confluence storage format (not Markdown). Structure:

**Header block:**
```xml
<p>
  <ac:structured-macro ac:name="jira">
    <ac:parameter ac:name="server">Jira</ac:parameter>
    <ac:parameter ac:name="key"><KEY></ac:parameter>
  </ac:structured-macro>
  &nbsp; <strong><KEY></strong> — <em><one-line Jira summary></em>
</p>
<hr/>
```

**Body sections** (converted from investigate.md, in order):

| investigate.md field | Confluence section heading |
|---|---|
| Symptom | ## Symptom |
| Root cause | ## Root Cause |
| Location | ## Location |
| Hypotheses tested | ## Hypotheses Tested |
| Fix | ## Fix |
| Evidence | ## Evidence |
| Regression test | ## Regression Test |
| Related | ## Related |
| Status + Concerns | ## Status |

Rules:
- Code/file paths → `<code>` tags.
- Multi-line code blocks → `<ac:structured-macro ac:name="code">` with `language` parameter.
- BLOCKED status: include all tested hypotheses and recommended next steps under ## Status.
- Omit sections whose value is `—` or empty.

## Step 4 — Create the page

Call `confluence_create_page` with:
- `space_key`: inferred from the parent page (call `confluence_get_page` on `confluence-parent` to get its space key)
- `parent_id`: `<confluence-parent>`
- `title`: `<KEY> — Root Cause and Solution`
- `content`: the storage-format string from Step 3
- `content_format`: `storage`

## Step 5 — Output

```
PUBLISHED <KEY>: <confluence-page-url>
```

On any MCP failure: output `FAIL <KEY>: <reason>` and stop. Do not partially create content.
