NoteWise Backend
AI Study Companion: REST API built with Node.js, Express and TypeScript
Overview
The NoteWise backend handles authentication, data storage, file uploads and every AI call. The frontend never talks to the database or the AI provider directly, so all secrets and security rules live here.
• Live API: https://notewise-api.vercel.app (update after deploy)
• API docs: /api/docs (Swagger)
• Frontend repo: <link to frontend repo>
Tech Stack
Area
Technology
Why
Runtime
Node.js
JavaScript on the server
Framework
Express
Simple, flexible REST API framework
Language
TypeScript
Type safety across the whole backend
Password hashing
bcrypt
Secure one-way password hashing with salt
Auth tokens
jsonwebtoken (JWT)
Short-lived access token and rotating refresh token
Cookies
cookie-parser
Read HTTP-only auth cookies from requests
CORS
cors
Allow only the frontend origin, with credentials
Validation
Zod
Validate body, params and query, and infer types
Status codes
http-status
Readable constants instead of magic numbers (httpStatus.CREATED)
Social login
google-auth-library
Verify the Google ID token sent by the frontend
File storage
Cloudinary + multer
PDF upload with type and size checks
Database
PostgreSQL + Prisma
Relational data, migrations and a typed client
Vector search
pgvector
Embeddings for RAG and semantic search
AI provider
Gemini or Groq
Summaries, quizzes, chat, study plan
Security extras
helmet, express-rate-limit
Security headers and rate limiting
Logging
pino
Fast structured logs
API docs
Swagger (OpenAPI)
Interactive endpoint documentation
Testing
Vitest, Supertest
Unit and API tests
Deployment
Vercel
Serverless deployment with vercel.json
Architecture
Every request follows the same path:
Request
  -> rate limit -> cookie-parser -> auth (JWT) -> role check -> Zod validation
  -> Route -> Controller -> Service -> Prisma / AI service -> Response
The controller only reads the request and sends the response. All business logic lives in the service. Errors go to one central error handler.
Project Structure
src/
  modules/
    auth/        route, controller, service, validation
    users/
    subjects/
    notes/
    flashcards/
    quizzes/
    goals/
    ai/          summarize, quiz, rag, search, planner
    admin/
  middlewares/   auth, role, rateLimit, validate, errorHandler
  utils/         catchAsync, sendResponse, jwt, logger, AppError
  config/        env, prisma, cloudinary, ai client
  app.ts         Express app setup
  server.ts      Server start
prisma/          schema.prisma, migrations
Environment Variables
Create .env (never commit it; commit .env.example only):
NODE_ENV=development
PORT=5000
CLIENT_URL=http://localhost:3000
DATABASE_URL=postgresql://user:pass@host:5432/notewise

JWT_ACCESS_SECRET=change-me
JWT_REFRESH_SECRET=change-me-too
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
BCRYPT_SALT_ROUNDS=10

GOOGLE_CLIENT_ID=your-google-client-id

CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=

