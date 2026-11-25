# Feature Specification: Cheque Clearing Eligibility and Exchange Preparation (Canada)

**Feature Branch**: `001-clearing-exchange-prep`
**Created**: 2026-11-25
**Status**: Draft
**Input**: User description: "Cheque clearing eligibility and exchange preparation (Canada)", given in full in `specPrompt.md`, lines 3 to 92
**User Interface**: yes
**Modernization**: no

## Purpose

A bank sends Canadian-dollar items to other banks as paper and as images. For each item,
this system decides whether it may be exchanged, shows which rule section drove the
decision, and prepares eligible items as paper bundles or image file groups. The users are
clearing operations staff.

The rules are Payments Canada Rule A1 (read as amended to 2026-10-26) and Rule A10 (as
amended to 2026-11-17). They are paraphrased here with section numbers. Their text is not
reproduced. "A1 s.9" means section 9 of Rule A1.

Discovery evidence is in `discovery/clearing-eligibility/`.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Decide whether an item may be exchanged (Priority: P1)

An item is submitted with its data. The system checks it and records an outcome of
eligible, waiting, needs review or not eligible, with the reasons and rule sections.

**Why this priority**: Nothing can be sent safely without this decision, and it has value
on its own.

**Independent Test**: Submit synthetic items covering each check and boundary, and read
back the outcome, reasons and sections for each.

**Acceptance Scenarios**:

1. **Given** a complete item of an acceptable kind, drawn on a member, dated today and
   under the limit, **When** it is checked, **Then** it is eligible and the decision lists
   the sections checked, the time and the rule version.
2. **Given** an item dated after today, **When** it is checked, **Then** it is waiting and
   the decision cites A1 s.10.
3. **Given** a waiting post-dated item, **When** its date arrives, **Then** it is checked
   again without anyone asking and gets a new decision.
4. **Given** an item that shows a month and year but no day, **When** it is checked before
   the first business day of that month, **Then** it is waiting until that day, citing A1
   s.11.
5. **Given** an item marked as disputed, **When** it is checked, **Then** it is not
   eligible, citing A1 s.9.
6. **Given** an item flagged as directing protest or notice of dishonour, **When** it is
   checked, **Then** it needs review, citing A1 s.12.
7. **Given** an item of exactly CAD 25,000,000.00, **When** it is checked, **Then** the
   amount limit does not stop it; **Given** CAD 25,000,000.01 with no exception, **Then**
   it is not eligible, citing A1 s.13.
8. **Given** an item over the limit that is an inter-member payment for one of the three
   listed purposes, **When** it is checked, **Then** the limit does not stop it and the
   decision records the exception used.
9. **Given** an item flagged as possibly split to avoid the limit, **When** it is checked,
   **Then** it needs review.
10. **Given** an item with a missing or unreadable date, **When** it is checked, **Then**
    it needs review and is never eligible by default.
11. **Given** an item of a kind that may not be exchanged as an image, with image as its
    method, **When** it is checked, **Then** it is not eligible, citing A10 s.8b.
12. **Given** a Canadian postal money order not drawn on a member, **When** it is checked,
    **Then** it is not rejected for membership, citing A1 s.5.

---

### User Story 2 - Change the rules on a set date (Priority: P2)

A new rule version is loaded ahead of time and takes effect on a chosen date. Decisions
already made keep the version that made them.

**Why this priority**: Both rules were amended recently. Without this, an amendment cannot
take effect on its date and old decisions cannot be explained.

**Independent Test**: Load a version with a future date and a different limit, check the
same item before and after that date, and compare the two decisions.

**Acceptance Scenarios**:

1. **Given** a version loaded with a future effective date, **When** an item is checked
   before that date, **Then** the current version is used.
2. **Given** the same version, **When** an item is checked on or after its date, **Then**
   the new version is used and recorded on the decision.
3. **Given** a decision made under an earlier version, **When** it is viewed later, **Then**
   it still shows that version, its reasons and its sections.
4. **Given** a paper pre-authorized debit, **When** it is checked on 2028-11-30, **Then**
   its kind is acceptable; **When** it is checked on 2028-12-01, **Then** it is not.

---

### User Story 3 - Prepare paper bundles (Priority: P3)

Eligible paper items whose preparation is complete are sorted into sorting streams and
placed in bundles for each receiving institution, with a list for each bundle and a
clearing log.

