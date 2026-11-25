# Shared Understanding: cheques-rules-engine

**Project**: cheques-rules-engine | **Topic**: clearing-eligibility (CLR) | **Track**: ideation | **Updated**: 2026-11-25
**Written by**: `/akka:discover <topic> confirm`

## Outcome *(required)*

- **Items**: 21 | **Agreed**: 21 | **Amended**: 0 | **Rethinking**: 0 | **Disagreed**: 0 | **Open**: 0
- **Open high-impact items**: 0
- **Last session**: 2026-11-25, with Sriram

## Items *(required)*

- [U-CLR-1] The system serves a bank sending items to other institutions as paper and as images, gives each item an eligibility decision naming the rule section, and prepares eligible items in groups | impact: high | covers: F-CLR-1, F-CLR-2, F-CLR-3, F-CLR-41 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-24
  - v1 (AI)
    - What I think you mean: Your bank sends cheques to other banks both on paper and as pictures of the cheques, and clearing operations staff need to see which may go, why, and in which group.
    - Why I am asking: This sets the purpose and the users, so every requirement depends on it.
    - An example: A staff member opens a cheque marked not eligible and sees that Rule A1 section 9 applied because it is disputed.
    - What changes if I am wrong: We would design for the wrong users or the wrong job, such as deciding customer deposits.
- [U-CLR-2] The first version decides which items belong in each image file but does not produce that file, send it, or handle settlement or returns | impact: high | covers: F-CLR-63 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-24
  - v1 (AI)
    - What I think you mean: The first release stops once it knows what should be sent, and another system or person does the sending and everything after it.
    - Why I am asking: This is the scope boundary of the first release.
    - An example: The system lists 12,000 items for one receiving bank, and a separate system turns the list into the real file.
    - What changes if I am wrong: File production, transmission or returns would be added, which needs specifications we do not have.
- [U-CLR-3] Only the item kinds Rule A1 lists, drawn on a Payments Canada member or covered by a listed exception, may be exchanged, and a few kinds may never go as images | impact: high | covers: F-CLR-4, F-CLR-5, F-CLR-6, F-CLR-43 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: The first check is whether the item is of an allowed kind and drawn on an allowed institution, with named exceptions such as Canadian postal money orders; bonds and paper remittances are among those that cannot go as images. This rests on my reading of the two rules.
    - Why I am asking: It determines the item data and reference lists the system needs.
    - My suggestion: Treat the list of member institutions as data that can be updated without changing code.
    - An example: A cheque drawn on an institution that is not on the member list and has no exception is not eligible.
    - What changes if I am wrong: Items would be wrongly sent or wrongly held back.
- [U-CLR-4] A disputed item, a post-dated item, an item dated with no day before that month's first business day, and an item demanding protest may not be exchanged | impact: high | covers: F-CLR-7, F-CLR-8, F-CLR-9, F-CLR-10 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: These four conditions in Rule A1 sections 9 to 12 stop an item from being sent. This rests on my reading of the rule.
    - Why I am asking: Each becomes a required check with its own tests.
    - An example: A cheque dated 2026-12-15 and checked on 2026-11-24 may not be sent.
    - What changes if I am wrong: A barred item could be sent to another bank.
- [U-CLR-5] A Canadian-dollar paper item over 25,000,000 dollars may not be exchanged except for named inter-member settlement purposes, and splitting a payment to get under that limit is also barred | impact: high | covers: F-CLR-11, F-CLR-12 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: Large paper items are blocked unless they are payments between institutions for settlement or emergency reasons, and several smaller items sent in the same cycle to avoid the limit are blocked too. This rests on my reading of the rule.
    - Why I am asking: It sets a threshold and a check across several items at once.
    - My suggestion: Flag possible splitting for a person to judge, because intent cannot be read from amounts alone.
    - An example: Exactly 25,000,000.00 dollars is allowed and 25,000,000.01 dollars is not.
    - What changes if I am wrong: The threshold test or the exception handling would be wrong.
- [U-CLR-6] An item sent as an image must pass the same eligibility checks as paper, so one set of checks serves both | impact: high | covers: F-CLR-42, F-CLR-60, F-CLR-61, F-CLR-62 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: Paper and images share the checks and differ only in how items are grouped and sent. This is my inference, because Rule A10 does not repeat the disqualifying conditions or state an amount limit.
    - Why I am asking: It decides whether there is one rules component or two, and whether the 25,000,000 dollar limit applies to images.
    - My suggestion: Apply every check, including the amount limit, to images until compliance says otherwise.
    - An example: An image of a 30,000,000 dollar cheque would be not eligible, like the paper.
    - What changes if I am wrong: Image items would be blocked that may be sent, or the two paths would need separate rules.
