# Research: cheques-rules-engine

**Project**: cheques-rules-engine | **Topic**: clearing-eligibility (CLR) | **Track**: ideation | **Updated**: 2026-11-25
**Written by**: `/akka:discover <topic> research`

## Summary

A new system for a bank that sends cheques to other banks, both as paper bundles and as images. It decides whether each Canadian-dollar item may be exchanged for clearing under Payments Canada Rule A1, prepares eligible items, and records which section drove each decision. Rule A10 governs the image path and requires Direct Clearers to exchange image files with each other from 2026-12-31. The prototype next to this project decides what happens to a customer's deposit, which is a different job.

Terms used below: a Member belongs to Payments Canada; a Direct Clearer exchanges items directly with other members; the Drawee is the institution a cheque is drawn on; MICR is the magnetic ink line at the bottom of a cheque.

## Sources *(required)*

- S-1: Rule A10, Image Rule | owner: Canadian Payments Association | date: 2026-11-17 | where: https://www.payments.ca/sites/default/files/a10eng.pdf | current: yes, lists amendments up to number 20, effective 2026-11-17

## Users and Needs *(required, or none)*

- [F-CLR-1] The procedures for preparing paper items in Rule A1 are duties of members, mostly of the delivering Direct Clearer that sends the items | evidence: S-CLR-1 s.14 to s.21; understanding.md U-CLR-1 | status: confirmed
- [F-CLR-2] The likely users are clearing operations staff who need to know which items may be exchanged, why, and what is in each bundle | evidence: reasoning: from F-CLR-1 and F-CLR-3; understanding.md U-CLR-1 | status: confirmed

## Outcomes and Measures *(required, or none)*

- [F-CLR-3] Success means each Canadian-dollar item gets an eligibility decision that names the Rule A1 section behind it, and eligible items are prepared as paper bundles or image file groupings | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-1 | status: confirmed
- [F-CLR-63] The first version works out which items belong in each image file but does not produce the ANSI file, send it, or handle settlement or returns | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-2 | status: confirmed
- [F-CLR-79] Capabilities left out of the first version may be added to this project later, and none of them is needed from it by 2026-12-31 | evidence: stated: Sriram, 2026-11-25; understanding.md U-CLR-14 | status: confirmed
- [F-CLR-41] The bank exchanges items both as paper bundles and as images | evidence: stated: Sriram, 2026-11-24; understanding.md U-CLR-1 | status: confirmed

## Constraints and Regulation *(required, or none)*

Rules, approvals, standards and dates that limit what can be built.

### Which items qualify

- [F-CLR-4] An item may be exchanged if it is drawn on a Member or on a local belonging to a member central or cooperative credit association, is payable on demand, is identified as Rule A3 requires, and meets the value and value date requirements | evidence: S-CLR-1 s.2; understanding.md U-CLR-3 | status: confirmed
- [F-CLR-5] The acceptable paper items are Canadian-dollar cheques on a domestic branch account, inter-member debits, money orders and bank drafts, settlement vouchers, paper pre-authorized debits, returned or redirected items, paper and truncated remittances, image printouts and replacement documents, and bill payment error correction debits | evidence: S-CLR-1 s.3; understanding.md U-CLR-3 | status: confirmed
- [F-CLR-6] Some items qualify without being drawn on a Member: claim or reimbursement drafts, Canadian postal money orders, traveller's cheques with a listed institution number, and items of an acquired or amalgamated former member for one year or longer at the President's discretion | evidence: S-CLR-1 s.5; understanding.md U-CLR-3 | status: confirmed

### Which items are disqualified

- [F-CLR-7] An item in dispute under Rule A6 may not be exchanged | evidence: S-CLR-1 s.9; understanding.md U-CLR-4 | status: confirmed
- [F-CLR-8] A post-dated item may not be exchanged | evidence: S-CLR-1 s.10; understanding.md U-CLR-4 | status: confirmed
- [F-CLR-9] An item with no day of the month may not be exchanged before the first business day of the month it shows | evidence: S-CLR-1 s.11; understanding.md U-CLR-4 | status: confirmed
- [F-CLR-10] An item that directs protest or notice of dishonour may not be exchanged | evidence: S-CLR-1 s.12; understanding.md U-CLR-4 | status: confirmed
- [F-CLR-11] A Canadian-dollar paper item over 25,000,000 dollars may not be exchanged, except an inter-member payment that manages settlement balances, corrects a flow imbalance between the two clearing systems (ACSS and Lynx) or pays during a Lynx emergency | evidence: S-CLR-1 s.13a; understanding.md U-CLR-5 | status: confirmed
- [F-CLR-12] Two or more paper items may not be exchanged in the same ACSS cycle for the purpose of avoiding the 25,000,000 dollar limit | evidence: S-CLR-1 s.13b; understanding.md U-CLR-5 | status: confirmed

