# Frontend Implementation Guide

This document provides the API contract and UI requirements for the frontend team to implement features corresponding to backend User Stories #13, #14, and #15.

---

## Overview

| US | Feature | Backend Branch | Status |
|----|---------|----------------|--------|
| US-8 | Team Member Invitations | `feature/us8-team-invitations` | In Progress |
| US-5 | Slack Integration | `feature/us13-slack-integration` | In Progress |
| US-6 | GitHub Integration | `feature/us14-github-integration` | In Progress |

---

## US-8: Team Member Invitations

### User Story
As a workspace owner, I want to invite team members via email so that we can collaborate on decisions together.

### Role Definitions
| Role | Permissions |
|------|-------------|
| `owner` | Full control: invite, remove members, change roles, delete workspace |
| `member` | Can view workspace, create/edit decisions, add comments |
| `viewer` | Can view workspace and decisions, read-only access |

### API Endpoints

#### 1. Create Invitation
```
POST /api/workspaces/:id/invitations
Authorization: Bearer <token>
Content-Type: application/json

{
  "email": "user@example.com",
  "role": "member"  // or "viewer"
}

Response 201 Created:
{
  "id": "uuid",
  "email": "user@example.com",
  "role": "member",
  "token": "invite_token_xyz",
  "expiresAt": "2026-04-01T00:00:00Z",
  "createdAt": "2026-03-23T00:00:00Z"
}

Response 403 Forbidden:  // If not owner
{ "error": "Only workspace owners can invite members" }

Response 409 Conflict:  // If email already invited or member
{ "error": "User is already a member or has pending invitation" }
```

#### 2. List Pending Invitations
```
GET /api/workspaces/:id/invitations
Authorization: Bearer <token>

Response 200 OK:
{
  "invitations": [
    {
      "id": "uuid",
      "email": "user@example.com",
      "role": "member",
      "expiresAt": "2026-04-01T00:00:00Z",
      "createdAt": "2026-03-23T00:00:00Z"
    }
  ]
}
```

#### 3. Cancel Invitation
```
DELETE /api/workspaces/:id/invitations/:invitationId
Authorization: Bearer <token>

Response 204 No Content
```

#### 4. Validate Invitation (Public)
```
GET /api/invitations/:token

Response 200 OK:
{
  "valid": true,
  "workspace": {
    "id": "uuid",
    "name": "Engineering Team"
  },
  "invitedBy": {
    "email": "owner@example.com"
  },
  "role": "member"
}

Response 410 Gone:
{ "error": "Invitation expired or invalid" }
```

#### 5. Accept Invitation
```
POST /api/invitations/:token/accept
Authorization: Bearer <token>  // User must be logged in

Response 200 OK:
{
  "workspaceId": "uuid",
  "role": "member"
}

Response 409 Conflict:
{ "error": "You are already a member of this workspace" }
```

#### 6. List Workspace Members
```
GET /api/workspaces/:id/members
Authorization: Bearer <token>

Response 200 OK:
{
  "members": [
    {
      "userId": 1,
      "email": "owner@example.com",
      "role": "owner",
      "joinedAt": "2026-03-01T00:00:00Z"
    },
    {
      "userId": 2,
      "email": "member@example.com",
      "role": "member",
      "joinedAt": "2026-03-23T00:00:00Z"
    }
  ]
}
```

#### 7. Update Member Role
```
PATCH /api/workspaces/:id/members/:userId
Authorization: Bearer <token>
Content-Type: application/json

{
  "role": "viewer"
}

Response 200 OK:
{
  "userId": 2,
  "email": "member@example.com",
  "role": "viewer",
  "joinedAt": "2026-03-23T00:00:00Z"
}

Response 403 Forbidden:
{ "error": "Cannot modify owner's role" }
```

#### 8. Remove Member
```
DELETE /api/workspaces/:id/members/:userId
Authorization: Bearer <token>

Response 204 No Content

Response 403 Forbidden:
{ "error": "Cannot remove workspace owner" }
```

### UI Components Required

1. **Workspace Settings Page** (`/workspaces/:id/settings/members`)
   - Member list with avatars, emails, roles
   - Role dropdown for each member (owner only)
   - Remove member button (owner only, with confirmation)
   - "Invite Member" button

2. **Invite Member Modal**
   - Email input field
   - Role selector (Member/Viewer)
   - Submit button
   - Success: Show invitation link (for copy-paste)

3. **Pending Invitations Section**
   - List of pending invites with email, role, expiry
   - Cancel button for each
   - Resend option

