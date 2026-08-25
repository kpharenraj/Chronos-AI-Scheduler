# Chronos AI Scheduler — Security Specification

**Project:** Chronos AI Scheduler
**Version:** 1.0
**Status:** Phase 0 Complete - Ready for Phase 1

---

# 1. Security Overview

Chronos AI Scheduler stores personal productivity information and therefore requires strong security controls.

Security must be considered throughout:

```text
Frontend
 ↓
API
 ↓
Authentication
 ↓
Authorization
 ↓
Business Logic
 ↓
Database
 ↓
AI Provider
```

---

# 2. Security Principles

The application follows:

* Least privilege
* Defense in depth
* Secure defaults
* Input validation
* Authentication
* Authorization
* Data minimization
* Secret management
* Secure logging
* Dependency management

---

# 3. Authentication

Authentication will use:

```text
Spring Security
+
JWT
```

Users authenticate using:

```text
Email
Password
```

Passwords must never be stored as plaintext.

---

# 4. Password Security

Passwords must be securely hashed using a modern password hashing algorithm supported by the backend security configuration.

The database stores:

```text
password_hash
```

not:

```text
password
```

Passwords must never appear in:

* Logs
* API responses
* Database queries
* AI prompts
* Error messages

---

# 5. JWT Security

JWT secrets must be stored in environment variables or a secure secret-management system.

Never commit:

```text
JWT_SECRET
```

to Git.

Tokens should have appropriate expiration times.

Long-lived authentication should use a secure refresh-token strategy where required.

---

# 6. Authorization

Authentication answers:

> Who are you?

Authorization answers:

> Are you allowed to access this resource?

Every protected resource must be checked for ownership.

Example:

```text
User A
 ↓
GET /api/tasks/100
 ↓
Does Task 100 belong to User A?
 ↓
YES → Return
NO → Reject
```

Never trust a user-provided `userId`.

---

# 7. Resource Ownership

The backend must derive the authenticated user from the security context.

For example:

```text
Authenticated User
        ↓
SecurityContext
        ↓
Current User ID
        ↓
Service
        ↓
Repository query filtered by user ID
```

This prevents users from accessing another user's tasks, schedules, goals, or analytics.

---

# 8. Input Validation

All external input must be validated.

Examples:

```text
Email format
Password requirements
Task title length
Duration
Priority
Deadline
Schedule times
IDs
Pagination parameters
```

Validation must happen on the backend even if the frontend performs validation.

---

# 9. SQL Injection Protection

The application should use:

```text
Spring Data JPA
Hibernate
Parameterized queries
```

Avoid constructing SQL using raw string concatenation.

Bad:

```text
"SELECT * FROM tasks WHERE title = '" + userInput + "'"
```

Use parameterized persistence mechanisms instead.

---

# 10. XSS Protection

User-generated content must be safely handled.

Potentially unsafe content includes:

* Task descriptions
* Notes
* AI-generated text
* Conversation messages

The frontend must not blindly render arbitrary HTML from users or AI providers.

---

# 11. CSRF

CSRF protection must be configured according to the selected authentication/token storage architecture.

If browser cookies are used for authentication, CSRF protection must be enabled appropriately.

If authentication is implemented using Authorization headers with tokens that are not automatically attached by browsers, the CSRF threat model differs and must be reviewed accordingly.

---

# 12. CORS

CORS should only allow trusted frontend origins.

Development may allow:

```text
http://localhost:5173
```

Production should allow only the actual deployed frontend origin.

Never use unrestricted wildcard origins in production when credentials are involved.

---

# 13. HTTPS

Production communication must use HTTPS.

The application must not transmit:

* Passwords
* JWTs
* User data
* AI requests

over unencrypted HTTP in production.

---

# 14. API Rate Limiting

Rate limiting should be applied especially to:

```text
Login
Registration
Password reset
AI endpoints
Conversation endpoints
```

Redis may later be used to implement distributed rate limiting.

---

# 15. AI Security

AI providers are external services.

The application must never send:

```text
Passwords
JWT secrets
Database credentials
API keys
Internal security configuration
```

to an AI provider.

Only necessary user context should be sent.

---

# 16. AI Prompt Injection

User-created content may contain instructions designed to manipulate the AI.

For example, a task title might contain malicious instructions.

The AI system should treat task content as untrusted data.

Architecture:

```text
User Content
 ↓
Sanitize / Validate
 ↓
Clearly separate data from instructions
 ↓
AI Prompt
 ↓
Structured Output
 ↓
Backend Validation
```

AI output must never automatically receive permission to perform arbitrary application actions.

---

# 17. AI Output Validation

AI output is untrusted.

For example:

```json
{
  "taskId": 999999,
  "startTime": "02:00",
  "endTime": "25:00"
}
```

must not be blindly accepted.

The backend must verify:

```text
Task exists
Task belongs to user
Time is valid
Duration is valid
No conflict exists
Within availability
Deadline is respected
```

---

# 18. Secret Management

Secrets must be stored outside source code.

Examples:

```text
DATABASE_PASSWORD
JWT_SECRET
AI_API_KEY
REDIS_PASSWORD
```

Development:

```text
.env
```

Production:

Use the deployment platform's secret management system or a dedicated secret manager.

---

# 19. Environment Files

The repository may contain:

```text
.env.example
```

but must never contain real secrets.

Example:

```text
AI_API_KEY=your_api_key_here
```

Real:

```text
AI_API_KEY=actual-secret
```

must never be committed.

---

# 20. Git Security

Before every commit, developers should ensure:

* No `.env` files
* No API keys
* No passwords
* No private certificates
* No database dumps containing sensitive data
* No production credentials

are committed.

---

# 21. Database Security

Database access should:

* Use dedicated credentials.
* Follow least privilege.
* Avoid using root accounts from the application.
* Use encrypted connections where appropriate.
* Restrict network access.
* Use strong passwords.

The application database user should have only the permissions required by the application.

---

# 22. Database Backup Security

Production backups should be:

* Encrypted
* Access-controlled
* Regularly created
* Tested for restoration

Backups should not be stored publicly.

---

# 23. Sensitive Data Minimization

The application should store only data required for its functionality.

For example, AI schedule generation does not need:

```text
password
password_hash
JWT
```

It may need:

```text
task title
duration
deadline
priority
availability
```

---

# 24. Logging Security

Logs must not contain:

* Passwords
* Tokens
* API keys
* Database credentials
* Sensitive personal information unnecessarily

Safe example:

```text
AI schedule request failed
provider=example
operation=schedule-generation
error=timeout
```

Unsafe:

```text
AI_KEY=actual-secret
```

---

# 25. Error Handling

Production errors should not expose:

* Stack traces
* SQL statements
* Internal file paths
* Credentials
* Internal architecture details

Users should receive safe error messages.

Detailed errors should be available only through protected server logs.

---

# 26. Dependency Security

Dependencies should be regularly reviewed.

This includes:

```text
Java dependencies
npm packages
Docker images
Operating system packages
```

Remove unused dependencies.

Security updates should be applied regularly.

---

# 27. Frontend Security

The frontend must:

* Avoid storing sensitive information unnecessarily.
* Avoid exposing API keys.
* Validate user input.
* Handle authentication securely.
* Avoid unsafe HTML rendering.
* Use HTTPS in production.

AI provider API keys must never be placed in frontend code.

---

# 28. API Key Protection

AI provider requests must go through the backend.

Correct:

```text
React
 ↓
Spring Boot
 ↓
AI Provider
```

Incorrect:

```text
React
 ↓
AI Provider
```

because the API key could be exposed to users.

---

# 29. Session Security

Authentication sessions/tokens should:

* Expire appropriately.
* Be invalidated when necessary.
* Avoid unnecessary long-lived access tokens.
* Be protected from client-side exposure as much as the selected authentication architecture allows.

---

# 30. Account Security

Future features may include:

* Email verification
* Password reset
* Login attempt protection
* Account lockout/rate limiting
* Two-factor authentication
* Session management
* Device/session revocation

These should be added as the authentication system matures.

---

# 31. Privacy

The application should provide clear information about:

* What data is stored.
* What data is sent to AI providers.
* Why the data is used.
* How long data is retained.
* How users can delete their data.

---

# 32. User Data Deletion

Users should eventually be able to request deletion of their account and associated data.

Deletion must consider:

```text
Tasks
Schedules
Focus Sessions
Goals
Habits
Analytics
Notifications
AI Conversations
AI Recommendations
```

Deletion behavior should be explicitly defined before implementing the feature.

---

# 33. Security Testing

Security testing should include:

```text
Authentication tests
Authorization tests
Input validation tests
API access tests
SQL injection tests
XSS tests
Rate-limit tests
AI prompt-injection tests
Dependency vulnerability scans
```

---

# 34. Security Checklist

Before production:

```text
- [ ] HTTPS enabled
- [ ] Strong password hashing
- [ ] JWT secrets protected
- [ ] AI API keys protected
- [ ] CORS restricted
- [ ] Authentication tested
- [ ] Authorization tested
- [ ] Input validation enabled
- [ ] Rate limiting enabled
- [ ] Production error handling configured
- [ ] Secrets removed from Git
- [ ] Database access restricted
- [ ] Backups configured
- [ ] Dependencies reviewed
- [ ] AI output validated
- [ ] Prompt injection risks reviewed
- [ ] Logging reviewed
```

---

# 35. Security Principle

The fundamental security principle of Chronos AI Scheduler is:

> **Never trust input simply because it came from the frontend or an AI model.**

Every external input must pass through authentication, authorization, validation, and appropriate business rules before it can affect application state.
