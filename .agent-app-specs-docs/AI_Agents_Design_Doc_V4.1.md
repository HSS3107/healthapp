# AI Agentic Design and Prompts Document
## AI-Powered Health Pattern Intelligence App - Version 2.0

**Prepared for:** AI/ML Engineering Team  
**Date:** March 8, 2026  
**Status:** Source of Truth for LangGraph Implementation  
**Aligned with:** Solution Doc v5.0, Data Model v4.1, Product Spec v4.1

---

## Document Purpose

This document is the **complete technical specification** for implementing the AI agent system using LangGraph. It provides AI engineers with:

- Complete LangGraph architecture (nodes, edges, state management)
- Exact system prompts for each agent
- Tool routing logic and decision trees
- Confidence scoring formulas
- Memory management strategies
- Token budgets and optimization rules
- Safety guardrails and crisis protocols
- Few-shot examples for each agent

This document translates the high-level AI strategy from Solution Doc v5.0 into **executable specifications** for LangGraph deployment.

---

## Table of Contents

1. Architecture Overview
2. LangGraph State Schema (AgentState)
3. Agent Graph Structure
4. Agent Specifications with Prompts
   - 4.1 Orchestrator (Output Governor)
   - 4.2 Onboarding Tool
   - 4.3 Logging Tool
   - 4.4 Insight Tool
   - 4.5 Wrap Tool
   - 4.6 Tone Formatter
   - 4.7 Error Handler
5. Routing Logic and Decision Trees
6. Memory Management Strategy
7. Confidence Scoring and Data Maturity
8. Safety Guardrails
9. Model Version Pinning
10. Token Budgets and Optimization
11. Phase Revision Logic Implementation
12. Testing and Validation

---

## 1. Architecture Overview

### 1.1 Core Philosophy

**Deep Agent Architecture** - Not disconnected AI personalities, but a central **Orchestrator** with long-term memory that governs specialized **Tools**.

**Key Principles:**
- **Orchestrator is the governor, not the voice** - Routes, suppresses, controls
- **Tone Formatter is the voice** - Enforces consistent persona
- **Memory continuity across interactions** - State persists via PostgresSaver
- **Insight fatigue management** - Central governor prevents overwhelming users
- **Sparse-data handling** - Days 1-14 behave differently from Days 15+
- **Experiment tracking** - Narrative continuity ("Last week we tried X...")

### 1.2 System Architecture Diagram

User Message
    ↓
[Entry: orchestrator]
    ↓
[Orchestrator Node]
├─ Intent Classification
├─ Data Maturity Check
├─ Insight Fatigue Check
├─ Experiment Continuity
├─ Safety Routing
└─ Output Control
    ↓
    ├─── onboarding_complete = FALSE → [Onboarding Tool] → [Tone Formatter] → User
    ├─── logging/multi-log intent → [Logging Tool] → User (bypasses formatter if tone_safe=true)
    ├─── insight_query intent → [Insight Tool] → [Tone Formatter] → User
    └─── general_chat → [Orchestrator direct response] → [Tone Formatter] → User
    
    ↓ (on any error)
[Error Handler]

[Wrap Tool] ← Separate cron workflow (Monday 04:00 UTC)

### 1.3 Persistence Layer

**LangGraph PostgresSaver:**
- Checkpointer connects to Neon PostgreSQL via `DATABASE_URL_DIRECT` (non-pooled)
- Chat state (AgentState) survives Cloud Run restarts and redeployments
- Thread ID = `user_id` ensures state continuity per user
- State includes: message history, onboarding progress, behavioral flags, active experiment

---

## 2. LangGraph State Schema (AgentState)

The AgentState is the **single source of truth** for all agent decisions. All agents read from and write to this shared state.

from typing import TypedDict, List, Dict, Optional, Literal
from datetime import datetime

class AgentState(TypedDict):
    # === User Identity ===
    user_id: str
    thread_id: str  # Same as user_id for single-user threads
    
    # === Conversation Context ===
    messages: List[Dict[str, str]]  # Last 10 messages for short-term memory
    # Each message: {"role": "user" | "assistant", "content": str, "timestamp": str}
    
    # === Onboarding State ===
    onboarding_complete: bool  # Gates access to logging tools
    onboarding_step: Literal["stats", "goal", "context", "lifestyle", "journey", "complete"]
    onboarding_data: Dict[str, any]  # Accumulated data during onboarding
    # Keys: weight_kg, height_cm, body_fat_pct, goal_text, context_signals, lifestyle_context
    
    # === Active Goal and Phase ===
    active_goal: Optional[Dict[str, any]]
    # Keys: goal_id, goal_text, goal_type, created_at
    
    phase_journey: List[Dict[str, any]]  # Full phase array from usergoals
    # Each phase: {phase_number, phase_name, duration_days, target_metric, focus_behavior, 
    #              projected_start, projected_end, status: "pending" | "active" | "completed" | "on_track" | "running_late" | "relapsed"}
    
    current_phase: Optional[Dict[str, any]]
    # Keys: phase_number, phase_name, days_elapsed, target_metric, focus_behavior, 
    #       projected_completion, status
    
    phase_status: Literal["on_track", "running_late", "relapsed", "completed_early"]
    
    # === Behavioral State ===
    data_maturity_days: int  # Days since first log, used for confidence adjustment
    sparse_data_flag: bool  # True if < 14 days of data
    adherence_score: float  # 0.0 to 1.0, logging consistency
    engagement_trajectory: Literal["increasing", "stable", "declining"]
    relapse_flag: bool  # Set when 7-day avg crosses back above phase start
    relapse_detected_at: Optional[datetime]
    
    # === Active Experiment ===
    active_experiment: Optional[Dict[str, any]]
    # Keys: experiment_id, description, domain_tag, start_date, end_date, status
    
    # === Recent Logs Summary ===
    recent_logs_summary: Dict[str, any]  # Compact JSON, not raw rows
    # Keys: last_7_days_avg_weight, last_14_days_avg_weight, tag_counts, outlier_flags
    
    # === Insight Fatigue ===
    insights_shown_this_week: int  # Counter, resets weekly
    insight_fatigue_suppression: bool  # True if > 3 insights shown this week
    
    # === Routing State ===
    intent: Literal["onboarding", "logging", "insight_query", "general_chat", "error"]
    tone_safe: bool  # If true, bypasses Tone Formatter for cost optimization
    suppress_response: bool  # If true, Orchestrator suppresses output entirely
    
    # === Error Context ===
    error_state: Optional[Dict[str, any]]
    # Keys: error_type, node, retry_count, timestamp

**State Hydration on Session Start:**

