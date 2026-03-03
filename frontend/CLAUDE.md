# Kanban Board — Project Context for Claude Code

## Project Overview

A full-stack Kanban board built as a portfolio project and experimentation ground. The goal is to build a polished, real-world app while learning Next.js and eventually migrating the backend to Spring Boot.

## Tech Stack

- **Frontend:** Next.js 16 (App Router) + React 19 + TypeScript 5 (strict)
- **Styling:** Tailwind CSS v4
- **Auth:** NextAuth.js (Auth.js v5)
- **ORM:** Prisma
- **Database:** PostgreSQL (Docker locally, Supabase or Railway for deployment)
- **Drag & Drop:** dnd-kit
- **Future backend migration:** Spring Boot (Phase 2 of the project)

## Core Features (v1)

- Drag and drop cards between columns
- Create, edit, and delete cards and columns
- User authentication (login/register)
- Full data persistence to PostgreSQL

---

## Commands

```bash
# Dev
npm run dev      # Start development server (http://localhost:3000)
npm run build    # Build for production
npm run start    # Start production server
npm run lint     # Run ESLint

# Database (run from frontend/)
docker compose up -d                              # Start Postgres on port 5432 (run from repo root)
npm run db:migrate -- --name <migration-name>    # Create migration + generate client
npm run db:generate                              # Regenerate Prisma client (after schema changes)
npm run db:push                                  # Push schema changes without a migration file
npm run db:studio                                # Open Prisma Studio at localhost:5555
```

No test framework is configured.

---

## Folder Structure

```
/
├── app/
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── (board)/
│   │   ├── layout.tsx
│   │   └── board/[boardId]/page.tsx
│   ├── api/
│   │   ├── auth/[...nextauth]/route.ts
│   │   ├── columns/route.ts
│   │   └── cards/route.ts
│   └── layout.tsx
├── components/
│   ├── board/
│   │   ├── Board.tsx
│   │   ├── Column.tsx
│   │   └── Card.tsx
│   └── ui/
│       └── (Modal, Button, Input, etc.)
├── lib/
│   ├── prisma.ts      # Prisma client singleton — import { prisma } from '@/lib/prisma'
│   └── auth.ts        # NextAuth config
├── prisma/
│   └── schema.prisma
├── middleware.ts       # Route protection
└── CLAUDE.md
```

**Path alias:** `@/*` maps to the project root (e.g., `@/app/...`, `@/components/...`).

---

## Data Model

### Entities

- **User** — owns many Boards
- **Board** — belongs to User, has many Columns
- **Column** — belongs to Board, has many Cards, has `order: Int`
- **Card** — belongs to Column, has `order: Int`, has title + optional description

### Key design note

Both `Column` and `Card` have an `order` integer field. This is how drag-and-drop position is persisted. Keep this simple for now (integer ordering). Fractional indexing can be added later if needed.

---

## Architecture Decisions

### Server vs Client Components

- Page-level components → **Server Components** (fetch initial data directly from DB, no API call needed)
- Board, Column, Card components → **Client Components** (`"use client"`) because they require interactivity and drag state

### Data Flow

```
Page (Server Component)
  └── fetches board data from DB directly
      └── passes to Board (Client Component)
            └── manages drag state locally via dnd-kit
                └── on drop: optimistically update UI
                    └── fire PATCH /api/cards with new order + columnId
                        └── rollback UI if API call fails
```

### Optimistic Updates

All drag-and-drop interactions should optimistically update the UI before the API call resolves. If the API call fails, roll back to the previous state. This makes the board feel instant and is a key pattern to implement correctly.

### Auth

Use NextAuth.js with credentials provider (email + password) to start. OAuth (e.g. GitHub) can be added later. Protect board routes using `middleware.ts`.

---

## Development Phases

### Phase 1 — Foundation
- Set up Next.js project with TypeScript and Tailwind
- Set up Prisma with PostgreSQL (Docker)
- Define the full schema (User, Board, Column, Card)
- Verify DB connection

### Phase 2 — Auth
- Wire up NextAuth with credentials provider
- Build login and register pages
- Add middleware to protect `/board/*` routes
- Test session handling

### Phase 3 — Board UI (no persistence yet)
- Build Board, Column, and Card components with static/mock data
- Implement drag and drop with dnd-kit
- Get the full interaction working visually before touching the database

### Phase 4 — Connect the API
- Build API routes for columns and cards (GET, POST, PATCH, DELETE)
- Replace mock data with real DB calls
- Implement optimistic updates on drag

### Phase 5 — Polish
- Modals for creating and editing cards
- Loading skeletons and empty states
- Error handling and toast notifications
- Responsive design

### Phase 6 — Spring Boot Migration
Once the Next.js API routes are working, replace them with a Spring Boot backend as a deliberate learning exercise. The Next.js frontend remains; only the data layer changes.

---

## Coding Conventions

- Use TypeScript strictly — no `any` types
- Prefer named exports for components
- Co-locate types with the files that use them unless shared across 3+ files
- API routes should return consistent JSON shapes: `{ data, error }`
- Keep components small and focused — if a component exceeds ~150 lines, consider splitting it
- No Prettier config; formatting is not enforced beyond ESLint rules

---

## Session Tracking

- Keep all project notes, todos, and session context **inside the repo** (not in Claude's internal memory directory) so they can be shared via version control.
- Use `TODO.md` at the **repo root** to track current tasks, in-progress work, and what was last worked on. Update it at the end of each session.
- See `TODO.md` for current status — it reflects actual progress more accurately than this file.