- [U-CLR-7] Each check ends as eligible, waiting, needs review or not eligible, with post-dated and month-dated items waiting and unclear data going to review | impact: high | covers: F-CLR-68, F-CLR-69, F-CLR-70, F-CLR-71 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: Waiting items are rechecked on their date, protest wording and suspected splitting go to a person, and disputed or over-limit items are not eligible.
    - Why I am asking: These outcomes shape the item's states and the work queue for staff.
    - An example: A cheque with an unreadable date goes to review.
    - What changes if I am wrong: Items would sit in the wrong queue or be treated as eligible without checking.
- [U-CLR-8] Some checks can be made from item data alone, some need outside lists or dispute status, and some need image analysis or a person | impact: high | covers: F-CLR-32, F-CLR-33, F-CLR-34 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: Dates, amounts and bundle size are computed; membership and disputes are looked up; protest wording, damage and intent are judged. This is my inference.
    - Why I am asking: It determines the integrations and what stays manual.
    - My suggestion: In the first version, receive protest wording and damage as flags from capture or staff rather than analyse images.
    - An example: The capture system marks an item as torn and the system sorts it as unqualified.
    - What changes if I am wrong: The first version would need image analysis.
- [U-CLR-9] A cheque and its image are one item that is later prepared and exchanged by one method | impact: high | covers: F-CLR-67, F-CLR-72 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: One record follows the cheque, and it goes either in a paper bundle or in an image file group.
    - Why I am asking: It sets the main identity in the design and prevents sending a cheque twice.
    - An example: An item already in an image file group cannot also be added to a paper bundle.
    - What changes if I am wrong: We would need two records and a way to link them.
- [U-CLR-10] Before paper items are sent, attachments are removed, amounts are encoded in the magnetic ink line, other rejected fields are not repaired, errors are corrected and each item is imaged or filmed | impact: high | covers: F-CLR-13, F-CLR-14, F-CLR-15, F-CLR-16, F-CLR-17, F-CLR-19, F-CLR-26 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: These are preparation duties in Rule A1 that happen on capture equipment or by hand. This rests on my reading of the rule.
    - Why I am asking: It decides what the system checks before an item can join a bundle.
    - My suggestion: The system records that each step happened and blocks items where it did not, without performing the steps itself.
    - An example: An item with no encoded amount cannot join an encoded bundle.
    - What changes if I am wrong: The system would need to drive equipment, or would ignore these duties.
- [U-CLR-11] Paper items are sorted into encoded, unqualified and remittance streams and bundled per receiving institution, at most 300 per bundle, with totalled lists and a clearing log | impact: high | covers: F-CLR-20, F-CLR-21, F-CLR-22, F-CLR-23, F-CLR-24, F-CLR-25, F-CLR-74 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: Each bundle holds one kind of item for one receiver, in order; items that cannot be machine read go in the unqualified stream, although those of 50,000 dollars or more may go with encoded items; the receiver reports differences.
    - Why I am asking: These are the bundling rules and the documents the system produces.
    - An example: 650 encoded items for one receiver make bundles of 300, 300 and 50.
    - What changes if I am wrong: Bundles would be rejected or disputed by the receiver.
- [U-CLR-12] Image items need usable front and back pictures at stated resolutions | impact: high | covers: F-CLR-44, F-CLR-45, F-CLR-46 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: A black and white picture at 200 to 240 dots per inch goes with an electronic item, and required writing and signatures must be visible.
    - Why I am asking: It sets an image quality gate before grouping.
    - My suggestion: Take a quality result from the capture system rather than measure images in this system.
    - An example: An image at 180 dots per inch cannot join an image file group.
    - What changes if I am wrong: Unusable images could be sent and returned.
- [U-CLR-13] Image items are grouped by receiver, exchange point, processing site, currency and direction, with at most 40,000 per file | impact: high | covers: F-CLR-48, F-CLR-49, F-CLR-50 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: These keys separate one image file from another, and two institutions may agree to combine exchange points; the file format itself is defined elsewhere and is not produced here.
    - Why I am asking: It sets how image file groups are formed.
    - An example: 90,000 items with the same keys make groups of 40,000, 40,000 and 10,000.
    - What changes if I am wrong: The system that writes the files could not use our groups.
- [U-CLR-14] The first version of this project will not meet the requirement that every Direct Clearer exchange image files with all others from 2026-12-31, and more capabilities will be added to the project later | impact: high | covers: F-CLR-47, F-CLR-79 | response: agree | version: 2 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: Rule A10 sets that date 37 days from today, and an existing system or arrangement has to cover it.
    - Why I am asking: A deadline this close would change the scope, plan and risk of the whole project.
    - My suggestion: Agree that this project has no part in meeting it, and have operations confirm what does.
    - An example: The first release of this system could go live in 2027 without affecting compliance.
    - What changes if I am wrong: The plan would be unrealistic and the work would need to be re-scoped now.
  - comment (Sriram, 2026-11-25): others will be added to this in the future
  - v2 (AI)
    - What I think you mean: Capabilities left out of the first version, such as producing and sending image files, may be added to this project later, but none of them is needed from it by 2026-12-31.
    - Why I am asking: Later additions affect how the design leaves room for them, and the date decides whether any of them is urgent.
    - My suggestion: Specify only the first version now, keep a clear boundary where image file groups are handed on, and have operations confirm what covers the date.
    - An example: A later version could take a closed image file group and produce the file from it, without changing the eligibility checks.
    - What changes if I am wrong: If you meant something else will be added, or something is needed by the date, the scope and plan would need to change.
