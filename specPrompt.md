/akka:specify Cheque clearing eligibility and exchange preparation (Canada)

Discovery: discovery/clearing-eligibility/. Cite the confirmed findings in square
brackets on each functional requirement, as given below. Paraphrase Payments Canada
Rules A1 and A10 with section numbers. Never copy rule text. [F-CLR-31]

PURPOSE
For a bank sending Canadian-dollar items to other banks, decide whether each item may
be exchanged, show which rule section drove every decision, and prepare eligible items
as paper bundles or image file groups. Users are clearing operations staff.
[F-CLR-1, F-CLR-2, F-CLR-3, F-CLR-41]

USER STORIES, IN PRIORITY ORDER (each independently testable)

1. Eligibility decision

- A cheque and its image are one item with one identity and decision history. It is
  exchanged by one method only. [F-CLR-67, F-CLR-72]
- Outcomes: eligible, waiting, needs review, not eligible. Each decision records the
  reasons, rule sections, time and rule version. [F-CLR-3, F-CLR-68]
- Check item kind, drawee membership and the listed exceptions (A1 s.2, 3, 5).
  [F-CLR-4, F-CLR-5, F-CLR-6]
- Post-dated items wait until their date. Items with a month but no day wait until the
  first business day of that month. Recheck them then. (A1 s.10, 11)
  [F-CLR-8, F-CLR-9, F-CLR-69]
- Disputed items are not eligible. Items directing protest or notice of dishonour need
  review. (A1 s.9, 12) [F-CLR-7, F-CLR-10, F-CLR-71]
- Items over CAD 25,000,000 are not eligible unless a listed inter-member exception
  applies. Suspected splitting to avoid the limit needs review. (A1 s.13)
  [F-CLR-11, F-CLR-12, F-CLR-71]
- Missing or unclear data needs review and is never eligible by default. [F-CLR-70]
- The same checks, including the amount limit, apply to images. Images also exclude
  certain item kinds (A10 s.8b). [F-CLR-43, F-CLR-61, F-CLR-62]
- Dates and amounts are computed; membership and disputes are looked up; protest
  wording, damage and intent arrive as flags from capture or staff.
  [F-CLR-32, F-CLR-33, F-CLR-34]

2. Rule versions

- A version can be loaded early and takes effect on a chosen date. Past decisions keep
  the version that made them. [F-CLR-64, F-CLR-65]
- Paper pre-authorized debits are accepted until 2028-12-01. [F-CLR-28]

3. Paper bundles
- Record that preparation steps happened (attachments removed, amount encoded, errors
  corrected, item imaged or filmed) and hold items where a required step is missing.
  The system does not perform the steps. An item whose attachment cannot be removed is
  not amount-encoded and goes as unqualified. (A1 s.14-18, 21)
  [F-CLR-13 to F-CLR-17, F-CLR-19, F-CLR-26]
- Sort into encoded, unqualified and remittance sorting streams. Unqualified items of
  CAD 50,000 or more may go with encoded items. [F-CLR-20, F-CLR-21, F-CLR-22]
- One sorting stream and one receiver per bundle, at most 300 items, order kept, with
  a totalled list per bundle and a clearing log per exchange.
  [F-CLR-23, F-CLR-24, F-CLR-74]

4. Image file groups

- Require usable front and back images, with a black and white image of 200-240 DPI.
  Take the quality result from capture. [F-CLR-44, F-CLR-45, F-CLR-46]
- Separate groups by receiver, exchange point, processing site, currency and direction,
  at most 40,000 items. Two clearers may agree to combine exchange points.
  [F-CLR-49, F-CLR-50]

5. Operations screen and group lifecycle
- Staff see each decision with its reasons, work the needs-review items, and see group
  contents.
- Groups go open, closed, exchanged. They close when full or at a cut-off, and list and
  totals are then produced. An authorized person may reopen a closed group before
  exchange, with the change and reason recorded. A person or the sending system marks
  it exchanged, after which it never changes.
  [F-CLR-73, F-CLR-75, F-CLR-76, F-CLR-77]
- Use "sorting stream" for paper classes and "settlement stream" for the image
  settlement identifier. [F-CLR-78]

OUT OF SCOPE FOR THE FIRST VERSION
Producing the ANSI image file (Standard 015), sending, acknowledgements, settlement,
returns, archiving and liability tracking. [F-CLR-18, F-CLR-51 to F-CLR-59, F-CLR-63]
Stale-date, account status, funds hold, fraud and duplicate-capture checks.
[F-CLR-30]
Meeting the 2026-12-31 image exchange date. Leave a clear hand-off point so file
production can be added later. [F-CLR-47, F-CLR-48, F-CLR-79]
The existing cheque deposit prototype is a design reference only. [F-CLR-38]

OPEN QUESTIONS: record as assumptions to confirm, not as facts
- Item data source and format. Assume items are submitted through an API.
- Paper or image: assume the method is supplied with each item.
- Receiver, exchange point and processing site: assume supplied reference data.
- Holiday calendar, cut-off times and volumes: assume configurable.
- Who may reopen a group.
- Which A1 s.5 and s.13a exceptions the bank uses.
- Compliance to confirm the amount limit for images and the pre-authorized debit date.
Use synthetic data only.

Before you run it
- Not settled or mocked: the topic is still marked active, and no screen mock exists. Discovery normally does both first. You can still run the prompt; if you consider discovery finished, tell me to mark the topic settled.
- My assumptions: the screen, API submission, and where paper/image method and receiver details come from were never confirmed. Change any that are wrong.
- For planning, not the spec: at /akka:plan, bring up keeping the checks as tested code with versioned dates and limits. Also bring up how to hold groups of up to 40,000 items without one oversized record.
