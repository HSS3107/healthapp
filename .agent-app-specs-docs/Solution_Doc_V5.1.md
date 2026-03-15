
# Technical Solutions Document — AI-Powered Health Pattern Intelligence App
## Version 5.1

*Prepared by: Solutions Architect*
*For: Harshit Shishodia, Founder*
*Date: March 14, 2026*
*Status: Source of Truth for Engineering, Design & AI Teams*
*Supersedes: Technical Solutions Document v5 (March 8, 2026)*

---
## Changelog v5 → v5.1

   ADDED PART 13 - MONOREPO SCAFFOLD

## Changelog v4.1 → v5.0

| # | Change | Reason |
|---|--------|--------|
| 1 | **Corrected Section 4** — Changed "Add micro_hook field to weekly_wraps" to "Add chat_prompt field to weekly_wraps" | chat_prompt is the stored field; micro_hook is a transient Logging Tool API response field only, never stored in DB |
| 2 | **Corrected Section 6.3 Wrap Tool Output Spec** — Changed stored fields from "micro_hook" to "chat_prompt" | Alignment with data model — chat_prompt is the wrap-to-chat bridge question stored in weekly_wraps |
| 3 | **Added AI Team Alignment Item for Wrap Tool** — "Wrap Tool must generate chat_prompt" with full specification | Explicit requirement for wrap-to-chat conversion mechanic |
| 4 | **Added chat_prompt_tapped to Section 7 Analytics Events** | Measures wrap-to-chat conversion rate |
| 5 | **Corrected References Section** — Changed "Data Model Schema Document v3.0" to "Data Model & Schema Document v1.1" | Accurate version reference from current session |
| 6 | **Preserved Logging Tool micro_hook specification** | Intentional and correct — micro_hook is the 1-line AI observation ending every log response (API field, not DB field) |
| 7 | **Preserved micro_hook_replied analytics event** | Tracks chat engagement mechanic from Logging Tool responses |
| 8 | **Preserved micro_hook reply rate ≥ 30% MVP metric** | Correct success metric for compulsive value delivery validation |

---

## Document Purpose

This document is the primary technical reference for building the MVP of the AI-powered health pattern intelligence app. It translates the **Product Experience Spec v4.1** into high-level engineering decisions, architecture, and product constraints.

**This is version 5.0 — a complete, standalone source of truth.** All corrections from v4.1 have been integrated. No need to reference previous versions.

**Note on Companion Documents:** To keep this architectural overview digestible, the exact database schemas, API endpoint payloads, and AI prompt engineering details have been decoupled. This document outlines the strategy for those systems and provides strict direction for the companion documents that must be updated for the dev teams.

---

# PART 1 — PRODUCT CONTEXT & PRINCIPLES

## 1.1 What We Are Building

This is not a calorie tracker, a fitness tracker, or a motivational app. It is a **personal pattern-intelligence system** — an AI companion that detects non-obvious behavioral correlations from a user's own logs and explains them in plain language.

**Strategic position:** Bad behaviors dissolve when a person clearly understands their own patterns. The system's job is to surface non-obvious roadblocks (e.g., *"Thursday stress + alcohol is your real plateau driver"*) and help the user run tightly scoped experiments. Behavior change follows understanding.

**Core loop:** Roadmap → Log → Detect → Explain → Experiment → Repeat

The journey map generated at onboarding is the first hook. Every subsequent log adds data. The pattern engine detects signal. The AI explains it. An experiment is proposed. The wrap closes the loop. Repeat weekly.

---

## 1.2 Target Persona

**"Dev" — The Skeptical Optimizer** (MVP test persona)
- Age 28–35, tech-savvy professional, based in Pune, India
- Seeks root-cause insights, not motivation slogans
- Wants candid, data-backed feedback — zero shame, zero fluff

**The persona is anyone.** Dev is a proxy. The system must serve:
- A 40-year-old woman managing perimenopause symptoms and energy levels
- A 22-year-old gym-goer optimizing a creatine + protein supplement stack
- A 55-year-old recovering from knee surgery, managing weight during limited mobility
- A 30-year-old with PCOS adjusting diet to manage insulin sensitivity

What unifies them is not demographics — it is the type of intelligence they need: pattern detection, honest explanation, goal-aware roadmapping, and a partner they can ask anything.

**Design implication:** Never hard-code goal types, tag vocabularies, or insight templates around fat loss. Every system must be parameterized by the user's actual stated goal and context.

---

## 1.3 MVP Success Metrics

| Metric | Target |
|--------|--------|
| Onboarding completion rate | ≥ 90% |
| Phase 1 lock rate | ≥ 80% (user sees and acknowledges phase journey before exiting onboarding) |
| Users receiving first pattern insight by Day 14 | ≥ 70% |
| Weekly Wrap open rate | ≥ 50% |
| Logging consistency (active users) | > 5 days/week |
| Micro-hook reply rate | ≥ 30% (user responds to or engages with a micro-hook within 24 hours) |

---

## 1.4 Non-Negotiable Product Principles

\begin{enumerate}
\item \textbf{Unified experience:} Chat for actions and logs, insights for reflection, profile for control.
\item \textbf{Reality over perfection:} Logs are immutable. Corrections create revisions, not erasures.
\item \textbf{Explainable intelligence:} Every insight must be traceable to observable evidence.
\item \textbf{Trust-first tone:} One unified AI voice. Candid, analytical, and never shaming.
\item \textbf{One experiment at a time:} The system never prescribes more than one testable action per week.
\item \textbf{User Sovereignty:} The user sets their own goals. The AI illuminates the gap, maps the phases, and surfaces what the data shows — never gates or prescribes what is "realistic."
\item \textbf{Holistic Partnership:} The AI engages across the full spectrum of health — diet, training, supplements, recovery, illness, mental state — because patterns don't respect category boundaries.
\item \textbf{Compulsive Value Delivery ("Every Log Is a Hit"):} Every single interaction must feel worth it — not through gamification or streaks, but through intelligence. If a user logs their weight and gets back \textit{"Logged 92kg."} — that is a failure. They must always get something back that makes them think.
\end{enumerate}

---

## 1.5 MVP Feature Scope — What Is IN Scope

\begin{itemize}
\item \textbf{Conversational Onboarding with Phase Journey Generation} — AI-led 5-step flow to collect baseline stats, accept user-stated goal, capture context signals, and generate + lock a multi-phase journey roadmap before logging begins.
\item \textbf{Unified Chat Interface} — The sole UI for text logging, querying insights, general health conversations, and profile commands.
\item \textbf{Frictionless Logging} — Natural language parsing for weight, food, and context tags, plus rough AI estimation for food photos. Retro-logging supported.
\item \textbf{Micro-Hook Engine} — Every log response ends with a 1-line AI observation or question. Not a notification — a message hook that drives re-engagement through intelligence.
\item \textbf{Outlier Context Prompt} — When a log deviates significantly from trend, the AI asks a single, non-judgmental context question. The answer becomes a tagged behavioral signal.
\item \textbf{Offline-First Sync Engine} — Local IndexedDB queuing for logs and photos, automatically syncing when connectivity returns.
\item \textbf{Nightly Pattern Engine} — Background cron job computing 7-day rolling averages and tag-based correlations. Goal-agnostic by design.
\item \textbf{Weekly Insight Wraps} — 7-day generated narrative: phase snapshot, progress snapshot, 1–2 strongest patterns, outlier narrative, and one recommended experiment.
\item \textbf{Phase Journey View} — Visual arc from onboarding to current phase to goal. Replaces progress percentage bar.
\item \textbf{Minimal Profile Control} — Phase journey view, data export, and account deletion.
\end{itemize}

---

## 1.6 What Is Explicitly OUT of MVP Scope

The following must not be designed, built, or partially implemented in the MVP:

\begin{itemize}
\item Intelligent content/knowledge feed
\item Bloodwork or medical report analysis
\item Advanced macro grading or nutrition scoring
\item Before/after photo simulations
\item Third-party integrations (wearables, Apple Health, etc.)
\item Structured supplement tracking (dedicated log type, dose/timing schema)
\item Structured training log (sets, reps, progressive overload)
\item Structured illness/symptom logging
\item Personalized supplement recommendation engine
\item Multi-variable predictive AI models
\item Tone/notification customization settings
\item Yearly wrap
\end{itemize}

**In MVP but conversational only (no structured extraction):**
General health conversations via chat (supplement questions, training discussions, illness context) — these are in MVP as conversational AI responses, without dedicated agents or structured data extraction.

---

# PART 2 — SOLUTION APPROACH & SYSTEM ARCHITECTURE

## 2.1 Architecture Philosophy

\begin{itemize}
\item \textbf{Mobile-first, browser-based Progressive Web App (PWA)} via Next.js. Instant deployment, zero install friction.
\item \textbf{Chat-first interaction model.} Intent routing via the backend to a LangGraph orchestrator.
\item \textbf{Offline-tolerant by design.} Logs captured locally first and background-synced.
\item \textbf{Goal-agnostic data model.} No hard-coded fat-loss logic anywhere in the stack — goal type is a parameter, not a branch.
\item \textbf{Phase-aware intelligence.} All pattern detection, progress reporting, and AI responses are framed relative to the user's current phase, not an absolute end goal.
\end{itemize}

---

## 2.2 High-Level System Architecture

┌─────────────────────────────────────────────────────────┐
│            MOBILE-FIRST PWA (Next.js 15)                │
│         Installed via browser · Runs offline            │
│              Push notifications                         │
│                                                         │
│           SINGLE CHAT INTERFACE                         │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTPS / REST / WebSocket
                       ▼
┌─────────────────────────────────────────────────────────┐
│              API GATEWAY (Node.js · Hono)               │
└───────┬───────────────────────────────┬─────────────────┘
        │                               │
        ▼                               ▼
┌───────────────┐             ┌─────────────────────┐
│   LangGraph   │             │   Pattern Engine    │
│  Orchestrator │             │   (Nightly Cron)    │
│               │             │                     │
│ · Onboarding  │             │ · 7-day rolling avg │
│ · Logging     │             │ · Tag correlations  │
│ · Insight     │             │ · Outlier detection │
│ · Wrap (cron) │             │ · Phase-aware score │
│ · Tone Format │             └─────────┬───────────┘
└───────┬───────┘                       │
        │                               │
        ▼                               ▼
┌─────────────────────────────────────────────────────────┐
│         PostgreSQL 15 + TimescaleDB · Neon              │
│  users · daily_logs · derived_metrics · experiments     │
│  pattern_insights · weekly_wraps · behavioral_state     │
└─────────────────────────────────────────────────────────┘
                       │
                       ▼
            ┌──────────────────┐
            │  Cloudflare R2   │
            │  (Photo storage) │
            └──────────────────┘

---

# PART 3 — TECHNOLOGY STACK

\begin{table}[h]
\begin{tabular}{lll}
\hline
\textbf{Layer} & \textbf{Technology} & \textbf{Rationale} \\
\hline
Frontend & Next.js 15 (App Router), Tailwind CSS, Serwist PWA, Dexie.js & Offline-first PWA; zero install friction \\
Backend & Node.js + TypeScript, Hono & Lightweight, fast edge-deployable API layer \\
AI Orchestration & LangGraph + GPT-4o (pinned versions) & Deep agent architecture with persistent memory \\
Database & PostgreSQL 15 + TimescaleDB via Neon (ap-south-1, Mumbai) & Time-series optimisation; lowest latency for IN user base \\
File Storage & Cloudflare R2 & Photo uploads; 7-day retention policy \\
Deployment & Google Cloud Run & Containerised, auto-scaling, aligns with LangGraph deployment \\
Observability & Sentry + custom analytics instrumentation & Error tracking, AI-specific metrics, retention signals \\
\hline
\end{tabular}
\end{table}

---

# PART 4 — DATA MODEL STRATEGY

The app relies on a structured, relational memory system rather than opaque vector databases. The data model must support strict log immutability, time-series analytics, phase-aware progress tracking, and longitudinal behavioral intelligence.

## Conceptual Entities

\begin{enumerate}
\item \textbf{User Identity \& Goals} — \texttt{users}, \texttt{user\_goals} (goal is user-stated free text; \texttt{goal\_type} is a parameterized field, never hard-coded)
\item \textbf{Phase Journey} — stored on \texttt{user\_goals} as \texttt{phase\_journey} (array of phases), \texttt{current\_phase}, \texttt{focus\_behavior}, \texttt{lifestyle\_context}
\item \textbf{Telemetry} — \texttt{daily\_logs} (immutable, with revision tracking), \texttt{idempotency\_keys}, \texttt{photo\_metadata}
\item \textbf{Derived Analytics} — \texttt{derived\_metrics} (rolling averages, variance, adherence), TimescaleDB continuous aggregates
\item \textbf{Intelligence} — \texttt{pattern\_insights} (detected correlations with confidence score, evidence window, domain tag), \texttt{weekly\_wraps} (narrative snapshots with phase snapshot + outlier narrative)
\item \textbf{Behavioral Memory} — \texttt{user\_behavioral\_state} (adherence, engagement trajectory, sparse-data flags), \texttt{experiments} (suggested interventions + follow-through for narrative continuity)
\item \textbf{Chat History} — \texttt{AgentState} (last N messages in MVP); post-MVP: full persistent \texttt{chat\_history} with semantic search for long-range referencing
\end{enumerate}

**Architecture note:** General health conversations (supplement questions, illness context, training discussions) are stored in `ChatHistory` / `AgentState`, not as `daily_logs`. Post-MVP: convert key conversation signals into structured tags via a Conversation Parser.

### Action Required — Companion Document

> Refer to **Data Model & Schema Document v1.1** for the complete PostgreSQL/TimescaleDB DDL schemas, indexing strategy, RLS policies, migration plan, and data retention rules.
>
> **v5.0 alignment items for Data Model team:**
> 
> \begin{itemize}
> \item Add \texttt{phase\_journey} (JSONB array), \texttt{current\_phase}, \texttt{focus\_behavior}, \texttt{lifestyle\_context} fields to \texttt{user\_goals}
> \item Add \texttt{phase\_status} VARCHAR to \texttt{user\_goals}: \texttt{on\_track} / \texttt{running\_late} / \texttt{relapsed} / \texttt{completed\_early} / \texttt{completed}
> \item Add \texttt{outlier\_flag} (bool) and \texttt{outlier\_context\_response} (text) to \texttt{daily\_logs}
> \item Add \texttt{domain} field to \texttt{pattern\_insights} (weight / food / supplement / sleep / training / general)
> \item Add \texttt{phase\_context} field to \texttt{pattern\_insights}
> \item Add \texttt{domain\_tag} to \texttt{weekly\_wraps.suggested\_experiment}
> \item \textbf{Add \texttt{chat\_prompt} field to \texttt{weekly\_wraps}} — a back-to-chat bridge question referencing a specific signal from the current week. This is NOT micro\_hook. \texttt{micro\_hook} is a transient Logging Tool API response field only — never stored in the DB.
> \item Add \texttt{relapse\_flag} (bool) and \texttt{relapse\_detected\_at} (TIMESTAMPTZ) to \texttt{user\_behavioral\_state}
> \item Add \texttt{completed\_early} (bool) and \texttt{actual\_completion\_date} (DATE) to each phase entry within \texttt{phase\_journey} JSONB
> \item Add \texttt{signal\_extraction\_nudge\_shown} (bool) and \texttt{signal\_extraction\_confirmed} (bool) to \texttt{daily\_logs} or as a separate \texttt{conversation\_signals} table post-MVP
> \item Ensure \texttt{goal\_type} is never hard-coded to fat-loss values in constraints or enums
> \end{itemize}

---

