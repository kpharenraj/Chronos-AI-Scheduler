# Chronos AI Scheduler — Development Guide

**Project:** Chronos AI Scheduler
**Version:** 1.0
**Status:** Phase 0 Complete - Ready for Phase 1

---

# 1. Development Overview

This document defines the development workflow for Chronos AI Scheduler.

The project uses:

```text
Frontend:
React + TypeScript

Backend:
Java 21 + Spring Boot

Database:
MySQL

Cache:
Redis

AI:
AI Provider through Spring AI

Build:
Maven + npm

Containerization:
Docker

Version Control:
Git
```

---

# 2. Repository Structure

The repository will use a monorepo structure.

```text
chronos-ai-scheduler/
│
├── docs/
├── frontend/
├── backend/
├── database/
├── ai/
├── tests/
├── scripts/
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
```

---

# 3. Required Software

Developers should have:

* Git
* JDK 21
* Maven
* Node.js LTS
* npm
* Docker
* Docker Compose
* Visual Studio Code or another IDE

Recommended VS Code extensions:

* Extension Pack for Java
* Spring Boot Extension Pack
* ESLint
* Prettier
* Docker
* GitLens
* Thunder Client or REST Client

---

# 4. Local Development

The application should eventually support running the full stack with:

```text
docker compose up
```

During early development, individual components may be run separately.

---

# 5. Environment Configuration

Create:

```text
.env
```

from:

```text
.env.example
```

Example variables:

```text
DATABASE_HOST=
DATABASE_PORT=
DATABASE_NAME=
DATABASE_USERNAME=
DATABASE_PASSWORD=

JWT_SECRET=

AI_API_KEY=

REDIS_HOST=
REDIS_PORT=
```

Never commit `.env`.

---

# 6. Git Workflow

The main branch should represent stable code.

Recommended branches:

```text
main
develop
feature/*
fix/*
refactor/*
docs/*
```

Example:

```text
feature/task-management
```

---

# 7. Commit Convention

Use clear commit messages.

Recommended format:

```text
type: description
```

Examples:

```text
feat: add task creation API
fix: prevent overlapping schedule items
docs: update database documentation
refactor: improve scheduler service
test: add task service tests
chore: update dependencies
```

---

# 8. Pull Request Workflow

Before merging:

1. Code must compile.
2. Tests must pass.
3. Formatting must pass.
4. No secrets should be committed.
5. Documentation should be updated when required.
6. API changes should update API documentation.

---

# 9. Backend Development

Backend modules should follow:

```text
module/
├── controller/
├── service/
├── repository/
├── entity/
└── dto/
```

Not every module requires every folder.

Business logic belongs in services rather than controllers.

---

# 10. Controller Rules

Controllers should:

* Receive requests.
* Validate requests.
* Call services.
* Return responses.

Controllers should not contain complex business logic.

Bad:

```text
Controller
 ├── calculate schedule
 ├── query database
 ├── call AI
 └── validate everything
```

Good:

```text
Controller
 ↓
Service
 ↓
Business Logic
 ↓
Repository / AI Service
```

---

# 11. Service Rules

Services contain application and business logic.

Examples:

```text
TaskService
ScheduleService
FocusService
AnalyticsService
AIService
```

Services should remain testable and should not depend unnecessarily on HTTP-specific concepts.

---

# 12. Repository Rules

Repositories handle persistence.

Example:

```text
TaskRepository
ScheduleRepository
UserRepository
```

Repositories should not contain business logic.

---

# 13. DTO Rules

API request and response DTOs should be separated from database entities.

Example:

```text
TaskCreateRequest
TaskUpdateRequest
TaskResponse
```

Entities should not automatically become public API responses.

---

# 14. Database Development

Database changes must be implemented through Flyway migrations.

Never manually modify the production database schema.

Example:

```text
V1__create_users.sql
V2__create_tasks.sql
V3__create_schedules.sql
```

Each migration should be committed to Git.

---

# 15. Frontend Development

Frontend components should be reusable.

Recommended organization:

```text
components/
├── ui/
├── layout/
├── dashboard/
├── tasks/
├── scheduler/
├── focus/
└── analytics/
```

Pages should compose components rather than contain large amounts of reusable UI code.

---

# 16. Frontend State Management

Use:

### Zustand

For client-side application state such as:

* Authentication state
* UI preferences
* Local scheduler state

Use:

### TanStack Query