When a user sends a message, the backend:
1. Loads LangGraph checkpoint for `thread_id = user_id`
2. Queries database for fresh `userbehavioralstate`, `usergoals`, `experiments`, `derivedmetrics`
3. Updates AgentState with current values
4. Passes to Orchestrator

---

## 3. Agent Graph Structure

### 3.1 Node Definitions

from langgraph.graph import StateGraph, END

# Initialize graph
workflow = StateGraph(AgentState)

# Add nodes
workflow.add_node("orchestrator", orchestrator_node)
workflow.add_node("onboarding_tool", onboarding_tool_node)
workflow.add_node("logging_tool", logging_tool_node)
workflow.add_node("insight_tool", insight_tool_node)
workflow.add_node("tone_formatter", tone_formatter_node)
workflow.add_node("error_handler", error_handler_node)

# Set entry point
workflow.set_entry_point("orchestrator")

### 3.2 Edge Logic

def orchestrator_routing(state: AgentState) -> str:
    """Determines next node based on orchestrator output."""
    
    if state.get("error_state"):
        return "error_handler"
    
    if not state["onboarding_complete"]:
        return "onboarding_tool"
    
    if state["intent"] == "logging":
        return "logging_tool"
    
    if state["intent"] == "insight_query":
        return "insight_tool"
    
    if state["intent"] == "general_chat":
        # Orchestrator handles directly, goes to tone formatter
        return "tone_formatter"
    
    return END

def tool_routing(state: AgentState) -> str:
    """Routes tool outputs to formatter or END."""
    
    if state.get("error_state"):
        return "error_handler"
    
    if state["tone_safe"]:
        # Bypass formatter for cost optimization
        return END
    
    return "tone_formatter"

# Add conditional edges
workflow.add_conditional_edges("orchestrator", orchestrator_routing)
workflow.add_conditional_edges("onboarding_tool", tool_routing)
workflow.add_conditional_edges("logging_tool", tool_routing)
workflow.add_conditional_edges("insight_tool", tool_routing)

# Formatter and error handler always end
workflow.add_edge("tone_formatter", END)
workflow.add_edge("error_handler", END)

# Compile graph
app = workflow.compile(checkpointer=PostgresSaver(...))

---

## 4. Agent Specifications with Prompts

### 4.1 Orchestrator (Output Governor)

**Role:** Central decision authority. Routes requests, suppresses responses based on behavioral state, enforces insight fatigue, manages safety flows.

**Does NOT generate user-facing content** - routes to tools or responds minimally.

#### System Prompt

You are the Orchestrator for a personal health pattern intelligence system. You are NOT the user-facing AI voice—you are the decision engine that routes requests to specialized tools and enforces behavioral guardrails.

## Your Responsibilities

1. **Intent Classification**: Determine if user message is:
   - `onboarding` (onboarding_complete = false)
   - `logging` (weight, food, context tags, photo)
   - `insight_query` (asking about patterns, progress, correlations)
   - `general_chat` (health questions, supplement discussions, training advice)
   - `safety_escalation` (crisis signals, self-harm, eating disorder distress)

2. **Data Maturity Check**: Apply confidence penalty if data_maturity_days < 14:
   - confidence_adjusted = min(base_confidence * (data_maturity_days / 14), 1.0)

3. **Insight Fatigue Check**: Suppress insight responses if:
   - insights_shown_this_week >= 3
   - Set suppress_response = true

4. **Experiment Continuity**: If active_experiment exists, pass context to tools:
   - "User is currently testing: {experiment.description}. Reference this in responses."

5. **Safety Routing**: If crisis signals detected:
   - intent = "safety_escalation"
   - Generate empathetic response with India crisis resources (iCall, Vandrevala Foundation)
   - Stop all coaching/pattern advice

6. **Output Control**: Decide whether to:
   - Route to tool
   - Respond directly (general_chat)
   - Suppress entirely (insight_fatigue, sparse_data with low confidence)

7. **Real-Time Phase Completion Check**: On every logging intent, check if:
   - 7-day rolling average meets phase completion criteria
   - If yes: Update phase_status = "completed_early", send inline milestone message, advance to next phase

8. **Phase Recalibration Check**: On every logging intent, check if:
   - 7-day rolling average tracking to miss projected_end by ≥20% of phase duration
   - If yes: Set phase_status = "running_late", flag for next Weekly Wrap

9. **Conflicting Information Rule**: If user asks health question and their personal data is relevant:
   - ALWAYS lead with personal data observation first
   - Never surface generic information when personal data contradicts it

## Current User State

- onboarding_complete: {state.onboarding_complete}
- data_maturity_days: {state.data_maturity_days}
- sparse_data_flag: {state.sparse_data_flag}
- phase_status: {state.phase_status}
- active_experiment: {state.active_experiment}
- insights_shown_this_week: {state.insights_shown_this_week}
- recent_logs_summary: {state.recent_logs_summary}

## User Message
{user_message}

## Your Output (JSON)
Return ONLY valid JSON:
{{
  "intent": "onboarding" | "logging" | "insight_query" | "general_chat" | "safety_escalation",
  "confidence": 0.0-1.0,
  "reasoning": "Brief explanation of decision",
  "suppress_response": true | false,
  "tool_context": {{
    "active_experiment": "...",
    "phase_status": "...",
    "data_maturity_penalty": 0.0-1.0
  }},
  "direct_response": null | "string if handling general_chat directly"
}}

#### Orchestrator Node Implementation

import json
from openai import OpenAI

def orchestrator_node(state: AgentState) -> AgentState:
    """
    Orchestrator: Routes requests and enforces behavioral guardrails.
    """
    client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
    
    # Build prompt with current state
    system_prompt = ORCHESTRATOR_SYSTEM_PROMPT.format(
        state=state,
        user_message=state["messages"][-1]["content"]
    )
    
    # Call GPT-4o
    response = client.chat.completions.create(
        model=os.getenv("ORCHESTRATOR_MODEL", "gpt-4o-2024-08-06"),
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": state["messages"][-1]["content"]}
        ],
        response_format={"type": "json_object"},
        temperature=0.3
    )
    
    decision = json.loads(response.choices[0].message.content)
    
    # Update state
    state["intent"] = decision["intent"]
    state["suppress_response"] = decision.get("suppress_response", False)
    state["tone_safe"] = False  # Default, tools can override
    
    # If general_chat and orchestrator handles directly
    if decision["intent"] == "general_chat" and decision.get("direct_response"):
        state["messages"].append({
            "role": "assistant",
            "content": decision["direct_response"],
            "timestamp": datetime.utcnow().isoformat()
        })
    
    return state

---

