# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Chef is an AI app builder that builds full-stack web apps with a built-in database (Convex), zero config auth, file uploads, real-time UIs, and background workflows. This is a fork of bolt.diy stable branch, built on top of Convex for the backend.

## Development Setup

### Initial Setup
```bash
nvm install
nvm use
npm install -g pnpm
pnpm i
echo 'VITE_CONVEX_URL=placeholder' >> .env.local
npx convex dev --once  # Create Convex project
```

### Running Locally
```bash
pnpm run dev        # Start frontend (port 5173)
npx convex dev      # Start Convex backend (separate terminal)
```

**Important**: Access at http://127.0.0.1:5173/ (NOT localhost). Wait a few seconds after starting, then RELOAD THE PAGE to use hot-reloading dev server.

### Common Commands
- `pnpm run lint:fix` - Fix linting and formatting issues
- `pnpm run typecheck` - Run TypeScript type checking
- `pnpm run test` - Run tests (very few exist)
- `pnpm run build` - Build for production
- `pnpm run rebuild-template` - Get directions for iterating on template

## Architecture

### Core Components

**Frontend (`app/`)**
- `components/` - UI components (React)
- `lib/` - Client-side logic for syncing local state with server
- `routes/` - Remix routes (client and server)
- Built with Remix, React, Vite

**Backend (`convex/`)**
- Database that stores chats and user metadata
- Uses Convex for real-time queries and mutations
- File-based routing for functions (e.g., `convex/messages.ts` → `api.messages.*`)

**Agent System (`chef-agent/`)**
- Handles the agentic loop
- Injects system prompts
- Defines tools for LLM
- Calls out to model providers (Anthropic, OpenAI, Google, xAI)

**Template (`template/`)**
- Starting template for all Chef projects
- Use `pnpm run rebuild-template` to iterate on it

**Testing (`test-kitchen/`)**
- Test harness for the Chef agent loop
- Evaluation framework

**CLI (`chefshot/`)**
- CLI interface for interacting with Chef webapp

### Key Technologies
- **Framework**: Remix (React framework) with Vite
- **Database**: Convex (real-time database with queries/mutations/actions)
- **AI SDKs**: Vercel AI SDK with providers for Anthropic, OpenAI, Google, xAI, AWS Bedrock
- **Styling**: Tailwind CSS
- **Auth**: WorkOS via @convex-dev/workos
- **Deployment**: Vercel
- **Container**: WebContainer API for running code in browser

## Convex Development Guidelines

### Function Types
- **Queries**: Read-only database access, automatically cached and subscribed
- **Mutations**: Transactional writes to database
- **Actions**: Can call external APIs, use Node.js runtime with `"use node";` directive
- **Internal functions**: Use `internalQuery`, `internalMutation`, `internalAction` for private functions

### Function Syntax (ALWAYS use new syntax)
```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

export const myQuery = query({
  args: { name: v.string() },
  returns: v.string(),
  handler: async (ctx, args) => {
    // Function body
  },
});
```

### Function References
- Use `api.filename.functionName` for public functions
- Use `internal.filename.functionName` for internal functions
- File-based routing: `convex/messages.ts` → `api.messages.*`

### Schema Location
Always define schema in `convex/schema.ts`.

### Important Patterns
- **Index naming**: Include all fields (e.g., `by_channel` for `["channel"]`)
- **Avoid `.filter()`**: Use indexes with `.withIndex()` instead
- **Return validators**: Always include return type, use `v.null()` if no return
- **TypeScript types**: Use `Id<'tableName'>` from `./_generated/dataModel`
- **System fields**: All docs have `_id` and `_creationTime`

## Project-Specific Rules

### From .cursor/rules/convex_rules.mdc

The extensive Convex guidelines in `.cursor/rules/convex_rules.mdc` cover:
- New function syntax with args/returns validators
- HTTP endpoint patterns in `convex/http.ts`
- Validator types (v.null(), v.array(), v.record(), v.union(), etc.)
- Query/mutation/action guidelines
- Schema design patterns
- File storage patterns
- Cron job scheduling
- Full-text search

**Key takeaway**: Follow the new Convex function syntax religiously and always include validators.

## Testing

- Tests use Vitest
- Very few tests exist, so don't worry about breaking them
- Test your work manually - no e2e tests
- Deploy previews available on Vercel for PR testing

## Branch Structure

- `main` - Default branch for PRs
- `staging` - Deployed to chef-staging.convex.dev
- `release` - Deployed to chef.convex.dev (production)

## Debugging

Global variables available in browser console:
- `chefWebContainer` - Unix-like container where code runs
- `chefMessages` - Raw messages
- `chefParsedMessages` - Parsed messages
- `chefSentryEnabled` - Sentry status
- `chefSetLogLevel("debug"|"info"|"tracing")` - Change logging level
- `chefAssertAdmin()` - Enable admin features (Convex team only)

## Common Patterns

### Convex Function Calling
- `ctx.runQuery(api.file.func, args)` - Call query from query/mutation/action
- `ctx.runMutation(internal.file.func, args)` - Call mutation from mutation/action
- `ctx.runAction(internal.file.func, args)` - Call action from action
- When calling functions in same file, add type annotation to work around TypeScript circularity

### Database Operations
- `ctx.db.get(id)` - Get by ID
- `ctx.db.query("table")` - Query table
- `ctx.db.insert("table", doc)` - Insert document
- `ctx.db.patch(id, partial)` - Partial update
- `ctx.db.replace(id, doc)` - Full replacement
- `ctx.db.delete(id)` - Delete document

### File Storage
- `ctx.storage.getUrl(storageId)` - Get signed URL (returns null if not found)
- Query `_storage` system table for metadata: `ctx.db.system.get(storageId)`
- Store items as `Blob` objects

## Code Style

- Uses Prettier for formatting
- ESLint for linting (separate configs for app and convex)
- TypeScript strict mode
- Use `pnpm run lint:fix` before committing