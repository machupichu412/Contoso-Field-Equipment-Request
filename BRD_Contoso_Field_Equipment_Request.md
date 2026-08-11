# Business Requirements Document
## Contoso Field Equipment Request (FER)

Equipment request, approval, and fulfilment management

| Field | Detail |
|---|---|
| Document title | Business Requirements Document – Contoso Field Equipment Request |
| Application | Contoso Field Equipment Request (FER) |
| Version | 1.0 |
| Date | 27 July 2026 |
| Author | Matthew Yeh |
| Business owner | Contoso Operations |
| Platform | Microsoft Power Platform |
| Status | Approved for build |

---

## 1. Executive Summary

Contoso field employees currently request equipment such as laptops, monitors, headsets, and networking hardware through email and ad-hoc spreadsheets. There is no consistent record of who requested what, whether it was approved, who approved it, or whether it was ever delivered. Managers approve informally and Operations reconciles fulfilment manually at month end.

The Field Equipment Request (FER) application replaces that process with a single system of record. A requester submits a structured request against a maintained equipment catalog, their manager approves or rejects it, and an Operations Administrator fulfils approved requests and records the issued asset tag. Every request carries a visible status from submission through to fulfilment, and every change is recorded for audit.

**Expected outcome:** a complete, auditable request history; a clear approval trail tied to the requester's manager; accurate equipment cost visibility for Operations and Finance; and removal of the manual reconciliation effort currently absorbed at month end.

---

## 2. Background and Business Case

### 2.1 Current State

Equipment requests today arrive by email, in varying formats, with inconsistent detail. The consequences are consistent across departments:

- Requests are submitted without justification, quantity, or a valid catalog item, requiring back-and-forth before they can be actioned.
- Approval is verbal or by email reply, with no durable record of who approved a request or when.
- Requesters have no visibility of status and chase Operations directly for updates.
- Issued equipment is not reliably linked back to the request that authorised it, so asset ownership is incomplete.
- Cost exposure is unknown until after fulfilment, removing any opportunity for budget control at the point of approval.
- No audit trail exists, so the process cannot evidence that spend was properly authorised.

### 2.2 Business Drivers

| Driver | Description |
|---|---|
| Spend control | Estimated cost must be visible to the approver before a request is authorised |
| Auditability | Every request must carry a durable record of who did what and when |
| Asset accountability | Issued equipment must be traceable to the request that authorised it |
| Requester experience | Status must be self-service rather than obtained by chasing Operations |
| Operational efficiency | Remove manual reconciliation of email-based requests at month end |
| Data quality | Requests must reference a maintained catalog rather than free-text descriptions |

---

## 3. Business Objectives and Success Measures

| ID | Objective | Success measure |
|---|---|---|
| OBJ-1 | Centralise all equipment requests in a single system of record | 100% of requests submitted through the application within one quarter of launch |
| OBJ-2 | Ensure every fulfilled request was formally approved | Zero fulfilled requests without a recorded approval decision |
| OBJ-3 | Provide requesters with self-service status visibility | 50% reduction in status enquiries raised with Operations |
| OBJ-4 | Establish a complete audit trail for every request | Every request has a retrievable history of its status changes |
| OBJ-5 | Give approvers cost visibility at the point of decision | Estimated cost is present on every request presented for approval |
| OBJ-6 | Link issued equipment to its authorising request | Every fulfilled request carries a recorded asset tag |

---

## 4. Scope

### 4.1 In Scope

- Submission of equipment requests by field employees against a maintained catalog.
- Manager approval and rejection, including a mandatory reason on rejection.
- Fulfilment by Operations, including recording of the issued asset tag.
- Status visibility for requesters across the full request lifecycle.
- Maintenance of the equipment catalog by Operations, including activation and deactivation of items.
- Notification to the approver on submission, and to the requester on decision.
- An audit history of request activity.
- Role-based access covering Requester, Approver, and Operations Administrator.

### 4.2 Out of Scope

- Procurement and purchase order raising with external suppliers.
- Inventory and stock level management.
- Equipment return, recovery, or disposal processes.
- Integration with the corporate asset management system, deferred to a later phase.
- Budget holder approval above the manager level, deferred to a later phase.
- Mobile phone and telecoms requests, which follow a separate process.
- Financial posting or chargeback to cost centres.

---

## 5. Stakeholders and User Roles

