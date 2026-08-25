# Chronos AI Scheduler — AI Architecture & Specification

**Project:** Chronos AI Scheduler
**Version:** 1.0
**Status:** Planning
**Document:** AI Architecture and Specification

---

# 1. Overview

Artificial intelligence is a core component of Chronos AI Scheduler.

The purpose of AI is to reduce the user's cognitive workload by helping with:

* Natural-language task creation
* Task prioritization
* Schedule generation
* Schedule optimization
* Automatic rescheduling
* Productivity analysis
* Daily summaries
* Personalized recommendations

AI should enhance the deterministic scheduling system rather than replace it.

---

# 2. AI Design Principle

The primary principle is:

> **AI recommends. The application validates and decides.**

The AI must never have unrestricted access to the database.

The correct flow is:

```text
User
 ↓
Backend
 ↓
AI Service
 ↓
AI Provider
 ↓
Structured Response
 ↓
Validation
 ↓
Scheduling Engine
 ↓
Database
```

---

# 3. AI Responsibilities

The AI layer is responsible for high-level reasoning and interpretation.

## AI should handle:

* Natural language understanding
* Task interpretation
* Task classification
* Priority recommendations
* Schedule suggestions
* Rescheduling recommendations
* Productivity pattern interpretation
* Personalized recommendations
* Daily and weekly summaries

## AI should not directly handle:

* Authentication
* Authorization
* Database writes
* Password management
* Ownership validation
* Financial transactions
* Security decisions
* Final schedule validation

---

# 4. AI Provider Abstraction

The application should not be tightly coupled to a single AI provider.

Architecture:

```text
AI Service
    ↓
AI Provider Interface
    ↓
 ┌─────────────┬─────────────┬─────────────┐
 │ Provider A  │ Provider B  │ Provider C  │
 └─────────────┴─────────────┴─────────────┘
```

The backend should define an internal interface such as:

```text
AIProvider
```

Possible implementations can be added later.

This allows the project to switch providers without rewriting the scheduling system.

---

# 5. AI Capabilities

## 5.1 Natural Language Task Parser

User:

> I need to study Java for two hours tomorrow evening.

AI should return structured data such as:

```json
{
  "title": "Study Java",
  "durationMinutes": 120,
  "date": "tomorrow",
  "preferredPeriod": "EVENING",
  "category": "STUDY"
}
```

The backend converts relative dates into actual dates and validates the result.

---

# 6. Task Prioritization

The AI can recommend task priority using:

```text
Deadline
Importance
Difficulty
Estimated duration
Goal relevance
Dependencies
User preferences
Current workload
```

Example:

```json
{
  "taskId": 42,
  "recommendedPriority": "HIGH",
  "reason": "The deadline is tomorrow and the task requires significant preparation."
}
```

The backend may combine AI recommendations with deterministic priority rules.

---

# 7. Schedule Generation

The AI receives contextual information such as:

```text
User availability
Existing commitments
Pending tasks
Deadlines
Task durations
Priorities
Goals
Scheduling preferences
Historical productivity
```

It returns a structured schedule proposal.

Example:

```json
{
  "schedule": [
    {
      "taskId": 12,
      "start": "18:00",
      "end": "19:30",
      "reason": "High priority task with a near deadline"
    },
    {
      "taskId": 18,
      "start": "20:00",
      "end": "21:00",
      "reason": "Lower priority task placed after the primary task"
    }
  ]
}
```

---

# 8. AI Schedule Constraints

The AI should be instructed to respect:

* User availability
* Existing commitments
* Task deadlines
* Task duration
* Required breaks
* User preferences
* Task dependencies

However, AI output must still be independently validated by the backend.

---

# 9. Schedule Validation

After receiving AI output:

```text
AI Response
 ↓
Schema Validation
 ↓
Task Validation
 ↓
Time Validation
 ↓
Conflict Detection
 ↓
Deadline Validation
 ↓
Dependency Validation
 ↓
Scheduling Engine
```

Invalid schedules must never be directly saved.

---

# 10. Automatic Rescheduling

When a scheduled task is missed:

```text
Missed Task
 ↓
Collect remaining tasks
 ↓
Calculate remaining available time
 ↓
Recalculate priorities
 ↓
AI recommendation
 ↓
Backend validation
 ↓
Scheduling engine
 ↓
Updated schedule
```

The AI should explain why tasks were moved.

Example:

> "The project was moved before revision because its deadline is earlier."

---

# 11. Productivity Coach

The AI can analyze aggregated productivity information.

Inputs may include:

```text
Completion rate
Focus time
Missed tasks
Estimated vs actual durations
Most productive periods
Schedule adherence
Goal progress
Habit consistency
```

Example output:

> You consistently complete difficult tasks more successfully during evening focus sessions. Consider scheduling your highest-priority study tasks during this period.

The system should avoid presenting correlations as absolute facts.

---

# 12. Task Duration Prediction

The system can eventually learn from historical task data.

Example:

```text
Estimated:
60 minutes

Actual historical average:
82 minutes
```

The system can recommend:

```text
Future estimate:
80–90 minutes
```

This should be based on sufficient historical data.

New users should use the manually supplied estimate until enough historical information exists.

---

# 13. AI Daily Summary

The AI receives the day's aggregated information.

Example:

```text
Tasks completed: 8
Tasks missed: 2
Focus time: 5h 40m
Planned time: 7h
Completion rate: 80%
```

AI generates a concise summary.

Example:

> You completed most of your high-priority tasks today. Your largest unfinished task was the Java project, which should be moved to tomorrow's first available high-focus period.

---

# 14. AI Weekly Analysis

The weekly AI analysis can identify:

* Productivity patterns
* Scheduling problems
* Frequent task delays
* Unrealistic estimates
* Strong productivity periods
* Goal progress
* Habit consistency

Example:

> Your average completion rate increased this week, but programming tasks are still being underestimated. Increasing development time blocks by 20–30 minutes may produce more realistic schedules.

---

# 15. Prompt Architecture

Prompts should not be hard-coded inside controllers.

Recommended structure:

```text
ai/
├── prompt/
│   ├── task-parser.md
│   ├── prioritizer.md
│   ├── schedule-generator.md
│   ├── rescheduler.md
│   ├── daily-summary.md
│   └── productivity-coach.md
```

The AI service loads the appropriate prompt template and supplies structured context.

---

# 16. Prompt Design

Prompts should clearly define:

1. Role
2. Objective
3. Context
4. Constraints
5. Input schema
6. Output schema
7. Rules
8. Examples where useful

Example conceptual structure:

```text
SYSTEM:
You are a personal scheduling assistant.

OBJECTIVE:
Generate a realistic schedule.

CONSTRAINTS:
Never overlap events.
Never schedule outside availability.
Respect hard deadlines.

INPUT:
<User context>

OUTPUT:
Return only the required structured response.
```

---

# 17. Structured AI Output

AI responses should use structured output whenever supported by the selected provider.

Example schema:

```json
{
  "schedule": [
    {
      "taskId": 0,
      "startTime": "HH:mm",
      "endTime": "HH:mm",
      "reason": "string"
    }
  ]
}
```

The parser should reject malformed responses.

---

# 18. AI Context Management

The AI should receive only the information necessary for the requested operation.

For example, generating today's schedule may require:

```text
User availability
Today's tasks
Today's commitments
Relevant goals
Scheduling preferences
Relevant historical productivity
```

It does not need the user's entire database.

This reduces:

* Token usage
* Latency
* Cost
* Privacy exposure
* Prompt complexity

---

# 19. Conversation Memory

AI conversations may be stored for conversational features.

However, permanent memory should be separated from conversation history.

Example:

```text
Conversation History
        ≠
User Preferences
        ≠
Productivity History
```

Important user preferences should be stored as structured application data rather than relying on the AI remembering them from old conversations.

---

# 20. AI Failure Handling

Possible failures:

* Provider unavailable
* Timeout
* Invalid response
* Rate limit
* Malformed JSON
* Token limit
* API authentication failure

The system should handle these gracefully.

Example:

```text
AI unavailable
 ↓
Log failure
 ↓
Use deterministic scheduler
 ↓
Return schedule
```

The user should still be able to use core scheduling functionality.

---

# 21. AI Retry Strategy

Retries should be limited.

Recommended behavior:

```text
Request
 ↓
Provider failure
 ↓
Retry once when appropriate
 ↓
If failure persists
 ↓
Fallback
```

The application should not repeatedly retry failed requests.

---

# 22. AI Cost Management

AI requests should be minimized.

Strategies:

* Send only relevant context
* Cache suitable results
* Avoid unnecessary repeated generation
* Use smaller models for simple tasks
* Use more capable models for complex planning
* Track AI usage

Potential future table:

```text
ai_usage
```

---

# 23. AI Security

Never send:

* Passwords
* Password hashes
* JWT secrets
* API keys
* Database credentials
* Unnecessary personal information

to the AI provider.

User data sent to an external AI provider should be minimized.

---

# 24. AI Privacy

The application should clearly define what data may be sent to an external AI provider.

AI requests should contain only the minimum required information.

For example, schedule generation may need:

```text
Task title
Task duration
Deadline
Priority
Availability
```

It does not require unrelated account information.

---

# 25. AI Logging

The application should log:

* AI request type
* Provider
* Model
* Latency
* Success/failure
* Error type
* Token usage where available

The application should avoid logging sensitive prompt content unnecessarily.

---

# 26. AI Versioning

AI prompts and schemas should be versioned.

Example:

```text
schedule-generator-v1
schedule-generator-v2
```

This makes it possible to compare and improve AI behavior over time.

---

# 27. AI Evaluation

AI features should be tested against predefined scenarios.

Example:

```text
Scenario:
Three tasks.
Two hours available.
One urgent deadline.

Expected:
Urgent task must receive priority.
No overlapping schedule.
Total duration must fit available time.
```

Evaluation should check:

* Correctness
* Constraint compliance
* Consistency
* Response structure
* Latency
* Cost

---

# 28. AI Golden Rules

The AI implementation must follow these rules:

1. Never bypass backend validation.
2. Never directly access the database.
3. Never receive secrets.
4. Never assume unavailable information.
5. Never create impossible schedules intentionally.
6. Always return structured data where required.
7. Explain recommendations when useful.
8. Fall back gracefully when unavailable.
9. Minimize transmitted user data.
10. Treat AI output as a recommendation, not authoritative truth.
