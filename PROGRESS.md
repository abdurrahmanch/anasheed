# PROGRESS.md — where we are

**Single source of truth for session resume.** New Claude session reads this first to know exact state.

## Update rules (MANDATORY)

Claude MUST update this file:
- After every completed step (check the box)
- After every commit (add commit SHA + message under Day log)
- When blocker hit (add to Blockers, with what was tried)
- When decision made outside CLAUDE.md (add to Decisions)
- When user says "done with X" / "X works" / "committed"

No batching. Update immediately. One step done → one edit here. Then remind user to commit + write NOTES.md.

If session resumes and PROGRESS.md not touched in N turns of work → Claude failed rule. User should call out.

---

## Current state

- **Day:** 1
- **Step:** application.yml (local + prod profiles)
- **Last commit:** _(uncommitted — scaffold + pom deps)_
- **Next action:** write `app/src/main/resources/application.yml` with local + prod profiles, env vars for secrets
- **Blocker:** none

---

## Last session

_(Claude writes 2-5 bullets before stopping. New session reads this first to know what just happened. Overwrite each session, do not append.)_

- 2026-06-11: scaffold generated via Initializr (Boot 4.1.0). Added jjwt 0.13.0, AWS SDK v2 BOM 2.46.8 + s3, Springdoc 3.0.3. `mvn compile` green. Pending commit. Next: application.yml profiles.

---

## Local env

_(Non-secret config pointers. Ports, env var names, bucket name, service names. No secrets, no passwords, no keys. Update when added.)_

- Java: 21
- Build: Maven
- Spring Boot: 4.1.0
- Package root: `com.anasheed.app`
- Local ports: postgres 5432, kafka 9092, zookeeper 2181, spring 8080
- Profiles: `local`, `prod`
- Env vars (names only): `DB_URL`, `DB_USER`, `DB_PASSWORD`, `JWT_SECRET`, `SPACES_ENDPOINT`, `SPACES_BUCKET`, `SPACES_ACCESS_KEY`, `SPACES_SECRET_KEY`, `KAFKA_BOOTSTRAP_SERVERS` _(placeholders — confirm/edit when wiring)_
- DO Spaces bucket: _(set when created)_
- DO Spaces region: _(set when created)_
- Kafka topic: `play-events` _(tentative)_
- Domain: `anasheed.app` (frontend, Vercel), `api.anasheed.app` (backend, Cloudflare Tunnel → homelab)

---

## Day 1 — Backend foundation

- [x] Prereqs verified (Java 21, Maven, Docker, DO Spaces creds, etc.) — Java 21.0.11, Maven 3.9.16, Docker hello-world ok, Compose 5.1.4. DO Spaces creds deferred to Day 2.
- [x] Spring Initializr scaffold generated
- [x] Package `com.anasheed.app` set
- [ ] `application.yml` with `local` + `prod` profiles, env vars for secrets
- [ ] `docker-compose.yml`: postgres (5432), kafka + zookeeper (9092)
- [ ] Liquibase changelog: 6 tables (users, artists, tracks, playlists, playlist_tracks, play_events)
- [ ] JPA entities for all 6 tables
- [ ] `GET /api/health` returns `{ "status": "ok" }`
- [ ] Verified: `docker compose up`, `mvn spring-boot:run`, curl health
- [ ] Committed
- [ ] NOTES.md updated

## Day 2 — Tracks + Spaces

- [ ] DO Spaces bucket created
- [ ] `S3Service` with AWS SDK v2, Spaces endpoint
- [ ] 15-25 nasheeds uploaded to Spaces
- [ ] Metadata rows inserted
- [ ] DTOs: `TrackDto`, `ArtistDto`, `PageDto<T>`
- [ ] `TrackController`: list, detail, stream-url
- [ ] Signed URL: 5-min expiry
- [ ] Verified: VLC plays nasheed from signed URL
- [ ] Committed
- [ ] NOTES.md updated

## Day 3 — Auth

- [ ] `SecurityFilterChain` bean
- [ ] `PasswordEncoder` bean (BCrypt)
- [ ] JWT util (sign, verify, extract claims) via jjwt
- [ ] `JwtAuthFilter extends OncePerRequestFilter`
- [ ] `AuthController`: register, login
- [ ] `AuthService`: register hashes, login validates + issues JWT
- [ ] Tracks now require auth
- [ ] Verified: register → login → JWT → `/api/tracks` works with Bearer, fails without
- [ ] Committed
- [ ] NOTES.md updated

## Day 4 — Playlists

- [ ] `Playlist` entity + `PlaylistTrack` join entity
- [ ] `PlaylistRepository`, `PlaylistTrackRepository`
- [ ] `PlaylistService`: create, list (current user), get (authz check), add track, remove track
- [ ] `PlaylistController`
- [ ] DTOs: `PlaylistDto`, `PlaylistDetailDto` (tracks embedded, ordered)
- [ ] Verified: full CRUD via curl + JWT
- [ ] Committed
- [ ] NOTES.md updated

## Day 5 — Kafka events

- [ ] Kafka producer config
- [ ] `PlayEvent` record
- [ ] `PlayEventProducer`
- [ ] `stream-url` publishes event before returning URL
- [ ] `@KafkaListener` writes to `play_events`
- [ ] Verified: row appears in `play_events` within 1-2 sec of hit
- [ ] Committed
- [ ] NOTES.md updated

