# Feature Specification: SplitSmart Expense Sharing

**Feature Branch**: `not created (no before_specify hook configured)`

**Created**: 2026-10-29

**Status**: Draft

**Input**: Authoritative customer requirements in `UserRequirement.md`; create a testable SplitSmart specification, preserve the complete customer feature set, and distinguish it from the Java Swing MVP implementation decisions.

## Clarifications

### Session 2026-10-29

- Q: Can the person who pays an expense be different from the people sharing it, and how should that affect their balance? (FR-004, FR-010) → A: The payer may or may not be a participant. Credit the payer for the full amount paid, then offset their own allocation if they participated.
- Q: How should account sign-in and group membership be controlled? (FR-001–FR-003) → A: People sign in with a verified email address and password; invitees must accept group invitations; removing a member revokes their access while preserving their historical attribution.
- Q: What rules should validate expense amounts and percentage splits, and how should fractional minor units be assigned? (FR-005, NFR-002) → A: Amounts must be positive and use valid currency precision; percentages must be within 0–100 and total exactly 100%; allocate fractional minor units by largest remainder, breaking ties by stable account ID.
- Q: When should expense notifications be sent, and which channels should SplitSmart use? (FR-009) → A: Send in-app notices to other active group members when an expense is submitted, and send an in-app decision notice to the submitter when it is approved or rejected.
- Q: When a member records a payment, when should it reduce the displayed balance? (FR-011) → A: Record the payment as pending until the payee confirms receipt; only confirmed payments reduce the balance, and rejected payments leave it unchanged. Partial payments are allowed.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create an account and group (Priority: P1)

A person creates an account, joins or creates a group, and sees the group members and shared activity relevant to them.

**Why this priority**: Expense sharing requires identifiable participants and a shared context before expenses can be attributed.

**Independent Test**: Create two accounts, create a group with one account, add the second, and verify that both can access the group while an unrelated account cannot.

**Acceptance Scenarios**:

1. **AC-001** **Given** a person without an account, **When** they register with a valid email address and password and verify the email address, **Then** they can sign in and see only information they are authorized to access.
2. **AC-002** **Given** a signed-in person and an invitee, **When** the person invites them to a group and the invitee accepts, **Then** both people appear as active members and can access that group's shared records.
3. **AC-003** **Given** a person who is not a group member, **When** they attempt to view that group's expenses or balances, **Then** access is denied.
4. **AC-026** **Given** a member is removed from a group, **When** they attempt to access its records, **Then** access is denied while their attribution on historical expenses and payments remains intact for authorized members.

### User Story 2 - Submit a shared expense (Priority: P1)

A group member records an expense, identifies who paid and who participated, chooses an equal or percentage split, and may attach a receipt image.

**Why this priority**: Capturing the amount and participants is the source of the group's balance calculations.

**Independent Test**: Submit an expense with multiple participants under each split method and verify the submitted details and allocations before approval.

**Acceptance Scenarios**:

1. **AC-004** **Given** a group member and an active group, **When** they submit an expense with a payer, amount, date, description, participants, and equal split, **Then** the expense is saved for review with equal allocations whose total matches the expense amount.
2. **AC-005** **Given** a group member submitting an expense, **When** they provide valid participant percentages totaling 100%, **Then** the expense is saved with each participant's percentage allocation.
3. **AC-006** **Given** an expense with an optional receipt image, **When** the expense is submitted, **Then** the image is associated with that expense and visible only to authorized users.
4. **AC-007** **Given** an expense with missing required information or percentages not totaling 100%, **When** it is submitted, **Then** the system identifies the invalid fields and does not submit it for approval.
5. **AC-024** **Given** an approved expense whose payer is not a participant, **When** balances are calculated, **Then** the payer is credited for the amount paid and participants owe their allocated shares to the payer.
6. **AC-025** **Given** an approved expense whose payer is also a participant, **When** balances are calculated, **Then** the payer's own allocation offsets their credit for the amount paid.
7. **AC-027** **Given** an expense with a zero or negative amount, an amount exceeding the currency's permitted precision, or a percentage outside 0–100%, **When** it is submitted, **Then** the system rejects it and identifies the invalid value.
8. **AC-028** **Given** an equal or percentage split whose exact shares include fractional minor units, **When** allocations are calculated, **Then** the largest-remainder rule assigns the remaining minor units and resolves ties by stable account ID so allocations equal the expense total.

### User Story 3 - Review and approve an expense (Priority: P1)

An authorized group reviewer reviews a submitted expense and approves or rejects it before it changes balances.

**Why this priority**: The customer requires review before an expense is accepted into the balance record.

**Independent Test**: Submit one expense, inspect balances while it is pending, then approve it and verify that the approved expense affects balances; repeat with a rejected expense.