### 4.2 Onboarding Tool (Phase Journey Generator)

**Trigger:** `onboarding_complete = false`

**Purpose:** Guide user through 5-step onboarding, generate phase journey roadmap, lock Phase 1.

#### System Prompt

You are the Onboarding Tool for a personal health pattern intelligence system. Your role is to guide the user through a 5-step conversational onboarding process and generate a personalized phase journey roadmap.

## Onboarding Steps

**Step 1: Current Stats**
- Ask: "Let's start with where you are right now. What's your current weight?"
- Collect: weight_kg, height_cm, optional body_fat_pct
- Output: {"next_step": "goal", "data": {...}}

**Step 2: Goal Capture**
- Ask: "What's your goal? Tell me in your own words."
- Accept ANY goal without gatekeeping (user sovereignty principle)
- If goal implies unsafe rate (>1.5% body weight loss/week):
  - Show physiological floor check: "That's a [X]% body weight loss per week. Research suggests 0.5-1% is more sustainable. You can proceed, but I'll flag if patterns suggest adjusting."
  - User can still proceed
- Output: {"next_step": "context", "data": {...}}

**Step 3: Context Signals**
- Ask: "What makes this hard for you? Pick 2-3 things."
- Present chips: stress_eating, poor_sleep, social_events, travel, inconsistent_schedule, late_night_hunger, alcohol, dining_out, weekend_binges, work_stress
- Output: {"next_step": "lifestyle", "data": {...}}

**Step 4: Lifestyle Context (Optional)**
- Ask: "Anything else I should know about your lifestyle or schedule? (Optional)"
- Free-text capture
- Output: {"next_step": "journey", "data": {...}}

**Step 5: Phase Journey Generation**
- Compute gap between current stats and goal
- Generate 3-5 phase roadmap with:
  - Phase duration (days)
  - Target metric for each phase
  - Primary focus behavior per phase
  - Projected completion dates
- Present Phase 1 clearly with full roadmap preview
- Output: {"next_step": "complete", "phase_journey": [...], "current_phase": {...}}

## Physiological Safety Rules

- Weight loss: Max 1% body weight/week recommended
- Weight gain: Max 0.5% body weight/week recommended
- If goal exceeds these, FLAG but DO NOT BLOCK

## Current Onboarding State

- Step: {state.onboarding_step}
- Data collected so far: {state.onboarding_data}

## User Message
{user_message}

## Your Output (JSON)
{{
  "response": "Conversational response to user",
  "next_step": "stats" | "goal" | "context" | "lifestyle" | "journey" | "complete",
  "data_update": {{...}},  // Fields to add to onboarding_data
  "phase_journey": [...],  // Only on step 5
  "current_phase": {{...}},  // Only on step 5
  "tone_safe": true  // Onboarding responses are pre-validated
}}

#### Few-Shot Examples

**Step 1 Example:**

User: "85kg"

{
  "response": "Got it—85kg. What's your height?",
  "next_step": "stats",
  "data_update": {"weight_kg": 85},
  "tone_safe": true
}

**Step 2 Example (Unsafe Rate):**

User: "Lose 15kg in 6 weeks"

{
  "response": "That's about 2.5kg per week, which is roughly 3% of your current body weight. Research suggests 0.5-1% per week (0.4-0.8kg for you) is more sustainable and safer. You can proceed with 15kg as your goal, but I'll flag if patterns suggest adjusting the timeline. Sound good?",
  "next_step": "context",
  "data_update": {"goal_text": "Lose 15kg in 6 weeks", "goal_type": "weight_loss", "physiological_flag": true},
  "tone_safe": true
}

**Step 5 Example (Phase Journey):**

{
  "response": "Here's your roadmap:\n\n**Phase 1: Initial Drop (30 days)**\nTarget: -3kg\nFocus: Calorie consistency\n\n[Phases 2-4 collapsed preview]\n\nPhase 1 starts now. Ready to lock this in?",
  "next_step": "complete",
  "data_update": {},
  "phase_journey": [
    {
      "phase_number": 1,
      "phase_name": "Initial Drop",
      "duration_days": 30,
      "target_metric": "-3kg",
      "focus_behavior": "Calorie consistency",
      "projected_start": "2026-03-09",
      "projected_end": "2026-04-08",
      "status": "active"
    },
    ...
  ],
  "current_phase": {
    "phase_number": 1,
    "phase_name": "Initial Drop",
    "days_elapsed": 0,
    "target_metric": "-3kg",
    "focus_behavior": "Calorie consistency",
    "projected_completion": "2026-04-08",
    "status": "on_track"
  },
  "tone_safe": true
}

---

### 4.3 Logging Tool (Natural Language Parser + Micro-Hook Engine)

**Trigger:** `intent = "logging"`

**Purpose:** Parse natural language logs (weight, food, context tags), detect outliers, generate micro-hooks, handle passive signal extraction.

#### System Prompt

You are the Logging Tool for a personal health pattern intelligence system. Your role is to parse natural language logs into structured data and ALWAYS return a micro-hook observation that drives re-engagement.

## Input Types You Handle

1. **Weight logs**: "92kg", "logged 185 lbs", "weighed in at 85.5"
2. **Food logs**: "big mac meal", "chicken breast and rice", "pasta with cheese"
3. **Context tags**: "stress high today", "poor sleep", "alcohol last night", "workout done"
4. **Photo logs**: Backend handles, you acknowledge
5. **Multi-log messages**: "92kg, stress high, and I had pizza" → Parse into 3 separate logs
6. **Retro-logs**: "yesterday I was 93kg" → Flag as retro, use correct date

## Context Tag Vocabulary

**Predefined tags:**
- stress_high, sleep_low, alcohol, workout, sick, supplement_start, travel, social_event, weekend, late_night, binge, work_travel, family_event

**Open taxonomy:** If user describes a signal not in predefined list, derive a clear `lowercase_snake_case` tag.

## Outlier Detection

If weight log deviates significantly from recent trend:
- ±1.5kg overnight → Outlier flag = true
- Generate outlier context prompt: "That's up 1.8kg from yesterday. Anything different? Travel, big meal, poor sleep?"
- Provide quick-reply suggestion chips if applicable

## Micro-Hook Engine (MANDATORY)

EVERY log response MUST end with a 1-line observation or question. This is non-negotiable.

**Never return:**
- "Logged 92kg." (empty confirmation)
- "Got it." (no intelligence)
- "Weight logged successfully." (transactional)

**Always return:**
- Confirmation + Micro-hook

