# Health App — Product Experience Spec v4.1

*Harshit Shishodia Production  
Vision + MVP + Future Scope*

---

## From The Founder

What we're building is not another health app. The world already has enough calorie counters, workout trackers, and motivational platforms telling people to "do better," yet outcomes remain poor. People start strong, stall, lose momentum, and eventually quit — not because they lack information, but because they lack understanding. The real problem in personal health isn't data collection; it's the absence of intelligence that explains why progress happens or why it stops.

At the core of our company is AI — not as a buzzword, but as the intelligence layer that finally makes health data meaningful. Our vision is to build a personal pattern-intelligence system for health: an AI companion that doesn't just track behavior but continuously interprets it. Every weight log, meal, photo, and small confession becomes signal. The AI analyzes these signals over time, identifies non-obvious behavioral patterns, and translates them into clear, human explanations. Instead of generic advice, it tells you what is specifically true for you. Instead of motivation slogans, it delivers insight. Instead of pushing perfection, it encourages honest reality.

The product is intentionally designed around simplicity so that the intelligence can do the heavy lifting. We begin with a focused MVP built on five tightly integrated layers: conversational onboarding, frictionless logging, an AI pattern engine, a weekly insight wrap, and transparent user control. The goal is not endless daily engagement or dopamine loops; the goal is weekly clarity. Users log their reality, the AI detects meaningful correlations, explains them in language they can trust, and recommends one small experiment to test next. That loop — roadmap, log, interpret, explain, experiment — is how real behavior change happens, and it is the foundation of our retention strategy.

What makes this opportunity compelling is that we are creating a new category. Existing apps use AI mainly for automation or recommendations; we use AI for personal understanding. The longer someone uses the product, the more the system learns their patterns, context, and triggers. Over time, the intelligence becomes uniquely personal, producing insights that no generic model or new competitor can replicate. That creates a defensible moat built on longitudinal behavioral intelligence — not content libraries or feature checklists.

From a business standpoint, we are execution-focused and capital efficient. Our early AI is practical and grounded — using explainable intelligence and observable correlations rather than opaque black-box promises. The MVP is designed to validate one core hypothesis: if an AI system can surface a non-obvious personal insight within the first two weeks, users will come back because they feel understood. Once that insight loop proves retention, the platform naturally expands into deeper predictive intelligence, more sophisticated experimentation, integrations with wearables and medical data, and eventually a unified AI health operating system centered around the individual.

Ultimately, we believe the future of digital health belongs to AI systems that help people understand themselves, not just measure themselves. People don't need another app telling them what healthy looks like. They need intelligence that can reveal why they behave the way they do — and guide them forward with evidence, clarity, and trust. That's the company we're building.

**The long-term vision is broader than fat loss or body composition.** Every person — whether they're an athlete optimizing recovery, a professional managing stress-related weight gain, a patient navigating a post-illness recovery, or someone experimenting with a new supplement stack — deserves one intelligent system that knows them. Not a calorie app. Not a fitness tracker. Not a symptom checker. One partner that understands their body, their patterns, their goals, and their life context — and grows more accurate and useful the longer they use it.

The MVP is deliberately focused. But the architecture, the data model, and the AI design must be built from Day 1 to accommodate this broader vision — because the moat is longitudinal behavioral intelligence, and that only compounds if the system is built to understand the whole person, not just one goal type.

*Date: February 21, 2026 (updated March 4, 2026 — v4.1)*

*Persona Focus: "Dev" is the MVP test persona — a proxy for a detail-oriented, analytically-minded person who wants to understand, not just track. The product is not for Dev alone. It is for anyone who wants a health partner that actually understands them.*

---

# 1. Executive Summary

This document defines what we are building, why it exists, and how it evolves from MVP to long-term vision. It is the source-of-truth experience specification intended to inform downstream documents such as tech stack, data model, agents design, analytics, and delivery planning.

**Core promise:** The app becomes the user's health intelligence partner — understanding their patterns, mapping their journey, running experiments with them, and being the one place they turn for any health question, decision, or discussion.

**North-star outcome (MVP):** User understands their current position and phase journey from Day 1, discovers at least one non-obvious personal pattern within 14 days, and returns to chat — not just to log, but to think about their health.

---

# 2. Product Vision (Long-Term)

**Category definition:** A personal health intelligence OS — one AI that knows your body, your patterns, your goals, and grows more valuable the longer you use it. Not a calorie tracker. Not a symptom checker. Not a coach app. The system that understands you specifically and helps you make better health decisions across everything: diet, training, supplements, recovery, habits, and life events.

