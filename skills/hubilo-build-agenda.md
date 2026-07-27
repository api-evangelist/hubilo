---
name: Build an event agenda in Hubilo
description: Create tracks, sessions, and speakers for a Hubilo event and pull session engagement analytics.
api: openapi/hubilo-openapi.yml
operations: [listEvents, addUpdateTrackForSessions, addUpdateSession, addUpdateSpeaker, listSessions, listSessionRegistrations, listSessionAttendanceDetails]
---

# Build an event agenda in Hubilo

Assemble an event's agenda — tracks, sessions, speakers — then read back engagement.

## Auth
`Authorization: Bearer <organiser access token>` on every call. Stay under 20 req/s per organiser.

## Steps
1. **Resolve the event** — `listEvents` to get the `eventId`.
2. **Create tracks** — `addUpdateTrackForSessions` (upsert) to define agenda tracks for the event.
3. **Create sessions** — `addUpdateSession` (upsert) with the session details and its track; capture
   the returned `sessionId` / `agendaId`.
4. **Attach speakers** — `addUpdateSpeaker` (upsert) with `eventId` + `sessionId` + speaker email.
5. **Verify** — `listSessions` to confirm the agenda.
6. **Measure** — `listSessionRegistrations` and `listSessionAttendanceDetails` (by `agendaId`) to pull
   who registered for and attended each session.

## Rules
- Session, speaker, and track writes are **upserts** — safe to replay on the same business key,
  which substitutes for the missing request-level idempotency key.
- Errors and rate-limit semantics: `errors/hubilo-problem-types.yml`,
  `conventions/hubilo-conventions.yml`.
