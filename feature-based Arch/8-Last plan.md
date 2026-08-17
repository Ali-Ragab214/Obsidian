# AI-Powered Language Learning Platform

## Production-Grade Architecture & Dependency-Ordered Implementation Plan

**Source of truth:** `6-AI-Powered_Language_Learning_Platform__Complete_Implementation_Plan.pdf` **Stack:** Java 21 · Spring Boot 3.x · Spring Security · Spring Data JPA · PostgreSQL · Redis · Spring AI · Flyway · Docker · WebSocket · Spring Scheduler **Style:** شرح بالعربي + كل الأسماء التقنية (classes, packages, endpoints, SQL, Mermaid) بالإنجليزي زي ما طلبت.

---

## 1. Executive Summary

المستند ده تحويل مباشر للـ PDF اللي بعتهولي لخطة تنفيذ (implementation-ready) بحيث تقدر تبدأ من Phase 1 وتمشي خطوة خطوة من غير ما تحتاج تسأل "الخطوة الجاية إيه؟".

المبادئ اللي اتبنى عليها كل قرار هنا:

1. **PDF هو الـ source of truth** — كل entity وكل business rule وكل endpoint مذكور في الـ PDF اتنقل هنا زي ما هو، وأي تعديل أو إضافة ليا بيتحدد صراحة كـ **ARCHITECTURAL RECOMMENDATION**.
2. **Feature-based Clean Architecture + CQRS** بدون أي generic bus.
3. **AI معزول تمامًا خلف ports** — الـ Domain والـ Application متعرفش أصلاً إن فيه Spring AI أو ChatClient.
4. **AI مش أول حاجة بتتبني** — الأساس (auth, domain, db, security, static content) الأول، بعدين AI infra (Phase 6 في خطة الـ PDF)، بعدين أول AI feature حقيقي.
5. **Ledger pattern** لـ XP و Streak (نفس اللي استخدمته في StockSavvy) — append-only، مفيش mutation مباشر على balance.
6. **الخطة دي لمطور واحد (solo developer)** — الترتيب اتصمم عشان يقلل الـ context switching، مش بس عشان يكون "correct" نظريًا.

هذا المستند طويل جدًا بشكل متعمد — كل قسم فيه هو حاجة هتحتاجها وانت بتكتب كود فعلي.

---

## 2. Requirements Extracted From PDF (Traceability)

### 2.1 Core Domain Concepts (من PDF §0.1)

- Learner (User), Language Track, Placement, Learning Path, Content Unit, Attempt/Submission, Assessment, SRS Card, Conversation Session, Progress Snapshot, Gamification Ledger.

### 2.2 Key Business Rules (من PDF §0.2 — 8 قواعد، كلها لازم تتحفظ حرفيًا)

1. Placement اختياري، مش إجباري قبل أي محتوى.
2. اليوزر ممكن يبقى عنده أكتر من Language Track شغالة في نفس الوقت (كل واحد progress مستقل).
3. Learning Path بيتعاد توليده لما: الـ placement يخلص، الـ Weekly Weakness Analysis يشتغل، أو اليوزر يطلب re-plan صراحة.
4. أي AI-graded submission (pronunciation/writing/quiz) لازم يتخزن فيه الـ raw submission + الـ structured AI feedback — مش score بس، عشان auditability و"AI Explain Mistakes".
5. الـ streak بينكسر لو الـ Daily Goal ما اتحققش لحد آخر اليوم **بتوقيت اليوزر** (لازم نخزن timezone لكل user).
6. SRS scheduling (SM-2) مستقل تمامًا عن الـ Learning Path — cadence منفصل طويل المدى.
7. XP/Coins بيتغيروش مباشر — append-only ledger entries مع reason code (نفس StockSavvy).
8. المحتوى ممكن يتخصص حسب اهتمام اليوزر (`learner_interest_tags`, e.g. "Backend Developer" → programming vocabulary) — يعني الـ content generation prompts لازم تتبني بمعامل `domain_tag`.

### 2.3 Ambiguities من PDF §0.3 (اتحسمت كقرارات معمارية ملزمة من دلوقتي)

|#|الغموض|القرار المعتمد (DECISION)|
|---|---|---|
|1|Multi-language tracks ولا track واحد active؟|**Multi-track من اليوم الأول.** `language_track_id` FK في كل مكان محتاجه. تكلفة بسيطة دلوقتي مقابل migration مؤلمة بعدين.|
|2|Placement إجباري؟|**لأ.** self-declared level بيفتح المحتوى فورًا؛ الـ placement flow اختياري لـ"تأكيد المستوى" فقط، تقليل onboarding friction.|
|3|AI cost quota؟|**آه، من Phase 6.** `daily_ai_quota` لكل يوزر، Redis counters، 429 لو اتخطى.|
|4|Real-time voice streaming ولا upload-then-transcribe؟|**upload-then-transcribe للـ MVP.** True streaming STT/TTS هو Phase 2.5 stretch goal، مش MVP requirement.|

هذه القرارات الأربعة **ملزمة** لباقي المستند ومبنيّ عليها كل تصميم لاحق (خصوصًا Speaking وPlacement وLanguageTrack).

---

## 3. Architectural Principles