# PART 5 — API CONTRACTS STRATEGY

The API layer acts as the bridge between the offline-first PWA frontend and the LangGraph orchestrator/database.

## Conceptual Approach

\begin{itemize}
\item \textbf{Idempotency is Critical.} Because the frontend queues logs while offline, every log request must include a \texttt{client\_log\_id} (UUID generated on the device). The backend uses this to prevent duplicate entries.
\item \textbf{Authoritative Timestamps.} The server must respect the \texttt{log\_date} generated by the client at time of typing, not the server receipt time.
\item \textbf{Parallel Responses.} Submitting a log triggers both a database write (structured data) and an AI agent call (conversational reply + micro-hook).
\item \textbf{Empty-state conventions.} \texttt{GET /goals/active}, \texttt{GET /experiments/active}, \texttt{GET /wraps/latest} return \texttt{200 OK} with a null object — never \texttt{404} — during early days before data exists.
\end{itemize}

### Action Required — Companion Document

> Refer to **API Contracts Document v1.2** for complete endpoint routes (\texttt{/api/v1/...}), request/response JSON payloads, HTTP status codes, authentication flows (OTP), rate limiting rules, and error code catalogs.
>
> **v5.0 alignment items for API team:**
> 
> \begin{itemize}
> \item \texttt{POST /goals} response must include \texttt{phase\_journey}, \texttt{current\_phase}, \texttt{focus\_behavior}
> \item \texttt{GET /goals/active} response must include full phase journey array
> \item \texttt{POST /logs} response \texttt{ai} block must always include \texttt{micro\_hook} field (never null) — \textbf{micro\_hook is not optional. Every logging tool response must include one.}
> \item Add \texttt{outlier\_context\_prompt} field to log response when \texttt{outlier\_flag} is true
> \item \texttt{GET /wraps/latest} response must include \texttt{phase\_snapshot}, \texttt{outlier\_narrative}, and \texttt{chat\_prompt} fields
> \item \texttt{GET /metrics/summary} response must include \texttt{phase\_progress} alongside existing rolling averages
> \end{itemize}

---

# PART 6 — AGENTIC AI DESIGN — THE LANGGRAPH PATTERN

We deploy a **LangGraph Deep Agent Architecture**. Instead of deploying disconnected AI personalities, we deploy a central Orchestrator equipped with long-term memory and behavioral state awareness, which governs specialized tools.

## 6.1 Core Design Philosophy

The Orchestrator is the single decision authority. Before responding, it checks `user_behavioral_state`, applies data maturity penalties, enforces insight fatigue caps, and decides whether to speak or suppress. It does not generate domain content itself — it routes.

**Why this architecture:**
- Memory continuity across interactions
- Consistent AI voice via Global Tone Formatter
- Insight fatigue management via central governor
- Sparse-data handling (Days 1–14 behave differently from Days 15+)
- Experiment tracking for narrative continuity ("Last week we tried X — here's what happened")

---

## 6.2 Agent Graph — Nodes, Edges & Persistence

Entry → orchestrator
orchestrator → onboarding_tool         (if onboarding_complete = false)
orchestrator → logging_tool            (logging or multi-log intent)
orchestrator → insight_tool            (insight query)
orchestrator → tone_formatter → user   (direct response)
Any tool → tone_formatter → user       (tone_safe = false)
Logging tool → user                    (tone_safe = true, bypasses formatter)
Any node → error_handler               (on failure)

**Wrap Tool** runs in a separate cron-driven workflow — not as a chat graph edge.

**LangGraph Persistence:** `PostgresSaver` checkpointer connects to the Neon PostgreSQL cluster via `DATABASE_URL_DIRECT` (direct, non-pooled). Chat state (`AgentState`) survives Cloud Run restarts and redeployments.

---

## 6.3 Agent Responsibilities

### Orchestrator — Output Governor
Routes requests and suppresses or permits responses based on behavioral state.

Core responsibilities:
\begin{enumerate}
\item Intent classification: \texttt{onboarding} / \texttt{logging} / \texttt{multi-log} / \texttt{insight\_query} / \texttt{general\_chat}
\item Data maturity check — applies confidence penalty using \texttt{data\_maturity\_days} and \texttt{sparse\_data\_flag}
\item Insight fatigue check — uses \texttt{insight\_fatigue\_suppression} and recent insight counts
\item Experiment continuity — passes active experiment context to tools
\item Safety routing — triggers safety flows for crisis/eating-disorder signals
\item Output control — chooses direct response vs. tool, and whether to suppress
\end{enumerate}

**Orchestrator Boundary:** If the user has not logged in 3+ days, the orchestrator does not trigger shame-based re-engagement. It checks for illness or life-event context first. If none, it sends one single low-friction re-entry prompt: *"Ready when you are — your Phase 1 progress is saved."* Then waits.

---

### Onboarding Tool — Phase Journey Generator
**Trigger:** `onboarding_complete = FALSE`

**Purpose:** Guide user through 5-step calibration, generate phase journey roadmap, and lock Phase 1 — before they log a single entry.

**5-Step Onboarding Flow:**
\begin{enumerate}
\item \textbf{Current stats first} — weight, height, optional body-fat estimate. No goal-setting before baseline.
\item \textbf{Goal capture} — user states their own goal in their own words. AI accepts without gatekeeping.
\item \textbf{Context signals} — 2–3 self-identified weaknesses, triggers, or lifestyle realities.
\item \textbf{Optional lifestyle context} — free text (shift work, irregular sleep, training history). Feeds pattern engine from Day 1.
\item \textbf{Phase journey generation + Phase 1 lock} — AI computes gap, generates multi-phase roadmap, presents Phase 1 clearly, locks it. User sees full arc before logging begins.
\end{enumerate}

**Physiological floor check:** The only AI guardrail on goal-setting. If a stated goal implies a physiologically unsafe rate (e.g., >1.5% body weight loss per week), the AI flags this with evidence — but does not block the user. User sovereignty is preserved.

**Outputs:**
- `phase_journey` array (phases with estimated durations and focus areas)
- `current_phase` definition: what to track, what to expect, what success looks like
- `primary_focus_behavior` — single variable to watch in Phase 1
- `lifestyle_context` — free text from Step 4

**Resumption logic:** `onboarding_step` (1–5) is stored in `AgentState` and persisted to DB. Onboarding resumes mid-flow if interrupted. Each tool response includes `next_onboarding_step`.

---

### Logging Tool — Natural Language Parser + Micro-Hook Engine
**Trigger:** Logging or multi-log intent from orchestrator

**Purpose:** Parse natural language input into structured JSON log entries, trigger outlier context prompts when needed, and always return a micro-hook observation.

**Input types:**
- Weight logs: `log_type: weight`, `weight_kg`
- Food logs: `log_type: food`, `food_description` (no client-side calorie/protein estimates)
- Context tags: `log_type: context`, `context_tags` (array)
- Photo logs: backend handles `POST /photos` workflow; Logging Tool acknowledges
- Retro-logs: user can log yesterday or earlier; system timestamps accurately and flags `is_retro: true`

**Extended context tag vocabulary:**
Original: `stress_high`, `sleep_low`, `alcohol`, `workout`, `sick`, `supplement_start`
Extended (consistent with onboarding weaknesses): `travel`, `social_event`, `weekend`, `late_night`, `binge`, `work_travel`, `family_event`
Open taxonomy: if no predefined tag fits, the tool derives a clear lowercase_snake_case tag from the user's phrase.

**Outlier Context Prompt:** When a log deviates significantly from trend (e.g., +2kg overnight), the AI asks a single, non-judgmental question: *"Anything different yesterday — travel, a big meal, poor sleep?"* The answer becomes a tagged signal, not a correction.

**Micro-Hook Engine:** Every log response ends with a 1-line AI observation or question. Never a confirmation alone. Never empty praise. Examples:
- *"That's 3 stress-high tags this week. Curious if the pattern holds — keep logging."*
- *"7-day average is down 0.4kg. Exactly on Phase 1 pace."*
- *"4 workout tags this week. Your average calories on workout days vs. rest days — worth watching."*