**Vision statement:** A super-intelligent health partner that learns from every log, every question, and every life event — maps the user's journey in phases, detects their unique behavioral patterns, runs experiments with them, and engages in honest, evidence-based conversations about anything affecting their health: food, training, supplements, stress, sleep, illness, or life context. Zero shame. Pure curiosity.

## 2.1 Design Principles

1. **Unified experience:** Chat for actions and logs, insights for reflection, profile for control.
2. **Reality over perfection:** Logs are immutable; corrections are tracked, not erased.
3. **Explainable intelligence:** Insights must be understandable and actionable.
4. **Trust-first tone:** Candid, analytical, non-shaming.
5. **Experiment mindset:** One small change at a time.
6. **User Sovereignty:** The user sets their own goals. The AI illuminates the gap, maps the phases, and surfaces what the data shows — never gates or prescribes what is "realistic."
7. **Holistic Partnership:** The AI is not a single-purpose tool. It engages across the full spectrum of health — diet, training, supplements, recovery, illness, mental state — because patterns don't respect category boundaries. A supplement stack, a bad week at work, and a new training block all affect the same body.
8. **Compulsive Value Delivery ("Every Log Is a Hit"):** The system must make every single interaction feel worth it — not through gamification or streaks, but through intelligence. Every log gets a micro-hook. Every wrap reveals something the user didn't know. Every experiment closes with a result. The product is addictive because it is genuinely useful every single time. If a user logs their weight and gets back "Logged 92kg." — that is a failure. They should always get something back that makes them think.

## 2.5 The Persona Is Anyone

"Dev" is the MVP test persona — a proxy for a detail-oriented, analytically-minded person who wants to understand, not just track. But the product is not for Dev. It is for anyone who wants a health partner that actually understands them.

The system should adapt to:

- A 40-year-old woman managing perimenopause symptoms and energy levels
- A 22-year-old gym-goer optimizing a creatine + protein supplement stack
- A 55-year-old recovering from knee surgery, managing weight during limited mobility
- A 30-year-old with PCOS adjusting diet to manage insulin sensitivity
- A 28-year-old software engineer (Dev) trying to lose 8kg with a chaotic schedule

What unifies them is not demographics — it is the type of intelligence they need: pattern detection, honest explanation, goal-aware roadmapping, and a partner they can ask anything. The onboarding flow, the pattern engine, and the chat must all be built to serve any of these users — not just the MVP test persona.

**Design implication:** Never hard-code goal types, tag vocabularies, or insight templates around fat loss. Every system should be parameterized by the user's actual stated goal and context.

---

# 3. MVP Scope (What We Are Building First)

**MVP objective:** Validate the full engagement loop — **Roadmap → Log → Detect → Explain → Experiment → Repeat.**

The journey map generated at onboarding is the first hook. Every subsequent log adds data. The pattern engine detects signal. The AI explains it. An experiment is proposed. The wrap closes the loop. Repeat weekly.

**Note on MVP scope:** The MVP validates the core loop using body composition as the primary goal domain. However, the architecture, data model, and AI agents must be built in a goal-agnostic way — no hard-coded fat-loss logic. This ensures post-MVP expansion to new goal types (performance, recovery, supplement tracking, illness management) requires configuration and prompts, not re-architecture.

## MVP Success Metrics

- Onboarding completion ≥ 90% with Phase 1 journey locked.
- Phase 1 lock rate ≥ 80% (user sees and acknowledges their phase journey before exiting onboarding).
- ≥ 70% of users receive first pattern insight by Day 14.
- Weekly Wrap open rate ≥ 50%.
- Logging consistency > 5 days/week (active users).
- Micro-hook reply rate ≥ 30% (user responds to or engages with a micro-hook message within 24 hours).

---

# 4. MVP Architecture (5 Layers) with Agentic AI

## Layer 1 — Conversational Onboarding (Calibration)

**Purpose:** Understand the user's current state and goal, generate their phase journey roadmap, and lock Phase 1 — before they log a single thing.

**Onboarding Flow (5 Steps):**

1. **Current stats first:** Weight, height, optional body-fat estimate. No goal-setting before baseline.
2. **Goal capture:** User states their own goal in their own words. The AI accepts it without gatekeeping — fat loss, muscle gain, energy improvement, recovery, or any self-defined health objective.
3. **Context signals:** 2–3 self-identified weaknesses, triggers, or lifestyle realities (e.g., "I travel every week," "I stress-eat at night").
4. **Optional lifestyle context:** "Anything else I should know about your lifestyle or schedule?" (free text — captures shift work, irregular sleep, training history). This feeds the pattern engine from Day 1.
5. **Phase journey generation + Phase 1 lock:** AI computes the gap between current state and goal, generates a multi-phase journey roadmap, presents Phase 1 clearly, and locks it. User sees their full arc before logging begins.

