---
name: ado-client
description: Azure DevOps REST API utility. Provides curl patterns for work items (read, create, update, comment, attachments). Referenced by /bugfix and other ADO-integrated skills. NOT user-invocable.
---

# ADO Client — Azure DevOps REST API Reference

Utility skill for interacting with Azure DevOps. Not invoked directly — referenced by skills like `/bugfix`.

## Prerequisites

Three environment variables MUST be set (configured in `~/.claude/settings.json`):

| Variable | Description | Example |
|---|---|---|
| `ADO_PAT` | Personal Access Token | `xxxxxxxxxxxx` |
| `ADO_ORG` | Organization name | `tr-ggo` |
| `ADO_PROJECT` | Project name | `Mastersaf Fiscal Solutions` |

### Validation

Before any API call, verify variables are set:

```bash
if [ -z "$ADO_PAT" ] || [ -z "$ADO_ORG" ] || [ -z "$ADO_PROJECT" ]; then
  echo "ERROR: ADO environment variables not configured."
  echo "Set ADO_PAT, ADO_ORG, ADO_PROJECT in ~/.claude/settings.json"
  exit 1
fi
```

### Authentication

All requests use Basic auth with the PAT. Build the auth header:

```bash
ADO_AUTH=$(echo -n ":$ADO_PAT" | base64)
ADO_BASE="https://dev.azure.com/$ADO_ORG/$(echo "$ADO_PROJECT" | sed 's/ /%20/g')"
```

Every `curl` command below uses:
```bash
-H "Authorization: Basic $ADO_AUTH"
```

---

## Operation 1: Fetch Work Item

Retrieve a work item by ID with all fields and relations (attachments, links).

```bash
ADO_AUTH=$(echo -n ":$ADO_PAT" | base64)
ADO_BASE="https://dev.azure.com/$ADO_ORG/$(echo "$ADO_PROJECT" | sed 's/ /%20/g')"

curl -s \
  -H "Authorization: Basic $ADO_AUTH" \
  "$ADO_BASE/_apis/wit/workitems/{ID}?\$expand=all&api-version=7.0"
```

**Key fields in the response:**

| JSON path | Content |
|---|---|
| `.fields["System.Title"]` | Bug title |
| `.fields["System.Description"]` | Description (HTML) |
| `.fields["Microsoft.VSTS.TCM.ReproSteps"]` | Repro Steps (HTML) — primary for Bug type |
| `.fields["Microsoft.VSTS.TCM.SystemInfo"]` | System Info |
| `.fields["Microsoft.VSTS.Common.Priority"]` | Priority (1=Critical, 4=Low) |
| `.fields["Microsoft.VSTS.Common.Severity"]` | Severity (1=Critical, 4=Low) |
| `.fields["System.State"]` | State (New, Active, Resolved, Closed) |
| `.fields["System.WorkItemType"]` | Type (Bug, Task, etc.) |
| `.fields["System.AssignedTo"].displayName` | Assigned to |
| `.relations[]` | Attachments, parent/child links |

**Extracting text from HTML fields:**

Repro Steps and Description are HTML. Strip tags for readable text:

```bash
echo "$REPRO_STEPS_HTML" | sed 's/<br[^>]*>/\n/g; s/<[^>]*>//g; s/&nbsp;/ /g; s/&amp;/\&/g; s/&lt;/</g; s/&gt;/>/g'
```

---

## Operation 2: List Attachments

Attachments are in the `relations` array of the work item response. Filter for `AttachedFile`:

```bash
echo "$WORK_ITEM_JSON" | python3 -c "
import json, sys
data = json.load(sys.stdin)
for rel in data.get('relations', []):
    if rel.get('rel') == 'AttachedFile':
        print(rel['url'] + ' | ' + rel.get('attributes', {}).get('name', 'unnamed'))
"
```

Each line outputs: `<download_url> | <filename>`

---

## Operation 3: Download Attachment

Download an attachment file (image, document) by its URL:

```bash
ADO_AUTH=$(echo -n ":$ADO_PAT" | base64)

curl -s -L \
  -H "Authorization: Basic $ADO_AUTH" \
  -o "/tmp/{filename}" \
  "{attachment_url}"
```

For images (PNG, JPG): save to `/tmp/` and use `Read` tool to view them (Claude is multimodal).

---

## Operation 4: Post Comment

Add a comment to a work item. Comments are in HTML format.

```bash
ADO_AUTH=$(echo -n ":$ADO_PAT" | base64)
ADO_BASE="https://dev.azure.com/$ADO_ORG/$(echo "$ADO_PROJECT" | sed 's/ /%20/g')"

curl -s \
  -X POST \
  -H "Authorization: Basic $ADO_AUTH" \
  -H "Content-Type: application/json" \
  -d "{\"text\": \"$COMMENT_HTML\"}" \
  "$ADO_BASE/_apis/wit/workitems/{ID}/comments?api-version=7.0-preview.4"
```

**Comment language:** Always PT-BR (team convention).

**Signature:** Always append `<br><br><em>Comentário gerado via Claude Code</em>` at the end.

---

## Operation 5: Update Work Item Fields

