# Compli-AI Backend
 
> AI-assisted compliance operations platform powering task execution, document validation, and stakeholder collaboration.
 
## Overview
Compli-AI Backend powers the NgNeXT Tech compliance workflow by pairing a robust REST API with AI-driven document analysis. It combines task orchestration, client management, reminders, and comment threads to give CA firms and compliance teams a full audit trail of every filing.
 
The service is built on Express and MongoDB, with role-based access control, OpenAI integration for document scoring, and scheduled email notifications to keep stakeholders on track.
 
## Highlights
- Task lifecycle management covering creation, updates, reassignments, bulk import, status changes, and historied audit trails.
- Document ingestion pipeline that hashes/uploads files, supports CSV/XLSX task imports, and lets teams download or re-validate artifacts on demand.
- AI document analysis using OpenAI (`gpt-4o-mini`) to score relevance, completion, risk, and recommended next steps for each task document.
- Automated reminders and notifications driven by cron jobs that escalate overdue tasks and email stakeholders using configurable SMTP credentials.
- Collaboration layer with threaded task comments, likes, and role-aware visibility for administrators, users, and superadmins.
- Client dossier tracking to tie tasks, filings, and reminders back to specific entities with minimal configuration.
- Built-in Swagger UI, request validation, and Jest test suites to support ongoing development and quality assurance.
 
## Core Technologies
- Node.js + Express server with JWT authentication and Passport RBAC.
- MongoDB via Mongoose models for users, tasks, clients, comments, documents, and task history.
- OpenAI SDK, pdf-parse, mammoth, and adm-zip for multi-format document extraction and analysis.
- Nodemailer, node-cron, and Winston/Morgan for notifications, scheduling, and logging.
- Docker, PM2, ESLint (Airbnb), Prettier, and Jest for deployment, linting, and testing.
 
## Project Layout
```
.
|-- src/
|   |-- app.js                # Express app setup, security middleware, scheduler bootstrap
|   |-- index.js              # Server bootstrap and Mongo connection
|   |-- config/               # Environment config, JWT, logging, passport, roles
|   |-- controllers/          # Auth, task, document, comment, client, LLM orchestration
|   |-- routes/v1/            # Versioned API routes mounted under /v1
|   |-- services/             # Auth, tokens, email, notifications, reminders, LLM analysis, scheduler
|   |-- middlewares/          # Auth guard, rate limiting, validation, upload handling
|   |-- models/               # Mongoose schemas (Task, Doc, TaskHistory, Comment, User, Client, Token)
|   |-- docs/                 # Swagger definition and components
|   |-- utils/                # ApiError, async handler, object utilities
|   `-- uploads/              # Runtime storage for imported tasks and documents
|-- tests/                    # Jest unit and integration suites
|-- docker-compose*.yml       # Compose profiles for dev/test/prod
|-- Dockerfile                # Node + yarn build image
|-- ecosystem.config.json     # PM2 process configuration
|-- API_DOCUMENTATION.md      # High-level API flow references
`-- README.md
```
 
## Getting Started
 
### Prerequisites
- Node.js 18 LTS or newer (>=12 supported, 18+ recommended)
- Yarn (preferred) or npm
- MongoDB instance (local or remote)
- OpenAI API key for document analysis (optional but required to unlock AI features)
- SMTP credentials if you want email notifications to reach users
 
### Installation
1. Clone the repository and install dependencies:
   ```bash
   git clone <repo-url>
   cd Compli-AI-backend
   yarn install
   ```
2. Create `.env` in the project root and define the required environment variables. A minimal example:
   ```bash
   NODE_ENV=development
   PORT=3000
   MONGODB_URL=mongodb://127.0.0.1:27017/compli-ai
   JWT_SECRET=super-secret-key
   JWT_ACCESS_EXPIRATION_MINUTES=60
   JWT_REFRESH_EXPIRATION_DAYS=30
   OPENAI_API_KEY=sk-...
   SMTP_HOST=smtp.example.com
   SMTP_PORT=587
   SMTP_USERNAME=username
   SMTP_PASSWORD=password
   EMAIL_FROM=no-reply@example.com
   ```
   Adjust values to match your local setup or deployment environment.
 
### Running Locally
```bash
yarn dev
```
This launches the API with nodemon, starts the task scheduler, and serves routes under `http://localhost:3000/v1`.
 
### Production Start
```bash
NODE_ENV=production yarn start
```
The production script leverages PM2 via `ecosystem.config.json` for graceful restarts and log aggregation.
 
## Environment Variables
The application expects the following keys in `.env`:
 
