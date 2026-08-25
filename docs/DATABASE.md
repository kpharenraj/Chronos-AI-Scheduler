# Chronos AI Scheduler — Database Design

**Project Name:** Chronos AI Scheduler
**Database:** MySQL 8.x
**ORM:** Hibernate / Spring Data JPA
**Migration Tool:** Flyway
**Version:** 1.0

---

# 1. Database Overview

MySQL will be the primary persistent data store for Chronos AI Scheduler.

The database will store:

* User accounts
* User preferences
* Tasks
* Categories
* Goals
* Habits
* Schedules
* Focus sessions
* Notifications
* Productivity statistics
* AI conversations
* AI recommendations

Redis may be used for temporary and cached data, but MySQL remains the source of truth.

---

# 2. Database Design Principles

The database should follow:

* Relational design
* Referential integrity
* Appropriate normalization
* Foreign-key relationships
* Indexing for frequently queried fields
* Audit timestamps
* Soft deletion where appropriate
* Secure storage of sensitive information

---

# 3. Entity Relationship Overview

```text
users
 │
 ├────────────── user_preferences
 │
 ├────────────── tasks
 │                  │
 │                  ├──────── categories
 │                  │
 │                  └──────── goals
 │
 ├────────────── schedules
 │                  │
 │                  └──────── schedule_items
 │                              │
 │                              └──── tasks
 │
 ├────────────── focus_sessions
 │                  │
 │                  └──── tasks
 │
 ├────────────── habits
 │
 ├────────────── goals
 │
 ├────────────── notifications
 │
 ├────────────── productivity_stats
 │
 ├────────────── ai_conversations
 │                  │
 │                  └──── ai_messages
 │
 └────────────── ai_recommendations
```

---

# 4. users

Stores account information.

## Fields

| Column        | Type         | Constraints        | Description        |
| ------------- | ------------ | ------------------ | ------------------ |
| id            | BIGINT       | PK, AUTO_INCREMENT | User ID            |
| name          | VARCHAR(100) | NOT NULL           | User name          |
| email         | VARCHAR(255) | NOT NULL, UNIQUE   | Login email        |
| password_hash | VARCHAR(255) | NOT NULL           | Hashed password    |
| timezone      | VARCHAR(100) | NOT NULL           | User timezone      |
| created_at    | TIMESTAMP    | NOT NULL           | Creation timestamp |
| updated_at    | TIMESTAMP    | NOT NULL           | Last update        |

---

# 5. user_preferences

Stores scheduling preferences.

## Fields

| Column                | Type      | Constraints        | Description          |
| --------------------- | --------- | ------------------ | -------------------- |
| id                    | BIGINT    | PK, AUTO_INCREMENT | Preference ID        |
| user_id               | BIGINT    | FK, UNIQUE         | User                 |
| wake_time             | TIME      | NOT NULL           | Preferred wake time  |
| sleep_time            | TIME      | NOT NULL           | Preferred sleep time |
| default_break_minutes | INT       | NOT NULL           | Default break        |
| focus_session_minutes | INT       | NOT NULL           | Focus duration       |
| work_start_time       | TIME      | NULL               | Work/college start   |
| work_end_time         | TIME      | NULL               | Work/college end     |
| created_at            | TIMESTAMP | NOT NULL           | Creation time        |
| updated_at            | TIMESTAMP | NOT NULL           | Update time          |

Relationship:

```text
users 1 ───── 1 user_preferences
```

---

# 6. categories

Stores task categories.

Examples:

```text
Study
Work
Exercise
Personal
Project
Health
```

## Fields

| Column      | Type         | Constraints        |
| ----------- | ------------ | ------------------ |
| id          | BIGINT       | PK, AUTO_INCREMENT |
| user_id     | BIGINT       | FK, NULL           |
| name        | VARCHAR(100) | NOT NULL           |
| description | VARCHAR(255) | NULL               |
| created_at  | TIMESTAMP    | NOT NULL           |

A nullable `user_id` can allow system-defined categories while non-null values represent user-specific categories.