### 5.1 Stakeholders

| Stakeholder | Interest |
|---|---|
| Contoso Operations | Owns the process; requires accurate fulfilment records and catalog control |
| Department managers | Approve spend for their direct reports and require cost visibility |
| Field employees | Require a simple submission route and visible status |
| Finance | Requires evidence that equipment spend was properly authorised |
| IT Asset Management | Requires issued equipment to be traceable to an authorised request |

### 5.2 User Roles

| Role | Description | Capabilities |
|---|---|---|
| Requester | Any field employee | Submit requests; view own requests and their status |
| Approver | The requester's line manager | Review, approve, or reject requests from their direct reports; record approver notes |
| Operations Administrator | Contoso Operations team member | View all requests; fulfil approved requests; record asset tags; maintain the equipment catalog |

---

## 6. Business Process

### 6.1 Process Flow

1. The requester opens the application and creates a new request.
2. The requester selects an item from the active equipment catalog, enters a quantity, and provides a business justification.
3. On submission, the request is recorded with the requester's identity and moves to Submitted status.
4. The requester's manager is notified that a request is awaiting their decision.
5. The approver reviews the request, including its estimated cost, and either approves it or rejects it with a reason.
6. The requester is notified of the decision.
7. Approved requests appear in the Operations fulfilment queue.
8. The Operations Administrator issues the equipment, records the asset tag, and marks the request Fulfilled.

### 6.2 Request Statuses

| Status | Meaning | Set by |
|---|---|---|
| Draft | Started but not yet submitted | Requester |
| Submitted | Awaiting manager decision | Requester |
| Approved | Authorised and awaiting fulfilment | Approver |
| Rejected | Declined with a recorded reason | Approver |
| Fulfilled | Equipment issued and asset tag recorded | Operations Administrator |

### 6.3 Business Rules

- A request must reference an item that is currently active in the equipment catalog.
- Quantity must be a whole number of one or more.
- Business justification is mandatory on every request.
- A request must be approved before it can be fulfilled.
- A rejection must carry a reason.
- A request may not be edited once it has been approved.
- Estimated cost is derived from the catalog unit cost multiplied by the requested quantity.
- An approver may only action requests raised by their own direct reports.

---

## 7. Business Requirements

Requirements are prioritised as **Must** (required for launch) or **Should** (required for full business benefit, may follow shortly after launch).

### 7.1 Request Submission

| ID | Requirement | Priority |
|---|---|---|
| BR-001 | A requester can submit a request specifying item, quantity, and business justification | Must |
| BR-002 | A request must record the requester identity automatically at submission | Must |
| BR-010 | Only active catalog items may be selected on a new request | Should |

### 7.2 Visibility and Status

| ID | Requirement | Priority |
|---|---|---|
| BR-003 | A requester can view the status of their own requests at any time | Must |
| BR-006 | Estimated cost and approver notes must not be visible to the requester | Must |

### 7.3 Approval

| ID | Requirement | Priority |
|---|---|---|
| BR-004 | A request must be approved by a manager before it can be fulfilled | Must |
| BR-005 | An approver can approve or reject with a mandatory reason on rejection | Must |
| BR-012 | Requests may not be edited after approval | Should |

### 7.4 Fulfilment

| ID | Requirement | Priority |
|---|---|---|
| BR-008 | An Operations Administrator can mark an approved request as fulfilled and record an asset tag | Must |

### 7.5 Audit and Notification

| ID | Requirement | Priority |
|---|---|---|
| BR-007 | Every status change must be written to an immutable audit trail | Must |
| BR-011 | A requester must be notified when their request is approved or rejected | Should |

### 7.6 Non-Functional

| ID | Requirement | Priority |
|---|---|---|
| BR-009 | The system should handle high request volumes appropriately | Should |

---

## 8. Data Requirements

The application maintains four business entities.

### 8.1 Equipment Request

One record per request raised. This is the core transactional entity.

| Attribute | Description |
|---|---|
| Request Number | Unique identifier for the request |
| Requester | The employee raising the request |
| Item Name | The catalog item being requested |
| Quantity | Number of units requested |
| Justification | Business reason for the request |
| Status | Current position in the request lifecycle |
| Estimated Cost | Catalog unit cost multiplied by quantity |
| Approver Notes | Decision commentary recorded by the approver |
| Asset Tag | Identifier of the equipment issued on fulfilment |
| Department | Requester's department |
| Cost Center | Requester's cost centre |
| Created On | Date the request was raised |

