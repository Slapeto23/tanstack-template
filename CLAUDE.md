# CLAUDE.md

This file provides guidance for AI assistants working with this codebase.

## Project Overview

A full-stack AI chat application built with TanStack React Start, powered by Anthropic's Claude API. Deployed on Netlify with optional Convex database persistence and Sentry error monitoring.

## Tech Stack

- **Framework**: TanStack React Start (v1.114+) with Vinxi
- **UI**: React 19, Tailwind CSS v4, Lucide icons
- **Routing**: TanStack Router (file-based, auto code-splitting)
- **State**: TanStack Store (`@tanstack/store` + `@tanstack/react-store`)
- **AI**: Anthropic SDK (`@anthropic-ai/sdk`) with streaming responses
- **Database**: Convex (optional, graceful fallback to local state)
- **Monitoring**: Sentry (optional, conditional on env vars)
- **Build**: Vite 6, TypeScript 5.7, PostCSS
- **Deployment**: Netlify (serverless preset)

## Project Structure

```
src/
  routes/           # TanStack Router file-based routes
    __root.tsx      # Root layout (HTML shell, Convex provider, devtools)
    index.tsx       # Main chat page (/)
  components/       # Reusable UI components (stateless, prop-driven)
  store/            # TanStack Store state management
    store.ts        # Store definition, actions, selectors
    hooks.ts        # Custom hooks (useAppState, useConversations)
  utils/
    ai.ts           # Server function for Claude API streaming
  api.ts            # TanStack React Start API handler
  client.tsx        # Client entry point
  ssr.tsx           # Server-side rendering entry
  router.tsx        # Router configuration
  convex.tsx        # Convex provider setup
  sentry.ts         # Sentry error monitoring setup
  styles.css        # Global styles and animations
  routeTree.gen.ts  # Auto-generated route tree (DO NOT EDIT)
convex/             # Convex backend (schema, API functions)
app.config.ts       # TanStack React Start config (server preset: netlify)
vite.config.js      # Vite plugins (TanStack Router, React, Tailwind, Sentry)
```

## Commands

- `npm run dev` - Start development server
- `npm run build` - Production build
- `npm run serve` - Preview production build
- `npm start` - Start dev server (alias)

There is no test runner, linter, or formatter configured in this project.

## Environment Variables

Defined in `.env` (see `.env.example`):

- `ANTHROPIC_API_KEY` - Required. Server-side only (no `VITE_` prefix for security).
- `VITE_CONVEX_URL` - Optional. Enables Convex database persistence.
- `VITE_SENTRY_DSN` - Optional. Enables Sentry error monitoring on client.
- `SENTRY_AUTH_TOKEN` - Optional. Enables Sentry source map uploads during build.

Never commit `.env` files. The `VITE_` prefix exposes variables to the client bundle; only use it for values safe to expose.

## Architecture Patterns

### Server Functions (AI Streaming)

AI responses use `createServerFn` from TanStack React Start (`src/utils/ai.ts`). The server function streams Claude responses as NDJSON (`application/x-ndjson`), with each line being a JSON object containing `content_block_delta` events. The client reads the stream with `getReader()` and renders text character-by-character.

### State Management

TanStack Store with a clear pattern in `src/store/store.ts`:
- **Store**: Single `Store<State>` instance with typed state
- **Actions**: Exported object of functions that call `store.setState()`
- **Selectors**: Exported object of pure functions extracting derived state
- **Hooks**: `useAppState()` for UI state, `useConversations()` for data with Convex integration

### Convex Integration (Optional)

`src/store/hooks.ts` implements a dual-layer pattern:
1. Local state updates immediately for UI responsiveness
2. Convex mutations fire asynchronously for persistence
3. `isConvexAvailable` flag (`Boolean(import.meta.env.VITE_CONVEX_URL)`) gates all Convex calls
4. Everything works without Convex; it falls back to in-memory state

### Routing

File-based routing in `src/routes/`. The route tree is auto-generated in `src/routeTree.gen.ts` -- never edit this file manually. New routes are created by adding files to `src/routes/`. Auto code-splitting is enabled in `vite.config.js`.

### Components

Components in `src/components/` are stateless and prop-driven. Business logic and data fetching belong in hooks (`src/store/hooks.ts`) or the route component (`src/routes/index.tsx`). Barrel exports via `index.ts` files.

### Styling

Tailwind CSS v4 with utility classes. Global styles and keyframe animations in `src/styles.css`. No component-level CSS modules.

## TypeScript Configuration

- Strict mode enabled (`strict: true`)
- `noUnusedLocals` and `noUnusedParameters` enforced
- Target: ES2022, Module: ESNext, JSX: react-jsx
- Bundler module resolution with `allowImportingTsExtensions`

## Key Conventions

- Use `uuid` (v4) for generating client-side IDs
- Markdown rendering uses `react-markdown` with `rehype-highlight`, `rehype-raw`, and `rehype-sanitize`
- Optional integrations (Convex, Sentry) check for env vars at runtime and degrade gracefully
- Server-side secrets use plain env var names; client-safe values use `VITE_` prefix
- The Anthropic client does not set a custom `baseURL` -- Netlify AI Gateway intercepts requests to `api.anthropic.com` automatically
