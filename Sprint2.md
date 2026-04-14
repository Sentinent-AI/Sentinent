# Sprint 2

## Sprint 2 Work Completed

### Backend
- Implemented JWT-based authentication with signup, login, bearer token support, and HttpOnly auth cookies.
- Added protected route handling through authentication middleware and context helpers for user ID and email.
- Added stricter CORS configuration with explicit allowed-origin validation.
- Built workspace management APIs for creating, listing, viewing, updating, and deleting workspaces.
- Added workspace membership tracking so access can be enforced by role instead of ownership alone.
- Built decision management APIs inside workspaces for create, list, view, update, and delete flows.
- Added invitation workflows so workspace owners can invite members by email, validate invitations, accept invitations, and cancel pending invitations.
- Added member management APIs so owners can list members, change roles, remove members, and transfer ownership.
- Built integrations infrastructure for Slack and GitHub, including OAuth entry points, callback handling, connection status, integration listing, and disconnect flows.
- Added GitHub repository sync support and a background sync service for provider integrations.
- Added signal APIs for listing signals, filtering by source and status, viewing individual signals, and updating read/archive state.
- Added GitHub webhook ingestion support for inbound repository activity.
- Expanded the SQLite schema to support workspaces, members, invitations, integrations, signals, signal status, and decisions.
- Added backend unit tests across handlers, middleware, services, and utilities.

### Frontend
- Connected the Angular frontend to the live backend APIs for authentication, workspaces, decisions, integrations, members, invitations, and signals.
- Added invitation acceptance UI and related test coverage.
- Added workspace member management UI and related test coverage.
- Added search UI and search service coverage for signals, decisions, and workspace-aware results.
- Added Angular unit tests across components, guards, and services.
- Added a Cypress login smoke test.

## Frontend Tests

### Angular Unit Tests
- `src/app/app.component.spec.ts`
  - App shell creation and basic bootstrap behavior.
- `src/app/components/accept-invitation/accept-invitation.spec.ts`
  - Invitation validation, acceptance flow, and auth-aware behavior.
- `src/app/components/dashboard/dashboard.spec.ts`
  - Dashboard initialization, workspace loading, and logout behavior.
- `src/app/components/login/login.spec.ts`
  - Login and signup validation, service calls, and error handling.
- `src/app/components/workspace-integrations/workspace-integrations.spec.ts`
  - Slack and GitHub integration UI behavior, sync actions, and selection persistence.
- `src/app/components/workspace-members/workspace-members.spec.ts`
  - Member listing, invitations, role updates, and member removal flows.
- `src/app/guards/auth-guard.spec.ts`
  - Route protection for authenticated and unauthenticated users.
- `src/app/services/integration.service.spec.ts`
  - Integration API request behavior and response mapping.
- `src/app/services/search.service.spec.ts`
  - Search aggregation behavior for signals, decisions, and workspaces.
- `src/app/services/signal.service.spec.ts`
  - Signal loading and action behavior.
- `src/app/services/workspace-member.service.spec.ts`
  - Workspace member and invitation service calls.
- `src/app/services/workspace.spec.ts`
  - Workspace service CRUD request behavior.

### Cypress End-to-End Test
- `cypress/e2e/login.cy.ts`
  - Verifies the login flow as a frontend smoke test.

## Backend Unit Tests

### Handler Tests
- `handlers/auth_test.go`
  - Covers signup, signin, invalid email handling, and secure cookie behavior in production mode.
- `handlers/integrations_test.go`
  - Covers Slack and GitHub OAuth state validation, callback validation, workspace-filtered integrations, deletion permissions, and integration status responses.
- `handlers/invitations_test.go`
  - Covers invitation lifecycle, listing and canceling invitations, ownership transfer, token generation, and path helper behavior.
- `handlers/signals_test.go`
  - Covers resolved signal status behavior for signal lists and workspace-scoped totals.
