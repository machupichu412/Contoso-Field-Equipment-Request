# Build Specification
## Contoso Field Equipment Request (FER) — Reference Test Application

**Internal build artifact. Not for distribution to the review agent.**

| Field | Detail |
|---|---|
| Document | Build Specification — FER Reference Test Application |
| Version | 1.0 |
| Date | 27 July 2026 |
| Author | Matthew Yeh |
| Parent initiative | PowerReview / Agentic Power Platform Reviewer |
| Companion artifacts | `BRD_Contoso_Field_Equipment_Request.md` (agent input — clean), `BRD_FER_Reference_Test_App_PowerReview_MVP.docx` (answer key) |
| Classification | Microsoft Internal — non-production test artifact |

---

## 0. How to use this document

This spec builds the app described in the clean FER BRD, then deliberately breaks it in 28 catalogued ways while keeping 12 implementations correct.

**Two-document rule.** The clean BRD is the *only* document supplied to PowerReview. This build spec and the answer key stay with the team. If the agent ever sees this file, every subsequent recall number is void.

**Build discipline.**

- Implement defects **exactly** as written. "Improving" a seeded defect during build silently invalidates its expected finding.
- Implement controls **exactly** as written. A sloppy control becomes an accidental defect and inflates the false-positive count.
- Every defect ID below maps 1:1 to a row in the answer key. Do not renumber.
- Two-person verification: the person who seeds a defect does not sign it off.

**Legend.** 🔴 seeded defect · 🟢 clean control · ⚪ neutral scaffolding

---

## 1. Environment and Solution Setup

### 1.1 Environments

| Environment | Purpose | Notes |
|---|---|---|
| `DEV-FER-REVIEW` | Primary build | Where all seeding happens |
| `TEST-FER-REVIEW` | Review target | Receives the export submitted to the agent |

No production environment is created. The app has no business owner.

> 🔴 **DEF-GOV-01** — *Solution left in the Default environment.* Build the **catalog data source and Flow B** in the tenant **Default** environment rather than `DEV-FER-REVIEW`. This must be pre-cleared with the CoE admin and recorded as an intentional exception so the environment is not auto-remediated mid-test.

### 1.2 Publisher and solution

```
Publisher display name : Contoso FER
Publisher prefix       : fer
Solution name          : contoso_fer_reviewbenchmark
Solution version       : 1.0.0.0
```

> 🔴 **DEF-ARC-05** — *Components outside a solution, default publisher, inconsistent prefix.* Create the **Employee** table and **Flow A** outside the solution, under the **default publisher** (`cr###` prefix). Only `Equipment_Request`, `Equipment_Catalog`, and `Request_Audit` sit inside `contoso_fer_reviewbenchmark` with the `fer` prefix. The resulting prefix mix is the finding.

> 🔴 **DEF-GOV-05** — *No Solution Checker run before export.* Export and submit for review without running Solution Checker. Do not run it "just to see" — the absence of a run record is the artifact.

> 🔴 **DEF-GOV-03** — *Missing app metadata.* Leave Description, Owner, and Support contact **blank** on the canvas app.

> 🔴 **DEF-GOV-04** — *Single owner, no co-owner.* Assign exactly one owner. Do not add a co-owner.

---

## 2. Dataverse Schema

Column names below match the tables already created. Build to these names exactly.

### 2.1 `Equipment_Request`

Primary column: `Request_Number` (Text).

| Column | Type | Notes |
|---|---|---|
| `Request_Number` | Text | Primary name |
| `Requester_Email` | Email | 🔴 see DEF-ARC-01 |
| `Requester_Name` | Text | 🔴 see DEF-ARC-01 |
| `Item_Name` | Text | 🟢 see CTL-02 |
| `Quantity` | Whole Number | |
| `Justification` | Multi-line Text | |
| `Status` | **Text** | 🔴 see DEF-ARC-02 |
| `Estimated_Cost` | Currency | 🔴 see DEF-SEC-01 |
| `Approver_Notes` | Multi-line Text | 🔴 see DEF-SEC-01 |
| `Asset_Tag` | Text | |
| `Department` | Text | 🔴 see DEF-ARC-03 |
| `Cost_Center` | Text | 🔴 see DEF-ARC-03 |
| `Created_On` | Date and Time | |

> 🔴 **DEF-ARC-01** — *Requester stored as free text, not a lookup.* `Requester_Email` and `Requester_Name` are plain columns with **no relationship** to `Employee`. There is no referential integrity: a request can name an employee who does not exist.