AI_API_KEY=
AI_DAILY_LIMIT=20
Validate all variables with Zod at startup so the app fails fast when one is missing.
Authentication and Security
Feature
How it works
Password hashing
bcrypt.hash(password, saltRounds) on register, bcrypt.compare on login
Access token
JWT valid for 15 minutes, sent in an HTTP-only cookie
Refresh token
JWT valid for 7 days, HTTP-only cookie, rotated on every refresh, old one revoked
Cookie options
httpOnly: true, secure: true in production, sameSite set for the deployed domains
Google OAuth
Frontend sends the Google ID token, backend verifies it with google-auth-library, then finds or creates the user and issues its own cookies
Email verification and password reset
Time-limited tokens sent by email
Rate limiting
Strict on login and AI routes
CORS
Only CLIENT_URL, credentials: true
Helmet
Standard security headers
Ownership check
Every query is filtered by userId. Another user's data returns 403 or 404
Role check
Admin routes allow only role === ADMIN
Input sanitization
Zod validation plus HTML sanitizing for note content
Security is enforced here, not only in frontend route guards.
Response Format
Every endpoint returns the same shape, using http-status constants:
{ "success": true, "message": "Note created", "data": { }, "meta": { "page": 1, "limit": 10, "total": 42 } }
Errors:
{ "success": false, "message": "Validation failed", "errors": [ { "path": "title", "message": "Required" } ] }
res.status(httpStatus.CREATED).json({ success: true, message: "Note created", data: note });
API Endpoints (summary)
Base path: /api/v1
Module
Endpoints
Auth
POST /auth/register, /auth/login, /auth/google, /auth/logout, /auth/refresh, /auth/verify-email, /auth/forgot-password, /auth/reset-password
Users
GET /users/me, PATCH /users/me
Subjects
GET /subjects, POST /subjects, PATCH /subjects/:id, DELETE /subjects/:id
Notes
GET /notes (search, tag, subjectId, page, limit, sort), POST /notes, POST /notes/upload, GET, PATCH, DELETE /notes/:id
Flashcards
GET, POST /flashcard-sets, POST /flashcard-sets/:id/cards
Quizzes
GET, POST /quizzes, POST /quizzes/:id/attempt, GET /attempts
Goals
GET, POST /goals, PATCH /goals/:id
Bookmarks
POST /bookmarks/toggle, GET /bookmarks
Dashboard
GET /dashboard/summary
AI
POST /ai/summarize, /ai/generate-quiz, /ai/chat, /ai/search, /ai/study-plan
Admin
GET /admin/users, PATCH /admin/users/:id/role, PATCH /admin/users/:id/block, GET /admin/stats
AI Features
Feature
Technique
Summarization
Prompt engineering with input length limit
Quiz and flashcard generation
Structured JSON output validated with Zod, one retry on invalid output
Chat over notes
RAG with pgvector, answers only from the user's own notes and shows sources
Semantic search
Embedding similarity search
Study plan
Tool calling (weak topics, goals), then structured JSON plan
All AI calls run on the server with a timeout, a daily limit per user (AI_DAILY_LIMIT) and usage logging in AiUsageLog.
Database
PostgreSQL with Prisma migrations. Main models: User, RefreshToken, Subject, Note, NoteChunk (embeddings), Tag, Bookmark, FlashcardSet, Flashcard, Quiz, Question, Attempt, AttemptAnswer, Goal, StudyPlanItem, ChatSession, ChatMessage, ActivityLog, AiUsageLog. Deleting a subject cascades to its notes. Quiz attempts are kept for score history.
Getting Started
git clone <repo-url>
cd notewise-backend
npm install
cp .env.example .env
npx prisma migrate dev
npm run dev
The API runs at http://localhost:5000.
Scripts
Command
Purpose
npm run dev
Start with hot reload
npm run build
Compile TypeScript
npm run start
Run the compiled app
npm run lint
ESLint
npm run test
Vitest and Supertest
npx prisma migrate dev
Create and apply migrations
Deployment (Vercel)
1.  Push the repo to GitHub and import it on Vercel.
2.  Add a vercel.json that routes all requests to the Express app entry file.
3.  Add every variable from the Environment Variables section in Project Settings.
4.  Use a hosted PostgreSQL database with pgvector (Neon or Supabase) and run npx prisma migrate deploy against it.
5.  Set CLIENT_URL to the deployed frontend URL.
6.  Because Vercel runs serverless functions, keep long background work (embedding large notes) short or move it to a job queue, and keep PDF uploads small.
Conventions
• One module per feature: route, controller, service, validation
• No unnecessary any, no secrets in the repo, .env in .gitignore
• Feature branches, meaningful commits and Pull Requests for major features
• Tests for auth, validation, APIs and AI output parsing
Author
Built by <your name> as an individual full-stack project.