4. **Accept Invitation Page** (`/invitations/:token`)
   - Show workspace name, inviter info
   - "Join Workspace" button (requires login)
   - Redirect to login if not authenticated
   - After login, redirect back to accept

### State Management

```typescript
// models/workspace-member.model.ts
interface WorkspaceMember {
  userId: number;
  email: string;
  role: 'owner' | 'member' | 'viewer';
  joinedAt: Date;
}

interface Invitation {
  id: string;
  email: string;
  role: 'member' | 'viewer';
  expiresAt: Date;
  createdAt: Date;
}

// services/workspace-member.service.ts
class WorkspaceMemberService {
  getMembers(workspaceId: string): Observable<WorkspaceMember[]>;
  inviteMember(workspaceId: string, email: string, role: string): Observable<Invitation>;
  updateRole(workspaceId: string, userId: number, role: string): Observable<WorkspaceMember>;
  removeMember(workspaceId: string, userId: number): Observable<void>;
  getPendingInvitations(workspaceId: string): Observable<Invitation[]>;
  cancelInvitation(workspaceId: string, invitationId: string): Observable<void>;
  validateInvitation(token: string): Observable<any>;
  acceptInvitation(token: string): Observable<any>;
}
```

### Route Guards

- **OwnerGuard**: Only owners can access `/workspaces/:id/settings/*`
- **MemberGuard**: Only members can access `/workspaces/:id/*` (viewers included)

---

## US-5: Slack Integration

### User Story
As a workspace member, I want to connect my Slack workspace so that I can receive and respond to Slack messages without leaving Sentinent.

### API Endpoints

#### 1. Initiate Slack OAuth
```
GET /api/integrations/slack/auth
Authorization: Bearer <token>

Response 200 OK:
{
  "authUrl": "https://slack.com/oauth/v2/authorize?client_id=...&scope=...&state=..."
}
```

#### 2. OAuth Callback (Backend handles, redirects to frontend)
```
GET /api/integrations/slack/callback?code=...&state=...

Success redirect: /dashboard?slack=connected
Error redirect: /dashboard?slack=error&message=...
```

#### 3. Get Connected Channels
```
GET /api/integrations/slack/channels
Authorization: Bearer <token>

Response 200 OK:
{
  "connected": true,
  "channels": [
    {
      "id": "C123456",
      "name": "general",
      "isConnected": true
    },
    {
      "id": "C789012",
      "name": "engineering",
      "isConnected": false
    }
  ]
}

Response 404 Not Connected:
{ "connected": false }
```

#### 4. Update Channel Selection
```
PUT /api/integrations/slack/channels
Authorization: Bearer <token>
Content-Type: application/json

{
  "channelIds": ["C123456", "C789012"]
}

Response 200 OK
```

#### 5. Disconnect Slack
```
DELETE /api/integrations/slack
Authorization: Bearer <token>

Response 204 No Content
```

#### 6. List Signals (Shared with GitHub)
```
GET /api/workspaces/:id/signals?source=slack&status=unread&page=1&limit=20
Authorization: Bearer <token>

Response 200 OK:
{
  "signals": [
    {
      "id": "uuid",
      "sourceType": "slack",
      "sourceId": "C123456",
      "externalId": "1234567890.123456",
      "title": "Message from #general",
      "content": "Hey team, check out the new deployment...",
      "author": "@john.doe",
      "status": "unread",
      "receivedAt": "2026-03-23T10:00:00Z",
      "url": "https://workspace.slack.com/archives/C123456/p1234567890123456",
      "metadata": {
        "channel": "general",
        "channelId": "C123456",
        "timestamp": "1234567890.123456",
        "user": "U123456"
      }
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "hasMore": true
  }
}
```

#### 7. Mark Signal as Read
```
POST /api/signals/:id/read
Authorization: Bearer <token>

Response 200 OK:
{
  "id": "uuid",
  "status": "read"
}
```

#### 8. Archive Signal
```
POST /api/signals/:id/archive
Authorization: Bearer <token>

Response 200 OK:
{
  "id": "uuid",
  "status": "archived"
}
```

### UI Components Required

1. **Integration Settings Page** (`/workspaces/:id/settings/integrations`)
   - Slack connection card
     - Connect button (opens OAuth popup/redirect)
     - Disconnect button
     - Connected workspace info
   - Channel selection (multi-select dropdown)
   - Sync status indicator

2. **Signals Dashboard** (`/dashboard` or `/workspaces/:id/signals`)
   - Filter tabs: All | Slack | GitHub | Decisions
   - Signal list with:
     - Source icon (Slack logo)
     - Channel name badge
     - Author
     - Preview text
     - Timestamp
     - Unread indicator
   - Mark as read / Archive actions
   - Infinite scroll pagination