**Multi-log messages:** Compound inputs (e.g., *"92kg, stress high today, and I had a big mac meal"*) are parsed into multiple structured log entries in a single response.

**General health conversations:** The Logging Tool also handles supplement questions, illness context, and training discussions — acknowledges them conversationally while flagging key signals for pattern consideration.

**Passive signal extraction (Conversation Parser):** On every general health conversation, the Logging Tool runs a lightweight extraction check alongside the conversational response. There is no separate Conversation Parser agent or service — this step lives entirely inside the Logging Tool.

If a loggable signal is detected, the Logging Tool appends a single **inline confirmation nudge** to the bottom of the AI message:

> *"You mentioned starting creatine — want me to tag this so I can watch for patterns? → Yes, log it / No thanks"*

If the user confirms, a `context` log is written with the relevant tag and a free-text note. If the user declines, no log is written and the conversation continues normally.

**Signal types the Logging Tool extracts passively:**

\begin{table}[h]
\begin{tabular}{ll}
\hline
\textbf{User says} & \textbf{Tag written} \\
\hline
\textit{"Started creatine today"} & \texttt{supplement\_start} + free-text note \\
\textit{"Feeling sick, skipping gym"} & \texttt{sick} \\
\textit{"Terrible sleep last night"} & \texttt{sleep\_low} \\
\textit{"Back from a 3-day work trip"} & \texttt{work\_travel} \\
\textit{"Starting a new training block"} & Free-text note in AgentState \\
\hline
\end{tabular}
\end{table}

Post-MVP: high-confidence detections (e.g., supplement starts) are tagged automatically without asking.

---

### Insight Tool — Human-Readable Correlation Engine
**Trigger:** Insight query intent from orchestrator

**Eligibility gate:** Only surface insights where `confidence_adjusted ≥ 0.5` AND `data_maturity_days ≥ 14`.

**Confidence adjustment formula:**
confidence_adjusted = min(base_confidence × (data_maturity_days / 14), 1.0)

**Goal-agnostic pattern detection:** All correlation methods must operate on any log type and tag vocabulary — not just fat-loss metrics. Valid insight domains:
- `weight` — trend, variance, rolling averages
- `food` — calorie/protein correlations
- `supplement` — timing and outcome correlations
- `sleep` — energy/calorie correlations
- `training` — hunger/recovery correlations
- `general` — cross-domain behavioral clusters

**Plain-language output:** Every insight includes evidence window and confidence in plain language:
*"Based on 21 days of logs, nights marked stress_high are followed by higher average calories — confidence 0.73."*

**Phase context framing:** All insights are framed relative to current phase, not absolute targets.

---

### Wrap Tool — Weekly Narrative Engine
**Trigger:** Cron job, every Monday 04:00 UTC — **not** a chat graph edge.

**Purpose:** Generate weekly narrative wrap summarizing the full week across 5 sections.

**Wrap sections:**
\begin{enumerate}
\item \textbf{Phase snapshot} — current phase, days elapsed, trend vs. phase target, projected phase completion
\item \textbf{Progress snapshot} — trend, consistency score, goal progress in phase terms
\item \textbf{1–2 strongest detected patterns} — with evidence window and confidence
\item \textbf{Outlier narrative} — if any outlier occurred this week, explain it as signal, not failure (\textit{"The Tuesday spike aligns with your stress\_high tag and the work trip you mentioned. Pattern noted for next phase."})
\item \textbf{One recommended experiment for next week} — any testable health variable, not limited to food/weight. Must be measurable with existing log types.
\end{enumerate}

**Experiment scope:** Sleep timing changes, supplement introductions, stress-management habits, training frequency adjustments are all valid. The only constraint: it must be measurable with existing log types (weight, food, context tags, or conversation check-ins).

**Narrative continuity:** Wrap Tool actively reads the `experiments` table to reference prior experiments: *"Last week you tried removing alcohol Thu–Sat. Your Mon–Tue water weight is down 0.4kg vs. the prior week."*

**NULL-safe weight change:** If `weight_change_kg` is null (inconsistent weigh-ins), the wrap does not write *"weight changed null kg"*. Instead: *"No reliable weight trend this week — weigh-ins were inconsistent."*

**Output stored to:** `weekly_wraps` (includes `phase_snapshot` JSONB, `outlier_narrative` text, `chat_prompt` text, `suggested_experiment_id` with domain tag)

---

### Global Tone Formatter — Persona Enforcement Layer
**Role:** Enforce the product voice — candid, analytical, non-shaming, no motivational fluff.

**Bypass rule (cost/latency optimization):** If a tool sets `tone_safe: true` (Logging Tool micro-feedback, simple confirmations), the Orchestrator sends the response directly without a Tone Formatter call. Insight Tool and Wrap Tool outputs must always go through the Tone Formatter (`tone_safe: false`).

---

### Error Handler Node
**Purpose:** Graceful failure messaging and structured error logging.

**Behavior:**
- For chat requests: return a short, neutral message — *"Something broke on my side while processing that — try again in a bit."*
- Log structured context: `user_id`, `node`, `error_type`, `stack_trace`
- Up to 2 retries with exponential backoff (0.5s, then 1.5s) for transient failures
- Do not retry deterministic failures (JSON schema violations indicate prompt bugs)

**Dead-letter behavior for Wraps:** If wrap generation fails after retries, write to a DLQ `audit_events` row with `user_id`, week range, and error reason. Do not block other wraps from generating.

---

### Health Conversation Agent *(Post-MVP Direction)*
Handles open-ended health discussions — supplement stacks, workout programming, illness management, diet philosophy — using the user's longitudinal data as context: *"Given your patterns, here's what the data suggests about X."*

Does not diagnose. Does not prescribe. Provides evidence-informed, personalized discussion grounded in the user's own history.

**Constraint in MVP:** General health conversations are handled conversationally by the Logging Tool + Orchestrator direct response path. No dedicated agent. No structured data extraction. *Talking about supplements in chat = MVP. Building a supplement tracker = not MVP.*

---

## 6.4 Memory Model & Token Budgets

**Memory layers:**
\begin{enumerate}
\item \textbf{Short-term memory} — last 10 messages in \texttt{AgentState.messages}
\item \textbf{Working memory} — \texttt{user\_behavioral\_state}, \texttt{active\_goal} (with phase journey), onboarding flags, \texttt{active\_experiment}
\item \textbf{Long-term memory} — \texttt{pattern\_insights}, \texttt{experiments}, \texttt{weekly\_wraps}, \texttt{derived\_metrics} — accessed via targeted DB queries, not in-prompt dumps
\end{enumerate}

`recent_logs_summary` is a compact JSON summary (last 7–14 day averages, tag counts derived from `derived_metrics`) rather than raw rows, to control token costs.

**Token budgets per path:**

\begin{table}[h]
\begin{tabular}{lccc}
\hline
\textbf{Path} & \textbf{Input Prompt (approx.)} & \textbf{Output} & \textbf{Tone Formatter} \\
\hline
Logging (weight/food/context) & ~300 tokens & ~100 tokens & Bypassed (tone\_safe) \\
Insight query & ~2,000 tokens & ~300 tokens & Required \\
Weekly Wrap & ~4,000–6,000 tokens & ~800 tokens & Required \\
Onboarding (per step) & ~500 tokens & ~200 tokens & Required \\
\hline
\end{tabular}
\end{table}

---

## 6.5 AI Guardrails & Safety