> 🔴 **DEF-ARC-02** — *Status as free text.* `Status` is a **Text** column, not a Choice. Nothing prevents an invalid value being written. Seed one row with `Status = "Aproved"` (misspelled) to make the consequence demonstrable.

> 🔴 **DEF-ARC-03** — *Denormalised department and cost centre.* `Department` and `Cost_Center` are copied onto the request at submission with **no sync mechanism**. If the employee moves department, historical and future requests silently diverge from `Employee`.

> 🔴 **DEF-SEC-01** — *No column-level security on sensitive fields.* `Estimated_Cost` and `Approver_Notes` have **column security disabled**. They are hidden from requesters only by control visibility in the canvas app (§4.2), so the data remains retrievable via the data source. This is the most severe seeded defect — BR-006 is a security requirement implemented as a cosmetic one.

### 2.2 `Equipment_Catalog`

| Column | Type |
|---|---|
| `Item_Name` | Text (primary) |
| `Category` | **Choice** — Computer, Peripheral, AV, Network, Storage |
| `Unit_Cost` | Currency |
| `Active` | Yes/No |

> 🟢 **CTL-03** — *Choice column used correctly.* `Category` is a proper Choice column. This sits deliberately adjacent to DEF-ARC-02: the agent must flag `Status` as free text **without** also flagging `Category`.

> 🔴 **DEF-ARC-04** — *Mixed data sources.* Serve the catalog from a **SharePoint list**, while all other tables are Dataverse. Create the list `FER Equipment Catalog` with the same four columns and populate the 15 rows there. The canvas app connects to SharePoint for catalog only.

### 2.3 `Employee`

| Column | Type |
|---|---|
| `Employee_Name` | Text (primary) |
| `Email` | Email |
| `Manager_Email` | Email |
| `Department` | Text |
| `Cost_Center` | Text |

> 🟢 **CTL-07** — *Column-level security applied correctly.* Enable **column security on `Cost_Center`**, granting read only to Operations Admin. This is the near-neighbour of DEF-SEC-01: correct column security here, absent there.

### 2.4 `Request_Audit`

| Column | Type |
|---|---|
| `Request` | Text |
| `Action` | Text |
| `Actor` | Text |
| `Timestamp` | Date and Time |

> 🔴 **DEF-REQ-02** — *Audit on create only.* Records are written **only** when a request is created. Approve, Reject, and Fulfil write nothing. BR-007 says *every status change*; the implementation covers one event. Ten audit rows against thirty requests is the visible symptom.

---

## 3. Security Roles

Three roles, created in the solution.

### 3.1 `FER Requester`

| Table | Create | Read | Write | Delete |
|---|---|---|---|---|
| Equipment_Request | User | User | User | **Organization** 🔴 |
| Equipment_Catalog | None | Organization | None | None |
| Request_Audit | User | None | None | None |

> 🔴 **DEF-SEC-02** — *Organization-level Delete granted to Requester.* A requester can delete **any** request in the system, including approved and fulfilled ones belonging to other employees. Privilege escalation by over-grant.

### 3.2 `FER Approver`

| Table | Create | Read | Write | Delete |
|---|---|---|---|---|
| Equipment_Request | None | **Organization** 🔴 | Organization | None |
| Equipment_Catalog | None | Organization | None | None |
| Request_Audit | User | Organization | None | None |

> 🔴 **DEF-SEC-05** — *Approver scope enforced in the gallery, not the role.* BR states an approver may only action their own direct reports. The **role grants Organization read on all requests**; the restriction exists solely as a gallery filter (§4.4). Any approver can read every request in the tenant.

### 3.3 `FER Operations Admin`

| Table | Create | Read | Write | Delete |
|---|---|---|---|---|
| Equipment_Request | None | **Business Unit** 🟢 | Business Unit | None |
| Equipment_Catalog | Business Unit | Business Unit | Business Unit | Business Unit |
| Request_Audit | User | Business Unit | None | None |

> 🟢 **CTL-06** — *Admin role correctly scoped to Business Unit, not Organization.* Near-neighbour of DEF-SEC-02. The agent must flag the Requester over-grant without flagging this.

### 3.4 App sharing

> 🔴 **DEF-SEC-03** — *Over-broad sharing.* Share the app with a broad security group (for example `All Contoso Employees`) rather than the intended pilot group of roughly 25 users.

---

## 4. Canvas App

Tablet layout. Five screens.

### 4.1 `App.OnStart`