**Outputs:**

- Phase journey roadmap (phases with estimated durations and focus areas).
- Phase 1 definition: what to track, what to expect, what success looks like.
- Primary focus behavior (single variable to watch in Phase 1).

**Physiological floor check:** The only AI guardrail on goal-setting. If a stated goal implies a physiologically unsafe rate (e.g., >1.5% body weight loss per week), the AI flags this with evidence — but does not block the user from proceeding. User sovereignty is preserved.

*Out of scope for MVP: medical uploads, psychological profiling, before/after simulations. "Commitment agreement" is removed — user intent is expressed through goal-setting, not a formal pledge.*

---

## Layer 2 — Logging Engine (Data Capture Foundation)

**Inputs:**

- Weight logs (daily or weekly).
- Food logs (manual entry + optional photo estimate).
- Simple context tags: sleep_low, stress_high, alcohol, workout, sick, supplement_start.
- Retro-logs: user can log yesterday or earlier; system timestamps accurately and flags as retro.
- Context tags on retro-logs: system prompts "anything different that day?" when retro-logging.

**Rules:**

- Food photo analysis is positioned as a rough estimate; user can adjust.
- Logs are immutable; edits are stored as revisions.
- Outlier prompt: when a log deviates significantly from trend (e.g., +2kg overnight), the AI asks a single, non-judgmental context question: "Anything different yesterday — travel, a big meal, poor sleep?" This becomes a tagged signal, not a correction.

**General health conversations → Structured Signal Pipeline:**

The Logging Tool runs a passive extraction check on every general health conversation. If it detects a loggable signal — supplement start, illness mention, training change, or travel — it appends a single inline confirmation nudge to the bottom of the AI response:

> *"You mentioned starting creatine — want me to tag this so I can watch for patterns? → Yes, log it / No thanks"*

If the user confirms, a context log is written with the relevant tag and a free-text note. If the user declines, no log is written and the conversation continues normally.

The Logging Tool owns this entirely. There is no separate Conversation Parser agent or service.

**Signal types the Logging Tool extracts passively:**

| User says | Tag written |
|---|---|
| "Started creatine today" | `supplement_start` + free-text note |
| "Feeling sick, skipping gym" | `sick` |
| "Terrible sleep last night" | `sleep_low` |
| "Back from a 3-day work trip" | `work_travel` |
| "Starting a new training block" | Free-text note in AgentState |

*Post-MVP: high-confidence detections (e.g., supplement starts) are tagged automatically without asking.*

*Architecture note: General conversations are stored in ChatHistory (AgentState), not as daily_logs. The signal pipeline bridges conversational context into structured tags via the inline confirmation nudge.*

---

## Layer 3 — AI Pattern Engine (Core Differentiator)

**Goal:** Surface clear, explainable correlations from any log type the user has provided.

**MVP methods:**

- 7-day rolling averages.
- Tagged vs. non-tagged day comparisons.
- Outlier detection using variance thresholds.
- Phase-aware progress: patterns are interpreted relative to current phase, not absolute targets.

**Micro-hook engine:** Every log response ends with a 1-line AI observation or question designed to drive re-engagement. Not a notification — a message hook. Examples: "That's 3 stress-high tags this week. Curious if the pattern holds — keep logging." / "7-day average is down 0.4kg. Exactly on Phase 1 pace."

**Example insights:**

- Alcohol-tagged days correlate with higher next-day weight.
- Low-protein days correlate with higher next-day hunger/logged calories.
- Stress_high tags in the last 7 days correlate with +380 kcal average daily intake.

**Goal-agnostic pattern detection:** All correlation methods (rolling averages, tag comparisons, outlier detection) must operate on any log type and tag vocabulary — not just fat-loss metrics. A user tracking energy levels, sleep quality, or supplement intake should get equally sharp pattern detection. The MVP implements this for body composition; the engine must be designed to generalize.

---

## Layer 4 — Weekly Wrap (Retention Engine)

**Cadence:** Every 7 days.

**Sections:**

