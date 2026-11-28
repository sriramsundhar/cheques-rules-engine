# Research: Cheque Clearing Eligibility and Exchange Preparation

**Branch**: `001-clearing-exchange-prep` | **Date**: 2026-11-28 | **Spec**: [spec.md](spec.md)

Sources: the spec, `discovery/clearing-eligibility/`, and the Akka documentation refreshed on
2026-11-28 (`akka-context/sdk/`). The project uses Akka SDK 3.7.3 and targets Java 21.

## R1. Where the contents of an exchange group live

**Decision**: One Event Sourced Entity per group holds the group state and its ordered
entries. An entry is the item id and the amount in cents. Item ids are at most 36
characters. Reads of the contents are paged, 500 entries at a time. This is proven by
measurement in delivery Phase 1 before anything is built on it.

**Rationale**: Adding an item and closing the group are handled by one single writer. The
spec requires that an item is either in the closed list or in a new group, never half of
each, and this gives that by construction.

**Size estimate** (not measured): 40,000 entries at 36 to 60 bytes is about 1.5 to 2.4 MB of
state. The documented hard limit for entity state is 10 MB. State or events over 1 MB do not
replicate across regions, and the documented target for requests is under 500 KB. Events stay
small and replies are paged, but the snapshot is large.

**Consequence**: The service is planned for one region. `snapshot-every` is raised from 100
after measurement, so that a full group does not write a large snapshot 400 times.

**Alternatives considered**:

- Group header entity plus segment entities of 1,000 entries. Each state stays under 100 KB.
  Not chosen now: closing becomes a process across up to 41 entities, with its own race
  between sealing and adding. This is the fallback if Phase 1 fails or multi-region is needed.
- Contents only in a View. Rejected: a View is eventually consistent, and the closed list
  must be exact.
- A smaller group limit, such as 5,000. Rejected: the spec expects groups to fill to 40,000.

## R2. How an eligible item joins a group

**Decision**: A Consumer starts one `PlacementWorkflow` per placeable item. The workflow reads
the open group from `ExchangeLaneEntity`, records the assignment on the item, adds the item to
the group, and confirms it on the item. If the group no longer takes items, the workflow
releases the assignment, rolls the lane over and tries again.

**Rationale**: The item is the single authority for which group it is in, so an item cannot be
in two groups (FR-030). Every step is safe to repeat. A failed step must be undone, which the
component guide gives as the reason to use a Workflow.

**Alternatives considered**:

- A Consumer that makes the three calls itself. Rejected: there is an assignment to undo, and
  nobody could see where a stuck placement stopped.
- The group entity choosing the item's group. Rejected: it cannot see assignments in other
  groups.

## R3. Closing a group

**Decision**: Closing has two steps. `beginClose` stops new items. `GroupCloseWorkflow` then
checks every entry again, in pages of 200, removes any that are no longer eligible, and calls
`completeClose`. A group with no entries left is discarded and gets no list.

**Rationale**: SC-003 says no waiting, review or not eligible item is in a group when it
closes. Removal by events alone can lose a race with a cut-off. The check at close also covers
conditions that change with the date, such as the pre-authorized debit end date.

**Risk**: A full image group needs 40,000 item calls to close. Phase 1 measures it.

**Alternatives considered**: Trusting the removal events (leaves a window). Checking only
items with a recent change (needs a second record of what changed).

## R4. Where eligibility is decided

**Decision**: The checks are plain Java in `domain`: item, context and date go in, a decision
comes out. The item entity runs them inside its command handler. The caller loads the context
(rule version, member list, holidays) and passes it with the command. The submit call returns
the decision in its reply.

**Rationale**: The checks are testable without the runtime. The decision and the item state
change together, so a concurrent change cannot produce a decision on old item data.

**Alternatives considered**: A rules component of its own (it has no state to keep). A Workflow
per submission (one step, which the guide rules out). A Consumer that decides after submission
(the submitter would not get the decision).

**Precedence when several reasons apply**: not eligible, then needs review, then waiting, then
eligible. All reasons are recorded.

## R5. Rule versions

**Decision**: One Event Sourced Entity, `RuleBookEntity`, holds the loaded versions. A version
has an effective date and the settings: amount limit, pre-authorized debit end date,
acceptable kinds, kinds barred from images, image resolution range, bundle and file group
limits, and the unqualified threshold. A baseline version is built into the service
configuration and applies when no loaded version is in force.