### 8.2 Equipment Catalog

The list of items that may be requested, maintained by Operations.

| Attribute | Description |
|---|---|
| Item Name | Name of the requestable item |
| Category | Grouping such as Computer, Peripheral, AV, Network, or Storage |
| Unit Cost | Cost per unit, used to derive estimated cost |
| Active | Whether the item may currently be requested |

### 8.3 Employee

Reference data identifying requesters and their approving manager.

| Attribute | Description |
|---|---|
| Employee Name | Full name of the employee |
| Email | Corporate email address, used as the unique identifier |
| Manager Email | Email address of the approving manager |
| Department | Department the employee belongs to |
| Cost Center | Cost centre the employee is assigned to |

### 8.4 Request Audit

The history of activity against a request.

| Attribute | Description |
|---|---|
| Request | The request the entry relates to |
| Action | The activity recorded |
| Actor | The person who performed the activity |
| Timestamp | Date and time the activity occurred |

---

## 9. Non-Functional Requirements

| Area | Requirement |
|---|---|
| Availability | Available during Contoso business hours across all operating regions |
| Access | Accessible to authenticated Contoso employees on corporate devices |
| Authentication | Single sign-on using corporate credentials; no separate login |
| Authorisation | Access is determined by assigned role; users see only what their role permits |
| Data retention | Request and audit records retained in line with Contoso retention policy |
| Usability | A requester can complete a submission without training or written guidance |
| Device support | Optimised for tablet and desktop use |
| Volume | The system should handle high request volumes appropriately |

---

## 10. Assumptions, Dependencies and Constraints

### 10.1 Assumptions

- Employee and manager relationships are available and maintained as reference data.
- Every requester has an assigned manager who can act as approver.
- Operations maintains the equipment catalog, including unit costs.
- Unit costs are indicative for approval purposes and are not the final invoiced amount.
- All users have corporate accounts and can authenticate with single sign-on.

### 10.2 Dependencies

| Dependency | Owner |
|---|---|
| Provision of employee and manager reference data | HR Systems |
| Initial population and ongoing maintenance of the equipment catalog | Contoso Operations |
| Assignment of users to application roles | IT Administration |
| Confirmation of data retention requirements | Compliance |

### 10.3 Constraints

- The solution must be delivered on Microsoft Power Platform.
- No integration with the corporate asset management system is available in this phase.
- Approval is limited to a single manager level; multi-level approval is not supported in this phase.

---

## 11. Acceptance Criteria

The application is accepted when the following scenarios pass in the test environment.

| ID | Scenario | Expected result |
|---|---|---|
| AC-1 | A requester submits a request with item, quantity, and justification | Request is created with Submitted status and the requester recorded automatically |
| AC-2 | A requester views their request list | Only their own requests are returned, each showing current status |
| AC-3 | A requester attempts to view cost or approver notes | Neither value is available to the requester |
| AC-4 | An approver approves a request | Status becomes Approved and the requester is notified |
| AC-5 | An approver rejects without entering a reason | The rejection is prevented until a reason is supplied |
| AC-6 | An attempt is made to fulfil a request that has not been approved | The action is prevented |
| AC-7 | An Operations Administrator fulfils an approved request | Status becomes Fulfilled and the asset tag is recorded |
| AC-8 | A request is edited after approval | The edit is prevented |
| AC-9 | A request passes through submission, approval, and fulfilment | Each status change is retrievable from the audit trail |
| AC-10 | A requester selects an item from the catalog | Only items marked Active are available for selection |

---

## 12. Approval

This document requires sign-off from the following parties before build commences.

| Role | Name | Signature | Date |
|---|---|---|---|
| Business owner | | | |
| Operations lead | | | |
| Finance representative | | | |
| IT delivery lead | | | |

---

## Appendix A – Glossary

| Term | Definition |
|---|---|
| Approver | The line manager responsible for authorising a request |
| Asset tag | Unique identifier applied to issued equipment |
| Catalog item | An item of equipment available to be requested |
| Cost centre | Financial code against which equipment cost is attributed |
| Estimated cost | Catalog unit cost multiplied by requested quantity |
| Fulfilment | The act of issuing approved equipment and recording its asset tag |
| Requester | The employee raising an equipment request |