1. **Phase snapshot:** Current phase, days elapsed, trend vs. phase target, projected phase completion (recalibrated timeline shown if applicable — see Phase Revision Logic below).
2. **Progress snapshot:** Trend, consistency score, goal progress.
3. **1–2 strongest detected patterns** (with evidence window and confidence).
4. **Outlier narrative:** If any outlier occurred this week, the wrap explains it in context — not as a failure, but as signal ("The Tuesday spike aligns with your stress_high tag and the work trip you mentioned. Pattern noted for next phase.").
5. **One recommended experiment for next week.**

**Experiment scope:** Experiments are not limited to food behavior. Any testable health variable is valid — a sleep timing change, a supplement introduction, a stress-management habit, a training frequency adjustment. The AI proposes one experiment per week based on the strongest detected pattern, regardless of domain. The only constraint is that it must be measurable with existing log types (weight, food, context tags, or conversation check-ins).

**UX goal:** User feels understood, sees their journey clearly, and is curious to test the experiment.

*Future section (post-MVP): "What We Discussed" — a recap of key health conversations from the week (supplement experiments started, illness logged, training changes noted). Turns the wrap from a data summary into a full health narrative.*

### Phase Revision Logic in the Weekly Wrap

The Weekly Wrap is the primary surface for communicating phase recalibration. The following rules govern how the wrap handles all phase transition states:

**Delayed completion (no regression):**  
If the 7-day rolling average is tracking to miss the projected phase end date by more than 20% of the phase duration (e.g., 6+ days late on a 30-day phase), the Orchestrator auto-recalibrates all downstream phase durations and projected end dates. The user is notified in the Weekly Wrap phase snapshot — not as a standalone alert:

> *"Phase 1 is running a bit longer than projected — and that's fine. Based on your current pace, I've updated your roadmap. Phase 1 is now projected to complete around [new date]. Everything downstream has shifted accordingly. Your goal hasn't changed — the timeline has just been recalibrated to match your actual pace."*

**Relapse signal (5+ day reversal of 7-day rolling average):**  
The pattern engine flags the relapse internally. The system does not alert the user mid-week. The following Weekly Wrap leads with the regression as its primary narrative — framed forensically, not as a failure:

> *"This week the 7-day average moved back up 0.6kg. Looking at your logs, 4 of the 7 days had stress_high tags and 2 had alcohol — the same pattern that showed up in Week 2. The regression is explainable, which means it's addressable. Phase 1 timeline has been recalibrated. New projected completion: [date]. One experiment for next week: [targeted experiment based on detected trigger]."*

**Hard relapse (rolling average crosses back above phase start value):**  
Weekly Wrap acknowledges it plainly:

> *"You're back near where Phase 1 began. The goal hasn't changed — the roadmap has been updated to reflect where you actually are. Here's the pattern that's most likely driving this..."*

**Phase transition acknowledgment:**  
If the user completed a phase transition since the previous wrap (whether early or on-time), the wrap's phase snapshot section acknowledges the transition explicitly.

---

## Layer 5 — Phase Journey View (Control & Transparency)

**Includes:**

- Phase journey timeline: visual arc from onboarding to current phase to goal, with phase milestones marked.
- Current phase detail: what's being tracked, what the AI is watching for, what defines phase completion.
- Data export.
- Delete account/data controls.

*Excluded from MVP: tone settings, notification granularity, medical archive, progress percentage bar (replaced by phase journey view).*

### Phase Journey View — Visual Behavior for All States

The Phase Journey View arc is a living timeline, not a one-way progress bar. It moves in both directions — forward on progress, backward on regression. This is a feature, not a flaw. It is honest.

| State | Visual Treatment |
|---|---|
| On track | Position marker on arc, projected completion date shown |
| Running late | Marker position unchanged, projected completion date shifts right |
| Relapse signal | Marker moves backward on arc — no alarm color |
| Hard relapse | Marker returns toward phase origin, arc reshapes, timeline extends |
| Phase completed early | Marker reaches phase end, next phase begins, arc compresses |

**UX rules:**

- Never use red, amber warning icons, or "streak broken" language for regression states.
- The recalibrated projected end date is always visible — it is the primary communication of phase status.
- The arc reshaping on recalibration is animated subtly — the user sees the timeline update, not just notices it changed.
- Phase completion (early or on-time) gets a visual milestone dot on the arc that persists as the user moves into later phases. The history of the journey stays visible.

### Phase Revision Logic — Full Ruleset

**Relapse classification is always based on the 7-day rolling average, never a single log. A single spike is noise. A sustained trend reversal is signal.**

**Defining relapse vs. normal variance:**