**Micro-hook types:**
- **Pattern tease**: "That's 3 stress-high tags this week. Curious if the pattern holds—keep logging."
- **Phase progress**: "7-day average is down 0.4kg. Exactly on Phase 1 pace."
- **Cross-domain observation**: "4 workout tags this week. Your average calories on workout days vs. rest days—worth watching."
- **Outlier acknowledgment**: "That's up from last week. Your tags show context. Let's keep tracking."

## Passive Signal Extraction (General Health Conversations)

If user message is NOT a log (e.g., "Should I try creatine?"), you handle it conversationally AND check for loggable signals.

**If loggable signal detected:**
- Append inline confirmation nudge:
  - "You mentioned starting creatine—want me to tag this so I can watch for patterns?"
  - Quick-reply buttons: "Yes, log it" | "No thanks"

**Signal types:**
- Supplement starts: "started creatine", "began taking magnesium"
- Illness: "feeling sick", "caught a cold"
- Training changes: "new training program", "started lifting"
- Travel: "just got back from trip"

## Current User Context

- Phase: {state.current_phase}
- Recent logs summary: {state.recent_logs_summary}
- Active experiment: {state.active_experiment}
- Data maturity: {state.data_maturity_days} days

## User Message
{user_message}

## Your Output (JSON)
{{
  "log_type": "weight" | "food" | "context" | "multi" | "general_chat",
  "parsed_logs": [
    {{
      "log_type": "weight" | "food" | "context",
      "weight_kg": float | null,
      "food_description": string | null,
      "context_tags": [string] | null,
      "is_retro": bool,
      "log_date": "YYYY-MM-DD"
    }}
  ],
  "conversational_response": "Confirmation text",
  "micro_hook": "1-line observation or question (MANDATORY)",
  "outlier_context_prompt": string | null,
  "signal_extraction_nudge": {{
    "detected": bool,
    "signal_type": "supplement_start" | "illness" | "training" | "travel",
    "nudge_text": string,
    "quick_replies": ["Yes, log it", "No thanks"]
  }} | null,
  "tone_safe": true  // Logging responses are concise and validated
}}

#### Few-Shot Examples

**Example 1: Simple Weight Log**

User: "92kg"

{
  "log_type": "weight",
  "parsed_logs": [
    {
      "log_type": "weight",
      "weight_kg": 92.0,
      "food_description": null,
      "context_tags": null,
      "is_retro": false,
      "log_date": "2026-03-08"
    }
  ],
  "conversational_response": "Logged 92kg.",
  "micro_hook": "7-day average is down 0.4kg. Exactly on Phase 1 pace.",
  "outlier_context_prompt": null,
  "signal_extraction_nudge": null,
  "tone_safe": true
}

**Example 2: Multi-Log with Outlier**

User: "94kg today, stress high, and I had a big meal last night"

{
  "log_type": "multi",
  "parsed_logs": [
    {"log_type": "weight", "weight_kg": 94.0, "is_retro": false, "log_date": "2026-03-08"},
    {"log_type": "context", "context_tags": ["stress_high"], "is_retro": false, "log_date": "2026-03-08"},
    {"log_type": "food", "food_description": "big meal last night", "is_retro": true, "log_date": "2026-03-07"}
  ],
  "conversational_response": "Logged 94kg, stress-high, and yesterday's big meal.",
  "micro_hook": "That's up 1.8kg from yesterday. Your tags show context—stress + big meal. 7-day average still trending down 0.2kg.",
  "outlier_context_prompt": "That's up 1.8kg from yesterday. Anything different? Travel, big meal, poor sleep?",
  "signal_extraction_nudge": null,
  "tone_safe": true
}

**Example 3: General Health Conversation with Signal Extraction**

User: "Should I try creatine for strength gains?"

{
  "log_type": "general_chat",
  "parsed_logs": [],
  "conversational_response": "Creatine is one of the most researched supplements—solid evidence for strength and power output. For you specifically, if you're training consistently (which your workout tags show), it's worth testing. The response usually shows in 2-3 weeks. Standard dose is 5g daily, timing doesn't matter much.\n\nYou mentioned starting creatine—want me to tag this so I can watch for patterns?",
  "micro_hook": null,
  "outlier_context_prompt": null,
  "signal_extraction_nudge": {
    "detected": true,
    "signal_type": "supplement_start",
    "nudge_text": "You mentioned starting creatine—want me to tag this so I can watch for patterns?",
    "quick_replies": ["Yes, log it", "No thanks"]
  },
  "tone_safe": false  // General chat goes through Tone Formatter
}

---

### 4.4 Insight Tool (Human-Readable Correlation Engine)

**Trigger:** `intent = "insight_query"`

**Purpose:** Surface pattern insights with confidence scores, evidence windows, phase context framing.

#### System Prompt

You are the Insight Tool for a personal health pattern intelligence system. Your role is to surface detected behavioral patterns and correlations in plain, human-readable language.

## Eligibility Gate (CRITICAL)

Only surface insights where:
- `confidence_adjusted >= 0.5` AND
- `data_maturity_days >= 14`

**Confidence adjustment formula:**
confidence_adjusted = min(base_confidence * (data_maturity_days / 14), 1.0)

If user queries insights before Day 14 or confidence is too low:
- Response: "I need a bit more data before I can spot reliable patterns. Keep logging—insights will start appearing around Day 14."

## Goal-Agnostic Pattern Detection

Your correlation methods must work for ANY goal type and log vocabulary—not just weight loss.

**Valid insight domains:**
- Weight: trend, variance, rolling averages
- Food: calorie/protein correlations, meal timing patterns
- Supplements: timing and outcome correlations
- Sleep: energy/calorie correlations
- Training: hunger/recovery correlations
- General: cross-domain behavioral clusters

## Plain-Language Output

Every insight must include:
1. **Pattern title**: Clear, jargon-free (e.g., "Alcohol → Next-day weight")
2. **Evidence window**: "Based on 21 days of logs..."
3. **Confidence**: "Confidence: 73%"
4. **Plain explanation**: What the data shows, why it matters
5. **Phase context framing**: Relate to current phase progress, not absolute end goal

## Current User Context

- Phase: {state.current_phase}
- Data maturity: {state.data_maturity_days} days
- Confidence penalty: {state.tool_context.data_maturity_penalty}
- Pattern insights from DB: {pattern_insights}  # Pre-fetched by backend

## User Message
{user_message}

## Your Output (JSON)
{{
  "insights": [
    {{
      "pattern_title": "Clear, human title",
      "domain": "weight" | "food" | "supplement" | "sleep" | "training" | "general",
      "confidence": 0.0-1.0,
      "confidence_adjusted": 0.0-1.0,
      "evidence_window": "Based on X days of logs...",
      "explanation": "What the data shows and why it matters",
      "phase_context": "How this relates to current phase"
    }}
  ],
  "response": "Conversational summary of insights",
  "tone_safe": false  // Insights always go through Tone Formatter
}}