| Name | Description |
| ---- | ----------- |
| `NODE_ENV` | `development`, `test`, or `production` |
| `PORT` | Port for the HTTP server |
| `MONGODB_URL` | MongoDB connection string |
| `JWT_SECRET` | Secret used to sign JWT access and refresh tokens |
| `JWT_ACCESS_EXPIRATION_MINUTES` | Access token lifetime (minutes) |
| `JWT_REFRESH_EXPIRATION_DAYS` | Refresh token lifetime (days) |
| `JWT_RESET_PASSWORD_EXPIRATION_MINUTES` | Reset token lifetime (minutes) |
| `JWT_VERIFY_EMAIL_EXPIRATION_MINUTES` | Verify email token lifetime (minutes) |
| `SMTP_HOST` | SMTP host for Nodemailer |
| `SMTP_PORT` | SMTP port |
| `SMTP_USERNAME` | SMTP username |
| `SMTP_PASSWORD` | SMTP password |
| `EMAIL_FROM` | Default `from` email address |
| `OPENAI_API_KEY` | API key used by the LLM document analysis services |
 
Unset SMTP or OpenAI keys will disable their downstream features gracefully, but the corresponding routes may return limited responses.
 
## Available Scripts
- `yarn dev` - start in development with live reload.
- `yarn start` - run the PM2-managed production server.
- `yarn test` / `yarn test:watch` / `yarn coverage` - execute Jest suites.
- `yarn lint`, `yarn lint:fix`, `yarn prettier`, `yarn prettier:fix` - static analysis and formatting.
- `yarn docker:dev`, `yarn docker:prod`, `yarn docker:test` - orchestrate the service with MongoDB via Docker Compose.
- `yarn coverage:coveralls` - generate coverage output suitable for Coveralls CI integration.
 
## Docker and Containerized Runs
Docker images and compose profiles live in the project root. Common flows:
 
```bash
# Build and run with MongoDB in the same network
yarn docker:dev
 
# Production-style deployment (remember to supply environment overrides)
yarn docker:prod
 
# Execute the Jest suite in containers
yarn docker:test
```
 
Volumes are mounted so local changes are reflected inside the container. Populate `.env` or use Compose overrides to inject secrets safely.
 
## Background Jobs and Notifications
The scheduler (`src/services/scheduler.service.js`) starts automatically when the app boots and handles:
- Promoting upcoming tasks to `open` when their scheduled start time arrives (every minute).
- Sending batched reminder emails each hour.
- Escalating overdue tasks with high-priority emails every 30 minutes.
 
Reminder utilities in `taskReminder.service.js` and notification helpers in `taskNotification.service.js` rely on valid SMTP credentials and user email addresses.
 
## AI Document Analysis
Document uploads are processed through `taskUpload.controller.js`, hashed with SHA-256, stored under `src/uploads/docs`, and analyzed via `services/llm.doc.service.js`. The analyzer:
- Extracts text from PDF, DOCX, TXT, and zipped documents.
- Calls OpenAI (`gpt-4o-mini` by default) with task context to produce structured completion metrics.
- Persists results on each `Doc` record (`ai_doc_suggestions`), enabling `/v1/task/analysis/:taskId` and re-analysis endpoints.
 
Without an `OPENAI_API_KEY`, uploads still store files, but responses fall back to safe defaults.
 
## API Surface
All endpoints are namespaced under `/v1`:
 
| Route | Purpose |
| ----- | ------- |
| `/auth` | Registration, login, token refresh, password reset, and email verification. |
| `/users` | CRUD and role-aware user management. |
| `/task` | Task creation, updates, history retrieval, bulk import, document upload, analysis, and comment access. |
| `/docs` | Swagger UI, document status changes, download, debug, and re-analysis. |
| `/clients` | Client onboarding and lookup for linking tasks to entities. |
| `/comment` | Threaded task or document comments, replies, and likes. |
 
Swagger UI is available at `http://localhost:<PORT>/v1/docs`. Additional deep dives live in `API_DOCUMENTATION.md`, `COMPREHENSIVE_API_DOCUMENTATION.md`, and scenario-specific guides in the repository root.
 
## Testing and Quality
Run `yarn test` to execute unit and integration suites under `tests/`. Coverage reports (`yarn coverage`) are Jest-driven. ESLint and Prettier guard the codebase via `yarn lint` and `yarn prettier`. Husky/lint-staged hooks can be added (config provided) to enforce checks pre-commit.
 
## File Storage and Security
Uploaded artifacts are stored beneath `src/uploads` in per-feature subdirectories. Each document record keeps a SHA-256 hash and optional AES-256-GCM metadata to detect tampering. Downloads automatically attempt to decrypt files before returning them to the caller.
 
## Frontend Stubs
The `frontend/` directory holds placeholder React component shells for future dashboard integrations. The backend remains fully usable without them.
 
## License
Distributed under the MIT License. See `LICENSE` for details.
