# AI-Powered Language Learning Platform — Complete Implementation Plan

**Stack:** Java 21 · Spring Boot 3.x · Spring Security · Spring Data JPA · PostgreSQL · Redis · Spring AI · Flyway · Docker · WebSocket · Spring Scheduler

---

## 0. Domain Analysis

### 0.1 Core Domain Concepts

| Concept                  | Description                                                                                                        |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| **Learner**              | The user, has a native language, learns 1+ target languages, has a level per language                              |
| **Language Track**       | A (User, Language) pairing — a learner can study German and French independently, each with its own level/progress |
| **Placement**            | An AI-driven interview that assigns a CEFR level (A1–C2) to a new Language Track                                   |
| **Learning Path**        | An AI-generated weekly plan of activities (vocab/grammar/speaking/listening) tailored to level + goals             |
| **Content Unit**         | Vocabulary word, grammar topic, listening clip, or writing/speaking prompt — the atomic teachable thing            |
| **Attempt / Submission** | A learner's response to a content unit (quiz answer, spoken audio, written paragraph)                              |
| **Assessment**           | AI-generated feedback/score on an Attempt                                                                          |
| **SRS Card**             | A flashcard under spaced-repetition scheduling                                                                     |
| **Conversation Session** | A chat/roleplay/voice session between learner and AI persona                                                       |
| **Progress Snapshot**    | Weekly aggregate of skill scores (grammar/vocab/speaking/listening/writing/reading) used for the Weakness Analyzer |
| **Gamification Ledger**  | XP, coins, streak, achievements — an append-only ledger, same pattern you used for StockSavvy's stock ledger       |

### 0.2 Key Business Rules (as stated / inferred)

1. A learner selects a target language and either self-declares a level or takes an AI Placement Test — placement is optional but recommended.
2. A learner may run multiple Language Tracks concurrently (studying German and French at once), each with independent progress.
3. The Learning Path is regenerated when: placement completes, a Weekly Weakness Analysis runs, or the learner explicitly requests a re-plan.
4. Every AI-graded submission (pronunciation, writing, quiz) must persist both the raw submission and the AI's structured feedback — never just a score, for auditability and for "AI Explain Mistakes."
5. Streaks break if the Daily Goal isn't met by end-of-day in the learner's timezone (timezone must be stored per user).
6. SRS scheduling (flashcards) must survive independent of the Learning Path — it's a long-running, separate cadence (classic SM-2 or a simplified variant).
7. XP/Coins are never mutated directly — always append-only ledger entries with a reason code, mirroring the ledger pattern already used in StockSavvy for auditability and to prevent race conditions on balance updates.
8. Content (vocab lists, grammar explanations) can be **specialized by user domain** (e.g., "Backend Developer" → programming vocabulary) — this means content generation prompts must be parameterized by a `learner_interest_tags` field.

### 0.3 Ambiguities Requiring Your Decision Before Phase 4

|#|Ambiguity|Recommendation|
|---|---|---|
|1|Can a user learn multiple languages simultaneously, or one active track at a time?|Support multiple tracks from day one — it's a small modeling cost now (`language_track_id` FK everywhere) vs. a painful migration later.|
|2|Is Placement Test mandatory before any learning content unlocks?|No — let self-declared level unlock content immediately; placement is an optional "confirm your level" flow. Reduces onboarding friction.|
|3|Do AI feature costs (Whisper, TTS, LLM calls) need per-user rate limiting / quota?|Yes — add a `daily_ai_quota` per user from Phase 4 onward, enforced via Redis counters, or AI API costs will be unbounded.|
|4|Is real-time voice conversation (Phase 2) actually WebSocket audio streaming, or upload-then-transcribe?|Recommend starting with **upload-then-transcribe** (record clip → POST → STT → LLM → TTS → return audio URL). True streaming STT/TTS is a significant scope increase; treat it as a Phase 2.5 stretch goal, not MVP-of-Phase-2.|

---

## 1. Architecture Overview

### 1.1 Layering (per feature)

```
feature/
├── domain/           # Entities, Value Objects, domain services, domain events — ZERO framework deps
├── application/       # Commands, Queries, Handlers, DTOs, port interfaces (repositories, AI clients)
├── infrastructure/    # JPA entities/repos, external AI client impls, Redis, mappers
└── api/                # REST controllers, request/response DTOs, exception mapping
```

- **Domain** depends on nothing.
- **Application** depends only on **Domain** (defines repository/port _interfaces_ here — Dependency Inversion).
- **Infrastructure** depends on **Application** (implements the ports) and **Domain**.
- **API** depends on **Application** (calls handlers), never directly on Infrastructure.

### 1.2 Dependency Matrix

|From \ To|Domain|Application|Infrastructure|API|
|---|---|---|---|---|
|**Domain**|—|❌|❌|❌|
|**Application**|✅|—|❌ (interfaces only)|❌|
|**Infrastructure**|✅|✅ (implements ports)|—|❌|
|**API**|❌ (only via DTOs)|✅|❌|—|