**Why this priority**: It turns paper decisions into something that can be sent.

**Independent Test**: Submit eligible paper items for two receivers with mixed streams,
close the bundles, and check contents, order, lists, totals and the log.

**Acceptance Scenarios**:

1. **Given** 650 eligible encoded items for one receiver, **When** they are grouped and the
   cut-off passes, **Then** there are bundles of 300, 300 and 50 in the order the items
   were added.
2. **Given** an eligible item with a required preparation step not recorded, **When**
   grouping runs, **Then** it is held out of every bundle and the missing step is shown.
3. **Given** an item whose attachment cannot be removed, **When** it is grouped, **Then**
   it goes to the unqualified stream and no encoded amount is required.
4. **Given** a paper remittance, **When** it is grouped, **Then** it goes to the remittance
   stream.
5. **Given** a closed bundle, **When** its list is viewed, **Then** it shows items in
   order, the total, the processing date and the delivering Direct Clearer.

---

### User Story 4 - Prepare image file groups (Priority: P4)

Eligible image items with usable images are placed in image file groups, separated as Rule
A10 requires.

**Why this priority**: It covers the second exchange method. Producing the file itself is
left for later.

**Independent Test**: Submit eligible image items that differ in each separation key and
check the groups formed.

**Acceptance Scenarios**:

1. **Given** 90,000 eligible items with the same keys, **When** they are grouped and the
   cut-off passes, **Then** there are groups of 40,000, 40,000 and 10,000.
2. **Given** items that differ only in processing site, **When** grouped, **Then** they are
   in different groups.
3. **Given** an agreement with a receiver to combine two exchange points, **When** items
   for those points are grouped, **Then** they may share a group.
4. **Given** an item whose capture result says its image is not usable or its black and
   white image is outside 200 to 240 DPI, **When** grouping runs, **Then** it is held out
   and the reason is shown.

---

### User Story 5 - Work decisions and groups on an operations screen (Priority: P5)

Operations staff see each decision with its reasons, work the items that need review, see
group contents, reopen a closed group when something is wrong, and mark a group exchanged.

**Why this priority**: People have to resolve what the checks cannot. The earlier stories
can be tested without a screen.

**Independent Test**: Using prepared synthetic items and groups, find a decision and its
reasons, resolve a review item, reopen and re-close a group, and mark it exchanged.

**Acceptance Scenarios**:

1. **Given** a not eligible item, **When** a staff member opens it, **Then** they see each
   reason, its rule section, the time and the rule version.
2. **Given** items that need review, **When** a staff member opens the review list,
   **Then** each item is shown with why it needs review.
3. **Given** a closed group that has not been exchanged, **When** an authorized person
   reopens it with a reason, **Then** it can be changed and the change, reason, person and
   time are recorded.
4. **Given** a closed group, **When** a person or the sending system reports it exchanged,
   **Then** the group and its items become exchanged and cannot be changed.
5. **Given** an exchanged group, **When** anyone tries to reopen or change it, **Then** the
   attempt is refused.
6. **Given** an item with a wrong protest or splitting flag, **When** an operator removes
   the flag with a reason, **Then** the checks run again and decide the outcome.
7. **Given** an item with a correct flag, **When** an operator confirms it with a reason,
   **Then** the item is not eligible.
8. **Given** an operator, **When** they try to reopen a group or load a rule version,
   **Then** the attempt is refused.
9. **Given** a reopened group, **When** a supervisor closes it again, **Then** its list and
   totals are produced again.
10. **Given** a flagged closed group, **When** a person marks it exchanged, **Then** they
    are warned first and the record shows it was exchanged while flagged.

### Edge Cases

- The same item identity is submitted twice: no second item is created. Detecting the same
  cheque captured twice under two identities is out of scope.
- New information, such as a dispute, arrives for an item already in an open group: the
  item is rechecked and, if no longer eligible, removed from the group.
- The same happens for an item in a closed, unexchanged group: the group and item are
  flagged for staff, and the contents stay fixed until an authorized person reopens it.
- New information arrives for an exchanged item: the group does not change.
- A cut-off passes with an open group holding no items: no empty bundle, list or file group
  is produced.
- An item becomes eligible after the cut-off: it joins a new group for the next cut-off.
- A first business day falls after weekends or holidays at the start of a month: the
  month-dated item waits until that day.