```powerfx
// 🔴 DEF-CR-02 — full-table load into a collection at startup
ClearCollect(
    colRequests,
    'Equipment_Request'
);

// 🔴 DEF-SEC-04 — role determined by a client-side variable
Set(varUserRole, "Requester");
Set(varIsAdmin, false);

// 🔴 DEF-CR-08 — declared, never referenced anywhere in the app
Set(varUnusedThreshold, 1000);
Set(varLegacyMode, true);
```

> 🔴 **DEF-CR-02** — `ClearCollect` of the entire request table on startup. Unbounded, non-delegable, and degrades linearly with table growth.

> 🔴 **DEF-SEC-04** — Role held in `varUserRole`, a client-side global. Screen visibility and the Admin Console entry point key off it. It is not backed by a Dataverse role check, so it is trivially influenced client-side.

> 🔴 **DEF-CR-08** — `varUnusedThreshold` and `varLegacyMode` are set and never read. Confirm with a full-app search before sign-off.

### 4.2 Screen 1 — `scrHome`

Landing screen. Role-based navigation buttons.

Mandatory disclaimer banner (`lblDisclaimer`):

```
This application contains intentional defects and is used for review tooling
validation only. It is not a reference implementation and must not be copied.
```

> ⚪ The disclaimer is scaffolding, not a defect. It prevents the app being mistaken for good practice. It is worded to avoid naming the review tool.

### 4.3 Screen 2 — `scrMyRequests`

Controls: `galRequests`, `btnNewRequest`.

```powerfx
// 🔴 DEF-CR-01 — non-delegable: Lower() and StartsWith() on a text column
Filter(
    'Equipment_Request',
    StartsWith(Lower(Requester_Email), Lower(User().Email))
)
```

> 🔴 **DEF-CR-01** — Wrapping the column in `Lower()` breaks delegation. Beyond 500 rows the requester silently sees an incomplete list — a correctness bug wearing a performance costume.

Status colour on `galRequests`:

```powerfx
// 🔴 DEF-CR-06 — six-level nested If where Switch is correct
If(ThisItem.Status = "Draft", Color.Gray,
    If(ThisItem.Status = "Submitted", Color.Orange,
        If(ThisItem.Status = "Approved", Color.Green,
            If(ThisItem.Status = "Rejected", Color.Red,
                If(ThisItem.Status = "Fulfilled", Color.Blue,
                    If(ThisItem.Status = "Cancelled", Color.DarkGray,
                        Color.Black
                    )
                )
            )
        )
    )
)
```

Cost visibility — this is the client-side half of DEF-SEC-01:

```powerfx
// lblEstimatedCost.Visible
varUserRole <> "Requester"
```

> 🔴 The control is hidden, the data is not protected. Pairs with the absent column security in §2.1.

### 4.4 Screen 3 — `scrNewRequest`

Item dropdown:

```powerfx
// 🟢 CTL-... catalog correctly filtered to active items (BR-010)
Filter('FER Equipment Catalog', Active = true)
```

Submit button:

```powerfx
// 🔴 DEF-CR-04 — no IfError, no validation, no user feedback
Patch(
    'Equipment_Request',
    Defaults('Equipment_Request'),
    {
        Request_Number: "REQ-" & Text(CountRows(colRequests) + 1, "0000"),
        Requester_Email: User().Email,
        Requester_Name: User().FullName,
        Item_Name: drpItem.Selected.Item_Name,
        Quantity: Value(txtQuantity.Text),
        Justification: txtJustification.Text,
        Status: "Submitted",
        Estimated_Cost: drpItem.Selected.Unit_Cost * Value(txtQuantity.Text),
        Created_On: Now()
    }
);
Navigate(scrMyRequests)
```

> 🔴 **DEF-CR-04** — Unguarded `Patch`. A failed save navigates away as though it succeeded. Note also `Request_Number` derived from a client-side row count — a collision risk under concurrency, which a strong agent may raise as an additional observation.

> 🟢 **CTL-02** — `Item_Name` populated from the catalog selection rather than free text. Near-neighbour of DEF-ARC-01.

### 4.5 Screen 4 — `Screen2` *(default name retained)*

Purpose: Approvals queue. **Do not rename.**

> 🔴 **DEF-CR-07** — *Default control names and orphaned screens.* Retain `Screen2`, `Button4`, `Form9`. Add two screens (`Screen5`, `Screen6`) that are unreachable from any navigation path.

Gallery items:

```powerfx
// 🟢 CTL-01 — delegable filter on an indexed column
Filter('Equipment_Request', Status = "Submitted")
```

> 🟢 **CTL-01** — Deliberately in the same app as DEF-CR-01. Delegable, no function wrapping the column. The agent must discriminate rather than flag every `Filter()`.

Approve (`Button4`):

```powerfx
Patch('Equipment_Request', galApprovals.Selected, {Status: "Approved"})
// No audit write — see DEF-REQ-02
```

Reject:

```powerfx
// 🟢 CTL-12 — BR-005 rejection reason correctly enforced
If(
    IsBlank(txtRejectReason.Text),
    Notify("A rejection reason is required.", NotificationType.Error),
    Patch(
        'Equipment_Request',
        galApprovals.Selected,
        {Status: "Rejected", Approver_Notes: txtRejectReason.Text}
    )
)
```

Category routing:

```powerfx
// 🟢 CTL-05 — Switch used correctly, adjacent to DEF-CR-06
Switch(
    drpCategory.Selected.Value,
    "Computer", Color.Blue,
    "Peripheral", Color.Green,
    "AV", Color.Purple,
    "Network", Color.Teal,
    Color.Gray
)
```

`OnVisible`:

```powerfx
// 🟢 CTL-10 — Concurrent used correctly for two independent loads
Concurrent(
    ClearCollect(colPendingApprovals, Filter('Equipment_Request', Status = "Submitted")),
    ClearCollect(colCatalog, Filter('FER Equipment Catalog', Active = true))
)
```

### 4.6 Screen 5 — `scrAdmin`

> 🟢 **CTL-09** — *Descriptive, prefixed naming throughout this screen.* `galAdminRequests`, `btnFulfil`, `txtAssetTag`, `lblAdminHeader`. Near-neighbour of DEF-CR-07: correct naming here, defaults on `Screen2`.

Fulfil action — **the approval gate is absent**:

```powerfx
// 🔴 DEF-REQ-01 — no status precondition; Draft or Submitted can go straight to Fulfilled
Patch(
    'Equipment_Request',
    galAdminRequests.Selected,
    {Status: "Fulfilled", Asset_Tag: txtAssetTag.Text}
)
```

> 🔴 **DEF-REQ-01** — Traces to **BR-004** ("must be approved by a manager before it can be fulfilled") and violates **AC-6**. Critical severity. The single most important find in the corpus: it is a business-logic failure invisible to any static checker.

Bulk update:

```powerfx
// 🔴 DEF-CR-03 — nested ForAll with record-by-record Patch
ForAll(
    colRequests,
    ForAll(
        Filter(colCatalog, Category = "Computer"),
        Patch(
            'Equipment_Request',
            LookUp('Equipment_Request', Request_Number = Request_Number),
            {Estimated_Cost: Unit_Cost * Quantity}
        )
    )
)
```

Hardcoded threshold:

```powerfx
// 🔴 DEF-CR-05 — literal 1000 repeated in four places
If(ThisItem.Estimated_Cost > 1000, "High value", "Standard")
// Repeat the same literal in: lblCostFlag.Text, icoWarning.Visible,
// galAdminRequests.TemplateFill, btnEscalate.DisplayMode
```

Unrequested function:

```powerfx
// 🔴 DEF-REQ-03 — no requirement authorises this
RemoveIf('Equipment_Request', true)
```

> 🔴 **DEF-REQ-03** — `btnDeleteAll`, labelled "Delete All Requests". Maps to **no BR**. Scope creep, and destructive. The agent should flag unrequested functionality, not merely that it is dangerous.

---

## 5. Power Automate

### 5.1 Flow A — `FER Request Submitted`

Trigger: When a row is added to `Equipment_Request`.

Actions:
1. Send email to approver
2. Add row to `Request_Audit` (Action = `Created`)

```
// 🔴 DEF-ARC-06 — hardcoded values instead of environment variables
To: manager@contoso.com
SharePoint site: https://contoso.sharepoint.com/sites/FEREquipment
```

> 🔴 **DEF-ARC-06** — Recipient and site URL hardcoded. The flow cannot be promoted between environments without editing.

> 🔴 **DEF-SEC-06** — *Implicitly shared maker-owned connection.* Flow A runs under the personal Office 365 Outlook connection of the maker. If that account is disabled the flow breaks, and all mail sends as an individual.

> ⚪ Flow A is built **outside the solution** under the default publisher — part of DEF-ARC-05.

### 5.2 Flow B — `FER Status Changed`

