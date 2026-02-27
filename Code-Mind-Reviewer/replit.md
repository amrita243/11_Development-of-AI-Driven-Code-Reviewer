# CodeMind AI - AI-Driven Code Reviewer

## Overview

CodeMind AI is a web application that provides AI-powered code review and analysis. Users can paste or upload code (starting with Python and JavaScript), and the system analyzes it for syntax/logical errors, coding style violations, and optimization opportunities. Results are presented with a quality score, categorized issues, and before/after optimization suggestions. The app targets both students seeking instant feedback and instructors tracking submission history.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Full-Stack Structure
The project follows a monorepo pattern with three top-level directories:
- `client/` — React SPA (frontend)
- `server/` — Express.js API (backend)
- `shared/` — Shared types, schemas, and route definitions used by both client and server

### Frontend (`client/src/`)
- **Framework**: React with TypeScript, bundled by Vite
- **Routing**: Wouter (lightweight client-side router)
- **State/Data Fetching**: TanStack React Query for server state management
- **UI Components**: shadcn/ui (new-york style) built on Radix UI primitives with Tailwind CSS
- **Styling**: Tailwind CSS with CSS variables for theming; strictly dark-mode first aesthetic
- **Animations**: Framer Motion for page transitions and UI animations
- **Charts**: Recharts for score visualization (pie charts for quality scores)
- **Code Display**: react-syntax-highlighter with VS Code Dark+ theme
- **Key Pages**:
  - `/` — Landing page (redirects authenticated users to dashboard)
  - `/dashboard` — Overview with stats, review history
  - `/new-review` — Code submission form (paste code, select language)
  - `/review/:id` — Detailed review results with issues, optimizations, score chart
- **Auth Hook**: `useAuth()` hook queries `/api/auth/user` for session-based auth via Replit Auth

### Backend (`server/`)
- **Framework**: Express.js with TypeScript, run via `tsx`
- **Database**: PostgreSQL with Drizzle ORM
- **Schema**: Defined in `shared/schema.ts` — main table is `reviews` storing code, language, filename, and a JSONB `analysis` column containing score, issues array, and optimizations array
- **Auth**: Replit Auth via OpenID Connect (passport + express-session with connect-pg-simple for session storage)
- **AI Integration**: OpenAI API (via Replit AI Integrations) for code analysis — the server uses the OpenAI client configured with `AI_INTEGRATIONS_OPENAI_API_KEY` and `AI_INTEGRATIONS_OPENAI_BASE_URL`
- **Storage Pattern**: `IStorage` interface in `server/storage.ts` with `DatabaseStorage` implementation for reviews CRUD
- **API Routes**: RESTful endpoints defined in `server/routes.ts`:
  - `GET /api/reviews` — List all reviews
  - `GET /api/reviews/:id` — Get single review
  - `POST /api/reviews` — Submit code for AI analysis, returns review with analysis results
- **Replit Integrations** (`server/replit_integrations/`): Pre-built modules for auth, chat, audio (voice), image generation, and batch processing — these are scaffolded integrations that may not all be actively used by the core code review feature

### Shared Layer (`shared/`)
- `schema.ts` — Drizzle table definitions (reviews, plus re-exported auth and chat models)
- `routes.ts` — API route contracts with Zod schemas for type-safe request/response validation
- `models/auth.ts` — Users and sessions tables (required for Replit Auth)
- `models/chat.ts` — Conversations and messages tables (for chat/voice integrations)

### Database Schema
- **`reviews`**: id, userId (nullable string), fileName, code, language, analysis (JSONB with score, issues, optimizations), createdAt
- **`users`**: id, email, firstName, lastName, profileImageUrl, timestamps (Replit Auth)
- **`sessions`**: sid, sess (JSONB), expire (Replit Auth session store)
- **`conversations`** and **`messages`**: For chat integration features

### Build & Deploy
- **Dev**: `npm run dev` runs `tsx server/index.ts` with Vite dev server middleware for HMR
- **Build**: `npm run build` runs a custom script that builds the client with Vite and bundles the server with esbuild
- **Production**: `npm start` serves the built assets from `dist/`
- **DB Migrations**: `npm run db:push` uses drizzle-kit to push schema changes

## External Dependencies

- **PostgreSQL**: Primary database, required via `DATABASE_URL` environment variable
- **OpenAI API (Replit AI Integrations)**: Used for code analysis; configured via `AI_INTEGRATIONS_OPENAI_API_KEY` and `AI_INTEGRATIONS_OPENAI_BASE_URL` environment variables
- **Replit Auth (OpenID Connect)**: Authentication provider; requires `ISSUER_URL`, `REPL_ID`, and `SESSION_SECRET` environment variables
- **Key npm packages**: drizzle-orm, express, @tanstack/react-query, wouter, framer-motion, recharts, react-syntax-highlighter, shadcn/ui (Radix UI), zod, date-fns