# Data Model: Cheque Clearing Eligibility and Exchange Preparation

**Spec**: [spec.md](spec.md) | **Research**: [research.md](research.md)

Domain types are Java records and enums in `com.example.domain`. They have no Akka imports,
except `@TypeName` on events.

## Item (`ItemEntity`, entity id = item id)

| Field | Type | Notes |
|-------|------|-------|
| itemId | String | 1 to 36 characters, letters, digits and `-` |
| kind | ItemKind | the kinds A1 lists, plus `OTHER` |
| amountCents | Long, optional | missing or unreadable means needs review |
| currency | String | only `CAD` is accepted |
| date | ItemDate, optional | full date, or year and month with no day |
| drawee | String, optional | institution number |
| localDrawee | boolean | drawn on a qualifying local |
| payableOnDemand, identifiedAsRequired, meetsValueConditions | Boolean, optional | supplied; missing means needs review |
| interMemberPurpose | enum, optional | `SETTLEMENT_BALANCE`, `FLOW_IMBALANCE`, `LYNX_EMERGENCY` |
| method | `PAPER` or `IMAGE` | may change only while in no group |
| flags | set | `PROTEST`, `POSSIBLE_SPLIT`, `DAMAGED`, `ATTACHMENT_NOT_REMOVABLE`, `NOT_MACHINE_READABLE`, `REPLACEMENT_NEEDED` |
| confirmedFlags | set | a confirmed `PROTEST` or `POSSIBLE_SPLIT` makes the item not eligible |
| disputed | boolean | may be set before the item is submitted |
| preparation | set | `ATTACHMENTS_REMOVED`, `FIELDS_ENCODED`, `ERRORS_CORRECTED`, `IMAGED_OR_ARRANGED` |
| imageQuality | record, optional | frontUsable, backUsable, blackAndWhiteDpi |
| decisions | list | newest last; see below |
| assignment | record, optional | groupId, confirmed |
| exchangedAt | Instant, optional | set once |

### Eligibility decision

outcome (`ELIGIBLE`, `WAITING`, `NEEDS_REVIEW`, `NOT_ELIGIBLE`), reasons (reason code, rule
section, own-words text), sectionsChecked, exceptionUsed (optional), recheckOn (date, only when
waiting), ruleVersionId, decidedAt.

A new decision is recorded when the outcome, reasons or rule version differ from the last one.
Submission and manual resolution always record one.

### Item states

The state is derived, not stored.

| State | Condition |
|-------|-----------|
| captured | submitted, no decision |
| assessed | has a decision, no confirmed assignment |
| prepared | confirmed assignment |
| exchanged | `exchangedAt` set; no further change is accepted |

### Events

`ItemSubmitted`, `DecisionRecorded`, `ItemCorrected`, `FlagRaised`, `FlagConfirmed`,
`DisputeRecorded`, `DisputeCleared`, `PreparationRecorded`, `ImageQualityRecorded`,
`ExchangeMethodChanged`, `GroupAssigned`, `PlacementConfirmed`, `GroupReleased`,
`ItemExchanged`.

Manual events carry actor, reason and time.

## Eligibility checks

Pure functions. Input: item, `EligibilityContext` (rule version settings, member list, holidays),
today. Each failing check adds one reason.

| Check | Section | Outcome when it fails |
|-------|---------|-----------------------|
| Required data present and readable | none | needs review |
| Acceptable kind, including the pre-authorized debit end date | A1 s.2, s.3 | not eligible |
| Drawn on a member or qualifying local, or a listed exception | A1 s.3, s.5 | not eligible; needs review if no member list is loaded |
| Payable on demand, identified, value conditions | A1 s.3 | not eligible |
| Not disputed | A1 s.9 | not eligible |
| Not post-dated | A1 s.10 | waiting |
| No-day date reached its first business day | A1 s.11 | waiting |
| No protest direction | A1 s.12 | needs review; not eligible once confirmed |
| Within the amount limit, or an inter-member exception | A1 s.13a | not eligible |
| Not flagged as split | A1 s.13b | needs review; not eligible once confirmed |
| Kind allowed as an image (image method only) | A10 s.8b | not eligible |

