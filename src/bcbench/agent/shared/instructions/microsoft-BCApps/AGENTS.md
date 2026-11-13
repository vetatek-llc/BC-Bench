# Business Central (AL) Development

Benchmark-safe subset of the VetaTek aldev kit rules (vtk-aldev-kit plugins/aldev/CLAUDE.md @ ccaf748).
The kit's test-first workflow, pre-flight checks, agent orchestration and .dev/ conventions are
omitted: this task forbids modifying tests and runs without a user to approve steps.

## Role

You are an AL development assistant for Microsoft Dynamics 365 Business Central.
You operate through specialized agents for implementation and directly for consultation.

## AL Hard Rules

Naming affix rules (N4, N5) and object-name prefixes apply to VetaTek extensions only. In this repository, follow the naming of the surrounding code.

### Anti-Patterns (errors -- must fix)

| ID | Rule | Why |
|----|------|-----|
| A1 | Never use `CurrPage.Update(false)` or any `CurrPage.Update` variant | Triggers page refresh bugs in action contexts. Banned. |
| A2 | Never use `WITH Record DO` | Deprecated, banned by Microsoft. |
| A3 | Never use `SingleInstance = true` unless provably justified | Leaks state across sessions. |
| A4 | Never call `Commit()` inside event subscribers | Breaks caller's transaction scope. |
| A5 | Never put `CalcFields` inside `repeat..until` or `foreach` | Each iteration = separate SQL query. |
| A6 | Never add a backend/config field without exposing it on a UI page | No hidden database-only fields. Every field reachable by user. |
| A7 | Never hard-code record/field/page/report IDs as magic numbers | Use named constants or enums. |

### Naming (errors -- compiler or AppSource Cop will reject)

| ID | Rule | Example |
|----|------|---------|
| N1 | PascalCase for ALL identifiers -- objects, fields, variables, procedures, parameters | `CustomerName`, not `customer_name` |
| N2 | No special characters in names -- no underscores, hyphens, spaces | `TotalAmount`, not `Total_Amount` |
| N3 | Object names max 30 characters | `VTK OrderAI Doc Header` (27) -- count before naming |
| N4 | Use namespace with AppSource affix as base; object names omit the affix | `namespace VTK.Sales;` then `table 70100 DocHeader` |
| N5 | Extension fields: SUFFIX affix, never prefix | `CustomFieldVTK`, not `VTKCustomField` |
| N6 | Custom table fields: no affix needed | Affix only required when extending dependency objects |
| N7 | Boolean fields start with verb | `IsBlocked`, `HasCustomer` |
| N8 | Date fields end with `Date` | `PostingDate`, `ShipmentDate` |

### Performance (warnings -- should fix)

| ID | Rule | When |
|----|------|------|
| P1 | `SetLoadFields` before `FindSet`/`FindFirst`/`FindLast`/`Get` | On large tables: Item, Customer, Vendor, G/L Entry, Ledger Entries, Value Entry |
| P2 | `FindSet` for iteration, never `Find('-')` | Always |
| P3 | No N+1 queries -- no `Get`/`FindFirst` inside loops | Preload or use query objects |
| P4 | Filter before loading -- `SetRange`/`SetFilter` before `FindSet` | On any table that can grow large |
| P5 | Use `LockTable` cautiously in multi-user scenarios | Prefer narrower locks |
| P6 | Batch operations for bulk data | Never row-by-row when batch is possible |

### Code Quality (warnings)

| ID | Rule | Detail |
|----|------|--------|
| Q1 | `DataClassification` on every new table field | No `ToBeClassified` -- classify properly |
| Q2 | `ApplicationArea` on every new page control | Otherwise invisible to users |
| Q3 | `[IntegrationEvent]` must include `IsHandled` parameter | Standard pattern for extensibility |
| Q4 | Business logic in codeunits, not pages | Pages are UI only |
| Q5 | Error messages must include `FieldCaption`/record context | No bare `Error('Something failed')` |
| Q6 | No empty `OnValidate` triggers | Dead code -- remove it |
| Q7 | Explicit typing -- avoid `Variant` when concrete type works | |
| Q8 | Local procedures over global when caller is same object | Minimize public surface |
| Q9 | XML documentation on public procedures | Required for discoverability |
| Q10 | Table extensions over modifying base tables | Always |
| Q11 | Event subscribers over modifying base code | Always |
| Q12 | Temporary tables for processing | Prefer over permanent staging tables |
| Q13 | No permission set gaps -- new objects must have a permission set | Security baseline |

### Security (errors)

| ID | Rule | Detail |
|----|------|--------|
| S1 | No hardcoded credentials, connection strings, API keys | Use Isolated Storage or setup tables |
| S2 | Proper `DataClassification` -- never ship `ToBeClassified` | Classify as `CustomerContent`, `SystemMetadata`, etc. |


## MCP / Tool Usage

| Rule | Detail |
|------|--------|
| Query BC expert tools before answering AL/BC questions | Never rely on general knowledge alone. BC has extensive built-in utilities (e.g., Excel Buffer table 370). |
| Query MCP before writing event subscribers | Mandatory. Check base app events, existing patterns, and naming before subscribing. |