- A drawee is not in the member list and no exception applies: the item is not eligible. If
  the list itself is unavailable, the item needs review.
- A non-Canadian-dollar item is submitted: it is refused as outside this system's scope.
- Two actions reach a group at once, such as closing while an item is added: the item is
  either in the closed contents and list, or in a new group, never half of each.

## Requirements *(mandatory)*

### Functional Requirements

**Items and decisions**

- **FR-001**: System MUST treat a cheque and its image as one item with one identity and one decision history, exchanged by one method only. [F-CLR-67] [F-CLR-72]
- **FR-002**: System MUST give each submitted Canadian-dollar item an outcome of eligible, waiting, needs review or not eligible, for operations staff to act on. [F-CLR-2] [F-CLR-41] [F-CLR-68]
- **FR-003** (constrained, documented): Every decision MUST record its outcome, each reason, the rule section behind it, the time and the rule version. For an eligible item it records the sections checked. [F-CLR-3] [F-CLR-65]
- **FR-004** (constrained, documented): System MUST check that the item is of an acceptable kind, is drawn on a member or a qualifying local, is payable on demand, is identified as required, and meets the value and value date conditions (A1 s.2, s.3). [F-CLR-4] [F-CLR-5]
- **FR-005** (constrained, documented): System MUST apply the listed exceptions to the membership condition (A1 s.5). [F-CLR-6]
- **FR-006** (constrained, documented): System MUST hold a post-dated item as waiting until its date (A1 s.10). [F-CLR-8] [F-CLR-69]
- **FR-007** (constrained, documented): System MUST hold an item with no day of the month as waiting until the first business day of the month it shows (A1 s.11). [F-CLR-9] [F-CLR-69]
- **FR-008**: System MUST check a waiting item again on the date it can qualify, without a person asking. [F-CLR-69]
- **FR-009** (constrained, documented): System MUST mark a disputed item not eligible (A1 s.9). [F-CLR-7] [F-CLR-71]
- **FR-010** (constrained, documented): System MUST mark an item that directs protest or notice of dishonour as needs review (A1 s.12). [F-CLR-10] [F-CLR-71]
- **FR-011** (constrained, documented): System MUST mark an item over CAD 25,000,000 not eligible unless it is an inter-member payment that manages settlement balances, corrects a flow imbalance between the two clearing systems, or pays during a Lynx emergency (A1 s.13a). [F-CLR-11] [F-CLR-71]
- **FR-012** (constrained, documented): System MUST mark an item flagged by capture or staff as possibly split to avoid the limit as needs review (A1 s.13b). It does not detect splitting itself. [F-CLR-12] [F-CLR-34] [F-CLR-71] [F-CLR-80]
- **FR-013**: System MUST mark an item with missing or unclear data as needs review, and MUST NOT treat it as eligible by default. [F-CLR-70]
- **FR-014**: System MUST apply the same eligibility checks, including the amount limit, to items exchanged as images as to paper. [F-CLR-42] [F-CLR-61] [F-CLR-62]
- **FR-015** (constrained, documented): System MUST mark as not eligible for image exchange Canadian-dollar bonds and coupons, paper remittances, the listed Government of Canada securities and returned item envelopes (A10 s.8b). [F-CLR-43]
- **FR-016**: System MUST compute date and amount conditions from item data, obtain membership and dispute status from sources outside the item, and accept protest wording, damage, unremovable attachments and intent to split as flags from capture or staff. [F-CLR-32] [F-CLR-33] [F-CLR-34]

**Rule versions**

- **FR-017**: System MUST allow a rule version to be loaded before it applies and to take effect on a chosen date. [F-CLR-64] [F-CLR-65]
- **FR-018**: System MUST keep every past decision with the version that made it; a new version MUST NOT rewrite earlier decisions. [F-CLR-65]
- **FR-019** (constrained, documented): System MUST accept paper pre-authorized debits as a kind when checked before 2028-12-01 and not from that date. [F-CLR-28]

**Paper bundles**