1. **Feature-based, not layer-based at the top level.** كل feature (auth, languagetrack, placement, learningpath, vocabulary, grammar, listening, speaking, writing, srs, progress, conversation, mistakepattern, weaknessanalyzer, contentgen, gamification, social) عبارة عن package مستقل جوه فيه الـ 4 layers بتاعته.
2. **Dependency Inversion هو أساس كل حاجة.** الـ Application بتعرف الـ interfaces (ports)، الـ Infrastructure بتنفذها. ده اللي بيخليك تقدر تعمل unit test للـ Application من غير Spring context خالص.
3. **CQRS بدون bus.** كل use case = Command/Query record واحد + Handler class واحد. مفيش `CommandBus`/`Mediator`.
4. **Domain نضيف قدر الإمكان.** بدون `@Entity`, `@Service`, `@Autowired` جوه `domain/`.
5. **AI معزول خلف application/port/ai/**. الـ business logic ميعرفش `ChatClient` أصلاً.
6. **Shared module محدود ومُبرر.** مفيش "common dumping ground" — كل package جوه `shared/` له سبب واضح ومرحلة دخول محددة.
7. **الـ AI plumbing (Phase 6) بيتبني مرة واحدة قبل أول AI feature**، مش جوه كل feature لوحده — ده أخطر نقطة rework في الخطة كلها لو اتجاهلت (زي ما الـ PDF نفسه أكد في §10).

---

## 4. Architecture Overview

### 4.1 بنية كل Feature (مطابقة لما طلبته بالظبط)

```text
feature/
├── api/
│   ├── XxxController.java
│   ├── request/
│   │   └── XxxRequest.java
│   └── response/
│       └── XxxResponse.java
├── application/
│   ├── command/
│   │   └── <verb>/
│   │       ├── XxxCommand.java
│   │       └── XxxHandler.java
│   ├── query/
│   │   └── <verb>/
│   │       ├── XxxQuery.java
│   │       └── XxxHandler.java
│   └── port/
│       ├── XxxRepository.java        (repository port)
│       └── ai/XxxAiService.java      (فقط للـ features اللي محتاجة AI)
├── domain/
│   ├── entity/
│   ├── valueobject/
│   ├── exception/
│   └── service/
└── infrastructure/
    ├── persistence/
    │   ├── XxxEntity.java
    │   └── XxxJpaRepository.java
    ├── mapper/
    │   └── XxxMapper.java
    ├── adapter/
    │   └── XxxRepositoryImpl.java
    └── ai/                            (فقط للـ features اللي محتاجة AI)
        └── XxxAiAdapter.java
```

> **ملاحظة معمارية مهمة:** في الـ course example اللي انت كتبته، الـ `repository/` interface كان جوه `domain/`. إحنا هنا هنحطه جوه `application/port/` بدل `domain/repository/`. **السبب (DECISION):** الـ repository port بيرجع Domain entities، لكنه في الحقيقة "كيف الـ Application بيوصل للبيانات" مش "قاعدة عمل Domain". حطه جوه `application/port/` بيخلي الـ Domain نضيف 100% (zero interfaces حتى)، وبيوضح إن الـ port ده جزء من الـ use-case orchestration مش جزء من الـ business invariants. النمطين (domain/repository أو application/port) سليمين معماريًا (كلاهما Dependency Inversion صحيح) — إحنا هنا بنختار `application/port/` بثبات في المشروع كله لتفادي التخبط.

### 4.2 مثال كامل — Feature: `languagetrack`

```text
languagetrack/
├── api/
│   ├── LanguageTrackController.java
│   ├── request/
│   │   ├── CreateLanguageTrackRequest.java
│   │   └── SetSelfDeclaredLevelRequest.java
│   └── response/
│       ├── LanguageTrackResponse.java
│       └── LanguageTrackListResponse.java
├── application/
│   ├── command/
│   │   ├── create/
│   │   │   ├── CreateLanguageTrackCommand.java
│   │   │   └── CreateLanguageTrackHandler.java
│   │   └── setlevel/
│   │       ├── SetSelfDeclaredLevelCommand.java
│   │       └── SetSelfDeclaredLevelHandler.java
│   ├── query/
│   │   └── listbyuser/
│   │       ├── GetUserLanguageTracksQuery.java
│   │       └── GetUserLanguageTracksHandler.java
│   └── port/
│       └── LanguageTrackRepository.java
├── domain/
│   ├── entity/
│   │   └── LanguageTrack.java
│   ├── valueobject/
│   │   └── (يستخدم CefrLevel من shared/domain المشترك — راجع §7)
│   ├── exception/
│   │   ├── DuplicateLanguageTrackException.java
│   │   └── LanguageTrackNotFoundException.java
│   └── service/
│       └── (مفيش — لا يوجد منطق يتجاوز حدود entity واحد)
└── infrastructure/
    ├── persistence/
    │   ├── LanguageTrackJpaEntity.java
    │   └── LanguageTrackJpaRepository.java
    ├── mapper/
    │   └── LanguageTrackMapper.java
    └── adapter/
        └── LanguageTrackRepositoryImpl.java
```

**No dedicated file required because:** فيه بعض الـ features (زي `languagetrack`) معندهاش `domain/service/` لأن كل الـ business rules بتاعتها (unique active track per user+language, reactivate on re-add) قادرة تتنفذ جوه الـ entity نفسه أو جوه الـ Handler مباشرة — مفيش logic بتربط بين أكتر من aggregate.

---

## 5. Dependency Rules

### 5.1 Dependency Matrix (نفس اللي في الـ PDF §1.2، موسّع بالشرح)

|From \ To|Domain|Application|Infrastructure|API|
|---|---|---|---|---|
|Domain|—|❌|❌|❌|
|Application|✅|—|❌ (interfaces فقط، مش implementations)|❌|
|Infrastructure|✅|✅ (بينفذ الـ ports)|—|❌|
|API|❌ (عن طريق DTOs فقط)|✅|❌|—|

### 5.2 شرح تفصيلي لكل سهم

- **Domain → لا حاجة:** الـ Domain عبارة عن Java classes نضيفة 100%. لو فتحت أي ملف جوه `domain/` ومش لاقي أي `import org.springframework.*` ولا `import jakarta.persistence.*`، يبقى ده صح. الـ Domain entity زي `LanguageTrack.java` هو POJO عادي فيه constructors + methods بتعبر عن سلوك، مش data bag.
- **Application → Domain:** الـ Handler بياخد الـ Domain entity، يستدعي methods عليه (زي `languageTrack.activate()`), يتعامل مع الـ exceptions اللي الـ Domain بيرميها.
- **Application → Application (ports فقط):** الـ `application/port/LanguageTrackRepository.java` عبارة عن `interface` فاضي من أي Spring/JPA annotation:

```java
public interface LanguageTrackRepository {
    Optional<LanguageTrack> findById(Long id);
    Optional<LanguageTrack> findActiveByUserAndLanguage(Long userId, String targetLanguage);
    LanguageTrack save(LanguageTrack track);
    List<LanguageTrack> findAllByUserId(Long userId);
}
```

هذا الـ interface **ممنوع** يستخدم `Pageable` من Spring Data أو `JpaRepository` extends. لو احتجت pagination جوه Application، اعمل abstraction بسيطة بتاعتك (`PageRequest`/`PageResult` جوه `shared/web`)، مش تسحب Spring Data type جوه الـ port.

- **Infrastructure → Application (implements ports) + Domain:**

```java
@Repository
class LanguageTrackRepositoryImpl implements LanguageTrackRepository {
    private final LanguageTrackJpaRepository jpaRepository;
    private final LanguageTrackMapper mapper;

    public Optional<LanguageTrack> findById(Long id) {
        return jpaRepository.findById(id).map(mapper::toDomain);
    }
    public LanguageTrack save(LanguageTrack track) {
        var entity = mapper.toEntity(track);
        return mapper.toDomain(jpaRepository.save(entity));
    }
}
```

هنا `LanguageTrackJpaEntity` (بـ `@Entity`) منفصل تمامًا عن `LanguageTrack` (Domain entity نضيف). الـ `LanguageTrackMapper` هو اللي بيحول بينهم. ده تكلفة إضافية (كلاس mapper إضافي لكل feature) لكنه اللي بيمنع تسريب `@Entity`/`@OneToMany` جوه الـ Domain.

- **API → Application فقط:** الـ Controller بيحقن الـ Handler مباشرة (constructor injection)، بيبني الـ Command/Query من الـ Request DTO، بيبعته للـ Handler، وبيحول الـ نتيجة (Domain entity أو read DTO) لـ Response DTO. الـ Controller **ممنوع** يشوف `LanguageTrackJpaRepository` أو أي كلاس جوه `infrastructure/` — ده بيتفرض عن طريق ArchUnit rule (راجع §22).

### 5.3 ليه الـ repository interface مش لازم يعتمد على Spring Data JPA

لو الـ port بيرث من `JpaRepository<LanguageTrackJpaEntity, Long>`، يبقى الـ Application اتربط بـ:

1. Spring Data (framework coupling).
2. الـ JPA entity نفسه (لازم يرجع `LanguageTrackJpaEntity` مش `LanguageTrack` الـ Domain).

وده بيكسر كل فايدة الـ Clean Architecture — مينفعش تعمل unit test للـ Handler من غير Spring context، ومينفعش تستبدل PostgreSQL بحاجة تانية من غير ما تلمس الـ business logic.

---

## 6. Domain Model

### 6.1 جدول Aggregates / Entities / Value Objects / Domain Services / Repositories لكل Feature

|Feature|Aggregate Root|Entities/VO المرتبطة|Value Objects|Domain Services|Repository Port|
|---|---|---|---|---|---|
|auth|`User`|`EmailVerificationToken`, `RefreshToken`|`Email`, `PasswordHash`|—|`UserRepository`, `RefreshTokenRepository`|
|languagetrack|`LanguageTrack`|—|`CefrLevel`|—|`LanguageTrackRepository`|
|placement|`PlacementTest`|—|`CefrLevel`|—|`PlacementTestRepository`|
|learningpath|`LearningPath`|`LearningPathItem` (child, داخل نفس aggregate)|`CefrLevel`, `ContentType` (enum)|—|`LearningPathRepository`|
|vocabulary|`VocabularyItem`|—|`CefrLevel`|—|`VocabularyRepository`|
|grammar|`GrammarTopic`|—|`CefrLevel`|—|`GrammarTopicRepository`|
|listening|(يستخدم `Submission`/`Assessment` من submission module)|`ListeningClip`|—|—|`ListeningClipRepository`|
|speaking|(يستخدم `Submission`/`Assessment`)|—|`AudioRef`|—|(يستخدم submission ports)|
|writing|(يستخدم `Submission`/`Assessment`)|—|—|—|(يستخدم submission ports)|
|submission (shared core aggregate)|`Submission`|`Assessment` (1:1)|`Score`|—|`SubmissionRepository`, `AssessmentRepository`|
|srs|`SrsCard`|—|—|`SrsScheduler` (SM-2)|`SrsCardRepository`|
|progress|`DailyGoal`, `DailyProgress`|—|—|`StreakCalculator`|`DailyGoalRepository`, `DailyProgressRepository`|
|gamification|`StreakLedgerEntry`, `XpLedgerEntry` (append-only, كل واحد aggregate root مستقل بحكم إنه immutable)|`Achievement`, `Badge` (Phase 13)|—|—|`StreakLedgerRepository`, `XpLedgerRepository`|
|conversation (Phase 2)|`ConversationSession`|`ConversationMessage` (child)|—|—|`ConversationSessionRepository`|
|mistakepattern (Phase 2)|`MistakePattern`|—|—|—|`MistakePatternRepository`|
|weaknessanalyzer (Phase 2)|`ProgressSnapshot`|—|—|`LevelProgressionPolicy`|`ProgressSnapshotRepository`|
|social (Phase 2)|`Friendship`, `Challenge`|`ChallengeParticipant`|—|—|`FriendshipRepository`, `ChallengeRepository`|

### 6.2 تمييز مهم: مش كل Table لازم تبقى Domain Aggregate

من أسئلتك المباشرة: هل كل جدول في §3 لازم يبقى aggregate؟ **لأ.** الفرق:

- **Aggregate root** = عنده business invariants يحميها، وبيتغير من خلال methods بتعبر عن سلوك (`track.activate()`, `card.review(grade)`), وله lifecycle مستقل.
- **Ledger row / append-only record** = `StreakLedgerEntry`, `XpLedgerEntry` — من الناحية الفنية دول "entities" بس معندهمش invariants تتغير بعد الإنشاء (immutable بعد الـ insert)، فبالتالي هما أقرب لـ **Value Object محفوظ في جدول مستقل** أكتر من "aggregate بمعنى الكامل". برضه هنعاملهم كـ aggregate root بسيط (عشان الـ repository pattern يفضل موحّد) لكن من غير أي setter أو state-transition method — الـ constructor بس.
- **LearningPathItem** مش aggregate مستقل — هو **child entity** جوه aggregate الـ `LearningPath` (زي ما الـ PDF وصفه "VO-ish, owned by LearningPath"). يعني مفيش `LearningPathItemRepository` منفصل — بيتحمّل ويتحفظ مع الـ `LearningPath` نفسه.
- **ProgressSnapshot** (Phase 2) هو aggregate مستقل لأنه بيتقرأ لوحده (مش دايمًا محمّل مع حاجة تانية) وله lifecycle مستقل (بيتولد أسبوعيًا من الـ Weakness Analyzer job).
- **Achievement/Badge** (Phase 13) أقرب لـ reference/catalog data (زي `GrammarTopic`) — configuration-like entities مش أساسها transactional.

### 6.3 Value Objects (من PDF §2.2 — تفصيليًا)

|Value Object|الحقول|القاعدة (Invariant)|مين بيستخدمه|
|---|---|---|---|
|`CefrLevel`|enum: `A1,A2,B1,B2,C1,C2`|ordinal-comparable (`isAtLeast(other)`)|languagetrack, placement, learningpath, vocabulary, grammar|
|`Email`|value: String|regex + normalization (lowercase)|auth/User|
|`AudioRef`|storageKey, durationSeconds, mimeType|durationSeconds > 0, mimeType ضمن whitelist|speaking/Submission|
|`Score`|value: int 0-100|لازم يبقى بين 0 و100 وإلا exception في الـ constructor|submission/Assessment|
|`PasswordHash` (ARCHITECTURAL RECOMMENDATION)|hash: String|تخزين bcrypt hash فقط، مينفعش يتبني من plain text مباشر — الـ hashing نفسه بيحصل في Infrastructure (BCryptPasswordEncoder) مش في الـ VO لأن الـ VO لازم يفضل framework-free|auth/User|

> **ARCHITECTURAL RECOMMENDATION:** الـ PDF ما وضحش إزاي هيتم الـ password hashing من ناحية الطبقة. القرار: الـ hashing algorithm (bcrypt) هو Infrastructure concern (يحتاج `PasswordEncoder` من Spring Security)، لكن الـ _invariant_ "مينفعش الـ User يتبني من غير password hash صالح لو LOCAL auth" هو Domain concern. الحل: الـ `RegisterUserHandler` (Application) بيستخدم port `PasswordHasher` (interface بسيط، مش `PasswordEncoder` من Spring مباشرة) يبعته للـ Domain factory method `User.registerLocal(email, hashedPassword)`.

### 6.4 Domain Services (من PDF §2.3)

|Domain Service|ليه هو Domain Service مش method على entity واحد|
|---|---|
|`SrsScheduler`|بياخد `SrsCard` + review grade ويحسب `easeFactor`/`intervalDays`/`dueAt` الجديدة بخوارزمية SM-2 — منطق حسابي مستقل معقد كفاية إنه يبقى class لوحده بدل ما يتكرر جوه الـ entity، وسهل يتعمله unit test منعزل بكل الـ edge cases (ease factor floor مثلاً).|
|`StreakCalculator`|محتاج "النهاردة بتوقيت اليوزر" (external context: timezone) + `DailyProgress` النهاردة + حالة الـ streak إمبارح، عشان يقرر increment أو reset — العملية دي بتلمس حالتين (progress + ledger) مش entity واحد.|
|`LevelProgressionPolicy` (Phase 2)|بيقرر لو نتيجة الـ Weakness Analysis المفروض تغيّر `currentLevel` بتاع `LanguageTrack` — بيربط بين `ProgressSnapshot` و`LanguageTrack`، أي منطق عابر بين aggregates بيتحط في domain service مش جوه أي أجرِگت.|

**كل باقي المنطق** (زي "duplicate active track" أو "max question count في placement") بيتحط كـ **factory method أو behavior method جوه الـ entity نفسه**، مثلاً:

```java
// domain/entity/LanguageTrack.java
public static LanguageTrack createNew(Long userId, String targetLanguage, CefrLevel declaredLevel) {
    return new LanguageTrack(null, userId, targetLanguage, declaredLevel, TrackStatus.ACTIVE, Instant.now());
}
public void reactivate() {
    if (this.status == TrackStatus.ACTIVE) throw new IllegalStateException("Already active");
    this.status = TrackStatus.ACTIVE;
}
```

---

## 7. Aggregates & Boundaries

**قاعدة الحدود (Aggregate Boundary Rule):** أي عملية بتحتاج تحدّث أكتر من aggregate root في نفس الوقت — إما إنها _مش_ لازم تكون atomic (يبقى نبعت domain event ونعالجها async)، أو إننا نعيد التفكير في الحدود.

أمثلة تطبيقية من المشروع:

- **`LearningPath` ↔ `LearningPathItem`:** حد واحد (single aggregate) — الـ items بتتحفظ وتتحمّل مع الـ path، معندهاش repository أو lifecycle مستقل. التحديث بيحصل عن طريق `learningPath.markItemCompleted(itemId)`.
- **`Submission` ↔ `Assessment`:** حدين منفصلين لكن بعلاقة 1:1 صارمة. **السبب:** الـ `Submission` بيتخلق فورًا (وقت الرفع)، لكن الـ `Assessment` بيتخلق لاحقًا (بعد AI processing، أحيانًا async زي Speaking). لو حطيناهم aggregate واحد هيبقى فيه فترة "الـ aggregate ناقص" وده مربك. الحل: `Submission` بيتحمّل ويتحدّث لوحده، و`Assessment` بيتحمّل ويتحدّث لوحده، مربوطين بـ `submissionId` FK (UNIQUE).
- **`XpLedgerEntry` ↔ `User` balance:** **متربطوش** — الـ `User` aggregate ماعندوش حقل `xpBalance` مباشر. الـ balance بيتحسب (`SUM(amount)`) أو بيتقرا من Redis sorted set (راجع §16). ده قرار متعمد يمنع race condition على تحديث رقم واحد من كذا مصدر في نفس الوقت.
- **`LanguageTrack` ↔ `PlacementTest`:** حدين منفصلين، الـ `PlacementTest` بيشاور على `languageTrackId` كـ reference، مش composition. لما الـ placement يخلص، الـ `LevelProgressionPolicy` (domain service) بيقرر تحديث `LanguageTrack.currentLevel` — العملية دي بتحصل جوه handler واحد (`CompletePlacementTestHandler`) بيحمّل الاتنين، يعدلهم، ويحفظهم في نفس transaction (transaction boundary = Application layer، مش Domain).

---

## 8. Value Objects — راجع §6.3

كل الـ Value Objects موثقة بالتفصيل في **§6.3** فوق (`CefrLevel`, `Email`, `AudioRef`, `Score`, `PasswordHash`) مع القاعدة (invariant) بتاعة كل واحد ومين بيستخدمه. مفيش تكرار هنا تفاديًا للـ duplication.

**إضافة واحدة (ARCHITECTURAL RECOMMENDATION):** `ContentType` (enum: `VOCAB, GRAMMAR, SPEAKING, LISTENING`) المستخدم في `LearningPathItem` هو Value Object كمان لأنه immutable وبيوصف نوع، مش identity.

---

## 9. Domain Services — راجع §6.4

موثقين بالتفصيل في **§6.4**: `SrsScheduler`, `StreakCalculator`, `LevelProgressionPolicy`. مفيش domain service رابع محتاج — أي منطق تاني قابل يتحط جوه entity واحد.

---

## 10. Complete Feature Map

> تنسيق كل feature: Purpose · Actors · Commands · Queries · Domain entities · Repository ports · External ports · Business rules · API endpoints · Request/Response DTOs · Validation · Exceptions · DB tables · Events · Background jobs · Cache · Tests · Dependencies.

### 10.1 `auth`

- **Purpose:** register, login, verify email, Google OAuth, issue/refresh JWT.
- **Actors:** Anonymous, RegisteredUser.
- **Commands:** `RegisterUserCommand`, `VerifyEmailCommand`, `LoginCommand`, `RefreshTokenCommand`, `GoogleLoginCommand`.
- **Queries:** `GetCurrentUserQuery`.
- **Domain entities:** `User`, `EmailVerificationToken`, `RefreshToken`.
- **Repository ports:** `UserRepository`, `EmailVerificationTokenRepository`, `RefreshTokenRepository`.
- **External ports:** `PasswordHasher`, `JwtTokenIssuer` (ينفذوا في `shared/security` لأنهم مش خاصين بـ auth feature وحده — راجع §17).
- **Business rules:** email فريد (unique constraint، مش pre-check query فقط)؛ password complexity (min 8 chars، حرف كبير+رقم — ARCHITECTURAL RECOMMENDATION لأن الـ PDF ماحددش)؛ unverified users يقدروا يعملوا login بس بيتحطوا في "verify your email" gate (بفلتر، مش handler-by-handler)؛ Google login بيعمل auto-verify للـ email؛ refresh token reuse detection (لو token اتستخدم مرتين، كل الـ family بتتلغي).
- **Endpoints:**
    - `POST /api/v1/auth/register`
    - `POST /api/v1/auth/verify-email`
    - `POST /api/v1/auth/login`
    - `POST /api/v1/auth/refresh`
    - `POST /api/v1/auth/google`
    - `GET /api/v1/users/me`
- **Request DTOs:** `RegisterRequest`, `VerifyEmailRequest`, `LoginRequest`, `RefreshTokenRequest`, `GoogleLoginRequest`.
- **Response DTOs:** `AuthTokenResponse` (accessToken, refreshToken, expiresIn), `UserResponse`.
- **Validation:** `@Email`, `@NotBlank`, `@Size(min=8)` على الـ Request DTOs (Jakarta Bean Validation).
- **Exceptions:** `DuplicateEmailException` (409), `InvalidCredentialsException` (401), `EmailNotVerifiedException` (403 — على content endpoints بس، مش login)، `ExpiredTokenException` (401), `RefreshTokenReuseDetectedException` (401 + revoke family), `EmailAlreadyLinkedToLocalAccountException` (409).
- **DB tables:** `users`, `email_verification_tokens`, `refresh_tokens`.
- **Events:** `UserRegisteredEvent` (يُستخدم لاحقًا لإرسال verification email — Application event، مش domain event محفوظ).
- **Background jobs:** لا يوجد في Phase 1 (تنضيف expired tokens ممكن تتضاف لاحقًا كـ recommendation).
- **Cache:** JWT blacklist (`jwt:blacklist:{jti}`) — راجع §16.
- **Tests:** unit (password hashing، token expiry logic)، handler tests (duplicate email → 409)، integration (register→verify→login flow، Testcontainers).
- **Dependencies:** لا يعتمد على أي feature تاني (هو الأساس).

### 10.2 `languagetrack`

- **Purpose:** اختيار target language + إعلان/تأكيد المستوى.
- **Actors:** RegisteredUser.
- **Commands:** `CreateLanguageTrackCommand`, `SetSelfDeclaredLevelCommand`.
- **Queries:** `GetUserLanguageTracksQuery`.
- **Domain entities:** `LanguageTrack`.
- **Repository ports:** `LanguageTrackRepository`.
- **External ports:** لا يوجد.
- **Business rules:** track واحد active لكل (user, language)؛ إعادة إضافة لغة متوقفة (paused) بتعمل reactivate مش duplicate.
- **Endpoints:** `POST /api/v1/language-tracks`, `GET /api/v1/language-tracks`.
- **Request/Response DTOs:** `CreateLanguageTrackRequest`, `LanguageTrackResponse`, `LanguageTrackListResponse`.
- **Validation:** `targetLanguage` ضمن whitelist مدعوم، `declaredLevel` قيمة `CefrLevel` صحيحة.
- **Exceptions:** `DuplicateLanguageTrackException` (409), `InvalidCefrLevelException` (422).
- **DB tables:** `language_tracks`.
- **Events:** `LanguageTrackCreatedEvent` (يُستخدم لاحقًا كـ trigger اختياري).
- **Background jobs:** لا يوجد.
- **Cache:** لا يوجد.
- **Tests:** unit (`reactivate()` behavior)، handler (duplicate active track → 409)، integration.
- **Dependencies:** auth (userId من الـ JWT).

### 10.3 `placement`

- **Purpose:** conversational interview → CEFR level عن طريق AI.
- **Actors:** RegisteredUser.
- **Commands:** `StartPlacementTestCommand`, `SubmitPlacementAnswerCommand`.
- **Queries:** `GetPlacementResultQuery`.
- **Domain entities:** `PlacementTest`.
- **Repository ports:** `PlacementTestRepository`.
- **External ports:** `PlacementTestAiService` (AI port).
- **Business rules:** max 6 أسئلة قبل ما يتفرض نتيجة؛ الـ AI لازم يرجع structured level (JSON schema)، مش free text؛ لو اليوزر سايب الاختبار نص، الـ track فاضل على self-declared level.
- **Endpoints:** `POST /api/v1/placement-tests`, `POST /api/v1/placement-tests/{id}/answers`, `GET /api/v1/placement-tests/{id}`.
- **Request/Response DTOs:** `StartPlacementTestRequest`, `SubmitAnswerRequest`, `PlacementTestResponse`.
- **Validation:** `languageTrackId` موجود ومملوك لليوزر.
- **Exceptions:** `PlacementTestNotFoundException` (404), `PlacementTestAlreadyCompletedException` (409), `MalformedAiResponseException` (يتعالج internally بـ retry-then-fallback، مش هيبان للـ client كـ error عادي).
- **DB tables:** `placement_tests`.
- **Events:** `PlacementTestCompletedEvent` (trigger لـ `GenerateLearningPathCommand`).
- **Background jobs:** لا يوجد.
- **Cache:** لا يوجد (كل test فريد، مفيش فايدة كاش).
- **Tests:** handler tests بـ mocked AI port بالكامل؛ AI fallback test (malformed response → retry once → fallback لـ self-declared)؛ live-AI integration test تحت `@Tag("ai-integration")`.
- **Dependencies:** languagetrack (لازم track موجود)، shared/ai (Phase 6 foundation).

### 10.4 `learningpath`

- **Purpose:** توليد خطة أسبوعية.
- **Commands:** `GenerateLearningPathCommand`.
- **Queries:** `GetActiveLearningPathQuery`.
- **Domain entities:** `LearningPath` (aggregate root)، `LearningPathItem` (child).
- **Repository ports:** `LearningPathRepository`.
- **External ports:** `LearningPathAiService` (AI port — بيختار ويرتب المحتوى، مش بيخترعه من الصفر).
- **Business rules:** التوليد idempotent لكل (track, week) — إعادة التوليد بتعمل supersede مش duplicate؛ لازم يسحب من `vocabulary_items`/`grammar_topics` الموجودين مفلترين بـ level+domain_tag؛ لو المحتوى مش كفاية للمستوى/اللغة → fallback لخطة افتراضية جاهزة (curated default).
- **Endpoints:** `POST /api/v1/learning-paths/generate`, `GET /api/v1/learning-paths/active`.
- **Request/Response DTOs:** `GenerateLearningPathRequest` (languageTrackId)، `LearningPathResponse` (مع items).
- **Validation:** `languageTrackId` مملوك لليوزر.
- **Exceptions:** `InsufficientContentPoolException` (يتعالج بـ fallback داخليًا)، `LanguageTrackNotFoundException`.
- **DB tables:** `learning_paths`, `learning_path_items`.
- **Events:** `LearningPathGeneratedEvent`.
- **Background jobs:** لا يوجد في Phase 1 (الإعادة التلقائية بعد Weakness Analysis بتحصل في Phase 2/11).
- **Cache:** `path:active:{trackId}` (1h TTL، evict on regenerate) — راجع §16.
- **Tests:** idempotency test (regenerate → supersede)، AI fallback behavior، handler test.
- **Dependencies:** languagetrack، vocabulary، grammar، shared/ai.

### 10.5 `vocabulary`

- **Commands:** `MarkVocabularyLearnedCommand`.
- **Queries:** `GetVocabularyItemQuery`, `ListVocabularyByLevelQuery`.
- **Domain entities:** `VocabularyItem`.
- **Repository ports:** `VocabularyRepository`.
- **Business rules:** تحديد "learned" بيعمل create لـ `SrsCard` جديدة (cross-feature call عن طريق event أو port — راجع القرار في §10.10).
- **Endpoints:** `GET /api/v1/vocabulary/{id}`, `GET /api/v1/vocabulary?language=&level=`, `POST /api/v1/vocabulary/{id}/learned`.
- **DTOs:** `VocabularyItemResponse`, `VocabularyListResponse` (paginated).
- **Exceptions:** `VocabularyItemNotFoundException` (404).
- **DB tables:** `vocabulary_items`.
- **Cache:** `vocab:{lang}:{level}` (6h TTL).
- **Tests:** query filtering، mark-learned → SrsCard creation (handler test بـ mocked SrsCard port).
- **Dependencies:** srs (لخلق الـ card).

### 10.6 `grammar`

- **Queries:** `GetGrammarExplanationQuery` (AI-personalized rendering).
- **Domain entities:** `GrammarTopic`.
- **Repository ports:** `GrammarTopicRepository`.
- **External ports:** `GrammarExplanationAiService` (بيعيد صياغة `explanationTemplate` بأسلوب: default/eli5/arabic/football/programming).
- **Endpoints:** `GET /api/v1/grammar/{topicId}?style=default|eli5|arabic|football|programming`.
- **DB tables:** `grammar_topics`.
- **Cache:** نفس prompt-templating utility المشتركة مع "AI Teacher" (Phase 2) — بتتبني مرة واحدة في `shared/ai/prompt`.
- **Tests:** query test بكل الـ styles، AI port mocked.
- **Dependencies:** shared/ai.

### 10.7 `listening`

- **Commands:** `SubmitListeningAnswerCommand`.
- **Domain entities:** `ListeningClip`؛ يستخدم `Submission`(type=QUIZ) + `Assessment` من submission core.
- **Endpoints:** `GET /api/v1/listening/{clipId}`, `POST /api/v1/listening/{clipId}/answers`.
- **DB tables:** `listening_clips` (ARCHITECTURAL RECOMMENDATION — الـ PDF ماذكرش جدول صريح للـ clip metadata، بس لازم يكون موجود عشان `GET /listening/{clipId}` يرجع حاجة).
- **Dependencies:** submission (core)، shared/ai.

### 10.8 `speaking`

- **Commands:** `SubmitSpeechCommand`.
- **Business rules:** async pipeline (STT → pronunciation scoring → Assessment)؛ endpoint بيرجع 202 + submission id، الـ client يعمل poll أو يستقبل WebSocket push.
- **Endpoints:** `POST /api/v1/speaking/submissions` (202), `GET /api/v1/speaking/submissions/{id}`.
- **Edge cases:** صوت قصير جدًا/صامت → reject قبل ما نستدعي STT مدفوعة؛ low-confidence STT → `Assessment.status = LOW_CONFIDENCE` بدل score مضلل.
- **External ports:** `SpeechToTextAiService`, `PronunciationScoringAiService`.
- **DB tables:** يستخدم `submissions`/`assessments`.
- **WebSocket:** `SpeakingAssessmentReadyNotifier` (يبعت على topic `/topic/speaking/{userId}`).
- **Dependencies:** submission core، shared/ai، shared/websocket.

### 10.9 `writing`

- **Commands:** `SubmitWritingCommand`.
- **Business rules:** الـ AI بيرجع structured `{ corrections[], naturalVersion, formalVersion, nativeVersion }` (JSON schema enforced).
- **Endpoints:** `POST /api/v1/writing/submissions`, `GET /api/v1/writing/submissions/{id}`.
- **External ports:** `WritingAssessmentAiService`.
- **DB tables:** يستخدم `submissions`/`assessments` (feedback_json بالـ schema المذكور).
- **Dependencies:** submission core، shared/ai.

### 10.10 `srs`

- **Commands:** `ReviewSrsCardCommand` (grade: again/hard/good/easy)، `CreateSrsCardCommand` (internal — بينده من `vocabulary` عند mark-learned).
- **Queries:** `GetDueSrsCardsQuery`.
- **Domain entities:** `SrsCard`.
- **Domain service:** `SrsScheduler` (SM-2).
- **Endpoints:** `GET /api/v1/srs/due`, `POST /api/v1/srs/{cardId}/review`.
- **DB tables:** `srs_cards` (index حرج على `(user_id, due_at)`).
- **Cross-feature call (DECISION):** `vocabulary` بيستدعي `CreateSrsCardCommand` مباشرة (نفس الـ pattern، Handler-to-Handler عن طريق interface بسيط `SrsCardCreationPort` جوه application/port لـ `vocabulary`) — **مش** عن طريق domain event async، لأن العملية لازم تحصل atomic (mark-learned من غير card = bug). لو عايز decoupling أكتر، البديل هو Spring `ApplicationEventPublisher` (sync listener، نفس transaction) — كلاهما مقبول، القرار هنا: **direct port call** لأنه أوضح وأسهل debugging لمطور واحد.
- **Dependencies:** vocabulary (caller)، shared/cache (due-reminder لاحقًا).

### 10.11 `progress` (Daily Goal + Daily Progress + Streak base)

- **Commands:** `SetDailyGoalCommand`, `RecordActivityCommand` (internal، بينده من features تانية عند كل نشاط مكتمل).
- **Queries:** `GetDailyProgressQuery`, `GetStreakQuery`.
- **Domain entities:** `DailyGoal`, `DailyProgress`.
- **Domain service:** `StreakCalculator`.
- **Endpoints:** `PUT /api/v1/goals/daily`, `GET /api/v1/progress/today`, `GET /api/v1/progress/streak`.
- **Business rules:** "النهاردة" بتوقيت اليوزر (`users.timezone`)، مش server UTC.
- **DB tables:** `daily_goals`, `daily_progress`, `streak_ledger`.
- **Dependencies:** auth (لـ timezone)، gamification (streak ledger).

### 10.12 `gamification` (XP/Streak core في Phase 1؛ Achievements/Leaderboard في Phase 13)

- **Commands:** `AppendXpLedgerEntryCommand` (internal)، `AppendStreakLedgerEntryCommand` (internal).
- **Domain entities:** `XpLedgerEntry`, `StreakLedgerEntry` (append-only).
- **Business rules:** لا mutation مباشر — insert بس، مع `reason_code`.
- **DB tables:** `xp_ledger`, `streak_ledger`.
- **Cache:** `leaderboard:xp:global` (Redis sorted set، `ZINCRBY` عند كل insert) — Phase 13/10.
- **Dependencies:** progress، srs، listening/writing/speaking (كل مصدر XP).

### 10.13 `conversation` (Phase 2)

- **Purpose:** chat/roleplay session مع AI persona.
- **Commands:** `StartConversationCommand`, `SendConversationMessageCommand`.
- **Domain entities:** `ConversationSession`, `ConversationMessage`.
- **External ports:** `ConversationAiService`.
- **Endpoints:** `POST /api/v1/conversations`, `POST /api/v1/conversations/{id}/messages`.
- **DB tables:** `conversation_sessions`, `conversation_messages`.
- **Dependencies:** shared/ai، languagetrack.

### 10.14 `mistakepattern` (Phase 2 — AI Memory)

- **Domain entities:** `MistakePattern`.
- **Events:** listener على `AssessmentCreatedEvent` — يحلل الـ feedback ويحدّث/يخلق pattern.
- **DB tables:** `mistake_patterns`.
- **Dependencies:** submission core (event source).

### 10.15 `weaknessanalyzer` (Phase 2)

- **Domain entities:** `ProgressSnapshot`.
- **Domain service:** `LevelProgressionPolicy`.
- **Background job:** Weekly Weakness Analyzer (`@Scheduled` cron، chunked per-user).
- **DB tables:** `progress_snapshots`.
- **Dependencies:** progress، submission core، languagetrack، learningpath (trigger لـ regenerate).

### 10.16 `contentgen` (Phase 2 — Vocabulary/Story/Quiz Generators + Translate)

- **Commands:** `GenerateVocabularyCommand`, `GenerateStoryCommand`, `GenerateQuizCommand`.
- **Queries:** `TranslateContextQuery` (stateless).
- **External ports:** `VocabularyGenerationAiService`, `StoryGenerationAiService`, `QuizGenerationAiService`, `TranslationAiService`.
- **Endpoints:** `POST /api/v1/vocabulary/generate`, `POST /api/v1/stories`, `POST /api/v1/quizzes/generate`, `POST /api/v1/translate`.
- **Cache:** `quiz:{topicId}:{level}` (24h).
- **DB tables:** يكتب على `vocabulary_items`، جدول جديد `generated_stories`.
- **Dependencies:** vocabulary، shared/ai.

### 10.17 `social` (Phase 2)

- **Domain entities:** `Friendship`, `Challenge`, `ChallengeParticipant`.
- **DB tables:** `friendships`, `challenges`, `challenge_participants`.
- **Dependencies:** gamification (XP كمقياس للتحدي)، auth.

---

## 11. CQRS Command/Query Inventory (Master Table)

|Feature|Type|Name|Handler|Input|Output|Mutates?|Dependencies|
|---|---|---|---|---|---|---|---|
|auth|Command|RegisterUserCommand|RegisterUserHandler|email, password, displayName, nativeLanguage, timezone|userId|✅|UserRepository, PasswordHasher|
|auth|Command|VerifyEmailCommand|VerifyEmailHandler|token|void|✅|EmailVerificationTokenRepository, UserRepository|
|auth|Command|LoginCommand|LoginHandler|email, password|AuthTokenResponse|✅ (refresh token insert)|UserRepository, PasswordHasher, JwtTokenIssuer|
|auth|Command|RefreshTokenCommand|RefreshTokenHandler|refreshToken|AuthTokenResponse|✅|RefreshTokenRepository, JwtTokenIssuer|
|auth|Command|GoogleLoginCommand|GoogleLoginHandler|googleIdToken|AuthTokenResponse|✅|UserRepository, GoogleTokenVerifier|
|auth|Query|GetCurrentUserQuery|GetCurrentUserHandler|userId|UserResponse|❌|UserRepository|
|languagetrack|Command|CreateLanguageTrackCommand|CreateLanguageTrackHandler|userId, targetLanguage, declaredLevel|trackId|✅|LanguageTrackRepository|
|languagetrack|Command|SetSelfDeclaredLevelCommand|SetSelfDeclaredLevelHandler|trackId, level|void|✅|LanguageTrackRepository|
|languagetrack|Query|GetUserLanguageTracksQuery|GetUserLanguageTracksHandler|userId|List<LanguageTrackResponse>|❌|LanguageTrackRepository|
|placement|Command|StartPlacementTestCommand|StartPlacementTestHandler|trackId|testId + firstQuestion|✅|PlacementTestRepository, PlacementTestAiService|
|placement|Command|SubmitPlacementAnswerCommand|SubmitPlacementAnswerHandler|testId, answer|nextQuestion or resultLevel|✅|PlacementTestRepository, PlacementTestAiService|
|placement|Query|GetPlacementResultQuery|GetPlacementResultHandler|testId|PlacementTestResponse|❌|PlacementTestRepository|
|learningpath|Command|GenerateLearningPathCommand|GenerateLearningPathHandler|trackId|pathId|✅|LearningPathRepository, VocabularyRepository, GrammarTopicRepository, LearningPathAiService|
|learningpath|Query|GetActiveLearningPathQuery|GetActiveLearningPathHandler|trackId|LearningPathResponse|❌|LearningPathRepository|
|vocabulary|Command|MarkVocabularyLearnedCommand|MarkVocabularyLearnedHandler|userId, vocabularyItemId|void|✅|VocabularyRepository, SrsCardCreationPort|
|vocabulary|Query|GetVocabularyItemQuery|GetVocabularyItemHandler|itemId|VocabularyItemResponse|❌|VocabularyRepository|
|vocabulary|Query|ListVocabularyByLevelQuery|ListVocabularyByLevelHandler|language, level, page|VocabularyListResponse|❌|VocabularyRepository|
|grammar|Query|GetGrammarExplanationQuery|GetGrammarExplanationHandler|topicId, style|GrammarExplanationResponse|❌|GrammarTopicRepository, GrammarExplanationAiService|
|listening|Command|SubmitListeningAnswerCommand|SubmitListeningAnswerHandler|clipId, userId, answer|submissionId + score|✅|ListeningClipRepository, SubmissionRepository, AssessmentRepository|
|listening|Query|GetListeningClipQuery|GetListeningClipHandler|clipId|ListeningClipResponse|❌|ListeningClipRepository|
|speaking|Command|SubmitSpeechCommand|SubmitSpeechHandler|userId, audioRef|submissionId (202)|✅|SubmissionRepository, SpeechToTextAiService (async)|
|speaking|Query|GetSpeakingSubmissionQuery|GetSpeakingSubmissionHandler|submissionId|SubmissionStatusResponse|❌|SubmissionRepository, AssessmentRepository|
|writing|Command|SubmitWritingCommand|SubmitWritingHandler|userId, text|submissionId|✅|SubmissionRepository, WritingAssessmentAiService|
|writing|Query|GetWritingSubmissionQuery|GetWritingSubmissionHandler|submissionId|SubmissionStatusResponse|❌|SubmissionRepository, AssessmentRepository|
|srs|Command|ReviewSrsCardCommand|ReviewSrsCardHandler|cardId, grade|nextDueAt|✅|SrsCardRepository, SrsScheduler|
|srs|Command|CreateSrsCardCommand (internal)|CreateSrsCardHandler|userId, vocabularyItemId|cardId|✅|SrsCardRepository|
|srs|Query|GetDueSrsCardsQuery|GetDueSrsCardsHandler|userId|List<SrsCardResponse>|❌|SrsCardRepository|
|progress|Command|SetDailyGoalCommand|SetDailyGoalHandler|userId, targetMinutes, targetWords, targetLessons|void|✅|DailyGoalRepository|
|progress|Command|RecordActivityCommand (internal)|RecordActivityHandler|userId, minutes, words, lessons|void|✅|DailyProgressRepository, StreakCalculator, StreakLedgerRepository|
|progress|Query|GetDailyProgressQuery|GetDailyProgressHandler|userId, date|DailyProgressResponse|❌|DailyProgressRepository|
|progress|Query|GetStreakQuery|GetStreakHandler|userId|StreakResponse|❌|StreakLedgerRepository|
|gamification|Command|AppendXpLedgerEntryCommand (internal)|AppendXpLedgerEntryHandler|userId, amount, reasonCode, refType, refId|entryId|✅|XpLedgerRepository|
|conversation|Command|StartConversationCommand|StartConversationHandler|trackId, mode|sessionId|✅|ConversationSessionRepository, ConversationAiService|
|conversation|Command|SendConversationMessageCommand|SendConversationMessageHandler|sessionId, message|aiReply|✅|ConversationSessionRepository, ConversationAiService|
|mistakepattern|Query|ListMistakePatternsQuery|ListMistakePatternsHandler|userId|List<MistakePatternResponse>|❌|MistakePatternRepository|
|weaknessanalyzer|Command|RunWeaknessAnalysisCommand (internal/scheduled)|RunWeaknessAnalysisHandler|userId|snapshotId|✅|ProgressSnapshotRepository, LevelProgressionPolicy|
|contentgen|Command|GenerateVocabularyCommand|GenerateVocabularyHandler|domainTag, level, language|List<vocabularyItemId>|✅|VocabularyRepository, VocabularyGenerationAiService|
|contentgen|Command|GenerateStoryCommand|GenerateStoryHandler|vocabularyItemIds|storyId|✅|GeneratedStoryRepository, StoryGenerationAiService|
|contentgen|Command|GenerateQuizCommand|GenerateQuizHandler|topicId, level|quizPayload|✅(cache)|QuizGenerationAiService|
|contentgen|Query|TranslateContextQuery|TranslateContextHandler|text, targetLanguage|translation|❌|TranslationAiService|
|gamification|Query|GetLeaderboardQuery|GetLeaderboardHandler|limit|LeaderboardResponse|❌|Redis sorted set|
|social|Command|SendFriendRequestCommand|SendFriendRequestHandler|userId, targetUserId|requestId|✅|FriendshipRepository|
|social|Command|CreateChallengeCommand|CreateChallengeHandler|userId, participantIds, metric, duration|challengeId|✅|ChallengeRepository|

**كل use case ظهر مرة واحدة بس.** الـ internal commands (زي `RecordActivityCommand`, `AppendXpLedgerEntryCommand`) بتتنده من features تانية عن طريق direct handler injection (نفس الـ pattern بتاع `CreateSrsCardCommand` في §10.10) — مفيش لينا داعي لـ event bus هنا لأن العدد صغير والعلاقات معروفة compile-time.