### Preparing paper items

- [F-CLR-13] Each member must make reasonable effort to keep the number of unqualified items (items that cannot be machine processed) to a minimum | evidence: S-CLR-1 s.14; understanding.md U-CLR-10 | status: confirmed
- [F-CLR-14] Attachments that do not belong to an item are removed before its amount is encoded, and an item whose attachment cannot be removed is not amount-encoded and counts as unqualified | evidence: S-CLR-1 s.15; understanding.md U-CLR-10 | status: confirmed
- [F-CLR-15] Every machine-processable item has its amount encoded in the MICR line before exchange at a regional exchange point | evidence: S-CLR-1 s.16a; understanding.md U-CLR-10 | status: confirmed
- [F-CLR-16] The delivering Direct Clearer also encodes other MICR fields as appropriate on returned item envelopes, pre-authorized debits, replacement documents and bill payment error correction debits | evidence: S-CLR-1 s.16b; understanding.md U-CLR-10 | status: confirmed
- [F-CLR-17] When fields other than the amount are rejected, the item is not repaired, and a replacement document follows Standard 014 | evidence: S-CLR-1 s.16c, s.16d; understanding.md U-CLR-10 | status: confirmed
- [F-CLR-18] The member that first encodes an amount, or exchanges an item a third party encoded, is responsible for the difference if the item is returned within 90 calendar days of the Drawee receiving it | evidence: S-CLR-1 s.17; understanding.md U-CLR-18 | status: confirmed
- [F-CLR-19] Wrong amount encoding, wrong currency and incorrect bundle listings found by the delivering Direct Clearer are corrected before exchange | evidence: S-CLR-1 s.18; understanding.md U-CLR-10 | status: confirmed
- [F-CLR-20] Items are separated for each Direct Clearer into three streams: encoded, unqualified and paper remittances | evidence: S-CLR-1 s.19; understanding.md U-CLR-11 | status: confirmed
- [F-CLR-21] The encoded stream may include unqualified items of 50,000 dollars and over | evidence: S-CLR-1 s.19a; understanding.md U-CLR-11 | status: confirmed
- [F-CLR-22] The unqualified stream covers items unsuitable for a MICR reader, including items with no institution number encoding or obvious domicile changes, mutilated items, unqualified returned or redirected items, items with attachments and adjusting entries | evidence: S-CLR-1 s.19b; understanding.md U-CLR-11 | status: confirmed
- [F-CLR-23] Items are exchanged by stream in separate bundles of at most 300 items, packaged to keep their order | evidence: S-CLR-1 s.20a; understanding.md U-CLR-11 | status: confirmed
- [F-CLR-24] Each bundle has a totalled list in item order showing the processing date and delivering Direct Clearer, and each exchange of bundles has a Cheque Clearing Log | evidence: S-CLR-1 s.20b, s.20c; understanding.md U-CLR-11 | status: confirmed
- [F-CLR-25] The receiving Direct Clearer reports any discrepancy between the bundles received and the logs or lists | evidence: S-CLR-1 s.20d; understanding.md U-CLR-11 | status: confirmed
- [F-CLR-26] Every item is imaged or microfilmed before exchange, with controls on image quality and integrity, or other arrangements are made with the receiver when equipment fails | evidence: S-CLR-1 s.21; understanding.md U-CLR-10 | status: confirmed

### Version, dependencies and limits of the rule

- [F-CLR-27] The copy read on 2026-11-24 lists amendments up to number 31, effective 2026-10-26 | evidence: S-CLR-1 pages 3 to 5; understanding.md U-CLR-20 | status: confirmed
- [F-CLR-28] Section 3 still lists paper pre-authorized debits and says they stop being acceptable on 2028-12-01, while amendment 30 is described as removing them with effect from 2026-07-27 | evidence: S-CLR-1 s.3 and amendment history item 30; understanding.md U-CLR-17 | status: confirmed
- [F-CLR-29] Rule A1 refers to other documents for detail, including Rules A3, A4, A6, A10, B2 and H and Standards 010 and 014 | evidence: S-CLR-1 s.2 to s.21; understanding.md U-CLR-20 | status: confirmed
- [F-CLR-30] Rule A1 sets no requirement about stale-dated cheques, account status, funds holds, fraud or duplicate items | evidence: S-CLR-1 pages 1 to 11, the whole document; understanding.md U-CLR-20 | status: confirmed
- [F-CLR-31] The Canadian Payments Association holds copyright in Rule A1 and requires written permission to reproduce it in whole or in part | evidence: S-CLR-1 page 1; understanding.md U-CLR-20 | status: confirmed