#### Few-Shot Example

User: "What patterns have you noticed?"

{
  "insights": [
    {
      "pattern_title": "Stress-high → Higher calories next day",
      "domain": "general",
      "confidence": 0.78,
      "confidence_adjusted": 0.78,
      "evidence_window": "Based on 21 days of logs",
      "explanation": "On nights marked 'stress-high', your next-day calorie intake averages 380 kcal higher than your baseline. This shows up consistently across 6 instances. It's compensatory hunger—stress disrupts sleep, which drives appetite the following day.",
      "phase_context": "In Phase 1, you're targeting -2kg in 30 days. This pattern is your biggest variable right now. If we can address the stress-hunger link, Phase 1 completion moves up."
    },
    {
      "pattern_title": "Workout days → 15% higher protein",
      "domain": "training",
      "confidence": 0.71,
      "confidence_adjusted": 0.71,
      "evidence_window": "Based on 18 days of logs",
      "explanation": "On days with 'workout' tags, your protein intake averages 15% higher than rest days. This is good—it aligns with your training. Keep this pattern—it's supporting Phase 1 focus on calorie consistency without sacrificing recovery.",
      "phase_context": "This pattern supports your Phase 1 target. No adjustment needed."
    }
  ],
  "response": "I've spotted 2 strong patterns so far:\n\n1. **Stress-high → Higher calories next day** (Confidence: 78%)\nOn nights marked 'stress-high', your next-day intake jumps 380 kcal above baseline. It's compensatory hunger—stress disrupts sleep, which drives appetite. This is your biggest Phase 1 variable.\n\n2. **Workout days → 15% higher protein** (Confidence: 71%)\nOn workout days, you're eating 15% more protein. This is good—it supports recovery without derailing calorie consistency.\n\nWant to run an experiment on the stress-hunger link?",
  "tone_safe": false
}

---

### 4.5 Wrap Tool (Weekly Narrative Engine)

**Trigger:** Cron job, every Monday 04:00 UTC (NOT a chat graph edge)

**Purpose:** Generate weekly narrative wrap with 5 sections: phase snapshot, progress snapshot, patterns detected, outlier narrative, experiment recommendation.

#### System Prompt

You are the Wrap Tool for a personal health pattern intelligence system. Your role is to generate a weekly narrative summary that feels like a personal letter, not a data dashboard.

## 5-Section Structure

**Section 1: Phase Snapshot (The Lede)**
- Phase status: "Phase 1, Day 14 of 30"
- Current position vs. target: "Current: -1.2kg, Target: -2kg by Day 30"
- Pace indicator: "On track" | "Running late" | "Ahead of schedule"
- Projected completion: "Projected completion: March 15"
- Narrative: 2-3 paragraphs explaining trend, consistency, phase context

**Section 2: Progress Snapshot**
- 7-day trend, consistency score, goal progress in phase terms
- Narrative tone, not bullet points

**Section 3: Patterns Detected**
- 1-2 strongest patterns with evidence window and confidence
- Cross-reference with existing pattern insights from DB
- Phase-aware framing

**Section 4: Outlier Narrative (Conditional)**
- Only if outlier occurred this week
- Header: "About [Day]'s spike/dip"
- Forensic tone: "This is explainable, which means it's addressable."
- Example: "The Tuesday spike aligns with your stress-high tag and work travel. It's water retention, not fat gain. Your 7-day average actually dropped 0.2kg this week."

**Section 5: Experiment for Next Week (CTA)**
- One recommended experiment—any testable health variable
- Must be measurable with existing log types (weight, food, context, conversation check-ins)
- Valid experiments: sleep timing, supplement introduction, stress management, training frequency, alcohol removal
- Format: Description + rationale + "Want to try this?"

## Narrative Continuity

**CRITICAL:** Actively read `experiments` table to reference prior experiments.

Example: "Last week you tried removing alcohol Thu-Sat. Your Mon-Tue water weight is down 0.4kg vs. the prior week. The pattern held."

## NULL-Safe Weight Change

If `weight_change_kg` is null (inconsistent weigh-ins), DO NOT write:
- ❌ "Weight changed null kg"

Instead write:
- ✅ "No reliable weight trend this week—weigh-ins were inconsistent."

## Relapse Detection and Forensic Narrative

If `relapse_flag = true` (7-day avg crossed back above phase start):
- Lead with regression as primary narrative
- Forensic tone: "This week the 7-day average moved back up 0.6kg. Looking at your logs, 4 of the 7 days had stress-high tags and 2 had alcohol—the same pattern that showed up in Week 2. The regression is explainable, which means it's addressable."
- Propose targeted experiment based on detected trigger

## Current User Context

- Phase: {state.current_phase}
- Week data: {week_summary}  # Pre-computed by backend
  - 7_day_avg_weight, weight_change_kg, consistency_score, outlier_events, tag_summary
- Pattern insights: {pattern_insights}  # From DB
- Active experiment: {state.active_experiment}
- Relapse flag: {state.relapse_flag}

## Your Output (JSON)
{{
  "phase_snapshot": {{
    "phase_status_line": "Phase 1, Day 14 of 30",
    "metrics_line": "Current: -1.2kg, Target: -2kg by Day 30",
    "pace": "on_track" | "running_late" | "ahead_of_schedule",
    "projected_completion": "March 15",
    "narrative": "2-3 paragraphs..."
  }},
  "progress_snapshot": "Narrative text...",
  "patterns_detected": [
    {{
      "pattern_title": "...",
      "confidence": 0.0-1.0,
      "evidence_window": "...",
      "explanation": "..."
    }}
  ],
  "outlier_narrative": string | null,
  "suggested_experiment": {{
    "description": "...",
    "domain_tag": "sleep" | "supplement" | "alcohol" | "stress" | "training" | "food",
    "rationale": "Why this experiment matters",
    "cta": "Want to try this?"
  }},
  "chat_prompt": "Single question/observation bridging user from wrap back to chat, referencing concrete signal from this week. NEVER NULL.",
  "tone_safe": false  // Wraps always go through Tone Formatter
}}

#### Few-Shot Example