3. **Signal Detail Modal/Drawer**
   - Full message content
   - Link to open in Slack
   - Author info
   - Thread context (if available)

### State Management

```typescript
// models/signal.model.ts
interface Signal {
  id: string;
  sourceType: 'slack' | 'github' | 'email' | 'decision';
  sourceId: string;
  externalId: string;
  title: string;
  content: string;
  author: string;
  status: 'unread' | 'read' | 'archived';
  receivedAt: Date;
  url?: string;
  metadata: Record<string, any>;
}

// services/integration.service.ts
class IntegrationService {
  getSlackAuthUrl(): Observable<{ authUrl: string }>;
  getSlackChannels(): Observable<{ connected: boolean; channels: SlackChannel[] }>;
  updateSlackChannels(channelIds: string[]): Observable<void>;
  disconnectSlack(): Observable<void>;
}

// services/signal.service.ts
class SignalService {
  getSignals(workspaceId: string, filters: SignalFilters): Observable<PaginatedSignals>;
  markAsRead(signalId: string): Observable<Signal>;
  archiveSignal(signalId: string): Observable<Signal>;
}

interface SignalFilters {
  source?: 'slack' | 'github' | 'email' | 'decision';
  status?: 'unread' | 'read' | 'archived';
  page?: number;
  limit?: number;
}
```

---

## US-6: GitHub Integration

### User Story
As a workspace member, I want to see my GitHub Issues and PRs assigned to me in my dashboard so that I can track development work alongside other notifications.

### API Endpoints

#### 1. Initiate GitHub OAuth
```
GET /api/integrations/github/auth
Authorization: Bearer <token>

Response 200 OK:
{
  "authUrl": "https://github.com/login/oauth/authorize?client_id=...&scope=read:user+read:org+repo&state=..."
}
```

#### 2. OAuth Callback
```
GET /api/integrations/github/callback?code=...&state=...

Success redirect: /dashboard?github=connected
Error redirect: /dashboard?github=error&message=...
```

#### 3. Get Connected Repositories
```
GET /api/integrations/github/repos
Authorization: Bearer <token>

Response 200 OK:
{
  "connected": true,
  "repos": [
    {
      "id": 123456,
      "name": "sentinent",
      "fullName": "Sentinent-AI/sentinent",
      "isConnected": true
    }
  ]
}
```

#### 4. Update Repository Selection
```
PUT /api/integrations/github/repos
Authorization: Bearer <token>
Content-Type: application/json

{
  "repoIds": [123456, 789012]
}

Response 200 OK
```

#### 5. Disconnect GitHub
```
DELETE /api/integrations/github
Authorization: Bearer <token>

Response 204 No Content
```

#### 6. Trigger Manual Sync
```
POST /api/integrations/github/sync
Authorization: Bearer <token>

Response 202 Accepted:
{
  "syncId": "uuid",
  "status": "in_progress"
}
```

#### 7. Get Sync Status
```
GET /api/integrations/github/sync/:syncId
Authorization: Bearer <token>

Response 200 OK:
{
  "syncId": "uuid",
  "status": "completed",  // or "in_progress", "failed"
  "itemsSynced": 42,
  "completedAt": "2026-03-23T10:05:00Z"
}
```

### Signal Format for GitHub

GitHub issues and PRs appear in the signals API with `sourceType: 'github'`:

```json
{
  "id": "uuid",
  "sourceType": "github",
  "sourceId": "Sentinent-AI/sentinent",
  "externalId": "42",
  "title": "Fix authentication bug",
  "content": "Users are reporting login issues...",
  "author": "@johndoe",
  "status": "unread",
  "receivedAt": "2026-03-23T09:00:00Z",
  "url": "https://github.com/Sentinent-AI/sentinent/issues/42",
  "metadata": {
    "type": "issue",  // or "pull_request"
    "number": 42,
    "repository": "Sentinent-AI/sentinent",
    "state": "open",  // or "closed"
    "labels": ["bug", "high-priority"],
    "assignees": ["@johndoe"],
    "createdAt": "2026-03-22T00:00:00Z",
    "updatedAt": "2026-03-23T09:00:00Z"
  }
}
```

### UI Components Required

1. **Integration Settings Page** (extend Slack section)
   - GitHub connection card
     - Connect button
     - Disconnect button
     - Connected account info
   - Repository selection (multi-select)
   - Last sync time
   - "Sync Now" button

