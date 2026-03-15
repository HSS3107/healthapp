<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# share the whole document again

Here is the full document:

***

# Data Model \& Schema Document V4.1

## AI-Powered Health Pattern Intelligence App — v1.0

**Aligned with:** Solution Doc v4.1 · Product Experience Spec v4.1 · UI/UX Design Doc v4.1
**Status:** Source of Truth for Engineering
**Database:** PostgreSQL 15 + TimescaleDB · Neon (ap-south-1, Mumbai)
**Date:** March 8, 2026

***

## Table of Contents

1. [Design Principles](#1-design-principles)
2. [Entity Map](#2-entity-map)
3. [Database Setup](#3-database-setup)
4. [Enum Types](#4-enum-types)
5. [Table Schemas](#5-table-schemas)
    - 5.1 users
    - 5.2 user_goals
    - 5.3 daily_logs (Hypertable)
    - 5.4 log_revisions
    - 5.5 photo_metadata
    - 5.6 derived_metrics
    - 5.7 pattern_insights
    - 5.8 weekly_wraps
    - 5.9 experiments
    - 5.10 user_behavioral_state
    - 5.11 idempotency_keys
    - 5.12 audit_events
6. [JSONB Structure Reference](#6-jsonb-structure-reference)
7. [TimescaleDB Configuration](#7-timescaledb-configuration)
8. [Indexes](#8-indexes)
9. [Row-Level Security (RLS)](#9-row-level-security-rls)
10. [Data Retention Rules](#10-data-retention-rules)
11. [v4.1 Migration Checklist](#11-v41-migration-checklist)
12. [Business \& Validation Rules](#12-business--validation-rules)
13. [LangGraph State Management](#13-langgraph-state-management)
14. [Connection Configuration](#14-connection-configuration)

***

## 1. Design Principles

These principles govern every schema decision. Do not deviate.

1. **Goal-agnostic.** `goal_type` is a free-text parameter, never an enum constrained to fat-loss values. No column, constraint, or default may assume a specific goal type.
2. **Log immutability.** `daily_logs` rows are never updated or deleted. Corrections create rows in `log_revisions`. The source log record remains intact.
3. **Client-authoritative timestamps.** `log_date` (the user's local date at time of typing) is the display and analytics date. `log_ts` (server receipt time) is the time-series partition key. Never conflate them.
4. **Idempotency by design.** Every log write uses a `client_log_id` (UUID generated on-device). The offline queue may retry. The backend deduplicates via `idempotency_keys`.
5. **Phase-aware, not goal-percentage.** Progress is always expressed in phase terms. No column stores "percent to end goal." Phase context is the frame.
6. **Relapse never resets.** Phase revision extends and recalibrates. It never resets or deletes phase history. JSONB phase entries accumulate; they are not overwritten.
7. **Sparse-data safety.** Pattern insights are gated by `data_maturity_days ≥ 14` AND `confidence_adjusted ≥ 0.5`. These gates are enforced in application logic, not DB constraints — but the fields that power them must always be populated.
8. **Single active goal.** A user may have only one goal with `is_active = TRUE` at any time. Enforced by partial unique index.

***

## 2. Entity Map

```
users
  └── user_goals (1:many, 1 active at a time)
        ├── phase_journey []  ← JSONB array embedded in user_goals
        └── experiments (1:many)

users
  └── daily_logs (1:many, TimescaleDB hypertable)
        ├── log_revisions (1:many)
        └── photo_metadata (0:1)

users
  └── derived_metrics (1:many, nightly computed)

users
  └── pattern_insights (1:many)
        └── weekly_wraps.pattern_ids []  ← FK reference in wrap JSONB

users
  └── weekly_wraps (1:many)
        └── experiments (1:1 proposed_from_wrap_id)

users
  └── user_behavioral_state (1:1)

users
  └── idempotency_keys (1:many)

users
  └── audit_events (1:many)

LangGraph PostgresSaver (managed externally)
  └── checkpoints, checkpoint_writes, checkpoint_migrations
```


***

## 3. Database Setup

### 3.1 Extensions

```sql
-- Run against DATABASE_URL_DIRECT only
CREATE EXTENSION IF NOT EXISTS timescaledb;
CREATE EXTENSION IF NOT EXISTS pgcrypto;      -- gen_random_uuid()
CREATE EXTENSION IF NOT EXISTS pg_trgm;       -- future: text search on food_description
```


### 3.2 Environment Variables

| Variable | Connection Type | Used By |
| :-- | :-- | :-- |
| `DATABASE_URL` | Pooled (PgBouncer, statement-level) | API/Hono service, Pattern Engine, nightly cron |
| `DATABASE_URL_DIRECT` | Direct (non-pooled) | Migrations (drizzle-kit), LangGraph PostgresSaver |

> **Critical:** LangGraph's PostgresSaver uses advisory locks that are incompatible with PgBouncer statement-level pooling. Always pass `DATABASE_URL_DIRECT` to the LangGraph checkpointer. Use `DATABASE_URL` everywhere else.

### 3.3 Migration Runner

```bash
npx drizzle-kit migrate --url=$DATABASE_URL_DIRECT
```


***

## 4. Enum Types

```sql
-- Log types
CREATE TYPE log_type_enum AS ENUM (
  'weight',
  'food',
  'context',
  'photo'
);
-- Note: retro-logs are not a separate type. is_retro = TRUE flags them.

-- Phase status (on user_goals)
CREATE TYPE phase_status_enum AS ENUM (
  'on_track',
  'running_late',
  'relapsed',
  'completed_early',
  'completed'
);

-- Insight domain
CREATE TYPE insight_domain_enum AS ENUM (
  'weight',
  'food',
  'supplement',
  'sleep',
  'training',
  'general'
);

-- Insight status
CREATE TYPE insight_status_enum AS ENUM (
  'active',
  'surfaced',
  'acknowledged',
  'dismissed',
  'expired'
);

-- Experiment status
CREATE TYPE experiment_status_enum AS ENUM (
  'proposed',
  'acknowledged',
  'active',
  'completed',
  'abandoned'
);

-- Wrap status
CREATE TYPE wrap_status_enum AS ENUM (
  'generating',
  'delivered',
  'read',
  'failed'
);

-- Engagement trajectory (on user_behavioral_state)
CREATE TYPE engagement_trajectory_enum AS ENUM (
  'rising',
  'stable',
  'declining',
  'inactive'
);

-- Audit event type
CREATE TYPE audit_event_type_enum AS ENUM (
  'onboarding_completed',
  'phase_1_locked',
  'log_submitted',
  'micro_hook_replied',
  'outlier_context_provided',
  'insight_surfaced',
  'insight_acknowledged',
  'insight_dismissed',
  'wrap_opened',
  'experiment_acknowledged',
  'experiment_completed',
  'general_health_chat_initiated',
  'signal_extraction_nudge_shown',
  'signal_extraction_confirmed',
  'phase_completed_early',
  'phase_recalibrated',
  'relapse_detected',
  'relapse_wrap_opened',
  'wrap_generation_failed'     -- DLQ entry
);
```


***

## 5. Table Schemas

### 5.1 `users`

Stores identity, authentication state, and baseline physical stats captured at onboarding.

```sql
CREATE TABLE users (
  id                    UUID          PRIMARY KEY DEFAULT gen_random_uuid(),

  -- Auth
  phone_number          TEXT          NOT NULL UNIQUE,   -- OTP auth identifier
  display_name          TEXT,                            -- Optional; user can set post-onboarding

  -- Baseline stats (captured in onboarding Step 1)
  height_cm             NUMERIC(5,1),                   -- e.g., 175.5
  body_fat_pct          NUMERIC(4,1),                   -- Optional; user-provided estimate

  -- Onboarding state
  onboarding_complete   BOOLEAN       NOT NULL DEFAULT FALSE,
  onboarding_step       SMALLINT      NOT NULL DEFAULT 1 CHECK (onboarding_step BETWEEN 1 AND 5),
  -- Persists step progress so onboarding resumes mid-flow on app reopen

  -- Soft delete
  deleted_at            TIMESTAMPTZ,

  created_at            TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
  updated_at            TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);

-- Trigger: auto-update updated_at
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN NEW.updated_at = NOW(); RETURN NEW; END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_updated_at
  BEFORE UPDATE ON users
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

**Notes:**

- `deleted_at IS NULL` filter must be applied by all application queries. Soft-delete only.
- `onboarding_step` is the resumption pointer. The Onboarding Tool reads this from `AgentState` (synced from DB) on every session start.
- `height_cm` and `body_fat_pct` are captured once at onboarding. Future edits go through a profile update flow (post-MVP).

***

### 5.2 `user_goals`

Stores the user's stated goal, the AI-generated phase journey roadmap, and current phase tracking state. One active goal per user at any time.

```sql
CREATE TABLE user_goals (
  id                      UUID              PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id                 UUID              NOT NULL REFERENCES users(id) ON DELETE CASCADE,

  -- Goal definition (user-stated, free text — never constrained to fat-loss)
  goal_text               TEXT              NOT NULL,
  -- e.g., "Lose 10kg, improve energy levels"
  goal_type               TEXT,
  -- Parameterized label, e.g., "weight_loss", "muscle_gain", "energy"
  -- NEVER an enum. Never hard-coded. Set by AI during onboarding.

  -- Baseline weight at goal creation (from onboarding Step 1)
  baseline_weight_kg      NUMERIC(5,1),

  -- AI-computed target rate
  target_rate_weekly      NUMERIC(4,2),
  -- kg/week (or equivalent unit for non-weight goals)
  physiological_flag      BOOLEAN           NOT NULL DEFAULT FALSE,
  -- TRUE if AI flagged goal implies unsafe rate (>1.5% body weight/week)
  -- User may proceed regardless — flag is informational only

  -- Phase journey (see Section 6.1 for JSONB structure)
  phase_journey           JSONB             NOT NULL DEFAULT '[]'::JSONB,
  current_phase_index     SMALLINT          NOT NULL DEFAULT 0,
  -- 0-based index into phase_journey array

  -- Current phase state (v4.1)
  phase_status            phase_status_enum NOT NULL DEFAULT 'on_track',
  focus_behavior          TEXT,
  -- Primary behavior to watch in current phase (e.g., "caloric consistency")

  -- Onboarding context (from Steps 3 & 4)
  context_signals         TEXT[]            NOT NULL DEFAULT '{}',
  -- e.g., {"stress_eating","travel","poor_sleep"}
  lifestyle_context       TEXT,
  -- Free-text from onboarding Step 4 (e.g., "shift work, irregular sleep")

  -- Goal active state
  is_active               BOOLEAN           NOT NULL DEFAULT TRUE,
  locked_at               TIMESTAMPTZ,
  -- Timestamp when Phase 1 was locked (end of onboarding)

  created_at              TIMESTAMPTZ       NOT NULL DEFAULT NOW(),
  updated_at              TIMESTAMPTZ       NOT NULL DEFAULT NOW()
);

CREATE TRIGGER user_goals_updated_at
  BEFORE UPDATE ON user_goals
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();

-- Enforce single active goal per user
CREATE UNIQUE INDEX user_goals_single_active
  ON user_goals (user_id)
  WHERE is_active = TRUE;
```

**Notes:**

- `goal_type` is free-text. Do not add a CHECK constraint or enum. Valid examples: `"weight_loss"`, `"muscle_gain"`, `"energy_optimization"`, `"recovery"`, `"general_wellness"`.
- `phase_journey` is a JSONB array. Each element represents one phase. See Section 6.1 for the full structure. Phase entries are **appended and updated in-place** — never deleted.
- `current_phase_index` is the pointer to the active phase. Updated by the Orchestrator on phase transition.
- `phase_status` reflects the state of the **current** phase, not the overall goal.
- Phase recalibration updates `phase_journey` JSONB projected dates AND `phase_status`. Both must be updated atomically in a single transaction.

***

### 5.3 `daily_logs` (TimescaleDB Hypertable)

The core telemetry table. Immutable. Every log submission creates a new row. Corrections create rows in `log_revisions`. Partitioned by `log_ts`.

```sql
CREATE TABLE daily_logs (
  id                            UUID              PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id                       UUID              NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  goal_id                       UUID              REFERENCES user_goals(id) ON DELETE SET NULL,
  -- Active goal at time of log. SET NULL if goal deleted (data preserved).

  -- Idempotency
  client_log_id                 UUID              NOT NULL UNIQUE,
  -- Generated on-device. Prevents duplicate inserts from offline queue retries.

  -- Log classification
  log_type                      log_type_enum     NOT NULL,

  -- Timestamps
  log_date                      DATE              NOT NULL,
  -- Client-authoritative display date. User's local date at time of input.
  log_ts                        TIMESTAMPTZ       NOT NULL DEFAULT NOW(),
  -- Server receipt time. Time-series partition key for TimescaleDB.

  -- Retro-log flag
  is_retro                      BOOLEAN           NOT NULL DEFAULT FALSE,
  -- TRUE when log_date < CURRENT_DATE

  -- Weight log fields (log_type = 'weight')
  weight_kg                     NUMERIC(5,1),

  -- Food log fields (log_type = 'food' or 'photo')
  food_description              TEXT,
  -- Natural language, e.g., "Big Mac meal + large Coke"
  calories_estimated            INTEGER,
  -- AI estimate (backend only, never from client)
  protein_estimated_g           INTEGER,
  food_source                   TEXT,
  -- 'manual' | 'photo_ai' | 'conversation'

  -- Context log fields (log_type = 'context')
  context_tags                  TEXT[]            NOT NULL DEFAULT '{}',
  -- Predefined: stress_high, sleep_low, alcohol, workout, sick, supplement_start
  -- Extended: travel, social_event, weekend, late_night, binge, work_travel, family_event
  -- Open: any lowercase_snake_case tag derived by the Logging Tool
  context_note                  TEXT,
  -- Free-text note (e.g., "started creatine 5g/day")

  -- Photo log fields (log_type = 'photo')
  photo_id                      UUID              REFERENCES photo_metadata(id) ON DELETE SET NULL,

  -- Outlier detection (v4.1)
  outlier_flag                  BOOLEAN           NOT NULL DEFAULT FALSE,
  -- TRUE when log deviates significantly from 7-day rolling average
  outlier_context_response      TEXT,
  -- User's response to the outlier context prompt

  -- Signal extraction (v4.1)
  signal_extraction_nudge_shown BOOLEAN           NOT NULL DEFAULT FALSE,
  signal_extraction_confirmed   BOOLEAN           NOT NULL DEFAULT FALSE,
  -- TRUE when user tapped "Yes, log it" on the inline confirmation nudge

  -- Phase context at time of log
  phase_index_at_log            SMALLINT,
  -- Snapshot of current_phase_index at moment of logging

  created_at                    TIMESTAMPTZ       NOT NULL DEFAULT NOW()
  -- No updated_at — daily_logs is immutable
);
```

**Convert to TimescaleDB hypertable** (run after table creation):

```sql
SELECT create_hypertable(
  'daily_logs',
  'log_ts',
  chunk_time_interval => INTERVAL '7 days'
);
```

**Notes:**

- The table is **append-only**. No UPDATE or DELETE from application code — ever.
- `log_date` is used for all user-facing display, rolling averages, and pattern detection. `log_ts` is only the partition key.
- `context_tags` uses an open taxonomy. Do not restrict with a CHECK constraint.
- `calories_estimated` and `protein_estimated_g` are populated by AI backend only. May be NULL.
- Multi-log messages create **multiple rows**, one per `log_type`, all from the same device submission.

***

### 5.4 `log_revisions`

Stores correction history when a user revises a previously submitted log. The original `daily_logs` row is never modified.

```sql
CREATE TABLE log_revisions (
  id                UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
  original_log_id   UUID          NOT NULL REFERENCES daily_logs(id) ON DELETE CASCADE,
  user_id           UUID          NOT NULL REFERENCES users(id) ON DELETE CASCADE,

  -- Snapshot of changed fields (only fields that changed are populated)
  revised_weight_kg         NUMERIC(5,1),
  revised_food_description  TEXT,
  revised_calories          INTEGER,
  revised_protein_g         INTEGER,
  revised_context_tags      TEXT[],
  revised_context_note      TEXT,

  revision_reason           TEXT,
  revised_at                TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);
```

**Notes:**

- Application always reads the **latest revision** when displaying a log to the user.
- The Pattern Engine uses original log values for historical trend integrity.

***

### 5.5 `photo_metadata`

Stores metadata for food photo uploads. The image binary lives in Cloudflare R2. This table holds the reference and AI analysis output.

```sql
CREATE TABLE photo_metadata (
  id                    UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id               UUID          NOT NULL REFERENCES users(id) ON DELETE CASCADE,

  -- R2 reference
  r2_object_key         TEXT          NOT NULL UNIQUE,
  -- e.g., "photos/user_uuid/2026-03-08_uuid.webp"
  original_filename     TEXT,
  size_bytes            INTEGER,
  mime_type             TEXT          NOT NULL DEFAULT 'image/webp',

  -- AI analysis output (populated asynchronously)
  ai_food_description   TEXT,
  ai_calories_estimated INTEGER,
  ai_protein_estimated_g INTEGER,
  analysis_status       TEXT          NOT NULL DEFAULT 'pending',
  -- 'pending' | 'completed' | 'failed'
  analysis_error        TEXT,

  uploaded_at           TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
  expires_at            TIMESTAMPTZ   NOT NULL DEFAULT (NOW() + INTERVAL '7 days'),
  -- 7-day retention. After this, R2 object is deleted, row is soft-archived.

  created_at            TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);
```

**Notes:**

- Photos are uploaded via `POST /photos` before the log is submitted. The pipeline is async.
- After `expires_at`, R2 object is deleted. DB row is preserved (set `analysis_status = 'archived'`). Food description and estimates are retained for pattern analysis.

***

### 5.6 `derived_metrics`

Stores nightly-computed rolling averages, adherence scores, and phase progress. Written by the Pattern Engine cron (nightly at 02:00 UTC).

```sql
CREATE TABLE derived_metrics (
  id                    UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id               UUID          NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  goal_id               UUID          REFERENCES user_goals(id) ON DELETE SET NULL,
  computed_at           DATE          NOT NULL,
  -- Date this snapshot was computed for (typically yesterday's date)

  -- Rolling averages (7-day window ending on computed_at)
  weight_7d_avg         NUMERIC(5,2),     -- NULL if <3 weight logs in window
  weight_7d_variance    NUMERIC(5,3),
  weight_7d_trend       NUMERIC(5,2),     -- Change vs. 7 days prior (positive = gaining)
  calorie_7d_avg        NUMERIC(7,1),     -- NULL if <3 food logs in window
  protein_7d_avg        NUMERIC(6,1),

  -- Adherence (last 7 days)
  adherence_score       NUMERIC(3,2)      CHECK (adherence_score BETWEEN 0 AND 1),
  days_logged_last_7    SMALLINT,

  -- Phase progress
  phase_progress        NUMERIC(5,2),
  -- Ratio of phase target achieved (0.0–1.0+). Can exceed 1.0 (early completion territory).
  phase_days_elapsed    SMALLINT,
  phase_days_total      SMALLINT,

  -- Data maturity
  data_maturity_days    INTEGER           NOT NULL DEFAULT 0,
  sparse_data_flag      BOOLEAN           NOT NULL DEFAULT TRUE,
  -- TRUE if data_maturity_days < 14 OR adherence_score < 0.43 (3/7 days)

  -- Tag day counts (last 7 days) — see Section 6.6 for JSONB structure
  tag_day_counts        JSONB             NOT NULL DEFAULT '{}'::JSONB,
  -- e.g., {"stress_high": 4, "alcohol": 2, "workout": 5}

  -- Outlier flag for the period
  outlier_detected      BOOLEAN           NOT NULL DEFAULT FALSE,
  outlier_log_date      DATE,

  created_at            TIMESTAMPTZ       NOT NULL DEFAULT NOW(),

  UNIQUE (user_id, computed_at)
);
```

**Notes:**

- `weight_7d_avg` is NULL if fewer than 3 weight logs exist in the window. **Wrap Tool must handle NULL gracefully**: write "No reliable weight trend this week — weigh-ins were inconsistent" — never "weight changed NULL kg."
- `data_maturity_days` and `sparse_data_flag` are read by the Orchestrator on every AgentState hydration.
- Future: upgrade to TimescaleDB continuous aggregates. For MVP, nightly cron is sufficient.

***

### 5.7 `pattern_insights`

Stores detected behavioral correlations produced by the Pattern Engine. Each row is one detected pattern.

```sql
CREATE TABLE pattern_insights (
  id                  UUID                  PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             UUID                  NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  goal_id             UUID                  REFERENCES user_goals(id) ON DELETE SET NULL,

  -- Pattern content
  domain              insight_domain_enum   NOT NULL,
  -- weight | food | supplement | sleep | training | general

  phase_context       TEXT,
  -- Phase name/description at time of detection (v4.1)
  -- e.g., "Phase 1: Caloric consistency, Day 14"
  -- Text snapshot — not a FK. Survives phase transitions.

  correlation_description TEXT              NOT NULL,
  -- Human-readable pattern
  -- e.g., "Nights marked stress_high are followed by higher average calories"

  -- Evidence (see Section 6.5 for JSONB structure)
  evidence            JSONB,
  evidence_window_days INTEGER             NOT NULL,

  -- Confidence
  base_confidence     NUMERIC(3,2)          NOT NULL CHECK (base_confidence BETWEEN 0 AND 1),
  confidence_adjusted NUMERIC(3,2)          NOT NULL CHECK (confidence_adjusted BETWEEN 0 AND 1),
  -- Formula: MIN(base_confidence × (data_maturity_days / 14), 1.0)
  -- Surfaceable only when confidence_adjusted >= 0.5

  -- Lifecycle
  status              insight_status_enum   NOT NULL DEFAULT 'active',
  surfaced_at         TIMESTAMPTZ,
  acknowledged_at     TIMESTAMPTZ,
  dismissed_at        TIMESTAMPTZ,

  weekly_wrap_id      UUID                  REFERENCES weekly_wraps(id) ON DELETE SET NULL,

  created_at          TIMESTAMPTZ           NOT NULL DEFAULT NOW(),
  updated_at          TIMESTAMPTZ           NOT NULL DEFAULT NOW()
);

CREATE TRIGGER pattern_insights_updated_at
  BEFORE UPDATE ON pattern_insights
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

**Surfacing gate (enforced in application code):**

```
Insight is surfaceable when:
  confidence_adjusted >= 0.5
  AND data_maturity_days >= 14       -- from derived_metrics
  AND status = 'active'
  AND insight_fatigue_suppression = FALSE  -- from user_behavioral_state
```


***

### 5.8 `weekly_wraps`

Stores generated weekly narrative wraps. One per user per week. Generated by the Wrap Tool cron (Mondays, 04:00 UTC).

```sql
CREATE TABLE weekly_wraps (
  id                  UUID              PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             UUID              NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  goal_id             UUID              REFERENCES user_goals(id) ON DELETE SET NULL,

  -- Week window
  week_start_date     DATE              NOT NULL,   -- Monday
  week_end_date       DATE              NOT NULL,   -- Sunday

  -- Wrap content (see Section 6.2–6.4 for JSONB structures)
  phase_snapshot      JSONB             NOT NULL DEFAULT '{}'::JSONB,
  progress_snapshot   JSONB             NOT NULL DEFAULT '{}'::JSONB,

  pattern_ids         UUID[]            NOT NULL DEFAULT '{}',
  -- pattern_insights.id references (1–2 max per wrap)

  outlier_narrative   TEXT,
  -- (v4.1) Forensic explanation of any outlier. NULL if none occurred.

  suggested_experiment JSONB            NOT NULL DEFAULT '{}'::JSONB,
  -- See Section 6.3 for structure

  chat_prompt          TEXT,
  -- (v4.1) 1-line re-engagement observation. Must never be NULL for a delivered wrap.

  -- Relapse state (v4.1)
  relapse_detected    BOOLEAN           NOT NULL DEFAULT FALSE,
  relapse_narrative   TEXT,

  -- Lifecycle
  status              wrap_status_enum  NOT NULL DEFAULT 'generating',
  generated_at        TIMESTAMPTZ,
  delivered_at        TIMESTAMPTZ,
  read_at             TIMESTAMPTZ,

  generation_error    TEXT,
  -- Populated if generation failed after retries

  created_at          TIMESTAMPTZ       NOT NULL DEFAULT NOW(),
  updated_at          TIMESTAMPTZ       NOT NULL DEFAULT NOW(),

  UNIQUE (user_id, week_start_date)
);

CREATE TRIGGER weekly_wraps_updated_at
  BEFORE UPDATE ON weekly_wraps
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

**Notes:**

- `chat_prompt` must never be NULL for `status = 'delivered'`. Block delivery if NULL and log an error.
- When generation fails after 2 retries: write `audit_events` row (`wrap_generation_failed`), set `status = 'failed'`, move on. Do not block other users.
- `pattern_ids` stores FK references only — wrap content does not duplicate the full insight text.

***

### 5.9 `experiments`

Tracks the experiment recommended in each Weekly Wrap and the user's follow-through. Enables narrative continuity ("Last week you tried X — here's what happened").

```sql
CREATE TABLE experiments (
  id                    UUID                    PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id               UUID                    NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  goal_id               UUID                    REFERENCES user_goals(id) ON DELETE SET NULL,
  weekly_wrap_id        UUID                    REFERENCES weekly_wraps(id) ON DELETE SET NULL,

  -- Experiment definition
  domain                insight_domain_enum     NOT NULL,
  description           TEXT                    NOT NULL,
  measurable_via        TEXT                    NOT NULL,
  -- 'weight' | 'food_log' | 'context_tag' | 'conversation'
  duration_days         SMALLINT                DEFAULT 7,

  -- Lifecycle
  status                experiment_status_enum  NOT NULL DEFAULT 'proposed',
  proposed_at           DATE                    NOT NULL,
  acknowledged_at       TIMESTAMPTZ,
  completed_at          TIMESTAMPTZ,
  abandoned_at          TIMESTAMPTZ,
  outcome_note          TEXT,

  created_at            TIMESTAMPTZ             NOT NULL DEFAULT NOW(),
  updated_at            TIMESTAMPTZ             NOT NULL DEFAULT NOW()
);

CREATE TRIGGER experiments_updated_at
  BEFORE UPDATE ON experiments
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

**Notes:**

- Only one active experiment per user at a time. Enforced in application logic.
- The Orchestrator injects the active experiment into `AgentState` on every session start.
- `measurable_via` must be one of the existing log types — experiment must be trackable without new infrastructure.

***

### 5.10 `user_behavioral_state`

One row per user. Live behavioral and engagement state used by the Orchestrator to govern AI responses.

```sql
CREATE TABLE user_behavioral_state (
  id                          UUID                          PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id                     UUID                          NOT NULL REFERENCES users(id) ON DELETE CASCADE UNIQUE,

  -- Data maturity (mirrored from latest derived_metrics for fast Orchestrator reads)
  data_maturity_days          INTEGER                       NOT NULL DEFAULT 0,
  sparse_data_flag            BOOLEAN                       NOT NULL DEFAULT TRUE,

  -- Insight fatigue
  insight_fatigue_suppression BOOLEAN                       NOT NULL DEFAULT FALSE,
  -- Set TRUE when insights_shown_last_7_days >= 2. Reset every Monday.
  last_insight_surfaced_at    TIMESTAMPTZ,
  insights_shown_last_7_days  SMALLINT                      NOT NULL DEFAULT 0,

  -- Engagement
  adherence_score             NUMERIC(3,2),
  engagement_trajectory       engagement_trajectory_enum    NOT NULL DEFAULT 'stable',
  last_log_date               DATE,
  days_since_last_log         SMALLINT GENERATED ALWAYS AS (
                                CASE
                                  WHEN last_log_date IS NULL THEN NULL
                                  ELSE (CURRENT_DATE - last_log_date)::SMALLINT
                                END
                              ) STORED,
  consecutive_logging_days    SMALLINT                      NOT NULL DEFAULT 0,

  -- Relapse state (v4.1)
  relapse_flag                BOOLEAN                       NOT NULL DEFAULT FALSE,
  relapse_detected_at         TIMESTAMPTZ,

  -- Re-engagement gate
  re_entry_prompt_sent_at     TIMESTAMPTZ,
  -- Orchestrator must NOT send another re-entry prompt within 7 days of this timestamp.

  updated_at                  TIMESTAMPTZ                   NOT NULL DEFAULT NOW()
);

CREATE TRIGGER user_behavioral_state_updated_at
  BEFORE UPDATE ON user_behavioral_state
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

**Notes:**

- `days_since_last_log` is a computed column — never update it manually.
- `relapse_flag` is set by the nightly Pattern Engine when the 5-day rolling average reversal criterion is met. Cleared after the next wrap delivers the relapse narrative.
- `insight_fatigue_suppression` hard cap: no more than 2 insights in any 7-day window per user.

***

### 5.11 `idempotency_keys`

Deduplicates log writes from the offline queue. The client generates a UUID per log at time of input.

```sql
CREATE TABLE idempotency_keys (
  id                UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
  client_log_id     UUID          NOT NULL UNIQUE,
  user_id           UUID          NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  resolved_log_id   UUID          REFERENCES daily_logs(id) ON DELETE SET NULL,
  status            TEXT          NOT NULL DEFAULT 'pending',
  -- 'pending' | 'processed' | 'failed'
  created_at        TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
  expires_at        TIMESTAMPTZ   NOT NULL DEFAULT (NOW() + INTERVAL '72 hours')
);
```

**API-layer logic:**

```
On POST /logs:
  1. Check idempotency_keys WHERE client_log_id = $1
  2. IF status = 'processed' → return 200 with original log (no duplicate write)
  3. IF status = 'pending'   → proceed with write, update status to 'processed'
  4. IF not found            → insert key as 'pending', proceed with write
```


***

### 5.12 `audit_events`

Lightweight event log for analytics instrumentation and Wrap Tool DLQ.

```sql
CREATE TABLE audit_events (
  id            UUID                    PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID                    REFERENCES users(id) ON DELETE SET NULL,
  event_type    audit_event_type_enum   NOT NULL,
  metadata      JSONB                   NOT NULL DEFAULT '{}'::JSONB,
  -- Event-specific payload examples:
  -- log_submitted:           {"log_type": "weight", "is_retro": false}
  -- wrap_generation_failed:  {"week_start": "2026-03-02", "error": "timeout"}
  -- phase_recalibrated:      {"old_end_date": "2026-04-07", "new_end_date": "2026-04-14"}
  created_at    TIMESTAMPTZ             NOT NULL DEFAULT NOW()
);
```

**Notes:**

- Append-only. No updates or deletes.
- For Wrap Tool DLQ: write `wrap_generation_failed` row with full error context in `metadata`. Cron moves on to the next user.

***

## 6. JSONB Structure Reference

These are binding contracts. API and AI tools must produce exactly these structures.

### 6.1 `user_goals.phase_journey` (Array)

```json
[
  {
    "phase_index": 0,
    "phase_name": "Phase 1",
    "focus_area": "Caloric consistency and baseline logging",
    "target_metric": {
      "field": "weight_kg",
      "direction": "decrease",
      "change_amount": 2.0,
      "unit": "kg"
    },
    "estimated_duration_days": 30,
    "projected_start_date": "2026-03-08",
    "projected_end_date": "2026-04-07",
    "actual_start_date": "2026-03-08",
    "actual_end_date": null,
    "completed_early": false,
    "actual_completion_date": null,
    "phase_status": "on_track",
    "recalibration_notes": []
  },
  {
    "phase_index": 1,
    "phase_name": "Phase 2",
    "focus_area": "Stress-eating pattern management",
    "target_metric": {
      "field": "weight_kg",
      "direction": "decrease",
      "change_amount": 3.0,
      "unit": "kg"
    },
    "estimated_duration_days": 35,
    "projected_start_date": "2026-04-08",
    "projected_end_date": "2026-05-13",
    "actual_start_date": null,
    "actual_end_date": null,
    "completed_early": false,
    "actual_completion_date": null,
    "phase_status": "on_track",
    "recalibration_notes": []
  }
]
```

**Field rules:**

- `target_metric.field` is free text — not an enum. Can be `"weight_kg"`, `"energy_level"`, `"avg_calories"`, etc.
- `recalibration_notes` is an append-only array. Each recalibration appends (never overwrites): `"Recalibrated 2026-03-28: running 8 days late. New end date: 2026-04-15"`.
- All future phases shift `projected_start_date` and `projected_end_date` atomically when any upstream phase is recalibrated.

***

### 6.2 `weekly_wraps.phase_snapshot`

```json
{
  "phase_name": "Phase 1",
  "phase_index": 0,
  "days_elapsed": 14,
  "days_total_estimated": 30,
  "phase_status": "on_track",
  "trend_vs_target": "on_pace",
  "weight_change_this_week_kg": -0.4,
  "projected_completion_date": "2026-04-07",
  "recalibrated": false,
  "recalibration_message": null,
  "narrative": "Phase 1, Day 14 of ~30. You're at -1.2kg against a -2kg target by April 7. Right on pace.",
  "phase_transition_acknowledged": false
}
```


***

### 6.3 `weekly_wraps.suggested_experiment`

```json
{
  "domain": "supplement",
  "description": "Add 5g creatine monohydrate to your morning routine for 7 days. You've been logging stress_high 4x this week and creatine has evidence for cognitive stress resilience. Log it as a context tag so we can watch for weight and energy pattern changes.",
  "measurable_via": "context_tag",
  "tag_to_log": "supplement_start",
  "duration_days": 7,
  "proposed_start_date": "2026-03-09"
}
```

**Field rules:**

- `domain` must use one of the `insight_domain_enum` values.
- `measurable_via` must be one of: `"weight"`, `"food_log"`, `"context_tag"`, `"conversation"`.
- `tag_to_log` is optional — only present when `measurable_via = 'context_tag'`.

***

### 6.4 `weekly_wraps.progress_snapshot`

```json
{
  "weight_7d_avg_kg": 91.3,
  "weight_change_vs_phase_start_kg": -1.2,
  "calorie_7d_avg": 1980,
  "protein_7d_avg_g": 142,
  "adherence_score": 0.86,
  "days_logged": 6,
  "consistency_description": "6 of 7 days logged",
  "goal_progress_description": "1.2kg of 2kg Phase 1 target"
}
```

**Null-safety rule:** If `weight_7d_avg_kg` is NULL, `goal_progress_description` must read `"Weigh-ins were inconsistent this week — no reliable trend."` Never `"null kg of 2kg target"`.

***

### 6.5 `pattern_insights.evidence`

```json
{
  "sample_size": 21,
  "tag_a": "stress_high",
  "tag_b": null,
  "metric": "calories_estimated",
  "tagged_day_avg": 2360,
  "untagged_day_avg": 1980,
  "difference": 380,
  "unit": "kcal",
  "window_start_date": "2026-02-15",
  "window_end_date": "2026-03-08",
  "plain_language_summary": "Based on 21 days of logs, days tagged stress_high are followed by an average of 380 kcal more than stress-free days — confidence 0.73."
}
```


***

### 6.6 `derived_metrics.tag_day_counts`

```json
{
  "stress_high": 4,
  "alcohol": 2,
  "workout": 5,
  "sleep_low": 3,
  "travel": 1
}
```

Only tags that appeared at least once in the 7-day window are included.

***

## 7. TimescaleDB Configuration

### 7.1 Hypertable Setup

```sql
SELECT create_hypertable(
  'daily_logs',
  'log_ts',
  chunk_time_interval => INTERVAL '7 days'
);
```


### 7.2 Compression Policy

```sql
ALTER TABLE daily_logs SET (
  timescaledb.compress,
  timescaledb.compress_segmentby = 'user_id'
);

SELECT add_compression_policy('daily_logs', INTERVAL '30 days');
```


### 7.3 Retention Policy (audit_events)

```sql
SELECT create_hypertable('audit_events', 'created_at', chunk_time_interval => INTERVAL '30 days');
SELECT add_retention_policy('audit_events', INTERVAL '1 year');
```


### 7.4 Pattern Engine Query (7-day rolling average)

```sql
-- Nightly cron: compute weight_7d_avg for a user
SELECT
  user_id,
  AVG(weight_kg) FILTER (WHERE weight_kg IS NOT NULL)   AS weight_7d_avg,
  STDDEV(weight_kg) FILTER (WHERE weight_kg IS NOT NULL) AS weight_7d_variance,
  COUNT(DISTINCT log_date) FILTER (WHERE log_type = 'weight') AS weight_log_count
FROM daily_logs
WHERE
  user_id = $1
  AND log_type = 'weight'
  AND log_date BETWEEN CURRENT_DATE - 7 AND CURRENT_DATE - 1
GROUP BY user_id;
```


***

## 8. Indexes

```sql
-- users
CREATE INDEX idx_users_phone ON users (phone_number);
CREATE INDEX idx_users_deleted ON users (deleted_at) WHERE deleted_at IS NULL;

-- user_goals
CREATE INDEX idx_user_goals_user_active ON user_goals (user_id) WHERE is_active = TRUE;

-- daily_logs
CREATE INDEX idx_daily_logs_user_date ON daily_logs (user_id, log_date DESC);
CREATE INDEX idx_daily_logs_user_type_date ON daily_logs (user_id, log_type, log_date DESC);
CREATE INDEX idx_daily_logs_goal_phase ON daily_logs (goal_id, phase_index_at_log);

-- derived_metrics
CREATE INDEX idx_derived_metrics_user_date ON derived_metrics (user_id, computed_at DESC);

-- pattern_insights
CREATE INDEX idx_pattern_insights_user_status ON pattern_insights (user_id, status)
  WHERE status = 'active';
CREATE INDEX idx_pattern_insights_user_domain ON pattern_insights (user_id, domain);

-- weekly_wraps
CREATE INDEX idx_weekly_wraps_user_week ON weekly_wraps (user_id, week_start_date DESC);
CREATE INDEX idx_weekly_wraps_status ON weekly_wraps (status) WHERE status = 'delivered';

-- experiments
CREATE INDEX idx_experiments_user_status ON experiments (user_id, status)
  WHERE status IN ('proposed', 'acknowledged', 'active');

-- idempotency_keys
CREATE INDEX idx_idempotency_keys_expires ON idempotency_keys (expires_at)
  WHERE status = 'pending';

-- audit_events
CREATE INDEX idx_audit_events_user_type ON audit_events (user_id, event_type, created_at DESC);
CREATE INDEX idx_audit_events_type_date ON audit_events (event_type, created_at DESC);
```


***

## 9. Row-Level Security (RLS)

All tables must have RLS enabled. Users may only read and write their own data. Service-role bypasses RLS for backend operations.

```sql
-- Enable RLS on all user-data tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_goals ENABLE ROW LEVEL SECURITY;
ALTER TABLE daily_logs ENABLE ROW LEVEL SECURITY;
ALTER TABLE log_revisions ENABLE ROW LEVEL SECURITY;
ALTER TABLE photo_metadata ENABLE ROW LEVEL SECURITY;
ALTER TABLE derived_metrics ENABLE ROW LEVEL SECURITY;
ALTER TABLE pattern_insights ENABLE ROW LEVEL SECURITY;
ALTER TABLE weekly_wraps ENABLE ROW LEVEL SECURITY;
ALTER TABLE experiments ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_behavioral_state ENABLE ROW LEVEL SECURITY;
ALTER TABLE idempotency_keys ENABLE ROW LEVEL SECURITY;
ALTER TABLE audit_events ENABLE ROW LEVEL SECURITY;

-- Policy template (repeat for each table, adjusting table/column names)

CREATE POLICY daily_logs_user_isolation ON daily_logs
  FOR ALL
  USING (user_id = auth.uid());
-- auth.uid() resolves to the UUID from the JWT issued after OTP verification.

CREATE POLICY users_self_only ON users
  FOR ALL
  USING (id = auth.uid());
```

**Notes:**

- Backend services (Hono API, Pattern Engine, LangGraph) connect as the service role and bypass RLS automatically.
- Never expose the service role connection string to the client.

***

## 10. Data Retention Rules

| Data | Retention | Policy |
| :-- | :-- | :-- |
| `daily_logs` | Indefinite | Never deleted except on account deletion |
| `log_revisions` | Indefinite | Follows parent `daily_logs` lifetime |
| `photo_metadata` (row) | Indefinite | Row preserved; R2 object deleted after 7 days |
| R2 photo objects | 7 days | R2 lifecycle rule + `expires_at` cleanup cron |
| `derived_metrics` | 2 years | TimescaleDB retention policy |
| `pattern_insights` | Indefinite | Preserved for longitudinal intelligence |
| `weekly_wraps` | Indefinite | Full wrap history preserved |
| `experiments` | Indefinite | Required for narrative continuity |
| `audit_events` | 1 year | TimescaleDB retention policy |
| `idempotency_keys` | 72 hours | `expires_at` cleanup job runs hourly |
| LangGraph `checkpoints` | Last 50 messages (MVP) | Managed by PostgresSaver config |

**Account deletion:** When `users.deleted_at` is set, a background job cascades deletion of all user data including R2 objects. A 30-day soft-delete window is provided before hard deletion executes.

***

## 11. v4.1 Migration Checklist

All items are **additions** to the v4.0 schema. No existing columns are removed or modified.

```sql
-- 1. user_goals: add phase_status, lifestyle_context, context_signals
ALTER TABLE user_goals
  ADD COLUMN IF NOT EXISTS phase_status phase_status_enum NOT NULL DEFAULT 'on_track',
  ADD COLUMN IF NOT EXISTS lifestyle_context TEXT,
  ADD COLUMN IF NOT EXISTS context_signals TEXT[] NOT NULL DEFAULT '{}';

-- 2. user_goals: backfill new JSONB fields on existing phase_journey entries
-- (adds completed_early, actual_completion_date, recalibration_notes to each phase object)
UPDATE user_goals
SET phase_journey = (
  SELECT jsonb_agg(
    phase || jsonb_build_object(
      'completed_early', false,
      'actual_completion_date', null,
      'recalibration_notes', '[]'::jsonb
    )
  )
  FROM jsonb_array_elements(phase_journey) AS phase
)
WHERE phase_journey != '[]'::jsonb;

-- 3. daily_logs: add outlier and signal extraction fields
ALTER TABLE daily_logs
  ADD COLUMN IF NOT EXISTS outlier_flag BOOLEAN NOT NULL DEFAULT FALSE,
  ADD COLUMN IF NOT EXISTS outlier_context_response TEXT,
  ADD COLUMN IF NOT EXISTS signal_extraction_nudge_shown BOOLEAN NOT NULL DEFAULT FALSE,
  ADD COLUMN IF NOT EXISTS signal_extraction_confirmed BOOLEAN NOT NULL DEFAULT FALSE,
  ADD COLUMN IF NOT EXISTS phase_index_at_log SMALLINT;

-- 4. pattern_insights: add domain and phase_context
ALTER TABLE pattern_insights
  ADD COLUMN IF NOT EXISTS domain insight_domain_enum NOT NULL DEFAULT 'general',
  ADD COLUMN IF NOT EXISTS phase_context TEXT;

-- 5. weekly_wraps: add outlier_narrative, chat_prompt, relapse fields
ALTER TABLE weekly_wraps
  ADD COLUMN IF NOT EXISTS outlier_narrative TEXT,
  ADD COLUMN IF NOT EXISTS chat_prompt TEXT,
  ADD COLUMN IF NOT EXISTS relapse_detected BOOLEAN NOT NULL DEFAULT FALSE,
  ADD COLUMN IF NOT EXISTS relapse_narrative TEXT;

-- 6. user_behavioral_state: add relapse fields
ALTER TABLE user_behavioral_state
  ADD COLUMN IF NOT EXISTS relapse_flag BOOLEAN NOT NULL DEFAULT FALSE,
  ADD COLUMN IF NOT EXISTS relapse_detected_at TIMESTAMPTZ;

-- 7. Confirm goal_type has no CHECK constraint restricting to fat-loss values
-- DROP CONSTRAINT IF EXISTS goal_type_check on user_goals (if it exists from v4.0)
```


***

## 12. Business \& Validation Rules

Enforced in **application code**, not DB constraints, unless noted.

### Logging

- A log write must always include a `client_log_id`. Reject `POST /logs` without one.
- `log_date` must not be in the future. Reject future dates.
- `log_date` can be up to 30 days in the past (retro-log window). Set `is_retro = TRUE`.
- `weight_kg` must be between 20.0 and 500.0 if provided.
- Multiple logs of different `log_type` may share the same `log_date` — this is valid and expected.
- Multi-log messages create multiple rows, one per `log_type`, all from the same device submission.


### Phase Revision

- Relapse classification is always based on the **7-day rolling average**, never a single log.
- `+0.5–1kg` overnight after a tagged event = normal variance. Outlier prompt only. No phase impact.
- Phase revision never resets a phase. It extends `projected_end_date` and appends to `recalibration_notes`.
- **Early completion**: detected by the Orchestrator in real-time on the log that crosses the threshold. Inline chat message sent immediately. Do NOT hold for the next wrap.
- **Delayed completion**: detected by the Wrap Tool. Recalibration and notification happen in the wrap, not mid-week.
- `phase_status` on `user_goals` and the `phase_status` within the JSONB entry must be updated in a single transaction.


### Pattern Insights

- Surfacing gate: `confidence_adjusted >= 0.5` AND `data_maturity_days >= 14`. Both required.
- Insight fatigue cap: no more than 2 insights surfaced per user in any 7-day window.
- Confidence formula: `confidence_adjusted = MIN(base_confidence × (data_maturity_days / 14), 1.0)`


### Weekly Wraps

- One wrap per user per week. `UNIQUE (user_id, week_start_date)` enforced at DB level.
- `chat_prompt` must never be NULL for a delivered wrap. Block delivery if NULL.
- Wrap cron: Mondays 04:00 UTC. Pattern Engine cron: nightly 02:00 UTC.
- Wrap failure after 2 retries: write `audit_events` row, set `status = 'failed'`, continue to next user.


### Re-engagement

- If `days_since_last_log >= 3`: check `AgentState` for illness/life-event context first.
- If no illness context: send **one** re-entry prompt. Set `re_entry_prompt_sent_at`.
- Do not send another re-entry prompt within 7 days. Check `re_entry_prompt_sent_at`.
- Never use shame-based language. Never reference missed streaks.


### Goal-Agnostic Invariant

- `goal_type` must never be stored as a hard-coded fat-loss value.
- No `DEFAULT 'fat_loss'`. No CHECK constraint restricting to specific goal types.
- All pattern detection, phase framing, and insight templates must parameterize on the user's actual `goal_text` and `goal_type`.

***

## 13. LangGraph State Management

LangGraph's `PostgresSaver` manages its own checkpoint tables in the same Neon database. Do not define or migrate these manually.

### LangGraph-Managed Tables

| Table | Purpose |
| :-- | :-- |
| `checkpoints` | Graph state snapshots per thread |
| `checkpoint_writes` | Intermediate write operations |
| `checkpoint_migrations` | Schema version tracking |

### AgentState Contract

Fields from application tables hydrated into `AgentState` on every session start:

```typescript
interface AgentState {
  // From users
  user_id: string;
  onboarding_complete: boolean;
  onboarding_step: number;             // 1–5, for resumption

  // From user_goals (active goal)
  goal_id: string | null;
  goal_text: string | null;
  goal_type: string | null;
  phase_journey: PhaseEntry[];         // Full JSONB array (v4.1)
  current_phase_index: number;
  current_phase: PhaseEntry | null;    // phase_journey[current_phase_index]
  phase_status: string | null;         // (v4.1)
  focus_behavior: string | null;
  baseline_weight_kg: number | null;

  // From user_behavioral_state
  data_maturity_days: number;
  sparse_data_flag: boolean;
  insight_fatigue_suppression: boolean;
  relapse_flag: boolean;               // (v4.1)
  days_since_last_log: number | null;
  engagement_trajectory: string;

  // From derived_metrics (latest row)
  weight_7d_avg: number | null;
  weight_7d_trend: number | null;
  adherence_score: number | null;
  phase_progress: number | null;
  tag_day_counts: Record<string, number>;

  // From experiments (active experiment)
  active_experiment: ExperimentSummary | null;

  // Chat messages (last 10, in-memory)
  messages: ChatMessage[];

  // Compact recent log summary (from derived_metrics — NOT raw log rows)
  recent_logs_summary: RecentLogsSummary | null;
}
```


### PostgresSaver Setup

```python
# Always use DATABASE_URL_DIRECT for LangGraph checkpointer
from langgraph.checkpoint.postgres import PostgresSaver

checkpointer = PostgresSaver.from_conn_string(os.getenv("DATABASE_URL_DIRECT"))
```

**Thread ID convention:** `thread_id = user_id`. One persistent conversation thread per user.

***

## 14. Connection Configuration

```bash
# .env (production)

# Pooled — for all application code
DATABASE_URL="postgresql://user:password@ep-xxx.ap-south-1.aws.neon.tech/neondb?sslmode=require"

# Direct — for migrations and LangGraph PostgresSaver only
DATABASE_URL_DIRECT="postgresql://user:password@ep-xxx.ap-south-1.aws.neon.tech/neondb?sslmode=require&pgbouncer=false"
```

| Service | Connection |
| :-- | :-- |
| Hono API server | `DATABASE_URL` (pooled) |
| Nightly Pattern Engine cron | `DATABASE_URL` (pooled) |
| Weekly Wrap cron | `DATABASE_URL` (pooled) |
| Drizzle migration runner | `DATABASE_URL_DIRECT` |
| LangGraph `PostgresSaver` | `DATABASE_URL_DIRECT` |


***

*Document version: v1.0 · March 8, 2026*
*Aligned with: Solution Doc v4.1 · Product Experience Spec v4.1 · UI/UX Design Doc v4.1*
*Companion documents: API Contracts v1.2 · AI Agents \& Prompt Engineering v2.0*

