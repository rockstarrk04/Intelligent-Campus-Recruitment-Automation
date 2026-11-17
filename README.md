# Project Pearl

Project Pearl is a full-stack campus placement management platform for students, faculty, and placement administrators. It provides role-based authentication, AI-assisted resume and job-description parsing, eligibility-aware placement drives, applications, applicant tracking, and talent-directory tools.

Pearl is implemented as a pnpm/Turborepo monorepo using React, Express, TypeScript, PostgreSQL, Prisma, and optional Google Gemini integration.

## Table of Contents

- [Project Overview](#project-overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
  - [1. Clone the Repository](#1-clone-the-repository)
  - [2. Install Dependencies](#2-install-dependencies)
  - [3. Environment Variables](#3-environment-variables)
  - [4. Database Setup](#4-database-setup)
  - [5. Seed Development Data](#5-seed-development-data)
  - [6. Start the Application](#6-start-the-application)
- [Application Workflows](#application-workflows)
- [API Modules](#api-modules)
- [Development Scripts](#development-scripts)
- [Docker Setup](#docker-setup)
- [CI](#ci)
- [Workspace Structure](#workspace-structure)
- [Naming Conventions](#naming-conventions)
- [Security Notes](#security-notes)

## Project Overview

Project Pearl is structured as a monorepo with the following components:

- **Frontend (`apps/web`):** A React 19 single-page application built with Vite, TypeScript, React Router, Material UI, Zustand, and TanStack Query.
- **Backend (`apps/api`):** An Express 5 REST API developed with TypeScript and bundled with tsup.
- **Database (`packages/database`):** PostgreSQL persistence managed with Prisma ORM.
- **Shared contracts (`packages/common`):** Shared Zod schemas and TypeScript types used by the API and web client.
- **Configuration (`packages/config`):** Centralized environment-variable validation using Zod.
- **Build system:** Turborepo for coordinating workspace development, builds, and linting.
- **Package manager:** pnpm.
- **AI integration:** Google Gemini `gemini-2.5-flash` for optional resume and job-description extraction.

## Key Features

### Students

- Sign up and log in using a college email address.
- Maintain a placement profile with department, batch year, CGPA, arrears, and academic scores.
- Upload a PDF resume up to 5 MB.
- Extract ATS score, skills, education, projects, certifications, experience, and achievements with Gemini.
- Review and edit parsed resume information before verifying it.
- Browse active placement drives based on eligibility criteria.
- Apply to drives and view application history and status.
- View a student profile dashboard.

### Faculty and administrators

- Create placement drives from a PDF job description or pasted text.
- Extract company, role, compensation, eligibility, department, skills, responsibilities, and perks with Gemini.
- Review and edit parsed job data before publishing.
- Configure CGPA, arrears, department, deadline, and skill requirements.
- List and manage created drives.
- Review applicants and update application statuses.
- Export drive applicants as CSV.
- Search the student talent directory and export search results as CSV.
- View a faculty or administrator profile dashboard.

### Platform features

- Role-based access control for `STUDENT`, `FACULTY`, and `ADMIN`.
- JWT sessions stored in HTTP-only, secure, same-site cookies.
- Password hashing with bcrypt.
- Rate limiting for authentication, AI parsing, and protected API routes.
- Shared API validation with Zod.
- Transactional resume verification and job publishing.
- Case-insensitive skill normalization and de-duplication.
- Optional AI parsing that does not prevent the API from starting when no Gemini key is configured.

## Architecture

```text
┌──────────────────────┐       HTTP + cookies       ┌──────────────────────┐
│ React + Vite web     │ ─────────────────────────▶ │ Express + TypeScript │
│ apps/web             │                            │ apps/api             │
└──────────────────────┘                            └──────────┬───────────┘
                                                               │ Prisma
                                                               ▼
                                                    ┌──────────────────────┐
                                                    │ PostgreSQL            │
                                                    │ packages/database     │
                                                    └──────────────────────┘

                          ┌────────────────────────┐
                          │ Google Gemini (optional)│
                          │ Resume/JD extraction   │
                          └────────────────────────┘
```

The web client communicates with the API using credentialed requests. The API validates the session cookie, applies role checks, validates request data, and uses Prisma for database access.

## Technology Stack

| Area | Technologies |
| --- | --- |
| Frontend | React 19, TypeScript, Vite 7, React Router 7, Material UI 7 |
| State and data fetching | Zustand, TanStack Query |
| Backend | Node.js, Express 5, TypeScript, tsup |
| Database | PostgreSQL 15, Prisma 6 |
| Validation | Zod |
| Authentication | JWT, HTTP-only cookies, bcrypt |
| AI parsing | Google Gemini `gemini-2.5-flash` |
| Monorepo | pnpm 10, Turborepo |
| Local infrastructure | Docker Compose and pgweb |
| Testing | Jest, Testing Library, jsdom |

## Prerequisites

Before beginning, install:

- [Node.js](https://nodejs.org/en/download/) 20 or later
- [pnpm](https://pnpm.io/installation) 10
- [Docker Desktop](https://www.docker.com/get-started) with Docker Compose
- [Git](https://git-scm.com/downloads)
- A Google Gemini API key if AI parsing is required

The CI workflow currently validates the project with Node.js 20.

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/Sabarish0206/pearl.git
cd pearl
```

### 2. Install Dependencies

Install all workspace dependencies using pnpm:

```bash
pnpm install
```

### 3. Environment Variables

Create a root `.env` file from the example:

```bash
cp .env.example .env
```

On Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

At minimum, configure:

```dotenv
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/pearl_local?schema=public"
JWT_SECRET="replace-with-a-random-secret-at-least-32-characters"
CORS_ORIGIN="http://localhost:5173"
VITE_API_URL="http://localhost:3000"
```

Set `GEMINI_API_KEY` to enable resume and job-description parsing. Keep `.env`, database credentials, JWT secrets, and API keys out of version control.

When the API runs directly on the host, use `localhost` in `DATABASE_URL`. When the API runs inside Docker Compose, use the Compose service name `db`.

### 4. Database Setup

Start PostgreSQL:

```bash
docker compose up -d db
```

To also start pgweb, run:

```bash
docker compose up -d db pgweb
```

Pgweb is available at [http://localhost:8080](http://localhost:8080).

Generate Prisma Client and apply the current schema:

```bash
pnpm db:generate
pnpm db:push
```

The database schema contains users, student and faculty profiles, resumes, skills, placement drives, drive skills, and applications.

### 5. Seed Development Data

To load local development data:

```bash
pnpm db:seed
```

The seed script creates:

- Common technical skills
- One faculty account
- Eight student accounts
- Five active placement drives
- Applications for eligible students

All seeded accounts use the development-only password:

```text
Seed@1234
```

Seed users use the `@psnacet.edu.in` email domain. Do not use the seed password outside local development.

### 6. Start the Application

Start the web client and API through Turborepo:

```bash
pnpm dev
```

The default local URLs are:

- Web client: [http://localhost:5173](http://localhost:5173)
- API: [http://localhost:3000](http://localhost:3000)
- API health check: [http://localhost:3000/health](http://localhost:3000/health)

If `PORT` or `VITE_API_URL` is changed, update both values so the web client points to the running API.

## Application Workflows

### Authentication

1. A student or faculty member signs up with an allowed college email.
2. The API hashes the password and creates the user and role-specific profile.
3. Login creates an `auth_token` cookie.
4. Protected routes verify the cookie and enforce the required role.

### Resume parsing and verification

1. A student uploads a PDF resume.
2. Gemini extracts structured resume data and an ATS score.
3. The API stores the result as a draft.
4. The student reviews or edits the extracted data.
5. Commit verifies the resume, updates academic metrics, and synchronizes normalized skills in a Prisma transaction.

### Job-drive creation and publishing

1. A faculty member or administrator uploads a PDF job description or submits raw text.
2. Gemini extracts job and eligibility information.
3. The API stores the result as a draft drive.
4. The creator reviews and edits the draft.
5. Publishing validates the data, upserts required skills, and marks the drive as `ACTIVE`.

### Eligibility and applications

Students can browse active drives when their CGPA, active arrears, and department satisfy the drive criteria. A student with an `OFFERED` application is treated as placed and receives no further eligible drives. The database prevents duplicate applications for the same student and drive.

### Applicant tracking

Faculty and administrators can list applicants, filter by application status, update statuses, and export applicant data as CSV.

## API Modules

All protected endpoints require the session cookie issued by the authentication routes.

| Method | Endpoint | Purpose | Access |
| --- | --- | --- | --- |
| `GET` | `/health` | Check API health | Public |
| `POST` | `/api/auth/signup` | Create a student or faculty account | Public |
| `POST` | `/api/auth/login` | Start a session | Public |
| `GET` | `/api/auth/me` | Get the current user | Authenticated |
| `POST` | `/api/auth/logout` | Clear the session | Public |
| `GET` | `/api/student/resume` | Get resume state | Student |
| `POST` | `/api/student/resume/parse` | Parse a PDF resume into a draft | Student |
| `PUT` | `/api/student/resume/commit` | Verify a reviewed resume draft | Student |
| `GET` | `/api/student/profile` | Get the student profile | Student |
| `GET` | `/api/student/jobs/eligible` | List eligible active drives | Student |
| `POST` | `/api/student/jobs/:id/apply` | Apply to a drive | Student |
| `GET` | `/api/student/applications` | View application history | Student |
| `GET` | `/api/admin/profile` | Get the faculty/admin profile | Faculty/Admin |
| `GET` | `/api/admin/jobs` | List created drives | Faculty/Admin |
| `POST` | `/api/admin/jobs/parse` | Parse a PDF or raw-text job description | Faculty/Admin |
| `GET` | `/api/admin/jobs/:id` | Get a drive | Faculty/Admin |
| `PUT` | `/api/admin/jobs/:id/publish` | Review and publish a draft drive | Faculty/Admin |
| `PATCH` | `/api/admin/jobs/:id/status` | Close, complete, or cancel a drive | Faculty/Admin |
| `GET` | `/api/admin/jobs/:id/applications` | List drive applicants | Faculty/Admin |
| `GET` | `/api/admin/jobs/:id/export` | Export drive applicants as CSV | Faculty/Admin |
| `PATCH` | `/api/admin/applications/:applicationId/status` | Update application status | Faculty/Admin |
| `POST` | `/api/admin/students/search` | Search the talent directory | Faculty/Admin |
| `POST` | `/api/admin/students/export` | Export talent-directory results as CSV | Faculty/Admin |

Resume parsing expects a multipart PDF field named `resume`. Job parsing accepts a multipart PDF field named `jd` or a JSON body containing `rawText`. Uploaded PDFs are limited to 5 MB.

## Development Scripts

The following commands are available from the repository root:

- `pnpm dev`: Start the API and web development servers through Turborepo.
- `pnpm build`: Build all workspaces.
- `pnpm lint`: Run workspace lint tasks.
- `pnpm db:generate`: Generate the Prisma Client.
- `pnpm db:push`: Push the Prisma schema to the configured database.
- `pnpm db:seed`: Load development seed data.
- `pnpm --filter @pearl/api build`: Build only the API.
- `pnpm --filter @pearl/web build`: Typecheck and build only the web client.
- `pnpm --filter @pearl/web test`: Run web tests with Jest and jsdom.
- `pnpm --filter @pearl/api start`: Start the built API.

The root `test` script is not configured as a project-wide test runner yet. Use package-level test commands for the tests currently present.

## Docker Setup

The Compose file defines:

- `db`: PostgreSQL 15 on host port `5432`
- `api`: Production-style API container on host port `80`
- `pgweb`: PostgreSQL browser on host port `8080`

Start the complete stack:

```bash
docker compose up --build
```

Stop the stack while preserving the database volume:

```bash
docker compose down
```

Remove the local database volume explicitly:

```bash
docker compose down -v
```

The API container builds the application, generates Prisma Client, waits for PostgreSQL, and starts the compiled API on container port `3000`.

## CI

GitHub Actions runs for pushes and pull requests targeting `main`. The workflow:

1. Installs dependencies with `pnpm install --frozen-lockfile`.
2. Validates and generates Prisma Client.
3. Typechecks the API and web packages.
4. Builds the API and web packages.

CI uses a syntactically valid PostgreSQL connection string and does not require a live database server.

## Workspace Structure

```text
pearl/
├── apps/
│   ├── web/              (@pearl/web) - React/Vite client application
│   └── api/              (@pearl/api) - Express REST API
├── packages/
│   ├── common/           (@pearl/common) - Shared Zod schemas and TypeScript types
│   ├── config/           (@pearl/config) - Environment-variable schemas
│   └── database/         (@pearl/database) - Prisma schema, client, and seed script
├── .github/workflows/    - GitHub Actions CI
├── turbo.json            - Turborepo task configuration
├── pnpm-workspace.yaml   - pnpm workspace configuration
├── docker-compose.yml    - PostgreSQL, API, and pgweb services
└── .env.example          - Environment-variable template
```

## Naming Conventions

All internal packages follow the `@pearl/` namespace:

- `@pearl/web`: Frontend application
- `@pearl/api`: Backend API
- `@pearl/database`: Database layer
- `@pearl/common`: Shared schemas, types, and utilities
- `@pearl/config`: Environment-variable validation

## Security Notes

- Never commit `.env`, JWT secrets, Gemini API keys, database passwords, or uploaded production files.
- Use a unique random `JWT_SECRET` of at least 32 characters in each environment.
- Restrict `CORS_ORIGIN` to the deployed web application origin.
- Authentication uses bcrypt-hashed passwords and HTTP-only cookies.
- Authentication and expensive AI parsing endpoints are rate-limited.
- AI output is validated before it is stored and is reviewed before resume verification or job publishing.
- Configure persistent, access-controlled storage for uploaded files in production.