**Dependency Inversion in practice:** `application/port/VocabularyRepository` is an interface. `infrastructure/persistence/JpaVocabularyRepository` implements it and is wired via Spring's `@Repository` + constructor injection. The Application layer never imports `org.springframework.data.jpa.*`. This is what lets you swap Postgres+JPA for something else later without touching business logic — and it's what makes Application-layer unit tests fast (mock the port, no Spring context needed).

### 1.3 Cross-Cutting Concerns (shared module, not a feature)

```
shared/
├── security/        # JWT filter, SecurityConfig, method-security annotations
├── exception/       # GlobalExceptionHandler (@ControllerAdvice), domain exception hierarchy
├── validation/      # Custom validators (e.g., CEFR level enum validator)
├── audit/           # Auditable base entity (createdAt/updatedAt/createdBy), JPA auditing config
├── ai/              # Spring AI ChatClient config, prompt templates, AI response parsing helpers
├── cache/           # Redis config, cache key conventions
├── event/           # Domain event publisher (Spring's ApplicationEventPublisher wrapper)
└── web/             # ApiResponse envelope, pagination DTOs, correlation-id filter
```

### 1.4 CQRS — Chosen Pattern

**Decision: No generic CommandBus/QueryBus framework.** Each use case gets one `XxxCommand`/`XxxQuery` record + one `XxxHandler` class, injected directly into the controller that needs it. This is what you already validated works well for you in Spring Boot without MediatR/Axon.

Why not a bus abstraction: with a bus you gain runtime dispatch + easy cross-cutting decorators (logging/validation pipelines), but you lose compile-time safety and add indirection for zero real benefit in a project with one team member and no plugin/extensibility requirement. If you later want pipeline behaviors (e.g., auto-validation, auto-logging on every command), add a **method-level pattern**: wrap handlers with a `@Around` AOP aspect matching a `CommandHandler` marker interface — you get the cross-cutting benefit without a bus.

```
Command/Query (record, immutable)
    ↓
Handler (single public `handle(...)` method, constructor-injected ports)
    ↓
Validation (Jakarta Bean Validation on the Command/Query itself, or a dedicated Validator for cross-field rules)
    ↓
Domain/Application logic (entity methods, domain services)
    ↓
Repository port → Infrastructure impl
    ↓
Response DTO (never return JPA entities across the API boundary)
```

