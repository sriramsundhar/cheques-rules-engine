# Concepts: cheques-rules-engine

**Project**: cheques-rules-engine | **Topic**: clearing-eligibility (CLR) | **Track**: ideation | **Updated**: 2026-11-25
**Written by**: `/akka:discover <topic> concepts`

## Glossary *(required)*

One row per term the people involved use.

| Term | Definition | Words to avoid | Finding |
|------|------------|----------------|---------|
| Item | A cheque or other payment item, whether it travels as paper or as an image | cheque image as a separate thing | [F-CLR-67] |
| Eligibility decision | The result of checking whether an item may be exchanged | approval, deposit decision | [F-CLR-68] |
| Exchange group | A paper bundle or an image file group prepared for one receiver | batch | [F-CLR-73] |
| Paper bundle | At most 300 paper items of one stream, kept in order | file | [F-CLR-74] |
| Image file group | The items that belong in one image file | ICP file, which is the ANSI file this version does not produce | [F-CLR-63] |
| Sorting stream | One of the three classes paper items are separated into: encoded, unqualified or remittances | stream on its own | [F-CLR-78] |
| Settlement stream | The identifier O or S on the entry that settles an image file | stream on its own | [F-CLR-78] |
| Member | An institution that belongs to Payments Canada | none | [F-CLR-4] |
| Direct Clearer | A Member that exchanges items directly with others, as sender (delivering) or receiver | bank, when the role matters | [F-CLR-1] |
| Drawee | The institution a cheque is drawn on | payer bank | [F-CLR-4] |
| MICR line | The magnetic ink line at the bottom of a cheque | barcode | [F-CLR-15] |

## Concepts *(required)*

### C-CLR-1: Item

- **What it is**: a payment item the bank may send to another institution
- **Identified by**: one identity given at capture and kept in both paper and image form
- **States**: captured, assessed with an outcome, prepared, exchanged
- **Related to**: has a history of eligibility decisions (C-CLR-2) and belongs to at most one exchange group (C-CLR-3)
- **Rules**: F-CLR-67, F-CLR-72

```visual
kind: state
title: Life of an item
state: Captured
state: Eligible | focus
state: Prepared
state: Exchanged | end
move: Captured -> Eligible | checked
move: Eligible -> Prepared | grouped
move: Prepared -> Exchanged | sent
```

- [F-CLR-67] A cheque and its image are one item with one identity and one decision history, and paper or image is its exchange method | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-9 | status: confirmed
- [F-CLR-72] An eligible item moves on to prepared when it is placed in a group and then to exchanged | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-9 | status: confirmed

### C-CLR-2: Eligibility decision

- **What it is**: the outcome of checking one item against the rules, with the reasons
- **Identified by**: the item, the time of the check and the rule version used
- **States**: none; a new check makes a new decision, and the outcomes are eligible, waiting, needs review and not eligible
- **Related to**: belongs to one item (C-CLR-1) and is made by one rule version (C-CLR-4)
- **Rules**: F-CLR-68, F-CLR-69, F-CLR-70, F-CLR-71, F-CLR-3

```visual
kind: relationship
title: What a decision records
focus: Decision
link: Item | about
link: Rule version | made by
link: Rule section | names
```

- [F-CLR-68] An eligibility check ends as eligible, waiting, needs review or not eligible | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-7 | status: confirmed
- [F-CLR-69] A post-dated or month-dated item waits and is checked again on the date it can qualify | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-7 | status: confirmed
- [F-CLR-70] An item with missing or unclear data goes to review and is never eligible by default | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-7 | status: confirmed
- [F-CLR-71] A protest direction or suspected splitting goes to review, while a dispute or an amount over the limit with no exception makes an item not eligible | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-7 | status: confirmed

### C-CLR-3: Exchange group

- **What it is**: a paper bundle or image file group prepared for sending
- **Identified by**: its receiver, its method, its separation keys and its sequence
- **States**: open, closed, exchanged
- **Related to**: holds eligible items (C-CLR-1)
- **Rules**: F-CLR-73, F-CLR-74, F-CLR-75, F-CLR-76, F-CLR-77, F-CLR-23, F-CLR-49, F-CLR-50

```visual
kind: state
title: Life of an exchange group
state: Open | focus
state: Closed
state: Exchanged | end
move: Open -> Closed | closes
move: Closed -> Exchanged | reported
move: Closed -> Open | reopened
```

- [F-CLR-73] An exchange group is a paper bundle or an image file group and moves from open to closed to exchanged | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-15 | status: confirmed
- [F-CLR-74] A paper bundle holds one sorting stream for one receiving Direct Clearer, and image file groups are separated as Rule A10 section 24 requires | evidence: stated: Sriram, 2026-11-24; S-CLR-1 s.19, s.20a; S-1 s.24e; understanding.md U-CLR-11 | status: confirmed
- [F-CLR-75] A group closes when full or at a cut-off time operations set, whichever comes first, and its contents are then fixed and its list and totals produced | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-15 | status: confirmed
- [F-CLR-76] The system learns that a group was exchanged from a person or from the sending system | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-15 | status: confirmed
- [F-CLR-77] An authorized person can reopen a closed group before it is exchanged, with the change and reason recorded, and an exchanged group never changes | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-15 | status: confirmed

### C-CLR-4: Rule version

- **What it is**: the set of checks in force from a given date
- **Identified by**: a version number and the date it takes effect
- **States**: loaded, active
- **Related to**: makes eligibility decisions (C-CLR-2)
- **Rules**: F-CLR-65, F-CLR-37

```visual
kind: state
title: Life of a rule version
state: Loaded
state: Active | focus
move: Loaded -> Active | on date
```

## Conflicting Usage *(required, or none)*

- [F-CLR-78] Stream means a sorting class of paper items in Rule A1 and an identifier on a settlement entry in Rule A10 | evidence: S-CLR-1 s.19; S-1 s.30; understanding.md U-CLR-19 | status: confirmed

## Shared with the Project *(required, or none)*

none: this is the only topic, and no concept was proposed for sharing

## Conflicts with Other Topics *(required, or none)*

none: no other topic exists

## Unknowns *(required, or none)*

- What are the cut-off times that close a group? | ask: clearing operations
- Who is authorized to reopen a closed group? | ask: clearing operations
- Does the bank hold a post-dated item until its date, or send it back to the customer? | ask: clearing operations