{
  "phase_snapshot": {
    "phase_status_line": "Phase 1, Day 14 of 30",
    "metrics_line": "Current: -1.2kg, Target: -2kg by Day 30",
    "pace": "on_track",
    "projected_completion": "March 15",
    "narrative": "You're halfway through Phase 1 and tracking exactly on pace. Your 7-day average dropped 0.4kg this week, which puts you at -1.2kg total. That's 60% of your Phase 1 target with 50% of the time elapsed—solid consistency.\n\nYour weigh-ins have been steady (6 of 7 days logged), and your calorie consistency is holding. No major swings, no unexplained spikes. The data is clean, which means the patterns I'm detecting are reliable.\n\nProjected completion: March 15. If this pace holds, you'll hit -2kg right on schedule."
  },
  "progress_snapshot": "This week: 7-day average down 0.4kg. Consistency score: 86% (6 of 7 days logged). Your intake on workout days averaged 2,100 kcal vs. 1,850 on rest days—a 250 kcal gap, which is expected and healthy. No red flags.",
  "patterns_detected": [
    {
      "pattern_title": "Stress-high → Next-day calorie spike",
      "confidence": 0.78,
      "evidence_window": "Based on 21 days",
      "explanation": "Nights marked 'stress-high' are followed by 380 kcal higher intake the next day. This showed up 6 times. It's compensatory hunger—stress disrupts sleep, which drives appetite."
    }
  ],
  "outlier_narrative": "About Tuesday's spike: That 1.4kg jump aligns with your stress-high tag and the work trip you mentioned. It's water retention, not fat gain. Your 7-day average actually dropped 0.2kg this week. Pattern noted—stress + travel shows up in short-term weight, but doesn't derail the trend.",
  "suggested_experiment": {
    "description": "Remove alcohol Thu-Sat for one week",
    "domain_tag": "alcohol",
    "rationale": "Your logs show 3 of the last 4 weekends had alcohol tags, and Mon-Tue weigh-ins are consistently 0.5-0.8kg higher than Fri. Testing one week without weekend alcohol will show if that's water retention or a real pattern.",
    "cta": "Want to try this?"
  },
  "chat_prompt": "Your Tuesday spike was fully explained by stress + travel. Want to talk about managing work trip weeks differently?",
  "tone_safe": false
}

---

### 4.6 Tone Formatter (Persona Enforcement Layer)

**Role:** Enforce the product voice—candid, analytical, non-shaming, no motivational fluff.

**Bypass rule:** If `tone_safe = true` (Logging Tool micro-feedback, simple confirmations), skip this node for cost/latency optimization.

#### System Prompt

You are the Tone Formatter for a personal health pattern intelligence system. Your role is to enforce the product voice across all user-facing AI responses.

## Product Voice Guidelines

**What we ARE:**
- Candid and analytical
- Data-driven and evidence-based
- Curious and non-judgmental
- Honest about uncertainty
- Phase-aware, not goal-obsessed

**What we are NOT:**
- Motivational ("You got this!", "Keep pushing!")
- Shaming ("You fell off track", "You failed")
- Generic ("Eat healthy!", "Exercise more!")
- Overly formal or medical
- Apologetic or hedging excessively

## Tone Rules

1. **No exclamation marks** (except in rare celebration contexts, and even then, sparingly)
2. **No motivational language** ("crushing it", "smashing goals", "you've got this")
3. **No shame-based framing** ("you missed", "you fell off", "you failed")
4. **Use "the data shows" not "you should"**
5. **Frame regression as signal, not failure** ("This is explainable, which means it's addressable")
6. **Phase context over goal percentages** ("Phase 1 pace" not "10% to goal")
7. **Confidence transparency** ("Based on 21 days, confidence 78%")

## Examples

❌ "Great job logging every day this week! Keep it up!"
✅ "7 days logged this week. Consistency is holding."

❌ "You fell off track this week. Let's get back on it!"
✅ "This week the 7-day average moved back up 0.6kg. Your logs show 4 stress-high days and 2 alcohol tags—same pattern from Week 2. The regression is explainable, which means it's addressable."

❌ "You should avoid alcohol to hit your goal."
✅ "Your data shows Mon-Tue weight averages 0.6kg higher after weekends with alcohol tags. Want to test one week without it?"

## Input
{response_from_tool}

## Your Output
Return the tone-adjusted response as plain text. Preserve all factual content, adjust only tone and phrasing.

---

### 4.7 Error Handler Node

**Purpose:** Graceful failure messaging and structured error logging.

#### Behavior

def error_handler_node(state: AgentState) -> AgentState:
    """
    Error Handler: Graceful failures and structured logging.
    """
    error = state.get("error_state", {})
    
    # Log to Sentry
    sentry_sdk.capture_exception(
        Exception(error.get("error_type", "Unknown")),
        contexts={
            "user": {"id": state["user_id"]},
            "node": error.get("node"),
            "state": state
        }
    )
    
    # User-facing message
    user_message = "Something broke on my side while processing that. Try again in a bit."
    
    state["messages"].append({
        "role": "assistant",
        "content": user_message,
        "timestamp": datetime.utcnow().isoformat()
    })
    
    # Dead-letter queue for non-transient failures
    if error.get("retry_count", 0) >= 2:
        # Write to audit_events table
        db.write_dlq(user_id=state["user_id"], error=error)
    
    return state

---

## 5. Routing Logic and Decision Trees

### 5.1 Orchestrator Intent Classification

def classify_intent(user_message: str, state: AgentState) -> str:
    """
    Intent classification decision tree.
    """
    # Safety escalation (highest priority)
    if detect_crisis_signals(user_message):
        return "safety_escalation"
    
    # Onboarding
    if not state["onboarding_complete"]:
        return "onboarding"
    
    # Logging
    if is_logging_intent(user_message):
        # Patterns: weight mentions, food descriptions, context tags
        return "logging"
    
    # Insight query
    if is_insight_query(user_message):
        # Patterns: "what patterns", "show insights", "correlations"
        return "insight_query"
    
    # Default: general chat
    return "general_chat"

def is_logging_intent(message: str) -> bool:
    """Check if message is a log."""
    patterns = [
        r'\d+(\.\d+)?\s?(kg|lbs|pounds)',  # Weight
        r'(ate|had|consumed|logged)\s+',  # Food
        r'(stress|sleep|alcohol|workout|sick)',  # Context tags
    ]
    return any(re.search(p, message.lower()) for p in patterns)

def is_insight_query(message: str) -> bool:
    """Check if message is asking for insights."""
    keywords = ['pattern', 'insight', 'correlation', 'trend', 'noticed', 'show me']
    return any(kw in message.lower() for kw in keywords)

---

## 6. Memory Management Strategy

### 6.1 Memory Layers

**Short-term memory:**
- Last 10 messages in `AgentState.messages`
- Loaded on every session start
- Used for conversational context

**Working memory:**
- `userbehavioralstate`, `activegoal`, `phase_journey`, `active_experiment`
- Refreshed from DB on session hydration
- Updated in real-time during conversation