---

# 7. tasks

Stores user tasks.

## Fields

| Column                     | Type         | Constraints        |
| -------------------------- | ------------ | ------------------ |
| id                         | BIGINT       | PK, AUTO_INCREMENT |
| user_id                    | BIGINT       | FK, NOT NULL       |
| category_id                | BIGINT       | FK, NULL           |
| goal_id                    | BIGINT       | FK, NULL           |
| title                      | VARCHAR(255) | NOT NULL           |
| description                | TEXT         | NULL               |
| priority                   | ENUM         | NOT NULL           |
| status                     | ENUM         | NOT NULL           |
| estimated_duration_minutes | INT          | NOT NULL           |
| actual_duration_minutes    | INT          | NULL               |
| difficulty                 | INT          | NULL               |
| deadline                   | DATETIME     | NULL               |
| is_recurring               | BOOLEAN      | NOT NULL           |
| recurrence_rule            | VARCHAR(255) | NULL               |
| completed_at               | DATETIME     | NULL               |
| created_at                 | TIMESTAMP    | NOT NULL           |
| updated_at                 | TIMESTAMP    | NOT NULL           |

Priority:

```text
LOW
MEDIUM
HIGH
URGENT
```

Status:

```text
PENDING
SCHEDULED
IN_PROGRESS
COMPLETED
CANCELLED
SKIPPED
```

Relationships:

```text
users 1 ───── N tasks

categories 1 ───── N tasks

goals 1 ───── N tasks
```

---

# 8. goals

Stores long-term user goals.

## Fields

| Column              | Type         | Constraints        |
| ------------------- | ------------ | ------------------ |
| id                  | BIGINT       | PK, AUTO_INCREMENT |
| user_id             | BIGINT       | FK, NOT NULL       |
| title               | VARCHAR(255) | NOT NULL           |
| description         | TEXT         | NULL               |
| priority            | ENUM         | NOT NULL           |
| target_date         | DATE         | NULL               |
| progress_percentage | DECIMAL(5,2) | NOT NULL           |
| status              | ENUM         | NOT NULL           |
| created_at          | TIMESTAMP    | NOT NULL           |
| updated_at          | TIMESTAMP    | NOT NULL           |

Goal statuses:

```text
ACTIVE
COMPLETED
PAUSED
CANCELLED
```

---

# 9. habits

Stores recurring habits.

## Fields

| Column         | Type         | Constraints        |
| -------------- | ------------ | ------------------ |
| id             | BIGINT       | PK, AUTO_INCREMENT |
| user_id        | BIGINT       | FK, NOT NULL       |
| title          | VARCHAR(255) | NOT NULL           |
| description    | TEXT         | NULL               |
| frequency      | VARCHAR(100) | NOT NULL           |
| target_minutes | INT          | NULL               |
| active         | BOOLEAN      | NOT NULL           |
| created_at     | TIMESTAMP    | NOT NULL           |
| updated_at     | TIMESTAMP    | NOT NULL           |

---

# 10. habit_completions

Tracks individual habit completion events.

## Fields

| Column           | Type     | Constraints        |
| ---------------- | -------- | ------------------ |
| id               | BIGINT   | PK, AUTO_INCREMENT |
| habit_id         | BIGINT   | FK, NOT NULL       |
| completed_at     | DATETIME | NOT NULL           |
| duration_minutes | INT      | NULL               |
| notes            | TEXT     | NULL               |

Relationship:

```text
habits 1 ───── N habit_completions
```

---

# 11. schedules

Represents a generated schedule for a particular day.

## Fields

| Column        | Type      | Constraints        |
| ------------- | --------- | ------------------ |
| id            | BIGINT    | PK, AUTO_INCREMENT |
| user_id       | BIGINT    | FK, NOT NULL       |
| schedule_date | DATE      | NOT NULL           |
| status        | ENUM      | NOT NULL           |
| generated_by  | ENUM      | NOT NULL           |
| version       | INT       | NOT NULL           |
| created_at    | TIMESTAMP | NOT NULL           |
| updated_at    | TIMESTAMP | NOT NULL           |