\begin{itemize}
\item No medical diagnosis.
\item No predictive claims beyond observed correlations.
\item Tone must remain neutral, candid, and non-judgmental.
\item \textbf{General health conversations are permitted and encouraged.} The AI can discuss supplements, training approaches, diet strategies, and illness recovery in an evidence-informed, personalized way. This is health intelligence, not medical advice.
\item \textbf{Never refuse health discussions with "I can't help with that."} The right response to \textit{"should I try creatine?"} is a thoughtful, personalized discussion — not a deflection. The only hard stop is active medical emergencies or diagnosis requests.
\item \textbf{Context carries across conversation types.} Supplement log + stress in chat + weight spike = the AI connects these signals even across different interaction modes.
\item \textbf{Personal data always leads in conflicting information scenarios.} If the user asks about any health topic and their own data is relevant — lead with the personal data observation, then discuss the topic through that lens. Never surface generic information first when a personal data contradiction exists. Example: user asks about keto while data shows higher intake on low-carb days → \textit{"Your data actually has something to say about this. On your low-carb days over the last 3 weeks, your average intake was 380 kcal higher than on normal days — likely compensatory hunger. Keto can work, but that pattern suggests the hunger response is worth watching closely before committing. Want to try a 5-day experiment?"} If no relevant personal data exists: discuss the topic directly, then note what signals to watch for once they start logging. The AI never says \textit{"X won't work for you."} It says \textit{"here's what your data suggests — here's how to test it."}
\end{itemize}

**Crisis safety flow (India persona):**
If the AI detects active self-harm, suicidal ideation, or severe eating disorder distress:
\begin{enumerate}
\item Stop all coaching or weight-loss advice
\item Respond with calm, empathetic language
\item Reference: iCall India (9152987821), Vandrevala Foundation (1860-2662-345)
\item Clarify it is not a doctor or emergency service
\end{enumerate}

---

## 6.6 Model Version Pinning

\begin{table}[h]
\begin{tabular}{ll}
\hline
\textbf{Agent} & \textbf{Model (pinned)} \\
\hline
Orchestrator & \texttt{gpt-4o-2024-11-20} \\
Logging Tool & \texttt{gpt-4o-mini-2024-11-20} \\
Insight Tool & \texttt{gpt-4o-2024-11-20} \\
Wrap Tool & \texttt{gpt-4o-2024-11-20} \\
Tone Formatter & \texttt{gpt-4o-mini-2024-11-20} \\
\hline
\end{tabular}
\end{table}

Model versions must be set via environment config — never via generic `gpt-4o` alias.

---

## 6.7 Phase Revision Logic

The system tracks phase progress continuously and handles four distinct scenarios. The Orchestrator owns real-time detection. The Wrap Tool owns weekly narrative for all phase state changes.

### Relapse vs. Normal Variance — Classification Thresholds

Phase revision logic is always based on the **7-day rolling average**, never a single log. A single spike is noise. A sustained trend reversal is signal.

\begin{table}[h]
\begin{tabular}{lll}
\hline
\textbf{Signal} & \textbf{Classification} & \textbf{System Response} \\
\hline
+0.5–1kg overnight after a tagged event & Normal variance & Outlier context prompt only \\
7-day rolling average trending back toward phase start for 5+ consecutive days & Relapse signal & Flagged internally — Wrap Tool leads with forensic narrative \\
7-day rolling average crosses back above phase start value & Hard relapse & Full roadmap recalibration — Wrap Tool narrative + Phase Journey View reshapes \\
Phase completion criteria met ahead of projected date & Early completion & Orchestrator detects in real time — inline chat message, Phase 2 begins \\
7-day average tracking to miss projected phase end by >20\% of phase duration & Delayed completion & Auto-recalibration — Wrap Tool notification \\
\hline
\end{tabular}
\end{table}

---

### Scenario A — Early Phase Completion

**Trigger:** Orchestrator detects phase completion criteria met on the log that crosses the threshold — real-time, not held for the wrap.

**Rules:**
- Phase advances immediately. Phase 2 begins with its original duration unchanged.
- The overall goal timeline is not changed — the projected end date moves earlier.
- An **inline chat message** is sent immediately:

> *"Phase 1 — done. You set out to hit [target] by [original date]. You got there on [actual date] — [X] days early. Phase 2 starts now. New focus: [Phase 2 primary behavior]. Your updated roadmap is in the Journey tab."*

- `phase_journey` is updated: Phase 1 `status` → `completed_early`, Phase 2 `status` → `active`, downstream projected dates recalculated.
- The following Weekly Wrap acknowledges the phase transition in its phase snapshot section.
- **Tone rule:** Milestone messages are factual and direct. No exclamation marks. No motivational language. The data speaks.

---

### Scenario B — Delayed Phase Completion (No Regression)

**Trigger:** Wrap Tool detects during weekly generation that the 7-day rolling average is tracking to miss the projected phase end date by >20% of phase duration (e.g., 6+ days late on a 30-day phase).

**Rules:**
- Orchestrator auto-recalibrates all downstream phase durations and projected end dates.
- `phase_journey` updated: current phase `projected_end_date` extended, all downstream phases shifted.
- `phase_status` on `user_goals` updated to `running_late`.
- User notified in the **Weekly Wrap** — not as a standalone alert:

> *"Phase 1 is running a bit longer than projected — and that's fine. Based on your current pace, I've updated your roadmap. Phase 1 is now projected to complete around [new date]. Everything downstream has shifted accordingly. Your goal hasn't changed — the timeline has just been recalibrated to match your actual pace."*

---

### Scenario C — Relapse Signal (5+ Day Rolling Average Reversal)

**Trigger:** Nightly pattern engine detects 7-day rolling average has trended back toward phase start for 5+ consecutive days. Internal `relapse_flag` is set on `user_behavioral_state`.

**Rules:**
- System does not alert the user mid-week. No push notification. No in-chat message.
- The following **Weekly Wrap leads with the regression** as its primary narrative — framed forensically, never as a failure:

> *"This week the 7-day average moved back up 0.6kg. Looking at your logs, 4 of the 7 days had stress_high tags and 2 had alcohol — the same pattern that showed up in Week 2. The regression is explainable, which means it's addressable. Phase 1 timeline has been recalibrated. New projected completion: [date]. One experiment for next week: [targeted experiment based on detected trigger]."*

- Phase 1 timeline extends. `phase_status` → `running_late`. Experiment proposed targets the detected trigger pattern.

---

### Scenario D — Hard Relapse (Rolling Average Crosses Back Above Phase Start)

**Trigger:** Nightly pattern engine detects 7-day rolling average has crossed back above the phase start value recorded at onboarding.

**Rules:**
- User stays in Phase 1. **No phase reset.**
- Full downstream roadmap is recalibrated. `phase_journey` rewritten with new projected dates.
- `phase_status` → `relapsed`. `relapse_flag` → `true` on `user_behavioral_state`.
- Weekly Wrap acknowledges it plainly — no alarm language:

> *"You're back near where Phase 1 began. The goal hasn't changed — the roadmap has been updated to reflect where you actually are. Here's the pattern that's most likely driving this..."*

- Phase Journey View arc reshapes: marker moves backward toward phase origin. No red indicators. No warning icons.

---

### Scenario E — User-Initiated Relapse Acknowledgment

**Trigger:** User tells the AI about a bad period before the data shows it (e.g., *"I had a really bad two weeks, ate terribly, barely logged anything"*).

**Rules:**
\begin{enumerate}
\item Orchestrator acknowledges without judgment: \textit{"That happens. Everything is saved exactly where you left off."}
\item Prompts for a single retro context tag if useful: \textit{"Anything specific going on — travel, stress, illness?"}
\item Does \textbf{not} ask the user to review their phase, re-onboard, or explain themselves.
\item Resumes normal logging. Pattern engine recalibrates at the next wrap — not immediately.
\item The system never makes the user feel like they need to restart.
\end{enumerate}

---

### Phase Journey View — Visual Behavior for All States

The Phase Journey View arc is a **living timeline**, not a one-way progress bar. It moves in both directions. This is a feature, not a flaw. It is honest.

