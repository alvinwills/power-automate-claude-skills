---
name: power-automate-connectors
description: Authoritative reference of valid Power Automate connector actions (operationId + parameters) for SharePoint, Office 365 Outlook, Excel Online, Approvals, OneDrive, Forms, plus trigger schemas. MUST be consulted whenever creating or editing Power Automate / Logic Apps flow JSON so that no non-existent action is emitted. Trigger: any work on .json flow definitions, Workflows/*.json files, OpenApiConnection actions, or when the user mentions Power Automate / Power Apps flows.
---

# Power Automate Connectors — Action Reference

## THE RULE (read first)

When creating or editing a Power Automate / Logic Apps flow definition:

1. **Never invent an `operationId`.** Every `host.operationId` must be a real operation from the connector's published reference. If it is not in this file, **do not use it** — fetch the official connector reference page first (URLs at the bottom) with WebFetch, confirm the exact `operationId` and parameter keys, then proceed.
2. **Parameter keys are exact and case-sensitive.** `dataset` not `siteAddress`, `table` not `listName`. Using the display name as the key is wrong. **Body-object properties get flattened with a prefix in flow JSON:**
   - `CreateNewFolder` / `CreateFile`: the folder/file path is under a `parameters` body object → use key `parameters/path` (and `parameters/name`, `parameters/body` for CreateFile), NOT bare `path`. `dataset` and `table` stay flat.
   - `PostItem` / `PatchItem` (SharePoint **and** Excel): list/table column values are flattened as `item/<ColumnInternalName>`, e.g. `"item/Title": "...", "item/ApprovalID": "..."` — NOT a single `item: {...}` object. `dataset`, `table`, `id` stay flat.
   - `CreateSharingLink` (SharePoint): body params are under a `permission` object → `permission/type` (`view`/`edit`/`embed`), `permission/scope` (`organization`/`anonymous`/`existingAccess`), `permission/expirationDateTime`. `dataset`, `table`, `id` stay flat.
   - When unsure whether a param is flat or `parameters/`-/`permission/`-/`item/`-prefixed, build the action once in the designer and export, or read a real exported flow.
2b. **`dataset` and `table` MUST be literal strings, never `@outputs('Compose_X')` or other expressions.** SharePoint (and Excel) actions with a `dynamic` `item`/schema parameter resolve their column schema *at design time* by reading `dataset`+`table`. If those are opaque expressions the designer can't resolve them → it flags every `item/...`/`parameters/...` field as "no longer present in the operation schema" and "X is required". Always hardcode the site URL and the **list/library GUID** (display names sometimes fail to resolve too — prefer the GUID, found at Site contents → list → List settings → URL `List=%7B<GUID>%7D`).
2b-fix. **When the designer/save says "X is no longer present in the operation schema" — diagnose before deleting.** Two causes, two different fixes:
    - **Cause 1 — unresolved dynamic schema (most common):** `dataset`/`table` (SharePoint/Excel) is an expression, so the column list can't resolve and EVERY `item/...` field is flagged at once. **Do NOT remove the fields** — hardcode `dataset`+`table` as literal URL/GUID strings so the schema resolves (see 2b).
    - **Cause 2 — a genuinely invalid/stale key:** the single key was renamed, never existed, or belongs to a different operation. **Remove that key from the action.** If the value still needs to land somewhere, look up the correct key on the live connector page and use that instead. A key the live schema doesn't list passes JSON validation but breaks at save/runtime — never keep it.
2c. **Modern flow import conflict — "unpublished active row":** if a flow was edited in the designer (creating a draft) you cannot `pac solution import --force-overwrite` over it; you get `you are attempting to do a published update ... when there exists an unpublished active row`. `pac solution publish` does NOT clear modern-flow drafts. Fix: in make.powerautomate.com delete the flow (no run history lost if never run) or discard its draft, then re-import — it's recreated fresh.
3. **`apiId` format:** `/providers/Microsoft.PowerApps/apis/shared_<connectorname>` — e.g. `shared_sharepointonline`, `shared_office365`, `shared_excelonlinebusiness`, `shared_approvals`, `shared_onedriveforbusiness`, `shared_microsoftforms`, `shared_office365users`.
4. **Action shape (modern "OpenApiConnection"):**
   ```json
   "Action_Name": {
     "type": "OpenApiConnection",
     "inputs": {
       "parameters": { "<key>": "<value>", ... },
       "host": {
         "apiId": "/providers/Microsoft.PowerApps/apis/shared_<connector>",
         "operationId": "<exact operationId>",
         "connectionName": "shared_<connector>"
       }
     },
     "runAfter": { "<PrevAction>": ["Succeeded"] }
   }
   ```
5. **Approval (and other webhook) actions use `OpenApiConnectionWebhook`**, not `OpenApiConnection`. They also support `"limit": { "timeout": "P14D" }`.
6. **Connection references** declared in solution source use the publisher prefix (e.g. `aaw_`). They cannot be auto-created by import — the user creates them in make.powerautomate.com first; Dataverse may preserve camelCase in the logical name (verify via `pac solution export` + grep `customizations.xml`).
7. **Always emit modern (new-designer) schema.** Manual trigger = `type: Request, kind: Button, inputs.schema: {type:object, properties:{...}, required:[...]}, metadata.operationMetadataId`. Never `type: Manual` (deprecated, breaks the new designer).

---

## SharePoint Online — `shared_sharepointonline`

### Files & folders
| Action | operationId | Required params (key) | Notes |
|---|---|---|---|
| Create new folder | `CreateNewFolder` | `dataset` (site URL), `table` (list/library name), `path` (e.g. `folder1/folder2`) | Returns folder metadata (dynamic) |
| Create file | `CreateFile` | `dataset`, `folderPath` (must start with an existing library), `name`, `body` (binary) | Returns `SPBlobMetadataResponse` |
| Get file content | `GetFileContent` | `dataset`, `id` (File Identifier string) | Returns binary |
| Get file content using path | `GetFileContentByPath` | `dataset`, `path` | Returns binary |
| Get file metadata | `GetFileMetadata` | `dataset`, `id` (File Identifier) | `SPBlobMetadataResponse` |
| Get file metadata using path | `GetFileMetadataByPath` | `dataset`, `path` | `SPBlobMetadataResponse` |
| Get file properties | `GetFileItem` | `dataset`, `table` (library), `id` (**integer** list-item id) | dynamic — use to get the integer Id needed by sharing actions |
| Get files (properties only) | `GetFileItems` | `dataset`, `table` (library); opt: `$filter`, `$orderby`, `$top`, `folderPath` | dynamic array; filter on `FileLeafRef` to find a folder's integer Id |
| Get folder metadata | `GetFolderMetadata` | `dataset`, `id` (File Identifier) | `SPBlobMetadataResponse` |
| Get folder metadata using path | `GetFolderMetadataByPath` | `dataset`, `path` | `SPBlobMetadataResponse` |
| Update file | `UpdateFile` | `dataset`, `id`, `body` | |
| Update file properties | `UpdateFileItem` | `dataset`, `table`, `id` (integer), `item` (dynamic) | |
| Copy file | `CopyFileAsync` | `dataset`, `parameter/destinationDataset`, `parameter/sourceFileId`, `parameter/destinationFolderPath`, `parameter/nameConflictBehavior` | |
| Copy folder | `CopyFolderAsync` | similar to Copy file | |
| Move file | `MoveFileAsync` | similar | |
| Move folder | `MoveFolderAsync` | similar | |
| Extract folder | `ExtractFolderV2` | `dataset`, `sourceFileId`, `destinationFolderPath`, `overwrite` | unzip |
| List folder | `ListFolder` | `dataset`, `id` (folder File Identifier) | array |
| List root folder | `ListRootFolder` | `dataset`, `table` (library) | array |
| Delete file | `DeleteFile` | `dataset`, `id` (File Identifier) | |
| Check out file | `CheckOutFile` | `dataset`, `table`, `id` | |
| Check in file | `CheckInFileV2` | `dataset`, `table`, `id`, `checkInComment`, `checkInType` | |
| Discard check out | `DiscardCheckOut` | `dataset`, `table`, `id` | |

### List items
| Action | operationId | Required params |
|---|---|---|
| Create item | `PostItem` | `dataset`, `table` (list name), `item` (dynamic) |
| Get items | `GetItems` | `dataset`, `table`; opt: `$filter`, `$orderby`, `$top`, `folderPath`, `viewScopeOption` — results at `body.value` |
| Get item | `GetItem` | `dataset`, `table`, `id` (integer) |
| Update item | `PatchItem` | `dataset`, `table`, `id` (integer), `item` (dynamic) |
| Delete item | `DeleteItem` | `dataset`, `table`, `id` (integer) |
| Get changes for an item or a file | `GetItemChanges` | `dataset`, `table`, `id`, `sinceVersionId` |

### Attachments
| Action | operationId | Required params |
|---|---|---|
| Add attachment | `PostAttachment` | `dataset`, `table`, `id` (item id), `attachmentDetails` (object: name, contentBytes) |
| Get attachments | `GetAttachments` | `dataset`, `table`, `id` (item id) |
| Get attachment content | `GetAttachmentContent` | `dataset`, `table`, `id` (item id), `fileIdentifier` |
| Delete attachment | `DeleteAttachment` | `dataset`, `table`, `itemId`, `attachmentId` |

### Sharing & permissions
| Action | operationId | Required params | Notes |
|---|---|---|---|
| Create sharing link for a file or folder | `CreateSharingLink` | `dataset`, `table` (library name), `id` (**integer item id**), `type` (`view`/`edit`/`embed`), `scope` (`anonymous`/`organization`/`existingAccess`); opt `expirationDateTime` (anonymous only) | **Needs the integer list-item Id, NOT a path.** Get it first via `GetFileItem` or `GetFileItems` ($filter on `FileLeafRef`). Response is `SharingLinkPermission`; URL is around `body/link/webUrl` (verify against a live run). There is **no** `CreateSharingLinkByPath`. |
| Stop sharing an item or a file | `UnshareItem` | `dataset`, `table` (list/library), `id` (**integer**) | Removes all links + direct-access people except owners |
| Grant access to an item or a folder | `GrantAccess` | `dataset`, `table`, `id` (**integer**), `recipients` (email collection), `roleValue` (role string); opt `emailBody`, `sendEmail` | |
| Resolve person | `SearchForUser` | `dataset`, `table`, `entityId` (column), `searchValue` (email or name) | populates Person-type columns |

### Lists & misc
| Action | operationId | Required params |
|---|---|---|
| Get lists | `GetTables` | `dataset` |
| Get all lists and libraries | `GetAllTables` | `dataset` |
| Get list views | `GetListViews` | `dataset`, `table` |
| Send an HTTP request to SharePoint | `HttpRequest` | `dataset`, `method`, `uri` (e.g. `_api/web/lists/getbytitle('Documents')`); opt `headers`, `body` |
| Set content approval status | `SetApprovalStatus` | `dataset`, `table`, `id`, `status`, `comment` |

### SP triggers
`When an item is created` (`OnNewItems`), `When an item is created or modified` (`OnUpdatedItems`), `When an item is deleted` (`OnDeletedItems`), `When a file is created (properties only)` (`OnNewFileItems`), `When a file is created or modified (properties only)` (`OnUpdatedFileItems`), `When a file is deleted` (`OnDeletedFileItems`), `For a selected item` (`SelectedItemTrigger`), `For a selected file` (`SelectedFileTrigger`).

---

## Office 365 Outlook — `shared_office365`

| Action | operationId | Key params |
|---|---|---|
| Send an email (V2) | `SendEmailV2` | `emailMessage/To`, `emailMessage/Subject`, `emailMessage/Body` (HTML); opt `emailMessage/Cc`, `emailMessage/Bcc`, `emailMessage/Importance`, `emailMessage/Attachments`, `emailMessage/IsHtml` |
| Send an email from a shared mailbox (V2) | `SharedMailboxSendEmailV2` | `MailboxAddress`, `emailMessage/To`, `emailMessage/Subject`, `emailMessage/Body` |
| Send an HTTP request | `HttpRequest` | `Uri`, `Method`; opt `Body`, `Headers` |
| Send approval email | `SendApprovalEmail` | `To`, `Subject`, `Options` (comma list), `UserOptions`; opt `Importance`, `HideHTMLMessage`, `ShowHTMLConfirmationDialog` |
| Get emails (V3) | `GetEmailsV3` | opt `folderPath`, `fetchOnlyUnread`, `top`, `searchQuery` |
| Get event (V2) | `GetEventV2` | `id`, `calendarId` |
| Create event (V4) | `V4CalendarPostItem` | `calendarId`, `subject`, `start`, `end`, `timeZone`; opt `body`, `requiredAttendees`, `location` |
| Get user profile (V2) — *note:* this is the **Office 365 Users** connector, not Outlook | see below | |

Triggers: `When a new email arrives (V3)` (`OnNewEmailV3`), `When an upcoming event is starting soon (V3)` (`CalendarGetOnUpcomingItems`).

---

## Office 365 Users — `shared_office365users`

| Action | operationId | Key params |
|---|---|---|
| Get my profile (V2) | `MyProfile_V2` | (none) |
| Get user profile (V2) | `UserProfile_V2` | `id` (UPN or object id); opt `$select` |
| Get manager (V2) | `Manager_V2` | `id` |
| Get direct reports (V2) | `DirectReports_V2` | `id` |
| Search for users (V2) | `SearchUser_V2` | `searchTerm`; opt `top` |

---

## Excel Online (Business) — `shared_excelonlinebusiness`

All take `source` (`me`/`drive`/`group`/`sites`), `drive` (drive id), `file` (file id), `table` (table name/id).

| Action | operationId | Extra params |
|---|---|---|
| Add a row into a table | `AddRowV2` | `item` (object of column→value) |
| Update a row | `PatchItem` | `idColumn` (key column name), `id` (key value), `item` (changed columns); note: param is `idColumn`+`id` for the Excel connector |
| Get a row | `GetItem` | `idColumn`, `id` |
| Delete a row | `DeleteItem` | `idColumn`, `id` |
| List rows present in a table | `GetItems` | opt `$filter`, `$orderby`, `$top`, `$skip` — results at `body.value` |
| Get tables | `GetTables` | (just source/drive/file) |
| Get worksheets | `GetWorksheets` | (just source/drive/file) |
| Create table | `CreateTable` | `address`, `hasHeaders` |
| Run script | `RunScriptProd` | `scriptId`, `scriptParameters` |

(There is no `PatchItem` by integer index for Excel — it uses a key column. `AddRowV2` is the current add op; `AddRow` is deprecated.)

---

## Approvals — `shared_approvals`  (webhook actions → `type: OpenApiConnectionWebhook`)

| Action | operationId | Key params | Notes |
|---|---|---|---|
| Start and wait for an approval | `StartAndWaitForAnApproval` | `approvalType` (`Approve/Reject`, `Custom Responses - Wait for one response`, `Custom Responses - Wait for all responses`), `WebhookApprovalCreationInput/title`, `.../assignedTo`, `.../details`, `.../itemLink`, `.../itemLinkDescription` | the simple approve/reject form |
| Start and wait for an approval (custom responses) | `StartAndWaitForAnApprovalCustomResponseV2` | same input object + `.../responseOptions` (comma list) | for >2 outcome options |
| Create an approval | `CreateAnApproval` | as above | fire-and-don't-wait variant |
| Create an approval (custom responses) | `CreateAnApprovalCustomResponseV2` | as above | |
| Wait for an approval | `WaitForAnApproval` | `approvalName` (from a prior Create) | pair with Create… |
| Send an email with options | (Outlook connector `SendApprovalEmail`, not Approvals) | | lightweight alternative |

Response shape after a wait: `body.outcome` (string — the selected option, e.g. `Approve`/`Reject` or a custom value), `body.responses` (array — note the **s**, plural) of `{responder:{id,displayName,email,...}, approverResponse, comments, requestDate, responseDate}`. Read as `outputs('Wait_X')?['body/outcome']` and `outputs('Wait_X')?['body/responses']`, comment as `first(outputs('Wait_X')?['body/responses'])?['comments']`. On timeout, `responses` is empty — guard with `if(empty(outputs('Wait_X')?['body/responses']), '<fallback>', outputs('Wait_X')?['body/outcome'])`. Add `"limit": { "timeout": "P14D" }` to the **wait** action. Downstream actions that must run on timeout need `runAfter: { "Wait_X": ["Succeeded", "TimedOut"] }`.

**PROVEN-WORKING PATTERN (use this, not `StartAndWaitForAnApproval`):** the single `StartAndWaitForAnApproval` op has a `dynamic` `WebhookApprovalCreationInput` whose sub-fields the designer often fails to resolve → "can't find item / X is no longer present in the operation schema". Instead use the two-action pattern that exports cleanly:
```json
"Send_X_Approval": {
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "approvalType": "CustomResponse",
      "ApprovalCreationInput/responseOptions": ["Approve", "Reject"],
      "ApprovalCreationInput/title": "@concat('...')",
      "ApprovalCreationInput/assignedTo": "@{variables('SomeEmail')};",
      "ApprovalCreationInput/details": "@concat('...')",
      "ApprovalCreationInput/itemLink": "@{outputs('...')?['body/WebUrl']}",
      "ApprovalCreationInput/itemLinkDescription": "View item"
    },
    "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_approvals", "operationId": "CreateAnApproval", "connectionName": "shared_approvals" }
  }
},
"Wait_X_Approval": {
  "type": "OpenApiConnectionWebhook",
  "inputs": {
    "parameters": { "approvalName": "@outputs('Send_X_Approval')?['body/name']" },
    "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_approvals", "operationId": "WaitForAnApproval", "connectionName": "shared_approvals" }
  },
  "limit": { "timeout": "P14D" }
}
```
Notes: `approvalType` value is **`CustomResponse`** (singular, no 's' — this is what makes the dynamic schema resolve; `Basic` often doesn't). `assignedTo` is a semicolon-delimited list of emails/UPNs/AAD-ids — include a trailing `;`. `responseOptions` is an array. The approval connection ref in the CBAS env is `new_sharedapprovals_eca22`.

---

## OneDrive for Business — `shared_onedriveforbusiness`

| Action | operationId | Key params |
|---|---|---|
| Create file | `CreateFile` | `folderPath`, `name`, `body` (binary) |
| Get file content | `GetFileContent` | `id` (file id) |
| Get file content using path | `GetFileContentByPath` | `path` |
| Get file metadata | `GetFileMetadata` | `id` |
| Get file metadata using path | `GetFileMetadataByPath` | `path` |
| Update file | `UpdateFile` | `id`, `body` |
| Delete file | `DeleteFile` | `id` |
| List files in folder | `ListFolderV2` | `id` (folder id) |
| List files in root folder | `ListRootFolderV2` | (none) |
| Copy file | `CopyFileV2` | `source`, `destination`, `overwrite` |
| Move or rename file | `MoveFileV2` | `source`, `destination`, `overwrite` |
| Create share link | `CreateShareLinkV2` | `id` (file/folder id), `type` (`View`/`Edit`), `scope` (`Anonymous`/`Organization`); opt `expirationDateTime`, `password` |
| Extract archive to folder | `ExtractFolderV2` | `source`, `destination`, `overwrite` |
| Convert file | `ConvertFile` | `id`, `targetType` (e.g. `pdf`) |
| Convert file using path | `ConvertFileByPath` | `path`, `targetType` |

(OneDrive's create-share-link IS path-capable in spirit but the op takes a file/folder **id**; OneDrive has no list-item integer-id concept like SP. Get the id from `GetFileMetadataByPath` first.)

Triggers: `When a file is created` (`OnNewFileV2`), `When a file is modified` (`OnUpdatedFileV2`).

---

## Microsoft Forms — `shared_microsoftforms`

| Action | operationId | Key params |
|---|---|---|
| Get response details | `GetFormResponseById` | `form_id`, `response_id` |
| List responses | (use the trigger; there is no first-class "list all responses" action — `GetFormResponses` exists in some tenants but is unreliable. To poll, prefer storing submissions into SharePoint/Dataverse from a trigger-driven flow and polling that.) | |

Trigger: `When a new response is submitted` (`CreateFormWebhook`) — params: `form_id`. Output gives `resourceData/responseId`; chain `GetFormResponseById` to read answers. Form file-upload questions store files in the **form owner's OneDrive** automatically; the response value for that question is a JSON array of `{name, link, id, ...}` — parse it, then `GetFileContent` (OneDrive) on each, then upload elsewhere.

Prefill a Form via URL: append `&r<questionId>=<urlencoded value>` to the form's response URL.

---

## Built-in (no connector) actions

| Action | `type` | Notes |
|---|---|---|
| Compose | `Compose` | `inputs`: any expression/value |
| Initialize variable | `InitializeVariable` | `inputs.variables`: `[{name, type, value}]` — only at top level |
| Set variable | `SetVariable` | `inputs`: `{name, value}` |
| Increment variable | `IncrementVariable` | `{name, value}` |
| Append to array variable | `AppendToArrayVariable` | `{name, value}` |
| Condition | `If` | `expression`: `{and:[…]}` / `{or:[…]}` / leaf `{equals:[a,b]}` etc.; `actions`, `else.actions` |
| Switch | `Switch` | `expression`, `cases: {CaseName: {case: <value>, actions: {…}}}`, `default: {actions: {…}}` |
| Apply to each | `Foreach` | `foreach`: array expression; `actions` |
| Do until | `Until` | `expression`, `limit: {count, timeout}`, `actions` |
| Scope | `Scope` | `actions` — grouping + try/catch via runAfter on `Failed` |
| Terminate | `Terminate` | `inputs.runStatus`: `Succeeded`/`Failed`/`Cancelled`; opt `runError` |
| Parse JSON | `ParseJson` | `inputs.content`, `inputs.schema` |
| Select | `Select` | `inputs.from`, `inputs.select` |
| Filter array | `Query` | `inputs.from`, `inputs.where` |
| HTTP | `Http` | `inputs.method/uri/headers/body` — premium |
| Response | `Response` | for "When an HTTP request is received" trigger flows |
| Delay | `Wait` | `inputs.interval: {count, unit}` |
| Delay until | `Wait` | `inputs.until: {timestamp}` |

### Common trigger schemas
- **Manual (instant):** `{"manual": {"type": "Request", "kind": "Button", "inputs": {"schema": {"type": "object", "properties": {...}, "required": [...]}}, "metadata": {"operationMetadataId": "<guid>"}}}`
- **Recurrence:** `{"Recurrence": {"type": "Recurrence", "recurrence": {"frequency": "Day"|"Hour"|..., "interval": 1, "timeZone": "Greenwich Standard Time", "schedule": {"hours": [8], "minutes": [0]}}}}`
- **HTTP request received:** `{"manual": {"type": "Request", "kind": "Http", "inputs": {"schema": {...}}}}`
- **Connector trigger (e.g. SP/Forms):** `{"<TriggerName>": {"type": "OpenApiConnectionWebhook", "inputs": {"parameters": {...}, "host": {"apiId": "...", "operationId": "<trigger operationId>", "connectionName": "..."}}}}`

### WDL expression cheat-sheet (frequently misused)
- Date: `utcNow()`, `addDays(ts, n)`, `addHours/addMinutes(ts, n)`, `formatDateTime(ts, 'yyyy-MM-dd')`, `ticks(ts)`, `startOfDay(ts)`. To parse a possibly-messy date string before `ticks`, wrap: `ticks(formatDateTime(x, 'yyyy-MM-dd'))`.
- Null-safe: `item()?['Field']`, `coalesce(a, b, c)`, `empty(x)`, `if(cond, a, b)`.
- Strings: `concat(a, b, …)`, `replace(s, old, new)`, `toLower/toUpper`, `substring`, `split`, `trim`, `guid()`.
- Arrays: `first(arr)`, `last(arr)`, `length(arr)`, `union/intersection`, `contains(arr, x)`.
- References: `triggerBody()`, `triggerOutputs()?['body/Field']`, `outputs('ActionName')`, `body('ActionName')`, `variables('Name')`, `items('Foreach_Name')`. **A reference to `outputs('X')`/`body('X')` is only legal if X is upstream in the runAfter graph** — otherwise "invalid reference" at save time; fix by adding X to the action's `runAfter`.

---

## Official connector reference URLs (fetch with WebFetch when in doubt)

- SharePoint: https://learn.microsoft.com/en-us/connectors/sharepointonline/
- Office 365 Outlook: https://learn.microsoft.com/en-us/connectors/office365/
- Office 365 Users: https://learn.microsoft.com/en-us/connectors/office365users/
- Excel Online (Business): https://learn.microsoft.com/en-us/connectors/excelonlinebusiness/
- Approvals: https://learn.microsoft.com/en-us/connectors/approvals/
- OneDrive for Business: https://learn.microsoft.com/en-us/connectors/onedriveforbusiness/
- Microsoft Forms: https://learn.microsoft.com/en-us/connectors/microsoftforms/
- Full connector index: https://learn.microsoft.com/en-us/connectors/connector-reference/
- WDL expression functions: https://learn.microsoft.com/en-us/azure/logic-apps/workflow-definition-language-functions-reference
- Workflow definition language schema: https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-definition-language

**If a connector or operation isn't in this file, WebFetch its reference URL above, confirm the exact `operationId` and parameter keys, then add it here.**
