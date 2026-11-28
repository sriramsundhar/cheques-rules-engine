# Implementation Plan: Cheque Clearing Eligibility and Exchange Preparation

**Branch**: `001-clearing-exchange-prep` | **Date**: 2026-11-28 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-clearing-exchange-prep/spec.md`
**Discovery**: `discovery/clearing-eligibility/understanding.md`, `discovery/clearing-eligibility/mocks/index.md` (there is no `behaviour.md` or `system-map.md`)

**Note**: This file is filled in by the `/akka:plan` command.

## Summary

One Akka service decides whether each Canadian-dollar item may be exchanged under Rule A1,
records why, and prepares eligible items as paper bundles or image file groups under Rules A1
and A10. Operations staff work the results on a screen.

The checks are plain Java with dates and limits held as date-effective rule versions. Each item
is an Event Sourced Entity with its decision history. Each exchange group is an Event Sourced
Entity that owns its ordered contents. Workflows place items, close groups and mark items
exchanged. See [research.md](research.md).

The largest risk is that one entity holds the contents of a 40,000-item image group. It is
measured first.

## Technical Context

**Language/Version**: Java 21 target (JDK 25 installed)
**Primary Dependencies**: Akka SDK 3.7.3 (`akka-javasdk-parent`). No other runtime dependency.
**Storage**: Akka entity journals and views. No external database.
**Testing**: JUnit 5, Akka TestKit (`EventSourcedTestKit`, `KeyValueEntityTestKit`, `TestKitSupport`)
**Target Platform**: Akka service, one region
**Project Type**: web service with a small served screen
**Performance Goals**: none set by the user. A response time or volume target is needed before sizing.
**Constraints**: entity state under 10 MB; entity requests and replies under 1 MB; synthetic data only; no rule text reproduced
**Scale/Scope**: assumed for design and tests, not given by the bank: 100,000 items a day, image groups up to 40,000, bundles up to 300

## Constitution Check

*GATE: checked before research and again after design, against constitution 1.1.0.*

| Principle | Before research | After design |
|-----------|-----------------|--------------|
| I. Akka SDK first | Pass | Pass. Entities, views, workflows, consumers, timed actions and endpoints only. No added dependency. |
| II. Domain independence | Pass | Pass. Checks, grouping keys and calendar logic are in `domain`. |
| II. API isolation | Pass | Pass. Endpoints own their request and response records. |
| II. Right component | Pass | Pass. See Component Architecture, with rejected alternatives. |
| II. Single responsibility and naming | Pass | Pass. |
| III. Test coverage | Pass | Pass. Each delivery phase has a test that can fail. |
| IV. Simplicity | Pass | Pass, with one watch item below. |

No violation needs justification. One watch item: the group entity state can pass the 1 MB
cross-region replication limit and the 500 KB performance target. It is a platform sizing
risk, not a constitution violation. It is a Decisions Log entry and delivery Phase 1.

## Project Structure

### Documentation (this feature)

```text
specs/001-clearing-exchange-prep/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── http-api.md
└── tasks.md             # created by /akka:tasks
```

### Source Code (repository root)

```text
src/
├── main/
│   ├── java/com/example/
│   │   ├── api/             # HTTP endpoints and their records
│   │   ├── application/     # entities, views, workflows, consumers, timed actions, Bootstrap
│   │   └── domain/          # item, eligibility, rule version, reference data, grouping
│   └── resources/
│       ├── application.conf # time zone, baseline rule version, bank settings
│       └── static-resources/  # operations screen
└── test/
    ├── java/com/example/    # unit and integration tests
    └── resources/           # synthetic eligibility test set