- `handlers/workspaces_test.go`
  - Covers workspace CRUD lifecycle and workspace decision lifecycle.

### Middleware Tests
- `middleware/auth_test.go`
  - Covers bearer token auth, cookie auth, missing tokens, and malformed token rejection.
- `middleware/cors_test.go`
  - Covers allowed origins, disallowed origins, preflight handling, and invalid origin configuration.
- `middleware/roles_test.go`
  - Covers workspace role fallback and role-based access helpers.

### Service Tests
- `services/github_test.go`
  - Covers GitHub issue and pull request splitting logic.
- `services/sync_test.go`
  - Covers Slack sync deduplication, metadata updates, and safe handling of Slack `not_in_channel` errors.

### Utility Tests
- `utils/encryption_test.go`
  - Covers token encryption and key padding behavior.
- `utils/validation_test.go`
  - Covers email validation logic.

## Backend API Documentation

Base URL: `http://localhost:8080`

Auth notes:
- Protected routes require JWT authentication.
- The backend accepts either a bearer token or the auth cookie set during login.
- Most protected routes return `401 Unauthorized` when auth is missing or invalid.

### Authentication

#### `POST /api/signup`
- Description: Creates a new user account.
- Auth: No
- Request body:
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```
- Success:
  - `201 Created`
- Common errors:
  - `400 Bad Request` for invalid JSON or invalid email format
  - `409 Conflict` if the email already exists

#### `POST /api/login`
- Description: Signs in a user and returns a JWT token in JSON while also setting an auth cookie.
- Auth: No
- Request body:
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```
- Success:
  - `200 OK`
```json
{
  "token": "jwt-token"
}
```
- Common errors:
  - `400 Bad Request` for invalid JSON or invalid email
  - `401 Unauthorized` for invalid credentials
  - `500 Internal Server Error` for missing server JWT configuration

#### `GET /api/protected`
- Description: Simple authenticated route used to verify auth wiring.
- Auth: Yes
- Success:
  - `200 OK`
  - Plain-text greeting response

### Workspaces

#### `GET /api/workspaces`
- Description: Lists workspaces where the authenticated user is a member.
- Auth: Yes
- Success:
  - `200 OK`

#### `POST /api/workspaces`
- Description: Creates a workspace and automatically adds the creator as owner.
- Auth: Yes
- Request body:
```json
{
  "name": "Platform",
  "description": "Platform engineering workspace"
}
```
- Success:
  - `201 Created`
- Common errors:
  - `400 Bad Request` for invalid JSON or blank name
  - `500 Internal Server Error` if workspace or membership creation fails

#### `GET /api/workspaces/:id`
- Description: Returns one workspace if the authenticated user is a member.
- Auth: Yes
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request` for invalid workspace ID
  - `403 Forbidden` if the user is not a member
  - `404 Not Found`

#### `PATCH /api/workspaces/:id`
- Description: Updates a workspace's name and description.
- Auth: Yes
- Authorization: Owner only
- Request body:
```json
{
  "name": "Platform Team",
  "description": "Updated workspace description"
}
```
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request`
  - `403 Forbidden`
  - `404 Not Found`

#### `DELETE /api/workspaces/:id`
- Description: Deletes a workspace and related records.
- Auth: Yes
- Authorization: Owner only
- Success:
  - `204 No Content`
- Common errors:
  - `400 Bad Request`
  - `403 Forbidden`

### Decisions

#### `GET /api/workspaces/:id/decisions`
- Description: Lists decisions for a workspace.
- Auth: Yes
- Authorization: Any workspace member
- Success:
  - `200 OK`

#### `POST /api/workspaces/:id/decisions`
- Description: Creates a decision in a workspace.
- Auth: Yes
- Authorization: Owner or member
- Request body:
```json
{
  "title": "Choose notification strategy",
  "description": "Decide how alerts should be prioritized",
  "status": "OPEN",
  "due_date": "2026-04-01T00:00:00Z"
}
```
- Success:
  - `201 Created`