`generated_by`:

```text
AI
RULE_ENGINE
HYBRID
MANUAL
```

The `version` field allows the system to keep track of rescheduled versions.

---

# 12. schedule_items

Stores individual time blocks.

## Fields

| Column      | Type         | Constraints        |
| ----------- | ------------ | ------------------ |
| id          | BIGINT       | PK, AUTO_INCREMENT |
| schedule_id | BIGINT       | FK, NOT NULL       |
| task_id     | BIGINT       | FK, NULL           |
| title       | VARCHAR(255) | NOT NULL           |
| start_time  | DATETIME     | NOT NULL           |
| end_time    | DATETIME     | NOT NULL           |
| item_type   | ENUM         | NOT NULL           |
| status      | ENUM         | NOT NULL           |
| ai_reason   | TEXT         | NULL               |
| created_at  | TIMESTAMP    | NOT NULL           |
| updated_at  | TIMESTAMP    | NOT NULL           |

Item types:

```text
TASK
BREAK
MEAL
EXERCISE
PERSONAL
COMMITMENT
OTHER
```

---

# 13. focus_sessions

Stores actual focus sessions.

## Fields

| Column           | Type      | Constraints        |
| ---------------- | --------- | ------------------ |
| id               | BIGINT    | PK, AUTO_INCREMENT |
| user_id          | BIGINT    | FK, NOT NULL       |
| task_id          | BIGINT    | FK, NULL           |
| schedule_item_id | BIGINT    | FK, NULL           |
| started_at       | DATETIME  | NOT NULL           |
| ended_at         | DATETIME  | NULL               |
| duration_seconds | INT       | NULL               |
| status           | ENUM      | NOT NULL           |
| created_at       | TIMESTAMP | NOT NULL           |

Statuses:

```text
ACTIVE
PAUSED
COMPLETED
CANCELLED
```

---

# 14. notifications

Stores application notifications.

## Fields

| Column            | Type         | Constraints        |
| ----------------- | ------------ | ------------------ |
| id                | BIGINT       | PK, AUTO_INCREMENT |
| user_id           | BIGINT       | FK, NOT NULL       |
| title             | VARCHAR(255) | NOT NULL           |
| message           | TEXT         | NOT NULL           |
| notification_type | VARCHAR(50)  | NOT NULL           |
| scheduled_at      | DATETIME     | NULL               |
| read_at           | DATETIME     | NULL               |
| created_at        | TIMESTAMP    | NOT NULL           |

---

# 15. productivity_stats

Stores aggregated productivity statistics.

## Fields

| Column             | Type         | Constraints        |
| ------------------ | ------------ | ------------------ |
| id                 | BIGINT       | PK, AUTO_INCREMENT |
| user_id            | BIGINT       | FK, NOT NULL       |
| stat_date          | DATE         | NOT NULL           |
| tasks_completed    | INT          | NOT NULL           |
| tasks_missed       | INT          | NOT NULL           |
| planned_minutes    | INT          | NOT NULL           |
| actual_minutes     | INT          | NOT NULL           |
| focus_minutes      | INT          | NOT NULL           |
| completion_rate    | DECIMAL(5,2) | NOT NULL           |
| productivity_score | DECIMAL(5,2) | NULL               |
| created_at         | TIMESTAMP    | NOT NULL           |

Recommended unique constraint:

```text
(user_id, stat_date)
```

This ensures one daily statistics record per user.

---

# 16. ai_conversations

Stores AI conversation sessions.

## Fields

| Column       | Type         | Constraints        |
| ------------ | ------------ | ------------------ |
| id           | BIGINT       | PK, AUTO_INCREMENT |
| user_id      | BIGINT       | FK, NOT NULL       |
| title        | VARCHAR(255) | NULL               |
| context_type | VARCHAR(50)  | NULL               |
| created_at   | TIMESTAMP    | NOT NULL           |
| updated_at   | TIMESTAMP    | NOT NULL           |