| Signal | Classification |
|---|---|
| +0.5–1kg overnight after a tagged event (alcohol, travel, stress) | Normal variance — outlier prompt only, no phase impact |
| 7-day rolling average trending back toward phase start for 5+ consecutive days | Relapse signal — flagged for next wrap |
| 7-day rolling average crosses back above phase start value | Hard relapse — full roadmap recalibration |

**3a. Early Phase Completion:**

If a user meets their phase completion criteria ahead of the projected date:

- Phase advances immediately. Phase 2 begins with its original duration unchanged.
- The overall goal timeline is not changed — the projected end date simply moves earlier.
- The Orchestrator detects completion in real time (on the log that crosses the threshold) and sends an inline chat message immediately — not held for the wrap:

> *"Phase 1 — done. You set out to hit [target] by [original date]. You got there on [actual date] — [X] days early. Phase 2 starts now. New focus: [Phase 2 primary behavior]. Your updated roadmap is in the Journey tab."*

- The Phase Journey View updates to reflect the new arc.
- The following Weekly Wrap acknowledges the phase transition in its phase snapshot section.

**3b. Delayed Phase Completion (No Regression):**

If the 7-day rolling average is tracking to miss the projected phase end date by more than 20% of the phase duration:

- The Orchestrator auto-recalibrates all downstream phase durations and projected end dates.
- The user is notified in the Weekly Wrap phase snapshot (see Layer 4 above).
- The Phase Journey View arc extends to reflect the new timeline.

**3c. Relapse:**

Core rule: **A relapse never resets the phase. It extends it — and the AI explains why.**

*Relapse signal (5+ day reversal):*
- The pattern engine flags the relapse internally — no mid-week alert.
- The following Weekly Wrap leads with the regression as primary narrative (see Layer 4 above).
- Phase 1 timeline recalibrates. Phase Journey View marker moves backward on arc.

*Hard relapse (rolling average crosses back above phase start):*
- User stays in Phase 1. No phase reset.
- Full downstream roadmap is recalibrated.
- Weekly Wrap acknowledges plainly (see Layer 4 above).
- Phase Journey View arc reshapes — marker moves toward phase origin. No alarm colors. No warning indicators.

*User-initiated relapse acknowledgment:*  
If the user tells the AI about a bad period before the data shows it:
- Acknowledge without judgment: "That happens. Everything is saved exactly where you left off."
- Prompt for a single retro context tag if useful: "Anything specific going on — travel, stress, illness?"
- Do not ask the user to review their phase, re-onboard, or explain themselves.
- Resume normal logging. Pattern engine recalibrates at the next wrap.

**The system never makes the user feel like they need to restart.**

---

# 5. MVP Data Model (Conceptual)

This section is intentionally high-level to drive downstream data-model docs.

**Core entities:**

- **User:** Profile, goal (user-stated, free text), goal_type (parameterized, not hard-coded), target_rate (AI-computed), phase_journey (array of phases), current_phase, focus_behavior, lifestyle_context (free text from onboarding Step 4).
- **DailyLog:** Timestamp, log_type (weight / food / context / retro), weight, food_estimates, protein/calories, tags, is_retro (bool), outlier_flag (bool), outlier_context_response.
- **DerivedMetrics:** Rolling averages, variance, adherence scores, phase_progress.
- **PatternInsight:** Correlation description, confidence score, evidence window, domain (weight / food / supplement / sleep / training / general), phase_context.
- **WeeklyWrap:** Phase snapshot, progress snapshot, surfaced insights, outlier narrative, suggested experiment (with domain tag), micro_hook.
- **ChatHistory:** Stored conversation turns (user messages + AI responses), indexed by date. Enables AI to reference prior health discussions — supplement starts, illness mentions, training changes — in future pattern analysis and wrap narratives. *Note: in MVP, chat context is held in AgentState (last N messages). Post-MVP: full persistent chat history with semantic search enables long-range referencing ("you mentioned X 6 weeks ago").*

*Downstream data model changes required for v4.1 (see Section 12):* `phase_status` on user_goals; `relapse_flag` on user_behavioral_state; `completed_early` bool on phase journey entries.

---

# 6. AI/Agents Boundaries for MVP

Guidance for agents design documents.

## Agent Responsibilities

**Onboarding Agent:**  
Collect current state first (stats before goals). Accept the user's own goal without gatekeeping — this can be fat loss, muscle gain, energy improvement, recovery optimization, or any self-defined health objective. Compute the gap between current state and goal. Generate the phase journey roadmap. Lock Phase 1. Apply physiological floor check only — flag unsafe rates with evidence, never block the user.

