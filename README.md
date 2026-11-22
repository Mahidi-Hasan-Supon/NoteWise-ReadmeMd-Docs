NoteWise: Frontend

NoteWise is an AI-powered study companion. Students store notes and PDFs, then use AI to summarize them, generate quizzes and flashcards, chat with their own notes (RAG), search by meaning, and get a personalized study plan.

This repository is the frontend (Next.js). The API lives in the backend repository.

Live: https://your-app.vercel.app
Backend API: https://your-api.onrender.com
Tech Stack
Core
Technology	Purpose
Next.js (App Router)	Framework: routing, layouts, SSR/CSR
TypeScript	Type safety across the app
Node.js	Runtime for dev server and build
UI and Styling
Technology	Purpose
Tailwind CSS	Utility-first styling, responsive layouts, dark mode
shadcn/ui (Radix UI)	Accessible, reusable UI components (Button, Card, Dialog, Select...)
next-themes	Light / dark mode
lucide-react	Icons
Framer Motion	Animations (flashcard flip, transitions)
Recharts	Dashboard charts
Tiptap	Rich text note editor
Data, Forms and State
Technology	Purpose
ofetch	HTTP client for talking to the backend API
TanStack Query	Server state: fetching, caching, mutations, loading/error states
React Hook Form	Form state and performance
Zod	Schema validation (forms and API responses)
@hookform/resolvers	Connects Zod to React Hook Form
Zustand	Small client-side state (current user, UI state)
Testing and Deployment
Technology	Purpose
Vitest + Testing Library	Unit and component tests
Playwright	End-to-end tests
Vercel	Hosting and CI/CD
Features

Public (Guest)

Landing page with feature cards
Register, Login, Google sign-in, Forgot/Reset password, Email verification
Clicking a feature card sends a guest to /login?redirect=<page> and returns them after login

Student

Dashboard: stats, quiz score chart, weak topics, goal progress, today's plan (view only)
Subjects and Notes (rich editor, tags, search, filter, pagination, bookmark)
PDF upload
Flashcards and Quizzes (manual and AI generated), attempt history
AI: summary, quiz/flashcard generation, chat with notes (RAG), semantic search, study plan
Study goals and planner
Profile and theme settings

Admin

Users and roles, content moderation, analytics, activity monitoring
Folder Structure
src/
  app/
    (public)/        landing, login, register, forgot-password
    (student)/       dashboard, subjects, notes, flashcards, quizzes,
                     chat, search, planner, bookmarks, profile
    (admin)/admin/   users, content, analytics
    layout.tsx
  components/
    ui/              shadcn/ui components
    shared/          Navbar, Sidebar, EmptyState, ErrorState, Loader
    features/        subjects/, notes/, quizzes/, ai/ ...
  hooks/             useSubjects, useNotes, useCreateNote ...
  lib/
    api.ts           ofetch instance (baseURL, credentials, error handling)
    utils.ts
  store/             Zustand stores
  schemas/           Zod schemas (shared form validation)
  types/             TypeScript types
  middleware.ts      route protection
Getting Started
Prerequisites
Node.js 20+
The backend running locally or a deployed API URL
Setup
bash
git clone https://github.com/<your-username>/notewise-frontend.git
cd notewise-frontend
npm install
cp .env.example .env.local
npm run dev

Open http://localhost:3000.

Environment Variables

Create .env.local (never commit it):

env
NEXT_PUBLIC_API_URL=http://localhost:5000/api/v1
NEXT_PUBLIC_GOOGLE_CLIENT_ID=your-google-client-id
Scripts
Command	Description
npm run dev	Start dev server
npm run build	Production build
npm run start	Run production build
npm run lint	Lint code
npm run test	Run Vitest
npm run test:e2e	Run Playwright
API Integration

All requests go through one ofetch instance with credentials: "include" so the HTTP-only auth cookies are sent automatically.

ts
// src/lib/api.ts
import { ofetch } from "ofetch";

export const api = ofetch.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  credentials: "include",
});

Data fetching uses TanStack Query hooks. Every page handles loading, empty, success and error states.

Auth flow

Login: backend sets access and refresh tokens as HTTP-only cookies.
Each request sends cookies. On 401, the client calls /auth/refresh once, then retries.
Protected routes check the session (GET /users/me) and redirect guests to /login?redirect=....
Role-based UI hides Admin pages from students. Real enforcement is on the backend.
Deployment (Vercel)
Push the repo to GitHub.
Import it in Vercel.
Add the environment variables (NEXT_PUBLIC_API_URL, ...).
Deploy. Every push to main redeploys automatically.

Cross-domain cookies: the backend must set SameSite=None; Secure cookies and allow the Vercel URL in CORS with credentials enabled.

Quality
Responsive: desktop, tablet, mobile
Accessible components (Radix primitives)
Light and dark mode
Zod validation on every form
Tests: Vitest (components), Playwright (key user flows)
Git Workflow
Feature branches: feature/notes, feature/auth
Meaningful commits, for example feat(notes): add subject select to note form
Pull Requests for major features
No secrets in the repo (.env* is in .gitignore)
Author

Your Name · GitHub · LinkedIn