---

# 17. ai_messages

Stores messages associated with AI conversations.

## Fields

| Column          | Type      | Constraints        |
| --------------- | --------- | ------------------ |
| id              | BIGINT    | PK, AUTO_INCREMENT |
| conversation_id | BIGINT    | FK, NOT NULL       |
| role            | ENUM      | NOT NULL           |
| content         | LONGTEXT  | NOT NULL           |
| token_count     | INT       | NULL               |
| created_at      | TIMESTAMP | NOT NULL           |

Roles:

```text
USER
ASSISTANT
SYSTEM
```

Sensitive information should not be stored unnecessarily.

---

# 18. ai_recommendations

Stores AI-generated recommendations.

## Fields

| Column              | Type         | Constraints        |
| ------------------- | ------------ | ------------------ |
| id                  | BIGINT       | PK, AUTO_INCREMENT |
| user_id             | BIGINT       | FK, NOT NULL       |
| recommendation_type | VARCHAR(100) | NOT NULL           |
| title               | VARCHAR(255) | NOT NULL           |
| content             | TEXT         | NOT NULL           |
| confidence_score    | DECIMAL(5,2) | NULL               |
| accepted            | BOOLEAN      | NULL               |
| created_at          | TIMESTAMP    | NOT NULL           |

Recommendations may include:

```text
SCHEDULE
PRODUCTIVITY
TASK_ESTIMATION
HABIT
GOAL
DAILY_SUMMARY
```

---

# 19. User Availability

The scheduler requires information about when the user is available.

This can be represented using a dedicated table.

## user_availability

| Column            | Type      | Constraints        |
| ----------------- | --------- | ------------------ |
| id                | BIGINT    | PK, AUTO_INCREMENT |
| user_id           | BIGINT    | FK, NOT NULL       |
| day_of_week       | TINYINT   | NOT NULL           |
| start_time        | TIME      | NOT NULL           |
| end_time          | TIME      | NOT NULL           |
| availability_type | ENUM      | NOT NULL           |
| created_at        | TIMESTAMP | NOT NULL           |

Availability types:

```text
AVAILABLE
UNAVAILABLE
```

Example:

```text
Monday
17:00 - 22:00
AVAILABLE
```

---

# 20. Task Dependencies

Some tasks may depend on other tasks.

Example:

```text
Complete research
        ↓
Write report
        ↓
Submit report
```

A task dependency table can represent this.

## task_dependencies

| Column             | Type      | Constraints  |
| ------------------ | --------- | ------------ |
| task_id            | BIGINT    | FK, NOT NULL |
| depends_on_task_id | BIGINT    | FK, NOT NULL |
| created_at         | TIMESTAMP | NOT NULL     |

Primary key:

```text
(task_id, depends_on_task_id)
```

The scheduler must not schedule a dependent task before its required predecessor is completed.

---

# 21. Relationships

Major relationships:

```text
User
 │
 ├── 1:1 UserPreferences
 │
 ├── 1:N Tasks
 │       │
 │       ├── N:1 Category
 │       └── N:1 Goal
 │
 ├── 1:N Goals
 │
 ├── 1:N Habits
 │       │
 │       └── 1:N HabitCompletions
 │
 ├── 1:N Schedules
 │       │
 │       └── 1:N ScheduleItems
 │                    │
 │                    └── N:1 Task
 │
 ├── 1:N FocusSessions
 │
 ├── 1:N Notifications
 │
 ├── 1:N ProductivityStats
 │
 ├── 1:N AIConversations
 │       │
 │       └── 1:N AIMessages
 │
 └── 1:N AIRecommendations
```

---

# 22. Indexing Strategy

Indexes should be added to frequently queried columns.

Important indexes include:

```text
users.email

tasks.user_id
tasks.deadline
tasks.status
tasks.priority

schedules.user_id
schedules.schedule_date

schedule_items.schedule_id
schedule_items.start_time

focus_sessions.user_id
focus_sessions.started_at

notifications.user_id
notifications.read_at

productivity_stats.user_id
productivity_stats.stat_date

ai_conversations.user_id
ai_messages.conversation_id
```