When several fail: not eligible wins, then needs review, then waiting.

## Rule version (`RuleBookEntity`, one instance)

versionId, effectiveFrom, loadedBy, loadedAt, amountLimitCents (2,500,000,000),
padEndDate (2028-12-01), acceptableKinds, imageBarredKinds, minDpi (200), maxDpi (240),
bundleMax (300), fileGroupMax (40,000), unqualifiedThresholdCents (5,000,000).

Rules: effectiveFrom is after the day of loading. No two versions share a date. The version in
force on a day is the latest effectiveFrom on or before it, or the baseline.

Event: `RuleVersionLoaded`.

## Reference data (Key Value Entities, one instance each)

| Entity | Holds |
|--------|-------|
| `MemberDirectoryEntity` | member institution numbers, listed exceptions, whether a list was ever loaded |
| `ExchangeRoutingEntity` | per drawee institution: receiver, exchange point, processing site; combination agreements |
| `BusinessCalendarEntity` | holidays, cut-off times |

An eligible item with no route is held out of grouping and shows `NO_ROUTE`.

## Exchange lane (`ExchangeLaneEntity`, id = lane key)

State: currentSequence. The open group id is `<laneKey>-<sequence>`.

Lane key:

- paper: `P`, receiver, sorting stream (`ENCODED`, `UNQUALIFIED`, `REMITTANCE`)
- image: `I`, receiver, exchange point (or the combined point), processing site, currency,
  direction

`rollOver(from)` advances only when the current sequence equals `from`.

## Exchange group (`ExchangeGroupEntity`, entity id = group id)

| Field | Notes |
|-------|-------|
| groupId, laneKey, type | `BUNDLE` or `FILE_GROUP` |
| status | `OPEN`, `CLOSING`, `CLOSED`, `EXCHANGED`, `DISCARDED` |
| reopened | a reopened group takes no new items |
| maxItems | from the rule version when the group was created |
| entries | ordered: sequence, itemId, amountCents |
| totalCents, count | kept with the entries |
| flags | itemId and reason, for closed groups |
| exchangeCycle | cut-off it belongs to |
| closeNumber, openedAt, closedAt, exchangedAt, deliveringClearer | |
| history | reopen, removal, close and exchange actions with actor, reason and time |

### Transitions

| From | Command | To |
|------|---------|----|
| (none) | `add` | OPEN |
| OPEN | `add` reaching maxItems | CLOSING |
| OPEN | `beginClose` (cut-off or supervisor) | CLOSING |
| CLOSING | `completeClose` | CLOSED, or DISCARDED when empty |
| CLOSED | `reopen` (supervisor, reason) | OPEN, reopened |
| CLOSED | `markExchanged` | EXCHANGED |

`withdraw(itemId)`: removes the entry when OPEN or CLOSING; flags the group when CLOSED; does
nothing when EXCHANGED. `markExchanged` on a flagged group needs an acknowledgement from a
person, and records that it was exchanged while flagged. A report from the sending system is
accepted and recorded the same way. Repeating any command is safe.

### Corrections to a grouped item

- Group closed: refused until a supervisor reopens it.
- Group open or reopened, item still eligible and same lane: the entry is amended in place.
- Otherwise: the item is withdrawn. If still eligible, it is placed again.

### Events

`GroupOpened`, `ItemAdded`, `ItemAmended`, `ItemRemoved`, `ClosingStarted`, `GroupClosed`,
`GroupDiscarded`, `GroupFlagged`, `GroupReopened`, `GroupExchanged`.

## Views

| View | Source | Queries |
|------|--------|---------|
| `ItemsView` | `ItemEntity` events | by outcome (paged); all (paged); unexchanged (paged); unexchanged by drawee |
| `ExchangeGroupsView` | `ExchangeGroupEntity` events | by status; open and created before a time; by receiver and exchange cycle |

Views hold summaries only. Decision history and group contents are read from the entities.
