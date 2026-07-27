---
name: Sync event attendees into Hubilo
description: Register, update, and reconcile attendees (users) for a Hubilo event, including incremental sync of changed records.
api: openapi/hubilo-openapi.yml
operations: [listEvents, addUser, updateUser, listUsers, listDeletedUsers, deregisterUser]
---

# Sync event attendees into Hubilo

Use the Hubilo (Virtual PRO) Public API to keep an event's attendee list in sync from an
external CRM or registration system.

## Auth
Send the organiser Access Token on every call: `Authorization: Bearer <token>`. All limits are
per organiser and combined across every API at **20 requests/second** — throttle accordingly
and treat HTTP `429` as backpressure.

## Steps
1. **Find the event** — call `listEvents` (paginate with `currentPage` + `limit`) and capture the
   integer `eventId`.
2. **Add attendees** — for each new person call `addUser` with `eventId`, `firstName`, `lastName`,
   `email`, and optional profile fields. Emails are the natural key.
3. **Update attendees** — call `updateUser` with `eventId` + `email` to change profile data or
   group membership.
4. **Incremental reconcile** — call `listUsers` with `lastUpdatedAt` + `lastUpdatedAtOp` to pull only
   records changed since your last sync instead of scanning the full list.
5. **Handle removals** — call `deregisterUser` (body: `eventId` + `emails[]`) to deregister people;
   audit removals with `listDeletedUsers`.

## Rules
- No request-level idempotency key exists; use `email` as the dedupe key and prefer `updateUser`
  over re-adding.
- `400` = payload/validation problem, `401` = bad/expired token, `429` = slow down. See
  `errors/hubilo-problem-types.yml`.
- Pagination and rate-limit conventions: `conventions/hubilo-conventions.yml`.
