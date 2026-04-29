# Sentinent

A unified aggregator dashboard designed for organizations to centralize signals, messages, and notifications from multiple platforms into a single, actionable interface. It brings together communication and workflow tools such as Slack, email, GitHub Issues, and Jira, allowing teams to monitor activity, track updates, and respond efficiently without context-switching across applications.

**Goal:** Improve visibility, reduce noise, and help teams stay aligned through a clean, consolidated dashboard experience.

---

## Team

### Front-End Engineers
- Neethika Prathigadapa
- Siddharth R. Nahar

### Back-End Engineers
- Jatin Salve
- Yash Rastogi

---

## Prerequisites

Before getting started, ensure you have the following installed:

| Tool | Version | Check Command |
|------|---------|---------------|
| Go | 1.21+ | `go version` |
| Node.js | 22.x+ | `node --version` |
| npm | 10.x+ | `npm --version` |
| Git | Latest | `git --version` |

---

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/Sentinent-AI/Sentinent.git
cd Sentinent
```

### 2. Initialize Submodules

The backend and frontend are maintained as submodules:

```bash
git submodule update --init --recursive
```

### 3. Start the Backend

```bash
cd backend-go
cp .env.example .env   # (Optional) Create your own .env
# Edit .env with your configuration (see Environment Variables below)
go mod tidy
go run .
```

The backend starts on `http://localhost:8080`.

### 4. Start the Frontend

In a new terminal:

```bash
cd frontend-angular
npm install
ng serve
```

The frontend starts on `http://localhost:4200` and proxies `/api` requests to the backend.

---

## Environment Variables

### Backend (`backend-go/.env`)

Create a `.env` file in the `backend-go/` directory:

```env
# Required
JWT_SECRET=your-super-secret-jwt-key-change-me
CORS_ALLOWED_ORIGINS=http://localhost:4200

# Optional - Application
APP_ENV=development               # Set to "production" for secure cookies
TOKEN_ENCRYPTION_KEY=your-32-plus-character-encryption-key
DATABASE_PATH=./sentinent.db      # Default: ./sentinent.db
API_BASE_URL=http://localhost:8080
FRONTEND_BASE_URL=http://localhost:4200

# Optional - SMTP (for password reset emails)
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USERNAME=your-smtp-username
SMTP_PASSWORD=your-smtp-password
SMTP_FROM_EMAIL=noreply@example.com
SMTP_FROM_NAME=Sentinent

# Optional - Slack Integration
SLACK_CLIENT_ID=your-slack-client-id
SLACK_CLIENT_SECRET=your-slack-client-secret
SLACK_SIGNING_SECRET=your-slack-signing-secret
SLACK_REDIRECT_URI=http://localhost:8080/api/integrations/slack/callback

# Optional - GitHub Integration
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret

# Optional - Gmail Integration
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_REDIRECT_URI=http://localhost:8080/api/integrations/gmail/callback

# Optional - Jira Integration
JIRA_CLIENT_ID=your-jira-client-id
JIRA_CLIENT_SECRET=your-jira-client-secret
```

> **Note:** In development, password reset returns a `reset_url` in the response when SMTP is not configured.

---

## Running Tests

### Backend Tests

```bash
cd backend-go
go test ./... -v
```

### Frontend Unit Tests

```bash
cd frontend-angular
ng test
```

### Frontend E2E Tests (Cypress)

```bash
cd frontend-angular
npx cypress open
# Or run headlessly:
npx cypress run
```

---

## Building for Production

### Backend

```bash
cd backend-go
go build -o sentinent-app .
./sentinent-app
```

### Frontend

```bash
cd frontend-angular
ng build --configuration production
# Output in dist/sentinent-frontend/
```

Serve the frontend build with any static file server:

```bash
npx serve -s dist/sentinent-frontend
```

---

## Project Structure

```
Sentinent/
├── backend-go/              # Go backend (submodule)
│   ├── cmd/                # CLI commands
│   ├── database/           # SQLite database layer
│   ├── handlers/           # HTTP request handlers
│   ├── middleware/          # Auth, CORS, role middleware
│   ├── models/             # Data models
│   ├── services/           # Business logic (integrations, mailer, sync)
│   ├── utils/              # Utilities (encryption, validation, env)
│   ├── main.go             # Entry point
│   └── .env                # Environment configuration
├── frontend-angular/       # Angular frontend (submodule)
│   ├── src/
│   │   ├── app/
│   │   │   ├── components/    # UI components
│   │   │   ├── services/      # API services
│   │   │   ├── guards/        # Route guards
│   │   │   └── models/        # TypeScript interfaces
│   │   └── environments/      # Environment configs
│   ├── cypress/            # E2E tests
│   └── proxy.conf.json     # Dev proxy to backend
├── docs/                   # Project documentation
│   ├── Sprint2.md          # Sprint 2 summary
│   ├── Sprint3.md          # Sprint 3 summary
│   ├── Sprint4.md          # Sprint 4 summary + API docs
│   └── testing/            # Test documentation
└── README.md               # This file
```

---

## API Overview

The backend exposes a RESTful API. Key endpoint groups:

| Group | Base Path | Description |
|-------|-----------|-------------|
| Auth | `/api/signup`, `/api/login`, `/api/logout` | User registration and authentication |
| Profile | `/api/profile` | User profile management |
| Workspaces | `/api/workspaces` | Workspace CRUD and member management |
| Invitations | `/api/invitations` | Workspace invitation flows |
| Signals | `/api/signals` | Unified signal aggregation |
| Integrations | `/api/integrations` | Slack, GitHub, Gmail, Jira connections |
| Webhooks | `/api/webhooks` | Real-time event ingestion |

> For complete API documentation including request/response formats, see **[Sprint4.md](Sprint4.md)**.

---

## Supported Integrations

| Platform | OAuth | Webhooks | Features |
|----------|-------|----------|----------|
| Slack | ✅ | ✅ Real-time | Channels, messages, threaded replies |
| GitHub | ✅ | ✅ | Issues, PRs, comments, state updates |
| Gmail | ✅ | ❌ | Email aggregation (OAuth only) |
| Jira | ✅ | ❌ | Issues, projects, status updates |

---

## Development Workflow

We follow a strict GitHub-centered workflow:

1. **Issues**: Create user stories in [`Sentinent-AI/Sentinent`](https://github.com/Sentinent-AI/Sentinent)
2. **Branches**: `feature/<issue-id>-<desc>` or `fix/<issue-id>-<desc>`
3. **Commits**: `type(scope): description (ref Sentinent-AI/Sentinent#ID)`
4. **PRs**: Open in `backend-go` or `frontend-angular` repos referencing the central issue

**Branch naming examples:**
```bash
git checkout -b feature/12-dashboard-signals
git checkout -b fix/30-slack-sync-cursor
```

---

## Documentation

- **[Sprint4.md](Sprint4.md)** — Sprint 4 summary, test list, and full API documentation
- **[backend-go/README.md](backend-go/README.md)** — Backend setup and environment reference
- **[frontend-angular/README.md](frontend-angular/README.md)** — Frontend CLI and testing reference

---

## License

This project is developed as part of the Software Engineering course. All rights reserved by the Sentinent team.