- Common errors:
  - `400 Bad Request` for invalid JSON or invalid fields
  - `403 Forbidden`

#### `GET /api/workspaces/:id/decisions/:decisionId`
- Description: Returns one decision in a workspace.
- Auth: Yes
- Authorization: Any workspace member
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request`
  - `403 Forbidden`
  - `404 Not Found`

#### `PATCH /api/workspaces/:id/decisions/:decisionId`
- Description: Updates a decision.
- Auth: Yes
- Authorization: Owner or member
- Request body:
```json
{
  "title": "Choose notification strategy",
  "description": "Updated decision context",
  "status": "CLOSED",
  "due_date": null
}
```
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request`
  - `403 Forbidden`
  - `404 Not Found`

#### `DELETE /api/workspaces/:id/decisions/:decisionId`
- Description: Deletes a decision.
- Auth: Yes
- Authorization: Owner or member
- Success:
  - `204 No Content`
- Common errors:
  - `400 Bad Request`
  - `403 Forbidden`
  - `404 Not Found`

### Invitations and Members

#### `POST /api/workspaces/:id/invitations`
- Description: Creates a workspace invitation for an email address.
- Auth: Yes
- Authorization: Owner only
- Request body:
```json
{
  "email": "teammate@example.com",
  "role": "member"
}
```
- Success:
  - `201 Created`
- Common errors:
  - `400 Bad Request` for invalid workspace ID, invalid JSON, or invalid email
  - `403 Forbidden`
  - `409 Conflict` if the user is already a member or an active invitation exists

#### `GET /api/workspaces/:id/invitations`
- Description: Lists active invitations for a workspace.
- Auth: Yes
- Authorization: Owner only
- Success:
  - `200 OK`

#### `DELETE /api/workspaces/:id/invitations/:invitationId`
- Description: Cancels a pending invitation.
- Auth: Yes
- Authorization: Owner only
- Success:
  - `204 No Content`
- Common errors:
  - `400 Bad Request`
  - `403 Forbidden`
  - `404 Not Found`

#### `GET /api/invitations/:token`
- Description: Validates an invitation token and returns invitation metadata.
- Auth: No
- Success:
  - `200 OK`
```json
{
  "valid": true,
  "workspace": {
    "id": 1,
    "name": "Platform"
  },
  "invited_by": {
    "email": "owner@example.com"
  },
  "role": "member",
  "expires_at": "2026-04-01T00:00:00Z"
}
```
- Common errors:
  - `400 Bad Request`
  - `404 Not Found`
  - `410 Gone` if expired

#### `POST /api/invitations/:token/accept`
- Description: Accepts an invitation for the currently authenticated user.
- Auth: Yes
- Success:
  - `200 OK`
```json
{
  "workspace_id": 1,
  "role": "member"
}
```
- Common errors:
  - `400 Bad Request`
  - `403 Forbidden` if the logged-in email does not match the invitation
  - `404 Not Found`
  - `409 Conflict` if already a member
  - `410 Gone` if expired

#### `GET /api/workspaces/:id/members`
- Description: Lists members in a workspace.
- Auth: Yes
- Authorization: Any workspace member
- Success:
  - `200 OK`

#### `PATCH /api/workspaces/:id/members/:userId`
- Description: Updates a member role or transfers ownership.
- Auth: Yes
- Authorization: Owner only
- Request body:
```json
{
  "role": "viewer"
}
```
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request` for invalid IDs, invalid JSON, or invalid role
  - `403 Forbidden`
  - `404 Not Found`

#### `DELETE /api/workspaces/:id/members/:userId`
- Description: Removes a member from a workspace.
- Auth: Yes
- Authorization: Owner only
- Success:
  - `204 No Content`
- Common errors:
  - `400 Bad Request`
  - `403 Forbidden`
  - `404 Not Found`

### Integrations

#### `GET /api/integrations`
- Description: Lists integrations for the authenticated user.
- Auth: Yes
- Query params:
  - `workspace_id` optional
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request` for invalid `workspace_id`

