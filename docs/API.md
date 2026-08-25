# Chronos AI Scheduler — API Specification

**Project:** Chronos AI Scheduler
**Version:** 1.0
**Status:** Planning
**Protocol:** HTTPS / REST
**Format:** JSON

---

# 1. API Overview

The backend exposes a REST API consumed by the React frontend.

Base path:

```text
/api
```

Example:

```text
GET /api/tasks
```

Production URLs will be configured through environment variables.

---

# 2. API Principles

The API should:

* Follow REST conventions.
* Use JSON request and response bodies.
* Use standard HTTP status codes.
* Validate all input.
* Authenticate protected endpoints.
* Authorize resource ownership.
* Return consistent errors.
* Avoid exposing internal implementation details.

---

# 3. Authentication

Authentication uses JWT-based authentication.

Login:

```text
POST /api/auth/login
```

Example request:

```json
{
  "email": "user@example.com",
  "password": "password"
}
```

Example response:

```json
{
  "success": true,
  "data": {
    "accessToken": "JWT_TOKEN",
    "tokenType": "Bearer"
  }
}
```

Protected requests:

```text
Authorization: Bearer <access-token>
```

---

# 4. HTTP Status Codes

The API should use:

| Status | Meaning                                  |
| ------ | ---------------------------------------- |
| 200    | Successful request                       |
| 201    | Resource created                         |
| 204    | Successful request with no response body |
| 400    | Invalid request                          |
| 401    | Authentication required/failed           |
| 403    | Access denied                            |
| 404    | Resource not found                       |
| 409    | Conflict                                 |
| 422    | Validation failure                       |
| 429    | Rate limit exceeded                      |
| 500    | Internal server error                    |
| 503    | External service unavailable             |

---

# 5. Standard Response Format

Successful response:

```json
{
  "success": true,
  "data": {}
}
```

Error response:

```json
{
  "success": false,
  "message": "Task not found",
  "errorCode": "TASK_NOT_FOUND",
  "timestamp": "2026-08-25T19:00:00"
}
```

Pagination responses may include:

```json
{
  "success": true,
  "data": [],
  "page": 0,
  "size": 20,
  "totalElements": 100,
  "totalPages": 5
}
```

---

# 6. Authentication API

## Register

```text
POST /api/auth/register
```

Request:

```json
{
  "name": "User",
  "email": "user@example.com",
  "password": "password"
}
```

Response:

```text
201 Created
```

---

## Login

```text
POST /api/auth/login
```

---

## Logout

```text
POST /api/auth/logout
```

---

## Refresh Token

```text
POST /api/auth/refresh
```

---

# 7. User API

## Get Profile

```text
GET /api/users/me
```

---

## Update Profile

```text
PUT /api/users/me
```

---

## Get Preferences

```text
GET /api/users/me/preferences
```

---

## Update Preferences

```text
PUT /api/users/me/preferences
```

---

# 8. Availability API

## Get Availability

```text
GET /api/users/me/availability
```

---

## Create Availability

```text
POST /api/users/me/availability
```

Example:

```json
{
  "dayOfWeek": 1,
  "startTime": "17:00",
  "endTime": "22:00",
  "type": "AVAILABLE"
}
```

---

## Update Availability

```text
PUT /api/users/me/availability/{id}
```

---

## Delete Availability

```text
DELETE /api/users/me/availability/{id}
```

---

# 9. Task API

## Create Task

```text
POST /api/tasks
```

Request:

```json
{
  "title": "Study Java",
  "description": "Study exception handling",
  "priority": "HIGH",
  "estimatedDurationMinutes": 90,
  "deadline": "2026-08-26T21:00:00",
  "categoryId": 1,
  "goalId": 2
}
```

Response:

```text
201 Created
```

---

## Get Tasks

```text
GET /api/tasks
```

Possible query parameters:

```text
status
priority
category
goal
from
to
page
size
sort
```

Example:

```text
GET /api/tasks?status=PENDING&priority=HIGH
```

---

## Get Task

```text
GET /api/tasks/{id}
```

---

## Update Task

```text
PUT /api/tasks/{id}
```

---

## Delete Task

```text
DELETE /api/tasks/{id}
```

---

## Complete Task

```text
POST /api/tasks/{id}/complete
```

---

## Skip Task

```text
POST /api/tasks/{id}/skip
```

---

# 10. Natural Language Task API

```text
POST /api/ai/tasks/parse
```

Request:

```json
{
  "input": "Study Java for two hours tomorrow evening"
}
```

Response:

```json
{
  "success": true,
  "data": {
    "title": "Study Java",
    "durationMinutes": 120,
    "preferredPeriod": "EVENING",
    "category": "STUDY"
  }
}
```

The parsed result should be confirmed or validated before creating the actual task.

---

# 11. Category API

## Create Category

```text
POST /api/categories
```

## Get Categories

```text
GET /api/categories
```

## Update Category

```text
PUT /api/categories/{id}
```

## Delete Category

```text
DELETE /api/categories/{id}
```

---

# 12. Goal API

## Create Goal

```text
POST /api/goals
```

## Get Goals

```text
GET /api/goals
```

