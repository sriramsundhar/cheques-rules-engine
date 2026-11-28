# Quickstart

This describes how the service is run once it is built. Nothing here works yet.

## Prerequisites

Java 21 or later, Maven, the Akka CLI, and a configured Akka repository token.

## Build and test

```shell
mvn verify
```

## Run locally

```shell
mvn compile exec:java
```

The service listens on `http://localhost:9000`. The screen is at `/ui/`.

## A development token

Dev mode and tests accept unsigned tokens. A deployed service does not.

```shell
token() {
  local h p
  h=$(printf '{"alg":"none"}' | base64 | tr -d '=\n' | tr '/+' '_-')
  p=$(printf '{"iss":"clearing-dev","sub":"%s","role":"%s"}' "$1" "$2" | base64 | tr -d '=\n' | tr '/+' '_-')
  printf '%s.%s.' "$h" "$p"
}
```

## Walk through (synthetic data only)

1. Load members, routing and the calendar as a rule administrator.

   ```shell
   curl -X PUT localhost:9000/reference/members \
     -H "Authorization: Bearer $(token admin-1 rule-administrator)" \
     -H 'Content-Type: application/json' \
     -d '{"members":["901","902"],"exceptions":[]}'
   ```

2. Submit an eligible item.

   ```shell
   curl -X POST localhost:9000/items \
     -H "Authorization: Bearer $(token capture-1 capture-system)" \
     -H 'Content-Type: application/json' \
     -d '{"itemId":"SYN-000101","kind":"CHEQUE","amountCents":125000,"currency":"CAD",
          "date":{"year":2026,"month":11,"day":27},"drawee":"901","localDrawee":false,
          "payableOnDemand":true,"identifiedAsRequired":true,"meetsValueConditions":true,
          "method":"PAPER","flags":[]}'
   ```

   Expect `ELIGIBLE` with the sections checked and the rule version.

3. Record the preparation steps. The item appears in an open bundle in `GET /groups`.
4. As a rule administrator, set a cut-off a few minutes ahead in the calendar, then see the
   bundle close.
5. Report it exchanged as the sending system. Reopening it is then refused.
6. Submit an item over CAD 25,000,000.00 and open it on the screen. Expect `NOT_ELIGIBLE`
   citing A1 s.13.

Institution numbers here are made up.
