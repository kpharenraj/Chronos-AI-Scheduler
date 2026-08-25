# Chronos AI Scheduler — System Architecture

**Project Name:** Chronos AI Scheduler
**Version:** 1.0
**Status:** Phase 0 Complete - Ready for Phase 1

---

# 1. Architecture Overview

Chronos AI Scheduler will use a modular full-stack architecture.

The primary architecture consists of:

```text
                    ┌─────────────────────┐
                    │       User          │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   React Frontend    │
                    │   TypeScript        │
                    └──────────┬──────────┘
                               │ HTTPS / REST
                               ▼
                    ┌─────────────────────┐
                    │   Spring Boot API   │
                    │       Java 21       │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
       ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
       │ Task Module │  │ Scheduler   │  │ Analytics   │
       │             │  │ Engine      │  │ Module      │
       └─────────────┘  └──────┬──────┘  └─────────────┘
                               │
                               ▼
                        ┌─────────────┐
                        │  AI Service │
                        └──────┬──────┘
                               │
                               ▼
                        ┌─────────────┐
                        │ AI Provider │
                        └─────────────┘

                               │
                               ▼
                        ┌─────────────┐
                        │    MySQL    │
                        └─────────────┘

                        ┌─────────────┐
                        │    Redis    │
                        └─────────────┘
```

---

# 2. Architectural Style

The backend will initially use a modular monolith architecture.

This means:

* One Spring Boot application
* Clearly separated business modules
* Shared database
* Internal module boundaries

We will not start with microservices.

Microservices would add unnecessary complexity for the initial project.

---

# 3. Frontend Architecture

The frontend will use:

```text
React
TypeScript
Vite
Tailwind CSS
shadcn/ui
React Router
TanStack Query
Zustand
Recharts
```

Frontend responsibilities:

* Render UI
* Handle user interaction
* Manage client-side state
* Communicate with backend APIs
* Display schedules
* Display analytics
* Run focus timers
* Provide forms and validation
* Handle authentication state

The frontend must not contain critical business rules.

---

# 4. Backend Architecture

The backend will use:

```text
Java 21
Spring Boot
Spring Web
Spring Security
Spring Data JPA
Hibernate
Bean Validation
Flyway
Spring AI
```

The backend will contain separate modules.

```text
auth
user
task
scheduler
ai
focus
analytics
goals
habits
notifications
common
```

---

# 5. Backend Module Responsibilities

## 5.1 Authentication Module

Responsible for:

* Registration
* Login
* Logout
* Password management
* JWT authentication
* Security context

---

## 5.2 User Module

Responsible for:

* User profile
* Preferences
* Timezone
* Availability
* Scheduling preferences

---

## 5.3 Task Module

Responsible for:

* Task creation
* Task modification
* Task deletion
* Task status
* Task priority
* Deadlines
* Estimated duration
* Categories

---

## 5.4 Scheduler Module

Responsible for:

* Available time calculation
* Conflict detection
* Schedule generation
* Schedule validation
* Rescheduling
* Time-block allocation

This module is the core deterministic scheduling engine.

---

## 5.5 AI Module

Responsible for:

* AI provider communication
* Prompt management
* Structured AI responses
* AI task parsing
* AI prioritization
* AI schedule suggestions
* AI recommendations
* AI summaries

The AI module must not directly persist arbitrary AI output.

---

# 6. AI Architecture

The AI layer should be provider-independent.

Instead of coupling the entire system to one provider:

```text
                    AI Service
                        │
                        ▼
                 AI Provider Interface
                  /        |        \
                 /         |         \
                ▼          ▼          ▼
             Provider A  Provider B  Provider C
```

This allows the AI provider to be replaced later.

---

# 7. AI Request Flow

Example: Generate today's schedule.