Trigger: When `Status` is modified.

Actions:
1. Notify requester
2. Route approved requests to the Operations queue

```
// 🟢 CTL-08 — environment variable used correctly
To: @{parameters('envNotificationRecipient')}
```

> 🟢 **CTL-08** — Near-neighbour of DEF-ARC-06: correct here, hardcoded in Flow A.

> 🟢 **CTL-11** — *Service-account connection with documented ownership.* Flow B uses `svc-fer-automation@contoso.com`, documented in the flow description. Near-neighbour of DEF-SEC-06.

> 🔴 **DEF-GOV-02** — *DLP-violating connector combination.* Combine a **Business**-classified connector (Dataverse) with a **Non-Business** connector (for example Twitter or a generic HTTP action) in the same flow, crossing the tenant DLP boundary. **Pre-clear with the CoE admin** before building, or the environment may be suspended mid-test.

> ⚪ Flow B is built in the **Default environment** — this is DEF-GOV-01.

---

## 6. Seed Data

| Table | Rows | Source |
|---|---|---|
| `Employee` | 25 | As built |
| `Equipment_Catalog` | 15 | As built (migrate to SharePoint list per DEF-ARC-04) |
| `Equipment_Request` | 30 | Prepared dataset |
| `Request_Audit` | 10 | Prepared dataset |

Status distribution across the 30 requests: Submitted 8 · Approved 10 · Fulfilled 10 · Rejected 2.

Seeding notes:

- All fulfilled requests carry an asset tag; approved and submitted requests do not. This makes AC-7 and DEF-REQ-01 both testable.
- Rejected requests carry approver notes; others are blank.
- Audit covers only the first ten requests — the visible evidence of DEF-REQ-02.
- Add one row with `Status = "Aproved"` to demonstrate the consequence of DEF-ARC-02.

---

## 7. Defect Coverage Matrix

| Agent | Defect IDs | Count |
|---|---|---|
| Code Review | DEF-CR-01 … DEF-CR-08 | 8 |
| Architecture | DEF-ARC-01 … DEF-ARC-06 | 6 |
| Security | DEF-SEC-01 … DEF-SEC-06 | 6 |
| Governance | DEF-GOV-01 … DEF-GOV-05 | 5 |
| Requirements | DEF-REQ-01 … DEF-REQ-03 | 3 |
| **Total** | | **28** |

Severity: Critical 4 · High 13 · Medium 7 · Low 4.

### Near-neighbour pairs

These pairs are the precision test. Each is one correct and one incorrect implementation of the same concept, in the same solution.

| Defect | Control | Concept under test |
|---|---|---|
| DEF-CR-01 | CTL-01 | Delegation |
| DEF-CR-04 | CTL-04 | Error handling |
| DEF-CR-06 | CTL-05 | Switch vs nested If |
| DEF-CR-07 | CTL-09 | Naming conventions |
| DEF-CR-02 | CTL-10 | Data loading strategy |
| DEF-ARC-01 | CTL-02 | Relationships vs free text |
| DEF-ARC-02 | CTL-03 | Choice vs text column |
| DEF-ARC-06 | CTL-08 | Environment variables |
| DEF-SEC-01 | CTL-07 | Column-level security |
| DEF-SEC-02 | CTL-06 | Role scoping |
| DEF-SEC-06 | CTL-11 | Connection ownership |
| DEF-REQ-01 | CTL-12 | Server-side rule enforcement |

> **CTL-04** sits on the New Request screen alongside DEF-CR-04. Implement a second, correctly guarded save on the same screen:
> ```powerfx
> IfError(
>     Patch('Equipment_Request', galRequests.Selected, {Justification: txtEditJustification.Text}),
>     Notify("Save failed. Please try again.", NotificationType.Error)
> )
> ```

---

## 8. Requirements Traceability Test Set

| ID | Scenario | Expected agent behaviour |
|---|---|---|
| RQ-1 | BR-004 requires approval before fulfilment; `scrAdmin` permits Draft → Fulfilled | Flag unimplemented mandatory requirement, cite BR-004, locate the fulfil action |
| RQ-2 | BR-007 requires all status changes audited; only create is audited | Flag partial implementation, cite BR-007, specify update events missing |
| RQ-3 | BR-009 states "handle high request volumes appropriately" with no threshold | Flag the requirement as untestable and request quantification — **not** a pass or fail |
| RQ-4 | "Delete All Requests" maps to no requirement | Flag unrequested functionality, recommend removal or a justifying requirement |

