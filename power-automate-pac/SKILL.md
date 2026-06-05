---
name: power-automate-pac
description: Build, edit, and deploy Microsoft Power Automate cloud flows by exporting solutions with PAC CLI, editing the flow definition JSON directly on disk, repacking, and re-importing. Use this skill whenever the user wants to programmatically modify Power Automate flows without clicking through the designer. Pairs with the power-automate-connectors skill (the action catalog) — install and use both together.
version: 1.0.0
requires: power-automate-connectors
---

# Power Automate via PAC CLI and direct JSON editing

> ## ⛔ HARD RULE — READ BEFORE WRITING ANY ACTION JSON
>
> **This is a BLOCKING REQUIREMENT, not a suggestion.**
>
> Before you emit, edit, or paste ANY `OpenApiConnection` / `OpenApiConnectionWebhook`
> action (or any connector trigger), you **MUST** first invoke the
> `power-automate-connectors` skill via the Skill tool and confirm the exact
> `operationId`, its `parameters`, and the `apiId` against that authoritative
> catalog (sourced from official Microsoft Learn connector documentation).
>
> Rules — no exceptions:
> 1. **If the catalog does not list your `operationId`, the action DOES NOT EXIST. Do not write it.** Stop and tell the user the action is unavailable; propose the closest documented alternative.
> 2. **Never infer or guess** an `operationId`, parameter name, or `apiId` from naming patterns ("it's probably `UpdateItem`"), from another connector's precedent, or from training-data memory. Connector schemas drift; memory is stale.
> 3. **Every parameter key must come from the catalog**, including the flattened slash-keys (`item/Title`, `emailMessage/To`, `ApprovalCreationInput/title`, `parameters/path`, etc.). A wrong or invented parameter key passes JSON validation but breaks at save/runtime with cryptic schema errors.
> 4. **Re-verify on every edit**, not just on creation. If you change a connector action's operation, re-check the new operation's full parameter set.
> 5. When in doubt, **consult, don't assume.** Asking the catalog costs nothing; a hallucinated action costs an import cycle and a confusing error.
>
> Why this exists: inventing a plausible-but-nonexistent `operationId` or
> parameter produces flows that pack and import "successfully" yet fail at
> flow-save time inside Power Automate with errors like
> `... is no longer present in the operation schema` or
> `The inputs of action 'X' are not valid`. Those are expensive to diagnose.
> The only reliable prevention is to verify every action against the catalog
> before it is written.
>
> Workflow: at the START of any task that creates or edits flow actions,
> invoke `power-automate-connectors` and load the bundle(s) for the
> connector(s) you will touch (SharePoint, Outlook, Excel, OneDrive, Forms,
> Approvals, Dataverse, etc.). Keep that reference open while you build.