### Image exchange under Rule A10

An ICP file is an electronic file of image captured payments sent from one Direct Clearer to another.

- [F-CLR-42] An image of a cheque made as Rule A10 requires may be treated as the original item, including under the Bills of Exchange Act | evidence: S-1 s.2, s.5 to s.7; understanding.md U-CLR-6 | status: confirmed
- [F-CLR-43] Canadian-dollar bonds and coupons, paper remittances, certain Government of Canada securities and returned item envelopes may not be exchanged as images | evidence: S-1 s.8b; understanding.md U-CLR-3 | status: confirmed
- [F-CLR-44] Images show the front and back of an item at a minimum of 200 DPI in black and white or 120 DPI in greyscale | evidence: S-1 s.10; understanding.md U-CLR-12 | status: confirmed
- [F-CLR-45] An item exchanged electronically has a black and white image between 200 and 240 DPI | evidence: S-1 s.11, s.24a; understanding.md U-CLR-12 | status: confirmed
- [F-CLR-46] The member that captures an image is responsible for it being usable, meaning that required fields are legible and signatures viewable | evidence: S-1 s.4, s.9; understanding.md U-CLR-12 | status: confirmed
- [F-CLR-47] From 2026-12-31 each Direct Clearer must be able to send and receive ICP files for forward presentment, have completed testing with every other Direct Clearer and exchange with all of them | evidence: S-1 s.21; understanding.md U-CLR-14 | status: confirmed
- [F-CLR-48] ICP files follow the ANSI X9.100-187-2008 specification and Standard 015 | evidence: S-1 s.24; understanding.md U-CLR-13 | status: confirmed
- [F-CLR-49] An ICP file holds at most 40,000 items | evidence: S-1 s.24d; understanding.md U-CLR-13 | status: confirmed
- [F-CLR-50] Each ICP file is unique to a receiving Direct Clearer, exchange point, processing site, currency and direction, and the two clearers may agree to combine exchange points | evidence: S-1 s.24e, s.24f; understanding.md U-CLR-13 | status: confirmed
- [F-CLR-51] A Canadian-dollar ICP file is sent no later than the last exchange time, and the receiver may reject a late file, accept it or hold it for the next cycle | evidence: S-1 s.26, s.27; understanding.md U-CLR-18 | status: confirmed
- [F-CLR-52] The receiving Direct Clearer acknowledges each ICP file within 5 minutes of receiving it | evidence: S-1 s.28; understanding.md U-CLR-18 | status: confirmed
- [F-CLR-53] Items settle on the business day after the clearing cycle in which the file is received, and a file that misses the deadline may settle one business day later | evidence: S-1 s.29; understanding.md U-CLR-18 | status: confirmed
- [F-CLR-54] After an acknowledgement the delivering Direct Clearer enters a debit in the clearing system (ACSS) against the receiver, under stream O or S, with the number and value of items | evidence: S-1 s.30; understanding.md U-CLR-18 | status: confirmed
- [F-CLR-55] A receiver that rejects an ICP file notifies the sender and contests the entry, and the sender may be allowed time to send a new file | evidence: S-1 s.31; understanding.md U-CLR-18 | status: confirmed
- [F-CLR-56] Each delivering Direct Clearer sends an end-of-day notice per currency to each receiver, with a nil report when no files were sent | evidence: S-1 s.32; understanding.md U-CLR-18 | status: confirmed
- [F-CLR-57] The capturing member archives images as Standard 018 requires and destroys the paper original within 120 calendar days when the image was made for exchange | evidence: S-1 s.12, s.17; understanding.md U-CLR-18 | status: confirmed
- [F-CLR-58] ICP files travel over the Payments Canada network or a qualifying private network, from a primary and a recovery site that is tested yearly | evidence: S-1 s.22, s.23, s.25; understanding.md U-CLR-18 | status: confirmed
- [F-CLR-59] An item received as an image is returned by ICP file no later than the business day after it reaches the part of the Drawee that can decide, and a wrong amount of 20 dollars or less is not a ground for return | evidence: S-1 s.47a, s.52b; understanding.md U-CLR-18 | status: confirmed
- [F-CLR-60] Rule A10 states no maximum item amount and does not restate the disqualifying conditions of Rule A1 | evidence: S-1 pages 1 to 28, the whole document; understanding.md U-CLR-6 | status: confirmed
- [F-CLR-61] An item exchanged as an image must also be eligible under Rule A1 | evidence: reasoning: S-1 s.2 and s.4 speak of eligible items, and S-CLR-1 s.4 lists image exchange among acceptable items; understanding.md U-CLR-6 | status: confirmed
- [F-CLR-62] One set of eligibility checks can serve both paper and image exchange, with the two differing in how eligible items are prepared and sent | evidence: reasoning: from F-CLR-23, F-CLR-49, F-CLR-50 and F-CLR-61; understanding.md U-CLR-6 | status: confirmed

