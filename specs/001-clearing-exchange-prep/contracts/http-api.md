# HTTP API Contract

JSON over HTTP. Every route except `/ui/**` needs a bearer token with a `role` claim and a
`sub` claim. Amounts are integers in cents. Dates are `YYYY-MM-DD`. Request and response
records belong to the endpoints, not to the domain.

Roles: `OP` operator, `SU` supervisor, `RA` rule-administrator, `SS` sending-system,
`CS` capture-system (proposed, see the Decisions Log).

## Items (`ItemEndpoint`)

| Method and path | Roles | Body | Result |
|-----------------|-------|------|--------|
| `POST /items` | CS | item data | 201 with decision; 200 when already submitted with the same data; 409 different data; 400 not CAD or malformed id |
| `GET /items?outcome=&pageToken=` | OP SU RA | | page of summaries |
| `GET /items/{itemId}` | OP SU RA | | item, state, decisions with reasons and sections |
| `PUT /items/{itemId}/preparation` | CS OP SU | steps done | item |
| `PUT /items/{itemId}/image-quality` | CS | frontUsable, backUsable, blackAndWhiteDpi | item |
| `PUT /items/{itemId}/exchange-method` | CS OP SU | method | item; 409 while in a group |
| `POST /items/{itemId}/flags` | CS OP SU | flag, reason | new decision |
| `PUT /items/{itemId}/dispute` | CS OP SU | reason | new decision |
| `DELETE /items/{itemId}/dispute` | CS OP SU | reason | new decision |
| `POST /items/{itemId}/corrections` | OP SU | changed fields, removed flags, reason | new decision; 409 when its group is closed |
| `POST /items/{itemId}/flag-confirmations` | OP SU | flag, reason | not eligible decision |

There is no route that sets an outcome (FR-036). A missing reason is a 400.

Item data: itemId, kind, amountCents, currency, date (`{year, month, day?}`), drawee,
localDrawee, payableOnDemand, identifiedAsRequired, meetsValueConditions, interMemberPurpose,
method, flags.

Decision: outcome, reasons (`code`, `section`, `explanation`), sectionsChecked, exceptionUsed,
recheckOn, ruleVersionId, decidedAt.

## Exchange groups (`ExchangeGroupEndpoint`)

| Method and path | Roles | Body | Result |
|-----------------|-------|------|--------|
| `GET /groups?status=&pageToken=` | OP SU RA | | page of group summaries |
| `GET /groups/{groupId}` | OP SU RA | | header, keys, totals, flags, history |
| `GET /groups/{groupId}/items?after=` | OP SU RA | | up to 500 entries in order |
| `POST /groups/{groupId}/reopen` | SU | reason | group; 409 unless closed |
| `DELETE /groups/{groupId}/items/{itemId}` | SU | reason | group; 409 unless reopened |
| `POST /groups/{groupId}/close` | SU | | 202; 409 unless reopened |
| `POST /groups/{groupId}/exchanged` | OP SU SS | acknowledgeFlag | group; 409 with the flags when a person has not acknowledged them |
| `GET /clearing-logs?receiver=&cycle=` | OP SU | | bundles, totals, processing date, delivering Direct Clearer |

The word for paper classes is `sortingStream`. The word for the identifier on an image
settlement entry is `settlementStream` (FR-038). This release has no settlement entries.

## Rule versions (`RuleVersionEndpoint`)

| Method and path | Roles | Result |
|-----------------|-------|--------|
| `GET /rule-versions` | OP SU RA | versions and the one in force |
| `POST /rule-versions` | RA | 201; 400 date not in the future; 409 date already used |

## Reference data (`ReferenceDataEndpoint`)

| Method and path | Roles (proposed) |
|-----------------|------------------|
| `PUT /reference/members` | RA |
| `PUT /reference/routing` | RA |
| `PUT /reference/calendar` | RA |
| `GET /reference/{members, routing, calendar}` | OP SU RA |

## Operations screen (`UiEndpoint`)

`GET /ui/**` serves the screen. It follows mock M-CLR-1: tabs for items, needs review and
groups.

## Errors

400 invalid request, 401 no valid token, 403 role not allowed, 404 unknown id, 409 not allowed
in the current state. The body is a short message in plain words with no rule text (FR-039).