> ## ⛔ HARD RULE: STRICT XML TEMPLATE ENFORCEMENT
>
> **Never generate Dataverse solution XML structures from memory.** This rule
> applies to three file types: `Other/Solution.xml`, `Other/Customizations.xml`,
> and every `Workflows/*.json.data.xml`.
>
> Why: AI agents frequently hallucinate missing or incorrect tags, attribute
> names, attribute orderings, namespaces, and parent nodes when writing these
> XMLs freehand. PAC's `solution pack` accepts the file, the import even
> succeeds, but the flow then fails to render, save, or activate with errors
> that don't point at the malformed XML.
>
> Rules — no exceptions:
> 1. **To create one of these files: copy the matching boilerplate below
>    verbatim** and string-replace only the `ALL_CAPS_PLACEHOLDERS`. Do NOT
>    re-derive the structure from what XML "should look like."
> 2. **To modify one of these files: read the existing file first, then perform
>    a targeted Edit on the specific node** (e.g. add a new `<RootComponent>`
>    line, bump the `<Version>`). Do NOT regenerate the surrounding scaffold
>    from memory.
> 3. When a target solution already exists, **export, unpack, edit in place,
>    repack.** The exported XML from PAC is the authoritative shape for that
>    environment — prefer it over the boilerplates here.
> 4. The placeholders below are **the only fields you should be modifying.**
>    Every other tag, attribute, and value must remain as in the template
>    unless the user explicitly says otherwise.
>
> Sources for the boilerplates: cross-referenced from verified Dataverse
> exports in [`pnp/powerplatform-samples`](https://github.com/pnp/powerplatform-samples)
> (`leave-request`, `sharepoint-followsites`, `patch-tuesday` samples) and
> locally-imported, working solutions.
>
> ### Boilerplate A — `Other/Solution.xml`
>
> ```xml
> <?xml version="1.0" encoding="utf-8"?>
> <ImportExportXml version="9.2.24032.204" SolutionPackageVersion="9.2" languagecode="1033" generatedBy="CrmLive" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
>   <SolutionManifest>
>     <UniqueName>SOLUTION_UNIQUE_NAME</UniqueName>
>     <LocalizedNames>
>       <LocalizedName description="SOLUTION_DISPLAY_NAME" languagecode="1033" />
>     </LocalizedNames>
>     <Descriptions />
>     <Version>SOLUTION_VERSION</Version>
>     <Managed>MANAGED_FLAG</Managed>
>     <Publisher>
>       <UniqueName>PUBLISHER_UNIQUE_NAME</UniqueName>
>       <LocalizedNames>
>         <LocalizedName description="PUBLISHER_DISPLAY_NAME" languagecode="1033" />
>       </LocalizedNames>
>       <Descriptions />
>       <EMailAddress xsi:nil="true"></EMailAddress>
>       <SupportingWebsiteUrl xsi:nil="true"></SupportingWebsiteUrl>
>       <CustomizationPrefix>PUBLISHER_PREFIX</CustomizationPrefix>
>       <CustomizationOptionValuePrefix>PUBLISHER_OPTION_VALUE_PREFIX</CustomizationOptionValuePrefix>
>       <Addresses>
>         <Address>
>           <AddressNumber>1</AddressNumber>
>           <AddressTypeCode>1</AddressTypeCode>
>           <City xsi:nil="true"></City>
>           <County xsi:nil="true"></County>
>           <Country xsi:nil="true"></Country>
>           <Fax xsi:nil="true"></Fax>
>           <FreightTermsCode xsi:nil="true"></FreightTermsCode>
>           <ImportSequenceNumber xsi:nil="true"></ImportSequenceNumber>
>           <Latitude xsi:nil="true"></Latitude>
>           <Line1 xsi:nil="true"></Line1>
>           <Line2 xsi:nil="true"></Line2>
>           <Line3 xsi:nil="true"></Line3>
>           <Longitude xsi:nil="true"></Longitude>
>           <Name xsi:nil="true"></Name>
>           <PostalCode xsi:nil="true"></PostalCode>
>           <PostOfficeBox xsi:nil="true"></PostOfficeBox>
>           <PrimaryContactName xsi:nil="true"></PrimaryContactName>
>           <ShippingMethodCode>1</ShippingMethodCode>
>           <StateOrProvince xsi:nil="true"></StateOrProvince>
>           <Telephone1 xsi:nil="true"></Telephone1>
>           <Telephone2 xsi:nil="true"></Telephone2>
>           <Telephone3 xsi:nil="true"></Telephone3>
>           <TimeZoneRuleVersionNumber xsi:nil="true"></TimeZoneRuleVersionNumber>
>           <UPSZone xsi:nil="true"></UPSZone>
>           <UTCOffset xsi:nil="true"></UTCOffset>
>           <UTCConversionTimeZoneCode xsi:nil="true"></UTCConversionTimeZoneCode>
>         </Address>
>         <Address>
>           <AddressNumber>2</AddressNumber>
>           <AddressTypeCode>1</AddressTypeCode>
>           <City xsi:nil="true"></City>
>           <County xsi:nil="true"></County>
>           <Country xsi:nil="true"></Country>
>           <Fax xsi:nil="true"></Fax>
>           <FreightTermsCode xsi:nil="true"></FreightTermsCode>
>           <ImportSequenceNumber xsi:nil="true"></ImportSequenceNumber>
>           <Latitude xsi:nil="true"></Latitude>
>           <Line1 xsi:nil="true"></Line1>
>           <Line2 xsi:nil="true"></Line2>
>           <Line3 xsi:nil="true"></Line3>
>           <Longitude xsi:nil="true"></Longitude>
>           <Name xsi:nil="true"></Name>
>           <PostalCode xsi:nil="true"></PostalCode>
>           <PostOfficeBox xsi:nil="true"></PostOfficeBox>
>           <PrimaryContactName xsi:nil="true"></PrimaryContactName>
>           <ShippingMethodCode>1</ShippingMethodCode>
>           <StateOrProvince xsi:nil="true"></StateOrProvince>
>           <Telephone1 xsi:nil="true"></Telephone1>
>           <Telephone2 xsi:nil="true"></Telephone2>
>           <Telephone3 xsi:nil="true"></Telephone3>
>           <TimeZoneRuleVersionNumber xsi:nil="true"></TimeZoneRuleVersionNumber>
>           <UPSZone xsi:nil="true"></UPSZone>
>           <UTCOffset xsi:nil="true"></UTCOffset>
>           <UTCConversionTimeZoneCode xsi:nil="true"></UTCConversionTimeZoneCode>
>         </Address>
>       </Addresses>
>     </Publisher>
>     <RootComponents>
>       <!-- Repeat one <RootComponent> line per component the solution carries. -->
>       <!-- type values: 1=Entity, 29=Workflow (cloud flow), 60=SystemForm, 61=WebResource, 62=AppModule, 80=SiteMap, 300=AppTemplate -->
>       <!-- For workflows/cloud flows the id is the WorkflowId GUID in lowercase, with curly braces. -->
>       <RootComponent type="29" id="{LOWERCASE_FLOW_GUID}" behavior="0" />
>     </RootComponents>
>     <MissingDependencies />
>   </SolutionManifest>
> </ImportExportXml>
> ```
>
> Placeholder reference (Boilerplate A):
> - `SOLUTION_UNIQUE_NAME` — internal name, no spaces, e.g. `LeaveRequestApproval`
> - `SOLUTION_DISPLAY_NAME` — human-readable, e.g. `Leave Request Approval`
> - `SOLUTION_VERSION` — `MAJOR.MINOR.BUILD.REVISION`, e.g. `1.0.0.0`. Bump on each redeploy.
> - `MANAGED_FLAG` — `0` unmanaged, `1` managed, `2` template/holding
> - `PUBLISHER_UNIQUE_NAME` — e.g. `Alvin`
> - `PUBLISHER_DISPLAY_NAME` — e.g. `Alvin`
> - `PUBLISHER_PREFIX` — lowercase 2-8 chars, e.g. `alv`
> - `PUBLISHER_OPTION_VALUE_PREFIX` — 5-digit integer assigned to the publisher (visible in the maker portal under Publisher details)
> - `LOWERCASE_FLOW_GUID` — the flow's `WorkflowId` GUID, lowercase, no curly braces
>
> ### Boilerplate B — `Other/Customizations.xml`
>
> ```xml
> <?xml version="1.0" encoding="utf-8"?>
> <ImportExportXml xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
>   <Entities />
>   <Roles />
>   <Workflows />
>   <FieldSecurityProfiles />
>   <Templates />
>   <EntityMaps />
>   <EntityRelationships />
>   <OrganizationSettings />
>   <optionsets />
>   <CustomControls />
>   <EntityDataProviders />
>   <connectionreferences>
>     <!-- Repeat one <connectionreference> block per connection reference shipped IN this solution. -->
>     <connectionreference connectionreferencelogicalname="CONNREF_LOGICAL_NAME">
>       <connectionreferencedisplayname>CONNREF_DISPLAY_NAME</connectionreferencedisplayname>
>       <connectorid>/providers/Microsoft.PowerApps/apis/CONNECTOR_API_NAME</connectorid>
>       <iscustomizable>1</iscustomizable>
>       <statecode>0</statecode>
>       <statuscode>1</statuscode>
>     </connectionreference>
>   </connectionreferences>
>   <Languages>
>     <Language>1033</Language>
>   </Languages>
> </ImportExportXml>
> ```
>
> Placeholder reference (Boilerplate B):
> - `CONNREF_LOGICAL_NAME` — Dataverse schema name, prefixed with the publisher prefix, e.g. `alv_SharedApprovals` or `alv_sharedoffice365_abcde`. Casing is preserved by Dataverse — pick once and stick with it.
> - `CONNREF_DISPLAY_NAME` — human-readable, e.g. `Standard Approvals`
> - `CONNECTOR_API_NAME` — `shared_<connector>` value such as `shared_approvals`, `shared_office365`, `shared_office365users`, `shared_microsoftforms`, `shared_sharepointonline`, `shared_onedriveforbusiness`, `shared_excelonlinebusiness`. Verify against the `power-automate-connectors` skill.
> - `<Languages><Language>1033</Language></Languages>` — `1033` = English (US). Optional but standard.
>
> ### Boilerplate C — `Workflows/<NAME>-<UPPERCASE_GUID>.json.data.xml`
>
> ```xml
> <?xml version="1.0" encoding="utf-8"?>
> <Workflow WorkflowId="{LOWERCASE_FLOW_GUID}" Name="FLOW_DISPLAY_NAME" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
>   <JsonFileName>/Workflows/FLOW_FILE_NAME-UPPERCASE_FLOW_GUID.json</JsonFileName>
>   <Type>1</Type>
>   <Subprocess>0</Subprocess>
>   <Category>5</Category>
>   <Mode>0</Mode>
>   <Scope>4</Scope>
>   <OnDemand>0</OnDemand>
>   <TriggerOnCreate>0</TriggerOnCreate>
>   <TriggerOnDelete>0</TriggerOnDelete>
>   <AsyncAutodelete>0</AsyncAutodelete>
>   <SyncWorkflowLogOnFailure>0</SyncWorkflowLogOnFailure>
>   <StateCode>STATE_CODE</StateCode>
>   <StatusCode>STATUS_CODE</StatusCode>
>   <RunAs>1</RunAs>
>   <IsTransacted>1</IsTransacted>
>   <IntroducedVersion>1.0</IntroducedVersion>
>   <IsCustomizable>1</IsCustomizable>
>   <BusinessProcessType>0</BusinessProcessType>
>   <IsCustomProcessingStepAllowedForOtherPublishers>1</IsCustomProcessingStepAllowedForOtherPublishers>
>   <ModernFlowType>0</ModernFlowType>
>   <PrimaryEntity>none</PrimaryEntity>
>   <LocalizedNames>
>     <LocalizedName languagecode="1033" description="FLOW_DISPLAY_NAME" />
>   </LocalizedNames>
> </Workflow>
> ```
>
> Placeholder reference (Boilerplate C):
> - `LOWERCASE_FLOW_GUID` — the flow's GUID, lowercase, with curly braces in `WorkflowId="{...}"`
> - `FLOW_DISPLAY_NAME` — human-readable, e.g. `Leave Request Approval Flow`
> - `FLOW_FILE_NAME` — the JSON filename's stem (no spaces), e.g. `LeaveRequestApprovalFlow`
> - `UPPERCASE_FLOW_GUID` — the same GUID, uppercase, no braces — must match the actual `.json` filename's GUID
> - `STATE_CODE` / `STATUS_CODE`:
>   - `0` / `1` → **Draft** (flow imports disabled; safe default for test deployments)
>   - `1` / `2` → **Activated** (flow imports running)
> - Constants (do not change unless you know why): `Type=1`, `Category=5`, `Mode=0`, `Scope=4`, `RunAs=1`, `IsTransacted=1`, `ModernFlowType=0`, `BusinessProcessType=0`, `PrimaryEntity=none`. These hold for standard cloud flows.

This skill teaches you the full lifecycle: authenticate → export solution → unpack → edit flow JSON → pack → import. It also documents the JSON schema for triggers, actions, runAfter chains, scope rules, connection references, expressions, and every gotcha worth knowing.

**See also:** the `power-automate-connectors` skill is the authoritative action catalog and MUST be used alongside this skill (see the HARD RULE above). This skill covers the *workflow and JSON structure*; that skill covers *which actions and parameters actually exist*.

The workflow assumes:
- Flows live inside a **Dataverse solution** (managed or unmanaged) — non-solution "personal" flows cannot be exported through PAC CLI. If a flow isn't in a solution, the user must add it via the maker portal first (`https://make.powerautomate.com → Solutions → <solution> → + Add existing → Cloud flow → From outside Dataverse`).
- PAC CLI is installed (`pac install latest` from a PowerShell prompt; or via .NET tool / MSI).
- The user has a Power Platform environment they can read/write.

---

## 1. PAC CLI authentication and environment selection

### 1.1 Auth profile

```powershell
pac auth create --environment <Env display name OR GUID OR URL>
```
- Opens a browser, signs you in interactively.
- Creates a default named profile.
- Subsequent `pac` commands use whichever profile is `[*]` in `pac auth list`.

To pin a profile non-interactively (CI scenarios), use a service principal:
```powershell
pac auth create --applicationId <appId> --clientSecret <secret> --tenant <tenantId> --environment <url>
```

Inspect:
```powershell
pac auth list             # show all profiles, * = active
pac auth select --index N # switch active profile
pac auth delete --index N
```

### 1.2 Environment commands

```powershell
pac env list                  # list environments visible to current user
pac env select --environment <name|guid|url>
pac env who                   # confirm current org URL, user, env
```
Most `pac solution` commands accept `--environment` to override the active env for one command.

---

## 2. Exporting and unpacking solutions

### 2.1 Find the solution

```powershell
pac solution list
```
Each row shows `Unique Name`, friendly name, version, managed/unmanaged. Use the **Unique Name** in all subsequent commands.

### 2.2 Export

```powershell
pac solution export `
  --name <UniqueName> `
  --path C:\path\Solution.zip `
  --overwrite
```
- Default export is **unmanaged** (editable); add `--managed` for managed.
- Add `--include general,customization,emailtemplates,calendar,isvconfig` if you want extras (rarely needed for flows).
- The exported zip is a sealed archive of XML + JSON describing every component.

### 2.3 Unpack the zip into a source tree

```powershell
pac solution unpack `
  --zipfile C:\path\Solution.zip `
  --folder  C:\path\Solution_SRC `
  --allowDelete
```
- `--allowDelete` lets it overwrite/remove existing files in the target folder.
- The unpacked folder is what you commit to git. **The zip is a build artifact; the unpacked folder is the source of truth.**

You can also add `--processCanvasApps` but it's deprecated and Canvas Apps unpack separately now.

---

## 3. Unpacked solution folder layout

After unpacking, a flow-bearing solution has roughly this shape:

```
Solution_SRC/
├── Other/
│   ├── Solution.xml          # solution manifest — version, publisher, root components
│   ├── Customizations.xml    # large XML of every customization
│   └── Relationships.xml
├── Workflows/
│   ├── MyFlow-<UPPER-GUID>.json         # flow definition (properties.definition + connectionReferences)
│   ├── MyFlow-<UPPER-GUID>.json.data.xml # flow metadata (StateCode, Name, etc.)
│   ├── AnotherFlow-<UPPER-GUID>.json
│   └── AnotherFlow-<UPPER-GUID>.json.data.xml
├── Entities/                 # only present if the solution carries Dataverse tables
│   └── <tableName>/...
├── connectionreferences/     # only present if the solution carries connection references
│   └── <logicalName>.json
├── environmentvariabledefinitions/
│   └── <schemaName>/...
├── PluginAssemblies/
├── WebResources/
├── Roles/
└── ... (many more folders, mostly empty for flow-only solutions)
```

### 3.1 `Other/Solution.xml`

The solution manifest. Key parts:
- `<UniqueName>` — never edit
- `<Version>` — bump this when you redeploy (e.g. `1.0.0.4` → `1.0.0.5`); not strictly required for unmanaged import but good practice
- `<Managed>` — `0` unmanaged, `1` managed
- `<Publisher>` — has `<CustomizationPrefix>` (the prefix on custom schema names, e.g. `contoso`)
- `<RootComponents>` — every component the solution carries, by GUID + type. **Adding a new flow requires adding a new `<RootComponent type="29" id="{lowercase-guid}" behavior="0" />` line here.** Type 29 = workflow.

> **To create or modify this file, use [Boilerplate A](#boilerplate-a--othersolutionxml) per the STRICT XML TEMPLATE ENFORCEMENT hard rule. Do not write the file structure from memory.**

### 3.2 `Workflows/<Name>-<GUID>.json` — the flow definition

The outer wrapper:
```json
{
  "properties": {
    "connectionReferences": { ... },
    "definition": { ... },
    "templateName": null
  },
  "schemaVersion": "1.0.0.0"
}
```

### 3.3 `Workflows/<Name>-<GUID>.json.data.xml` — flow metadata

Per-flow XML carrying the Dataverse workflow record properties. Key fields:
- `WorkflowId="{lower-guid}"` — must match the JSON filename's GUID (filename is UPPERCASE; XML braces use lowercase)
- `Name="..."` — display name shown in the maker portal
- `JsonFileName="/Workflows/<Name>-<UPPERGUID>.json"`
- `<StateCode>` and `<StatusCode>`:
  - `0` / `1` → Draft / Off (flow is disabled, won't fire on triggers)
  - `1` / `2` → Activated / On
- `<Type>1`, `<Category>5`, `<Mode>0`, `<Scope>4`, `<RunAs>1` — for cloud flows leave at these defaults
- `<ModernFlowType>0`, `<PrimaryEntity>none` — for non-Dataverse-triggered cloud flows
- `<IsCustomizable>1`, `<IsCustomProcessingStepAllowedForOtherPublishers>1` — leave true unless deliberately locking

> **To create this file, use [Boilerplate C](#boilerplate-c--workflowsname-uppercase_guidjsondataxml) per the STRICT XML TEMPLATE ENFORCEMENT hard rule. Do not write the file structure from memory.**

### 3.4 `connectionreferences/<logicalName>.json`

Only present for connection references defined IN this solution. Their schema name (e.g. `<prefix>_SharePointConnRef`) is what the flow's `connectionReferenceLogicalName` field points at. Note: Dataverse **preserves the exact casing** of the schema name once created (including camelCase) — don't rename casually.

> Connection references are declared inside `Other/Customizations.xml`'s `<connectionreferences>` block — **use [Boilerplate B](#boilerplate-b--othercustomizationsxml) per the STRICT XML TEMPLATE ENFORCEMENT hard rule. Do not write the file structure from memory.**

---

## 4. The flow definition JSON in depth

### 4.1 `properties.connectionReferences` (the wrapper)

```json
"connectionReferences": {
  "shared_sharepointonline": {
    "runtimeSource": "embedded",
    "connection": {
      "connectionReferenceLogicalName": "<prefix>_SharePointConnRef"
    },
    "api": {
      "name": "shared_sharepointonline"
    }
  },
  "shared_office365": { ... }
}
```

- The KEY (`shared_sharepointonline`) is the **logical alias used inside `host.connectionName`** of every action.
- `connectionReferenceLogicalName` is the Dataverse schema name of the connection reference record (must already exist in the solution / environment).
- `api.name` must match the connector's swagger name (`shared_sharepointonline`, `shared_office365`, `shared_excelonlinebusiness`, `shared_onedriveforbusiness`, `shared_microsoftforms`, `shared_approvals`, `shared_office365users`, `shared_cloudmersiveconvert`, etc.).
- Add a new connector here whenever you add an action that uses it — otherwise the import succeeds but the action errors at save-time with `Property 'host.connectionReferenceName' is missing` or similar.
- Some flows use suffixed keys (e.g. `shared_office365-1`) when the same connector is needed with two different connections. The suffix is arbitrary; the value must still be in your connectionReferences map.

### 4.2 `properties.definition` (the workflow definition)

```json
"definition": {
  "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "$authentication": { "defaultValue": {}, "type": "SecureObject" },
    "$connections":    { "defaultValue": {}, "type": "Object" }
  },
  "triggers": { ... },
  "actions":  { ... },
  "outputs":  {},
  "description": "Optional but required by some update paths"
}
```

- The `$schema` and `contentVersion` are mandatory and verbatim.
- `parameters` block must include the two `$authentication` and `$connections` parameters.
- `description` is optional in storage but is required by the live update-flow API path.

### 4.3 Triggers

Triggers go in `definition.triggers`. Common shapes:

**Microsoft Forms response (webhook):**
```json
"When_a_new_response_is_submitted": {
  "type": "OpenApiConnectionWebhook",
  "inputs": {
    "parameters": { "form_id": "<formId>" },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_microsoftforms",
      "operationId": "CreateFormWebhook",
      "connectionName": "shared_microsoftforms"
    }
  }
}
```
The webhook's body delivered to the flow is:
```json
{ "value": [ { "webhookId": "...", "eventType": "ResponseAdded",
               "resourceData": { "formId": "...", "responseId": <int> },
               "eventTime": "..." } ] }
```
So inside actions you read the responseId with:
```
@first(triggerOutputs()?['body/value'])?['resourceData']?['responseId']
```

**Recurrence:**
```json
"Recurrence": {
  "type": "Recurrence",
  "recurrence": { "frequency": "Day", "interval": 1, "startTime": "2026-01-01T08:00:00Z" }
}
```

**HTTP request:**
```json
"manual": {
  "type": "Request",
  "kind": "Http",
  "inputs": { "schema": { "type": "object", "properties": { ... } } }
}
```

**Button trigger (Power Apps):** `type: "Request", kind: "Button"`.

**Dataverse "When a row is added/modified":** OpenApiConnectionWebhook against `shared_commondataserviceforapps`.

### 4.4 Actions

The general shape of any action:
```json
"<Action_Name>": {
  "type": "OpenApiConnection",
  "inputs": { "parameters": { ... }, "host": { "apiId": "...", "operationId": "...", "connectionName": "..." } },
  "runAfter": { "<UpstreamAction>": [ "Succeeded" ] }
}
```

Action **name keys** must be valid JSON keys, **case sensitive**, and **referenced exactly** in `outputs('Name')`, `body('Name')`, and `runAfter`. Use underscores or hyphens; the designer auto-replaces spaces with underscores.

Common action `type` values:
| type | Purpose |
|---|---|
| `OpenApiConnection` | Most connector actions (SP, Excel, OneDrive, Outlook, Forms, Approvals, etc.) |
| `OpenApiConnectionWebhook` | Webhook-style triggers and the `WaitForAnApproval` action |
| `Compose` | Set/return an arbitrary value: `"inputs": "@<expression>"` |
| `InitializeVariable` | Declare a variable (only valid at top level, before any nested scope) |
| `SetVariable` / `IncrementVariable` / `DecrementVariable` / `AppendToStringVariable` / `AppendToArrayVariable` | Variable ops |
| `If` | Conditional with `expression`, `actions`, `else.actions` |
| `Switch` | Multi-branch with `expression`, `cases.<caseName>.{case, actions}`, `default.actions` |
| `Until` | Do-until loop with `expression`, `limit: {count, timeout}`, `actions` |
| `Foreach` | Loop with `foreach: "@<arrayExpr>"`, `actions` |
| `Scope` | Plain container that groups actions and exposes one overall status |
| `Terminate` | End the run early. `inputs: { runStatus: "Succeeded"|"Cancelled"|"Failed", runError: {...} }` |
| `Wait` | Delay action: `inputs: { interval: { count: N, unit: "Minute"|"Hour"|"Day"|"Second" } }` |
| `Http` | Premium HTTP action |
| `ApiConnection` | Older connector format (avoid for new actions) |

### 4.5 `runAfter` — the dependency graph

Every action except the very first in a scope has `runAfter`:
```json
"runAfter": {
  "<PredecessorAction>": [ "Succeeded" ],
  "<AnotherPredecessor>": [ "Failed", "TimedOut" ]
}
```
- The KEY is the upstream action name.
- The VALUE is an array of acceptable upstream statuses: `"Succeeded"`, `"Failed"`, `"Skipped"`, `"TimedOut"`.
- Empty `runAfter: {}` = "I'm the first action in my scope; run me when the scope starts."
- Multiple keys in `runAfter` = AND (all predecessors must finish with one of the listed statuses).
- To express OR, you typically chain through a `Scope` and check its overall status.

**Scope rule (this trips everyone up):** `runAfter` may ONLY reference actions in the **same scope** (same parent `actions` dict). You cannot put a top-level action in the `runAfter` of an action nested inside a Switch case.

### 4.6 Cross-scope output references (different from runAfter)

You CAN reference outputs of actions in a different scope (`outputs('X')`, `body('X')`), but only if `X` is "upstream in execution order." The runtime/designer enforces this by checking that the scope containing your current action eventually depends (transitively) on the scope containing `X`.

Example: an action inside `Switch.cases.Approve.actions` can read `outputs('Log_Row_to_X')` where `Log_Row_to_X` is at the top level, **only if** the Switch's `runAfter` chain eventually depends on `Log_Row_to_X` (or a scope containing it). If not, you get `InvalidTemplate: cannot reference action 'X' because it must be in the runAfter path or in a scope action on the runAfter path`.

**Fix pattern:** when adding a new top-level action that nested actions need to reference, find the pivot — the scope action (`Until`, `Switch`, `If`, etc.) whose `runAfter` will be the carrier — and add your new action to that pivot's `runAfter`. Don't try to add cross-scope deps on the inner actions directly; that yields a different error: `must belong to same level`.

### 4.7 Connection references inside actions

In every connector action's `host` block:
```json
"host": {
  "apiId": "/providers/Microsoft.PowerApps/apis/shared_<api>",
  "operationId": "<OpId>",
  "connectionName": "<key from properties.connectionReferences>"
}
```

**`connectionName` is the right key for modern flows.** Common confusion:
- Older / legacy designer code may use `connection` (string) — works but deprecated.
- Some surfaced examples show `connectionReferenceName` — that's the WRONG modern key. If you see save errors saying `host.connectionReferenceName' is missing` or that it is `Null`, you've got the wrong key. Switch to `connectionName`.

### 4.8 Parameter flattening

Many connector parameters use a path-flattened key with slashes:
| Connector / op | Key pattern |
|---|---|
| Outlook `SendEmailV2` | `emailMessage/To`, `emailMessage/Subject`, `emailMessage/Body`, `emailMessage/Importance`, `emailMessage/Attachments`, etc. |
| Approvals `CreateAnApproval` | `ApprovalCreationInput/title`, `ApprovalCreationInput/assignedTo`, `ApprovalCreationInput/responseOptions[N]`, `ApprovalCreationInput/details`, `ApprovalCreationInput/itemLink` |
| SharePoint `CreateNewFolder` | `parameters/path` |
| SharePoint `CreateSharingLink` | `permission/type`, `permission/scope`, `permission/expirationDateTime` |
| SharePoint `PostItem` / `PatchItem` | `item/<columnInternalName>` (e.g. `item/Title`, `item/field_1`) |
| OneDrive `CreateFile` | `folderPath`, `name`, `body` |
| OneDrive `GetFileContentByPath` | `path` |
| Excel `GetItem` | `source`, `drive`, `file`, `table`, `idColumn`, `id` |
| Excel `PatchItem` | `source`, `drive`, `file`, `table`, `idColumn`, `id`, `item/<col>` |

Crucial: for Excel `dataset`/`table` and SharePoint `dataset`/`table`, the values **must be literal strings (the URL and GUID)** — not `@outputs(...)` references. The designer will throw `item/X is no longer present in the operation schema` if you try to make them dynamic.

### 4.9 SharePoint OData filters (`$filter` and friends)

Build at runtime with `concat`:
```
"$filter": "@concat('field_1 eq ''', variables('Department'), ''' and field_4 eq ''Manager''')"
```
- Two single quotes inside a single-quoted OData string = escaped single quote.
- Use column **internal** names in the filter (often `field_1`, `field_2`, ... when columns were added via the UI; the display names you see are aliases).
- Pair with `$top: 1` for "find me one row." Result is in `body/value` (array).
- To consume in downstream actions: `@first(outputs('Get_X')?['body/value'])?['<internalColumn>']`.

### 4.10 Expressions and dynamic content (`@` syntax)

Any string starting with `@` (or containing `@{...}` inside a larger string) is evaluated:
- Bare expression: `"inputs": "@guid()"` → output is the result of `guid()`.
- Inline interpolation: `"inputs": "Hello @{variables('Name')}!"` → string with the variable spliced in.
- Escape a literal `@`: write `@@`.

Common functions you'll use constantly:
| Function | Purpose |
|---|---|
| `outputs('Action')` | full outputs object of action; usually you want `outputs('A')?['body']` or `outputs('A')?['body/Foo']` |
| `body('Action')` | shortcut for `outputs('Action')?['body']` |
| `triggerOutputs()` / `triggerBody()` | trigger's outputs / body |
| `variables('Name')` | read a variable |
| `parameters('Name')` | read a definition parameter (rare in cloud flows) |
| `items('Foreach_Action')` | current iteration value inside a Foreach |
| `item()` / `items()` | shorthand for the current item inside the nearest Foreach |
| `iterationIndexes('Loop')` | current index in nested loops |
| `concat`, `replace`, `substring`, `length`, `toLower`, `toUpper`, `trim`, `split`, `join` | string ops |
| `equals`, `greater`, `less`, `and`, `or`, `not`, `if`, `coalesce` | logic |
| `first`, `last`, `take`, `skip`, `union`, `intersection`, `contains`, `empty` | collection ops |
| `json`, `string`, `int`, `float`, `bool`, `array`, `createArray` | conversions |
| `utcNow`, `addDays`, `addHours`, `addMinutes`, `formatDateTime`, `convertTimeZone`, `ticks` | dates |
| `guid()` | new GUID |
| `base64`, `base64ToString`, `dataUri`, `dataUriToBinary` | encoding |

The `?` operator on `[...]` access is null-safe — `outputs('A')?['body/foo']` returns null if `body` or `foo` doesn't exist, rather than failing.

### 4.11 Inline strings vs expressions

When a parameter value is a **string** in the JSON, prefix the entire value with `@` to make it an expression:
```json
"inputs": "@variables('Name')"
```
When you need a string with embedded expressions, use `@{...}` inline:
```json
"inputs": "Dear @{variables('Name')}, your code is @{outputs('Compose_Code')}"
```
The two are NOT interchangeable; using `@variables(...)` inside a longer string will be sent literally instead of evaluated.

---

## 5. Building flows from scratch

### 5.1 Minimal end-to-end skeleton

```json
{
  "properties": {
    "connectionReferences": {
      "shared_office365": {
        "runtimeSource": "embedded",
        "connection": { "connectionReferenceLogicalName": "<your_conn_ref_logical_name>" },
        "api":        { "name": "shared_office365" }
      }
    },
    "definition": {
      "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "$authentication": { "defaultValue": {}, "type": "SecureObject" },
        "$connections":    { "defaultValue": {}, "type": "Object" }
      },
      "triggers": {
        "Manually_trigger_a_flow": {
          "type": "Request",
          "kind": "Button",
          "inputs": {}
        }
      },
      "actions": {
        "Send_Email": {
          "type": "OpenApiConnection",
          "inputs": {
            "parameters": {
              "emailMessage/To": "someone@example.com",
              "emailMessage/Subject": "Hello",
              "emailMessage/Body": "<p>Hi from a freshly built flow.</p>"
            },
            "host": {
              "apiId":         "/providers/Microsoft.PowerApps/apis/shared_office365",
              "operationId":   "SendEmailV2",
              "connectionName": "shared_office365"
            }
          },
          "runAfter": {}
        }
      },
      "outputs": {}
    },
    "templateName": null
  },
  "schemaVersion": "1.0.0.0"
}
```

Pair it with a `.json.data.xml` — **per the STRICT XML TEMPLATE ENFORCEMENT hard rule, do NOT write this from memory. Copy [Boilerplate C](#boilerplate-c--workflowsname-uppercase_guidjsondataxml) verbatim** and string-replace the placeholders (`LOWERCASE_FLOW_GUID`, `FLOW_DISPLAY_NAME`, `FLOW_FILE_NAME`, `UPPERCASE_FLOW_GUID`, `STATE_CODE`/`STATUS_CODE`). For first deploys, use `STATE_CODE=0` and `STATUS_CODE=1` so the flow imports as Draft and doesn't fire until you turn it on.

Register the flow in the solution — **do NOT regenerate `Other/Solution.xml` from scratch.** If the file already exists (e.g. you exported and unpacked an existing solution), perform a targeted Edit to add one new `<RootComponent>` line inside the existing `<RootComponents>` block:
```xml
<RootComponent type="29" id="{LOWERCASE_FLOW_GUID}" behavior="0" />
```
If you are creating a brand-new solution and no `Other/Solution.xml` exists yet, copy [Boilerplate A](#boilerplate-a--othersolutionxml) verbatim and string-replace its placeholders. Same rule applies to `Other/Customizations.xml` — use [Boilerplate B](#boilerplate-b--othercustomizationsxml) and add one `<connectionreference>` block per connection reference shipping in the solution.

### 5.2 Parallel branches

Two siblings with the same `runAfter` execute in parallel:
```json
"actions": {
  "A": { "runAfter": {},  ... },
  "B": { "runAfter": {"A": ["Succeeded"]}, ... },
  "C": { "runAfter": {"A": ["Succeeded"]}, ... },
  "D": { "runAfter": {"B": ["Succeeded"], "C": ["Succeeded"]}, ... }
}
```
A → fan out to B and C in parallel → D joins after both.

### 5.3 Approval chain pattern

The standard pattern is a two-action split: send + wait. This lets actions run between the request and the response, and the wait can be cancelled.
```json
"Send_Approval": {
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "approvalType": "CustomResponse",
      "ApprovalCreationInput/responseOptions": ["Approve", "Reject"],
      "ApprovalCreationInput/title":          "@concat('Approval for ', variables('AppName'))",
      "ApprovalCreationInput/assignedTo":     "user@contoso.com",
      "ApprovalCreationInput/details":        "<markdown body>",
      "ApprovalCreationInput/itemLink":       ""
    },
    "host": {
      "apiId":         "/providers/Microsoft.PowerApps/apis/shared_approvals",
      "operationId":   "CreateApproval",
      "connectionName": "shared_approvals"
    }
  }
},
"Wait_For_Approval": {
  "type": "OpenApiConnectionWebhook",
  "inputs": {
    "parameters": { "approvalId": "@outputs('Send_Approval')?['body/name']" },
    "host": {
      "apiId":         "/providers/Microsoft.PowerApps/apis/shared_approvals",
      "operationId":   "WaitForAnApproval",
      "connectionName": "shared_approvals"
    }
  },
  "runAfter": { "Send_Approval": ["Succeeded"] }
}
```

After the Wait, the outcome is `body('Wait_For_Approval')?['outcome']` (e.g. `"Approve"`, `"Reject"`, or any custom response option). Switch on it for branching.

### 5.4 Do Until loop

```json
"Poll_Until_Done": {
  "type": "Until",
  "expression": "@equals(outputs('Get_Status')?['body/status'], 'Done')",
  "limit": { "count": 60 },
  "actions": {
    "Delay_30s": {
      "type": "Wait",
      "inputs": { "interval": { "count": 30, "unit": "Second" } },
      "runAfter": {}
    },
    "Get_Status": {
      "type": "OpenApiConnection",
      "inputs": { ... },
      "runAfter": { "Delay_30s": ["Succeeded"] }
    }
  },
  "runAfter": { "Kick_Off_Job": ["Succeeded"] }
}
```
- `expression` is evaluated **after** each iteration. When it returns true, the loop exits.
- `limit.count` is the max iterations.
- `limit.timeout` is ISO 8601 (e.g. `P1D` = 1 day, `PT30M` = 30 minutes). **Hard cap: workflow lifetime, which is 30 days (`P30D`)**. Setting `P60D` errors with `WorkflowRunActionTimeoutLimitInvalid`. If you don't need a timeout, omit the field entirely.

### 5.5 Switch / If

```json
"Decide": {
  "type": "Switch",
  "expression": "@outputs('Get_Status')?['body/outcome']",
  "cases": {
    "Approve":  { "case": "Approve",  "actions": { ... } },
    "Reject":   { "case": "Reject",   "actions": { ... } }
  },
  "default": { "actions": { "Terminate_Run": { "type": "Terminate", "inputs": {"runStatus":"Failed"}, "runAfter": {} } } },
  "runAfter": { "Get_Status": ["Succeeded"] }
}
```

```json
"If_Found": {
  "type": "If",
  "expression": { "greater": [ "@length(outputs('Lookup')?['body/value'])", 0 ] },
  "actions": { "Do_Thing": { ... } },
  "else":    { "actions": { "Other_Thing": { ... } } },
  "runAfter": { "Lookup": ["Succeeded"] }
}
```
Note `If.expression` uses the OData-style structured comparator (e.g. `{ "equals": [a,b] }`, `{ "greater": [...] }`), whereas `Switch.expression` is a bare value expression.

### 5.6 Variables

Variables are declared at the **top level only** with `InitializeVariable`, **before** any nested scope:
```json
"Init_Counter": {
  "type": "InitializeVariable",
  "inputs": { "variables": [ { "name": "Counter", "type": "integer", "value": 0 } ] },
  "runAfter": {}
}
```
Types: `boolean`, `integer`, `float`, `string`, `object`, `array`. Set/append later with `SetVariable`, `IncrementVariable`, etc. — those CAN be in nested scopes.

### 5.7 Foreach

```json
"For_Each_File": {
  "type": "Foreach",
  "foreach": "@outputs('Get_Files')?['body/value']",
  "actions": {
    "Process_File": {
      "type": "OpenApiConnection",
      "inputs": { "parameters": { "id": "@items('For_Each_File')?['id']" }, ... },
      "runAfter": {}
    }
  },
  "runAfter": { "Get_Files": ["Succeeded"] }
}
```
Inside the loop, `items('For_Each_File')` is the current item. For deeply nested loops use `item()` (innermost) or `items('OuterLoopName')`.

Default concurrency: parallel up to 20. Force serial:
```json
"runtimeConfiguration": { "concurrency": { "repetitions": 1 } }
```

---

## 6. Editing existing flows

### 6.1 Workflow

1. Export & unpack the solution (Section 2).
2. **Initialize git** in the unpacked source folder if you haven't:
   ```powershell
   cd Solution_SRC
   git init -b main
   git add -A
   git commit -m "baseline export <date>"
   ```
   Every subsequent edit becomes a commit. Easy rollback.
3. Edit `Workflows/<Flow>-<GUID>.json` directly. Almost always you're touching `properties.definition.actions`.
4. Validate locally with a Python or JS walk (see Section 6.6).
5. Pack and import (Section 7).

### 6.2 Finding an action inside a large flow

The `definition.actions` dict at the top level is small. The interesting actions are usually nested 3–6 levels deep inside Switch cases, If branches, Until loops, etc. Helpers:

```python
def find_action(acts, name):
    if name in acts:
        return acts[name]
    for v in acts.values():
        if not isinstance(v, dict):
            continue
        for sub in [
            v.get('actions'),
            (v.get('else') or {}).get('actions'),
            *(c.get('actions') for c in (v.get('cases') or {}).values() if isinstance(c, dict)),
            (v.get('default') or {}).get('actions'),
        ]:
            if isinstance(sub, dict):
                r = find_action(sub, name)
                if r is not None:
                    return r
    return None
```

For "give me the parent dict so I can mutate it":
```python
def find_parent(acts, name):
    if name in acts:
        return acts
    for v in acts.values():
        if not isinstance(v, dict):
            continue
        for sub in [
            v.get('actions'),
            (v.get('else') or {}).get('actions'),
            *(c.get('actions') for c in (v.get('cases') or {}).values() if isinstance(c, dict)),
            (v.get('default') or {}).get('actions'),
        ]:
            if isinstance(sub, dict):
                r = find_parent(sub, name)
                if r is not None:
                    return r
    return None
```

### 6.3 Adding an action

Three things to consider:
1. **Where**: which scope's `actions` dict does it belong in?
2. **runAfter**: what predecessor(s) in that same scope must complete first? (`{}` only if it's the first sibling.)
3. **Downstream**: do any siblings need to start AFTER your new action? Update their `runAfter` to include your action.

Common pattern — insert action **between** A and B (where B currently `runAfter: A`):
```python
parent = find_parent(defn['actions'], 'A')
parent['NewAction'] = { "type": "...", "inputs": {...}, "runAfter": {"A": ["Succeeded"]} }
parent['B']['runAfter'] = {"NewAction": ["Succeeded"]}
```

Pattern — add a **parallel** action alongside an existing one (both with the same `runAfter`):
```python
orig = find_action(defn['actions'], 'A')
parent = find_parent(defn['actions'], 'A')
parent['A_parallel'] = { ..., "runAfter": dict(orig.get('runAfter', {})) }
```
This makes them both run after the same predecessor, in parallel.

### 6.4 Removing an action

1. Find every action whose `runAfter` references the doomed action — those references need to be redirected to whatever the doomed action's `runAfter` pointed to (or to another suitable upstream).
2. Find every expression in any action's inputs that references `outputs('Doomed')` or `body('Doomed')` — those will break unless you also rewire the data flow.
3. Then `del parent['Doomed']`.

Skipping step 2 produces a runtime error (or a save-time validation error if the designer catches it).

### 6.5 Replacing an action's contents but keeping the name

**This is the safest editing pattern when migrating connectors.** You preserve every downstream `outputs('X')` reference — only the BODY format may change.

Example — switching `Get_Approver` from Excel `GetItem` to SharePoint `GetItems`:
- Action keeps the same NAME.
- Action's `host` and `inputs.parameters` change to SP.
- Every downstream `outputs('Get_Approver')?['body/Email']` becomes `first(outputs('Get_Approver')?['body/value'])?['<spInternalCol>']` because the body shape changed.

If the shape change is too invasive, the alternative is "add a parallel action with a new name" (see 6.3) so the original stays. The user can compare both, then later delete the old one.

### 6.6 Validation pass

After every batch of edits, walk the definition and check that every action name referenced in expressions actually exists:
```python
import json, re
defn = json.load(open(path))
def walk(acts, names):
    for n, a in acts.items():
        names.add(n)
        if isinstance(a, dict):
            for sub in [a.get('actions'), (a.get('else') or {}).get('actions'),
                        *(c.get('actions') for c in (a.get('cases') or {}).values() if isinstance(c, dict)),
                        (a.get('default') or {}).get('actions')]:
                if isinstance(sub, dict):
                    walk(sub, names)
names = set(); walk(defn['actions'], names)
refs = set(re.findall(r"(?:outputs|body|actions)\('([^']+)'\)", json.dumps(defn)))
missing = refs - names
print(missing or "all refs resolve")
```
Also validate JSON parses cleanly and that no action has the deprecated `host.connection` or `host.connectionReferenceName` keys.

### 6.7 Editing the wrapper

The wrapper (`properties.connectionReferences`, `properties.templateName`, `schemaVersion`) lives in the same JSON. Add a new connection reference when you introduce a new connector:
```python
flow['properties']['connectionReferences']['shared_sharepointonline'] = {
    'runtimeSource': 'embedded',
    'connection': {'connectionReferenceLogicalName': '<prefix>_SharePointConnRef'},
    'api':        {'name': 'shared_sharepointonline'}
}
```
The connection reference record (`<prefix>_SharePointConnRef`) must already exist in the target environment — either created in the maker portal or shipped in the same solution's `connectionreferences/` folder.

---

## 7. Packing and importing

### 7.1 Pack

```powershell
pac solution pack `
  --zipfile C:\path\Solution_PATCHED.zip `
  --folder  C:\path\Solution_SRC
```
- Re-zips the unpacked source folder.
- Validates XML structure of the manifest; flow JSON is treated as opaque.

### 7.2 Import

```powershell
pac solution import `
  --path C:\path\Solution_PATCHED.zip `
  --force-overwrite `
  --publish-changes `
  --async
```
- `--force-overwrite` — replace existing components without merging. Required if you're updating flows already in the env.
- `--publish-changes` — publish customizations after import (otherwise unmanaged customizations stay unpublished). Always include for flows.
- `--async` — poll the long-running import operation; PAC blocks until it finishes. Without this, large solutions return immediately and you don't know if it succeeded.
- `--activate-plugins` — for plugin assemblies; doesn't affect flows.
- Returns an Import ID (GUID) on success — useful for searching the Dynamics async-operations log.

Typical successful tail:
```
Solution Imported successfully. Import ID: <guid>
Published All Customizations.
```

### 7.3 Versioning the source

Two-tier strategy:

- **Zip snapshots** at major milestones (`versions/<solution>_<date>_<label>.zip`). Quick rollback is `pac solution import --path <snapshot.zip> --force-overwrite --publish-changes`.
- **Git in the unpacked source folder** for diff-level history. Commit after every successful import.

### 7.4 Managed vs unmanaged

| Aspect | Unmanaged | Managed |
|---|---|---|
| Source of truth | The unpacked folder you edit | The solution zip |
| Editable in target env | Yes (changes saved to the unmanaged layer of that env) | No (the env can only customize via overlays) |
| Use case | Dev/test environments | Production releases |
| Import behavior | New components add; existing replace if `--force-overwrite` | Patches accumulate; uninstall removes the whole layer |

For the build-edit-deploy loop you're learning, **stay unmanaged**. Only export `--managed` when packaging for prod.

### 7.5 Connection-reference rebinding on import

When you import a solution into a different environment:
- The connection reference RECORDS (`<prefix>_*`) must already exist in the target, OR be in the solution.
- They start UNBOUND — the import succeeds but the flow can't run.
- A user (typically a System Admin) opens the connection ref in the maker portal and binds it to an actual connection (e.g. signs into the SharePoint account).
- Once all refs are bound, the flow can be turned on.

If a flow refuses to save with `host.connectionName' value is of type 'Null'`, the wrapper either doesn't declare the key or the connection-reference RECORD doesn't exist in the env yet. Check both.

---

## 8. Common errors and how to read them

| Error | Meaning | Fix |
|---|---|---|
| `Property 'host.connectionReferenceName' is missing` | You used `host.connection: "x"` or omitted the key | Use `host.connectionName: "<key>"` |
| `host.connectionReferenceName' is of type 'Null'` | You used `connectionReferenceName` (wrong key in modern schema) | Rename to `connectionName` |
| `The 'runAfter' property of action 'X' ... action 'Y' must belong to same level as action 'X'` | You set `X.runAfter` to reference `Y` but `Y` is in a different scope | Either move Y into the same scope, OR rely on cross-scope outputs() visibility through the parent scope's runAfter chain (don't put Y in X.runAfter) |
| `cannot reference action 'Y'. Action 'Y' must either be in 'runAfter' path or within a scope action on the 'runAfter' path` | The cross-scope chain doesn't visit Y or a scope containing Y | Find the pivot scope that wraps X's chain and add Y (or its containing scope) to that pivot's runAfter |
| `item/X is no longer present in the operation schema` | Excel/SP `dataset` or `table` is dynamic (e.g. `@outputs(...)`) and the connector can't resolve the column list at design time | Use literal strings for `dataset`/`table` |
| `WorkflowRunActionTimeoutLimitInvalid ... timeout '60.00:00:00' > workflow lifetime '30.00:00:00'` | An Until loop's `limit.timeout` exceeds 30 days | Set `limit.timeout` to `<= P30D`, or omit it entirely (only `count` remains) |
| `You are attempting to do a published update of publishable component in an unmodified active context when there exists an unpublished active row` | The flow has an in-designer draft (someone made an edit and hit save but didn't publish) | Delete the flow in the maker portal and re-import |
| `The dynamic operation request to API 'sharepointonline' operation 'GetTable' failed with status code 'Forbidden'` | The connection account doesn't have access to the SP site/list | Grant the connection's user account access in the SharePoint site permissions |
| `The response ID in request URL is invalid` (Forms `GetFormResponseById`) | Wrong path to `responseId` in trigger body | Use `@first(triggerOutputs()?['body/value'])?['resourceData']?['responseId']` (NOT `body/responseId` or `body/resourceData/responseId`) |
| `Workflow does not exist` (from `pac solution add-solution-component`) | You're trying to add a non-solution cloud flow to a solution via CLI | Add it via the maker portal: Solutions → + Add existing → Cloud flow → From outside Dataverse |
| `UserImpersonationScopeMissing` on admin migrateFlows API | The PAC user lacks tenant-admin "Power Platform Admin" role | Add flow to solution via portal (no admin needed) |
| Pre-commit hook fails / hook-blocked git commit | Hook in `.git/hooks/*` rejected your commit | Read the hook output; fix the underlying issue (don't bypass with `--no-verify` unless explicitly authorized) |

---

## 9. Gotchas, large and small

### 9.1 The `connection` / `connectionName` / `connectionReferenceName` triangle

Three different key names appear in the wild:
- `host.connection: "shared_x"` — older/legacy designer; works at runtime but throws `host.connectionReferenceName' is missing` on save in the modern designer.
- `host.connectionReferenceName: "shared_x"` — sometimes shown in examples/docs but is the WRONG key for modern flows; produces `value is of type 'Null'`.
- `host.connectionName: "shared_x"` — the CORRECT key for modern flows. Matches the key inside `properties.connectionReferences`.

Pattern-match every action in your flow to use `connectionName`.

### 9.2 SharePoint list column internal names

When a SP list column is added in the UI, its INTERNAL name is often auto-generated as `field_1`, `field_2`, ... — independent of the display name. The OData filters and `item/<x>` keys must use the INTERNAL name.

Three ways to discover internal names:
1. In SP, click the column header → Column settings → Edit → expand More options → URL of the edit page contains `Field=<internalName>`.
2. Export a Power Automate action's code view that targets the list (e.g. a sample PostItem you set up in the designer).
3. Query `https://<site>/_api/web/lists(guid'<listGuid>')/fields?$select=StaticName,Title` directly.

If the list was created from CSV import or from columns added with explicit camelCase names, internal names sometimes preserve the display name. **Always verify, never assume.**

### 9.3 Forms webhook payload shape

A common mistake is reading the responseId at the wrong path:
- ❌ `triggerOutputs()?['body/responseId']`
- ❌ `triggerOutputs()?['body/resourceData/responseId']`
- ✅ `first(triggerOutputs()?['body/value'])?['resourceData']?['responseId']`

The webhook body is `{ "value": [ { "resourceData": { "responseId": <int>, "formId": "..." }, ... } ] }`. responseId is often a small integer (1, 2, 5...), not a GUID.

### 9.4 The `_N` suffix convention for repeat submissions

A common Forms pattern: query a "submissions" store (Excel table or SP list) for prior responses by the same email; count them; suffix the submitter's folder name with `_<count>` so concurrent/repeat submissions don't collide. If the email has 0 prior rows, folder = `Name_0`. Otherwise `Name_<count+1>`. Be aware a naive `count+1` formula skips `_1` when the first submission used `_0` — verify the numbering matches your intent.

### 9.5 Excel Online file locks

`List_rows_present_in_a_table`, `Add a row`, `Update a row`, etc. all open the workbook with an exclusive write lock. If you (or anyone) opens the same `.xlsx` in **desktop Excel**, the flow's connector errors with 409 or "file is in use." Mitigations:
- Open via "Open in browser" only.
- Keep the file closed when you're not viewing it.
- Disable OneDrive sync for that specific file.
- Migrate the workload to a SharePoint list — SP has no exclusive-write lock and supports concurrent writes natively.

### 9.6 Empty string vs null

In WDL, `null` and empty string are NOT the same:
- `equals(variables('X'), '')` returns false when X is null.
- Use `coalesce(variables('X'), '')` to normalize, or `or(empty(X), equals(X, null))` to check both.

For OData filters, `<col> eq null` rarely works; use `<col> eq ''` for text columns, or omit the filter and post-filter with `where`/`if`.

### 9.7 ApprovalCreationInput key escaping

In some JSON dumps the key shows as `ApprovalCreationInput\title` (single backslash) — that's an escaping artifact of how the designer serializes. Always use forward slash in flat-key form: `ApprovalCreationInput/title`.

### 9.8 Microsoft Forms file uploads

The file-upload question in a Forms response returns a **string** that is a JSON-array literal. To iterate:
```
@json(outputs('Get_response_details')?['body/r<questionId>'])
```
Each item has `{ id, name, link, size, type, referenceId, driveId, status, uploadSessionUrl, badgerToken }`. Files always land in the form-owner's OneDrive under `/Apps/Microsoft Forms/<formname>/<question>/<filename>`.

### 9.9 `update_live_flow` payload size

The Power Automate management API has a soft limit (~100KB) on the `definition` blob when patching a live flow. For large flows the PAC export-edit-pack-import path scales better than direct API calls.

### 9.10 PAC interactive vs CI

`pac auth create` opens a browser by default. For CI/headless setups, supply `--applicationId / --clientSecret / --tenant`. `pac` runs non-interactively in sandboxed environments only if a profile is already cached.

### 9.11 Schema GUID quirks

- Flow JSON filename: `<Name>-<UPPERCASE-GUID>.json`
- Flow data.xml `WorkflowId`: `{<lowercase-guid>}`
- Solution.xml `<RootComponent id>`: `{<lowercase-guid>}`
- Inconsistencies between these throw `Workflow does not exist` or silent failures.

### 9.12 The Switch/Case statement's `case` field

Inside a Switch:
```json
"cases": {
  "Approve_Case": { "case": "Approve", "actions": { ... } }
}
```
The KEY (`Approve_Case`) is a label; the `case` field is what's compared against the Switch's `expression`. They don't have to match (and often the key is a sanitized version of the value). Common error: forgetting the `case` field and only setting the key.

### 9.13 Action name with trailing characters

Watch for action names like `Update_Status_to_Approved_` (note the trailing underscore) — the designer sometimes auto-appends underscores when duplicating actions or resolving name collisions. They're easy to miss when scripting `find_action(...)` lookups; copy the exact key from the JSON rather than retyping it.

---

## 10. Best practices

### 10.1 Action naming

- Use **underscores** to separate words; the designer auto-replaces spaces with underscores.
- Prefix by stage when there's a long pipeline: `Stage1_Send_Approval`, `Stage1_Wait_Approval`, `Stage1_Log_Row`.
- Keep names unique within a scope (required) and ideally unique across the whole flow (much easier to grep and reference).
- Avoid hyphens in keys you'll reference often — string-concat in expressions gets harder.

### 10.2 Error handling

End failure paths with a `Terminate` action that stops the run with a clear status and error, so a failed run surfaces a meaningful message instead of a generic connector error:
```json
"Terminate_Failure": {
  "type": "Terminate",
  "inputs": {
    "runStatus": "Failed",
    "runError": { "code": "RECORD_MISSING", "message": "Required record not found" }
  },
  "runAfter": {}
}
```
`Terminate` also supports `runStatus: "Cancelled"` or `"Succeeded"` (e.g. to short-circuit cleanly when a guard condition isn't met). Use the `runAfter` status filters (`Failed`, `TimedOut`, `Skipped`) on downstream actions to route what happens when an upstream action fails.

### 10.3 Child flows vs inline

Use a CHILD flow (HTTP-triggered, called from the parent with the "Run a child flow" action) when:
- The same logic is needed in 2+ parent flows.
- A piece of the pipeline is independently versioned/tested.
- You want to isolate connection refs (e.g., the child uses a service-account SP connection while the parent runs as the user).

Keep inline when:
- The logic is only used once.
- You need shared variable state without serializing back through HTTP.
- You're optimizing for editor visibility / single-flow debuggability.

### 10.4 Maintainability

- Add a `Compose` named `Constants_*` at the top of each major branch — central place for hard-coded values.
- Avoid magic strings inside expressions; promote them to variables initialized once.
- Document why a non-obvious expression exists with an adjacent `Compose` whose name explains the intent (`Compose_WhyWeSkipUnderscoreOne`).
- Don't go beyond 3 levels of nesting (If inside Switch inside Until inside If) without extracting to a child flow or scope — the designer becomes painful to navigate.

### 10.5 Version control

A pragmatic two-layer scheme:
- **Zip snapshots** in `versions/<solution>_<YYYY-MM-DD>_<label>.zip` for major milestones. Easy one-command rollback.
- **Git in the unpacked source folder** for line-level history.

Recommended commit cadence:
- Commit the baseline export immediately after unpacking.
- Commit after every successful import (with the Import ID in the message).
- Commit failed-import attempts too, so you can `git diff` to see what made the import work.

### 10.6 Don't fight the designer

If a flow has been opened and saved in the designer, the designer rewrites the JSON in subtle ways:
- Reorders keys
- Adds/removes `metadata.operationMetadataId` GUIDs
- Inserts default `authentication: "@parameters('$authentication')"` on every connector action
- Normalizes whitespace inside HTML email bodies

After such edits, the file may look textually different from what you pushed even if functionally identical. Expect noisy diffs on the next export. Don't fight it; commit and move on.

### 10.7 Bundle changes

Each `pac solution import` is ~30–60 seconds of overhead. When you have multiple unrelated edits queued (rename + add column + email-body tweak), **batch them into one import**. Iterative one-edit-per-import loops waste a lot of time.

### 10.8 Use draft state for risky test flows

When duplicating a production flow into a test copy:
- Generate a NEW GUID (and new filename/`WorkflowId` to match).
- Set the data.xml to `<StateCode>0</StateCode><StatusCode>1</StatusCode>` so it imports as Draft and **does not auto-fire on the same webhook the production flow is listening to**.
- Manually toggle it on once you're ready.

### 10.9 Backups before destructive ops

Before any of:
- `pac solution import` of a major change
- Deleting a flow in the portal to clear an "unpublished active row" conflict
- Replacing a flow's GUID or filename

…export the current solution into a timestamped zip and commit the unpacked source. Rollback is then trivial.

---

## 11. End-to-end cheat sheet

```powershell
# 1. Auth + env (one-time per session)
pac auth create --environment "<EnvName>"
pac env who

# 2. Export + unpack
pac solution export --name <SolutionUniqueName> --path Sol.zip --overwrite
pac solution unpack --zipfile Sol.zip --folder Sol_SRC --allowDelete

# 3. Initialize / update source git
cd Sol_SRC
git init -b main      # one time
git add -A
git commit -m "baseline export"

# 4. Edit Workflows/<flow>.json — directly, or via a Python script that
#    loads the JSON, mutates the actions dict, validates, and saves.

# 5. Pack
pac solution pack --zipfile Sol_PATCHED.zip --folder Sol_SRC

# 6. Import
pac solution import --path Sol_PATCHED.zip --force-overwrite --publish-changes --async

# 7. Snapshot + commit
Copy-Item Sol_PATCHED.zip ..\versions\Sol_<date>_<label>.zip
git add -A
git commit -m "<change summary> (Import <import-id>)"
```

---

## 12. Quick reference: commonly used action types

> ⚠️ This table is a **convenience cross-check of frequently used actions** — it is NOT the authoritative catalog and NOT exhaustive. Per the HARD RULE at the top, you must still verify every action against the `power-automate-connectors` skill before writing it. If an action you need is not in this table, that means nothing about whether it exists — go check the catalog.

| Task | type / operationId / apiId |
|---|---|
| Send an Outlook email | `OpenApiConnection` / `SendEmailV2` / `shared_office365` |
| Create an approval | `OpenApiConnection` / `CreateApproval` / `shared_approvals` |
| Wait for the approval response | `OpenApiConnectionWebhook` / `WaitForAnApproval` / `shared_approvals` |
| Look up an Excel row by key | `OpenApiConnection` / `GetItem` / `shared_excelonlinebusiness` |
| List Excel rows | `OpenApiConnection` / `ListRowsPresentInATable` / `shared_excelonlinebusiness` |
| Add an Excel row | `OpenApiConnection` / `AddRowV2` / `shared_excelonlinebusiness` |
| Update an Excel row | `OpenApiConnection` / `PatchItem` / `shared_excelonlinebusiness` |
| SP `Create item` | `OpenApiConnection` / `PostItem` / `shared_sharepointonline` |
| SP `Get items` (with $filter) | `OpenApiConnection` / `GetItems` / `shared_sharepointonline` |
| SP `Update item` | `OpenApiConnection` / `PatchItem` / `shared_sharepointonline` |
| SP `Delete item` | `OpenApiConnection` / `DeleteItem` / `shared_sharepointonline` |
| SP `Create new folder` (in lib) | `OpenApiConnection` / `CreateNewFolder` / `shared_sharepointonline` |
| SP `Create sharing link for a file or folder` | `OpenApiConnection` / `CreateSharingLink` / `shared_sharepointonline` |
| SP `Stop sharing an item or a file` | `OpenApiConnection` / `UnshareItem` / `shared_sharepointonline` (files only — fails on folders) |
| OneDrive `Create file` | `OpenApiConnection` / `CreateFile` / `shared_onedriveforbusiness` |
| OneDrive `Get file content by path` | `OpenApiConnection` / `GetFileContentByPath` / `shared_onedriveforbusiness` |
| OneDrive `Get file metadata` | `OpenApiConnection` / `GetFileMetadata` / `shared_onedriveforbusiness` |
| OneDrive `Get file content` (by id) | `OpenApiConnection` / `GetFileContent` / `shared_onedriveforbusiness` |
| OneDrive `Delete file` | `OpenApiConnection` / `DeleteFile` / `shared_onedriveforbusiness` |
| OneDrive `Create share link by path` | `OpenApiConnection` / `CreateShareLinkByPathV2` / `shared_onedriveforbusiness` (no expirationDateTime support!) |
| Forms — when a new response is submitted | `OpenApiConnectionWebhook` / `CreateFormWebhook` / `shared_microsoftforms` |
| Forms — get response details by id | `OpenApiConnection` / `GetFormResponseById` / `shared_microsoftforms` |
| Office 365 Users — get user profile | `OpenApiConnection` / `UserProfile_V2` / `shared_office365users` |
| Cloudmersive — HTML to PDF | `OpenApiConnection` / `ConvertDocumentToPdf` / `shared_cloudmersiveconvert` |

Use this as a "did I get the host right?" cross-check while building — then confirm against the `power-automate-connectors` catalog per the HARD RULE.

---

End of skill.