#### `DELETE /api/integrations/:id`
- Description: Deletes an owned integration record.
- Auth: Yes
- Success:
  - `204 No Content`
- Common errors:
  - `400 Bad Request`
  - `404 Not Found`

#### `GET /api/integrations/status`
- Description: Returns provider configuration and connection status.
- Auth: Yes
- Query params:
  - `workspace_id` optional
- Success:
  - `200 OK`

#### `GET /api/integrations/slack/auth`
- Description: Starts Slack OAuth and returns the authorization URL.
- Auth: Yes
- Query params:
  - `workspace_id` required
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request`
  - `503 Service Unavailable` when Slack is not configured

#### `GET /api/integrations/slack/callback`
- Description: Handles the Slack OAuth callback and stores the integration.
- Auth: No
- Query params:
  - `code` required
  - `state` required
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request`
  - `500 Internal Server Error`
  - `503 Service Unavailable`

#### `GET /api/integrations/slack/channels`
- Description: Lists Slack channels for a connected integration.
- Auth: Yes
- Query params:
  - `integration_id` required
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request`
  - `404 Not Found`
  - `429 Too Many Requests`

#### `GET /api/integrations/github/auth`
- Description: Starts GitHub OAuth and returns the authorization URL.
- Auth: Yes
- Success:
  - `200 OK`
- Common errors:
  - `503 Service Unavailable` when GitHub is not configured

#### `GET /api/integrations/github/callback`
- Description: Handles the GitHub OAuth callback and stores the integration.
- Auth: No
- Query params:
  - `code` required
  - `state` required
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request`
  - `401 Unauthorized`
  - `500 Internal Server Error`

#### `GET /api/integrations/github/repos`
- Description: Lists repositories available through the authenticated user's GitHub integration.
- Auth: Yes
- Success:
  - `200 OK`

#### `POST /api/integrations/github/sync`
- Description: Starts a GitHub sync job.
- Auth: Yes
- Success:
  - `200 OK`
```json
{
  "status": "sync_started"
}
```

#### `DELETE /api/integrations/github`
- Description: Disconnects the authenticated user's GitHub integration.
- Auth: Yes
- Success:
  - `200 OK`

### Signals

#### `GET /api/signals`
- Description: Lists signals for the authenticated user.
- Auth: Yes
- Query params:
  - `source_type` optional
  - `status` optional
- Success:
  - `200 OK`

#### `GET /api/workspaces/:id/signals`
- Description: Lists workspace-scoped signals with optional filters and pagination.
- Auth: Yes
- Query params:
  - `source_type` optional
  - `status` optional
  - `limit` optional, defaults to `50`, max `100`
  - `offset` optional
- Success:
  - `200 OK`
```json
{
  "signals": [],
  "total": 0
}
```
- Common errors:
  - `400 Bad Request`

#### `GET /api/signals/:id`
- Description: Returns a single signal for the authenticated user.
- Auth: Yes
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request`
  - `404 Not Found`

#### `POST /api/signals/:id/read`
- Description: Marks a signal as read for the authenticated user.
- Auth: Yes
- Success:
  - `204 No Content`
- Common errors:
  - `400 Bad Request`
  - `404 Not Found`

#### `POST /api/signals/:id/archive`
- Description: Marks a signal as archived for the authenticated user.
- Auth: Yes
- Success:
  - `204 No Content`
- Common errors:
  - `400 Bad Request`
  - `404 Not Found`

### Webhooks

#### `POST /api/webhooks/github`
- Description: Receives GitHub webhook payloads.
- Auth: No
- Headers:
  - `X-GitHub-Event` required
- Success:
  - `200 OK`
- Common errors:
  - `400 Bad Request`
  - `405 Method Not Allowed`