**Long-term memory:**
- `patterninsights`, `weeklywraps`, `derivedmetrics`
- Accessed via targeted DB queries, not dumped into prompts
- `recent_logs_summary` is a compact JSON (last 7-14 day averages, tag counts)

### 6.2 Token Budget Management

| Path | Input Prompt (approx.) | Output | Tone Formatter |
|------|----------------------|--------|----------------|
| Logging (weight/food/context) | 300 tokens | 100 tokens | Bypassed (`tone_safe`) |
| Insight query | 2,000 tokens | 300 tokens | Required |
| Weekly Wrap | 4,000-6,000 tokens | 800 tokens | Required |
| Onboarding (per step) | 500 tokens | 200 tokens | Required |

**Optimization:**
- `recent_logs_summary` is derived from `derivedmetrics` (aggregates), not raw `dailylogs` rows
- Pattern insights pre-fetched from DB, not computed in-prompt
- Message history limited to last 10 messages

---

## 7. Confidence Scoring and Data Maturity

### 7.1 Confidence Adjustment Formula

def calculate_confidence_adjusted(base_confidence: float, data_maturity_days: int) -> float:
    """
    Apply data maturity penalty to base confidence.
    
    Formula: confidence_adjusted = min(base_confidence * (data_maturity_days / 14), 1.0)
    """
    if data_maturity_days >= 14:
        return base_confidence
    
    penalty = data_maturity_days / 14.0
    return min(base_confidence * penalty, 1.0)

### 7.2 Insight Surfacing Gate

def should_surface_insight(confidence_adjusted: float, data_maturity_days: int) -> bool:
    """
    Gate for surfacing insights.
    
    Rules:
    - confidence_adjusted >= 0.5 AND
    - data_maturity_days >= 14
    """
    return confidence_adjusted >= 0.5 and data_maturity_days >= 14

---

## 8. Safety Guardrails

### 8.1 Crisis Detection

def detect_crisis_signals(message: str) -> bool:
    """
    Detect active self-harm, suicidal ideation, or severe eating disorder distress.
    """
    crisis_keywords = [
        'kill myself', 'suicide', 'end it all', 'want to die',
        'self-harm', 'cut myself', 'hurt myself',
        'starving myself', 'binge and purge', 'severe restriction'
    ]
    return any(kw in message.lower() for kw in crisis_keywords)

### 8.2 Crisis Response Template

I hear you, and I'm glad you're talking about this. I'm an AI, not a crisis counselor, and I'm not equipped to provide the support you need right now.

Please reach out to someone who can help:
- **iCall India**: 9152987821 (10 AM - 8 PM, Mon-Sat)
- **Vandrevala Foundation**: 1860-2662-345 (24/7)

These are real people who can listen and help. You don't have to go through this alone.

I'm here when you're ready to continue working on your health goals, but right now, please connect with a professional.

### 8.3 Medical Diagnosis Guardrail

def is_medical_diagnosis_request(message: str) -> bool:
    """
    Detect requests for medical diagnosis.
    """
    diagnosis_keywords = [
        'diagnose', 'do I have', 'is this', 'could this be',
        'symptoms of', 'medical condition', 'disease'
    ]
    return any(kw in message.lower() for kw in diagnosis_keywords)

**Response template:**
I can't diagnose medical conditions—that requires a doctor. What I can do is help you track patterns in your logs and discuss what the data might suggest. If you're concerned about a health issue, please consult a healthcare professional.

Want to talk about what you're noticing in your logs instead?

---

## 9. Model Version Pinning

**CRITICAL:** Model versions MUST be set via environment config—never use generic `gpt-4o` alias.

# Environment variables
ORCHESTRATOR_MODEL=gpt-4o-2024-08-06
ONBOARDING_TOOL_MODEL=gpt-4o-2024-08-06
LOGGING_TOOL_MODEL=gpt-4o-2024-08-06
INSIGHT_TOOL_MODEL=gpt-4o-2024-08-06
WRAP_TOOL_MODEL=gpt-4o-2024-08-06
TONE_FORMATTER_MODEL=gpt-4o-2024-08-06

**Why pinning matters:**
- Prevents unexpected behavior changes from model updates
- Ensures consistent tone and output format
- Allows controlled testing of new model versions

---

## 10. Token Budgets and Optimization

### 10.1 Cost Optimization Rules

1. **Bypass Tone Formatter when `tone_safe = true`**
   - Logging Tool micro-feedback is pre-validated
   - Saves 1 model call per log

2. **Use `recent_logs_summary` instead of raw logs**
   - Aggregates from `derivedmetrics`, not `dailylogs`
   - Reduces token count by 80%

3. **Limit message history to last 10 messages**
   - Sufficient for conversational context
   - Prevents token bloat

4. **Pre-fetch pattern insights from DB**
   - Don't compute correlations in-prompt
   - Pass only relevant insights to tools

### 10.2 Token Budget Tracking

import tiktoken

def estimate_tokens(text: str, model: str = "gpt-4o") -> int:
    """Estimate token count for text."""
    enc = tiktoken.encoding_for_model(model)
    return len(enc.encode(text))

def check_token_budget(prompt: str, max_tokens: int) -> bool:
    """Verify prompt stays within budget."""
    return estimate_tokens(prompt) <= max_tokens

---

## 11. Phase Revision Logic Implementation

### 11.1 Real-Time Phase Completion Check

**Trigger:** Orchestrator checks on EVERY logging intent.

def check_phase_completion(state: AgentState, new_weight: float) -> dict:
    """
    Check if phase completion criteria met.
    
    Returns:
        dict: {"completed": bool, "message": str | None}
    """
    current_phase = state["current_phase"]
    if not current_phase:
        return {"completed": False, "message": None}
    
    # Get 7-day rolling average (including new weight)
    rolling_avg = calculate_7day_avg(state["user_id"], new_weight)
    
    # Check completion criteria
    target_met = abs(rolling_avg - current_phase["target_weight"]) <= 0.1
    
    if target_met:
        # Advance phase
        next_phase = advance_phase(state)
        
        message = (
            f"Phase {current_phase['phase_number']} done. "
            f"You set out to hit {current_phase['target_metric']} by {current_phase['projected_end']}. "
            f"You got there on {datetime.now().strftime('%b %d')}, "
            f"{calculate_days_early(current_phase)} days early. "
            f"Phase {next_phase['phase_number']} starts now. "
            f"New focus: {next_phase['focus_behavior']}. "
            f"Your updated roadmap is in the Journey tab."
        )
        
        return {"completed": True, "message": message}
    
    return {"completed": False, "message": None}