2. **Signals Dashboard** (shared with Slack)
   - GitHub filter tab
   - GitHub-specific signal cards:
     - Issue/PR icon
     - Repository badge
     - Labels (color-coded)
     - State badge (Open/Closed)
     - Assignee avatars

3. **Signal Detail for GitHub**
   - Issue/PR title and description
   - Labels
   - Link to GitHub
   - Comment count
   - State indicator

### State Management

```typescript
// Extend integration.service.ts
class IntegrationService {
  // ... Slack methods ...

  getGitHubAuthUrl(): Observable<{ authUrl: string }>;
  getGitHubRepos(): Observable<{ connected: boolean; repos: GitHubRepo[] }>;
  updateGitHubRepos(repoIds: number[]): Observable<void>;
  disconnectGitHub(): Observable<void>;
  syncGitHub(): Observable<{ syncId: string }>;
  getSyncStatus(syncId: string): Observable<SyncStatus>;
}

interface GitHubRepo {
  id: number;
  name: string;
  fullName: string;
  isConnected: boolean;
}

interface SyncStatus {
  syncId: string;
  status: 'in_progress' | 'completed' | 'failed';
  itemsSynced?: number;
  completedAt?: Date;
}
```

---

## Shared Components

### Integration Card Component
Reusable card for connection status:
```typescript
interface IntegrationCardProps {
  provider: 'slack' | 'github';
  icon: string;
  name: string;
  description: string;
  isConnected: boolean;
  onConnect: () => void;
  onDisconnect: () => void;
  accountInfo?: {
    name: string;
    avatar?: string;
  };
}
```

### Signal List Component
```typescript
interface SignalListProps {
  workspaceId: string;
  filters: SignalFilters;
  onSignalClick: (signal: Signal) => void;
  onMarkAsRead: (signalId: string) => void;
  onArchive: (signalId: string) => void;
}
```

### Signal Filters Component
```typescript
interface SignalFiltersProps {
  filters: SignalFilters;
  onChange: (filters: SignalFilters) => void;
  options: {
    sources: Array<{ value: string; label: string; icon: string }>;
    statuses: Array<{ value: string; label: string }>;
  };
}
```

---

## Route Updates

Add to `app.routes.ts`:

```typescript
{
  path: 'workspaces/:id/settings',
  loadComponent: () => import('./components/workspace-settings/workspace-settings').then(m => m.WorkspaceSettingsComponent),
  canActivate: [authGuard, ownerGuard],
  children: [
    { path: 'members', component: WorkspaceMembersComponent },
    { path: 'integrations', component: WorkspaceIntegrationsComponent },
    { path: '', redirectTo: 'members', pathMatch: 'full' }
  ]
},
{
  path: 'invitations/:token',
  loadComponent: () => import('./components/accept-invitation/accept-invitation').then(m => m.AcceptInvitationComponent)
}
```

---

## Environment Variables

Add to environment files:

```typescript
export const environment = {
  // ... existing ...
  slackClientId: 'your-slack-client-id',
  githubClientId: 'your-github-client-id',
};
```

---

## Error Handling

Common error responses to handle:

| Status | Meaning | UI Action |
|--------|---------|-----------|
| 401 | Unauthorized | Redirect to login |
| 403 | Forbidden | Show "permission denied" message |
| 404 | Not found | Show 404 page |
| 409 | Conflict | Show specific error (e.g., "Already a member") |
| 410 | Gone | Show "invitation expired" |
| 429 | Rate limited | Show "too many requests, try later" |

---

## Testing Checklist

### US-8 Tests
- [ ] Owner can invite new member
- [ ] Owner can invite viewer
- [ ] Non-owner cannot invite
- [ ] Expired invitation shows error
- [ ] Accept invitation joins workspace
- [ ] Owner can change member role
- [ ] Owner can remove member
- [ ] Cannot remove owner

### US-5 Tests
- [ ] Slack OAuth flow completes
- [ ] Channels load after connection
- [ ] Channel selection persists
- [ ] Slack messages appear as signals
- [ ] Mark as read works
- [ ] Archive works
- [ ] Disconnect removes integration

### US-6 Tests
- [ ] GitHub OAuth flow completes
- [ ] Repositories load after connection
- [ ] Repo selection persists
- [ ] Issues/PRs appear as signals
- [ ] Manual sync triggers update
- [ ] Disconnect removes integration

---

*Document created: 2026-03-23*
*Backend branches: feature/us8-team-invitations, feature/us13-slack-integration, feature/us14-github-integration*