Composite indexes should be added where query patterns justify them.

---

# 23. Foreign Key Strategy

Foreign keys should enforce relationships.

Example:

```text
tasks.user_id
    ↓
users.id
```

When deleting users, dependent records should be handled carefully.

For personal productivity data, the preferred strategy is usually controlled deletion rather than blindly cascading every table.

---

# 24. Timestamps

Major entities should include:

```text
created_at
updated_at
```

Event-based entities such as focus sessions should additionally record their actual event timestamps.

All timestamps should be handled consistently with the user's timezone and persisted using a well-defined server-side convention.

---

# 25. Soft Delete

For important user data, soft deletion may be preferred.

Instead of immediately deleting a record:

```text
deleted_at = current_timestamp
```

This allows recovery and auditing.

Soft deletion should be introduced selectively rather than applied blindly to every table.

---

# 26. Database Migration Strategy

Flyway will manage schema changes.

Example:

```text
db/migration/

V1__create_users.sql
V2__create_user_preferences.sql
V3__create_categories.sql
V4__create_goals.sql
V5__create_tasks.sql
V6__create_habits.sql
V7__create_habit_completions.sql
V8__create_schedules.sql
V9__create_schedule_items.sql
V10__create_focus_sessions.sql
V11__create_notifications.sql
V12__create_productivity_stats.sql
V13__create_ai_conversations.sql
V14__create_ai_messages.sql
V15__create_ai_recommendations.sql
V16__create_user_availability.sql
V17__create_task_dependencies.sql
```

The exact migration sequence may change during implementation.

---

# 27. Database Constraints

The database should enforce important constraints where practical.

Examples:

```text
users.email → UNIQUE

productivity_stats:
(user_id, stat_date) → UNIQUE

task_dependencies:
(task_id, depends_on_task_id) → PRIMARY KEY

user_preferences.user_id → UNIQUE
```

Application-level validation should complement database constraints.

---

# 28. Data Integrity Rules

The system must prevent:

* Tasks belonging to another user from being scheduled.
* Schedule items with invalid time ranges.
* Negative task durations.
* Invalid progress percentages.
* Duplicate daily productivity statistics.
* Invalid task dependency relationships.
* Orphaned schedule items.

---

# 29. Example Data Flow

When a user creates a task:

```text
React
 ↓
POST /api/tasks
 ↓
TaskController
 ↓
TaskService
 ↓
Validate request
 ↓
Verify authenticated user
 ↓
TaskRepository
 ↓
MySQL
```

When generating a schedule:

```text
User
 ↓
Schedule API
 ↓
TaskService
 ↓
AvailabilityService
 ↓
AI Service
 ↓
Scheduler Engine
 ↓
Validation
 ↓
ScheduleRepository
 ↓
MySQL
```

---

# 30. Database as Source of Truth

MySQL is the authoritative persistent source of application data.

AI-generated data must not bypass:

```text
Service Layer
      ↓
Validation
      ↓
Repository
      ↓
Database
```

This prevents invalid AI output from corrupting the application state.

---

# 31. Future Database Extensions

Possible future tables:

```text
calendar_integrations
external_events
device_sessions
subscription_plans
usage_limits
ai_usage
notification_preferences
user_productivity_patterns
task_estimation_history
```

These should only be introduced when their corresponding features are implemented.

---

# 32. Final Database Architecture

The initial database will use a relational MySQL schema centered around:

```text
USERS
  │
  ├── USER_PREFERENCES
  ├── USER_AVAILABILITY
  ├── TASKS
  ├── GOALS
  ├── HABITS
  ├── SCHEDULES
  ├── FOCUS_SESSIONS
  ├── NOTIFICATIONS
  ├── PRODUCTIVITY_STATS
  ├── AI_CONVERSATIONS
  └── AI_RECOMMENDATIONS
```

The schema is intentionally designed to support both the MVP and future AI-driven personalization without requiring a complete database redesign.
