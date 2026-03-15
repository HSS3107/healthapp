**UI/UX Design Document v4.1**

**AI-Powered Health Pattern Intelligence App**

**Prepared for:** Frontend Development & UI Design Team  
**Date:** March 8, 2026  
**Status:** Source of Truth for Frontend Builds & Visual Design  
**Supersedes:** UI/UX Design Document v2.0  
**Aligned with:** Product Experience Spec v4.1, Solution Doc v4.1

**Document Purpose**

This document is the **comprehensive UI/UX specification** for building the frontend of the AI-powered health pattern intelligence app MVP. It translates the product vision and technical architecture into detailed screen designs, interaction patterns, component specifications, visual design requirements, and user flow definitions.

This document serves as the **single source of truth** for:

- Frontend developers building the Next.js PWA
- UI designers creating visual assets and design systems
- QA teams validating user experience flows
- Product stakeholders reviewing design decisions

**Table of Contents**

- [Design Philosophy & Principles](#design_philosophy_principles)
- [Visual Design System](#visual_design_system)
- [Screen Architecture & Navigation](#screen_architecture_navigation)
- [User Flow Definitions](#user_flow_definitions)
- [Detailed Screen Specifications](#detailed_screen_specifications)
- [Component Library](#component_library)
- [Interaction Patterns](#interaction_patterns)
- [Responsive Design & PWA Requirements](#responsive_design_pwa_requirements)
- [Accessibility Standards](#accessibility_standards)
- [Animation & Motion Design](#animation_motion_design)
- [Error States & Edge Cases](#error_states_edge_cases)
- [Performance Considerations](#performance_considerations)

**1\. Design Philosophy & Principles**

**1.1 Core Design Philosophy**

**The chat is everything.** The primary design challenge is making the chat interface feel like talking to a sharp expert, not using an app. There is no logging screen to design. There is no separate "Add Log" button.

**Key philosophical commitments:**

- **Intelligence over decoration** - Every UI element must serve the delivery of intelligence. No gamification. No streaks. No badges. No motivational fluff.
- **Chat-first architecture** - The chat interface is the sole input mechanism for logging, querying, and conversing. All other screens are reflection surfaces.
- **Offline states are first-class** - Design calm, subtle UI indicators for "Will sync when online." Never block user input due to connectivity.
- **Reality over perfection** - Visual language must embrace honesty. Phase regression is not failure-it's context. The arc moves backward on regression. This is a feature, not a flaw.
- **Micro-hooks as discovery** - Micro-hook messages must visually feel like a discovery, not a notification. They are part of the conversation, not a separate UI element.
- **Weekly Wrap as personal letter** - The wrap should feel like reading a personal letter or newsletter, not a data-heavy dashboard. Phase snapshot is the lede. Outlier narrative is the story. Experiment is the CTA.
- **Phase Journey as living timeline** - Not a progress bar. It is a living timeline-a visual arc that moves in both directions. Forward on progress, backward on regression. Honest, not punitive.

**1.2 Non-Negotiable Design Constraints**

- No logging screens-chat is the sole logging interface
- No gamification elements-no streaks, badges, points, or motivational language
- No alarm colors for regression-no red, no amber warnings
- No progress percentage bars-phase journey arc replaces them
- No separate notification center-micro-hooks live in chat
- No empty confirmations-every log response includes a micro-hook observation
- No shame-based re-engagement-no "You're behind" or "Get back on track" language
- No generic responses-every insight must be grounded in user's actual data

**1.3 User Experience Goals**

**By Day 1:** User understands their current position and sees their phase journey before logging a single entry.

**By Day 14:** User has discovered at least one non-obvious personal pattern and feels understood by the AI.

**By Week 4:** User returns to chat not just to log, but to think about their health-asking questions, exploring patterns, running experiments.

**Long-term:** The app becomes the user's health intelligence partner-the one place they turn for any health question, decision, or discussion.

**2\. Visual Design System**

**2.1 Color Palette**

**Primary Colors:**

| **Name**            | **Hex** | **Usage**                            |
| ------------------- | ------- | ------------------------------------ |
| Primary Teal        | #21808D | Primary actions, links, focus states |
| Primary Teal Hover  | #1D7480 | Hover states for primary actions     |
| Primary Teal Active | #1A6873 | Active/pressed states                |

Table 1: Primary color system

**Neutral Colors:**

| **Name**             | **Hex** | **Usage**                 |
| -------------------- | ------- | ------------------------- |
| Background Cream     | #FCFCF9 | Light mode background     |
| Surface White        | #FFFFFD | Light mode cards/surfaces |
| Background Charcoal  | #1F2121 | Dark mode background      |
| Surface Dark         | #262828 | Dark mode cards/surfaces  |
| Text Primary Light   | #13343B | Light mode primary text   |
| Text Secondary Light | #626C71 | Light mode secondary text |
| Text Primary Dark    | #F5F5F5 | Dark mode primary text    |
| Text Secondary Dark  | #A7A9A9 | Dark mode secondary text  |

Table 2: Neutral color system

**Semantic Colors:**

| **Name**       | **Hex** | **Usage**                                |
| -------------- | ------- | ---------------------------------------- |
| Success Teal   | #21808D | Phase on-track, positive patterns        |
| Info Gray      | #626C71 | Neutral information states               |
| Warning Orange | #A84B2F | Attention needed (not alarm)             |
| Error Red      | #C0152F | System errors only (never user behavior) |

Table 3: Semantic color system-CRITICAL: Never use error red or warning orange for user behavior states like regression or missed logs

**Phase State Colors (CRITICAL):**

| **Phase State** | **Visual Treatment**   |
| --------------- | ---------------------- |
| On track        | Primary Teal (#21808D) |
| Running late    | Info Gray (#626C71)    |
| Relapse signal  | Info Gray (#626C71)    |
| Hard relapse    | Info Gray (#626C71)    |
| Completed early | Success Teal (#21808D) |

Table 4: Phase state colors-NEVER use red or amber for regression states

**Rationale:** Regression is not failure-it's context. The arc position and recalibrated projected end date communicate status without alarm. Color remains neutral.

**2.2 Typography**

**Font Stack:**

font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI',  
'Roboto', 'Helvetica Neue', Arial, sans-serif;

**Type Scale:**

| **Level**  | **Size** | **Weight** | **Usage**                       |
| ---------- | -------- | ---------- | ------------------------------- |
| Display    | 32px     | 600        | Onboarding titles               |
| H1         | 28px     | 600        | Screen titles                   |
| H2         | 24px     | 600        | Section headers                 |
| H3         | 20px     | 600        | Subsection headers              |
| Body Large | 18px     | 400        | Chat messages (AI)              |
| Body       | 16px     | 400        | Chat messages (user), body text |
| Body Small | 14px     | 400        | Labels, secondary text          |
| Caption    | 12px     | 400        | Timestamps, micro-text          |

Table 5: Typography scale

**Chat Typography Rules:**

- User messages: 16px, regular weight (400), left-aligned
- AI messages: 18px, regular weight (400), left-aligned
- Micro-hooks: 16px, medium weight (500), subtle teal tint (#21808D at 80% opacity)
- Timestamps: 12px, regular weight (400), secondary text color
- Message status chips: 11px, regular weight (400), uppercase

**2.3 Spacing System**

**8px base grid system:**

| **Token** | **Value** | **Usage**                      |
| --------- | --------- | ------------------------------ |
| space-xs  | 4px       | Micro-spacing, icon padding    |
| space-sm  | 8px       | Compact spacing                |
| space-md  | 16px      | Standard spacing, card padding |
| space-lg  | 24px      | Section spacing                |
| space-xl  | 32px      | Major section breaks           |
| space-2xl | 48px      | Screen-level spacing           |

Table 6: Spacing tokens

**2.4 Border Radius**

| **Token** | **Value** | **Usage**            |
| --------- | --------- | -------------------- |
| radius-sm | 8px       | Buttons, chips, tags |
| radius-md | 12px      | Cards, input fields  |
| radius-lg | 16px      | Modal containers     |
| radius-xl | 24px      | Chat bubbles         |

Table 7: Border radius tokens

**2.5 Elevation & Shadows**

| **Level**   | **Shadow**                  |
| ----------- | --------------------------- |
| elevation-0 | none                        |
| elevation-1 | 0 1px 3px rgba(0,0,0,0.06)  |
| elevation-2 | 0 2px 8px rgba(0,0,0,0.08)  |
| elevation-3 | 0 4px 16px rgba(0,0,0,0.12) |

Table 8: Elevation system

**3\. Screen Architecture & Navigation**

**3.1 App Structure**

The app consists of **three primary screens** accessible via bottom navigation:

- **Chat** (Home) - Primary interface for logging, querying, conversing
- **Insights** - Reflection surface for Weekly Wraps and pattern insights
- **Profile** - Phase Journey View, data export, account controls

**Additional screens:**

- Onboarding Flow (5 steps, modal overlay)
- Weekly Wrap Detail (full-screen overlay from Insights)
- Phase Journey Detail (expandable from Profile)

**3.2 Bottom Navigation Bar**

**Position:** Fixed bottom, always visible (except during onboarding)

**Height:** 64px (including safe area insets)

**Items:**

| **Tab**  | **Icon**            | **Label** |
| -------- | ------------------- | --------- |
| Chat     | Message bubble icon | Chat      |
| Insights | Lightbulb icon      | Insights  |
| Profile  | User circle icon    | Profile   |

Table 9: Bottom navigation items

**States:**

- **Active:** Icon and label in Primary Teal (#21808D), subtle background tint
- **Inactive:** Icon and label in Text Secondary color
- **Notification badge:** Small red dot (8px) for new Weekly Wrap (Insights tab only)

**Behavior:**

- Tapping active tab scrolls to top of current screen
- Tab transitions use subtle fade (150ms ease-out)
- Bottom bar remains visible during scroll (no auto-hide)

**3.3 Top Navigation Bar**

**Height:** 56px (plus safe area insets)

**Background:** Surface color with bottom border (1px, border color)

**Contents vary by screen:**

**Chat Screen:**

- Left: App logo (wordmark, 24px height)
- Right: Offline status indicator (if offline), network icon

**Insights Screen:**

- Left: "Insights" title (H2)
- Right: Filter icon (future: filter by domain)

**Profile Screen:**

- Left: "Profile" title (H2)
- Right: Settings icon (future)

**4\. User Flow Definitions**

**4.1 Onboarding Flow (First Launch)**

**Goal:** Collect baseline data, accept user-stated goal, generate phase journey, lock Phase 1 before first log.

**Flow Structure:** 5-step conversational modal overlay, full-screen, non-dismissible until complete.

**Step-by-Step Flow:**

**Step 1: Current Stats**

**Screen Layout:**

- Progress indicator: 1/5 (dots at top)
- AI message: "Let's start with where you are right now. What's your current weight?"
- Input field: Number input with unit selector (kg/lbs)
- Secondary inputs (revealed after weight entered): Height, optional body fat estimate
- CTA button: "Next" (enabled after weight entered)

**Behavior:**

- User enters weight → height input appears
- User enters height → optional body fat prompt appears ("Have a body fat estimate? Optional.")
- "Next" becomes enabled after weight + height entered

**Step 2: Goal Capture**

**Screen Layout:**

- Progress indicator: 2/5
- AI message: "What's your goal? Tell me in your own words."
- Input field: Multi-line text area, placeholder: "e.g., Lose 10kg, build muscle, improve energy..."
- CTA button: "Next"

**Behavior:**

- User types goal (free text, no validation beyond min 3 characters)
- AI may display inline physiological floor check if goal implies unsafe rate:
  - Warning message (not blocking): "That's a 1.5% body weight loss per week. Research suggests 0.5-1% is more sustainable. You can proceed, but I'll flag if patterns suggest adjusting."
  - User can proceed regardless
- "Next" enabled after goal text entered

**Step 3: Context Signals**

**Screen Layout:**

- Progress indicator: 3/5
- AI message: "What makes this hard for you? Pick 2-3 things."
- Selection grid: Multi-select chips (max 3)
  - Stress eating
  - Poor sleep
  - Social events
  - Travel
  - Inconsistent schedule
  - Late-night hunger
  - Alcohol
  - Dining out
  - Weekend binges
  - Work stress
- CTA button: "Next" (enabled after at least 1 selected)

**Behavior:**

- User taps chips to select (max 3)
- Selected chips: Primary Teal background, white text
- Unselected chips: Light gray background, dark text
- After 3 selected, remaining chips become disabled (opacity 50%)

**Step 4: Lifestyle Context (Optional)**

**Screen Layout:**

- Progress indicator: 4/5
- AI message: "Anything else I should know about your lifestyle or schedule? (Optional)"
- Input field: Multi-line text area, placeholder: "e.g., Shift work, irregular sleep, training 5x/week..."
- CTA buttons: "Skip" (secondary), "Next" (primary)

**Behavior:**

- Both buttons enabled immediately (skip is valid)
- Optional free-text capture

**Step 5: Phase Journey Generation**

**Screen Layout:**

- Progress indicator: 5/5
- AI message: "Generating your roadmap..."
- Loading indicator: Animated dots (3 dots pulsing)
- After 2-3 seconds, phase journey appears:
  - Visual arc (simplified preview)
  - Phase 1 card:
    - Title: "Phase 1: \[Focus Behavior\]"
    - Duration: "~30 days"
    - Target: "\[Specific metric\], e.g., -2kg"
    - Focus area: "\[Primary behavior to watch\]"
  - Collapsed view of Phase 2, 3, 4... (just phase names, grayed out)
- CTA button: "Lock Phase 1 & Start Logging"

**Behavior:**

- User reviews phase journey
- Cannot edit at this stage (user sovereignty-AI computed based on stated goal)
- Tapping "Lock Phase 1" closes onboarding modal, navigates to Chat screen
- Phase 1 is now active, onboarding flag set to complete

**Resumption Logic:**

- If user closes app mid-onboarding (e.g., after Step 2), reopening app returns to exact step
- Progress indicator shows current step
- Data entered in previous steps is preserved (stored in local state)

**4.2 Primary Logging Flow (Chat Interface)**

**Entry Point:** Chat screen (default home screen after onboarding)

**Goal:** Allow user to log weight, food, context tags, or retro-logs via natural language chat.

**Flow:**

- User types in chat input field (bottom of screen)
- User sends message (tap send button or Enter key)
- User message appears as chat bubble (right-aligned, light teal background)
- Message status chip appears below bubble: "Logging..." → "Logged ✓" → "Synced ✓"
- AI response appears (left-aligned, white background card):
  - Structured log confirmation (weight, food, tags parsed)
  - Micro-hook (1-line observation or question, visually distinct-medium weight, subtle teal tint)
- Chat scrolls to show latest message
- User can continue conversation or enter next log

**Message Status States:**

| **Status**                | **Chip Text**    | **Visual Treatment**                   |
| ------------------------- | ---------------- | -------------------------------------- |
| Queueing                  | "Logging..."     | Gray, 11px uppercase, 50% opacity      |
| Logged (DB write)         | "Logged ✓"       | Gray, 11px uppercase, check icon       |
| Synced (offline → online) | "Synced ✓"       | Gray, 11px uppercase, check icon       |
| Failed                    | "Failed - Retry" | Red text, 11px uppercase, tap to retry |

Table 10: Message status chips

**Offline Behavior:**

- If offline, message status shows "Will sync when online" (gray, 11px)
- User can continue logging-messages queue locally
- When online, status updates to "Synced ✓"
- Offline indicator appears in top nav (small icon + "Offline" text)

**4.3 Outlier Context Prompt Flow**

**Trigger:** AI detects significant outlier (e.g., weight spike >1.5kg overnight)

**Flow:**

- User logs outlier data (e.g., "94kg today")
- AI response includes:
  - Log confirmation: "Logged: 94kg"
  - Outlier context prompt: "That's up 1.8kg from yesterday. Anything different? Travel, big meal, poor sleep?"
  - Input suggestion chips (optional quick replies):
    - "Travel"
    - "Big meal"
    - "Poor sleep"
    - "Alcohol"
    - "Nothing unusual"
- User can tap a chip or type free-form response
- AI acknowledges: "Got it-tagged as \[context\]. Your 7-day average is still trending down 0.3kg."

**Visual Treatment:**

- Outlier context prompt uses same conversational tone as other AI messages
- Not visually alarming-no red, no warning icons
- Suggestion chips: Small, rounded, light gray background, tap to select
- Feels like a curious question, not an interrogation

**4.4 Micro-Hook Interaction Flow**

**Goal:** Every log response includes a micro-hook-a 1-line AI observation or question designed to drive re-engagement.

**Visual Treatment:**

- Appears at end of AI log confirmation message
- Visually distinct from confirmation text:
  - Medium weight (500), subtle teal tint (#21808D at 80% opacity)
  - Separated by 8px vertical space from confirmation text
  - No border or background-integrated into message card

**Examples:**

- "Thats 3 stress-high tags this week. Curious if the pattern holds-keep logging."
- "7-day average is down 0.4kg. Exactly on Phase 1 pace."
- "4 workout tags this week. Your average calories on workout days vs. rest days-worth watching."

**User Response Options:**

- User can reply to micro-hook conversationally ("Why does that matter?", "Tell me more")
- AI engages in deeper discussion (opens Insight Tool path)
- Or user ignores and continues logging (no action required)

**Metric to Track:** Micro-hook reply rate (target: 30% within 24 hours)

**4.5 Weekly Wrap Flow**

**Trigger:** Every Monday, new Weekly Wrap generated (cron job, 4:00 AM UTC)

**Entry Point:** Notification badge appears on Insights tab in bottom nav

**Flow:**

- User taps Insights tab
- Insights screen shows latest Weekly Wrap card at top (hero position)
- Card preview:
  - Title: "Weekly Wrap: \[Date Range\]"
  - Subtitle: "Phase 1, Week 2"
  - Preview text: First 2 lines of phase snapshot
  - CTA: "Read Wrap" button
- User taps "Read Wrap"
- Full-screen overlay opens with complete wrap content:
  - Section 1: Phase Snapshot (lede)
  - Section 2: Progress Snapshot
  - Section 3: Patterns Detected (1-2 strongest patterns)
  - Section 4: Outlier Narrative (if applicable)
  - Section 5: Experiment for Next Week (CTA)
- User can scroll through wrap, then close (X button top-right)
- After first open, notification badge disappears

**Visual Treatment (Critical-Wrap Should Feel Like a Personal Letter):**

- **Typography:** Larger, more readable (18px body text vs. 16px in chat)
- **Spacing:** Generous whitespace between sections (32px)
- **Section headers:** H3 (20px, semi-bold), subtle teal accent line below
- **Tone:** Conversational, not dashboard-like-narrative flow, not bullet points
- **Experiment CTA:** Primary button at end, "Start This Experiment"

**Phase Snapshot Visual Structure:**

Phase 1, Day 14 of ~30  
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  
Current position: -1.2kg (target: -2kg by Day 30)  
Pace: On track ✓ | Projected completion: March 15

\[Narrative text explaining trend, consistency, phase context\]

**Outlier Narrative Example:**

About Tuesday's spike:

That 1.4kg jump aligns with your "stress-high" tag and  
the work travel you mentioned. It's water retention, not  
fat gain. Your 7-day average actually dropped 0.2kg this  
week. Pattern noted-stress + travel shows up in short-term  
weight, but doesn't derail the trend.

**4.6 Phase Journey View Flow**

**Entry Point:** Profile screen → "View Phase Journey" card

**Flow:**

- User taps "View Phase Journey" card in Profile screen
- Phase Journey Detail screen opens (full-screen)
- Visual arc displayed:
  - Horizontal timeline arc, left to right
  - Milestone dots for each phase
  - Current position marker (animated)
  - Phase labels below dots
  - Projected completion dates below labels
- User can tap any phase dot to see phase details (modal overlay):
  - Phase name, duration, target, focus behavior
  - Completion status (on track, running late, completed early, etc.)
- User can swipe between phases or close detail view

**Arc Visual Behavior for All Phase States:**

| **State**       | **Visual Treatment**                                               |
| --------------- | ------------------------------------------------------------------ |
| On track        | Marker on arc at current position, projected date shown            |
| Running late    | Marker position unchanged, projected date shifts right             |
| Relapse signal  | Marker moves backward on arc (no alarm color)                      |
| Hard relapse    | Marker returns toward phase origin, arc reshapes, timeline extends |
| Completed early | Marker reaches phase end, next phase begins, arc compresses        |

Table 11: Phase journey visual behavior-CRITICAL: Arc moves bidirectionally

**Recalibration Animation:**

- When phase timeline recalibrates (delay or relapse), arc smoothly animates to new shape
- Duration: 800ms ease-out
- User sees timeline update, not just notices it changed
- Projected dates update with fade transition (300ms)

**Milestone Dots:**

- Completed phases: Solid teal dot with checkmark
- Current phase: Pulsing teal dot (subtle animation)
- Future phases: Gray outline dot
- Dots persist as user moves through phases-journey history stays visible

**4.7 General Health Conversation Flow (Passive Signal Extraction)**

**Trigger:** User asks a health question in chat (e.g., "Should I try creatine?")

**Flow:**

- User types question in chat
- AI responds conversationally (engaging with question, not deflecting)
- **If** AI detects a loggable signal (supplement start, illness mention, training change):
  - AI appends inline confirmation nudge at bottom of response:
  - "You mentioned starting creatine-want me to tag this so I can watch for patterns?"
  - Two quick-reply buttons: "Yes, log it" | "No thanks"
- **If** user taps "Yes, log it":
  - Context log written with tag and free-text note
  - AI confirms: "Tagged: supplement-start (creatine). I'll watch for patterns."
- **If** user taps "No thanks" or ignores:
  - No log written, conversation continues normally

**Visual Treatment (Inline Confirmation Nudge):**

- Appears at bottom of AI message card, separated by subtle divider line (1px, light gray)
- Text: 14px, medium weight (500), Info Gray color
- Buttons: Small chips (height 32px), rounded corners (8px)
  - "Yes, log it": Primary Teal background, white text
  - "No thanks": Light gray background, dark text
- Feels optional, not intrusive-user can ignore and continue conversation

**Signal Types Detected:**

| **User Says**                 | **Tag Written**                        |
| ----------------------------- | -------------------------------------- |
| "Started creatine today"      | supplement-start (free-text: creatine) |
| "Feeling sick, skipping gym"  | sick                                   |
| "Terrible sleep last night"   | sleep-low                              |
| "Back from 3-day work trip"   | work-travel                            |
| "Starting new training block" | Free-text note in AgentState           |

Table 12: Signal extraction examples

**5\. Detailed Screen Specifications**

**5.1 Chat Screen (Home)**

**Layout Structure:**

┌─────────────────────────────────────┐  
│ Top Nav Bar (56px + safe area) │ ← Logo, offline indicator  
├─────────────────────────────────────┤  
│ │  
│ Chat Message List │  
│ (scrollable, flex-end aligned) │ ← Messages stack from bottom  
│ │  
│ \[User message bubble\] │  
│ \[AI response card\] │  
│ \[User message bubble\] │  
│ \[AI response card + micro-hook\] │  
│ ... │  
│ │  
├─────────────────────────────────────┤  
│ Chat Input Bar (64px + safe area) │ ← Text input, send button  
├─────────────────────────────────────┤  
│ Bottom Nav Bar (64px) │ ← Chat | Insights | Profile  
└─────────────────────────────────────┘

**Message Bubble Specifications:**

**User Message Bubble:**

- Alignment: Right (flexbox align-self: flex-end)
- Background: rgba(#21808D, 0.08)-light teal tint
- Text color: Text Primary
- Padding: 12px 16px
- Border radius: 24px 24px 4px 24px (right-bottom corner sharp)
- Max width: 80% of screen width
- Font: 16px, regular weight (400)
- Shadow: elevation-1

**AI Response Card:**

- Alignment: Left (flexbox align-self: flex-start)
- Background: Surface White (light mode) / Surface Dark (dark mode)
- Border: 1px solid border color (rgba(0,0,0,0.08))
- Padding: 16px
- Border radius: 24px 24px 24px 4px (left-bottom corner sharp)
- Max width: 85% of screen width
- Font: 18px, regular weight (400)
- Shadow: elevation-2

**AI Response Card Structure (with Micro-Hook):**

┌─────────────────────────────────────┐  
│ Logged: 92kg │ ← Confirmation text  
│ │  
│ ───────────────────────────── │ ← 8px vertical space  
│ │  
│ 7-day average is down 0.4kg. │ ← Micro-hook (500 weight,  
│ Exactly on Phase 1 pace. │ teal tint 80% opacity)  
└─────────────────────────────────────┘  
Logged ✓ 11:43 AM ← Status chip + timestamp

**Message Status Chip:**

- Position: Below message bubble (8px margin-top)
- Text: 11px, uppercase, regular weight (400)
- Color: Text Secondary (gray)
- Icon: Small checkmark or sync icon (12px)

**Chat Input Bar:**

┌─────────────────────────────────────┐  
│ \[Text input field\] \[Send\] │  
│ "Log weight, food, or ask me..." │ ← Placeholder text  
└─────────────────────────────────────┘

**Input Field:**

- Height: 48px (single line), expands to 96px (multi-line)
- Padding: 12px 16px
- Border radius: 24px (pill shape)
- Border: 1px solid border color
- Focus state: Border color changes to Primary Teal, 2px width
- Font: 16px, regular weight (400)

**Send Button:**

- Size: 48px × 48px (circular)
- Background: Primary Teal (enabled), light gray (disabled)
- Icon: Paper plane icon (white, 20px)
- Enabled only when input has text (min 1 character)
- Tap behavior: Sends message, clears input, disabled during send

**Offline Indicator (Top Nav):**

- Position: Top-right corner
- Icon: Cloud with slash (16px), gray color
- Text: "Offline" (12px, gray)
- Appears only when offline
- Tapping shows tooltip: "Will sync when online"

**Empty State (First Launch After Onboarding):**

┌─────────────────────────────────────┐  
│ │  
│ 👋 │ ← Centered icon  
│ │  
│ Ready to start Phase 1? │ ← H3 heading  
│ │  
│ Log your first weight, meal, │ ← Body text  
│ or just ask me a question. │  
│ │  
└─────────────────────────────────────┘

**5.2 Insights Screen**

**Layout Structure:**

┌─────────────────────────────────────┐  
│ Top Nav Bar │ ← "Insights" title  
├─────────────────────────────────────┤  
│ │  
│ Weekly Wrap Card (Hero) │ ← Latest wrap, large card  
│ ┌─────────────────────────────┐ │  
│ │ Weekly Wrap: Mar 1-7 │ │  
│ │ Phase 1, Week 2 │ │  
│ │ │ │  
│ │ Phase 1, Day 14 of ~30... │ │ ← Preview text  
│ │ │ │  
│ │ \[Read Wrap\] button │ │  
│ └─────────────────────────────┘ │  
│ │  
│ ───────────────────────────── │  
│ │  
│ Pattern Insights (List) │ ← Scrollable list  
│ ┌─────────────────────────────┐ │  
│ │ Pattern: Alcohol → Weight │ │  
│ │ Confidence: 73% │ │  
│ │ \[View Details\] │ │  
│ └─────────────────────────────┘ │  
│ ┌─────────────────────────────┐ │  
│ │ Pattern: Stress → Calories │ │  
│ │ Confidence: 68% │ │  
│ │ \[View Details\] │ │  
│ └─────────────────────────────┘ │  
│ ... │  
│ │  
├─────────────────────────────────────┤  
│ Bottom Nav Bar │  
└─────────────────────────────────────┘

**Weekly Wrap Card (Hero Position):**

- Size: Full width (minus 16px margin left/right), height auto (min 200px)
- Background: Surface White with subtle gradient (light teal tint at top)
- Border: 1px solid border color
- Border radius: 16px (radius-lg)
- Padding: 20px
- Shadow: elevation-2

**Card Content:**

- Title: "Weekly Wrap: \[Date Range\]" (H3, 20px, semi-bold)
- Subtitle: "Phase 1, Week 2" (14px, Text Secondary)
- Preview text: First 2 lines of phase snapshot (16px, Text Primary)
- CTA button: "Read Wrap" (primary button, bottom-right of card)

**Notification Badge:**

- Position: Top-right corner of card
- Visual: Small red dot (8px diameter)
- Appears when new wrap available (disappears after first open)

**Pattern Insights List:**

- Section header: "Pattern Insights" (H3, 20px, semi-bold, margin-top 32px)
- Empty state (if no patterns yet): "Keep logging-patterns will appear here after ~14 days."

**Pattern Insight Card:**

- Size: Full width, height auto (min 100px)
- Background: Surface White
- Border: 1px solid border color
- Border radius: 12px (radius-md)
- Padding: 16px
- Margin-bottom: 16px
- Shadow: elevation-1

**Card Content:**

- Pattern title: "Pattern: \[Tag\] → \[Outcome\]" (16px, semi-bold)
- Confidence: "Confidence: \[N\]%" (14px, Text Secondary)
- Evidence window: "Based on 21 days" (12px, Text Secondary)
- CTA: "View Details" link (14px, Primary Teal)

**Tapping "View Details":**

- Opens pattern detail modal (full-screen overlay)
- Shows complete insight explanation, evidence data, phase context

**5.3 Profile Screen**

**Layout Structure:**

┌─────────────────────────────────────┐  
│ Top Nav Bar │ ← "Profile" title  
├─────────────────────────────────────┤  
│ │  
│ User Info Card │  
│ ┌─────────────────────────────┐ │  
│ │ Dev │ │ ← User name (optional)  
│ │ Goal: Lose 10kg │ │  
│ │ Phase 1, Day 14 of ~30 │ │  
│ └─────────────────────────────┘ │  
│ │  
│ Phase Journey Card │  
│ ┌─────────────────────────────┐ │  
│ │ Phase Journey │ │  
│ │ \[Visual arc preview\] │ │ ← Simplified arc  
│ │ \[View Full Journey\] button │ │  
│ └─────────────────────────────┘ │  
│ │  
│ Data & Privacy │  
│ ┌─────────────────────────────┐ │  
│ │ Export My Data │ │  
│ │ Delete Account │ │  
│ └─────────────────────────────┘ │  
│ │  
│ About │  
│ Version 1.0.0 │  
│ │  
├─────────────────────────────────────┤  
│ Bottom Nav Bar │  
└─────────────────────────────────────┘

**User Info Card:**

- Background: Surface White
- Border: 1px solid border color
- Border radius: 12px
- Padding: 16px
- Shadow: elevation-1

**Content:**

- User name (if provided): H3, 20px, semi-bold
- Goal text: 16px, Text Primary
- Phase status: 14px, Text Secondary

**Phase Journey Card:**

- Background: Surface White
- Border: 1px solid border color
- Border radius: 12px
- Padding: 20px
- Shadow: elevation-1

**Visual Arc Preview (Simplified):**

- Horizontal line (2px height, light gray)
- 4-5 phase dots along line
- Current phase dot: Pulsing teal (12px diameter)
- Completed phases: Solid teal dot with checkmark (10px)
- Future phases: Gray outline dot (10px)

**CTA Button:** "View Full Journey" (primary button, centered)

**Data & Privacy Section:**

- Two list items (tap targets)
- Height per item: 56px
- Border-bottom: 1px solid border color (between items)
- Icon left (20px), text center-left (16px, semi-bold), chevron right (16px)

**Export My Data:**

- Tapping triggers data export flow (API call, download link sent via email or direct download)
- Confirmation modal: "Export includes all logs, insights, and phase data. Continue?"

**Delete Account:**

- Tapping shows confirmation modal with warning:
  - "This will permanently delete all your data. This cannot be undone."
  - Two buttons: "Cancel" (secondary), "Delete My Account" (destructive, red text)

**5.4 Weekly Wrap Detail Screen (Full-Screen Overlay)**

**Layout Structure:**

┌─────────────────────────────────────┐  
│ \[X\] Close button (top-right) │  
├─────────────────────────────────────┤  
│ │  
│ Weekly Wrap: Mar 1-7 │ ← H1 title  
│ Phase 1, Week 2 │ ← Subtitle  
│ │  
│ ═════════════════════════════ │ ← Section divider  
│ │  
│ Phase Snapshot │ ← Section 1 (H3 header)  
│ ───────────────────────────── │  
│ Phase 1, Day 14 of ~30 │  
│ Current position: -1.2kg │  
│ (target: -2kg by Day 30) │  
│ Pace: On track ✓ │  
│ Projected completion: March 15 │  
│ │  
│ \[Narrative text explaining trend, │  
│ consistency, phase context...\] │  
│ │  
│ ═════════════════════════════ │  
│ │  
│ Progress Snapshot │ ← Section 2  
│ ───────────────────────────── │  
│ \[Trend, consistency score, goal │  
│ progress narrative...\] │  
│ │  
│ ═════════════════════════════ │  
│ │  
│ Patterns Detected │ ← Section 3  
│ ───────────────────────────── │  
│ 1. Alcohol → Next-day weight │  
│ \[Evidence window, confidence\] │  
│ 2. Stress-high → Calorie intake │  
│ \[Evidence window, confidence\] │  
│ │  
│ ═════════════════════════════ │  
│ │  
│ Outlier Narrative │ ← Section 4 (if applicable)  
│ ───────────────────────────── │  
│ About Tuesday's spike: │  
│ \[Narrative explaining outlier...\] │  
│ │  
│ ═════════════════════════════ │  
│ │  
│ Experiment for Next Week │ ← Section 5 (CTA)  
│ ───────────────────────────── │  
│ \[Experiment description...\] │  
│ │  
│ \[Start This Experiment\] button │ ← Primary CTA  
│ │  
└─────────────────────────────────────┘

**Visual Treatment (CRITICAL-Feels Like a Personal Letter):**

- **Background:** Surface White (light mode) or Surface Dark (dark mode)
- **Padding:** 24px left/right, 32px top/bottom
- **Typography:** 18px body text (larger than chat), 1.6 line-height
- **Section headers:** H3 (20px, semi-bold), teal accent line below (2px, 40px width)
- **Section dividers:** Subtle gray line (1px, full width), 32px margin top/bottom
- **Tone:** Narrative, conversational-not bullet points or dashboard metrics

**Phase Snapshot Section:**

- **Structure:**
  - Phase status line: "Phase 1, Day 14 of ~30" (16px, semi-bold)
  - Metrics line: "Current position: -1.2kg (target: -2kg by Day 30)"
  - Pace indicator: "Pace: On track ✓" (or "Running late", "Ahead of schedule")
  - Projected completion: "Projected completion: March 15"
  - Narrative text: 2-3 paragraphs explaining trend, consistency, phase context
- **Visual treatment for pace indicator:**
  - "On track ✓": Success Teal color, checkmark icon
  - "Running late": Info Gray color, no icon (neutral)
  - "Ahead of schedule": Success Teal color, star icon

**Outlier Narrative Section (Conditional):**

- Only appears if outlier occurred this week
- Header: "About \[Day\]'s spike:" (or "dip", "change")
- Narrative text: 1-2 paragraphs explaining outlier in context (stress, travel, alcohol, etc.)
- Tone: Forensic, not punitive-"This is explainable, which means it's addressable."

**Experiment CTA Section:**

- **Structure:**
  - Header: "Experiment for Next Week" (H3)
  - Description: 2-3 sentences explaining experiment and rationale
  - CTA button: "Start This Experiment" (primary button, full width, centered)
- **Button behavior:**
  - Tapping button acknowledges experiment (writes to DB)
  - Closes wrap overlay, returns to Insights screen
  - Chat opens with confirmation message: "Experiment started: \[Brief description\]. I'll check in at next week's wrap."

**5.5 Phase Journey Detail Screen (Full-Screen)**

**Layout Structure:**

┌─────────────────────────────────────┐  
│ \[← Back\] Phase Journey │ ← Top nav with back button  
├─────────────────────────────────────┤  
│ │  
│ Visual Arc (Horizontal Timeline) │  
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │ ← Interactive arc  
│ ●────●────●────○────○ │  
│ │ │ │ │ │ │  
│ P1 P2 P3 P4 Goal │ ← Phase labels  
│ ✓ ✓ Now Later Later │ ← Status labels  
│ Mar15 Apr12 May10 Jun5 Jun30 │ ← Projected dates  
│ │  
│ ───────────────────────────── │  
│ │  
│ Current Phase Detail │  
│ ┌─────────────────────────────┐ │  
│ │ Phase 1: Initial Drop │ │ ← Phase name  
│ │ Duration: ~30 days │ │  
│ │ Target: -2kg │ │  
│ │ Focus: Calorie consistency │ │  
│ │ │ │  
│ │ \[Phase description text...\] │ │  
│ └─────────────────────────────┘ │  
│ │  
│ Phase History (Expandable) │  
│ \[Tap to view completed phases\] │  
│ │  
└─────────────────────────────────────┘

**Visual Arc Specifications:**

- **Timeline structure:**
  - Horizontal line (4px height, light gray base)
  - Active segment (from start to current position): Primary Teal gradient
  - Future segment (from current to end): Light gray
- **Phase dots:**
  - Size: 16px diameter
  - Completed phases: Solid teal with white checkmark (12px icon)
  - Current phase: Pulsing teal dot (subtle animation, 1.2s loop)
  - Future phases: Gray outline dot (2px border)
- **Phase labels:**
  - Position: Below each dot (12px margin-top)
  - Text: 14px, semi-bold
  - Color: Text Primary (completed/current), Text Secondary (future)
- **Status labels:**
  - Position: Below phase labels (4px margin-top)
  - Text: 12px, regular weight
  - Examples: "✓ Completed", "Now", "Later"
- **Projected dates:**
  - Position: Below status labels (4px margin-top)
  - Text: 12px, Text Secondary
  - Format: "Mar 15", "Apr 12"

**Arc Recalibration Animation (When Phase Extends or Compresses):**

- **Duration:** 800ms
- **Easing:** ease-out
- **Behavior:**
  - Dots slide to new positions along timeline
  - Dates fade out, update, fade in (300ms each)
  - Active segment extends or compresses smoothly
  - Current phase marker moves backward (relapse) or forward (progress)

**Current Phase Detail Card:**

- **Background:** Surface White
- **Border:** 1px solid border color
- **Border radius:** 12px
- **Padding:** 20px
- **Shadow:** elevation-2

**Card Content:**

- Phase name: H3, 20px, semi-bold
- Duration: 16px, Text Secondary
- Target: 16px, Text Secondary
- Focus behavior: 16px, Text Primary
- Phase description: 16px, Text Primary, 1.6 line-height, 2-3 sentences

**Phase History (Expandable Section):**

- **Collapsed state:** "Phase History" header with chevron-down icon
- **Tapping header:** Expands to show list of completed phases
- **Each completed phase card:**
  - Phase name, duration, actual completion date
  - "Completed \[early/on-time/late\]" status
  - Target met indicator

**6\. Component Library**

**6.1 Buttons**

**Primary Button:**

| **Property**  | **Value**              |
| ------------- | ---------------------- |
| Height        | 48px                   |
| Padding       | 12px 24px              |
| Border radius | 8px (radius-sm)        |
| Background    | Primary Teal (#21808D) |
| Text color    | White                  |
| Font          | 16px, semi-bold (600)  |
| Shadow        | elevation-1            |

Table 13: Primary button specifications

**States:**

- **Hover:** Background darkens to Primary Teal Hover (#1D7480)
- **Active/Pressed:** Background darkens to Primary Teal Active (#1A6873)
- **Disabled:** Background light gray (#E0E0E0), text gray, opacity 50%, no pointer events

**Secondary Button:**

| **Property**  | **Value**              |
| ------------- | ---------------------- |
| Height        | 48px                   |
| Padding       | 12px 24px              |
| Border radius | 8px                    |
| Background    | Transparent            |
| Border        | 2px solid Primary Teal |
| Text color    | Primary Teal           |
| Font          | 16px, semi-bold (600)  |

Table 14: Secondary button specifications

**States:**

- **Hover:** Background light teal tint (rgba(#21808D, 0.08))
- **Active/Pressed:** Background darker teal tint (rgba(#21808D, 0.12))
- **Disabled:** Border gray, text gray, opacity 50%

**Destructive Button (Delete Actions):**

| **Property**     | **Value**              |
| ---------------- | ---------------------- |
| Background       | Error Red (#C0152F)    |
| Text color       | White                  |
| Other properties | Same as primary button |

Table 15: Destructive button-use only for account deletion or irreversible actions

**6.2 Input Fields**

**Text Input:**

| **Property**  | **Value**                                 |
| ------------- | ----------------------------------------- |
| Height        | 48px (single line), 96px (multi-line)     |
| Padding       | 12px 16px                                 |
| Border radius | 8px                                       |
| Border        | 1px solid border color (rgba(0,0,0,0.12)) |
| Font          | 16px, regular (400)                       |
| Background    | Surface White                             |

Table 16: Input field specifications

**States:**

- **Focus:** Border changes to Primary Teal, 2px width, subtle glow shadow
- **Error:** Border changes to Error Red, 2px width, error message below field (14px, red text)
- **Disabled:** Background light gray, text gray, no pointer events

**Placeholder Text:**

- Color: Text Secondary (rgba gray, 60% opacity)
- Font: 16px, regular (400)
- Example: "Log weight, food, or ask me..."

**Number Input (Weight, Height):**

- Same as text input, with numeric keyboard on mobile
- Optional unit selector (dropdown): kg/lbs, cm/ft

**6.3 Chips (Tags, Quick Replies)**

**Selection Chip (Multi-select):**

| **Property**  | **Value**          |
| ------------- | ------------------ |
| Height        | 36px               |
| Padding       | 8px 16px           |
| Border radius | 8px                |
| Font          | 14px, medium (500) |

Table 17: Selection chip specifications

**States:**

- **Unselected:** Light gray background (#F5F5F5), dark text
- **Selected:** Primary Teal background, white text
- **Disabled:** Opacity 50%, no pointer events

**Context Tag Chip (Read-only):**

| **Property**  | **Value**            |
| ------------- | -------------------- |
| Height        | 28px                 |
| Padding       | 4px 12px             |
| Border radius | 14px (pill shape)    |
| Background    | Light gray (#F0F0F0) |
| Text color    | Text Primary         |
| Font          | 12px, medium (500)   |

Table 18: Context tag chip (display only, not interactive)

**6.4 Cards**

**Standard Card (Insights, Profile):**

| **Property**  | **Value**              |
| ------------- | ---------------------- |
| Background    | Surface White          |
| Border        | 1px solid border color |
| Border radius | 12px (radius-md)       |
| Padding       | 16px                   |
| Shadow        | elevation-1            |

Table 19: Standard card specifications

**Hero Card (Weekly Wrap Preview):**

| **Property**  | **Value**                                                   |
| ------------- | ----------------------------------------------------------- |
| Background    | Surface White with subtle gradient (light teal tint at top) |
| Border        | 1px solid border color                                      |
| Border radius | 16px (radius-lg)                                            |
| Padding       | 20px                                                        |
| Shadow        | elevation-2                                                 |
| Min height    | 200px                                                       |

Table 20: Hero card specifications

**6.5 Modals & Overlays**

**Full-Screen Overlay (Weekly Wrap Detail, Phase Journey Detail):**

| **Property** | **Value**                                        |
| ------------ | ------------------------------------------------ |
| Background   | Surface White (or Surface Dark in dark mode)     |
| Padding      | 24px left/right, 32px top/bottom                 |
| Close button | Top-right corner (X icon, 24px, tap target 44px) |
| Animation    | Slide up from bottom (300ms ease-out)            |

Table 21: Full-screen overlay specifications

**Confirmation Modal (Delete Account, Experiment Start):**

| **Property**  | **Value**                                |
| ------------- | ---------------------------------------- |
| Width         | 90% of screen width (max 400px)          |
| Background    | Surface White                            |
| Border radius | 16px                                     |
| Padding       | 24px                                     |
| Shadow        | elevation-3                              |
| Backdrop      | Semi-transparent black (rgba(0,0,0,0.5)) |

Table 22: Confirmation modal specifications

**Modal Structure:**

- Title: H3 (20px, semi-bold)
- Body text: 16px, Text Primary, 1.5 line-height
- Button row: Two buttons side-by-side (secondary left, primary/destructive right)

**Animation:**

- Modal: Scale up from 0.9 to 1.0 (200ms ease-out)
- Backdrop: Fade in (200ms)

**6.6 Loading Indicators**

**Spinner (Inline Loading):**

- Size: 20px diameter
- Color: Primary Teal
- Animation: Rotate 360deg, 1s linear infinite

**Skeleton Screen (Chat Message Loading):**

- Placeholder bubble with pulsing gray background (light gray → slightly darker gray, 1.5s loop)
- Same dimensions as typical AI response card

**Loading Dots (Onboarding Phase Generation):**

- 3 dots (8px diameter each), horizontal row, 8px spacing
- Animation: Each dot scales up/down sequentially (0.8 → 1.2 scale, 600ms loop)

**7\. Interaction Patterns**

**7.1 Chat Scroll Behavior**

**Auto-scroll on new message:**

- When new message appears (user or AI), chat scrolls to bottom smoothly (300ms ease-out)
- If user has scrolled up to view older messages, auto-scroll is paused
- "New message" indicator appears at bottom when paused (tap to scroll to bottom)

**Manual scroll:**

- User can scroll freely through message history
- Pull-to-refresh (future): Loads older messages if available

**7.2 Swipe Gestures**

**Not used in MVP.** Standard tap interactions only. Future consideration: Swipe to delete/edit messages.

**7.3 Pull-to-Refresh**

**Chat screen:**

- Pull down from top of message list to refresh (fetch latest messages from server)
- Refresh indicator: Spinner at top (20px diameter)

**Insights screen:**

- Pull down to check for new Weekly Wrap (even if notification badge hasn't appeared)

**Profile screen:**

- Pull down to refresh user data (goal, phase status)

**7.4 Haptic Feedback (Mobile PWA)**

**When to use:**

- Button press (light haptic)
- Chip selection (light haptic)
- Phase completion inline message appears (medium haptic)
- Error state (strong haptic)

**Guideline:** Use sparingly-only for significant interactions or confirmations.

**7.5 Keyboard Behavior (Chat Input)**

**Focus state:**

- Tapping chat input field brings up keyboard
- Chat scrolls to keep input field visible (above keyboard)
- Send button becomes enabled when text entered

**Enter key behavior:**

- Desktop: Enter sends message, Shift+Enter creates new line
- Mobile: Keyboard "Send" button sends message

**Dismissing keyboard:**

- Tapping outside input field dismisses keyboard (mobile)
- Scrolling up in chat keeps keyboard open

**8\. Responsive Design & PWA Requirements**

**8.1 Viewport Breakpoints**

| **Breakpoint** | **Width**      | **Target Device**     |
| -------------- | -------------- | --------------------- |
| Mobile         | 320px - 767px  | Phones                |
| Tablet         | 768px - 1023px | Tablets, large phones |
| Desktop        | 1024px+        | Desktop browsers      |

Table 23: Responsive breakpoints

**Design priority:** Mobile-first. The app is optimized for phone screens (375px - 414px width typical).

**8.2 Safe Area Insets (Notch/Island Support)**

**iOS devices with notch or Dynamic Island:**

- Use env(safe-area-inset-top) for top nav padding
- Use env(safe-area-inset-bottom) for bottom nav padding
- Chat input bar must clear bottom safe area (home indicator)

**Android devices:**

- Respect system navigation bar insets

**8.3 PWA Manifest & Install Behavior**

**Manifest.json specifications:**

{  
"name": "Health Pattern Intelligence",  
"short_name": "HealthAI",  
"start_url": "/",  
"display": "standalone",  
"background_color": "#FCFCF9",  
"theme_color": "#21808D",  
"orientation": "portrait-primary",  
"icons": \[  
{  
"src": "/icon-192.png",  
"sizes": "192x192",  
"type": "image/png"  
},  
{  
"src": "/icon-512.png",  
"sizes": "512x512",  
"type": "image/png"  
}  
\]  
}

**Install prompt:**

- Show browser-native install prompt after user logs first 3 entries (high engagement signal)
- Custom install banner (optional): Subtle banner at top of screen, dismissible
- Banner text: "Install app for offline access and notifications"

**Offline capabilities:**

- Service worker caches app shell (HTML, CSS, JS)
- IndexedDB stores queued logs and recent messages
- Offline indicator appears in top nav when disconnected

**8.4 Dark Mode Support**

**Detection:**

- Respect system preference: prefers-color-scheme: dark
- Optional future: Manual toggle in Profile settings

**Color scheme adjustments:**

- Background: Charcoal (#1F2121)
- Surface: Dark Surface (#262828)
- Text Primary: Light Gray (#F5F5F5)
- Text Secondary: Medium Gray (#A7A9A9)
- All other semantic colors remain same (Primary Teal, Success, etc.)

**Chat bubbles in dark mode:**

- User message: Darker teal background (rgba(#21808D, 0.15))
- AI message: Surface Dark background

**9\. Accessibility Standards**

**9.1 WCAG 2.1 AA Compliance**

**Color contrast:**

- Text Primary on Background: Minimum 7:1 contrast ratio
- Text Secondary on Background: Minimum 4.5:1 contrast ratio
- Button text on Primary Teal: Minimum 4.5:1 contrast ratio

**Interactive elements:**

- Minimum tap target size: 44px × 44px
- Focus indicators: 2px solid Primary Teal outline, 2px offset

**Keyboard navigation:**

- All interactive elements focusable via Tab key
- Focus order follows logical reading order (top to bottom, left to right)
- Escape key closes modals/overlays

**9.2 Screen Reader Support**

**Semantic HTML:**

- Use &lt;button&gt; for buttons (not &lt;div&gt; with click handlers)
- Use &lt;input&gt; with proper type attributes
- Use &lt;nav&gt; for bottom navigation bar
- Use &lt;main&gt; for primary content areas

**ARIA labels:**

- Chat input: aria-label="Log weight, food, or ask a question"
- Send button: aria-label="Send message"
- Bottom nav tabs: aria-label="Chat", aria-label="Insights", aria-label="Profile"
- Phase dots: aria-label="Phase 1, Completed", aria-label="Phase 2, Current"

**Live regions:**

- New chat messages: aria-live="polite" on message list container
- Loading states: aria-live="polite" on spinner/loading indicators
- Error messages: aria-live="assertive" on error text

**9.3 Focus Management**

**Modal open:**

- Focus moves to first interactive element inside modal (close button or first input field)
- Focus trapped inside modal (Tab cycles through modal elements only)

**Modal close:**

- Focus returns to element that triggered modal (e.g., "Read Wrap" button)

**Chat input:**

- Focus returns to chat input after sending message (ready for next log)

**9.4 Reduced Motion Support**

**Detection:**

- Respect system preference: prefers-reduced-motion: reduce

**Behavior when reduced motion enabled:**

- All animations reduced to instant transitions (duration: 0ms) or simple fades (100ms max)
- No pulsing, sliding, or bouncing animations
- Arc recalibration: Instant position update instead of smooth animation

**10\. Animation & Motion Design**

**10.1 Animation Principles**

**Guidelines:**

- Purposeful: Animations should guide attention or provide feedback, not decorate
- Fast: Keep durations short (150ms - 300ms typical, 800ms max for complex transitions)
- Smooth: Use ease-out or ease-in-out easing (avoid linear)
- Respectful: Disable for users with reduced motion preference

**10.2 Key Animations**

**Message appear (Chat):**

- Slide up from bottom + fade in
- Duration: 200ms
- Easing: ease-out

**Button press:**

- Scale down to 0.95 on active/pressed state
- Duration: 100ms
- Easing: ease-out

**Modal open:**

- Backdrop: Fade in (200ms)
- Modal: Scale up from 0.9 to 1.0 (200ms ease-out)

**Modal close:**

- Backdrop: Fade out (150ms)
- Modal: Scale down to 0.9 (150ms ease-in)

**Arc recalibration (Phase Journey):**

- Phase dots slide to new positions
- Duration: 800ms
- Easing: ease-out
- Projected dates: Fade out → update → fade in (300ms each)

**Micro-hook reveal:**

- Subtle slide up (8px distance) + fade in
- Duration: 250ms
- Easing: ease-out
- Delay: 150ms after log confirmation appears (staged reveal)

**Weekly Wrap card notification badge:**

- Pulse animation: Scale 1.0 → 1.2 → 1.0 (1.5s loop)
- Red dot grows/shrinks subtly to draw attention

**10.3 Loading States**

**Spinner (inline):**

- Rotate 360deg continuously
- Duration: 1s
- Easing: linear
- Color: Primary Teal

**Skeleton screen (message loading):**

- Pulsing background gradient (light gray → slightly darker gray → light gray)
- Duration: 1.5s loop
- Easing: ease-in-out

**Loading dots (onboarding Phase Journey generation):**

- 3 dots scale up/down sequentially
- Each dot: Scale 0.8 → 1.2 → 0.8 (600ms)
- Stagger delay: 200ms between dots

**11\. Error States & Edge Cases**

**11.1 Network Errors**

**Offline state (no connectivity):**

- Top nav shows offline indicator (cloud icon with slash, "Offline" text)
- Chat messages show status: "Will sync when online"
- User can continue logging (messages queue locally)
- No blocking modals or alerts

**Sync failure after reconnecting:**

- Message status chip shows "Failed - Retry" (red text)
- Tapping chip retries sync
- After 3 failed retries, show modal: "Some messages couldn't sync. Try again later or contact support."

**11.2 API Errors**

**5xx server errors:**

- Chat message shows error state: "Something broke on my side. Try again in a bit."
- Retry button below message (secondary button)
- Error logged to Sentry (backend)

**4xx client errors (validation failures):**

- Inline error message below input field (red text, 14px)
- Example: "Weight must be between 30-300 kg."
- Input field border turns red

**11.3 Empty States**

**Chat screen (first launch after onboarding):**

┌─────────────────────────────────────┐  
│ │  
│ 👋 │  
│ │  
│ Ready to start Phase 1? │  
│ │  
│ Log your first weight, meal, │  
│ or just ask me a question. │  
│ │  
└─────────────────────────────────────┘

**Insights screen (no patterns yet):**

┌─────────────────────────────────────┐  
│ Pattern Insights │  
│ ───────────────────────────── │  
│ │  
│ 🔍 │  
│ │  
│ Keep logging-patterns will │  
│ appear here after ~14 days. │  
│ │  
└─────────────────────────────────────┘

**Weekly Wrap (no wrap yet, Days 1-6):**

┌─────────────────────────────────────┐  
│ Weekly Wrap │  
│ ───────────────────────────── │  
│ │  
│ 📅 │  
│ │  
│ Your first wrap arrives Monday. │  
│ Keep logging to build your data. │  
│ │  
└─────────────────────────────────────┘

**11.4 Data Loading States**

**Chat screen loading (initial hydration):**

- Show skeleton screen (3-4 placeholder message bubbles with pulsing background)
- Duration: Typically <1s, but can be longer on slow connections
- After 3 seconds, show spinner with text: "Loading your data..."

**Insights screen loading:**

- Show skeleton cards (Weekly Wrap card + 2 pattern insight cards)
- Pulsing gray backgrounds

**Profile screen loading:**

- Show skeleton user info card + phase journey card

**11.5 Edge Case Scenarios**

**User logs weight spike after relapse:**

- AI acknowledges without alarm: "That's up from last week. Your tags show \[context\]. Let's keep tracking."
- Phase Journey arc moves backward (if hard relapse threshold crossed)
- No red colors, no warning icons

**User completes phase early:**

- Inline chat message appears immediately (on the log that crosses threshold):
  - "Phase 1 done. You set out to hit target by \[original date\]. You got there on \[actual date\], X days early. Phase 2 starts now. New focus: \[Phase 2 behavior\]. Your updated roadmap is in the Journey tab."
- Phase Journey arc updates (next phase begins)
- Following Weekly Wrap acknowledges transition in phase snapshot section

**User hasn't logged in 3 days:**

- Orchestrator does NOT send shame-based re-engagement ("You're behind!")
- Checks for illness/life-event context first (from chat history)
- If none, sends single low-friction re-entry prompt: "Ready when you are-your Phase 1 progress is saved."
- Then waits (no follow-up nagging)

**User self-reports relapse ("I had a bad two weeks"):**

- AI acknowledges without judgment: "That happens. Everything is saved exactly where you left off."
- Prompts for single retro context tag if useful: "Anything specific going on-travel, stress, illness?"
- Does not ask user to review phase, re-onboard, or explain themselves
- Resumes normal logging

**12\. Performance Considerations**

**12.1 Load Time Targets**

**First Contentful Paint (FCP):** <1.5s on 3G connection  
**Time to Interactive (TTI):** <3s on 3G connection  
**Largest Contentful Paint (LCP):** <2.5s on 3G connection

**12.2 Optimization Strategies**

**Code splitting:**

- Lazy-load Weekly Wrap Detail screen (only when user taps "Read Wrap")
- Lazy-load Phase Journey Detail screen (only when user taps "View Full Journey")
- Onboarding flow loaded separately (only for first-time users)

**Image optimization:**

- Food photo uploads: Compress to max 800px width, WebP format
- Icon assets: SVG where possible (scalable, small file size)
- App icons (PWA): PNG, 192px and 512px sizes

**IndexedDB usage:**

- Store queued logs locally (offline sync)
- Store last 50 chat messages for instant load
- Clear old data after 30 days (user can manually export full history)

**Service worker caching:**

- Cache app shell (HTML, CSS, JS, fonts) for offline access
- Cache static assets (icons, images) with versioned filenames
- Network-first strategy for API calls (fallback to cached data if offline)

**12.3 Render Performance**

**Chat scroll performance:**

- Virtualize message list if >100 messages (render only visible messages + buffer)
- Use transform: translateY() for smooth scroll animations (GPU-accelerated)

**Arc animation performance:**

- Use CSS transforms (translate, scale) instead of position changes
- Trigger animations with requestAnimationFrame()
- Limit arc complexity to max 8 phase dots (typical user journey: 4-5 phases)

**Avoid layout thrashing:**

- Batch DOM reads and writes
- Use will-change CSS property for elements that will animate (phase dots, modals)

**References**

\[1\] Health App Product Experience Spec v4.1. (2026). Harshit Shishodia.

\[2\] Technical Solutions Document v4.1. (2026). AI-Powered Health Pattern Intelligence App.

\[3\] Data Model Schema Document v3.0. (2026). PostgreSQL/TimescaleDB schema for MVP.

\[4\] AI Agents & Prompt Engineering v2.0. (2026). LangGraph-based agent architecture.

\[5\] API Contracts MVP v1.2. (2026). REST + WebSocket API specification.

\[6\] WCAG 2.1 Guidelines. (2018). W3C Web Accessibility Initiative. <https://www.w3.org/WAI/WCAG21/quickref/>

\[7\] PWA Best Practices. (2024). Google Developers. <https://web.dev/progressive-web-apps/>

\[8\] Material Design Motion Guidelines. (2024). Google Design. <https://material.io/design/motion/>

**CANVAS_OLD_STR**