```text
User
 ↓
POST /api/schedules/generate
 ↓
Schedule Controller
 ↓
Schedule Service
 ↓
Retrieve:
 - User preferences
 - Tasks
 - Goals
 - Existing events
 - Availability
 ↓
AI Service
 ↓
AI Provider
 ↓
Structured JSON response
 ↓
AI Response Parser
 ↓
Validation
 ↓
Scheduling Engine
 ↓
Conflict Detection
 ↓
Validated Schedule
 ↓
Database
 ↓
API Response
 ↓
React UI
```

---

# 8. AI Must Not Be the Source of Truth

The AI is a recommendation engine.

The application database and backend rules are the source of truth.

For example, if AI returns:

```text
18:00 - 20:00 Task A
19:00 - 21:00 Task B
```

the backend must detect that the blocks overlap.

The system should reject or repair the invalid schedule.

---

# 9. Scheduling Engine

The scheduling engine should be deterministic whenever possible.

It should understand:

```text
Available time
+
Existing commitments
+
Task duration
+
Task priority
+
Deadline
+
Dependencies
+
Break requirements
+
User preferences
```

The engine should ensure:

* No overlapping tasks
* No tasks outside availability
* No invalid durations
* No tasks scheduled after their hard deadline
* Reasonable breaks
* Correct user ownership

---

# 10. Scheduling Strategy

The scheduling process should roughly follow:

```text
1. Load tasks
2. Remove completed/cancelled tasks
3. Load user availability
4. Load existing commitments
5. Calculate free time
6. Rank tasks
7. Request AI recommendations if required
8. Validate AI recommendations
9. Resolve conflicts
10. Allocate time blocks
11. Add breaks
12. Save schedule
```

---

# 11. Rescheduling Architecture

When a task is missed:

```text
Missed Task
     ↓
Rescheduling Request
     ↓
Retrieve Remaining Tasks
     ↓
Retrieve Remaining Available Time
     ↓
Recalculate Priorities
     ↓
AI Recommendation
     ↓
Validation
     ↓
Scheduling Engine
     ↓
Updated Schedule
```

The system should not automatically move all tasks without considering priority and deadlines.

---

# 12. Database Layer

The persistence layer will use:

```text
MySQL
+
Spring Data JPA
+
Hibernate
+
Flyway
```

Responsibilities:

* Persist users
* Persist tasks
* Persist schedules
* Persist focus sessions
* Persist goals
* Persist habits
* Persist analytics
* Persist AI recommendations

---

# 13. Redis

Redis will be introduced for:

* Caching
* Rate limiting
* Temporary scheduling data
* AI response caching where appropriate
* Background-job support in future versions

Redis is not the primary database.

MySQL remains the source of truth.

---

# 14. API Architecture

The backend exposes REST APIs.

Base URL:

```text
/api
```

Major API groups:

```text
/api/auth
/api/users
/api/tasks
/api/schedules
/api/focus
/api/goals
/api/habits
/api/analytics
/api/ai
/api/notifications
```

---

# 15. API Design Principles

APIs should:

* Use REST conventions.
* Use JSON.
* Use HTTP status codes correctly.
* Validate request data.
* Return consistent response structures.
* Never expose internal exceptions directly.
* Require authentication where appropriate.
* Verify resource ownership.

---

# 16. Authentication Flow

```text
User
 ↓
Login
 ↓
POST /api/auth/login
 ↓
Spring Security
 ↓
Validate credentials
 ↓
Generate JWT
 ↓
Frontend stores authentication state
 ↓
Frontend sends token with API requests
```

Protected request:

```text
Authorization: Bearer <token>
```

---

# 17. Error Handling

The backend should have centralized exception handling.

Example structure:

```text
common/
└── exception/
    ├── GlobalExceptionHandler.java
    ├── ResourceNotFoundException.java
    ├── UnauthorizedException.java
    ├── ValidationException.java
    └── SchedulingConflictException.java
```

API errors should use a consistent structure.

Example:

```json
{
  "success": false,
  "message": "Task deadline cannot be earlier than its scheduled start time",
  "errorCode": "INVALID_SCHEDULE",
  "timestamp": "2026-08-25T19:00:00"
}
```

---

# 18. Security Architecture

Security layers:

```text
Frontend
   ↓
HTTPS
   ↓
Spring Security
   ↓
JWT Authentication
   ↓
Authorization
   ↓
Controller
   ↓
Service
   ↓
Repository
```

The application should never trust user-supplied IDs without verifying ownership.

---

# 19. Environment Configuration

Secrets must never be committed to Git.

Configuration should use environment variables.

Example:

```text
DATABASE_URL
DATABASE_USERNAME
DATABASE_PASSWORD

JWT_SECRET

AI_API_KEY

REDIS_HOST
REDIS_PORT
```

A `.env.example` file should contain placeholders but never real secrets.

---

# 20. Deployment Architecture

Initial production architecture:

```text
                 Internet
                    │
                    ▼
             ┌─────────────┐
             │  Frontend   │
             │   Hosting   │
             └──────┬──────┘
                    │ HTTPS
                    ▼
             ┌─────────────┐
             │ Spring Boot │
             │   Backend   │
             └──────┬──────┘
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
     ┌─────────┐         ┌─────────┐
     │  MySQL  │         │  Redis  │
     └─────────┘         └─────────┘
                    │
                    ▼
               AI Provider
```

---

# 21. Docker Architecture

Development environment:

```text
Docker Compose
│
├── frontend
├── backend
├── mysql
└── redis
```

The application should be runnable using a single Docker Compose configuration during development.

---

# 22. Testing Architecture

Testing will exist at multiple levels.

## Backend

```text
JUnit
Mockito
Spring Boot Test
```

Tests:

* Unit tests
* Service tests
* Repository tests
* Controller tests
* Scheduling engine tests

## Frontend

Use:

```text
Vitest
React Testing Library
```

## End-to-End

Use:

```text
Playwright
```

Critical workflows should be tested end-to-end:

```text
Register
 → Login
 → Create Task
 → Generate Schedule
 → Start Focus Session
 → Complete Task
```

---

# 23. Logging and Monitoring

Backend should use structured logging.

Important events:

* Authentication attempts
* API errors
* Scheduling failures
* AI failures
* Database failures
* Background job failures

Sensitive information must not be logged.

---

# 24. Background Jobs

Future background jobs may handle:

* Notifications
* Daily summaries
* Reminder generation
* Productivity calculations
* Habit tracking
* Weekly AI reports

The initial implementation can use Spring's scheduling facilities.

A queue-based architecture can be introduced later if necessary.

---

# 25. Architecture Principles

The project follows these principles:

### Principle 1 — Separation of Concerns

Each module should have a clear responsibility.

### Principle 2 — Backend Is the Source of Truth

The frontend and AI cannot bypass backend validation.

### Principle 3 — AI Is Assistive

AI provides intelligence and recommendations.

It does not directly control persistent application state.

### Principle 4 — Modular Monolith First

Start simple and modular.

Do not introduce microservices without a real need.

### Principle 5 — Security by Default

Authentication, authorization, validation, and secret management are part of the architecture.

### Principle 6 — Replaceable AI Provider

AI integrations should be abstracted so providers can be changed.

### Principle 7 — Graceful AI Failure

The application must remain usable when the AI service is unavailable.

---

# 26. High-Level Data Flow

```text
User Input
    ↓
React
    ↓
REST API
    ↓
Controller
    ↓
Service
    ↓
 ┌───────────────┐
 │ Business Logic│
 └───────┬───────┘
         │
    ┌────┴─────┐
    ▼          ▼
 Database      AI
    │          │
    └────┬─────┘
         ▼
   Validation
         ↓
    Final Result
         ↓
       React
```

---

# 27. Final Architecture

Chronos AI Scheduler will initially be implemented as a secure, modular monolith consisting of:

```text
React + TypeScript
        ↓
Spring Boot + Java 21
        ↓
Business Modules
        ↓
Scheduling Engine + AI Service
        ↓
MySQL + Redis
```

This architecture provides a good balance between:

* Simplicity
* Maintainability
* Security
* AI integration
* Scalability
* Learning value
* Future extensibility