RQ-3 is scored separately. An agent that silently assumes a threshold is failing quietly, which is worse than reporting the gap. Correctly returning "this cannot be validated as written" is a full pass.

---

## 9. Build Order

| Phase | Work | Verify before proceeding |
|---|---|---|
| 1 | Environments, publisher, solution shell | DEF-GOV-01 exception logged with CoE |
| 2 | Four tables, columns, relationships | DEF-ARC-01/02/03, CTL-03 in place |
| 3 | Three security roles, app sharing | DEF-SEC-02/03/05, CTL-06/07 in place |
| 4 | Seed data across all four tables | Status distribution matches §6 |
| 5 | `scrHome`, `scrMyRequests`, `scrNewRequest` | DEF-CR-01/02/04/06/08, CTL-02/04 in place |
| 6 | `Screen2`, `scrAdmin`, orphaned screens | DEF-CR-03/05/07, DEF-REQ-01/03, CTL-01/05/09/10/12 in place |
| 7 | Flow A and Flow B | DEF-ARC-06, DEF-SEC-06, DEF-GOV-02, CTL-08/11 in place |
| 8 | Catalog migration to SharePoint | DEF-ARC-04 in place |
| 9 | Metadata and ownership left incomplete | DEF-GOV-03/04 in place |
| 10 | Export unmanaged solution, **skip Solution Checker** | DEF-GOV-05 satisfied by omission |

If time is compressed, build in agent-priority order: **Security → Requirements → Architecture → Code Review → Governance.** Security and Requirements carry all four Critical defects and are the strongest differentiators against existing static tooling.

---

## 10. Pre-Submission Checklist

- [ ] All 28 defects present and independently verified by a second person
- [ ] All 12 controls present and verified correct by the Tech SME
- [ ] Near-neighbour pairs both present in the same solution
- [ ] Seed data loaded; status distribution correct
- [ ] Audit table contains 10 rows, not 30
- [ ] Disclaimer banner visible on `scrHome`
- [ ] DEF-GOV-01 and DEF-GOV-02 pre-cleared with CoE admin
- [ ] Answer key version-controlled and hashed
- [ ] Clean BRD published to SharePoint as the sole agent input
- [ ] **This build spec is NOT in any location the agent can reach**
- [ ] Solution exported without a Solution Checker run
- [ ] Scorecard template ready for the run

---

## Appendix — Defect Quick Reference

| ID | Summary | Severity |
|---|---|---|
| DEF-CR-01 | Non-delegable filter using `Lower()` | High |
| DEF-CR-02 | Full-table `ClearCollect` in `App.OnStart` | High |
| DEF-CR-03 | Nested `ForAll` with record-by-record `Patch` | High |
| DEF-CR-04 | `Patch` with no error handling | Medium |
| DEF-CR-05 | Hardcoded threshold repeated four times | Medium |
| DEF-CR-06 | Six-level nested `If` | Low |
| DEF-CR-07 | Default control names, orphaned screens | Low |
| DEF-CR-08 | Unused global variables | Low |
| DEF-ARC-01 | Requester as free text, not a lookup | High |
| DEF-ARC-02 | Status as text, not a Choice | High |
| DEF-ARC-03 | Denormalised department and cost centre | Medium |
| DEF-ARC-04 | Catalog on SharePoint, rest on Dataverse | Medium |
| DEF-ARC-05 | Outside solution, default publisher | High |
| DEF-ARC-06 | Hardcoded URL and email | High |
| DEF-SEC-01 | No column security on cost and notes | **Critical** |
| DEF-SEC-02 | Organization Delete for Requester | **Critical** |
| DEF-SEC-03 | App shared far beyond pilot audience | High |
| DEF-SEC-04 | Client-side role variable | High |
| DEF-SEC-05 | Approver scope in gallery, not role | High |
| DEF-SEC-06 | Maker-owned implicit connection | Medium |
| DEF-GOV-01 | Solution in Default environment | High |
| DEF-GOV-02 | DLP-violating connector pair | **Critical** |
| DEF-GOV-03 | No description, owner, or support contact | Low |
| DEF-GOV-04 | Single owner, no co-owner | Medium |
| DEF-GOV-05 | No Solution Checker run before export | Medium |
| DEF-REQ-01 | Approval gate absent (BR-004) | **Critical** |
| DEF-REQ-02 | Audit on create only (BR-007) | High |
| DEF-REQ-03 | Unrequested delete-all action | High |