- **FR-020** (constrained, documented): System MUST record the preparation steps for a paper item (foreign attachments removed, amount and other required fields encoded, errors in amount, currency and listing corrected, item imaged or filmed or another arrangement made with the receiver) and MUST keep an item with a required step missing out of every bundle. The system does not perform the steps (A1 s.15, s.16, s.18, s.21). [F-CLR-1] [F-CLR-14] [F-CLR-15] [F-CLR-16] [F-CLR-19] [F-CLR-26]
- **FR-021** (constrained, documented): System MUST treat an item whose attachment cannot be removed as unqualified, with no amount encoding required (A1 s.15). [F-CLR-14]
- **FR-022** (constrained, documented): System MUST NOT repair rejected fields other than the amount; it records that a replacement document is needed (A1 s.16c, s.16d). [F-CLR-17]
- **FR-023** (constrained, documented): System MUST sort paper items into encoded, unqualified and remittance sorting streams (A1 s.19). [F-CLR-20] [F-CLR-22]
- **FR-024** (constrained, documented): System MUST be able to place unqualified items of CAD 50,000 or more in the encoded stream (A1 s.19a). [F-CLR-21]
- **FR-025** (constrained, documented): Each paper bundle MUST hold one sorting stream for one receiving Direct Clearer and at most 300 items, and MUST keep their order (A1 s.19, s.20a). [F-CLR-23] [F-CLR-74]
- **FR-026** (constrained, documented): System MUST produce, for each bundle, a totalled list in item order showing the processing date and delivering Direct Clearer, and a clearing log for each exchange of bundles (A1 s.20b, s.20c). [F-CLR-24]

**Image file groups**

- **FR-027** (constrained, documented): System MUST keep an image item out of every group unless it has front and back images that capture reports as usable, including a black and white image of 200 to 240 DPI (A10 s.9 to s.11). [F-CLR-44] [F-CLR-45] [F-CLR-46]
- **FR-028** (constrained, documented): Each image file group MUST be unique to a receiving Direct Clearer, exchange point, processing site, currency and direction, except where the two clearers have agreed to combine exchange points (A10 s.24). [F-CLR-50] [F-CLR-74]
- **FR-029** (constrained, documented): An image file group MUST hold at most 40,000 items (A10 s.24d). [F-CLR-49]

**Group lifecycle**

- **FR-030**: Only an eligible item MUST be able to join a group, and it MUST be in no more than one group. Placing it makes it prepared. [F-CLR-72]
- **FR-031**: A group MUST move from open to closed to exchanged. It closes when full or at a cut-off time set by operations, whichever comes first; its contents are then fixed and its list and totals produced. [F-CLR-73] [F-CLR-75]
- **FR-032**: A supervisor MUST be able to reopen a closed group before it is exchanged, with the change and reason recorded, and to close it again without waiting for the next cut-off, which produces its list and totals again. [F-CLR-77] [F-CLR-82]
- **FR-033**: System MUST mark a group exchanged when a person or the sending system reports it, and MUST NOT allow an exchanged group to change. [F-CLR-76] [F-CLR-77]
- **FR-034**: When new information makes an item in an unexchanged group no longer eligible, system MUST remove it from an open group, or flag a closed group for staff. A flagged group MAY still be marked exchanged: a person is warned first, and the exchange of a flagged group MUST be recorded. [F-CLR-3] [F-CLR-7] [F-CLR-77] [F-CLR-83]

**Operations screen and access**

- **FR-035**: Operations staff MUST be able to see, on a screen, each item's decisions with reasons and sections, the items that need review, and the contents, list and totals of each group. [F-CLR-2] [F-CLR-3]
- **FR-036**: To resolve a needs-review item, a person MUST either correct its data or flag, which runs the checks again, or confirm that a protest or splitting flag is right, which makes the item not eligible. Each resolution MUST record who, when and why. A person MUST NOT be able to mark an item eligible directly. [F-CLR-70] [F-CLR-71] [F-CLR-81]
- **FR-037**: System MUST enforce these permissions: operators view, resolve reviews and mark groups exchanged; supervisors do the same and also reopen groups and close them again; rule administrators view and load rule versions. The sending system may only report groups exchanged. [F-CLR-76] [F-CLR-82]
- **FR-038**: Screens, lists and records MUST say "sorting stream" for the paper classes and "settlement stream" for the identifier on an image settlement entry. [F-CLR-78]
- **FR-039**: Explanations MUST cite rule sections in the system's own words and MUST NOT reproduce rule text. [F-CLR-31]

### Key Entities