## Day 6 — Frontend scaffold (separate repo `anasheed-web`)

- [ ] `create-next-app` done
- [ ] Tailwind + shadcn/ui wired
- [ ] `/signup`, `/login` pages
- [ ] `/tracks` browse page
- [ ] HTML5 audio player component
- [ ] API client with JWT in `Authorization`
- [ ] Verified: end-to-end signup → browse → play in browser
- [ ] Committed
- [ ] NOTES.md updated

## Day 7 — Playlists UI + shuffle

- [ ] `/playlists` page
- [ ] `/playlists/[id]` page
- [ ] "Add to playlist" button on track cards
- [ ] Persistent now-playing bar (React context or Zustand)
- [ ] Queue: play, pause, next, prev, shuffle (Fisher-Yates)
- [ ] Verified: create playlist → add tracks → shuffle cycles
- [ ] Committed
- [ ] NOTES.md updated

## Day 8 — Deploy

- [ ] Backend Docker image built
- [ ] Homelab `docker-compose.yml` updated
- [ ] Cloudflare Tunnel route for `api.anasheed.app`
- [ ] Frontend deployed to Vercel
- [ ] DNS: `anasheed.app` → Vercel, `api.anasheed.app` → tunnel → homelab
- [ ] Verified: anasheed.app loads, full flow works
- [ ] Committed
- [ ] NOTES.md updated

## Day 9-10 — Polish

- [ ] Bugs fixed
- [ ] `README.md` with mermaid architecture diagram
- [ ] GitHub repo public
- [ ] Mobile browser tested
- [ ] Resume bullet drafted
- [ ] Committed
- [ ] NOTES.md updated

---

## Change log (file-level work history)

_(append-only. One entry per completed unit of work — usually one per commit. Format below. Newest at bottom.)_

**Entry format:**

```
### YYYY-MM-DD — Day N, step "<step name>"
Commit: <SHA or "uncommitted">
Files:
- path/to/File.java:LN — Class.method() — what changed + why
- path/to/Other.java:LN-LN — added field `foo` — why
Verified: <how confirmed working — curl, test, manual>
Notes: <gotcha, follow-up, link to NOTES.md section if any>
```

**Rules:**
- Always include `file:line` for every change. Class + method name when applicable.
- "why" in one short clause. Not "what" — diff shows what.
- New file? Write `path/to/File.java:1 — new file — purpose`.
- Deleted? `path/to/File.java — deleted — reason`.
- Renamed/moved? `old → new — reason`.
- Migration? `changelog/NNN-xxx.xml — added table X / column Y`.
- Config? `application.yml:LN — set `key` — reason`.
- If many trivial files (formatting, imports), one line: `bulk: N files reformatted via <tool>`.

**Entries:**

### 2026-06-08 — Day 0, step "init scaffolding"
Commit: uncommitted
Files:
- CLAUDE.md:1 — new file — rules of engagement, locked stack/data/API/architecture, session resume protocol, PROGRESS.md update rule
- INIT_PROMPT.md:1 — new file — saved original init prompt for reference
- NOTES.md:1 — new file — user learning log (user writes)
- PROGRESS.md:1 — new file — single source of truth for session state
Verified: files exist, contents readable
Notes: no code yet, awaiting prereq verification

### 2026-06-11 — Day 1, step "Spring Initializr scaffold + pom deps"
Commit: uncommitted
Files:
- app/ — new dir — Maven project root from start.spring.io (Boot 4.1.0, Java 21, package `com.anasheed.app`)
- app/pom.xml:1 — new file — Initializr base (webmvc, jpa, security, validation, kafka, liquibase, postgres, lombok, devtools, sliced test starters)
- app/pom.xml:31-33 — added properties `jjwt.version=0.13.0`, `awssdk.version=2.46.8`, `springdoc.version=3.0.3` — single source of truth for dep versions
- app/pom.xml:35-46 — added `<dependencyManagement>` with AWS SDK v2 BOM import — version table for all AWS deps, scales when more AWS modules added
- app/pom.xml:117-133 — added jjwt-api (compile), jjwt-impl + jjwt-jackson (runtime) — JWT auth in Day 3, split keeps imports clean
- app/pom.xml:134-137 — added `software.amazon.awssdk:s3` (no version, BOM resolves) — signed URLs for DO Spaces in Day 2
- app/pom.xml:138-142 — added `springdoc-openapi-starter-webmvc-ui` 3.0.3 — OpenAPI/Swagger UI
- app/src/main/java/com/anasheed/app/AppApplication.java:1 — Initializr-generated main class
Verified: `mvn compile` BUILD SUCCESS in 41s
Notes: Boot version in Local env was stale `3.x`, corrected to `4.1.0`. Next: application.yml local/prod profiles. Springdoc 3.0.3 assumed Boot 4 / Jackson 3 compat fixed — verify at runtime.

---

## Decisions log

_(decisions made beyond CLAUDE.md lock — append-only)_

- 2026-06-08: project init. Repo created, CLAUDE.md + INIT_PROMPT.md + NOTES.md + PROGRESS.md scaffolded.

## Blockers

_(active blockers only. Resolved ones move to Day log.)_

- none

## Open questions for user

_(things Claude needs answered before progressing)_

- none
