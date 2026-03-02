# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start development server (http://localhost:3000)
npm run build    # Build for production
npm run start    # Start production server
npm run lint     # Run ESLint
```

No test framework is configured.

## Architecture

This is a **Next.js 16** application using the **App Router** (`/app` directory).

- **Stack:** Next.js 16, React 19, TypeScript 5 (strict), Tailwind CSS v4
- **Routing:** App Router — layouts and pages live in `app/`
- **Styling:** Tailwind CSS v4 via PostCSS; theme customization done inline in `globals.css` using CSS custom properties
- **Linting:** ESLint 9 flat config (`eslint.config.mjs`) extending `eslint-config-next/core-web-vitals` and `eslint-config-next/typescript`

## Path Aliases

`@/*` maps to the project root (e.g., `@/app/...`, `@/components/...`).

## Database

Prisma ORM with PostgreSQL. Start the database with Docker Compose (run from the repo root, one level up from `frontend/`):

```bash
docker compose up -d          # Start Postgres on port 5432
npm run db:migrate -- --name <migration-name>  # Create migration + generate client
npm run db:generate           # Regenerate Prisma client (after schema changes)
npm run db:push               # Push schema changes without a migration file
npm run db:studio             # Open Prisma Studio at localhost:5555
```

- **Singleton:** Import `{ prisma }` from `@/lib/prisma` in Server Components or API routes.
- **Schema:** `prisma/schema.prisma` — models: `Board`, `Column`, `Card`
- **Env:** `DATABASE_URL` in `.env` (gitignored). See `.env.example` for the template.

## Notes

- No Prettier config is present; formatting is not enforced beyond ESLint rules.

## Session Tracking

- Keep all project notes, todos, and session context **inside the repo** (not in Claude's internal memory directory) so they can be shared via version control.
- Use `TODO.md` at the **repo root** to track current tasks, in-progress work, and what was last worked on. Update it at the end of each session.