- **Item**: A payment item the bank may send, in paper and image form. One identity from
  submission, with its kind, amount, currency, date, drawee, exchange method, flags,
  preparation steps and image quality result. Moves from captured to assessed, prepared and
  exchanged.
- **Eligibility decision**: The result of one check of one item: outcome, reasons, rule
  sections, time and rule version. A new check adds a decision.
- **Rule version**: The checks, limits and dates in force from a given date.
- **Exchange group**: A paper bundle or image file group for one receiver, with its
  separation keys, ordered items, totals and state (open, closed, exchanged).
- **Clearing log**: The record of one exchange of paper bundles.
- **Reference data**: Members and listed exceptions, dispute status, receivers, exchange
  points, processing sites, combination agreements and the business day calendar.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001** (constrained, documented): 100% of decisions show their outcome, rule sections, time and rule version. [F-CLR-3] [F-CLR-65]
- **SC-002**: A synthetic test set covering every check, exception and boundary in this specification gives the expected outcome for 100% of its items. [F-CLR-68]
- **SC-003**: No item that is waiting, needs review or is not eligible appears in the contents of a group when it closes. [F-CLR-70] [F-CLR-72]
- **SC-004** (constrained, documented): No paper bundle exceeds 300 items or mixes streams or receivers, and no image file group exceeds 40,000 items or mixes separation keys without an agreement. [F-CLR-23] [F-CLR-49] [F-CLR-50]
- **SC-005**: A staff member can find why a given item was not eligible in under 1 minute. [F-CLR-2]
- **SC-006**: A rule version loaded in advance applies from its date with no action on that date, and no earlier decision changes. [F-CLR-65]

## Out of Scope

- Detecting possible payment splitting automatically. [F-CLR-80]
- Producing the ANSI X9.100-187-2008 image file under Standard 015. [F-CLR-48] [F-CLR-63]
- Sending, acknowledgements, settlement entries, end-of-day notices, returns, archiving and
  encoding liability. [F-CLR-18] [F-CLR-51] to [F-CLR-59]
- Stale-date, account status, funds hold, fraud and duplicate-capture checks. [F-CLR-30]
- Meeting the 2026-12-31 requirement to exchange image files with all Direct Clearers.
  [F-CLR-47] [F-CLR-79]
- The existing cheque deposit prototype, which is a design reference only. [F-CLR-38]

Capabilities left out may be added later. A closed image file group is the hand-off point
for producing files. [F-CLR-79]

## Assumptions

To confirm with the people named. None of these is established fact.

- Other systems submit items to this system through a programming interface (a constraint
  given by the user). The data source and format are unknown. (item capture system owner)
- The exchange method is supplied with each item. It may be changed while the item is in no
  group, which causes a new check. (clearing operations)
- Receiver, exchange point, processing site, combination agreements, member list and
  dispute status are supplied as reference data. (clearing operations)
- Whether an item is payable on demand, identified as required (Rule A3, not yet read) and
  meets value conditions is supplied with the item. (item capture system owner)
- The holiday calendar, cut-off times and daily volumes are configurable. No volume or
  response time target is set; one is needed before sizing. (clearing operations)
- Which A1 s.5 and s.13a exceptions the bank handles is unknown; all listed ones are
  supported. (clearing operations)
- Applying the amount limit to images is a working position. (compliance)
- The paper pre-authorized debit end date is a working position, because the rule's
  amendment history appears to say otherwise. (compliance)
- Whether unqualified items of CAD 50,000 or more go with encoded items is a bank setting,
  off by default. (clearing operations)
- A clearing log covers the bundles for one receiver at one cut-off. (clearing operations)
- A rule version cannot be given an effective date in the past, and the version in force on
  the day of a check is the one used.
- How long items and decisions are kept is not yet decided. (compliance)
- Only synthetic data is used in development and testing.

## Traceability

