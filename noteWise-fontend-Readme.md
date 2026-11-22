NoteWise Frontend
AI Study Companion: the web client built with Next.js and TypeScript
Overview
NoteWise lets students store notes and PDFs, then study with AI: summaries, quizzes, flashcards, a chatbot over their own notes (RAG), semantic search and a personalized study plan. This repository is the frontend. It talks to the NoteWise backend over a REST API and keeps all secrets on the server.
• Live URL: https://notewise.vercel.app (update after deploy)
• Backend repo: <link to backend repo>
Tech Stack
Area
Technology
Why
Framework
Next.js (App Router)
Routing, layouts, route groups for public, student and admin areas
Language
TypeScript
Type safety across components, hooks and API types
Styling
Tailwind CSS
Utility-first, responsive and dark mode with the dark: class
UI components
shadcn/ui (Radix UI)
Accessible Card, Dialog, Select, Tabs, Table and Form primitives
Server state
TanStack Query
Caching, loading and error states, refetch after mutations
API client
ofetch
Small fetch wrapper with base URL, interceptors and typed errors
Forms
React Hook Form
Fast form state handling
Validation
Zod + @hookform/resolvers
One schema for form validation and TypeScript types
Client state
Zustand
Light global state (current user, UI preferences)
Google login
@react-oauth/google
Google sign-in button, returns an ID token for the backend to verify
Theme
next-themes
Light and dark mode
Editor
Tiptap
Rich text note editor
Charts
Recharts
Dashboard charts
Animation
Framer Motion
Flashcard flip, page transitions
Icons and toasts
lucide-react, sonner
Icons, success and error notifications
Testing
Vitest, Playwright
Component tests and end-to-end tests
Deployment
Vercel
Git-based deploys with preview URLs
Features (Student area)
• Auth: register, login, logout, Google login, profile, theme switch
• Subjects and notes with search, tags, filters and pagination
• PDF upload, bookmarks
• Flashcards, quizzes, attempt history
• AI: summary, quiz and flashcard generation, RAG chat, semantic search, study plan
• Study goals and a read-only dashboard with charts
• Admin dashboard: users, roles, content, analytics
Every data page handles loading, empty, error and success states.
Routes
Route
Access
/, /login, /register, /forgot-password, /verify-email
Public
/dashboard, /subjects, /notes, /flashcards, /quizzes, /chat, /search, /planner, /bookmarks, /profile
Student
/admin/*
Admin
A guest who clicks a feature card is sent to /login?redirect=/target and returned to that page after login. Route guards only improve the experience. The real protection lives in the backend.
Project Structure
src/
  app/
    (public)/      landing, login, register
    (student)/     dashboard, subjects, notes, quizzes, chat ...
    (admin)/       admin pages
  components/
    ui/            shadcn components
    shared/        navbar, sidebar, empty state, loaders
    features/      SubjectCard, NoteForm, QuizPlayer ...
  hooks/           useSubjects, useNotes, useCreateNote ...
  lib/             api client (ofetch), utils, constants
  store/           Zustand stores
  schemas/         Zod schemas
  types/           shared TypeScript types
Environment Variables
Create .env.local (never commit it):
NEXT_PUBLIC_API_URL=http://localhost:5000/api/v1
NEXT_PUBLIC_GOOGLE_CLIENT_ID=your-google-client-id
Only values prefixed with NEXT_PUBLIC_ reach the browser. No AI or database keys belong in this repo.
API Communication
All requests go through one ofetch instance in lib/api.ts:
import { ofetch } from "ofetch";

export const api = ofetch.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  credentials: "include", // send the HTTP-only auth cookies
});
• JWT tokens live in HTTP-only cookies set by the backend, so the frontend never reads or stores tokens.
• On a 401, the client calls the refresh endpoint once, then retries the request.
• TanStack Query hooks wrap every API call (useQuery for reads, useMutation for writes) and invalidate the related query keys after a mutation.
Forms
Every form uses React Hook Form with a Zod schema:
const form = useForm<NoteInput>({ resolver: zodResolver(noteSchema) });
The same rules also exist in the backend, so invalid data is rejected on both sides.
Getting Started
git clone <repo-url>
cd notewise-frontend
npm install
cp .env.example .env.local
npm run dev
The app runs at http://localhost:3000. Start the backend first.
Scripts
Command
Purpose
npm run dev
Start the dev server
npm run build
Production build
npm run start
Run the production build
npm run lint
ESLint
npm run test
Vitest
npm run test:e2e
Playwright
Deployment (Vercel)
1.  Push the repo to GitHub.
2.  Import it on Vercel, framework preset: Next.js.
3.  Add NEXT_PUBLIC_API_URL and NEXT_PUBLIC_GOOGLE_CLIENT_ID in Project Settings, Environment Variables.
4.  Deploy. Each push to main redeploys automatically.
5.  Add the Vercel URL to the backend CLIENT_URL and CORS allow list, and to the Google OAuth authorised origins.
Conventions
• Feature branches (feature/notes) and meaningful commit messages
• Reusable components and hooks, no unnecessary any
• Responsive on mobile, tablet and desktop, with light and dark mode
• Accessible markup (labels, focus states, keyboard support)
Author
Built by <your name> as an individual full-stack project.