\begin{table}[h]
\begin{tabular}{ll}
\hline
\textbf{State} & \textbf{Visual Treatment} \\
\hline
On track & Position marker on arc, projected completion date shown \\
Running late & Marker position unchanged, projected completion date shifts right \\
Relapse signal & Marker moves backward on arc — no alarm color \\
Hard relapse & Marker returns toward phase origin, arc reshapes, timeline extends \\
Phase completed early & Marker reaches phase end, next phase begins, arc compresses \\
\hline
\end{tabular}
\end{table}

**UX rules:**
\begin{itemize}
\item Never use red, amber warning icons, or "streak broken" language for regression states.
\item The recalibrated projected end date is always visible and is the primary communication of phase status.
\item The arc reshaping on recalibration is animated subtly — the user sees the timeline update.
\item Phase completion (early or on-time) gets a persistent milestone dot on the arc. The journey history stays visible as the user moves into later phases.
\end{itemize}

---

### Phase Revision — Recalibration Trigger Summary

\begin{table}[h]
\begin{tabular}{llll}
\hline
\textbf{Scenario} & \textbf{Who Triggers} & \textbf{When} & \textbf{User Notification} \\
\hline
Phase completed early & Orchestrator (real-time log check) & Immediately when criteria met & Inline chat message \\
Phase on track & — & — & Phase snapshot in Weekly Wrap \\
Phase running late (>20\% over) & Wrap Tool (weekly) & Monday wrap generation & Weekly Wrap lead section \\
Relapse signal (5+ day reversal) & Wrap Tool (weekly) & Monday wrap generation & Weekly Wrap lead story \\
Hard relapse (back to start) & Wrap Tool (weekly) & Monday wrap generation & Weekly Wrap lead story + Phase Journey View reshapes \\
User self-reports relapse & Orchestrator (real-time chat) & Immediately & Inline acknowledgment + single context prompt \\
\hline
\end{tabular}
\end{table}

---

### Action Required — Companion Document

> Refer to **AI Agents & Prompt Engineering Document v2.0** for complete LangGraph node/edge structures, tool routing logic, exact system prompts, confidence scoring formula, guardrail definitions, and few-shot examples.
>
> **v5.0 alignment items for AI team:**
> 
> \begin{itemize}
> \item Update Onboarding Tool: generate \texttt{phase\_journey} array, \texttt{current\_phase}, \texttt{focus\_behavior} in output JSON
> \item Update Logging Tool: \texttt{micro\_hook} is mandatory in every log response; add \texttt{outlier\_context\_prompt} logic; add passive signal extraction check with inline confirmation nudge on general health conversations
> \item Update Insight Tool: add \texttt{domain} and \texttt{phase\_context} fields to output; goal-agnostic correlation scope
> \item \textbf{Update Wrap Tool: Wrap Tool must generate \texttt{chat\_prompt} — a single question/observation that bridges the user from the wrap back into chat. Must reference a concrete signal from the current week. Must never be NULL for a delivered wrap. Distinct from \texttt{suggested\_experiment} and from \texttt{micro\_hook}.}
> \item Update Wrap Tool: add \texttt{phase\_snapshot} and \texttt{outlier\_narrative} sections; expand experiment domain scope; add relapse detection logic; add forensic relapse narrative format; add early completion acknowledgment in phase snapshot
> \item Update Orchestrator: add real-time phase completion check on every log submission; add phase recalibration trigger (>20\% delay threshold); add \texttt{phase\_status} write on recalibration; add conflicting information handling rule (personal data leads)
> \item Update AgentState: add \texttt{phase\_journey}, \texttt{current\_phase}, \texttt{phase\_status}, \texttt{relapse\_flag} from active goal and behavioral state
> \end{itemize}

---

# PART 7 — ANALYTICS INSTRUMENTATION

Analytics events must be instrumented from Day 1 to validate the MVP hypothesis and drive the post-MVP decision gate.

## Core Events to Track

\begin{table}[h]
\begin{tabular}{ll}
\hline
\textbf{Event} & \textbf{Why It Matters} \\
\hline
\texttt{onboarding\_completed} & Baseline funnel metric \\
\texttt{phase\_1\_locked} & First hook — user sees their journey \\
\texttt{log\_submitted} (by type) & Logging consistency signal \\
\texttt{micro\_hook\_replied} & Compulsive value delivery validation \\
\texttt{outlier\_context\_provided} & Signal quality — behavioral tagging engagement \\
\texttt{insight\_surfaced} & Pattern engine output \\
\texttt{insight\_engaged} (dismissed / acknowledged) & Insight quality signal \\
\texttt{wrap\_opened} & Retention signal \\
\texttt{experiment\_acknowledged} & Behavior change intent signal \\
\texttt{experiment\_completed} & Behavior change follow-through \\
\texttt{general\_health\_chat\_initiated} & Holistic partnership engagement \\
\texttt{signal\_extraction\_nudge\_shown} & Passive extraction working \\
\texttt{signal\_extraction\_confirmed} & User opted into tagging from conversation \\
\texttt{phase\_completed\_early} & Early completion — milestone \\
\texttt{phase\_recalibrated} & Timeline extended — delayed or relapsed \\
\texttt{relapse\_detected} & Pattern engine signal quality \\
\texttt{relapse\_wrap\_opened} & User engagement with difficult narrative \\
\textbf{\texttt{chat\_prompt\_tapped}} & \textbf{Measures wrap-to-chat conversion rate — how many wrap readers re-enter a chat session} \\
\hline
\end{tabular}
\end{table}

## Decision Gate (Post-MVP Expansion)

> **Do not expand scope until:** Day-14 retention > 40\% AND Weekly Wrap open rate is stable at ≥ 50\%.

---

# PART 8 — 90-DAY DELIVERY PLAN

## Phase 1 (Days 0–30) — Core Loop

\begin{itemize}
\item Next.js PWA shell, OTP auth
\item Neon DB provisioned (ap-south-1) with TimescaleDB extension
\item LangGraph Orchestrator core
\item Onboarding Tool with 5-step flow and phase journey generation + Phase 1 lock
\item Logging Tool with multi-log support, extended tag vocabulary, and micro-hook engine
\item Outlier detection + outlier context prompt
\item Phase-aware baseline pattern calculations
\item Basic IndexedDB offline queue
\item Sparse-data state handling (Days 1–14)
\item Weekly Wrap v1 with phase snapshot and outlier narrative
\item Analytics instrumentation (all core events)
\end{itemize}

## Phase 2 (Days 30–60) — Intelligence Depth

\begin{itemize}
\item Photo food estimation (AI vision pipeline)
\item Expanded pattern correlations (supplement and sleep tags)
\item Insight Tool with domain tagging and phase context framing
\item Clearer insight explanations with evidence windows and confidence scores in plain language
\item Retro-logging support with context prompts
\item Experiments table + narrative continuity in Weekly Wrap
\item Web Push notifications for Wrap delivery
\end{itemize}

## Phase 3 (Days 60–90) — Robustness & Breadth

\begin{itemize}
\item Robust offline sync engine with conflict resolution
\item General health conversation handling (supplement questions, illness context, training discussions) as conversational AI responses — no structured extraction
\item Phase Journey View in Profile (visual arc)
\item Sentry error tracking + AI-specific observability (routing accuracy, confidence distribution, token usage)
\item Full analytics dashboard for MVP hypothesis validation
\end{itemize}

---

# PART 9 — DB SETUP (NEON v3.0)

\begin{enumerate}
\item Provision a Neon project in the \texttt{ap-south-1} (Mumbai) region.
\item Enable the TimescaleDB extension after project creation:
   \begin{verbatim}
   CREATE EXTENSION IF NOT EXISTS timescaledb;
   CREATE EXTENSION IF NOT EXISTS pgcrypto;
   \end{verbatim}
\item Set the following environment variables:
\end{enumerate}