**Rationale**: The user agreed that checks are code and that dates and limits are versioned
settings (U-CLR-16). Who loaded which version matters later, so events are kept.

**Alternatives considered**: A Key Value Entity (loses the record of who loaded what). One
entity per version with a View to find the one in force (a decision must not depend on an
eventually consistent read). A rules language that operations edit (not asked for).

## R6. Checking a waiting item again

**Decision**: A Consumer schedules one Timed Action per waiting item for the start of its
qualifying day. The timer carries only the item id. If the item is still waiting when it fires,
the action schedules again.

**Rationale**: The guide gives "run something once later" to a Timed Action.

**Alternatives considered**: A daily sweep of a View (depends on the View being current). A
Workflow timer (the process would be only "wait, then call").

## R7. Cut-off

**Decision**: `CutOffTimedAction` runs at each cut-off time. It asks a View for open groups
created before that time and calls `beginClose` on each. It runs one more pass 30 seconds later
to catch groups the View had not shown yet, then schedules the next cut-off. A group that fills
before a cut-off belongs to the next cut-off's exchange cycle.

**Rationale**: A schedule is a Timed Action. The second pass covers View delay.

**Alternatives considered**: A registry entity of open groups (a second record of the same
fact, and a hot spot).

## R8. New information that touches many items

**Decision**: `RevalidationWorkflow` pages through unexchanged items and checks each again. It
starts when the member list changes (limited to the affected drawees) and when a loaded rule
version comes into force.

**Rationale**: The work is long, repeatable and worth a visible status.

**Alternatives considered**: Doing it in the endpoint request (too long). A Consumer on the
member list (a Key Value Entity change does not say which member went).

## R9. Access control

**Decision**: Endpoints validate a bearer token (`@JWT`) and read a `role` claim. The roles are
`operator`, `supervisor`, `rule-administrator` and `sending-system`, from FR-037. The caller's
identity comes from the `sub` claim and is recorded on every manual action.

**Rationale**: Akka validates tokens at the endpoint. No identity code is written here.

**Open**: The spec names no role for the system that submits items, preparation steps, image
results and disputes, or for who loads reference data. The plan proposes `capture-system` for
the first and `rule-administrator` for the second. Both are in the Decisions Log as open. The
bank's identity provider is unknown.

**Local development**: Akka accepts unsigned tokens in dev mode and tests. Deployed services
must have signing keys configured.

## R10. Time

**Decision**: One configured business time zone, default `America/Toronto`. Application code
gets the current time from an injected `java.time.Clock`. Domain code takes dates as
parameters.

**Rationale**: Post-dated, month-dated and end-date rules need exact day boundaries, and tests
must set the date.

## R11. Lists and clearing logs

**Decision**: A group's list and totals come from the closed group entity. The clearing log is
computed on read: the closed paper bundles for one receiver and one exchange cycle.

**Rationale**: Nothing needs to be stored that cannot be derived. After exchange the bundles
cannot change, so the log cannot change.

**Alternatives considered**: A clearing log entity (a copy that must follow every reopen).

## R12. HTTP or gRPC for other systems

**Decision**: HTTP with JSON for every caller.

**Rationale**: The operations screen needs HTTP. The capture system and its format are unknown.
One API is simpler.

**Alternatives considered**: gRPC for the capture and sending systems, which the guide prefers
for non-browser callers. Revisit when the capture system is known.

## R13. Operations screen

**Decision**: Static HTML, CSS and JavaScript served by the service from
`static-resources/`, built from the approved mock M-CLR-1. No front-end framework and no build
step. The screen reloads data after each action.

**Rationale**: Constitution principle IV. The mock is already plain HTML and JavaScript.

## R14. Duplicates

**Decision**: The item id is the entity id. Submitting the same id with the same data returns
the existing decision. The same id with different data is refused with a conflict.

## R15. Volume

**Decision**: No target is set. Design and tests assume 100,000 items a day and one image group
of 40,000. These are assumptions for sizing, not figures from the bank.

## Not decided here

- Retention of items and decisions. Nothing is deleted until compliance decides.
- Real cut-off times, holidays, members and routing. They are loaded as reference data.
