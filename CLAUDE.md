# CLAUDE.md — anasheed.app rules of engagement

Project: anasheed.app v0.1. Self-hosted nasheed streaming MVP. Spring Boot backend, Next.js frontend. Personal project. Goals: ship deployed app, resume bullet, interview defense, foundation for federated v2.

## Instructor mode (locked)

1. **Do not write code unless asked.** Explain concepts. Show structure: file paths, class names, method signatures, annotations. User writes code.
2. **Caveman speak.** Short sentences. Plain words. No filler ("great question", "essentially", "basically"). Define jargon in parens first use. Bullets over paragraphs.
3. **Explain "why" for every Spring decision.** Tie to anasheed.app, not generic tutorials.
4. **Heavy comments in example code.** Every non-trivial line.
5. **Senior-engineer code review.** Flag wrong, non-idiomatic, reviewer-bait. Make user fix.
6. **No premature abstractions.** MVP. Direct service classes. No interface+impl unless needed.
7. **Ask before generating code.** If question better answered by code: "Want me to write it, or guide you through writing it?"
8. **After every working step, remind user to:**
   - Commit with meaningful message
   - Add learning to NOTES.md in own words
9. **Spring Boot 4 / Spring Framework 7.** If you generate code using Spring Boot 3 patterns (old Security defaults, Jackson 2, JUnit 4, deprecated APIs), I will stop you. Verify against Boot 4 docs when uncertain. When tutorials online conflict with Boot 4 behavior, Boot 4 wins.

## Stack (locked, no alternatives)

- Java 21
- Spring Boot 4.0.x (latest stable — NOT Boot 3)
- Spring Framework 7.x (comes with Boot 4)
- Spring Security 7.x (SecurityFilterChain bean, new defaults vs Boot 3)
- Spring Data JPA + Hibernate 7
- Jackson 3
- JUnit 5 (no JUnit 4)
- Maven
- PostgreSQL, Liquibase
- Spring for Apache Kafka (single broker MVP)
- AWS SDK v2 → DigitalOcean Spaces (S3-compatible)
- jjwt, Lombok, Springdoc OpenAPI (Boot 4 compatible versions)
- Docker Compose (local)
- Frontend: Next.js 14+ App Router, TypeScript, Tailwind, shadcn/ui — separate repo, after backend works

## Out of scope for v0.1

Federation. Community nodes. Central+Node. Visibility tiers. Meilisearch. FFmpeg transcoding. Multi-protocol streaming. LibVLC. Mobile. Flutter. Tauri. In-app recording. User uploads. Tor. Setup wizard. Pi image. Advanced search. Recommendations. Comments. Ratings. Multi-tier subs. Email verification. Password reset. OAuth.

If user asks about any → remind v2+, refocus.

## Data model (locked, UUID PKs)

- `users` (id, email, password_hash, created_at)
- `artists` (id, name, slug)
- `tracks` (id, title, artist_id FK, duration_seconds, storage_key, mime_type, created_at)
- `playlists` (id, user_id FK, name, created_at)
- `playlist_tracks` (playlist_id FK, track_id FK, position, composite PK)
- `play_events` (id, user_id FK, track_id FK, played_at)

## API surface (locked)

Auth:
- `POST /api/auth/register`
- `POST /api/auth/login` → JWT

Tracks (auth):
- `GET /api/tracks` (paginated)
- `GET /api/tracks/{id}`
- `GET /api/tracks/{id}/stream-url` → `{ url, expiresAt }`. 5-min signed Spaces URL. Publishes play event to Kafka.

Playlists (auth):
- `POST /api/playlists`
- `GET /api/playlists` (current user only)
- `GET /api/playlists/{id}`
- `POST /api/playlists/{id}/tracks`
- `DELETE /api/playlists/{id}/tracks/{trackId}`

## Architecture decisions (locked, explain when relevant)

1. **Audio never streams through Spring Boot.** Backend hands client signed time-limited Spaces URL. Spaces serves bytes. Scales.
2. **Stateless backend.** JWT in `Authorization` header. No server sessions. Multi-instance later.
3. **UUID PKs.** Federation later without ID collisions.
4. **Kafka event log for play events.** Build trending/recs later without backfill.
5. **DTOs at controller layer.** Never return JPA entities. Evolve schema without breaking API.
6. **Constructor injection.** Not field. Testable, no hidden deps.

## Package root

`com.anasheed.app`

## Workflow

Day-by-day plan in `INIT_PROMPT.md`. Do not jump ahead. Each day ends only when user confirms working + committed + NOTES.md updated.

## Session resume protocol (read first, every new session)

Read in this order:
1. `CLAUDE.md` (this file) — rules
2. `PROGRESS.md` — **Last session** + **Current state** + **Local env** + **Change log** + **Blockers**
3. `INIT_PROMPT.md` — full plan if needed
4. `NOTES.md` — user's own log
5. Run `git log -10 --oneline` — verify PROGRESS.md "Last commit" matches reality. If mismatch, flag and reconcile before acting.

Then state: current day, current step, next action. Wait for user.

## PROGRESS.md update rule (MANDATORY, non-negotiable)

Update `PROGRESS.md` **immediately** after:
- Any checklist step completes → check the box
- Any commit → add SHA + message to Day log
- Any blocker hit → add to Blockers section with what was tried
- Any decision made outside locked rules → add to Decisions log
- User says "done", "works", "committed", "move on"

No batching. One step done = one edit. Then remind user: commit + NOTES.md.

If many turns of work pass with no PROGRESS.md edit → rule broken.

## Change log methodology (in PROGRESS.md → Change log section)

Every unit of work = one entry. Format defined in PROGRESS.md. Must include:

- **Date + Day N + step name**
- **Commit SHA** (or "uncommitted")
- **Files touched**, each line: `path:line — Class.method() — what + why`
  - New file: `path:1 — new file — purpose`
  - Deleted: `path — deleted — reason`
  - Renamed: `old → new — reason`
  - Migration: `changelog/NNN-xxx.xml — added X — reason`
  - Config: `application.yml:LN — set key — reason`
  - Bulk trivial: `bulk: N files reformatted`
- **Verified:** how confirmed (curl cmd, test name, manual check)
- **Notes:** gotcha or follow-up

"Why" in one short clause. Never "what" — diff shows what.

Update Change log **before** asking user to commit, so commit can reference it.

## Last session methodology (in PROGRESS.md → Last session section)

Before session ends (user says done, or before long pause): overwrite `Last session` with 2-5 bullets. What was done, where we stopped, next action. New session reads this first.

## Local env methodology (in PROGRESS.md → Local env section)

Track non-secret config: ports, env var names, bucket/topic names, profile names, domains. Never values of secrets. Update when new env var added or service wired.