**Logging Agent (Logging Tool):**  
Parse structured logs (weight, food, context tags, photos). Parse retro-logs and timestamp accurately. Trigger outlier context prompts when variance threshold is exceeded. Append micro-hooks to every log response — no log response ends with just a confirmation.

In addition, the Logging Tool runs a passive extraction check on every general health conversation. If it detects a loggable signal (supplement start, illness mention, training change, travel), it appends a single inline confirmation nudge to the bottom of the AI response. If the user confirms, a context log is written with the relevant tag and free-text note. If the user declines, no log is written. The Logging Tool owns this pipeline entirely — there is no separate Conversation Parser agent.

Signal detection table: see Layer 2 above.

**Insight Agent:**  
Generate pattern statements from any detected correlation — not limited to weight/food/tag patterns. Supplement timing correlations, sleep-energy correlations, training-hunger correlations are all valid. Surface patterns in plain language with evidence window and confidence score. Always frame in current phase context.

**Wrap Agent (Wrap Tool):**  
Compose weekly narrative including phase snapshot, outlier narrative, and experiment recommendation. Also responsible for:
- Relapse detection logic: flag relapse signal (5+ day rolling average reversal) and hard relapse (rolling average crosses phase start) and trigger appropriate narrative framing.
- Recalibration trigger: when relapse or delay is detected, trigger downstream phase duration and projected date recalibration.
- Milestone message format: when phase completion is acknowledged in the wrap, use factual, direct tone — no exclamation marks, no praise language.

Experiments can be proposed across any health domain the user has logged or discussed — not only food/weight.

**Orchestrator:**  
Performs real-time phase completion check on every log submission. If a log crosses the phase completion threshold:
- Immediately sends inline chat milestone message (see Layer 5, Section 3a).
- Does not hold the message for the next wrap.
- Updates the Phase Journey View arc.

If the user has not logged in 3+ days, the orchestrator does not trigger shame-based re-engagement. It checks for illness or life-event context first. If none, it sends a single low-friction re-entry prompt ("Ready when you are — your Phase 1 progress is saved.") and waits.

**Health Conversation Agent (post-MVP direction):**  
Handles open-ended health discussions — supplement stacks, workout programming questions, illness management context, diet philosophy. Uses the user's longitudinal data as context. Does not diagnose. Does not prescribe. Provides evidence-informed, personalized discussion grounded in the user's own history.

## Constraints

- No medical diagnosis.
- No predictive claims beyond observed correlations.
- Tone must remain neutral, candid, and non-judgmental.
- **General health conversations are permitted and encouraged.** The AI can discuss supplements, training approaches, diet strategies, and illness recovery in an evidence-informed, personalized way — grounded in what the user's own data shows.
- **Never refuse health discussions with "I can't help with that."** The right response to "should I try creatine?" is a thoughtful, personalized discussion — not a deflection. The only hard stop is active medical emergencies or diagnosis requests.
- **Context carries across conversation types.** If the user logs a supplement start, discusses stress in chat, and then logs a weight spike — the AI connects these signals even if they came through different interaction modes (structured log vs. casual chat).
- **When personal data is relevant to a health question, lead with the personal data.** The personal observation is always the most valuable part of the answer. The AI never gives generic information first and surfaces the data contradiction as an afterthought (see Section 10 — Conflicting Information Handling guardrail).

---

# 7. What Is Explicitly Out of MVP

**Out of MVP (do not build):**  
Intelligent content feed, bloodwork analysis, advanced grading, before/after simulations, third-party integrations, yearly wrap, multi-variable predictive AI, structured supplement tracking (dedicated log type, dose/timing schema), training program tracking (sets, reps, progressive overload), illness/symptom logging as structured data, personalized supplement recommendation engine.

**In MVP but not as structured features (conversational only):**  
General health conversations via chat (supplement questions, training discussions, illness context) — these are in MVP as conversational responses, with passive signal extraction via the Logging Tool's inline confirmation nudge. Talking about supplements in chat = MVP. Building a supplement tracker = not MVP.

---

# 8. 90-Day Delivery Plan

**Phase 1 (0–30 days):** Conversational onboarding with phase journey generation + Phase 1 lock. Manual logging (weight, food, context tags). Outlier detection and outlier context prompt. Phase-aware baseline pattern calculations. Micro-hook engine on every log response. Weekly Wrap v1 with phase snapshot and outlier narrative.

**Phase 2 (30–60 days):** Photo food estimation. Expanded pattern correlations (supplement and sleep tags). Clearer insight explanations with evidence windows and confidence scores. Retro-logging support.