**Command vs Query rule:** if it mutates state → Command (returns only an ID/void/minimal confirmation DTO). If it only reads → Query (returns a rich read DTO, may bypass the domain model entirely and use a projection/native query for performance — this is where CQRS actually pays off: queries don't have to go through the aggregate).

---

## 2. Domain Model

### 2.1 Aggregates & Entities (MVP scope)

- **User** (aggregate root) — id, email, passwordHash (nullable if OAuth-only), displayName, nativeLanguage, timezone, emailVerified, authProvider (LOCAL/GOOGLE), createdAt
- **LanguageTrack** (aggregate root, references User) — id, userId, targetLanguage, currentLevel (CEFR enum), status (ACTIVE/PAUSED), startedAt
- **PlacementTest** — id, languageTrackId, status (IN_PROGRESS/COMPLETED), transcript (JSON of Q&A), resultLevel, completedAt
- **LearningPath** — id, languageTrackId, weekNumber, generatedAt, status (ACTIVE/SUPERSEDED)
- **LearningPathItem** (VO-ish, owned by LearningPath) — id, learningPathId, dayIndex, contentType (VOCAB/GRAMMAR/SPEAKING/LISTENING), contentUnitId, completed
- **VocabularyItem** — id, languageTrack-agnostic (shared content pool), word, translation, ipa/pronunciation, exampleSentence, level, domainTag (nullable, e.g. "backend")
- **GrammarTopic** — id, name, level, explanationTemplate
- **SrsCard** — id, userId, vocabularyItemId, easeFactor, intervalDays, dueAt, repetitions (SM-2 fields)
- **Submission** (base) — id, userId, type (WRITING/SPEAKING/QUIZ), rawContentRef (text or audio URL), submittedAt
- **Assessment** — id, submissionId, score, feedbackJson, aiModel, createdAt
- **DailyGoal** — id, userId, targetMinutes, targetWords, targetLessons
- **DailyProgress** — id, userId, date, minutesSpent, wordsLearned, lessonsCompleted, goalMet
- **StreakLedgerEntry** (append-only, ledger pattern) — id, userId, date, delta, reasonCode
- **XpLedgerEntry** (append-only) — id, userId, amount, reasonCode, refType, refId, createdAt

### 2.2 Value Objects

- `CefrLevel` (enum: A1..C2) — used everywhere instead of raw strings, with a `compareTo`-friendly ordinal.
- `Email` — validated wrapper, prevents primitive obsession in domain logic.
- `AudioRef` — { storageKey, durationSeconds, mimeType } — avoids passing raw strings for audio pointers.
- `Score` — { value: 0–100 } with invariant enforcement (can't be constructed out of range).

### 2.3 Domain Services (only where logic doesn't naturally belong to one entity)

- `SrsScheduler` — computes next `dueAt`/`easeFactor` from a review grade (SM-2 algorithm) — spans no single entity's responsibility.
- `StreakCalculator` — given today's DailyProgress + yesterday's streak state, decides increment/reset — needs "today in user's timezone" context, doesn't belong on the User entity itself.
- `LevelProgressionPolicy` — decides if a Weakness Analysis result should trigger a level change on LanguageTrack.

---

## 3. Database Design

> All tables: `id BIGSERIAL PK`, `created_at TIMESTAMPTZ NOT NULL DEFAULT now()`, `updated_at TIMESTAMPTZ`, soft-delete via `deleted_at TIMESTAMPTZ NULL` only on `users`, `language_tracks`, `submissions` (things a user might want "removed"); everything else is hard-deletable/immutable-append.

|Table|Key Columns|Notes|
|---|---|---|
|`users`|email (UNIQUE, NOT NULL), password_hash (NULL), display_name, native_language, timezone, email_verified BOOLEAN DEFAULT false, auth_provider ENUM, google_sub (NULLABLE UNIQUE)|Index on `email`, `google_sub`|
|`language_tracks`|user_id FK→users, target_language, current_level ENUM, status ENUM, started_at|UNIQUE (user_id, target_language) WHERE deleted_at IS NULL|
|`placement_tests`|language_track_id FK, status ENUM, transcript JSONB, result_level ENUM NULL, completed_at NULL|Index on language_track_id|
|`learning_paths`|language_track_id FK, week_number INT, status ENUM, generated_at|Index (language_track_id, status) — fast "get active path" lookup|
|`learning_path_items`|learning_path_id FK, day_index SMALLINT, content_type ENUM, content_unit_id BIGINT, completed BOOLEAN DEFAULT false|Composite index (learning_path_id, day_index)|
|`vocabulary_items`|word, translation, ipa, example_sentence, level ENUM, domain_tag NULLABLE, language|Index (language, level), GIN index on domain_tag if array|
|`grammar_topics`|name, level ENUM, explanation_template TEXT, language|Index (language, level)|
|`srs_cards`|user_id FK, vocabulary_item_id FK, ease_factor NUMERIC(4,2), interval_days INT, due_at TIMESTAMPTZ, repetitions INT|Index (user_id, due_at) — the hot query: "cards due today"|
|`submissions`|user_id FK, type ENUM, raw_content_ref TEXT, submitted_at|Index (user_id, submitted_at DESC)|
|`assessments`|submission_id FK UNIQUE, score SMALLINT, feedback_json JSONB, ai_model VARCHAR, created_at|1:1 with submission|
|`daily_goals`|user_id FK UNIQUE, target_minutes, target_words, target_lessons||
|`daily_progress`|user_id FK, date DATE, minutes_spent, words_learned, lessons_completed, goal_met BOOLEAN|UNIQUE (user_id, date)|
|`streak_ledger`|user_id FK, date DATE, delta SMALLINT, reason_code VARCHAR|Index (user_id, date DESC)|
|`xp_ledger`|user_id FK, amount INT, reason_code VARCHAR, ref_type VARCHAR, ref_id BIGINT, created_at|Index (user_id, created_at DESC) — balance = SUM(amount), never stored denormalized without a reconciliation job|

### 3.1 N+1 / Performance Risks Identified

1. **Dashboard endpoint** (Section 8's "Vocabulary/Grammar/Listening/Speaking/Writing/Reading progress") will naturally want to join across `daily_progress`, `xp_ledger`, `srs_cards`, `assessments` — **do not** lazy-load these via JPA relationships one-by-one. Use a dedicated read-model query (native SQL or JPQL projection) per dashboard section, executed in parallel or as one UNION query. This is a textbook CQRS "query side" case: a hand-tuned read query, no aggregate loading.
2. **`srs_cards` due-today query** — must hit the `(user_id, due_at)` index directly; never load all of a user's cards into memory to filter in Java.
3. **`learning_path_items` → content_unit** — `content_unit_id` is polymorphic (vocab or grammar), so it's _not_ a JPA `@ManyToOne` — resolve it in the application layer by `content_type`, avoiding a fake shared-supertype JPA mapping that would force eager fetch complexity. Use `@EntityGraph` only on the genuinely single-typed relations (e.g., `LearningPath.items`).
4. **XP balance** — never `SELECT SUM(amount)` on every page load for a "leaderboard" of many users; maintain a Redis sorted set (`ZINCRBY`) updated on each ledger write, and treat the Postgres ledger as source of truth reconciled nightly, not the live read path.

---

## 4. Feature Breakdown — Phase 1 (MVP)

> Format per feature: Purpose · Actors · Commands/Queries · Domain entities · Business rules · Key endpoints · Edge cases · Core tests.

### 4.1 Authentication

- **Purpose:** register, login, verify email, Google OAuth, issue/refresh JWT.
- **Actors:** Anonymous, RegisteredUser.
- **Commands:** `RegisterUserCommand`, `VerifyEmailCommand`, `LoginCommand`, `RefreshTokenCommand`, `GoogleLoginCommand`.
- **Queries:** `GetCurrentUserQuery`.
- **Entities:** User.
- **Business rules:** email unique; password min complexity; unverified users can log in but see a "verify your email" gate on content endpoints (via a filter, not per-handler checks); Google login auto-verifies email.
- **Endpoints:** `POST /api/v1/auth/register`, `POST /api/v1/auth/verify-email`, `POST /api/v1/auth/login`, `POST /api/v1/auth/refresh`, `POST /api/v1/auth/google`, `GET /api/v1/users/me`.
- **Edge cases:** duplicate registration race (unique constraint, not just a pre-check query), expired verification token, Google account email already registered via LOCAL (link vs reject — recommend: reject with a clear "sign in with password" error), refresh token reuse detection.
- **Tests:** unit (password hashing, token expiry logic), handler tests (duplicate email → 409), integration (full register→verify→login flow via Testcontainers Postgres).

### 4.2 Language Track Selection & Onboarding

- **Purpose:** pick target language + declare/confirm level.
- **Commands:** `CreateLanguageTrackCommand`, `SetSelfDeclaredLevelCommand`.
- **Queries:** `GetUserLanguageTracksQuery`.
- **Business rules:** one active track per (user, language); level must be a valid CEFR value.
- **Endpoints:** `POST /api/v1/language-tracks`, `GET /api/v1/language-tracks`.
- **Edge cases:** re-adding a paused language should reactivate, not duplicate.

### 4.3 AI Placement Test

- **Purpose:** conversational interview → CEFR level.
- **Commands:** `StartPlacementTestCommand`, `SubmitPlacementAnswerCommand`.
- **Queries:** `GetPlacementResultQuery`.
- **Entities:** PlacementTest.
- **Business rules:** fixed max question count (e.g., 6) before forcing a result; AI must return a structured level, not free text — enforce via Spring AI's structured output / JSON schema response.
- **Endpoints:** `POST /api/v1/placement-tests`, `POST /api/v1/placement-tests/{id}/answers`, `GET /api/v1/placement-tests/{id}`.
- **Edge cases:** AI returns malformed JSON → retry once, then fall back to self-declared level; user abandons mid-test → track stays at self-declared level.
- **Tests:** mock the AI port entirely for handler tests; one live-AI integration test behind a `@Tag("ai-integration")` excluded from default CI runs (costs money/is flaky).

### 4.4 AI Personalized Learning Path

- **Purpose:** weekly plan generation.
- **Commands:** `GenerateLearningPathCommand`.
- **Queries:** `GetActiveLearningPathQuery`.
- **Business rules:** generation is idempotent per (track, week) — regenerating supersedes, never duplicates; must pull from existing `vocabulary_items`/`grammar_topics` pools filtered by level+domain_tag, with AI selecting/sequencing rather than inventing content from scratch (keeps content quality controllable and reusable across users — cheaper and more consistent than per-user generation).
- **Endpoints:** `POST /api/v1/learning-paths/generate`, `GET /api/v1/learning-paths/active`.
- **Edge cases:** insufficient content pool for a level/language → fall back to a curated default path instead of failing.

### 4.5 Vocabulary

- **Commands:** `MarkVocabularyLearnedCommand`.
- **Queries:** `GetVocabularyItemQuery`, `ListVocabularyByLevelQuery`.
- **Business rules:** marking learned creates a `SrsCard`.
- **Endpoints:** `GET /api/v1/vocabulary/{id}`, `GET /api/v1/vocabulary?language=&level=`, `POST /api/v1/vocabulary/{id}/learned`.

### 4.6 Grammar

- **Queries:** `GetGrammarExplanationQuery` (AI-personalized rendering of `explanation_template` for the learner's level).
- **Endpoints:** `GET /api/v1/grammar/{topicId}?style=default|eli5|arabic|football|programming` — reuses the same AI-explanation-style idea from Phase 2's "AI Teacher," so build the prompt-templating utility once and share it.

### 4.7 Listening

- **Commands:** `SubmitListeningAnswerCommand`.
- **Entities:** Submission (type=QUIZ tied to a listening clip), Assessment.
- **Endpoints:** `GET /api/v1/listening/{clipId}`, `POST /api/v1/listening/{clipId}/answers`.

### 4.8 Speaking

- **Commands:** `SubmitSpeechCommand` (uploads audio ref).
- **Business rules:** async pipeline — STT → pronunciation scoring → Assessment; endpoint returns 202 + submission id, client polls or gets a WebSocket push when ready (reuse your existing NestJS/StockSavvy WebSocket notification pattern here, just in Spring with a `SimpMessagingTemplate` or plain `WebSocketHandler`).
- **Endpoints:** `POST /api/v1/speaking/submissions`, `GET /api/v1/speaking/submissions/{id}`.
- **Edge cases:** audio too short/silent → reject before calling paid STT API; STT low-confidence → mark assessment `LOW_CONFIDENCE` rather than a misleading score.

### 4.9 Writing

- **Commands:** `SubmitWritingCommand`.
- **Business rules:** AI returns structured `{ corrections[], naturalVersion, formalVersion, nativeVersion }` — enforce via structured output schema, not prompt-and-hope.
- **Endpoints:** `POST /api/v1/writing/submissions`, `GET /api/v1/writing/submissions/{id}`.

### 4.10 Flashcards (SRS)

- **Commands:** `ReviewSrsCardCommand` (grade: again/hard/good/easy).
- **Queries:** `GetDueSrsCardsQuery`.
- **Domain service:** `SrsScheduler` (SM-2).
- **Endpoints:** `GET /api/v1/srs/due`, `POST /api/v1/srs/{cardId}/review`.

### 4.11 Daily Goal & Progress / Gamification base

- **Commands:** `SetDailyGoalCommand`, `RecordActivityCommand` (called internally by other features, not directly by client, on every completed action).
- **Queries:** `GetDailyProgressQuery`, `GetStreakQuery`.
- **Endpoints:** `PUT /api/v1/goals/daily`, `GET /api/v1/progress/today`, `GET /api/v1/progress/streak`.
- **Edge cases:** timezone-boundary activity (learner active at 11:58pm) must count for the correct local date — always compute "today" from `users.timezone`, never server UTC date.

---

## 5. Feature Breakdown — Phase 2 (AI Depth)

Only headline design decisions (full CQRS breakdown mirrors Phase 1's pattern):

|Feature|Key design decision|
|---|---|
|AI Teacher|Single `GET /api/v1/grammar/{id}?style=` reused (see 4.6) — no separate feature needed, just a style parameter threaded through the prompt template.|
|AI Conversation Partner|New `ConversationSession` aggregate; `POST /api/v1/conversations`, `POST /api/v1/conversations/{id}/messages`; store full transcript for the AI Memory feature to mine later.|
|AI Role Play|Same `ConversationSession` with a `scenario` field (AIRPORT/RESTAURANT/JOB_INTERVIEW/HOTEL) that selects a system prompt persona.|
|AI Pronunciation Coach|Extends the Speaking Assessment feedback_json shape — not a new feature, just a richer AI response schema per submission.|
|AI Vocabulary Generator|`POST /api/v1/vocabulary/generate?domain=backend` — writes into the shared `vocabulary_items` pool with `domain_tag` set, reusable across users (cache-friendly, avoids regenerating "Repository/Controller/Docker" for every backend-dev learner).|
|AI Story Generator|`POST /api/v1/stories/generate` — takes a list of vocabulary_item_ids the learner is studying, returns a short story; store as content, not regenerate every view.|
|AI Quiz Generator|`POST /api/v1/quizzes/generate?topicId=` — generated quiz cached per (topic, level) for a TTL, not literally unique per request, to control AI spend.|
|AI Explain Mistakes|Enrich `Assessment.feedback_json` with a `ruleExplanation` field — same entity, better prompt, not a new table.|
|AI Memory (recurring mistakes)|New `MistakePattern` table: (user_id, category, example, frequency, last_seen) — updated by a post-Assessment event listener, read by the Weakness Analyzer and by prompt-building (inject "known weak areas" into every generation prompt).|
|AI Weakness Analyzer|Scheduled job (`@Scheduled` weekly) aggregating `daily_progress` + `assessments` + `mistake_patterns` into a `ProgressSnapshot`; triggers `GenerateLearningPathCommand` if thresholds crossed.|
|AI Daily Conversation|Scheduler seeds one `ConversationSession` per user per day with a rotating scenario list — a simple cron + round-robin, no AI needed for the _scheduling_ itself, only for the conversation content.|
|AI Translate Context|Stateless `POST /api/v1/translate?word=&context=` — no persistence needed beyond an optional cache.|
|Gamification (XP/Levels/Streak/Leaderboard/Achievements/Badges)|XP/Streak as described in 4.11; `Achievement` is a rules table evaluated by an event listener on `xp_ledger`/`streak_ledger` writes (event-driven, not a polling job). Leaderboard via Redis sorted set as noted in §3.1.|
|Social (Friends/Challenges/Weekly competitions)|Standard `Friendship` (requester/addressee/status) + `Challenge` (participants, metric, window) tables — lowest technical risk in the whole plan, build last.|

---

## 6. API Design Conventions

- All endpoints versioned under `/api/v1/`.
- Response envelope: `{ data, meta }` for success; `{ error: { code, message, details[] } }` for failures — handled centrally by `GlobalExceptionHandler`, never hand-rolled per controller.
- Auth: `Authorization: Bearer <jwt>` except `/auth/*`.
- Pagination: `?page=&size=`, capped `size` server-side (max 100) to prevent abuse.
- Every mutating endpoint returns `201`/`200` with the created/updated resource's id at minimum; async endpoints (speaking) return `202` + a poll/status URL.
- Standard error codes: `400` validation, `401` unauthenticated, `403` unauthorized, `404` not found, `409` conflict (duplicate email, duplicate track), `422` business rule violation (distinct from plain validation), `429` AI quota exceeded, `500` unhandled.

---

## 7. Caching Strategy (Redis)

|Cache|Key pattern|TTL|Invalidation|
|---|---|---|---|
|JWT blacklist (logout)|`jwt:blacklist:{jti}`|until token expiry|none needed (self-expiring)|
|Active learning path|`path:active:{trackId}`|1h|evict on `GenerateLearningPathCommand`|
|Vocabulary by level|`vocab:{lang}:{level}`|6h|evict on new vocab insert for that (lang, level)|
|AI daily quota counter|`ai-quota:{userId}:{date}`|24h (natural expiry)|none — counter|
|Generated quiz|`quiz:{topicId}:{level}`|24h|none — natural expiry is fine for quiz variety|
|Leaderboard|Redis sorted set `leaderboard:xp:global`|n/a (live)|`ZINCRBY` on every xp_ledger write|

---

## 8. Background Jobs / Scheduling

- **Weekly Weakness Analyzer** — `@Scheduled(cron)` per-user batch job (chunked, not all-users-in-one-transaction).
- **Daily Conversation Seeder** — `@Scheduled` daily cron.
- **Streak Reset Check** — `@Scheduled` daily, per-timezone-bucketed (run hourly, process users whose local midnight just passed — don't run one global midnight job for all timezones).
- **SRS Due-Reminder Notification** — `@Scheduled` + WebSocket/push, off the `srs_cards.due_at` index.
- Use Spring's built-in `@Scheduled` + `ShedLock` (or a Postgres advisory lock) once you deploy more than one instance, to prevent duplicate job runs — flag this explicitly as a Phase 12/13 concern, not needed for a single-instance MVP deploy.

---

## 9. Testing Strategy

|Layer|Tool|What|
|---|---|---|
|Domain|JUnit 5, no Spring context|Entity invariants (e.g., `Score` can't be constructed >100), `SrsScheduler` algorithm correctness, `StreakCalculator` timezone edge cases|
|Application/Handlers|JUnit 5 + Mockito|Mock repository ports + AI ports; verify handler logic, validation rejection paths, idempotency (e.g., regenerating a path supersedes not duplicates)|
|Infrastructure/Repository|Testcontainers (real Postgres)|Query correctness, index-dependent queries (SRS due query, dashboard projections), migration correctness|
|API|`@SpringBootTest` + MockMvc/WebTestClient|Auth flow end-to-end, status codes, validation error shape, authorization (user A can't read user B's progress)|
|AI integration|Tagged `@Tag("ai-integration")`, excluded from default `mvn test`|Real Spring AI calls against a cheap model, run manually/nightly, not on every commit|
|Contract|JSON schema assertions on AI structured outputs|Placement result, writing feedback, assessment feedback all validated against a schema so a model/prompt change that breaks structure fails CI immediately|

**Priority tests per feature (non-negotiable minimum):**

- Auth: duplicate registration, wrong password, expired refresh token, Google-email-collision.
- Placement: malformed AI response fallback, max-question cutoff.
- Learning path: idempotent regeneration.
- SRS: due-date math after each grade, ease-factor floor (never below minimum per SM-2).
- XP/Streak ledgers: concurrent writes don't lose entries (this is exactly the race condition ledger patterns exist to prevent — write a concurrency test).
- Authorization: every "get my X" endpoint tested against a second user's data.

---

## 10. Implementation Phases (Dependency-Ordered)

|Phase|Focus|Depends on|Complexity|
|---|---|---|---|
|0|Domain analysis (this document)|—|—|
|1|Solution structure, shared module skeleton, Docker Compose (Postgres+Redis), Flyway baseline|0|Low|
|2|Domain model: User, LanguageTrack + Value Objects, unit tests|1|Low|
|3|Infrastructure foundation: JPA config, auditing, base repository ports, GlobalExceptionHandler, ApiResponse envelope|2|Low|
|4|Authentication & Authorization (full JWT + Google OAuth + Spring Security config)|3|Medium|
|5|Language Track selection + Vocabulary/Grammar read features (no AI yet — static content CRUD)|4|Low|
|6|Spring AI integration foundation: ChatClient config, structured-output helper, prompt template loader, AI quota guard|5|Medium|
|7|Placement Test + Learning Path generation (first real AI features)|6|High|
|8|Listening, Writing, Speaking (incl. async submission pipeline + WebSocket notifications)|7|High|
|9|SRS Flashcards + Daily Goal/Progress + Streak/XP ledgers|5 (parallel to 6–8 if solo, but sequence after 8 to avoid context switching)|Medium|
|10|Caching layer (Redis) applied across 5–9|9|Low|
|11|Background jobs: Weakness Analyzer, streak reset, SRS reminders|9, 6|Medium|
|12|Phase 2 AI features: Conversation/Roleplay, AI Memory, Vocabulary/Story/Quiz generators|11|High|
|13|Gamification (Achievements/Badges/Leaderboard) + Social|9, 10|Medium|
|14|Testing hardening pass (fill coverage gaps from every prior phase)|all|Medium|
|15|Docker production compose, CI pipeline, logging/observability (structured logs, correlation-id, actuator)|14|Medium|
|16|Security hardening (rate limiting, quota enforcement audit, dependency scan)|15|Medium|
|17|Performance pass (verify indexes under load, N+1 audit from §3.1)|16|Medium|
|18|API documentation (springdoc-openapi) + README + architecture diagram|17|Low|

**Why this order:** AI plumbing (Phase 6) is built once, deliberately before the first AI feature (Placement, Phase 7), rather than ad hoc inside each feature — every subsequent AI feature (Roleplay, Story Generator, Weakness Analyzer) reuses the same ChatClient config, structured-output parsing, and quota guard. Building it inside Placement first and retrofitting later is the single biggest rework risk in this plan if skipped.

---

## 11. Final Project Structure

```
language-platform/
├── docker-compose.yml
├── pom.xml (or Gradle multi-module — recommend single Maven module with feature packages
│            unless you specifically want compiler-enforced boundaries via Gradle modules;
│            for a solo dev, package-by-feature + ArchUnit tests enforcing the dependency
│            matrix from §1.2 gives 90% of the benefit with far less build complexity)
├── src/main/java/com/yourorg/langplatform/
│   ├── shared/                     # §1.3 cross-cutting concerns
│   ├── auth/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   └── api/
│   ├── languagetrack/
│   │   ├── domain/ application/ infrastructure/ api/
│   ├── placement/
│   ├── learningpath/
│   ├── vocabulary/
│   ├── grammar/
│   ├── listening/
│   ├── speaking/
│   ├── writing/
│   ├── srs/
│   ├── progress/           # daily goal, daily progress, streak
│   ├── conversation/        # Phase 2: chat, roleplay
│   ├── mistakepattern/      # Phase 2: AI memory
│   ├── weaknessanalyzer/    # Phase 2: scheduled analysis
│   ├── contentgen/          # Phase 2: vocab/story/quiz generators
│   ├── gamification/        # xp, streak ledger, achievements, leaderboard
│   └── social/              # friends, challenges
├── src/main/resources/
│   ├── db/migration/         # Flyway V1__..., V2__...
│   ├── application.yml
│   └── prompts/              # AI prompt templates as resource files, not inline strings
└── src/test/java/...          # mirrors main structure
```

**ArchUnit** (add in Phase 1): write one test class asserting the dependency matrix in §1.2 — e.g., `noClasses().that().resideInAPackage("..domain..").should().dependOnClassesThat().resideInAPackage("..infrastructure..")`. This turns your architecture rules into something CI enforces, not just something you remember.

---

## 12. Master Implementation Checklist

**Phase 1 — Foundation**

- [ ] Init Maven project, add Spring Boot Web/Data JPA/Security/Validation/Actuator starters
- [ ] `docker-compose.yml`: Postgres, Redis, app service
- [ ] Flyway baseline migration setup, `application.yml` profiles (local/docker/prod)
- [ ] `shared/` package: `ApiResponse`, `GlobalExceptionHandler`, base domain exception hierarchy
- [ ] `shared/audit`: `Auditable` base entity + JPA auditing config
- [ ] ArchUnit dependency-rule test

**Phase 2 — Domain Model**

- [ ] `User` entity + `Email`, `CefrLevel` value objects + unit tests
- [ ] `LanguageTrack` entity + unit tests
- [ ] Domain exception classes (e.g., `DuplicateLanguageTrackException`)

**Phase 3 — Infra Foundation**

- [ ] JPA entity mappings for User/LanguageTrack, first Flyway migration (`V1__init_users_and_tracks.sql`)
- [ ] Repository ports (`application/port`) + JPA implementations (`infrastructure/persistence`)
- [ ] Verify GlobalExceptionHandler maps domain exceptions → correct HTTP codes

**Phase 4 — Auth**

- [ ] `RegisterUserCommand` + handler + validation + `POST /auth/register`
- [ ] Email verification token entity/flow + `POST /auth/verify-email`
- [ ] `LoginCommand` + JWT issuance + `POST /auth/login`
- [ ] Refresh token flow + reuse detection + `POST /auth/refresh`
- [ ] Google OAuth2 login + account-collision handling + `POST /auth/google`
- [ ] `SecurityConfig`: JWT filter chain, public vs protected routes
- [ ] `GET /users/me`
- [ ] Integration tests: full register→verify→login, duplicate email race, expired token

**Phase 5 — Track Selection + Static Content**

- [ ] `CreateLanguageTrackCommand`/handler + unique-active-track rule + endpoint
- [ ] `vocabulary_items`/`grammar_topics` migrations + seed data
- [ ] List/get vocabulary & grammar queries + endpoints

**Phase 6 — AI Foundation**

- [ ] Spring AI `ChatClient` bean config (model, API key from env)
- [ ] Structured-output helper (JSON schema → typed record) + unit tests with mocked responses
- [ ] Prompt template loader (resources/prompts/*.st or similar)
- [ ] `ai-quota` Redis guard (per-user daily counter, 429 on exceed)

**Phase 7 — Placement + Learning Path**

- [ ] `PlacementTest` entity + migration
- [ ] Start/answer/get placement endpoints + AI-call handler + malformed-response fallback
- [ ] `LearningPath`/`LearningPathItem` entities + migration
- [ ] `GenerateLearningPathCommand` (idempotent per week) + endpoint
- [ ] Tests: idempotency, AI fallback behavior

**Phase 8 — Listening/Writing/Speaking**

- [ ] `Submission`/`Assessment` entities + migration
- [ ] Listening clip + answer submission + AI grading
- [ ] Writing submission + structured feedback (natural/formal/native versions)
- [ ] Speaking: audio upload endpoint, async STT+scoring pipeline, WebSocket push on completion
- [ ] Tests: silent-audio rejection, low-confidence handling

**Phase 9 — SRS + Progress**

- [ ] `SrsCard` entity + migration + `SrsScheduler` (SM-2) + unit tests
- [ ] Due-cards query (index-backed) + review command + endpoints
- [ ] `DailyGoal`/`DailyProgress` entities + endpoints
- [ ] `StreakLedger`/`XpLedger` entities + `StreakCalculator` (timezone-aware) + concurrency test

**Phase 10 — Caching**

- [ ] Redis config bean
- [ ] Cache-aside for active learning path, vocabulary lists, generated quizzes
- [ ] Cache eviction on relevant writes (verify with tests, not just manually)

**Phase 11 — Scheduled Jobs**

- [ ] Weekly Weakness Analyzer job + `ProgressSnapshot` entity
- [ ] Daily Conversation seeder job
- [ ] Timezone-bucketed streak-reset job
- [ ] SRS due-reminder job

**Phase 12 — Phase 2 AI Features**

- [ ] `ConversationSession` entity + chat/roleplay endpoints
- [ ] `MistakePattern` entity + event listener on Assessment creation
- [ ] Vocabulary/Story/Quiz generator endpoints (cache per §7 table)
- [ ] AI Translate Context endpoint

**Phase 13 — Gamification + Social**

- [ ] `Achievement` rules table + event-listener evaluator
- [ ] Redis leaderboard sorted set + endpoint
- [ ] `Friendship`/`Challenge` entities + endpoints

**Phase 14 — Testing Hardening**

- [ ] Coverage audit against §9 priority-test list; fill gaps
- [ ] Authorization cross-user tests for every "my data" endpoint

**Phase 15 — Docker/CI**

- [ ] Production `Dockerfile` (multi-stage build)
- [ ] CI pipeline: build, unit+handler+repo tests, ArchUnit, exclude `ai-integration` tag
- [ ] Structured JSON logging + correlation-id filter + Actuator health/metrics

**Phase 16 — Security Hardening**

- [ ] Global rate limiting (per-IP + per-user)
- [ ] Confirm AI quota enforced on every AI-calling endpoint, not just placement
- [ ] Dependency vulnerability scan (OWASP dependency-check)

**Phase 17 — Performance**

- [ ] EXPLAIN ANALYZE on the §3.1 hot queries under seeded load
- [ ] Confirm no N+1 in dashboard endpoint via Hibernate statistics logging in a test

**Phase 18 — Documentation**

- [ ] springdoc-openapi wired, all endpoints annotated
- [ ] README with architecture diagram (mirrors §1.1/§11)

---

## 13. Risks Summary

- **AI cost risk** — uncontrolled without the Phase 6 quota guard; build it before any AI feature, not after.
- **Scope risk** — Phase 2's full feature list (12+ AI features) is CV-impressive but large; if time-constrained, ship Phase 1 fully polished (tested, documented, deployed) before touching Phase 2 — a complete MVP beats a half-built feature-rich app in an interview.
- **Real-time voice** — treat true streaming STT/TTS as explicitly out of scope for the plan above (see §0.3, ambiguity 4); revisit only after Phase 1 ships.
- **Single-instance scheduling** — `@Scheduled` jobs will double-run if you ever deploy 2+ instances; flagged in §8, add ShedLock before any horizontal scaling.