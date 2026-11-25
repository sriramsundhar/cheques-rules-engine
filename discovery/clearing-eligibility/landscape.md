# Landscape: cheques-rules-engine

**Project**: cheques-rules-engine | **Topic**: clearing-eligibility (CLR) | **Track**: ideation | **Updated**: 2026-11-25
**Written by**: `/akka:discover <topic> landscape`

## Question Being Compared

Decide which cheques and other items may be exchanged for clearing, group them for sending as paper or images, and be able to show why.

## Options *(required)*

Every way the job is done today or could be done, including doing nothing.

| Option | Kind | Who uses it | Finding |
|--------|------|-------------|---------|
| O-1 Today's process | status quo | the bank | unknown |
| O-2 Checks built into the item capture system | alternative | unknown | unknown |
| O-3 A bought product or outsourced service | competitor | unknown | unknown |
| O-4 Manual review by staff | substitute | unknown | unknown |
| O-5 Extend the cheque deposit prototype | alternative | prototype only | [F-CLR-38] |
| O-6 A new Akka service | alternative | nobody yet | [F-CLR-3] |

## Dimensions *(required)*

The dimensions the reader chose, each with why it matters to them. A dimension is
numbered L-<n>; D-<n> is kept for the plan's decisions.

- L-1 Traceability: the goal asks that every decision show the section that drove it, so this comes first
- L-2 Ease of change: an amended rule has to take effect on its date
- L-3 Timeliness: the day's volume has to be processed before the exchange cut-off
- L-4 Fit: the result has to work with existing capture and exchange systems
- L-5 Cost: what it takes to build and run

## Comparison *(required)*

Every cell names the finding that supports it, or reads `unknown`.

| Option | L-1 Traceability | L-2 Ease of change | L-3 Timeliness | L-4 Fit | L-5 Cost |
|--------|------------------|--------------------|----------------|---------|----------|
| O-1 Today's process | unknown | unknown | unknown | unknown | unknown |
| O-2 Capture system checks | unknown | unknown | unknown | unknown | unknown |
| O-3 Product or service | unknown | unknown | unknown | unknown | unknown |
| O-4 Manual review | unknown | unknown | unknown | unknown | unknown |
| O-5 Extend the prototype | Records rule set version and fired rules [F-CLR-36] | Versions are loaded and then activated [F-CLR-37] | unknown | Built for deposit decisions [F-CLR-38] | unknown |
| O-6 New Akka service | Required by the goal, not yet built [F-CLR-3] | unknown | unknown | unknown | unknown |

## Findings *(required)*

- [F-CLR-64] Both rules were amended recently, with amendment 31 to Rule A1 effective 2026-10-26 and amendment 20 to Rule A10 effective 2026-11-17 | evidence: S-CLR-1 pages 3 to 5; S-1 pages 4 to 5; understanding.md U-CLR-16 | status: confirmed

## What This Means for the Specification *(required, or none)*

Inferences only; each one is a finding with status inferred.

- [F-CLR-65] A rule version should be able to take effect on a chosen date, and each decision should record the version that made it | evidence: reasoning: from F-CLR-64, F-CLR-3 and F-CLR-37; understanding.md U-CLR-16 | status: confirmed
- [F-CLR-66] The comparison does not yet show that building is better than buying, because today's process and available products have not been examined | evidence: reasoning: from F-CLR-3 and the unknown cells for options O-1 to O-4; understanding.md U-CLR-21 | status: confirmed

## Unknowns *(required, or none)*

- How is this job done today, including any checks already in the capture system? | ask: clearing operations
- Which products or services does the bank use or consider for eligibility and exchange preparation? | ask: clearing operations
- What daily volume and exchange cut-off times must be met? | ask: clearing operations
- What do public sources say about products that do this job? | ask: Sriram, to approve a web search