**Acceptance Scenarios**:

1. **AC-008** **Given** a submitted expense awaiting review, **When** an authorized reviewer approves it under the group's approval policy, **Then** its status becomes approved and it is included in balances.
2. **AC-009** **Given** a submitted expense awaiting review, **When** an authorized reviewer rejects it, **Then** its status becomes rejected and it does not affect balances.
3. **AC-010** **Given** a pending expense, **When** a member views group balances before approval, **Then** the pending expense is excluded.
4. **AC-011** **Given** an expense already resolved under the approval policy, **When** a reviewer attempts to resolve it again, **Then** the system prevents a duplicate or conflicting decision.

### User Story 4 - Understand balances and record a payment (Priority: P1)

A group member reviews what they owe or are owed by each other member and records a payment that settles all or part of a balance.

**Why this priority**: Accurate balances and settlement records are the primary practical outcome of expense sharing.

**Independent Test**: Approve expenses involving at least two members, verify each pairwise balance, record a partial and a final payment, and verify the remaining balance after each payment.

**Acceptance Scenarios**:

1. **AC-012** **Given** approved expenses involving multiple members, **When** a member opens group balances, **Then** the system shows the amount owed between each relevant pair of members.
2. **AC-013** **Given** an outstanding balance, **When** the debtor records a full or partial payment to the creditor, **Then** it is pending confirmation and does not yet reduce the displayed balance.
3. **AC-014** **Given** a payment amount greater than the outstanding amount, **When** the debtor attempts to record it against the balance owed to the creditor, **Then** the system prevents an invalid settlement and explains the amount available to settle.
4. **AC-015** **Given** a pending or rejected expense, **When** balances are calculated, **Then** that expense does not increase any member's balance.
5. **AC-030** **Given** a pending payment, **When** the payee confirms receipt, **Then** the payment becomes confirmed and the outstanding balance is reduced by its amount exactly once.
6. **AC-031** **Given** a pending payment, **When** the payee rejects it, **Then** the payment is marked rejected and the outstanding balance remains unchanged.

### User Story 5 - Receive expense notifications (Priority: P2)

A group member is notified when another member creates an expense in a group they belong to.

**Why this priority**: Timely notice lets members review activity and respond to expenses that require approval.

**Independent Test**: Create an expense as one member and verify that other members of the group receive a notification identifying the new expense without exposing it to non-members.

**Acceptance Scenarios**:

1. **AC-016** **Given** a group with multiple active members, **When** one member submits an expense, **Then** every other active member receives an in-app notice identifying the new expense.
2. **AC-017** **Given** a user outside the group, **When** an expense is submitted in that group, **Then** that user receives no notification and cannot access the expense.
3. **AC-029** **Given** a submitted expense, **When** an authorized reviewer approves or rejects it, **Then** the submitter receives an in-app notice of the decision.

### User Story 6 - Review expense, balance, and payment reports (Priority: P2)

A member reviews summaries and requests a report for a chosen date range and relevant group or record filters.

**Why this priority**: Reporting helps members understand spending and verify that balances and settlements are complete.

**Independent Test**: Create expenses and payments on dates inside and outside a selected range, then verify the report includes only matching records and totals.

**Acceptance Scenarios**:

1. **AC-018** **Given** a member with group activity, **When** they open a summary, **Then** it shows expenses, balances, and payments they are authorized to view.
2. **AC-019** **Given** a member selecting a date range and report filters, **When** they request a report, **Then** the results include matching records and correctly calculated totals and exclude records outside the range.
3. **AC-020** **Given** a report with no matching records, **When** it is requested, **Then** the system presents an empty result without reporting misleading nonzero totals.

### User Story 7 - Access SplitSmart on the required platforms (Priority: P1)

A customer uses SplitSmart through its primary web experience and through applications that run on Android and iOS, with their authorized account and group information available on the supported clients.

**Why this priority**: These platforms are explicitly part of the customer request and define the delivery target for the full product.

**Independent Test**: Complete an account and group workflow on each required client and verify that authorized group data and actions are available consistently.

**Acceptance Scenarios**:

1. **AC-021** **Given** the full product is released, **When** a customer accesses it on the web, **Then** the web experience is the primary supported experience.
2. **AC-022** **Given** the full product is released, **When** a customer uses an Android or iOS device, **Then** an application runs on that platform and supports the agreed customer workflows.
3. **AC-023** **Given** a user changes an approved shared record on one supported client, **When** they access it from another supported client, **Then** the same authorized record and resulting balance are shown.

### Edge Cases