\begin{table}[h]
\begin{tabular}{lll}
\hline
\textbf{Variable} & \textbf{Connection Type} & \textbf{Used By} \\
\hline
\texttt{DATABASE\_URL} & Pooled (PgBouncer) & API/Hono service, Pattern Engine, nightly cron \\
\texttt{DATABASE\_URL\_DIRECT} & Direct (non-pooled) & Migrations, LangGraph PostgresSaver \\
\hline
\end{tabular}
\end{table}

**Why two URLs?** Neon's PgBouncer operates in statement-level pooling mode, which is incompatible with DDL migrations and LangGraph's advisory locks used for checkpoint consistency. Always use `DATABASE_URL_DIRECT` for the migration runner and the AI agent checkpointer. Use `DATABASE_URL` (pooled) everywhere else.

4. Run migration scripts against `DATABASE_URL_DIRECT` only:
   npx drizzle-kit migrate

---

# PART 10 — NOTE TO EACH DISCIPLINE TEAM

## To the UI/UX Team

The chat is everything. Your primary job is to make the chat interface feel like talking to a sharp expert, not using an app. There is no logging screen to design. There is no separate "Add Log" button.

\begin{itemize}
\item \textbf{Offline states are first-class.} Design calm, subtle UI indicators for \textit{"Will sync when online."}
\item \textbf{Message status chips} — \textit{Logged · Synced · Failed} — should be tiny micro-text under the chat bubble.
\item \textbf{Micro-hook messages} must visually feel like a discovery, not a notification. They are part of the conversation — not a separate UI element.
\item \textbf{Weekly Wrap} should feel like reading a personal letter or newsletter, not a data-heavy dashboard. The phase snapshot is the lede. The outlier narrative is the story. The experiment is the CTA.
\item \textbf{Phase Journey View} is not a progress bar. It is a \textbf{living timeline} — a visual arc that moves in both directions. Forward on progress, backward on regression. This is a feature, not a flaw. It is honest. The arc reshapes on every recalibration. Milestone dots persist at phase completion points. The user should see their full journey history, not just current state.
\item \textbf{Phase state colors:} Never use red or amber for regression states. The arc position and recalibrated projected end date communicate status without alarm. Regression is not failure — it is context.
\item \textbf{Inline milestone messages} (early phase completion) must feel like a real-time discovery in the chat — not a banner or modal. They are part of the conversation.
\item \textbf{Outlier context prompt} must be visually gentle. It is not an alarm. It is a curious question.
\end{itemize}

## To the Backend/API Team

\begin{itemize}
\item Every log write is idempotent by \texttt{client\_log\_id}. The offline queue may retry. Design for it.
\item \texttt{log\_date} (user-local date) is the authoritative display date. \texttt{log\_ts} is the time-series partition key. Never conflate them.
\item The AI response to every log (including \texttt{micro\_hook}) is part of the same API response — not a separate async call. Parallel: DB write + AI call, merge response.
\item Pattern Engine cron runs nightly at 02:00 UTC. Wrap cron runs every Monday at 04:00 UTC. Both use \texttt{DATABASE\_URL\_DIRECT}.
\end{itemize}

## To the AI/ML Team

\begin{itemize}
\item The Orchestrator is the governor — not the voice. It routes, suppresses, and controls. The Tone Formatter is the voice.
\item \textbf{\texttt{micro\_hook} is not optional. Every logging tool response must include one. A response with only \textit{"Logged 92kg."} is a product failure.}
\item Insight surfacing gate is non-negotiable: \texttt{confidence\_adjusted} ≥ 0.5 AND \texttt{data\_maturity\_days} ≥ 14. Hard-code this check.
\item General health conversations are in scope. Never deflect supplement or training questions with \textit{"I can't help with that."}
\item Phase context must be loaded into \texttt{AgentState} on every session hydration. All pattern framing and progress responses must reference current phase.
\item \textbf{Wrap Tool must generate \texttt{chat\_prompt} — a wrap-to-chat bridge question referencing a concrete signal from the current week. Must never be NULL. Distinct from \texttt{suggested\_experiment} and from \texttt{micro\_hook}.}
\end{itemize}

---

# PART 11 — FUTURE SCOPE (Post-MVP Evolution)

**Gate condition:** Do not expand scope until Day-14 retention exceeds 40% and weekly wrap engagement is stable.

## Phase 4 (90–180 days) — Goal Type Breadth
\begin{itemize}
\item New goal types in onboarding: muscle gain, energy optimization, recovery from injury/illness, general wellness
\item Structured supplement logging: name, dose, timing, subjective notes
\item Training log: type, duration, intensity (simple tags initially)
\item Expanded pattern engine: supplement-outcome correlations, training-recovery correlations
\item Visual phase journey timelines across all goal types
\end{itemize}

## Phase 5 (180–365 days) — Deep Health Partner Mode
\begin{itemize}
\item Health Conversation Agent: dedicated agent for open health discussions, grounded in user's own data history
\item Illness/symptom logging: structured capture of sick days, symptoms, medications
\item Pattern engine cross-domain: connect supplement timing + sleep + energy + weight into multi-variable narratives
\item \textit{"What We Discussed"} section in Weekly Wrap — turns the wrap from a data summary into a full health narrative
\item Medical report uploads with strict disclaimers
\item Decision-support chat (training, nutrition choices) tied to longitudinal history
\end{itemize}

## Phase 6 (Year 2) — Health OS
\begin{itemize}
\item Wearable integration: sleep, HRV, steps as passive log inputs
\item Medical context: lab results (with strict disclaimers), prescription context
\item Multi-goal management: parallel goal journeys (fat loss + strength gain)
\item Yearly wrap: full 12-month behavioral intelligence summary
\item Predictive experiments: \textit{"Based on your patterns, here's what will likely happen if you try X."}
\end{itemize}

## Phase 7 (Year 3+) — Personalized Health Intelligence
\begin{itemize}
\item Truly individualized AI model fine-tuned on longitudinal user data
\item AI that proactively surfaces health risks based on pattern deviations
\item Integration with healthcare providers (with explicit consent)
\item Multi-variable predictive models and deep simulations
\end{itemize}

---

# PART 12 — OPEN QUESTIONS

**All three open questions from v4.0 were resolved in the founder session on March 4, 2026. Decisions are incorporated throughout this document. Summary below.**

**Resolved in downstream documents:**
\begin{itemize}
\item Confidence scoring formula → AI Agents \& Prompt Engineering v2.0
\item Data retention and privacy model for immutable logs → Data Model \& Schema v1.1
\item Experiment recommendation safety guardrails → AI Agents \& Prompt Engineering v2.0
\item Analytics event schema → instrumented in Phase 1 build
\end{itemize}

**Resolved in this version (v5.0):**

\begin{enumerate}
\item \textbf{General conversation → structured signal pipeline} ✓ The Logging Tool owns passive signal extraction entirely. No separate Conversation Parser agent. An inline confirmation nudge is surfaced when a loggable signal is detected. User confirms or declines. See Part 6.3 (Logging Tool).
\item \textbf{Conflicting information handling} ✓ If personal data is relevant to a health question, the AI leads with the personal data — always. Generic information is never surfaced first when a personal data contradiction exists. See Part 6.5 (AI Guardrails).
\item \textbf{Phase revision logic} ✓ Full ruleset defined: early completion (advance phase, compress timeline), delayed completion (auto-recalibrate, notify in wrap), relapse signal (wrap leads with forensic narrative), hard relapse (full recalibration, arc reshapes). See Part 6.3 (Orchestrator, Wrap Tool) and Part 6.7 (Phase Revision Logic).
\end{enumerate}

**No open questions remain for v5.0.**

---

# PART 13 - MONOREPO SCAFFOLD