```

**Structure Decision**: One Akka service in the existing project, using the existing
`com.example` packages `api`, `application` and `domain`.

## Component Architecture

| Domain concept / process | Component | Why this component | Rejected alternative and why not (close-call decisions) |
|--------------------------|-----------|--------------------|----------------------------------|
| Item and its decision history | `ItemEntity` (Event Sourced Entity) | Every decision and manual action must be explained later | Key Value Entity: it would need a hand-kept history |
| Eligibility checks | plain classes in `domain` | Pure calculation | A component: there is no state, schedule or API of its own |
| Rule versions | `RuleBookEntity` (Event Sourced Entity, one instance) | Who loaded which version matters | Key Value Entity: no record. Entity per version with a View: a decision must not depend on a delayed read |
| Member list | `MemberDirectoryEntity` (Key Value Entity) | Only the current list matters | Event Sourced Entity: nobody consumes changes or needs history |
| Routing and combination agreements | `ExchangeRoutingEntity` (Key Value Entity) | Current value only | Event Sourced Entity: same reason |
| Holidays and cut-off times | `BusinessCalendarEntity` (Key Value Entity) | Current value only | Configuration file: operations must change it without a deployment |
| Which group is open for a key | `ExchangeLaneEntity` (Key Value Entity) | One writer per key, current value only | View of open groups: delayed, so two groups could be opened |
| Exchange group and its contents | `ExchangeGroupEntity` (Event Sourced Entity) | State machine, audit of reopen and exchange, exact ordered contents | Segment entities: more coordination, kept as the fallback. View: not exact at close |
| Start placing an item | `ItemPlacementConsumer` | Reacts to a recorded fact | Direct call from the endpoint: misses items that become eligible later |
| Withdraw an item that stopped being eligible | `ItemWithdrawalConsumer` | Reaction to a recorded fact | Same consumer as placement: two separate reactions |
| Place an item in a group | `PlacementWorkflow` | Several steps, with an assignment to undo | Consumer calls: no place to undo or see a stuck placement |
| React to group events | `ExchangeGroupConsumer` | Starts close and exchange processes, releases removed items | Calls inside the entity: entities must not call out |
| Close a group | `GroupCloseWorkflow` | Long, paged, must finish | Consumer: no durable progress |
| Mark items exchanged | `GroupExchangeWorkflow` | Up to 40,000 items, paged | Consumer: one handler would run too long |
| Recheck many items after new information | `RevalidationWorkflow` | Long, paged, has a status | Endpoint request: too long |
| Schedule rechecks and rule effective dates | `RecheckScheduleConsumer` | Turns recorded facts into timers | Scheduling in the endpoint: misses other paths |
| Recheck a waiting item; start revalidation on a version date | `ItemRecheckTimedAction`, `RuleVersionTimedAction` | Run once later | Workflow timer: the only content would be waiting. Daily View sweep: depends on a current View |
| Cut-off | `CutOffTimedAction` | Runs on a schedule | Workflow: it is one call per group |
| Items by outcome, drawee and not exchanged | `ItemsView` | Lists across items | Entity reads: only by id |
| Groups by status, age, receiver and cycle | `ExchangeGroupsView` | Lists across groups | Entity reads: only by id |
| Clearing log | computed in the endpoint from `ExchangeGroupsView` and group entities | Derivable | Entity: a copy that must follow every reopen |
| Programming interface and screen data | `ItemEndpoint`, `ExchangeGroupEndpoint`, `RuleVersionEndpoint`, `ReferenceDataEndpoint` (HTTP) | Browser and system callers | gRPC for systems: capture system unknown, one API is simpler |
| Operations screen | `UiEndpoint` (HTTP, static files) | Serves the screen | Separate front-end hosting: nothing needs it |

### How the parts work together

1. `POST /items` loads the rule version, member list and holidays, then sends one command to
   `ItemEntity`. The reply carries the decision.
2. A waiting decision sets a timer. An eligible and prepared item starts a `PlacementWorkflow`.
3. The group closes when full, at a cut-off, or when a supervisor closes it again.
   `GroupCloseWorkflow` checks each entry again before the contents are fixed.
4. A later dispute or correction records a new decision. The item is withdrawn from an open
   group, or its closed group is flagged.
5. Exchange is recorded on the group, then on each item.

## Complexity Tracking

No constitution violation to justify.

## Delivery Phases

Critical path: Phase 1, Phase 2, Phase 4, Phase 6, Phase 7. Phase 3 and Phase 5 can run beside
their neighbours.

### Phase 1: Group capacity proof

- **Goal**: Show that one group entity can hold, close and page 40,000 items, or switch to segments.
- **Steps**: group domain type and `ExchangeGroupEntity` with add, close and paged read; a load test of 40,000 adds; measure snapshot size, add latency, reload time and a 40,000-item verification pass; choose `snapshot-every`.
- **Parallel with**: none
- **Acceptance test**: `ExchangeGroupCapacityIntegrationTest` adds 40,000 items, the 40,001st is refused, paged reads return all in order, and serialized state is under 5 MB. Times are printed and recorded in research.md.
- **Gate**: group size review (see Review Gates)
- **Risk**: state or latency is too high, which changes the group design.
- **Rollback**: delete the group entity; nothing else depends on it yet.
- **Size**: S

### Phase 2: Eligibility decision (User Story 1)

- **Goal**: Submit an item and read back outcome, reasons, sections and rule version.
- **Steps**: domain checks and reason texts; `ItemEntity`; reference data entities and endpoint; `ItemEndpoint` submit and read with role checks; `ItemsView`; synthetic test set.
- **Parallel with**: none
- **Acceptance test**: `EligibilityRulesTest` and the synthetic set give the expected outcome for every item (SC-002); `ItemEndpointIntegrationTest` covers User Story 1 scenarios.
- **Gate**: data model review before the first entity events are written
- **Risk**: a rule is read wrongly. Compliance has not reviewed the readings.
- **Rollback**: revert the phase commits; Phase 1 stays.
- **Size**: L

### Phase 3: Rule versions and waiting items (User Story 2)

- **Goal**: Load a future version; waiting items are checked again on their day.
- **Steps**: `RuleBookEntity` and endpoint; injected clock; recheck timers; `RevalidationWorkflow`.
- **Parallel with**: Phase 4
- **Acceptance test**: `RuleVersionIntegrationTest` shows the limit changing on the date and the old decision unchanged (SC-006); `WaitingItemRecheckIntegrationTest` moves the clock and sees a new decision; the pre-authorized debit passes on 2028-11-30 and fails on 2028-12-01.
- **Gate**: none
- **Risk**: day boundaries and time zone errors.
- **Rollback**: the baseline version stays in force; remove the version routes and timers.
- **Size**: M

### Phase 4: Paper bundles (User Story 3)

- **Goal**: Prepared eligible paper items appear in bundles with lists and a clearing log.
- **Steps**: preparation steps and sorting streams; lanes; `PlacementWorkflow`; withdrawal; `GroupCloseWorkflow`; cut-off timer; `ExchangeGroupsView`; clearing log.
- **Parallel with**: Phase 3
- **Acceptance test**: `PaperBundleIntegrationTest` gets bundles of 300, 300 and 50 in order, holds out an unprepared item, and discards an empty group; a test that disputes an item during close shows it is not in the closed list (SC-003).
- **Gate**: none
- **Risk**: races between adding, disputing and closing.
- **Rollback**: stop the consumers and timers; decisions keep working.
- **Size**: L

### Phase 5: Image file groups (User Story 4)

- **Goal**: Usable image items are grouped by the A10 keys, up to 40,000.
- **Steps**: image quality; image lane keys and combination agreements; reuse placement and close.
- **Parallel with**: Phase 6
- **Acceptance test**: `ImageFileGroupIntegrationTest` covers User Story 4 scenarios 2 to 4; the 90,000-item split of scenario 1 is checked with a lowered limit in tests and once at full size by hand.
- **Gate**: none
- **Risk**: close time for a full group is long.
- **Rollback**: refuse image placement; paper is unaffected.
- **Size**: M

### Phase 6: Group lifecycle, reviews and roles (User Story 5, without the screen)

- **Goal**: Resolve reviews, reopen, close again and mark exchanged, with every permission enforced.
- **Steps**: corrections and flag confirmations; reopen and removal; `GroupExchangeWorkflow`; flagged exchange warning; role tests.
- **Parallel with**: Phase 5
- **Acceptance test**: `GroupLifecycleIntegrationTest` covers User Story 5 scenarios 3 to 10; `RolePermissionIntegrationTest` tries every route with every role and gets the result in the contract.
- **Gate**: permissions review for the proposed `capture-system` role and reference data access
- **Risk**: a permission is too wide.
- **Rollback**: remove the manual action routes; grouping keeps working.
- **Size**: M

### Phase 7: Operations screen (User Story 5)

- **Goal**: Staff do the work of User Story 5 in a browser.
- **Steps**: build the screen from mock M-CLR-1 version 1; connect it to the API; development sign-in only.
- **Parallel with**: none
- **Acceptance test**: a browser walkthrough of User Story 5 scenarios 1 to 5 against a local service, with screenshots; finding why an item was not eligible takes under 1 minute (SC-005).
- **Gate**: mock M-CLR-1 (approved by Sriram on 2026-11-25)
- **Risk**: sign-in for real users depends on an unknown identity provider.
- **Rollback**: remove the static files and `UiEndpoint`; the API stays.
- **Size**: M

## Review Gates

| Before | Artifact to see | Reviewer | State |
|--------|-----------------|----------|-------|
| Phase 2 | Phase 1 measurements and the choice of group design | architect | open |
| Phase 2 | [data-model.md](data-model.md), with the check table | product owner and compliance | open |
| Phase 6 | role table in [contracts/http-api.md](contracts/http-api.md) | security and clearing operations | open |
| Phase 7 | mock M-CLR-1 version 1, `discovery/clearing-eligibility/mocks/operations-screen.html` | product owner | approved 2026-11-25 |

## Baseline

none: the spec is not a modernization, `discovery/clearing-eligibility/` has no `system-map.md` or `behaviour.md`, no finding measures today's process, and there is no `.akka/discovery/timing.json`.

## Metrics and Targets

| Target | Serves | How and when measured |
|--------|--------|-----------------------|
| 100% of decisions show outcome, sections, time and version | SC-001 | assertion over the synthetic set, Phase 2 |
| 100% expected outcomes on the synthetic set | SC-002 | `SyntheticEligibilityTest`, Phase 2 and every later phase |
| No ineligible item in a closed group | SC-003 | contents checked against decisions, Phase 4 and 5 |
| Limits and keys never broken | SC-004 | group tests, Phase 4 and 5 |
| Find why an item was not eligible in under 1 minute | SC-005 | timed walkthrough, Phase 7 |
| A loaded version applies on its date with no action | SC-006 | `RuleVersionIntegrationTest`, Phase 3 |
| No target yet: submit latency, add latency, full group close time | no requirement | measured in Phase 1, reported to clearing operations |

## Model Plan

| Phase | Purpose | Model | Effort | Cost | Speed | Accuracy | Why |
|-------|---------|-------|--------|------|-------|----------|-----|
| Phase 1 | design | Opus 5.5 | high | highest | lowest | highest | A wrong group design spreads to later phases |
| Phase 1 | implementation | Sonnet 5.5 | medium | medium | medium | medium | Follows a fixed design |
| Phase 2 | design | Opus 5.5 | high | highest | lowest | highest | Rule readings and event shapes are hard to change |
| Phase 2 | implementation | Sonnet 5.5 | medium | medium | medium | medium | Fixed check table |
| Phase 2 | review | Opus 5.5 | high | highest | lowest | highest | A missed rule sends a barred item |
| Phase 3 | implementation | Sonnet 5.5 | medium | medium | medium | medium | Known timer and entity patterns |
| Phase 4 | implementation | Sonnet 5.5 | high | medium | medium | medium | Concurrency needs care |
| Phase 4 | review | Opus 5.5 | high | highest | lowest | highest | Races are easy to miss |
| Phase 5 | implementation | Sonnet 5.5 | medium | medium | medium | medium | Reuses Phase 4 |
| Phase 6 | implementation | Sonnet 5.5 | medium | medium | medium | medium | Follows the contract |
| Phase 6 | review | Opus 5.5 | high | highest | lowest | highest | Permission errors matter |
| Phase 7 | implementation | Sonnet 5.5 | medium | medium | medium | medium | Follows the approved mock |
| Phase 7 | visual checks | Haiku 5.5 | low | lowest | highest | lowest | Many small checkable screenshots |

Downgrade rule: at 85% of the plan allowance every sub-agent moves one tier down. The AI does
not change the main session. Exceptions: the Phase 2 and Phase 4 reviews stay on the highest
tier.

Rough relative estimate: Phase 2 and Phase 4 are the largest, Phase 1 the smallest, the others
in between. No prices were given.

## Decisions Log

No `understanding.md` Decisions Raised item is open (the section says none), and there is no
`behaviour.md`.

| Decision | Recommendation and reason | Owner | Status |
|----------|---------------------------|-------|--------|
| Group contents in one entity or in segments | One entity, because add and close are then atomic. Confirm with Phase 1 measurements | architect | open |
| Must the service run in more than one region | Decide before Phase 1 ends. One region as planned; more than one means segments | architect | open |
| Identity for the system that submits items, results and disputes | A separate `capture-system` role, because FR-037 does not cover it | security | open |
| Who loads members, routing and calendar | Rule administrators, who already control what decides outcomes | clearing operations | open |
| Identity provider and how staff sign in | Use the bank's token issuer; the development sign-in is not for real users | security | open |
| Outcome when several reasons apply | Not eligible, then needs review, then waiting. The strictest result wins | compliance | open |
| Check every item again at close | Yes, because it is the only way to hold SC-003 against races | architect | open |
| An eligible item a supervisor removes from a reopened group | It joins the next group, so it is not stranded | clearing operations | open |
| Route per drawee institution | One route per institution until real routing data is known | clearing operations | open |
| Business time zone | `America/Toronto`, configurable | clearing operations | open |
| Volume and response targets | Give both before sizing; design assumes 100,000 items a day | clearing operations | open |
| Retention of items and decisions | Keep everything until decided | compliance | open |
| Amount limit on images and the pre-authorized debit end date | Keep the spec's working positions and confirm | compliance | open |
| Item and rule history: event sourced | Yes, for audit | architect | open |
| HTTP for system callers rather than gRPC | HTTP until the capture system is known | architect | open |