### What software can decide

- [F-CLR-32] Post-dating, month-dating, the single item amount limit, stream choice and bundle size can be decided from item data, given a business day calendar | evidence: reasoning: from F-CLR-8, F-CLR-9, F-CLR-11, F-CLR-20 and F-CLR-23; understanding.md U-CLR-8 | status: confirmed
- [F-CLR-33] Drawee membership and dispute status need data from outside the item | evidence: reasoning: from F-CLR-4 and F-CLR-7; understanding.md U-CLR-8 | status: confirmed
- [F-CLR-34] Protest directions, intent to split an amount, unremovable attachments and physical damage are likely to need image analysis or a person | evidence: reasoning: from F-CLR-10, F-CLR-12, F-CLR-14 and F-CLR-22; understanding.md U-CLR-8 | status: confirmed

## Prior Work *(required, or none)*

Earlier attempts, internal documents and existing decisions that bear on this work.

- [F-CLR-35] The prototype evaluates rules whose conditions are combined with AND, and the same facts and rule set version always give the same decision | evidence: S-CLR-2 domain/Rule.java:6; S-CLR-2 domain/RuleEngine.java:12; understanding.md U-CLR-21 | status: confirmed
- [F-CLR-36] The prototype records with each decision the rule set version, every rule that fired and any referral reasons | evidence: S-CLR-2 domain/RuleTrace.java:5; understanding.md U-CLR-21 | status: confirmed
- [F-CLR-37] The prototype stores a loaded rule set version as inactive until it is activated, and caches compiled versions on each node | evidence: S-CLR-2 application/RuleSetEntity.java:15; S-CLR-2 application/RuleSetCache.java:7; understanding.md U-CLR-21 | status: confirmed
- [F-CLR-38] The prototype's decisions are accept, hold, refer or return for a customer deposit | evidence: S-CLR-2 domain/Decision.java:3; understanding.md U-CLR-21 | status: confirmed
- [F-CLR-39] The prototype returns post-dated cheques, which overlaps with the post-dating condition in Rule A1 | evidence: /Users/sriram/code/rbc-poc/rbc-cheque-deposit-prototype/src/main/resources/synthetic/rules/rules-v1.csv:6; S-CLR-1 s.10; understanding.md U-CLR-21 | status: confirmed
- [F-CLR-40] The prototype counts weekdays as business days and does not model holidays | evidence: S-CLR-2 domain/RuleEngine.java:18; understanding.md U-CLR-21 | status: confirmed

## Unknowns *(required, or none)*

What nobody has answered yet, with the role most likely to know.

- Does the 25,000,000 dollar limit, which Rule A1 section 13 words for paper items, apply to items exchanged as images? | ask: compliance
- Is the bank a Direct Clearer, and what meets its obligation under Rule A10 section 21 from 2026-12-31? | ask: clearing operations
- What decides whether an item goes as paper or as an image? | ask: clearing operations
- Does the bank hold Standard 015 and the ANSI X9.100-187-2008 specification, which Rule A10 says is sold by ANSI? | ask: clearing operations
- Where does item data come from, and in what form? | ask: item capture system owner
- How many items arrive per day, and what are the exchange cut-off times? | ask: clearing operations
- Which holiday calendar defines a business day? | ask: clearing operations
- Which exceptions in sections 5 and 13a does the bank actually handle? | ask: clearing operations
- Are paper pre-authorized debits still to be accepted until 2028-12-01? | ask: compliance
- What do Rules A3, A4, A6 and B2 and Standards 010, 014, 015 and 018 add to these requirements? | ask: Sriram, to approve reading the public copies

## Requests for Other People *(required, or none)*

What to ask of whom, each with one sentence on why it matters.

- Clearing operations: describe the path from item capture to exchange, including who handles exceptions | why: it decides where the system sits and which checks a person still performs.
- Compliance or legal: confirm whether rule text may be reproduced in the product or its documents | why: decision explanations may be limited to section numbers and our own wording.