Update fields on a work item using JSON Patch format.

```bash
ADO_AUTH=$(echo -n ":$ADO_PAT" | base64)
ADO_BASE="https://dev.azure.com/$ADO_ORG/$(echo "$ADO_PROJECT" | sed 's/ /%20/g')"

curl -s \
  -X PATCH \
  -H "Authorization: Basic $ADO_AUTH" \
  -H "Content-Type: application/json-patch+json" \
  -d '[
    {"op": "replace", "path": "/fields/{FieldPath}", "value": {VALUE}}
  ]' \
  "$ADO_BASE/_apis/wit/workitems/{ID}?api-version=7.0"
```

**Common field paths:**

| Field | Path | Value type |
|---|---|---|
| State | `/fields/System.State` | String: "New", "Active", "Resolved", "Closed" |
| Completed Work | `/fields/Microsoft.VSTS.Scheduling.CompletedWork` | Number (hours) |
| Original Estimate | `/fields/Microsoft.VSTS.Scheduling.OriginalEstimate` | Number (hours) |
| Remaining Work | `/fields/Microsoft.VSTS.Scheduling.RemainingWork` | Number (hours) |
| Assigned To | `/fields/System.AssignedTo` | String (email or display name) |

**Multiple fields in one call** — add multiple objects to the JSON array:

```bash
curl -s \
  -X PATCH \
  -H "Authorization: Basic $ADO_AUTH" \
  -H "Content-Type: application/json-patch+json" \
  -d '[
    {"op": "replace", "path": "/fields/Microsoft.VSTS.Scheduling.CompletedWork", "value": 2},
    {"op": "replace", "path": "/fields/System.State", "value": "Resolved"}
  ]' \
  "$ADO_BASE/_apis/wit/workitems/{ID}?api-version=7.0"
```

---

## Operation 6: Create Work Item (Task)

Create a new Task work item, linked as child of a parent Bug.

```bash
ADO_AUTH=$(echo -n ":$ADO_PAT" | base64)
ADO_BASE="https://dev.azure.com/$ADO_ORG/$(echo "$ADO_PROJECT" | sed 's/ /%20/g')"

curl -s \
  -X POST \
  -H "Authorization: Basic $ADO_AUTH" \
  -H "Content-Type: application/json-patch+json" \
  -d '[
    {"op": "add", "path": "/fields/System.Title", "value": "Fix: {description}"},
    {"op": "add", "path": "/fields/Microsoft.VSTS.Scheduling.OriginalEstimate", "value": {HOURS}},
    {"op": "add", "path": "/relations/-", "value": {
      "rel": "System.LinkTypes.Hierarchy-Reverse",
      "url": "'"$ADO_BASE"'/_apis/wit/workitems/{PARENT_BUG_ID}",
      "attributes": {"name": "Parent"}
    }}
  ]' \
  "$ADO_BASE/_apis/wit/workitems/\$Task?api-version=7.0"
```

**Response:** Returns the created work item JSON. Extract the new ID:

```bash
echo "$RESPONSE" | python3 -c "import json,sys; print(json.load(sys.stdin)['id'])"
```

---

## Operation 7: Link Work Items (Parent/Child)

Already handled by Operation 6 via the `relations/-` field during creation.

To add a link to an **existing** work item (e.g., link a Bug to an existing Task):

```bash
curl -s \
  -X PATCH \
  -H "Authorization: Basic $ADO_AUTH" \
  -H "Content-Type: application/json-patch+json" \
  -d '[
    {"op": "add", "path": "/relations/-", "value": {
      "rel": "System.LinkTypes.Hierarchy-Forward",
      "url": "'"$ADO_BASE"'/_apis/wit/workitems/{CHILD_ID}",
      "attributes": {"name": "Child"}
    }}
  ]' \
  "$ADO_BASE/_apis/wit/workitems/{PARENT_ID}?api-version=7.0"
```

**Relation types:**
- `System.LinkTypes.Hierarchy-Reverse` = "this is a child of X" (parent link)
- `System.LinkTypes.Hierarchy-Forward` = "this is a parent of X" (child link)

---

## Error Handling

| HTTP Status | Meaning | Action |
|---|---|---|
| 200/201 | Success | Continue |
| 401 | Unauthorized | PAT expired or invalid. Ask user to update `ADO_PAT` in `~/.claude/settings.json` |
| 403 | Forbidden | PAT lacks required scope. Needs: `Work Items (Read, Write)` |
| 404 | Not Found | Work item ID doesn't exist or wrong project |
| 400 | Bad Request | Malformed JSON or invalid field path. Check payload. |

Always check HTTP status before processing the response:

```bash
HTTP_STATUS=$(curl -s -o /tmp/ado_response.json -w "%{http_code}" \
  -H "Authorization: Basic $ADO_AUTH" \
  "$ADO_BASE/_apis/wit/workitems/{ID}?\$expand=all&api-version=7.0")

if [ "$HTTP_STATUS" != "200" ]; then
  echo "ADO API error: HTTP $HTTP_STATUS"
  cat /tmp/ado_response.json
  exit 1
fi
```