- Equal and percentage splits can produce fractional minor units; assign residual units by largest remainder and resolve ties by stable account ID.
- Percentage values are outside 0–100% or do not sum to exactly 100%; reject the split.
- An expense amount is zero, negative, or has greater fractional precision than the currency permits; reject the amount.
- The payer is not included among the expense participants; this is allowed, and the payer is credited for the amount paid while participants owe their allocated shares. A participant who is not an active group member is invalid.
- A receipt image is absent, unreadable, or cannot be attached.
- A member is removed from a group after contributing to historical expenses; the removed member loses access, while historical records and balances remain attributed to them and reviewable by current authorized members.
- A reviewer tries to approve their own expense, acts without authorization, or submits conflicting decisions; apply the group's approval policy and preserve the pending state when policy is unmet.
- A payment is partial, duplicates a previously recorded payment, or exceeds the amount owed; pending and rejected payments must not reduce the balance.
- A report's start date is after its end date, its range has no matching records, or records fall on either date boundary.
- A member attempts to access another group's receipt, expense, report, or balance.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow each person to register and sign in with a verified email address and password to access their own expense and balance information. (CR-001)
- **FR-002**: System MUST allow authorized members to create groups, invite people, add invitees to membership only after invitation acceptance, and remove members. (CR-002)
- **FR-003**: System MUST restrict group expenses, receipts, balances, reports, and notifications to the account owner and active members of the relevant group. Removing a member MUST revoke their group access while preserving their attribution on historical records visible to current authorized members. (CR-001, CR-002)
- **FR-004**: System MUST allow an authorized member to submit an expense with amount, date, description, payer, split method, and the participating users. The payer MAY be outside the participant set; the payer MUST be credited for the full amount paid, with any payer allocation offsetting that credit. (CR-003)
- **FR-005**: System MUST support equal and percentage-based expense allocations. Expense amounts MUST be positive and MUST NOT exceed the active currency's permitted precision. Each percentage MUST be between 0% and 100%, inclusive, and percentages MUST total exactly 100%. The system MUST allocate fractional minor units by the largest-remainder method, breaking ties by stable account ID, so participant allocations equal the expense amount. (CR-003)
- **FR-006**: System MUST allow an expense submitter to attach an optional receipt image and associate it with the expense. (CR-003)
- **FR-007**: System MUST keep submitted expenses pending review until the group's approval policy is satisfied; pending expenses MUST NOT affect balances. (CR-005)
- **FR-008**: System MUST allow authorized reviewers to approve or reject an expense and record its status and decision; only approved expenses MUST affect balances. (CR-005, CR-006)
- **FR-009**: System MUST send an in-app notice to every other active member of the relevant group when an expense is submitted, and MUST send the submitter an in-app notice when the expense is approved or rejected. (CR-004)
- **FR-010**: System MUST calculate and display amounts owed between each relevant pair of users from approved expenses and confirmed payments. For each approved expense, the payer receives credit for the amount paid and each participant incurs their allocation as an amount owed; a participating payer's allocation offsets their credit. (CR-006, CR-007)
- **FR-011**: System MUST allow a debtor to record a full or partial payment against an outstanding balance as pending. The creditor MUST be able to confirm or reject it; only a confirmed payment MUST reduce the balance, and a rejected payment MUST leave the balance unchanged. (CR-007)
- **FR-012**: System MUST provide summaries of expenses, balances, and payments to authorized users. (CR-008)
- **FR-013**: System MUST allow authorized users to request reports using a date range and supported report parameters, including group and member filters. (CR-008)
- **FR-014**: The full product MUST provide a primarily web-based experience and applications that run on Android and iOS. (CR-009)

### Customer Requirements Inventory

These IDs preserve the customer's original scope independently of implementation decisions.

| ID | Customer requirement from `UserRequirement.md` | Addressed by | Verified by |
| --- | --- | --- | --- |
| CR-001 | Each person has an account and can access their own expense and balance information. | FR-001, FR-003 | AC-001, AC-003 |
| CR-002 | Users can create and manage groups of people. | FR-002, FR-003 | AC-002, AC-003, AC-026 |
| CR-003 | Users can create expenses with amount, date, description, equal or percentage split, optional receipt image, and involved users. | FR-004, FR-005, FR-006 | AC-004 through AC-007, AC-024, AC-025, AC-027, AC-028 |
| CR-004 | Group members are notified when another member creates an expense. | FR-009 | AC-016, AC-017, AC-029 |
| CR-005 | Expenses are reviewed and approved before entering the system and changing balances; groups may need one or multiple approval levels. | FR-007, FR-008 | AC-008 through AC-011 |
| CR-006 | The system tracks what each user owes each other user based on entered expenses. | FR-008, FR-010 | AC-008, AC-012, AC-015 |
| CR-007 | Users can record payments to settle balances. | FR-010, FR-011 | AC-013, AC-014, AC-030, AC-031 |
| CR-008 | Users receive expense, balance, and payment summaries and can request custom reports, including date ranges and other parameters. | FR-012, FR-013 | AC-018 through AC-020 |
| CR-009 | The product is primarily web-based and also has applications that run on Android and iOS. | FR-014 | AC-021 through AC-023 |