- [U-CLR-15] A group closes when full or at a set cut-off, can be reopened by an authorized person only before exchange, and is marked exchanged when a person or sending system reports it | impact: high | covers: F-CLR-73, F-CLR-75, F-CLR-76, F-CLR-77 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: Contents are fixed at closing, corrections before sending are recorded, and nothing changes after sending.
    - Why I am asking: These are the group's states, timers and permissions.
    - An example: A wrong amount found at 14:00 on a closed, unsent bundle is corrected after a supervisor reopens it.
    - What changes if I am wrong: The audit trail and correction process would be different.
- [U-CLR-16] Rules change on set dates, so each rule version takes effect on a chosen date and each decision records its version | impact: high | covers: F-CLR-64, F-CLR-65 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: A new version can be prepared early and switched on when the amendment takes effect. The need is my inference from recent amendments to both rules.
    - Why I am asking: It affects how rules are stored and how old decisions are explained.
    - My suggestion: Keep the checks as tested code with their dates and limits as versioned settings, unless operations must edit rules themselves.
    - An example: A decision made on 2026-10-25 still shows the version before amendment 31.
    - What changes if I am wrong: We would build more or less rule configuration than is needed.
- [U-CLR-17] Rule A1 still allows paper pre-authorized debits until 2028-12-01 even though its amendment history describes their removal from 2026-07-27 | impact: high | covers: F-CLR-28 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: The two passages appear to disagree, and the dated note in section 3 is the more specific one.
    - Why I am asking: It decides whether these items are eligible today.
    - My suggestion: Treat them as allowed until 2028-12-01 and have compliance confirm.
    - An example: A paper pre-authorized debit presented in 2027 would pass the item kind check.
    - What changes if I am wrong: The system would accept items that are no longer allowed.
- [U-CLR-18] Duties after sending, including deadlines, acknowledgements, settlement entries, archiving, returns and liability for encoding errors, are outside the first version | impact: normal | covers: F-CLR-18, F-CLR-51, F-CLR-52, F-CLR-53, F-CLR-54, F-CLR-55, F-CLR-56, F-CLR-57, F-CLR-58, F-CLR-59 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: The rules do set these duties, as I have recorded them, but this release does not carry them out.
    - Why I am asking: It confirms the rule facts and that none of them is a first-version requirement.
    - An example: The 5 minute acknowledgement of an image file is handled by the sending system.
    - What changes if I am wrong: Tracking of one or more of these duties would be added.
- [U-CLR-19] We will say sorting stream for paper item classes and settlement stream for the identifier on an image settlement entry | impact: normal | covers: F-CLR-78 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: The two rules use the word stream for different things, so we name them apart.
    - Why I am asking: It keeps names in the specification and code unambiguous.
    - My suggestion: Use the two longer names.
    - An example: A bundle has a sorting stream of unqualified.
    - What changes if I am wrong: Names would need changing later.
- [U-CLR-20] Rule A1 does not cover deposit checks such as stale dates or fraud, depends on other rules and standards, was read as amended to 2026-10-26, and may not be copied without permission | impact: normal | covers: F-CLR-27, F-CLR-29, F-CLR-30, F-CLR-31 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: We cite sections in our own words, and checks A1 does not mention are not assumed to be part of this system.
    - Why I am asking: It limits where requirements come from and what text the product may show.
    - An example: A decision screen shows the section number and our short description, not the rule's text.
    - What changes if I am wrong: Extra checks would be added, or explanations could quote the rule.
- [U-CLR-21] The existing prototype is a design reference only, and we have not shown that building is better than buying | impact: normal | covers: F-CLR-35, F-CLR-36, F-CLR-37, F-CLR-38, F-CLR-39, F-CLR-40, F-CLR-66 | response: agree | version: 1 | reason: none given | by: Sriram | date: 2026-11-25
  - v1 (AI)
    - What I think you mean: The prototype decides customer deposits; we may borrow how it versions rules and records what fired, but it overlaps only slightly and ignores holidays.
    - Why I am asking: It decides whether this is a separate system and how strong the case for building it is.
    - My suggestion: Build separately and borrow the ideas.
    - An example: Our business day calendar includes holidays, which the prototype's does not.
    - What changes if I am wrong: The work would become an extension of the prototype, or a purchase.

## Follow-ups *(required, or none)*

none: all 21 items were checked, and none was disagreed or amended

## Decisions Raised *(required, or none)*

none: all 21 items were checked, and none raised a choice that is still open
