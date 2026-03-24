# Sentinent - Proposed User Stories

This document contains proposed user stories for future sprints of the Sentinent project. These stories progress naturally from the current MVP and align with the core mission: *centralizing signals, messages, and notifications from multiple platforms into a single, actionable interface.*

**Current MVP Status:**
- User Authentication (JWT-based) ✓
- Workspace management (create, view workspaces) ✓
- Decisions CRUD within workspaces (DRAFT, OPEN, CLOSED status) ✓

---

## 1. External Platform Integration (Core to Mission)

These stories deliver the core aggregator value proposition by connecting external platforms.

### US-5: Slack Integration
> **As a** workspace member, **I want** to connect my Slack workspace **so that** I can receive and respond to Slack messages without leaving Sentinent.

| Attribute | Details |
|-----------|---------|
| **Business Value** | Reduces context-switching; centralizes communication. |
| **Technical Scope** | Slack OAuth, webhook handlers, message storage, sync queue. |
| **Priority** | High |
| **Sprint** | Sprint 3 |

**Acceptance Criteria:**
- [ ] User can authenticate with Slack via OAuth
- [ ] User can select channels to monitor
- [ ] Slack messages appear as signals in the dashboard
- [ ] User can mark Slack messages as read from Sentinent
- [ ] Rate limiting and error handling for Slack API

---

### US-6: GitHub Integration
> **As a** workspace member, **I want** to see my GitHub Issues and PRs assigned to me in my dashboard **so that** I can track development work alongside other notifications.

| Attribute | Details |
|-----------|---------|
| **Business Value** | Unified view of dev work; nothing falls through cracks. |
| **Technical Scope** | GitHub OAuth, polling/API webhooks, issue caching. |
| **Priority** | High |
| **Sprint** | Sprint 3 |

**Acceptance Criteria:**
- [ ] User can connect GitHub account via OAuth
- [ ] Assigned issues and PRs appear as signals
- [ ] Status changes in GitHub reflect in Sentinent
- [ ] User can filter signals by GitHub specifically

---

### US-7: Email Integration
> **As a** workspace member, **I want** to connect my email account **so that** important emails appear as signals in my dashboard.

| Attribute | Details |
|-----------|---------|
| **Business Value** | Reduces email overload; prioritizes actionable messages. |
| **Technical Scope** | IMAP/OAuth for Gmail, email parsing, filtering rules. |
| **Priority** | Medium |
| **Sprint** | Sprint 4 |

**Acceptance Criteria:**
- [ ] User can connect Gmail via OAuth (or IMAP for other providers)
- [ ] Emails can be filtered by importance/priority
- [ ] Emails appear as signals with sender, subject, and preview
- [ ] User can archive emails from Sentinent

---

## 2. Team Collaboration & Access Control

### US-8: Team Member Invitations
> **As a** workspace owner, **I want** to invite team members via email **so that** we can collaborate on decisions together.

| Attribute | Details |
|-----------|---------|
| **Business Value** | Enables team coordination; workspace becomes multi-user. |
| **Technical Scope** | Invitation system, member roles (owner/member/viewer), join flows. |
| **Priority** | Critical |
| **Sprint** | Sprint 3 |

**Acceptance Criteria:**
- [ ] Workspace owner can invite users by email
- [ ] Invited users receive email with join link
- [ ] Member roles: Owner, Member, Viewer
- [ ] Owners can manage members (remove, change roles)
- [ ] Members can view and contribute based on role

---

### US-9: @Mentions and Notifications
> **As a** workspace member, **I want** to @mention colleagues in decision comments **so that** they get notified of relevant updates.

| Attribute | Details |
|-----------|---------|
| **Business Value** | Drives engagement; ensures visibility. |
| **Technical Scope** | Comment model, notification system, mention parsing. |
| **Priority** | Medium |
| **Sprint** | Sprint 4 |

**Acceptance Criteria:**
- [ ] User can add comments to decisions
- [ ] Typing @ shows list of workspace members
- [ ] Mentioned users receive in-app notification
- [ ] Optional: Email notification for mentions

---

## 3. Dashboard & Signal Management

### US-10: Signal Filtering
> **As a** dashboard user, **I want** to filter signals by source (Slack, GitHub, Email) and status **so that** I can focus on what matters now.

| Attribute | Details |
|-----------|---------|
| **Business Value** | Reduces noise; improves productivity. |
| **Technical Scope** | Filter UI, query params, backend filtering. |
| **Priority** | High |
| **Sprint** | Sprint 3 |