For server state such as:

* Tasks
* Schedules
* Goals
* Analytics
* Notifications

Avoid putting all API data into Zustand.

---

# 17. API Communication

Frontend API communication should go through centralized services.

Example:

```text
services/
├── api.ts
├── authApi.ts
├── taskApi.ts
├── scheduleApi.ts
├── aiApi.ts
└── analyticsApi.ts
```

Components should not contain scattered raw `fetch()` calls.

---

# 18. TypeScript Rules

Avoid unnecessary:

```text
any
```

Prefer explicit interfaces and types.

Example:

```typescript
interface Task {
  id: number;
  title: string;
  priority: TaskPriority;
  status: TaskStatus;
}
```

Types representing API responses should remain synchronized with the backend API contract.

---

# 19. UI Development

The UI should prioritize:

* Simplicity
* Readability
* Responsive design
* Accessibility
* Clear hierarchy
* Fast interaction

The scheduler should make the user's current task immediately visible.

---

# 20. AI Development

AI functionality should be isolated under:

```text
backend/.../ai/
```

Prompts should be version-controlled under:

```text
ai/prompts/
```

Schemas should be stored under:

```text
ai/schemas/
```

AI changes should be tested independently.

---

# 21. Testing Strategy

Testing levels:

```text
Unit Tests
     ↓
Integration Tests
     ↓
API Tests
     ↓
End-to-End Tests
```

---

# 22. Backend Unit Testing

Use:

```text
JUnit
Mockito
```

Test:

* Services
* Scheduling algorithms
* Validation
* Utility classes
* AI parsers

---

# 23. Backend Integration Testing

Use Spring Boot testing facilities.

Test:

* Database integration
* Repository behavior
* API endpoints
* Security
* Scheduling workflows

---

# 24. Frontend Testing

Use:

```text
Vitest
React Testing Library
```

Test:

* Components
* Forms
* Hooks
* State behavior
* User interactions

---

# 25. End-to-End Testing

Use Playwright.

Critical workflow:

```text
Register
 ↓
Login
 ↓
Create task
 ↓
Generate schedule
 ↓
Start focus session
 ↓
Complete task
 ↓
View analytics
```

---

# 26. Code Quality

The project should use:

* Consistent formatting
* ESLint
* Prettier
* Java formatting conventions
* Meaningful names
* Small functions
* Clear responsibilities
* Minimal duplication

---

# 27. Dependency Management

Dependencies should be added only when they solve a real problem.

Avoid adding libraries simply because they are popular.

Dependencies should be:

* Maintained
* Compatible
* Necessary
* Documented

---

# 28. Local Development Workflow

Recommended workflow:

```text
1. Pull latest changes
2. Create feature branch
3. Implement feature
4. Write tests
5. Run formatter
6. Run tests
7. Review changes
8. Commit
9. Push branch
10. Create pull request
```

---

# 29. Development Commands

The exact commands will be finalized during Phase 1.

Expected backend commands:

```bash
./mvnw spring-boot:run
```

Expected frontend commands:

```bash
npm install
npm run dev
```

Expected Docker command:

```bash
docker compose up
```

---

# 30. Documentation Rules

When implementing a feature:

If it changes:

```text
API
Database
Architecture
AI behavior
Security
```

the corresponding documentation should be updated.

---

# 31. Feature Development Process

Every major feature should follow:

```text
Requirement
 ↓
Design
 ↓
Database changes if required
 ↓
API design
 ↓
Backend implementation
 ↓
Frontend implementation
 ↓
Tests
 ↓
Documentation
 ↓
Review
```

---

# 32. AI-Assisted Development

AI will be used extensively during development.

However, AI-generated code must be:

* Reviewed
* Tested
* Understood
* Security-checked
* Integrated into the existing architecture

AI should not blindly generate entire features without architectural review.

---

# 33. Definition of Done

A feature is considered complete when:

* Requirements are implemented.
* Backend code is complete.
* Frontend code is complete.
* Database migrations are complete if required.
* Tests are written.
* Tests pass.
* API documentation is updated.
* Security implications are reviewed.
* Code is formatted.
* No secrets are committed.

---

# 34. Development Environment Principle

Development should remain reproducible.

A new developer should eventually be able to clone the repository and start the application with minimal manual configuration.

The long-term goal is:

```text
git clone
 ↓
configure .env
 ↓
docker compose up
 ↓
application running
```