**Phase 3 (60–90 days):** Contextual nudges triggered by detected patterns (no full feed). General health conversation handling (supplement questions, illness context, training discussions) as conversational AI responses. Logging Tool passive signal extraction with inline confirmation nudge.

---

# 9. Future Scope (Post-MVP Evolution)

**Gate condition:** Do not expand scope until Day-14 retention exceeds 40% and weekly wrap engagement is stable.

**Phase 4 (90–180 days) — Breadth of Goal Types:**
- New goal types in onboarding: muscle gain, energy optimization, recovery from injury/illness, general wellness.
- Structured supplement logging: name, dose, timing, subjective notes.
- Training log: type, duration, intensity (simple tags initially).
- Expanded pattern engine: supplement-outcome correlations, training-recovery correlations.
- Visual progress timelines: see the full arc from onboarding to current phase.
- Post-MVP signal pipeline: high-confidence conversation detections auto-tagged without inline nudge.

**Phase 5 (180–365 days) — Deep Health Partner Mode:**
- Health Conversation Agent: dedicated agent for open health discussions, grounded in user's own data history.
- Illness/symptom logging: structured capture of sick days, symptoms, medications.
- Pattern engine cross-domain: connect supplement timing + sleep + energy + weight into multi-variable narratives.
- Visual journey timelines across all goal types.
- "What We Discussed" section in Weekly Wrap.
- Medical report uploads with strict disclaimers.
- Decision-support chat (training, nutrition choices) tied to history.

**Phase 6 (Year 2) — Health OS:**
- Wearable integration: sleep, HRV, steps as passive log inputs.
- Medical context: lab results (with strict disclaimers), prescription context.
- Multi-goal management: user can have parallel goals (fat loss + strength gain) with phase journeys for each.
- Yearly wrap: full 12-month behavioral intelligence summary.
- Predictive experiments: "Based on your patterns, here's what will likely happen if you try X."
- Expanded profile settings and archives.

**Phase 7 (Year 3+) — Personalized Health Intelligence:**
- Truly individualized AI model fine-tuned on longitudinal user data.
- AI that proactively surfaces health risks based on pattern deviations.
- Integration with healthcare providers (with explicit consent).
- Multi-variable predictive models and deep simulations.

---

# 10. Experience Tone & UX Guardrails

**On goal language — never editorialize:**  
Good: "You've set a goal of losing 10kg. Here's your phase journey." Bad: "That's ambitious — let's set a more realistic target." The AI never qualifies the user's goal as too hard, too easy, or unrealistic. It computes the gap and presents the journey. The user decides.

**On micro-hooks — make them feel like discovery, not reminders:**  
Good: "That's 4 stress-high tags this week. Your average intake on those days is 380kcal higher than stress-free days. Worth watching." Bad: "Great job logging today! Keep it up!" Hooks must carry a data observation or a genuine open question — never empty praise.

**On outliers — context over alarm:**  
Good: "Up 1.8kg overnight — probably water weight from the travel day you tagged. Your 7-day trend is still down 0.3kg." Bad: "You're up 1.8kg. Stay on track." Every outlier gets an explanation. Never present a number without context.

**On phase framing — the journey is the frame:**  
Good: "You're 11 days into Phase 1. The phase target is -2kg by Day 30 — you're at -0.8kg, right on pace." Bad: "You're 40% to your goal." Progress is always expressed in phase terms, not as a single percentage of the end goal.

**On general health conversations — be the knowledgeable friend:**  
Good: "You've been logging stress_high 4 days this week and your calorie average is up 400kcal. If you're thinking about ashwagandha for stress, that pattern is actually a good reason to try it — stress-driven eating is exactly the use case it's studied for. Want to track it as an experiment?" Bad: "I can only discuss your logged data, not give supplement advice."

**On illness or off-track periods — zero judgment, full context:**  
Good: "Looks like a rough week — 3 sick days logged and no weight entries. That's fine. When you're ready, let's pick up where Phase 1 left off. Your progress from before is still there." Bad: "You missed your logging streak." Illness, travel, stress, life — these are not failures. They are context that makes the pattern engine smarter.

**On being the one place for health thinking:**  
The user should never feel they need to go elsewhere to think about their health. Whether it's "should I try intermittent fasting?", "is this supplement worth it for me specifically?", or "I've been sick — what should I watch for?", the AI engages. It doesn't deflect. It discusses, personalizes to the user's own data, and helps them decide.

