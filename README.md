# NoteWise: Backend API

REST API for **NoteWise**, an AI-powered study companion. It handles authentication, notes, quizzes, file uploads and all AI features (summary, quiz generation, RAG chat, semantic search, study plan).

This repository is the **backend** (Express + TypeScript). The UI lives in the [frontend repository](#).

- API base URL: `https://your-api.onrender.com/api/v1`
- API docs (Swagger): `https://your-api.onrender.com/api-docs`

---

## Tech Stack

### Core

| Technology | Purpose |
|---|---|
| **Node.js** | Runtime |
| **Express.js** | Web framework |
| **TypeScript** | Type safety |
| **PostgreSQL** | Main database |
| **Prisma** | ORM, migrations, type-safe queries |
| **pgvector** | Vector storage for embeddings (RAG and semantic search) |

### Authentication and Security

| Technology | Purpose |
|---|---|
| **jsonwebtoken (JWT)** | Access and refresh tokens |
| **bcrypt** | Password hashing |
| **cookie-parser** | Read HTTP-only auth cookies |
| **cors** | Allow only the frontend origin, with credentials |
| **Google OAuth 2.0** (`google-auth-library`) | Sign in with Google |
| **helmet** | Security headers |
| **express-rate-limit** | Rate limiting (login and AI routes are stricter) |
| **zod** | Request validation |
| **sanitize-html** | Clean user-written note content (XSS) |

### API Utilities

| Technology | Purpose |
|---|---|
| **http-status** | Readable HTTP status codes |
| **Zod** | Body, query and params validation middleware |
| **pino** | Logging |
| **swagger-ui-express** | API documentation |
| **dotenv** | Environment variables |

### Files, Email and AI

| Technology | Purpose |
|---|---|
| **Cloudinary** | PDF and image storage |
| **multer** | File upload handling (type and size checks) |
| **pdf-parse** | Extract text from uploaded PDFs |
| **nodemailer** | Verification and password-reset emails |
| **Gemini / Groq SDK** | LLM, embeddings and structured output |

### Testing and Deployment

| Technology | Purpose |
|---|---|
| **Vitest** + **Supertest** | API and integration tests |
| **Render / Railway** | Backend hosting (recommended) |
| **Neon / Supabase** | Managed PostgreSQL with pgvector |
| **Vercel** | Frontend hosting |

---

## Features

- **Auth:** register, login, logout, Google sign-in, email verification, password reset, refresh token rotation, profile
- **RBAC:** `STUDENT` and `ADMIN` roles enforced on the server
- **Subjects and Notes:** CRUD, tags, search, filter, sort, pagination, bookmarks
- **PDF upload:** Cloudinary storage and text extraction
- **Flashcards and Quizzes:** CRUD, attempts, server-side scoring
- **Goals and planner**
- **Dashboard summary:** one aggregated endpoint (read only)
- **AI (5 features):**
  1. Summarization
  2. Quiz and flashcard generation (structured output, Zod validated)
  3. Chat with your notes (RAG with pgvector)
  4. Semantic search (embeddings)
  5. Personalized study plan (tool calling, multi-step)
- **Admin:** users, roles, block, content moderation, analytics, activity logs

---

## Folder Structure

```
src/
  modules/
    auth/ users/ subjects/ notes/ flashcards/ quizzes/
    goals/ bookmarks/ dashboard/ ai/ admin/
      (each module)  *.route.ts
                     *.controller.ts
                     *.service.ts
                     *.validation.ts
  middlewares/       auth, role, rateLimit, validate, errorHandler, upload
  utils/             catchAsync, sendResponse, ApiError, jwt, logger
  config/            env, prisma, cloudinary, ai client, google
  app.ts
  server.ts
prisma/
  schema.prisma
  migrations/
```

Request flow:
`Route -> rateLimit -> auth -> role -> Zod validation -> Controller -> Service -> Prisma`

---

## Getting Started

### Prerequisites
- Node.js 20+
- PostgreSQL with the `pgvector` extension (Neon and Supabase support it)

### Setup

```bash
git clone https://github.com/<your-username>/notewise-backend.git
cd notewise-backend
npm install
cp .env.example .env
npx prisma migrate dev
npm run dev
```

Server runs on `http://localhost:5000`.

### Environment Variables

Create `.env` (never commit it):

```env
NODE_ENV=development
PORT=5000
CLIENT_URL=http://localhost:3000

DATABASE_URL=postgresql://user:password@localhost:5432/notewise

JWT_ACCESS_SECRET=change-me
JWT_REFRESH_SECRET=change-me-too
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
BCRYPT_SALT_ROUNDS=10

GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=

GEMINI_API_KEY=

SMTP_HOST=
SMTP_PORT=587
SMTP_USER=
SMTP_PASS=
```

### Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start with hot reload |
| `npm run build` | Compile TypeScript |
| `npm run start` | Run compiled build |
| `npx prisma migrate dev` | Apply migrations (dev) |
| `npx prisma migrate deploy` | Apply migrations (production) |
| `npx prisma studio` | Browse the database |
| `npm run test` | Run Vitest and Supertest |

---

## API Overview

Base path: `/api/v1`. Every response uses one shape:

```json
{ "success": true, "message": "Notes fetched", "data": {}, "meta": { "page": 1, "limit": 10, "total": 42 } }
```

| Module | Endpoints |
|---|---|
| Auth | `POST /auth/register`, `/login`, `/logout`, `/refresh`, `/google`, `/verify-email`, `/forgot-password`, `/reset-password` |
| Users | `GET /users/me`, `PATCH /users/me` |
| Subjects | `GET /subjects`, `POST /subjects`, `PATCH /subjects/:id`, `DELETE /subjects/:id` |
| Notes | `GET /notes`, `POST /notes`, `GET /notes/:id`, `PATCH /notes/:id`, `DELETE /notes/:id`, `POST /notes/upload` |
| Flashcards | `GET/POST /flashcard-sets`, `POST /flashcard-sets/:id/cards` |
| Quizzes | `GET/POST /quizzes`, `POST /quizzes/:id/attempt`, `GET /attempts` |
| Goals | `GET/POST /goals`, `PATCH /goals/:id` |
| Bookmarks | `POST /bookmarks/toggle`, `GET /bookmarks` |
| Dashboard | `GET /dashboard/summary` |
| AI | `POST /ai/summarize`, `/ai/generate-quiz`, `/ai/chat`, `/ai/search`, `/ai/study-plan` |
| Admin | `GET /admin/users`, `PATCH /admin/users/:id/role`, `PATCH /admin/users/:id/block`, `GET /admin/stats` |

Notes list query: `GET /notes?search=heat&subjectId=...&tag=exam&page=1&limit=10&sort=newest`

---

## Authentication Flow

1. **Register:** password hashed with `bcrypt`, verification email sent.
2. **Login:** server sets `accessToken` (15 min) and `refreshToken` (7 days) as **HTTP-only** cookies.
3. **Every request:** `auth` middleware reads the cookie (`cookie-parser`), verifies the JWT, attaches the user.
4. **Refresh:** `/auth/refresh` issues a new pair and invalidates the old refresh token (rotation).
5. **Google OAuth:** frontend sends the Google ID token, the server verifies it, then creates or finds the user and sets cookies.

Cookie settings for production (frontend and API on different domains):

```ts
{ httpOnly: true, secure: true, sameSite: "none" }
```

CORS:

```ts
app.use(cors({ origin: process.env.CLIENT_URL, credentials: true }));
```

---

## Security

- Passwords hashed with bcrypt, never stored or logged in plain text
- JWT in HTTP-only cookies, refresh token rotation
- Role checks and ownership checks (`userId`) on the server for every protected resource
- Rate limiting, with stricter limits on login and AI routes
- Helmet security headers, CORS restricted to `CLIENT_URL`
- Zod validation for body, query and params
- HTML sanitization for note content
- Secrets only in environment variables

---

## AI Architecture

All AI calls run on the server. API keys never reach the browser.

```
Note saved -> split into chunks -> embeddings -> stored in pgvector

Question -> embedding -> nearest chunks (WHERE userId = ?) -> prompt + chunks -> LLM -> answer + sources
```

- **Structured output:** LLM returns JSON, validated with Zod, retried once on failure
- **Tool calling:** study plan fetches weak topics and goals first, then plans
- **Limits:** daily AI usage per user (`AiUsageLog`), timeouts, graceful error messages

---

## Deployment

**Database:** create a Neon or Supabase project, enable `pgvector`, copy `DATABASE_URL`.

**Backend (Render or Railway):**
1. Connect the GitHub repo
2. Build command: `npm install && npx prisma generate && npm run build`
3. Start command: `npx prisma migrate deploy && npm run start`
4. Add all environment variables
5. Set `CLIENT_URL` to the Vercel frontend URL

> Vercel can host Express through serverless functions, but long-running AI and embedding jobs may hit function time limits. Render or Railway is the safer choice for this API.

**Frontend:** deployed on Vercel (see the frontend repository).

---

## Testing

- Unit tests for services and utilities
- Integration tests with Supertest for auth, notes, quizzes and role access
- Tests for validation errors and unauthorized access (`401`, `403`)

---

## Git Workflow

- Feature branches: `feature/auth`, `feature/notes`
- Meaningful commits, for example `feat(notes): validate subject ownership`
- Pull Requests for major features
- `.env` is in `.gitignore`

---

## Author

**Your Name** · [GitHub](https://github.com/your-username) · [LinkedIn](#)