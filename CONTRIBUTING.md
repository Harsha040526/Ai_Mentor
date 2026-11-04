# Contributing to AI Mentor

New to the project? Start here.

## Table of contents

- [1. Fork the repository](#1-fork-the-repository)
- [2. Clone your fork](#2-clone-your-fork)
- [3. Follow the detailed environment setup](#3-follow-the-detailed-environment-setup)
- [4. Install dependencies and run locally](#4-install-dependencies-and-run-locally)
  - [First-time contributor checklist](#first-time-contributor-checklist)
  - [Starting services via root commands](#starting-services-via-root-commands)
- [Contributor service map and responsibilities](#contributor-service-map-and-responsibilities)
  - [1. `frontend/` — Learner Web Application](#1-frontend--learner-web-application)
  - [2. `backend/` — User API and AI Gateway](#2-backend--user-api-and-ai-gateway)
  - [3. `frontendAdmin/` — Administrator Web Application](#3-frontendadmin--administrator-web-application)
  - [4. `backendAdmin/` — Administrator API](#4-backendadmin--administrator-api)
  - [5. `ai_service/` — AI Video and Lesson Microservice](#5-ai_service--ai-video-and-lesson-microservice)
  - [Quick "Start Here" service directory and commands table](#quick-start-here-service-directory-and-commands-table)
  - [Root orchestration vs. direct service commands](#root-orchestration-vs-direct-service-commands)
- [How to work on a task](#how-to-work-on-a-task)
- [Rules](#rules)
- [Secret scanning](#secret-scanning)
- [Review and smoke-test scripts](#review-and-smoke-test-scripts)
- [Commit message format](#commit-message-format)
- [Review process](#review-process)
- [Stuck?](#stuck)
- [Further reading](#further-reading)

---

# 1. Fork the repository

First, fork the repository to your own GitHub account:

https://github.com/Deepak3699/Ai_Mentor

Click the **Fork** button in the top-right corner and create a fork under your GitHub account.

# 2. Clone your fork

Clone **your forked repository**, not the original repository:

```bash
git clone https://github.com/<your-github-username>/Ai_Mentor.git
cd Ai_Mentor
git remote add upstream https://github.com/Deepak3699/Ai_Mentor.git
```

Replace `<your-github-username>` with your GitHub username.

# 3. Follow the detailed environment setup

Configure the five `.env` files as described in `initalSetup.md` (Neon database + environment credentials). Never commit any `.env` file to git.

# 4. Install dependencies and run locally

### First-time contributor checklist

Follow this checklist to get from a fresh repository clone to a fully functional local development stack:

- [ ] **1. Verify Prerequisites**:
  - **Node.js**: version 18 or newer (Node.js 20 recommended)
  - **npm**: installed with Node.js
  - **Python**: version 3.10 or newer (Python 3.11 recommended)
  - **PostgreSQL**: local instance or hosted connection string (e.g. Neon)
  - **Redis**: local Redis or Memurai listening on port `6379` (required for asynchronous video generation jobs)
  - **FFmpeg**: installed and discoverable on your system `PATH` (required for AI video rendering)
  - **Git**: installed and configured
- [ ] **2. Clone & Add Upstream**:
  - Clone your fork and link the main repository as `upstream`:
    ```bash
    git clone https://github.com/<your-username>/Ai_Mentor.git
    cd Ai_Mentor
    git remote add upstream https://github.com/Deepak3699/Ai_Mentor.git
    ```
- [ ] **3. Configure Environment Files**:
  - Copy `.env.example` to `.env` in all five runtime locations as documented in `initalSetup.md`:
    - `backend/.env`
    - `frontend/.env`
    - `backendAdmin/.env`
    - `frontendAdmin/.env`
    - `ai_service/backend/.env`
- [ ] **4. Install Dependencies**:
  - From the repository root, run the setup orchestrator:
    ```bash
    npm run setup
    ```
  - _(This installs dependencies across the root and all four Node apps, creates `ai_service/.venv`, installs `requirements.txt`, and validates tool paths)._
- [ ] **5. Start Local Services**:
  - Run `npm run dev` to boot all five services simultaneously, or use `npm run dev:user` / `npm run dev:admin` to launch a targeted subset.
- [ ] **6. Quality Checks Before Submitting**:
  - Before committing or pushing, verify that your code conforms to project standards:
    ```bash
    npm run lint          # Run ESLint across all JavaScript/React packages
    npm run format:check  # Check Prettier code style
    ```

---

### Starting services via root commands

From the repository root:

```bash
npm run setup          # Full installation (Node apps + Python venv)
npm run setup:ai       # AI-only installation
```

Then start the stack using the root orchestration runner:

```bash
npm run dev            # Start or reuse all 5 services with unified dashboard
npm run dev:user       # User API (:5000) + Learner Web (:5173)
npm run dev:admin      # Admin API (:5001) + Admin Web (:5174)
npm run dev:ai         # Python AI Service only (:8000)
```

Target individual services from root with:

- `npm run dev:backend` (User API)
- `npm run dev:frontend` (Learner Web)
- `npm run dev:backend-admin` (Admin API)
- `npm run dev:frontend-admin` (Admin Web)

The root runner checks backend health (`/health`) before starting frontends and safely reuses healthy services already running in other terminal windows. `Ctrl+C` cleanly stops only processes started by the active terminal runner.

### Formatting

Use the project-local Prettier installation before committing frontend, JavaScript, JSON, Markdown, CSS, or HTML changes:

```bash
npm run format
npm run format:check
```

Do not use an unpinned global or temporary Prettier version. Keep functional changes separate from repository-wide formatting changes so pull-request diffs remain reviewable.

---

## Contributor service map and responsibilities

AI Mentor is organized as five cooperating runtime services sharing one PostgreSQL database and coordinated through root development scripts.

```text
Ai_Mentor/
├── frontend/        Learner Web Application     React 19 + Vite 7 + Tailwind 4      :5173
├── backend/         User API & AI Gateway        Express 5 + Sequelize 6 + Postgres  :5000
├── frontendAdmin/   Administrator Web Portal     React 19 + Vite 7 + Tailwind 4      :5174
├── backendAdmin/    Administrator API            Express 5 + Sequelize 6 + Postgres  :5001
└── ai_service/      AI Video & Lesson Service    FastAPI + Gemini + TTS + FFmpeg     :8000
```

`backend` and `backendAdmin` share the **same PostgreSQL database**. The User API (`backend`) acts as the application-facing bridge and security boundary to the Python `ai_service`.

### 1. `frontend/` — Learner Web Application

- **Directory**: `frontend/`
- **Local URL**: [http://localhost:5173](http://localhost:5173)
- **Primary Tech**: React 19, Vite 7, Tailwind CSS 4, React Router DOM v7, Axios, Lucide React, Recharts, i18next
- **Responsibilities & Intent**:
  - The primary student-facing learning portal.
  - User authentication with email/password and Firebase Google OAuth sign-in.
  - Course discovery catalog, course previews, and checkout (Stripe and Razorpay integration).
  - Learning workspace (`/learning/:id`) with video player, subtitle toggling, and real-time transcript synchronization.
  - Learner analytics dashboard (`/analytics`) featuring course progress, completion breakdown donut charts, and task calendar.
  - Community discussion boards, bookmarks, watched-video history, and profile settings.
  - Automatically redirects authenticated administrators to the `frontendAdmin` portal.

### 2. `backend/` — User API and AI Gateway

- **Directory**: `backend/`
- **Local URL**: [http://localhost:5000](http://localhost:5000) (`http://localhost:5000/health` for health status)
- **Primary Tech**: Node.js, Express 5, Sequelize 6, PostgreSQL, JWT, BullMQ, Redis, Zod, Firebase Admin
- **Responsibilities & Intent**:
  - Core REST API serving the learner application.
  - Learner authentication, JWT issuance, password reset workflows, and user profile management.
  - Course catalog queries, module metadata, and student enrollment records.
  - Payment authorization and webhook listeners for Stripe and Razorpay.
  - Background asynchronous video generation queue processing (BullMQ backed by Redis).
  - Secure gateway and proxy between the frontend and the Python `ai_service`.

### 3. `frontendAdmin/` — Administrator Web Application

- **Directory**: `frontendAdmin/`
- **Local URL**: [http://localhost:5174](http://localhost:5174)
- **Primary Tech**: React 19, Vite 7, Tailwind CSS 4, React Router DOM v7, Axios, Lucide React, ExcelJS, jsPDF
- **Responsibilities & Intent**:
  - Dedicated management portal for instructors, admins, and super-administrators.
  - Course authoring, syllabus composition, module ordering, and publishing state management.
  - User account moderation (viewing learners, updating roles, banning, unblocking, and account deletion).
  - Platform enrollment metrics, revenue analytics, and report export tools (CSV, Excel, PDF).
  - Community moderation and reported course resolution.

### 4. `backendAdmin/` — Administrator API

- **Directory**: `backendAdmin/`
- **Local URL**: [http://localhost:5001](http://localhost:5001) (`http://localhost:5001/health` for health status)
- **Primary Tech**: Node.js, Express 5, Sequelize 6, PostgreSQL, JWT, Zod
- **Responsibilities & Intent**:
  - Role-protected API for administrator and super-administrator workflows.
  - Shares the underlying PostgreSQL database with `backend` while isolating administrative privilege checks.
  - Transactional syllabus and course operations with rollback safety.
  - Admin audit trails, course reporting handlers, and administrative notifications.

### 5. `ai_service/` — AI Video and Lesson Microservice

- **Directory**: `ai_service/backend/`
- **Local URL**: [http://127.0.0.1:8000](http://127.0.0.1:8000) (`/` for health, `/docs` for interactive Swagger UI)
- **Primary Tech**: Python 3.10+, FastAPI, Uvicorn, Google Gemini (`google-genai`), Groq, Edge TTS (`edge-tts`), FFmpeg, Cloudinary
- **Responsibilities & Intent**:
  - Generates structured educational lesson scripts using Google Gemini (with Groq fallback).
  - Synthesizes narration audio from scripts using Edge TTS.
  - Composes video slides, narration audio, and animated subtitles via FFmpeg.
  - Delivers generated media to Cloudinary (with local storage fallback) and cleans up temporary local files.
  - Generates course syllabi, quizzes, flashcards, and transcripts.

---

### Quick "Start Here" service directory and commands table

| Service Name      | Directory             | Local URL / Port        | Root Orchestration Command   | Direct Service Command                               | Intent / Primary Function                        |
| :---------------- | :-------------------- | :---------------------- | :--------------------------- | :--------------------------------------------------- | :----------------------------------------------- |
| **Learner Web**   | `frontend/`           | `http://localhost:5173` | `npm run dev:frontend`       | `cd frontend && npm run dev`                         | Learner portal, course player, analytics         |
| **User API**      | `backend/`            | `http://localhost:5000` | `npm run dev:backend`        | `cd backend && npm run dev`                          | Main learner API & AI service gateway            |
| **Admin Web**     | `frontendAdmin/`      | `http://localhost:5174` | `npm run dev:frontend-admin` | `cd frontendAdmin && npm run dev`                    | Admin management portal & reports                |
| **Admin API**     | `backendAdmin/`       | `http://localhost:5001` | `npm run dev:backend-admin`  | `cd backendAdmin && npm run dev`                     | Role-protected admin & course API                |
| **AI Service**    | `ai_service/backend/` | `http://127.0.0.1:8000` | `npm run dev:ai`             | `cd ai_service/backend && python graceful_server.py` | FastAPI video/audio generation & syllabus engine |
| **All Services**  | Root (`.`)            | Ports 5000–8000         | `npm run dev`                | _N/A (use root runner)_                              | Spawns all 5 services with unified dashboard     |
| **Learner Stack** | Root (`.`)            | Ports 5000 & 5173       | `npm run dev:user`           | _N/A (use root runner)_                              | Spawns User API + Learner Web                    |
| **Admin Stack**   | Root (`.`)            | Ports 5001 & 5174       | `npm run dev:admin`          | _N/A (use root runner)_                              | Spawns Admin API + Admin Web                     |

---

### Root orchestration vs. direct service commands

Understanding when to run commands from the repository root versus inside a specific service directory will save you time and prevent common setup issues:

#### Root Orchestration Commands (Recommended for everyday development)

- **Where to run**: Repository root (`Ai_Mentor/`).
- **How they work**: Implemented via `scripts/dev-runner.cjs` and `scripts/install-dependencies.cjs`.
- **Key benefits**:
  - **Coordinated startup**: Frontends wait for their matching backend `/health` endpoint before starting, eliminating connection errors on launch.
  - **Port awareness & reuse**: Detects whether a service is already running on its port in another terminal or background process and reuses it without failing.
  - **Safe cleanup**: `Ctrl+C` cleanly shuts down only the processes started by that specific terminal session.
  - **Unified tooling**:
    - `npm run setup`: Installs dependencies for root, all four Node apps, creates the Python virtual environment (`ai_service/.venv`), and checks system prerequisites (Node, npm, Python, FFmpeg, Redis).
    - `npm run lint`: Runs ESLint across `backend`, `backendAdmin`, `frontend`, and `frontendAdmin` in a single pass.
    - `npm run format`: Runs project-pinned Prettier across all markdown, code, and config files.
    - `npm test`: Runs all service test suites in sequence.

#### Direct Service Commands (For focused, deep debugging)

- **Where to run**: Inside the individual service folder (`cd frontend`, `cd backend`, etc.).
- **When to use**:
  - Inspecting raw, unbuffered server logs without the multi-app dashboard.
  - Running service-specific database migrations or seeds:
    - `backend/`: `npm run seed:courses` (populate sample course catalog).
    - `backendAdmin/`: `npm run seed:superadmin` (seed default super-administrator).
  - Running single-app test runners during active development:
    - `frontend/`: `npm test` or `npx vitest run`.
    - `frontendAdmin/`: `npm test` (Vitest).
    - `backendAdmin/`: `npm run test:postgres-rollback` (transaction rollback integration tests).
    - `ai_service/`: `pytest` (using `ai_service/.venv`).

---

## How to work on a task

```bash
# 1. Always start from the latest code
git checkout main
git pull origin main

# 2. Create your own branch
git checkout -b fix/short-description

# 3. Write code, run it locally, test it

# 4. Run the linter, otherwise CI will fail
npm run lint                  # from the repo root — checks all four Node/React services
npm run lint:backend-admin    # Backend Admin only

# 5. Commit
git add .
git commit -m "fix: admin panel pointing to wrong port"

# 6. Push
git push origin fix/short-description

# 7. Open a pull request on GitHub
```

---

## Rules

### Do

- One pull request does one thing — keep it under roughly 400 changed lines
- Run the service locally before you push
- `npm run lint` must pass
- Attach screenshots for any UI change
- Use the commit format `type: description` (see below)
- Try on your own for 30 minutes before asking for help

### Do not

- Push directly to `main`
- Commit a `.env` file — ever
- Put API keys or passwords in the code
- Merge someone else's pull request without being asked
- Leave `console.log` statements behind
- Bundle five unrelated changes into one pull request

---

## Secret scanning

CI job **Check for committed secrets** does two things:

1. Fails if any file named `.env` is tracked by git.
2. Runs [gitleaks](https://github.com/gitleaks/gitleaks) over the full commit history and file content, looking for API keys, tokens, private keys, webhook URLs and passwords inside database connection strings, whatever the file is called. Findings are redacted in the logs.

Run it locally before you push (install gitleaks first):

```bash
gitleaks git . --config .gitleaks.toml --redact --no-banner
scripts/test-secret-scan.sh        # self-test with fake credentials
```

### If the scan fails

- **It is a real secret:** rotate it at the provider first. Removing it from the code is not enough, because it stays in git history.
- **It is a false positive:** keep the exception as narrow as possible.
  1. Run the scan with `--report-format json --report-path report.json` and copy the `Fingerprint` of the finding.
  2. Add it to `.gitleaksignore` with a comment line above it saying why it is safe.
  3. Only for a whole test fixture file, add its exact path to `[allowlist]` in `.gitleaks.toml` instead.
  4. Open the pull request. Changes to these files need Team Lead review (see `.github/CODEOWNERS`). Never allowlist something you have not confirmed is fake.

Use obvious placeholders in docs and examples (`your_password`, `xxxxx`), which the scanner ignores.

---

## Review and smoke-test scripts

```bash
scripts/review-pr.sh 123          # check a PR (needs gh; run npm run setup first)
scripts/smoke-test.sh             # is the local stack alive? (--quick = services only)
scripts/test-review-scripts.sh    # self-test of both scripts (also runs in CI)
```

Both scripts list exactly which checks passed, failed, or were skipped. Neither runs the automated test suites (CI does), so a pass means "the listed checks passed", not "this change is correct".

`review-pr.sh` stops before switching branches if you have uncommitted or untracked files (ignored files such as `node_modules` are fine), and it restores your branch when it finishes or is interrupted. It fetches the base branch first and prints the exact base commit: `upstream/main` if you have an `upstream` remote, otherwise `origin/main`. Use `BASE_REF=<remote>/<branch>` to pick another base, or `NO_FETCH=1` to skip the fetch (the result then carries a warning).

`smoke-test.sh` treats only HTTP 400, 401, 403 and 404 as a safe rejection in the transcript checks. A 5xx fails, and no response, a redirect or any other status warns.

| Exit code | Meaning                                                                                     |
| --------- | ------------------------------------------------------------------------------------------- |
| `0`       | Every required check ran and passed, no warnings                                            |
| `1`       | A check failed, or the script could not run (bad input, local work in the way, `curl` gone) |
| `2`       | Warnings, or a required check was skipped (for example `node_modules` missing)              |

Exit code `2` is never a safe-to-merge result. Install what is missing and run the script again.

---

## Commit message format

```
type: short description (lowercase, present tense)
```

| Type       | When to use it                |
| ---------- | ----------------------------- |
| `feat`     | A new feature                 |
| `fix`      | A bug fix                     |
| `docs`     | Documentation only            |
| `style`    | Formatting, no logic change   |
| `refactor` | Better code, same behaviour   |
| `test`     | Adding tests                  |
| `chore`    | Dependencies or configuration |

Examples:

```
feat: add certificate download button
fix: point admin frontend to port 5001
docs: add backendAdmin setup to README
chore: bump express to v5
```

---

## Review process

1. Open your pull request and fill in the template
2. Automated checks run (lint and build, about three minutes)
3. Red cross? Click "Details", read the error, fix it, push again
4. Green checks plus one approval — then merge
5. Delete your branch after merging

---

## Stuck?

1. Try on your own first — give it 30 minutes and read the error carefully
2. Search Google or Stack Overflow with the exact error message
3. Still stuck? Ask in the team group, and include:
   - What you already tried
   - The full error message (screenshot or pasted text)
   - Which service and which file

Asking for help is fine. Just do not ask with "it is not working" — nobody can
help you from that alone.

---

## Further reading

- `initalSetup.md` — environment setup
- `backend/BACKEND_DOCUMENTATION.md` — the most detailed backend reference
- `ai_service/README.md` — the AI video pipeline

---

Questions? Ask the team lead. If you spot a mistake in these rules, send a pull
request, this file should improve too.