## Get Goal

```text
GET /api/goals/{id}
```

## Update Goal

```text
PUT /api/goals/{id}
```

## Delete Goal

```text
DELETE /api/goals/{id}
```

---

# 13. Habit API

## Create Habit

```text
POST /api/habits
```

## Get Habits

```text
GET /api/habits
```

## Update Habit

```text
PUT /api/habits/{id}
```

## Delete Habit

```text
DELETE /api/habits/{id}
```

## Complete Habit

```text
POST /api/habits/{id}/complete
```

---

# 14. Schedule API

## Get Daily Schedule

```text
GET /api/schedules/{date}
```

Example:

```text
GET /api/schedules/2026-08-25
```

---

## Generate Schedule

```text
POST /api/schedules/generate
```

Request:

```json
{
  "date": "2026-08-25"
}
```

---

## Regenerate Schedule

```text
POST /api/schedules/{id}/regenerate
```

---

## Get Schedule Items

```text
GET /api/schedules/{id}/items
```

---

## Update Schedule Item

```text
PUT /api/schedule-items/{id}
```

---

# 15. Rescheduling API

```text
POST /api/schedules/{id}/reschedule
```

Request:

```json
{
  "reason": "TASK_MISSED",
  "taskId": 42
}
```

The backend should recalculate the remaining schedule.

---

# 16. Focus API

## Start Focus Session

```text
POST /api/focus/sessions
```

Request:

```json
{
  "taskId": 42,
  "scheduleItemId": 100
}
```

---

## Pause Session

```text
POST /api/focus/sessions/{id}/pause
```

---

## Resume Session

```text
POST /api/focus/sessions/{id}/resume
```

---

## Complete Session

```text
POST /api/focus/sessions/{id}/complete
```

---

## Get Focus History

```text
GET /api/focus/sessions
```

---

# 17. Analytics API

## Daily Analytics

```text
GET /api/analytics/daily/{date}
```

---

## Weekly Analytics

```text
GET /api/analytics/weekly
```

---

## Productivity Trends

```text
GET /api/analytics/productivity
```

Possible parameters:

```text
from
to
```

---

# 18. AI API

## Generate Schedule

```text
POST /api/ai/schedule
```

---

## Prioritize Tasks

```text
POST /api/ai/prioritize
```

---

## Reschedule

```text
POST /api/ai/reschedule
```

---

## Daily Summary

```text
POST /api/ai/daily-summary
```

---

## Productivity Recommendations

```text
GET /api/ai/recommendations
```

---

# 19. AI Conversation API

## Create Conversation

```text
POST /api/ai/conversations
```

---

## Get Conversations

```text
GET /api/ai/conversations
```

---

## Get Conversation

```text
GET /api/ai/conversations/{id}
```

---

## Send Message

```text
POST /api/ai/conversations/{id}/messages
```

Request:

```json
{
  "message": "What should I work on now?"
}
```

---

# 20. Notification API

## Get Notifications

```text
GET /api/notifications
```

---

## Mark Notification as Read

```text
POST /api/notifications/{id}/read
```

---

## Mark All as Read

```text
POST /api/notifications/read-all
```

---

# 21. API Authorization

Every protected endpoint must identify the authenticated user.

The backend must verify that the requested resource belongs to that user.

Example:

```text
User A
 ↓
GET /api/tasks/500
 ↓
Does task 500 belong to User A?
 ↓
YES → Return task
NO → 403/404
```

Never rely solely on the frontend to enforce ownership.

---

# 22. API Validation

Requests must be validated using backend validation.

Examples:

```text
Title cannot be empty
Duration must be positive
Deadline must be valid
Priority must be recognized
Start time must be before end time
```

Invalid requests should return appropriate validation errors.

---

# 23. API Versioning

The initial API may use:

```text
/api
```

If breaking changes become necessary, a versioned API can be introduced:

```text
/api/v1
/api/v2
```

API versioning should be introduced when required rather than adding unnecessary complexity to the MVP.

---

# 24. OpenAPI Documentation

The backend will expose an OpenAPI/Swagger specification.

Development documentation should allow developers to inspect:

* Endpoints
* Request schemas
* Response schemas
* Authentication
* Validation errors

The generated OpenAPI specification should be treated as the authoritative API contract.

---

# 25. API Security Rules

Never:

* Return password hashes.
* Return JWT secrets.
* Return API keys.
* Accept user IDs blindly.
* Trust AI-generated IDs without validation.
* Expose stack traces in production.

Always:

* Authenticate protected requests.
* Validate input.
* Authorize resources.
* Rate-limit sensitive endpoints.
* Use HTTPS in production.

---

# 26. API Request Flow

```text
React
 ↓
HTTP Request
 ↓
Security Filter
 ↓
JWT Validation
 ↓
Controller
 ↓
Request Validation
 ↓
Service
 ↓
Business Logic
 ↓
Repository / AI Service
 ↓
Response
 ↓
React
```

---

# 27. API Contract Principle

The API should act as the boundary between the frontend and backend.

The frontend should never directly access:

* MySQL
* Redis
* AI provider APIs
* Backend internal services

All communication must go through controlled backend APIs.