| Requirement | Source | Test |
|-------------|--------|------|
| FR-001 | F-CLR-67, F-CLR-72 | Edge Cases, duplicate identity; test that an item cannot take a second exchange method once grouped |
| FR-002 | F-CLR-2, F-CLR-41, F-CLR-68 | User Story 1, scenarios 1, 2, 5, 6 |
| FR-003 | F-CLR-3, F-CLR-65 | User Story 1, scenario 1 |
| FR-004 | F-CLR-4, F-CLR-5 | User Story 1, scenario 1 |
| FR-005 | F-CLR-6 | User Story 1, scenario 12 |
| FR-006 | F-CLR-8, F-CLR-69 | User Story 1, scenario 2 |
| FR-007 | F-CLR-9, F-CLR-69 | User Story 1, scenario 4 |
| FR-008 | F-CLR-69 | User Story 1, scenario 3 |
| FR-009 | F-CLR-7, F-CLR-71 | User Story 1, scenario 5 |
| FR-010 | F-CLR-10, F-CLR-71 | User Story 1, scenario 6 |
| FR-011 | F-CLR-11, F-CLR-71 | User Story 1, scenarios 7 and 8 |
| FR-012 | F-CLR-12, F-CLR-34, F-CLR-71, F-CLR-80 | User Story 1, scenario 9 |
| FR-013 | F-CLR-70 | User Story 1, scenario 10 |
| FR-014 | F-CLR-42, F-CLR-61, F-CLR-62 | Test of each User Story 1 check with image as the method |
| FR-015 | F-CLR-43 | User Story 1, scenario 11 |
| FR-016 | F-CLR-32, F-CLR-33, F-CLR-34 | User Story 1, scenarios 5, 6, 7 |
| FR-017 | F-CLR-64, F-CLR-65 | User Story 2, scenarios 1 and 2 |
| FR-018 | F-CLR-65 | User Story 2, scenario 3 |
| FR-019 | F-CLR-28 | User Story 2, scenario 4 |
| FR-020 | F-CLR-1, F-CLR-14, F-CLR-15, F-CLR-16, F-CLR-19, F-CLR-26 | User Story 3, scenario 2 |
| FR-021 | F-CLR-14 | User Story 3, scenario 3 |
| FR-022 | F-CLR-17 | Test that a rejected non-amount field is recorded and left unchanged |
| FR-023 | F-CLR-20, F-CLR-22 | User Story 3, scenarios 3 and 4 |
| FR-024 | F-CLR-21 | Test of an unqualified item at 49,999.99 and 50,000.00 with the setting on |
| FR-025 | F-CLR-23, F-CLR-74 | User Story 3, scenario 1 |
| FR-026 | F-CLR-24 | User Story 3, scenario 5 |
| FR-027 | F-CLR-44, F-CLR-45, F-CLR-46 | User Story 4, scenario 4 |
| FR-028 | F-CLR-50, F-CLR-74 | User Story 4, scenarios 2 and 3 |
| FR-029 | F-CLR-49 | User Story 4, scenario 1 |
| FR-030 | F-CLR-72 | User Story 3, scenario 2; test of adding a grouped item again |
| FR-031 | F-CLR-73, F-CLR-75 | User Story 3, scenario 1; User Story 4, scenario 1 |
| FR-032 | F-CLR-77, F-CLR-82 | User Story 5, scenarios 3 and 9 |
| FR-033 | F-CLR-76, F-CLR-77 | User Story 5, scenarios 4 and 5 |
| FR-034 | F-CLR-3, F-CLR-7, F-CLR-77, F-CLR-83 | Edge Cases, dispute on a grouped item; User Story 5, scenario 10 |
| FR-035 | F-CLR-2, F-CLR-3 | User Story 5, scenarios 1 and 2 |
| FR-036 | F-CLR-70, F-CLR-71, F-CLR-81 | User Story 5, scenarios 6 and 7 |
| FR-037 | F-CLR-76, F-CLR-82 | User Story 5, scenario 8; refusal test for each role and the sending system |
| FR-038 | F-CLR-78 | Review of screen and list wording |
| FR-039 | F-CLR-31 | Review of every explanation text |
| SC-001 | F-CLR-3, F-CLR-65 | User Story 1, scenario 1, across the synthetic set |
| SC-002 | F-CLR-68 | Synthetic eligibility test set |
| SC-003 | F-CLR-70, F-CLR-72 | Check of closed group contents against decisions |
| SC-004 | F-CLR-23, F-CLR-49, F-CLR-50 | User Story 3, scenario 1; User Story 4, scenarios 1 and 2 |
| SC-005 | F-CLR-2 | Timed walkthrough of User Story 5, scenario 1 |
| SC-006 | F-CLR-65 | User Story 2, scenarios 2 and 3 |