### Non-Functional Requirements

- **NFR-001**: System MUST prevent an account from accessing another user's private information or group data without group authorization.
- **NFR-002**: Monetary allocations, balances, and payment totals MUST reconcile exactly to the currency's minor unit; fractional allocations MUST use the largest-remainder method with stable account ID as the tie-breaker.
- **NFR-003**: For a usability evaluation with at least 10 first-time participants, at least 9 MUST complete account setup, join or create a group, and submit an expense without facilitator assistance.
- **NFR-004**: For a group with up to 10,000 historical expense and payment records, at least 95% of balance and report views MUST be available to the user within 5 seconds under the agreed test conditions.
- **NFR-005**: Receipt images and financial records MUST be visible only to their owner and authorized group members, and user-facing failures MUST not expose another user's information.
- **NFR-006**: A full-product release MUST provide the customer workflows defined by this specification on the primary web experience and on both Android and iOS applications.

### Key Entities *(include if data involved)*

- **Account**: A person and their private access to SplitSmart.
- **Group**: A shared expense context created and managed by members.
- **Membership**: A person's membership and authorization within a group.
- **Expense**: A proposed or approved shared charge, including amount, date, description, payer, participants, split method, allocations, optional receipt, and review status.
- **Allocation**: A participant's share of an expense under the selected split method.
- **Approval Decision**: A reviewer's decision and the resulting expense status under a group's approval policy.
- **Notification**: In-app notice to active group members about an expense submission, or to its submitter about an approval decision.
- **Payment**: A manually recorded full or partial transfer between a debtor and creditor; it remains pending until confirmed or rejected by the creditor, and only confirmation reduces the outstanding balance.
- **Report**: A requested summary of authorized expenses, balances, and payments for selected parameters and a date range.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: All acceptance tests for equal and percentage splits reconcile participant allocations to the expense total with no minor-unit discrepancy.
- **SC-002**: At least 9 of 10 first-time usability participants complete account setup, join or create a group, and submit an expense without help.
- **SC-003**: At least 95% of balance and report views for 10,000 historical records are available within 5 seconds under the agreed test conditions.
- **SC-004**: In all approval and settlement workflow tests, pending and rejected expenses cause zero balance change, approved expenses update affected balances exactly once, and only confirmed payments reduce outstanding balances.
- **SC-005**: At least 9 of 10 usability participants can identify a selected group's current pairwise balances and payment history within 60 seconds.
- **SC-006**: A full-product release passes the account, group, expense, balance, and payment acceptance scenarios on the primary web experience and on Android and iOS.
- **SC-007**: At least 8 of 10 usability participants rate the expense, approval, and settlement history as understandable at 4 or higher on a 5-point scale.

## Assumptions

- Each expense uses one currency; conversion between currencies is outside the stated customer scope.
- A submitted expense has one payer. A payment is a manually recorded settlement, not a transfer initiated by SplitSmart.
- In-app notification is the only specified channel; email, SMS, and push delivery are out of scope unless selected later.
- Report parameters include group, member, record type, and date range; additional custom dimensions are not assumed.
- The Java Swing MVP is an implementation subset for desktop evaluation. It does not satisfy the separate customer requirement for a primary web product and Android/iOS applications.
- Performance criteria will be measured on test conditions agreed before verification; no production traffic or availability target was provided.

## Implementation Decisions (Separate from Customer Requirements)

### Java Swing MVP Subset

- **MVP-001**: The initial desktop MVP uses Java 21, Maven, and Java Swing.
- **MVP-002**: The Swing MVP subset covers account and group workflows, expense submission, review and approval, notifications, balances, payment recording, and reports.
- **MVP-003**: Java business rules are separated from Swing presentation and event handling; money calculations use `BigDecimal`; automated Java tests use JUnit 5; maintained architecture and interaction diagrams use PlantUML.
- **MVP-004**: AI-generated work is reviewed and verified by a human before acceptance.

These are implementation decisions governed by the project constitution, not customer-originated requirements. A Swing-only MVP MUST NOT be presented as delivery of CR-009. The product-level platform decision remains subject to customer confirmation.

## Open Questions

1. **Platform rollout**: Is the Java Swing MVP a desktop prototype before the required primary web, Android, and iOS clients, or is desktop itself an additional supported client? Which release must deliver each customer-required platform?
2. **Approval policy**: Should groups choose a single reviewer or multiple reviewers, and may an expense submitter approve their own expense? The default assumption is one designated reviewer and no self-approval.