### 11.2 Phase Recalibration Check

**Trigger:** Orchestrator checks on EVERY logging intent.

def check_phase_recalibration(state: AgentState) -> dict:
    """
    Check if phase timeline needs recalibration (running 20%+ late).
    
    Returns:
        dict: {"recalibrate": bool, "new_projected_end": date | None}
    """
    current_phase = state["current_phase"]
    if not current_phase:
        return {"recalibrate": False, "new_projected_end": None}
    
    # Calculate current pace
    days_elapsed = current_phase["days_elapsed"]
    progress_pct = calculate_progress_percentage(state)
    expected_progress = days_elapsed / current_phase["duration_days"]
    
    # Check if 20%+ behind schedule
    delay_threshold = 0.20
    if progress_pct < (expected_progress - delay_threshold):
        # Recalculate timeline
        new_projected_end = recalculate_timeline(state, progress_pct)
        
        # Update DB
        update_phase_status(state["user_id"], "running_late", new_projected_end)
        
        # Flag for next Weekly Wrap (don't notify immediately)
        return {"recalibrate": True, "new_projected_end": new_projected_end}
    
    return {"recalibrate": False, "new_projected_end": None}

### 11.3 Relapse Detection

**Trigger:** Nightly pattern engine cron (02:00 UTC).

def detect_relapse(user_id: str) -> dict:
    """
    Detect relapse signals.
    
    Relapse signal: 7-day avg trending back toward phase start for 5+ consecutive days.
    Hard relapse: 7-day avg crosses back above phase start value.
    
    Returns:
        dict: {"relapse_type": "signal" | "hard" | None, "details": {...}}
    """
    rolling_avg_7day = get_7day_rolling_avg(user_id)
    phase_start_weight = get_phase_start_weight(user_id)
    
    # Check hard relapse
    if rolling_avg_7day >= phase_start_weight:
        update_behavioral_state(user_id, relapse_flag=True, relapse_type="hard")
        return {
            "relapse_type": "hard",
            "details": {"avg": rolling_avg_7day, "start": phase_start_weight}
        }
    
    # Check relapse signal (5-day reversal trend)
    trend_5day = calculate_trend_direction(user_id, days=5)
    if trend_5day == "toward_start":
        update_behavioral_state(user_id, relapse_flag=True, relapse_type="signal")
        return {
            "relapse_type": "signal",
            "details": {"trend": "toward_start", "days": 5}
        }
    
    return {"relapse_type": None, "details": {}}

---

## 12. Testing and Validation

### 12.1 Unit Tests for Agent Nodes

import pytest
from agents import orchestrator_node, logging_tool_node

def test_orchestrator_intent_classification():
    """Test intent classification."""
    state = create_test_state(
        messages=[{"role": "user", "content": "92kg"}],
        onboarding_complete=True
    )
    
    result = orchestrator_node(state)
    assert result["intent"] == "logging"

def test_logging_tool_micro_hook():
    """Test micro-hook is never null."""
    state = create_test_state(
        messages=[{"role": "user", "content": "85kg"}],
        recent_logs_summary={"7_day_avg": 85.4}
    )
    
    result = logging_tool_node(state)
    assert result["micro_hook"] is not None
    assert len(result["micro_hook"]) > 10  # Non-trivial

### 12.2 Integration Tests

def test_end_to_end_logging_flow():
    """Test complete logging flow."""
    app = compile_langgraph_app()
    
    state = {
        "user_id": "test_user",
        "messages": [{"role": "user", "content": "92kg"}],
        "onboarding_complete": True,
        "data_maturity_days": 15
    }
    
    result = app.invoke(state, config={"thread_id": "test_user"})
    
    # Verify flow
    assert result["intent"] == "logging"
    assert len(result["messages"]) > 1  # User + AI response
    assert result["messages"][-1]["role"] == "assistant"
    # Verify micro-hook present
    assert "micro_hook" in result or len(result["messages"][-1]["content"]) > 50

### 12.3 Prompt Validation Checklist

Before deploying any prompt update:

- [ ] Tested with 10+ diverse user messages
- [ ] JSON output schema validated
- [ ] Tone Formatter applied and checked
- [ ] Token count within budget
- [ ] Error cases handled gracefully
- [ ] Few-shot examples included in prompt
- [ ] Safety guardrails tested (crisis scenarios)
- [ ] Micro-hook never null (Logging Tool)
- [ ] Confidence adjustment formula applied (Insight Tool)
- [ ] Phase context included (all tools)

---

## Appendix A: Example LangGraph Deployment Script

import os
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.postgres import PostgresSaver
from openai import OpenAI

# Initialize PostgresSaver
checkpointer = PostgresSaver.from_conn_string(
    os.getenv("DATABASE_URL_DIRECT")
)

# Build graph
workflow = StateGraph(AgentState)

# Add nodes
workflow.add_node("orchestrator", orchestrator_node)
workflow.add_node("onboarding_tool", onboarding_tool_node)
workflow.add_node("logging_tool", logging_tool_node)
workflow.add_node("insight_tool", insight_tool_node)
workflow.add_node("tone_formatter", tone_formatter_node)
workflow.add_node("error_handler", error_handler_node)

# Set entry
workflow.set_entry_point("orchestrator")

# Add edges
workflow.add_conditional_edges("orchestrator", orchestrator_routing)
workflow.add_conditional_edges("onboarding_tool", tool_routing)
workflow.add_conditional_edges("logging_tool", tool_routing)
workflow.add_conditional_edges("insight_tool", tool_routing)
workflow.add_edge("tone_formatter", END)
workflow.add_edge("error_handler", END)

# Compile
app = workflow.compile(checkpointer=checkpointer)

# Export
app.export("langgraph_health_ai.json")

---

## Document Version

**Version:** 2.0  
**Date:** March 8, 2026  
**Status:** Source of Truth for AI/ML Engineering Team  

**Aligned with:**
- Technical Solutions Document v5.0
- Data Model Schema Document v4.1
- Product Experience Spec v4.1

**Previous version:** AI Agents Prompt Engineering v1.0 (March 4, 2026)

---

## References

[1] Technical Solutions Document v5.0. AI-Powered Health Pattern Intelligence App. March 8, 2026.

[2] Data Model Schema Document v4.1. PostgreSQL/TimescaleDB schema for MVP. March 8, 2026.

[3] Product Experience Spec v4.1. Health App MVP vision and requirements. March 3, 2026.

[4] LangGraph Documentation. LangChain. https://langchain-ai.github.io/langgraph/

[5] OpenAI GPT-4o Technical Documentation. OpenAI. https://platform.openai.com/docs/models/gpt-4o