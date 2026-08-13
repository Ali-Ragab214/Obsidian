# Full Project Structure — File-Level Tree

Companion to `language-platform-architecture-plan.md`. Every feature package follows the same 4-layer shape (`domain / application / infrastructure / api`) established in §1.1 of that doc. Below is the literal file tree — every class, interface, and config file, root to leaf.

```
language-platform/
├── docker-compose.yml
├── docker-compose.prod.yml
├── Dockerfile
├── pom.xml
├── .env.example
├── README.md
│
├── src/main/java/com/yourorg/langplatform/
│   │
│   ├── LangPlatformApplication.java
│   │
│   ├── shared/
│   │   ├── security/
│   │   │   ├── SecurityConfig.java
│   │   │   ├── JwtTokenProvider.java
│   │   │   ├── JwtAuthenticationFilter.java
│   │   │   ├── CustomUserDetails.java
│   │   │   ├── CustomUserDetailsService.java
│   │   │   ├── CurrentUser.java                 # @CurrentUser param annotation
│   │   │   └── CurrentUserArgumentResolver.java
│   │   ├── exception/
│   │   │   ├── GlobalExceptionHandler.java       # @ControllerAdvice
│   │   │   ├── DomainException.java              # base
│   │   │   ├── ResourceNotFoundException.java
│   │   │   ├── ConflictException.java
│   │   │   ├── BusinessRuleViolationException.java
│   │   │   ├── UnauthorizedException.java
│   │   │   └── AiQuotaExceededException.java
│   │   ├── validation/
│   │   │   ├── ValidCefrLevel.java
│   │   │   ├── CefrLevelValidator.java
│   │   │   ├── ValidTimezone.java
│   │   │   └── TimezoneValidator.java
│   │   ├── audit/
│   │   │   ├── Auditable.java                    # @MappedSuperclass
│   │   │   ├── JpaAuditingConfig.java
│   │   │   └── AuditorAwareImpl.java
│   │   ├── ai/
│   │   │   ├── AiClientConfig.java                # Spring AI ChatClient bean
│   │   │   ├── StructuredOutputParser.java
│   │   │   ├── PromptTemplateLoader.java
│   │   │   ├── AiQuotaGuard.java                  # Redis-backed per-user daily cap
│   │   │   └── AiResponseException.java
│   │   ├── cache/
│   │   │   ├── RedisConfig.java
│   │   │   └── CacheKeys.java                     # centralized key-naming constants
│   │   ├── event/
│   │   │   ├── DomainEvent.java
│   │   │   └── DomainEventPublisher.java          # wraps ApplicationEventPublisher
│   │   ├── scheduling/
│   │   │   └── ShedLockConfig.java                # added in Phase 16 (multi-instance safety)
│   │   └── web/
│   │       ├── ApiResponse.java                   # { data, meta } envelope
│   │       ├── ApiError.java                      # { code, message, details[] }
│   │       ├── PageResponse.java
│   │       └── CorrelationIdFilter.java
│   │
│   ├── auth/
│   │   ├── domain/
│   │   │   ├── User.java
│   │   │   ├── Email.java                         # value object
│   │   │   ├── AuthProvider.java                  # enum: LOCAL, GOOGLE
│   │   │   ├── EmailVerificationToken.java
│   │   │   ├── RefreshToken.java
│   │   │   └── exception/
│   │   │       ├── DuplicateEmailException.java
│   │   │       ├── InvalidCredentialsException.java
│   │   │       └── TokenExpiredException.java
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── RegisterUserCommand.java
│   │   │   │   ├── RegisterUserHandler.java
│   │   │   │   ├── VerifyEmailCommand.java
│   │   │   │   ├── VerifyEmailHandler.java
│   │   │   │   ├── LoginCommand.java
│   │   │   │   ├── LoginHandler.java
│   │   │   │   ├── RefreshTokenCommand.java
│   │   │   │   ├── RefreshTokenHandler.java
│   │   │   │   ├── GoogleLoginCommand.java
│   │   │   │   └── GoogleLoginHandler.java
│   │   │   ├── query/
│   │   │   │   ├── GetCurrentUserQuery.java
│   │   │   │   └── GetCurrentUserHandler.java
│   │   │   ├── port/
│   │   │   │   ├── UserRepository.java
│   │   │   │   ├── EmailVerificationTokenRepository.java
│   │   │   │   ├── RefreshTokenRepository.java
│   │   │   │   ├── EmailSender.java
│   │   │   │   └── GoogleTokenVerifier.java
│   │   │   └── dto/
│   │   │       ├── AuthResponse.java
│   │   │       ├── TokenPair.java
│   │   │       └── UserResponse.java
│   │   ├── infrastructure/
│   │   │   ├── persistence/
│   │   │   │   ├── JpaUserRepository.java          # Spring Data interface
│   │   │   │   ├── UserRepositoryAdapter.java       # implements application/port/UserRepository
│   │   │   │   ├── JpaEmailVerificationTokenRepository.java
│   │   │   │   ├── EmailVerificationTokenRepositoryAdapter.java
│   │   │   │   ├── JpaRefreshTokenRepository.java
│   │   │   │   └── RefreshTokenRepositoryAdapter.java
│   │   │   ├── email/
│   │   │   │   └── SmtpEmailSenderAdapter.java
│   │   │   └── google/
│   │   │       └── GoogleTokenVerifierAdapter.java
│   │   └── api/
│   │       ├── AuthController.java
│   │       ├── UserController.java
│   │       ├── request/
│   │       │   ├── RegisterRequest.java
│   │       │   ├── LoginRequest.java
│   │       │   ├── VerifyEmailRequest.java
│   │       │   ├── RefreshTokenRequest.java
│   │       │   └── GoogleLoginRequest.java
│   │       └── mapper/
│   │           └── AuthApiMapper.java
│   │
│   ├── languagetrack/
│   │   ├── domain/
│   │   │   ├── LanguageTrack.java
│   │   │   ├── TrackStatus.java                    # enum: ACTIVE, PAUSED
│   │   │   ├── CefrLevel.java                       # shared value object, lives here or shared/domain
│   │   │   └── exception/DuplicateLanguageTrackException.java
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── CreateLanguageTrackCommand.java
│   │   │   │   ├── CreateLanguageTrackHandler.java
│   │   │   │   ├── SetSelfDeclaredLevelCommand.java
│   │   │   │   └── SetSelfDeclaredLevelHandler.java
│   │   │   ├── query/
│   │   │   │   ├── GetUserLanguageTracksQuery.java
│   │   │   │   └── GetUserLanguageTracksHandler.java
│   │   │   ├── port/
│   │   │   │   └── LanguageTrackRepository.java
│   │   │   └── dto/
│   │   │       └── LanguageTrackResponse.java
│   │   ├── infrastructure/
│   │   │   └── persistence/
│   │   │       ├── JpaLanguageTrackRepository.java
│   │   │       └── LanguageTrackRepositoryAdapter.java
│   │   └── api/
│   │       ├── LanguageTrackController.java
│   │       └── request/
│   │           ├── CreateLanguageTrackRequest.java
│   │           └── SetLevelRequest.java
│   │
│   ├── placement/
│   │   ├── domain/
│   │   │   ├── PlacementTest.java
│   │   │   ├── PlacementStatus.java                 # enum: IN_PROGRESS, COMPLETED
│   │   │   └── PlacementTranscriptEntry.java        # VO: { question, answer }
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── StartPlacementTestCommand.java
│   │   │   │   ├── StartPlacementTestHandler.java
│   │   │   │   ├── SubmitPlacementAnswerCommand.java
│   │   │   │   └── SubmitPlacementAnswerHandler.java
│   │   │   ├── query/
│   │   │   │   ├── GetPlacementResultQuery.java
│   │   │   │   └── GetPlacementResultHandler.java
│   │   │   ├── port/
│   │   │   │   ├── PlacementTestRepository.java
│   │   │   │   └── PlacementAiService.java          # interviews + returns CEFR level
│   │   │   └── dto/
│   │   │       ├── PlacementQuestionResponse.java
│   │   │       └── PlacementResultResponse.java
│   │   ├── infrastructure/
│   │   │   ├── persistence/
│   │   │   │   ├── JpaPlacementTestRepository.java
│   │   │   │   └── PlacementTestRepositoryAdapter.java
│   │   │   └── ai/
│   │   │       └── PlacementAiServiceAdapter.java   # Spring AI + structured output
│   │   └── api/
│   │       ├── PlacementController.java
│   │       └── request/SubmitAnswerRequest.java
│   │
│   ├── learningpath/
│   │   ├── domain/
│   │   │   ├── LearningPath.java
│   │   │   ├── LearningPathItem.java
│   │   │   ├── LearningPathStatus.java              # ACTIVE, SUPERSEDED
│   │   │   └── ContentType.java                      # VOCAB, GRAMMAR, SPEAKING, LISTENING
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── GenerateLearningPathCommand.java
│   │   │   │   └── GenerateLearningPathHandler.java
│   │   │   ├── query/
│   │   │   │   ├── GetActiveLearningPathQuery.java
│   │   │   │   └── GetActiveLearningPathHandler.java
│   │   │   ├── port/
│   │   │   │   ├── LearningPathRepository.java
│   │   │   │   └── LearningPathAiPlanner.java
│   │   │   └── dto/
│   │   │       └── LearningPathResponse.java
│   │   ├── infrastructure/
│   │   │   ├── persistence/
│   │   │   │   ├── JpaLearningPathRepository.java
│   │   │   │   └── LearningPathRepositoryAdapter.java
│   │   │   └── ai/
│   │   │       └── LearningPathAiPlannerAdapter.java
│   │   └── api/
│   │       └── LearningPathController.java
│   │
│   ├── vocabulary/
│   │   ├── domain/
│   │   │   └── VocabularyItem.java
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── MarkVocabularyLearnedCommand.java
│   │   │   │   └── MarkVocabularyLearnedHandler.java
│   │   │   ├── query/
│   │   │   │   ├── GetVocabularyItemQuery.java
│   │   │   │   ├── GetVocabularyItemHandler.java
│   │   │   │   ├── ListVocabularyByLevelQuery.java
│   │   │   │   └── ListVocabularyByLevelHandler.java
│   │   │   ├── port/
│   │   │   │   └── VocabularyRepository.java
│   │   │   └── dto/
│   │   │       └── VocabularyItemResponse.java
│   │   ├── infrastructure/
│   │   │   └── persistence/
│   │   │       ├── JpaVocabularyRepository.java
│   │   │       └── VocabularyRepositoryAdapter.java
│   │   └── api/
│   │       └── VocabularyController.java
│   │
│   ├── grammar/
│   │   ├── domain/
│   │   │   ├── GrammarTopic.java
│   │   │   └── ExplanationStyle.java                # enum: DEFAULT, ELI5, ARABIC, FOOTBALL, PROGRAMMING
│   │   ├── application/
│   │   │   ├── query/
│   │   │   │   ├── GetGrammarExplanationQuery.java
│   │   │   │   └── GetGrammarExplanationHandler.java
│   │   │   ├── port/
│   │   │   │   ├── GrammarTopicRepository.java
│   │   │   │   └── GrammarExplanationAiService.java
│   │   │   └── dto/
│   │   │       └── GrammarExplanationResponse.java
│   │   ├── infrastructure/
│   │   │   ├── persistence/
│   │   │   │   ├── JpaGrammarTopicRepository.java
│   │   │   │   └── GrammarTopicRepositoryAdapter.java
│   │   │   └── ai/
│   │   │       └── GrammarExplanationAiServiceAdapter.java
│   │   └── api/
│   │       └── GrammarController.java
│   │
│   ├── listening/
│   │   ├── domain/
│   │   │   └── ListeningClip.java
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── SubmitListeningAnswerCommand.java
│   │   │   │   └── SubmitListeningAnswerHandler.java
│   │   │   ├── query/
│   │   │   │   ├── GetListeningClipQuery.java
│   │   │   │   └── GetListeningClipHandler.java
│   │   │   ├── port/
│   │   │   │   └── ListeningClipRepository.java
│   │   │   └── dto/
│   │   │       └── ListeningClipResponse.java
│   │   ├── infrastructure/
│   │   │   └── persistence/
│   │   │       ├── JpaListeningClipRepository.java
│   │   │       └── ListeningClipRepositoryAdapter.java
│   │   └── api/
│   │       └── ListeningController.java
│   │
│   ├── submission/                                   # shared Submission/Assessment aggregate
│   │   ├── domain/
│   │   │   ├── Submission.java
│   │   │   ├── SubmissionType.java                   # WRITING, SPEAKING, QUIZ
│   │   │   ├── Assessment.java
│   │   │   └── AudioRef.java                         # value object
│   │   ├── application/
│   │   │   ├── port/
│   │   │   │   ├── SubmissionRepository.java
│   │   │   │   └── AssessmentRepository.java
│   │   │   └── dto/
│   │   │       └── AssessmentResponse.java
│   │   └── infrastructure/
│   │       └── persistence/
│   │           ├── JpaSubmissionRepository.java
│   │           ├── SubmissionRepositoryAdapter.java
│   │           ├── JpaAssessmentRepository.java
│   │           └── AssessmentRepositoryAdapter.java
│   │
│   ├── speaking/
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── SubmitSpeechCommand.java
│   │   │   │   └── SubmitSpeechHandler.java          # async: kicks off STT+scoring pipeline
│   │   │   ├── query/
│   │   │   │   ├── GetSpeakingSubmissionQuery.java
│   │   │   │   └── GetSpeakingSubmissionHandler.java
│   │   │   ├── port/
│   │   │   │   ├── SpeechToTextService.java
│   │   │   │   ├── PronunciationScorer.java
│   │   │   │   └── AudioStorage.java
│   │   │   └── event/
│   │   │       └── SpeechAssessedEvent.java          # triggers WebSocket push
│   │   ├── infrastructure/
│   │   │   ├── ai/
│   │   │   │   ├── WhisperSpeechToTextAdapter.java
│   │   │   │   └── AiPronunciationScorerAdapter.java
│   │   │   ├── storage/
│   │   │   │   └── S3AudioStorageAdapter.java        # or local disk in dev profile
│   │   │   └── websocket/
│   │   │       ├── SpeakingWebSocketConfig.java
│   │   │       └── SpeakingWebSocketHandler.java
│   │   └── api/
│   │       ├── SpeakingController.java
│   │       └── request/SubmitSpeechRequest.java       # multipart audio upload
│   │
│   ├── writing/
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── SubmitWritingCommand.java
│   │   │   │   └── SubmitWritingHandler.java
│   │   │   ├── query/
│   │   │   │   ├── GetWritingSubmissionQuery.java
│   │   │   │   └── GetWritingSubmissionHandler.java
│   │   │   ├── port/
│   │   │   │   └── WritingFeedbackAiService.java
│   │   │   └── dto/
│   │   │       └── WritingFeedbackResponse.java       # { corrections[], natural, formal, native }
│   │   ├── infrastructure/
│   │   │   └── ai/
│   │   │       └── WritingFeedbackAiServiceAdapter.java
│   │   └── api/
│   │       ├── WritingController.java
│   │       └── request/SubmitWritingRequest.java
│   │
│   ├── srs/
│   │   ├── domain/
│   │   │   ├── SrsCard.java
│   │   │   ├── ReviewGrade.java                       # enum: AGAIN, HARD, GOOD, EASY
│   │   │   └── SrsScheduler.java                       # domain service, SM-2 algorithm
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── ReviewSrsCardCommand.java
│   │   │   │   └── ReviewSrsCardHandler.java
│   │   │   ├── query/
│   │   │   │   ├── GetDueSrsCardsQuery.java
│   │   │   │   └── GetDueSrsCardsHandler.java
│   │   │   └── port/
│   │   │       └── SrsCardRepository.java
│   │   ├── infrastructure/
│   │   │   └── persistence/
│   │   │       ├── JpaSrsCardRepository.java
│   │   │       └── SrsCardRepositoryAdapter.java
│   │   └── api/
│   │       ├── SrsController.java
│   │       └── request/ReviewCardRequest.java
│   │
│   ├── progress/                                      # daily goal/progress + streak
│   │   ├── domain/
│   │   │   ├── DailyGoal.java
│   │   │   ├── DailyProgress.java
│   │   │   ├── StreakLedgerEntry.java
│   │   │   └── StreakCalculator.java                   # domain service, timezone-aware
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── SetDailyGoalCommand.java
│   │   │   │   ├── SetDailyGoalHandler.java
│   │   │   │   ├── RecordActivityCommand.java           # called internally by other features
│   │   │   │   └── RecordActivityHandler.java
│   │   │   ├── query/
│   │   │   │   ├── GetDailyProgressQuery.java
│   │   │   │   ├── GetDailyProgressHandler.java
│   │   │   │   ├── GetStreakQuery.java
│   │   │   │   └── GetStreakHandler.java
│   │   │   └── port/
│   │   │       ├── DailyGoalRepository.java
│   │   │       ├── DailyProgressRepository.java
│   │   │       └── StreakLedgerRepository.java
│   │   ├── infrastructure/
│   │   │   └── persistence/
│   │   │       ├── JpaDailyGoalRepository.java
│   │   │       ├── DailyGoalRepositoryAdapter.java
│   │   │       ├── JpaDailyProgressRepository.java
│   │   │       ├── DailyProgressRepositoryAdapter.java
│   │   │       ├── JpaStreakLedgerRepository.java
│   │   │       └── StreakLedgerRepositoryAdapter.java
│   │   └── api/
│   │       ├── ProgressController.java
│   │       └── request/SetGoalRequest.java
│   │
│   ├── conversation/                                   # Phase 2: chat + roleplay
│   │   ├── domain/
│   │   │   ├── ConversationSession.java
│   │   │   ├── ConversationMessage.java
│   │   │   └── Scenario.java                            # enum: AIRPORT, RESTAURANT, JOB_INTERVIEW, HOTEL, FREE_CHAT
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── StartConversationCommand.java
│   │   │   │   ├── StartConversationHandler.java
│   │   │   │   ├── SendMessageCommand.java
│   │   │   │   └── SendMessageHandler.java
│   │   │   ├── query/
│   │   │   │   ├── GetConversationQuery.java
│   │   │   │   └── GetConversationHandler.java
│   │   │   ├── port/
│   │   │   │   ├── ConversationSessionRepository.java
│   │   │   │   └── ConversationAiService.java            # persona-aware chat
│   │   │   └── dto/
│   │   │       └── ConversationMessageResponse.java
│   │   ├── infrastructure/
│   │   │   ├── persistence/
│   │   │   │   ├── JpaConversationSessionRepository.java
│   │   │   │   └── ConversationSessionRepositoryAdapter.java
│   │   │   └── ai/
│   │   │       └── ConversationAiServiceAdapter.java
│   │   └── api/
│   │       ├── ConversationController.java
│   │       └── request/SendMessageRequest.java
│   │
│   ├── mistakepattern/                                  # Phase 2: AI memory
│   │   ├── domain/
│   │   │   └── MistakePattern.java
│   │   ├── application/
│   │   │   ├── query/
│   │   │   │   ├── GetUserMistakePatternsQuery.java
│   │   │   │   └── GetUserMistakePatternsHandler.java
│   │   │   ├── port/
│   │   │   │   └── MistakePatternRepository.java
│   │   │   └── listener/
│   │   │       └── AssessmentCreatedListener.java        # updates patterns on new Assessment
│   │   └── infrastructure/
│   │       └── persistence/
│   │           ├── JpaMistakePatternRepository.java
│   │           └── MistakePatternRepositoryAdapter.java
│   │
│   ├── weaknessanalyzer/                                 # Phase 2: scheduled analysis
│   │   ├── domain/
│   │   │   └── ProgressSnapshot.java
│   │   ├── application/
│   │   │   ├── job/
│   │   │   │   └── WeeklyWeaknessAnalysisJob.java         # @Scheduled, chunked per-user
│   │   │   ├── query/
│   │   │   │   ├── GetLatestSnapshotQuery.java
│   │   │   │   └── GetLatestSnapshotHandler.java
│   │   │   └── port/
│   │   │       └── ProgressSnapshotRepository.java
│   │   ├── infrastructure/
│   │   │   └── persistence/
│   │   │       ├── JpaProgressSnapshotRepository.java
│   │   │       └── ProgressSnapshotRepositoryAdapter.java
│   │   └── api/
│   │       └── WeaknessController.java
│   │
│   ├── contentgen/                                       # Phase 2: vocab/story/quiz generators + translate
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   ├── GenerateVocabularyCommand.java
│   │   │   │   ├── GenerateVocabularyHandler.java
│   │   │   │   ├── GenerateStoryCommand.java
│   │   │   │   ├── GenerateStoryHandler.java
│   │   │   │   ├── GenerateQuizCommand.java
│   │   │   │   └── GenerateQuizHandler.java
│   │   │   ├── query/
│   │   │   │   ├── TranslateInContextQuery.java
│   │   │   │   └── TranslateInContextHandler.java
│   │   │   ├── port/
│   │   │   │   ├── VocabularyGeneratorAiService.java
│   │   │   │   ├── StoryGeneratorAiService.java
│   │   │   │   ├── QuizGeneratorAiService.java
│   │   │   │   └── ContextTranslatorAiService.java
│   │   │   └── dto/
│   │   │       ├── GeneratedStoryResponse.java
│   │   │       └── GeneratedQuizResponse.java
│   │   ├── infrastructure/
│   │   │   └── ai/
│   │   │       ├── VocabularyGeneratorAiServiceAdapter.java
│   │   │       ├── StoryGeneratorAiServiceAdapter.java
│   │   │       ├── QuizGeneratorAiServiceAdapter.java
│   │   │       └── ContextTranslatorAiServiceAdapter.java
│   │   └── api/
│   │       ├── ContentGenController.java
│   │       └── TranslateController.java
│   │
│   ├── gamification/
│   │   ├── domain/
│   │   │   ├── XpLedgerEntry.java
│   │   │   ├── Achievement.java
│   │   │   ├── AchievementRule.java
│   │   │   └── UserAchievement.java
│   │   ├── application/
│   │   │   ├── command/
│   │   │   │   └── AwardXpCommand.java + AwardXpHandler.java   # append-only ledger write
│   │   │   ├── query/
│   │   │   │   ├── GetXpBalanceQuery.java
│   │   │   │   ├── GetXpBalanceHandler.java
│   │   │   │   ├── GetLeaderboardQuery.java
│   │   │   │   └── GetLeaderboardHandler.java
│   │   │   ├── port/
│   │   │   │   ├── XpLedgerRepository.java
│   │   │   │   ├── AchievementRepository.java
│   │   │   │   └── LeaderboardStore.java                      # Redis sorted-set port
│   │   │   └── listener/
│   │   │       └── AchievementEvaluationListener.java          # on xp/streak ledger writes
│   │   ├── infrastructure/
│   │   │   ├── persistence/
│   │   │   │   ├── JpaXpLedgerRepository.java
│   │   │   │   ├── XpLedgerRepositoryAdapter.java
│   │   │   │   ├── JpaAchievementRepository.java
│   │   │   │   └── AchievementRepositoryAdapter.java
│   │   │   └── cache/
│   │   │       └── RedisLeaderboardStoreAdapter.java
│   │   └── api/
│   │       └── GamificationController.java
│   │
│   └── social/
│       ├── domain/
│       │   ├── Friendship.java
│       │   ├── FriendshipStatus.java                        # PENDING, ACCEPTED, BLOCKED
│       │   └── Challenge.java
│       ├── application/
│       │   ├── command/
│       │   │   ├── SendFriendRequestCommand.java + Handler
│       │   │   ├── RespondFriendRequestCommand.java + Handler
│       │   │   └── CreateChallengeCommand.java + Handler
│       │   ├── query/
│       │   │   ├── ListFriendsQuery.java + Handler
│       │   │   └── ListChallengesQuery.java + Handler
│       │   └── port/
│       │       ├── FriendshipRepository.java
│       │       └── ChallengeRepository.java
│       ├── infrastructure/
│       │   └── persistence/
│       │       ├── JpaFriendshipRepository.java
│       │       ├── FriendshipRepositoryAdapter.java
│       │       ├── JpaChallengeRepository.java
│       │       └── ChallengeRepositoryAdapter.java
│       └── api/
│           └── SocialController.java
│
├── src/main/resources/
│   ├── application.yml
│   ├── application-local.yml
│   ├── application-docker.yml
│   ├── application-prod.yml
│   ├── db/migration/
│   │   ├── V1__init_users.sql
│   │   ├── V2__init_language_tracks.sql
│   │   ├── V3__init_placement_tests.sql
│   │   ├── V4__init_vocabulary_and_grammar.sql
│   │   ├── V5__init_learning_paths.sql
│   │   ├── V6__init_listening_clips.sql
│   │   ├── V7__init_submissions_and_assessments.sql
│   │   ├── V8__init_srs_cards.sql
│   │   ├── V9__init_daily_goal_progress_streak.sql
│   │   ├── V10__init_xp_ledger.sql
│   │   ├── V11__init_conversations.sql
│   │   ├── V12__init_mistake_patterns.sql
│   │   ├── V13__init_progress_snapshots.sql
│   │   ├── V14__init_achievements.sql
│   │   └── V15__init_social.sql
│   └── prompts/
│       ├── placement-interview.st
│       ├── learning-path-generation.st
│       ├── grammar-explanation.st
│       ├── writing-feedback.st
│       ├── pronunciation-feedback.st
│       ├── conversation-persona.st
│       ├── vocabulary-generation.st
│       ├── story-generation.st
│       ├── quiz-generation.st
│       └── translate-context.st
│
└── src/test/java/com/yourorg/langplatform/
    ├── architecture/
    │   └── LayerDependencyRulesTest.java              # ArchUnit — enforces §1.2 matrix
    ├── shared/
    │   └── ai/StructuredOutputParserTest.java
    ├── auth/
    │   ├── domain/UserTest.java
    │   ├── application/RegisterUserHandlerTest.java
    │   ├── application/LoginHandlerTest.java
    │   ├── infrastructure/UserRepositoryAdapterIT.java   # Testcontainers
    │   └── api/AuthControllerIT.java                     # full register→verify→login
    ├── languagetrack/
    │   └── application/CreateLanguageTrackHandlerTest.java
    ├── placement/
    │   └── application/SubmitPlacementAnswerHandlerTest.java   # incl. malformed-AI-response fallback
    ├── learningpath/
    │   └── application/GenerateLearningPathHandlerTest.java    # idempotency test
    ├── srs/
    │   ├── domain/SrsSchedulerTest.java                          # SM-2 math
    │   └── infrastructure/SrsCardRepositoryAdapterIT.java        # due-query index test
    ├── progress/
    │   ├── domain/StreakCalculatorTest.java                      # timezone edge cases
    │   └── application/RecordActivityHandlerConcurrencyTest.java # ledger race condition
    ├── gamification/
    │   └── application/AwardXpHandlerConcurrencyTest.java
    └── ai/
        └── PlacementAiServiceAdapterLiveIT.java   # @Tag("ai-integration"), excluded from default run
```

### Notes on conventions used above

- **`XxxRepository` (in `application/port`) vs `JpaXxxRepository` + `XxxRepositoryAdapter` (in `infrastructure/persistence`)** — the port is the interface Application code depends on; `JpaXxxRepository` is the raw Spring Data interface; `XxxRepositoryAdapter` implements the port and delegates to the Spring Data interface (or wraps native/JPQL projection queries for read-heavy cases like the dashboard, per §3.1 of the architecture plan).
- **AI ports** (`PlacementAiService`, `WritingFeedbackAiService`, etc.) each get exactly one adapter that calls the shared `AiClientConfig` bean + `PromptTemplateLoader` + `StructuredOutputParser` — the AI plumbing itself lives once in `shared/ai`, never duplicated per feature.
- **`submission/`** is a small shared package (not a full feature) because `Submission`/`Assessment` are genuinely reused by `listening`, `speaking`, and `writing` — giving each its own copy would duplicate the aggregate; this is the one intentional exception to strict per-feature ownership, called out so it doesn't look like an inconsistency.
- Files written as `XxxCommand.java + Handler` in the social package listing are shorthand for two separate files — every other package spells both out in full; kept social terse since it's the lowest-risk, last-built feature per §13 of the architecture plan.