health-app/
│
├── apps/
│   │
│   ├── web/                                  # Next.js 15 PWA → deployed on Vercel
│   │   ├── app/
│   │   │   ├── (auth)/
│   │   │   │   └── login/
│   │   │   │       └── page.tsx              # OTP login screen
│   │   │   ├── (app)/
│   │   │   │   ├── page.tsx                  # Chat — default home screen
│   │   │   │   ├── insights/
│   │   │   │   │   └── page.tsx              # Weekly Wraps + pattern cards
│   │   │   │   └── profile/
│   │   │   │       └── page.tsx              # Phase Journey View + controls
│   │   │   ├── onboarding/
│   │   │   │   └── page.tsx                  # Full-screen modal overlay (non-dismissible)
│   │   │   └── layout.tsx
│   │   ├── components/
│   │   │   ├── chat/
│   │   │   │   ├── ChatInput.tsx
│   │   │   │   ├── MessageBubble.tsx         # User bubble + AI response card
│   │   │   │   ├── MicroHook.tsx             # Visually distinct micro-hook line
│   │   │   │   ├── MessageStatusChip.tsx     # Logging… / Logged / Synced / Failed
│   │   │   │   ├── OutlierPrompt.tsx         # Suggestion chips for outlier context
│   │   │   │   └── SignalNudge.tsx           # Yes log it / No thanks inline nudge
│   │   │   ├── insights/
│   │   │   │   ├── WrapCard.tsx              # Hero wrap preview card
│   │   │   │   ├── WrapDetail.tsx            # Full-screen wrap overlay
│   │   │   │   └── PatternCard.tsx
│   │   │   ├── profile/
│   │   │   │   ├── PhaseArc.tsx              # Simplified arc preview
│   │   │   │   └── PhaseJourneyDetail.tsx    # Full-screen arc view
│   │   │   └── ui/                           # Shared primitives: Button, Chip, Card, Modal
│   │   ├── lib/
│   │   │   ├── dexie/
│   │   │   │   └── db.ts                     # IndexedDB schema via Dexie.js
│   │   │   └── sync/
│   │   │       └── queue.ts                  # Offline log queue + sync engine
│   │   ├── public/
│   │   │   └── sw.js                         # Serwist service worker
│   │   ├── next.config.ts
│   │   ├── tailwind.config.ts
│   │   └── package.json
│   │
│   ├── api/                                  # Node.js / Hono API gateway → Cloud Run service
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   │   ├── auth.ts                   # OTP send + verify
│   │   │   │   ├── onboarding.ts             # POST /onboarding/step, GET /onboarding/status
│   │   │   │   ├── logs.ts                   # POST /logs, GET /logs
│   │   │   │   ├── goals.ts                  # GET /goals/active
│   │   │   │   ├── insights.ts               # GET /insights, POST acknowledge/dismiss
│   │   │   │   ├── wraps.ts                  # GET /wraps/latest, GET /wraps, POST /wraps/:id/read
│   │   │   │   ├── experiments.ts            # GET /experiments/active, POST acknowledge/complete
│   │   │   │   ├── metrics.ts                # GET /metrics/summary
│   │   │   │   └── photos.ts                 # POST /photos → Cloudflare R2
│   │   │   ├── middleware/
│   │   │   │   ├── auth.ts                   # JWT verification
│   │   │   │   └── rate-limit.ts
│   │   │   ├── lib/
│   │   │   │   ├── ai-client.ts              # Internal HTTP client → apps/ai service
│   │   │   │   └── r2-client.ts              # Cloudflare R2 upload helper
│   │   │   └── index.ts
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   └── ai/                                   # Python / LangGraph agents → Cloud Run service + jobs
│       ├── agents/
│       │   ├── orchestrator.py               # Intent routing + behavioral state governor
│       │   ├── onboarding_tool.py            # 5-step flow + phase journey generation
│       │   ├── logging_tool.py               # NL parser + micro-hook engine + signal extraction
│       │   ├── insight_tool.py               # Correlation engine (confidence-gated, day 14+)
│       │   ├── wrap_tool.py                  # Weekly narrative engine (cron-driven)
│       │   ├── tone_formatter.py             # Enforces product voice (bypassed if tone_safe=true)
│       │   └── error_handler.py              # Retry logic + DLQ for failed wraps
│       ├── graph/
│       │   ├── state.py                      # AgentState TypedDict
│       │   └── workflow.py                   # StateGraph wiring + PostgresSaver checkpointer
│       ├── crons/
│       │   ├── pattern_engine.py             # Nightly 02:00 UTC — 7-day rolling avg + tag correlations
│       │   └── wrap_generator.py             # Every Monday 04:00 UTC — weekly wrap generation
│       ├── server.py                         # FastAPI server — exposes /invoke for Hono ai-client
│       ├── Dockerfile
│       ├── pyproject.toml
│       └── requirements.txt
│
├── packages/
│   │
│   ├── types/                                # Shared TypeScript types (web + api only)
│   │   ├── src/
│   │   │   ├── domain.ts                     # DailyLog, PatternInsight, WeeklyWrap, PhaseJourney, AgentState
│   │   │   └── api.ts                        # All API request/response payload shapes
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── db/                                   # Drizzle ORM schema + migrations + Node client
│   │   ├── src/
│   │   │   ├── schema/
│   │   │   │   ├── users.ts
│   │   │   │   ├── goals.ts                  # incl. phase_journey JSONB, current_phase, focus_behavior
│   │   │   │   ├── daily-logs.ts             # immutable + idempotency_key + outlier_flag
│   │   │   │   ├── derived-metrics.ts        # TimescaleDB continuous aggregates
│   │   │   │   ├── pattern-insights.ts       # confidence_adjusted, domain, phase_context
│   │   │   │   ├── weekly-wraps.ts           # phase_snapshot, outlier_narrative, chat_prompt
│   │   │   │   ├── experiments.ts
│   │   │   │   └── behavioral-state.ts       # relapse_flag, sparse_data_flag, engagement_trajectory
│   │   │   ├── client.ts                     # Exports pooled (DATABASE_URL) + direct (DATABASE_URL_DIRECT)
│   │   │   └── index.ts
│   │   ├── migrations/                       # Drizzle migration files — run via DATABASE_URL_DIRECT only
│   │   ├── drizzle.config.ts
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── config/                               # Shared env parsing + constants
│   │   ├── src/
│   │   │   ├── env.ts                        # Typed env vars (DATABASE_URL, AI_SERVICE_URL, R2, Sentry, etc.)
│   │   │   └── constants.ts                  # Cron schedules, token budgets, confidence thresholds
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── analytics/                            # Shared event names + instrumentation helpers
│   │   ├── src/
│   │   │   └── events.ts                     # All analytics event names: onboarding_completed,
│   │   │                                     # log_submitted, micro_hook_replied, wrap_opened,
│   │   │                                     # insight_surfaced, experiment_acknowledged, etc.
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── eslint-config/
│   │   └── index.js
│   │
│   └── tsconfig/
│       ├── base.json
│       ├── nextjs.json
│       └── node.json
│
├── infra/
│   ├── vercel/
│   │   └── project.json                      # Vercel project config for apps/web
│   ├── cloud-run/
│   │   ├── api-service.yaml                  # Always-on Cloud Run service — Hono API
│   │   ├── ai-service.yaml                   # Always-on Cloud Run service — FastAPI/LangGraph
│   │   ├── pattern-engine-job.yaml           # Cloud Run Job — nightly 02:00 UTC
│   │   └── wrap-generator-job.yaml           # Cloud Run Job — Monday 04:00 UTC
│   └── cloud-scheduler/
│       ├── pattern-engine.yaml               # Triggers pattern-engine-job at 02:00 UTC
│       └── wrap-generator.yaml               # Triggers wrap-generator-job at 04:00 UTC (Monday)
│
├── .env.example                              # DATABASE_URL, DATABASE_URL_DIRECT,
│                                             # AI_SERVICE_URL, OPENAI_API_KEY,
│                                             # R2_BUCKET, SENTRY_DSN, etc.
├── turbo.json
├── pnpm-workspace.yaml
└── package.json


*Document version: v5.1 — March 14, 2026*
*Previous version: Technical Solutions Document v5 (March 8, 2026)*
*Aligned with: Health App MVP Experience Spec v4.1 (March 3, 2026)*

---