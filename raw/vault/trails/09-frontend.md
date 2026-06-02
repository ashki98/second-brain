# Trail: Frontend

**Objective:** Understand the modern frontend stack from rendering strategy to state management to performance.
**Prerequisites:** Trail 03 — Networking (for HTTP and browser context)
**Review time:** ~20 min
**Nodes:** 4

---

1. [[SSG SSR Edge]]
   > Start with the architectural decision: where does HTML get generated? Client-side rendering (React SPA), Server-Side Rendering (Next.js SSR), Static Site Generation (build-time HTML), or Edge rendering (SSR at the CDN). This choice shapes every performance and SEO trade-off you'll make. Read this first so the rest of the trail has context.

2. [[React Basic Learnings]]
   > React's core model: components as functions of state, one-way data flow, the virtual DOM reconciliation. This note covers hooks (useState, useEffect, useCallback), component composition, and the mental model for thinking in React. The foundation everything else in the frontend builds on.

3. [[React Redux Redux-Saga]]
   > When local component state isn't enough: Redux manages global app state as a single immutable tree mutated by pure reducers. Redux-Saga handles async side effects (API calls, timers) as generator-based middleware. This note is the full state management picture for a large React application.

4. [[FE Download]]
   > Performance: how the browser loads your JavaScript. This note covers code splitting, lazy loading, bundle analysis, and the performance budget. After understanding SSR/SSG (where HTML comes from) and React (how the app runs), this is about minimizing the cost of getting there.