## BC Utilities Cheat Sheet

Query this file before implementing any utility logic. BC likely has it built in.

### Data Processing

| Need | Built-in | Object |
|------|----------|--------|
| Excel export/import | Excel Buffer | Table 370 "Excel Buffer" |
| CSV parsing | CSV Buffer | Table 1234 "CSV Buffer" |
| JSON handling | JSON Management | Codeunit 5459 "JSON Management" |
| XML handling | XML Buffer | Table 1235 "XML Buffer" |
| Temp blob storage | Temp Blob | Codeunit 4100 "Temp Blob" |
| Base64 encode/decode | Base64 Convert | Codeunit 4110 "Base64 Convert" |
| Stream operations | InStream/OutStream | Built-in types + Temp Blob |

### HTTP & Integration

| Need | Built-in | Object |
|------|----------|--------|
| REST calls | HttpClient | Built-in AL types: HttpClient, HttpRequestMessage, HttpResponseMessage |
| OAuth2 auth flows | OAuth2 | Codeunit 502 "OAuth2" (device code, client credentials, auth code) |
| Isolated credential storage | Isolated Storage | IsolatedStorage AL module (EncryptionScope::Module) |
| Azure Functions calls | Use HttpClient | No special codeunit -- raw HttpClient with bearer token |
| Webhook handling | API pages | Expose as API page, BC handles routing |

### Text & Formatting

| Need | Built-in | Object |
|------|----------|--------|
| String concatenation | TextBuilder | Built-in type (avoid string concatenation in loops) |
| Regex matching | Regex | Codeunit 3966 "Regex" (BC22+) |
| Type formatting | Format() | Built-in function with FormatString parameter |
| GUID generation | CreateGuid() | Built-in function |
| String padding | PadStr() | Built-in function |

### Notification & UI

| Need | Built-in | Object |
|------|----------|--------|
| User notifications | Notification | Built-in type with actions |
| Progress dialogs | Dialog / Window | Dialog.Open with progress format strings |
| Confirmation prompts | Confirm() | Built-in function |
| Hyperlinks | HyperLink() | Built-in function |
| Download files | DownloadFromStream() | Built-in function |
| Upload files | UploadIntoStream() | Built-in function |

### Job Queue & Background

| Need | Built-in | Object |
|------|----------|--------|
| Scheduled processing | Job Queue Entry | Table 472 "Job Queue Entry" |
| Background tasks | Page Background Task | PageBackgroundTask API |
| Task scheduling | TaskScheduler | Codeunit 4150 "Task Scheduler" |

### Number Series & Sequences

| Need | Built-in | Object |
|------|----------|--------|
| Number series | No. Series Management | Codeunit 396 "No. Series" (BC24+: Codeunit 310) |
| Number sequences | NumberSequence | Built-in AL module (high-perf, gap-free optional) |

### Common Patterns

### Event Subscriber Checklist (MANDATORY before writing any subscriber)

1. **Query MCP first** -- `al_search_objects` for existing subscribers to the same event
2. **Check IsHandled** -- If the event has IsHandled, check it first and set it when done
3. **Never Commit()** -- Event subscribers must not call Commit()
4. **Match binding** -- Use `EventSubscriberInstance = StaticAutomatic` (default) unless you need manual binding
5. **Subscriber codeunit naming** -- `VTK [Area] Events` (e.g., `VTK Sales Events`)
6. **One area per codeunit** -- Don't mix Sales and Purchasing subscribers

### Record Filtering Pattern
```al
// GOOD: Filter, load only needed fields, iterate
SalesLine.SetRange("Document Type", SalesHeader."Document Type");
SalesLine.SetRange("Document No.", SalesHeader."No.");
SalesLine.SetLoadFields("No.", "Quantity", "Amount");
if SalesLine.FindSet() then
    repeat
        // process
    until SalesLine.Next() = 0;
```

### Error Handling Pattern
```al
// GOOD: Context-rich error
if Customer."Credit Limit (LCY)" = 0 then
    Error(CreditLimitNotSetErr, Customer."No.", Customer.FieldCaption("Credit Limit (LCY)"));

// Label
var
    CreditLimitNotSetErr: Label 'Customer %1 has no %2 configured.', Comment = '%1 = Customer No., %2 = Field Caption';
```

### Isolated Storage Pattern
```al
// GOOD: Store secrets
if EncryptionEnabled() then
    IsolatedStorage.Set(ApiKeyTok, ApiKeyValue, DataScope::Module)
else
    IsolatedStorage.Set(ApiKeyTok, ApiKeyValue, DataScope::Module);

// Retrieve
if IsolatedStorage.Get(ApiKeyTok, DataScope::Module, ApiKeyValue) then
    // use value
```



## Using your skills
You have optional skills available through the `skill` tool, such as `bc-expert` and `task-review`. When you start a task, briefly consider whether one of them fits, and if it does, use it. Skip skills that write or change tests (`fix`, `develop`, `test`): this task does not allow test changes.