**Acceptance Criteria:**
- [ ] Filter by source: Slack, GitHub, Email, Decisions
- [ ] Filter by status: Unread, Read, Archived
- [ ] Filter by date range
- [ ] Filters persist in URL for sharing

---

### US-11: Signal Status Management
> **As a** dashboard user, **I want** to mark signals as "read" or archive them **so that** my inbox stays manageable.

| Attribute | Details |
|-----------|---------|
| **Business Value** | Inbox-zero workflow; prevents overload. |
| **Technical Scope** | Signal status (unread/read/archived), batch actions. |
| **Priority** | High |
| **Sprint** | Sprint 3 |

**Acceptance Criteria:**
- [ ] User can mark individual signals as read/unread/archived
- [ ] User can select multiple signals for batch actions
- [ ] "Mark all as read" option available
- [ ] Archive is reversible (can unarchive)

---

### US-12: Search Across Signals and Decisions
> **As a** dashboard user, **I want** to search across all signals and decisions **so that** I can quickly find past discussions or issues.

| Attribute | Details |
|-----------|---------|
| **Business Value** | Knowledge retrieval; faster decision-making. |
| **Technical Scope** | Full-text search (SQLite FTS), search UI, indexing. |
| **Priority** | Medium |
| **Sprint** | Sprint 4 |

**Acceptance Criteria:**
- [ ] Search by keyword across titles and content
- [ ] Filter search by source type
- [ ] Search results highlight matching terms
- [ ] Recent searches are saved

---

## 4. Decisions Enhancement

### US-13: Decision Due Dates and Reminders
> **As a** decision owner, **I want** to set due dates on decisions and receive reminders **so that** time-sensitive choices aren't missed.

| Attribute | Details |
|-----------|---------|
| **Business Value** | Accountability; timely decision-making. |
| **Technical Scope** | Date fields, reminder scheduler, notification queue. |
| **Priority** | Medium |
| **Sprint** | Sprint 4 |

**Acceptance Criteria:**
- [ ] User can set optional due date when creating/editing decision
- [ ] Due decisions show visual indicator (e.g., red badge)
- [ ] Reminder notification sent 24 hours before due date
- [ ] Overdue decisions appear in priority section

---

### US-14: Voting and Reactions on Decisions
> **As a** workspace member, **I want** to vote on decisions or add reactions **so that** we can gauge consensus quickly.

| Attribute | Details |
|-----------|---------|
| **Business Value** | Faster alignment; democratic decision-making. |
| **Technical Scope** | Vote model, reaction types (👍/👎/❓), tally display. |
| **Priority** | Low |
| **Sprint** | Sprint 4 |

**Acceptance Criteria:**
- [ ] Users can react with 👍, 👎, or ❓
- [ ] Reaction counts are visible to all members
- [ ] Users can change or remove their reaction
- [ ] Decision owner sees summary of reactions

---

## Recommended Implementation Priority

| Order | Story | Rationale |
|-------|-------|-----------|
| 1 | US-8: Team Invitations | Unlocks multi-user value; prerequisite for collaboration features |
| 2 | US-10: Signal Filtering | Essential UX for managing signal volume |
| 3 | US-11: Signal Status | Completes basic signal workflow (inbox-zero) |
| 4 | US-5: Slack Integration | Core aggregator value; most requested integration |
| 5 | US-6: GitHub Integration | High value for developer teams |
| 6 | US-12: Search | Needed once data volume grows |
| 7 | US-9: @Mentions | Builds on US-8; adds collaboration layer |
| 8 | US-13: Due Dates | Decision workflow maturity |
| 9 | US-7: Email Integration | Complex; lower priority than real-time sources |
| 10 | US-14: Voting | Nice-to-have; adds engagement |

---

## Technical Notes

### SQLite Viability
All proposed features work within SQLite's capabilities:
- **Full-text search**: SQLite FTS5 extension
- **Webhook queuing**: Simple job queue table with polling
- **Notifications**: In-app notification table
- **Rate limiting**: In-memory or simple table-based tracking

No external databases or message queues required.

### API Considerations
- Implement pagination for all list endpoints
- Add caching headers for static data (workspaces, members)
- Use webhooks where possible to avoid polling
- Implement exponential backoff for external API retries

### Security Considerations
- Store external OAuth tokens encrypted at rest
- Validate all webhook signatures
- Implement rate limiting on public endpoints
- Scope permissions minimally (e.g., read-only where possible)

---

*Document created: 2026-03-23*
*Next review: After Sprint 2 completion*