**On illness, injury, or life disruption — become the steadiest presence:**  
When a user signals they are sick, injured, burnt out, or going through a difficult period — the AI does not pivot to logging reminders or pattern catch-ups. It acknowledges the situation first, offers relevant context if useful ("water weight from inflammation is normal during illness — don't panic if the scale is up"), and simply stays present. "When you're ready to pick up again, everything is saved exactly where you left it." The app never makes the user feel behind. There is no behind. There is only current state and next phase.

**On conflicting information — personal data leads:**  
When a user asks about any health topic (diet strategy, supplement, training approach), the AI always checks whether the user's own data is relevant to the question. If personal data is relevant → lead with the personal data. Discuss the topic through that lens.

The personal observation is always the most valuable part of the answer. The AI never gives generic information first and surfaces the data contradiction as an afterthought.

Example — user asks about keto while their data shows higher intake on low-carb days:
> *"Your data actually has something to say about this. On your low-carb days over the last 3 weeks, your average intake was 380 kcal higher than on normal days — likely compensatory hunger. Keto can work, but that pattern suggests the hunger response is worth watching closely before committing. Want to try a 5-day experiment and see if the pattern holds?"*

If no relevant personal data exists: discuss the topic directly, then note what signals to watch for once they start logging.

Guardrail: The AI presents the data observation and offers an experiment. It never says "X won't work for you." It says "here's what your data suggests — here's how to test it."

**On phase regression — context over alarm:**  
Good: "The 7-day average is back up 0.6kg. Your stress_high tags this week likely explain it — same pattern as Week 2. Roadmap updated." Bad: "You've lost your Phase 1 progress. Get back on track."

**On phase recalibration — normalise it:**  
Good: "Your roadmap has been updated to match your actual pace. New projected completion: [date]." Bad: "You're behind schedule."

**On early completion — factual, not effusive:**  
Good: "Phase 1 complete — 12 days ahead of schedule. Phase 2 starts now." Bad: "Amazing work! You crushed Phase 1! 🎉"

**On user-initiated relapse acknowledgment — steady presence:**  
Good: "That happens. Everything is saved exactly where you left off. When you're ready, we'll pick up from here." Bad: "You missed a lot of logs. Let's get back on track!"

**On milestone messages — the data speaks for itself:**  
Milestone messages are factual and direct. No exclamation marks. No "amazing job." Examples follow the pattern: "Phase 1 — done. You set out to hit [target] by [original date]. You got there [X] days early. Phase 2 starts now."

---

# 11. Open Questions

*Section 11 is empty following the March 4, 2026 founder session. All previously open questions have been resolved and incorporated into this document.*

- General conversation → structured signal pipeline: **Resolved** — see Section 4, Layer 2 and Section 6 (Logging Tool responsibilities).
- Conflicting information handling: **Resolved** — see Section 10 (Conflicting information guardrail) and Section 6 (Constraints).
- Phase revision logic: **Resolved** — see Section 4, Layer 4 (Weekly Wrap phase revision rules), Layer 5 (Phase Journey View visual behavior), and Section 10 (regression/recalibration/milestone tone guardrails).

*New open questions should be added here as they emerge.*

---

# 12. Companion Document Alignment

The following downstream documents require updates following v4.1 publication:

| Document | Update Required |
|---|---|
| Solution Doc v4.0 | Part 6: extend Orchestrator and Wrap Tool with relapse detection, phase completion check, recalibration trigger. Part 12: close all three open questions. |
| Data Model v3.0 | New fields: `phase_status` on user_goals; `relapse_flag` on user_behavioral_state; `completed_early` bool on phase journey entries. |
| AI Agents v2.0 | Wrap Tool: relapse detection logic, recalibration trigger, milestone message format. Orchestrator: real-time phase completion check on log submission. Logging Tool: passive extraction + inline confirmation nudge. |
| UI/UX Design v2.0 | Phase Journey View: bidirectional arc, no alarm colors, recalibrated timeline display, milestone dot markers, inline milestone message component. |

---

*Document version: v4.1 — Updated March 4, 2026*  
*Previous version: Health App MVP Experience Spec v4.0 (March 3, 2026)*  
*Changes in v4.1: Three areas incorporated from March 4, 2026 founder session — (1) General conversation → structured signal pipeline (Logging Tool passive extraction + inline nudge); (2) Conflicting information handling guardrail (personal data leads); (3) Phase revision logic full ruleset (early completion, delayed completion, relapse signal, hard relapse, user-initiated relapse acknowledgment, Phase Journey View visual behavior for all states). Section 11 open questions fully resolved and cleared.*
