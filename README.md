# Mynu — Full-Stack Learning Roadmap

> **Learner:** 3rd-year CS student, returning from 8-month QA co-op
> **Time commitment:** ~10 hrs/week
> **Estimated duration:** 24–26 weeks (6 months)
> **Project:** Mynu — a recipe & menu management app

---

## What is Mynu?

**My Menu (Mynu)** is a website/mobile app that allows you to store all of your recipes in one place. It lets you create recipes and customized menus of dishes which link to their recipe. You can share menus and dishes, explore popular dishes, and add them to your own menu.

**Advanced features (later phases):**
- Search dishes in your menu given available ingredients
- Plan weekly meals
- Social media layer — share your menus/dishes publicly, follow other cooks, discover trending recipes

---

## Why This Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Language | **TypeScript** | One language everywhere; type safety catches bugs early; #1 most demanded language on job boards for full-stack roles |
| Frontend | **React + Next.js** | Industry standard, huge ecosystem, Next.js handles routing/SSR so you learn production patterns from day one |
| Styling | **Tailwind CSS** | Utility-first, fast prototyping, widely adopted — avoids "CSS file sprawl" |
| Backend | **Next.js API Routes → then standalone Node/Express** | Start simple (API routes in same project), then graduate to a separate service to learn real microservice patterns |
| Database | **Supabase (PostgreSQL) + Prisma ORM** | Managed Postgres with built-in auth, storage, and realtime — plus a type-safe ORM that generates your types automatically |
| Auth | **Clerk** | Best-in-class DX for auth — handles OAuth, sessions, user management, and pre-built UI components so you ship fast |
| Email | **Resend** | Modern email API — simple, developer-friendly, great for transactional emails (shopping lists, meal plans) |
| AI | **OpenAI API + Anthropic API** | Learn prompt engineering, structured output, RAG, and embeddings by building real features |
| Storage | **Cloudflare R2** | S3-compatible object storage with no egress fees — perfect for recipe images |
| Caching | **Upstash Redis** | Serverless Redis that works seamlessly with Vercel — no server to manage |
| Analytics | **PostHog** | Open-source product analytics — learn to think like a product engineer, track feature usage, run A/B tests |
| Monitoring | **Sentry** | Error tracking and performance monitoring — catch bugs before your users do |
| DevOps | **Docker → GitHub Actions → Vercel** | Containers, CI/CD, cloud deployment — the full pipeline |

---

## Phase 0 — Rust Removal (Weeks 1–2)

*Goal: Shake off the cobwebs. No Mynu code yet — just drills.*

### Week 1: TypeScript & JS Fundamentals
- [ ] Variables, types, interfaces, enums, generics
- [ ] Functions: arrow functions, closures, callbacks
- [ ] Promises, async/await, error handling
- [ ] Array methods: `map`, `filter`, `reduce`, `find`
- [ ] ES modules (`import`/`export`)
- **Exercise:** Build a CLI recipe calculator in TypeScript (reads a JSON recipe, scales ingredients by servings)

### Week 2: React Refresher
- [ ] Components, JSX, props
- [ ] `useState`, `useEffect`, `useRef`
- [ ] Conditional rendering, lists & keys
- [ ] Event handling, controlled inputs
- [ ] Component composition patterns
- **Exercise:** Build a static recipe card component that takes props and renders nicely

---

## Phase 1 — Mynu MVP: Frontend (Weeks 3–5)

*Goal: Build the UI shell of Mynu. No backend yet — use mock data.*

### Week 3: Next.js Foundations
- [ ] Project setup (`create-next-app` with TypeScript + Tailwind)
- [ ] File-based routing (App Router)
- [ ] Layouts, pages, and navigation (`Link`)
- [ ] Client vs. Server Components — when to use each
- **Build:** Mynu project scaffold with pages: Home, Recipes, Menus, Settings

### Week 4: Tailwind + UI Components
- [ ] Tailwind utility classes, responsive design (`sm:`, `md:`, `lg:`)
- [ ] Building a component library: Button, Card, Input, Modal
- [ ] Dark mode support
- [ ] Accessibility basics (semantic HTML, ARIA, keyboard nav)
- **Build:** Recipe list page with search/filter UI, recipe detail page with ingredients & steps

### Week 5: State Management & Forms
- [ ] React Hook Form or native form handling
- [ ] Zod for validation (shared frontend + backend later)
- [ ] Context API for global state (theme, user preferences)
- [ ] Custom hooks: `useRecipes`, `useDebounce`
- **Build:** "Add Recipe" form with validation — title, description, ingredients (dynamic list), steps, tags, prep/cook time

---

## Phase 2 — Mynu MVP: Backend (Weeks 6–9)

*Goal: Real data, real API, real database.*

### Week 6: Database Design & Supabase + Prisma
- [ ] Relational modeling: entities, relationships, normalization
- [ ] Supabase project setup and dashboard overview
- [ ] Prisma schema design for Mynu (Recipe, Ingredient, Menu, User)
- [ ] Connecting Prisma to Supabase PostgreSQL
- [ ] Migrations, seeding
- [ ] Prisma Client: CRUD operations, relations, filtering
- **Build:** Mynu database schema on Supabase + seed script with 10+ sample recipes

### Week 7: API Routes & REST
- [ ] HTTP methods, status codes, REST conventions
- [ ] Next.js Route Handlers (App Router API routes)
- [ ] Request validation with Zod
- [ ] Error handling middleware pattern
- **Build:** Full CRUD API for recipes (`GET`, `POST`, `PUT`, `DELETE`)

### Week 8: Connecting Frontend ↔ Backend
- [ ] `fetch` and data fetching patterns in Next.js
- [ ] Server Components with direct DB access vs. API calls
- [ ] Loading states, error boundaries, optimistic UI
- [ ] SWR or TanStack Query for client-side caching
- **Build:** Wire up recipe list, detail, and create pages to the real API

### Week 9: Authentication
- [ ] Auth concepts: sessions, tokens, OAuth, JWTs
- [ ] Clerk setup: provider config, pre-built UI components (`<SignIn>`, `<UserButton>`)
- [ ] Clerk middleware for protected routes
- [ ] User-specific data (my recipes vs. public recipes)
- [ ] Webhook integration: sync Clerk users to your database
- **Build:** Login/signup flow with Clerk, protect recipe creation behind auth, show user's recipes on profile

---

## Phase 3 — Intermediate Features (Weeks 10–13)

*Goal: Features that teach important patterns.*

### Week 10: Menu Builder
- [ ] Drag-and-drop (dnd-kit) or list-based menu composition
- [ ] Many-to-many relationships (Menu ↔ Recipe)
- [ ] Aggregation queries (total cost, total prep time, combined shopping list)
- [ ] Resend integration: email shopping lists and meal plans
- **Build:** Create/edit menus by adding recipes, auto-generate a shopping list, email it to yourself via Resend

### Week 11: Search & Filtering
- [ ] Full-text search with PostgreSQL (`tsvector`, `tsquery`)
- [ ] Faceted filtering (by tag, cuisine, cook time, dietary restriction)
- [ ] Debounced search input
- [ ] Pagination (cursor-based vs. offset)
- **Build:** Advanced search page with combined filters and paginated results

### Week 12: Image Uploads & Media
- [ ] File upload patterns (presigned URLs vs. direct upload)
- [ ] Cloudflare R2 setup and S3-compatible SDK usage
- [ ] Image optimization and thumbnails
- [ ] Drag-and-drop upload UI
- **Build:** Upload recipe photos to R2, display in cards and detail pages

### Week 13: Testing
- [ ] Unit tests with Vitest (utility functions, hooks)
- [ ] Component tests with React Testing Library
- [ ] API route integration tests
- [ ] E2E tests with Playwright (one critical flow: create recipe → find in search)
- **Build:** Test suite covering recipe CRUD, search, and auth flows

---

## Phase 4 — AI & Agentic Integration (Weeks 14–19)

*Goal: AI features that are genuinely useful, not gimmicks.*

### Week 14: AI Foundations
- [ ] How LLMs work (tokens, context windows, temperature)
- [ ] API integration (OpenAI & Anthropic SDKs)
- [ ] Prompt engineering: system prompts, few-shot examples, structured output
- [ ] Streaming responses
- **Build:** "AI Recipe Suggester" — describe what's in your fridge, get recipe ideas

### Week 15: Structured AI Output
- [ ] JSON mode / tool use for reliable structured data
- [ ] Parsing and validating AI responses with Zod
- [ ] Error handling and retry logic for AI calls
- [ ] Cost tracking and rate limiting
- **Build:** "Import Recipe from URL" — paste a link, AI extracts structured recipe data (title, ingredients, steps)

### Week 16: RAG (Retrieval-Augmented Generation)
- [ ] Embeddings: what they are and why they matter
- [ ] Vector storage (pgvector extension via Supabase — no extra service needed)
- [ ] Semantic search vs. keyword search
- [ ] Building a RAG pipeline: embed recipes → query → augment prompt → generate
- **Build:** "Ask Mynu" — chat with your recipe collection ("What can I make with chicken and rice?")

### Week 17: AI-Powered Features
- [ ] Meal planning with constraints (budget, calories, dietary needs)
- [ ] Recipe scaling intelligence (not just multiplying — "use 2 cans instead of 1.7")
- [ ] Nutritional estimation from ingredients
- **Build:** AI meal planner: input constraints, get a week's menu with shopping list

### Week 18: Agentic AI — Foundations
- [ ] What makes an agent: perception → reasoning → action loops
- [ ] Tool use / function calling deep-dive (OpenAI & Anthropic patterns)
- [ ] Designing tool schemas: when to give the AI a tool vs. hardcode logic
- [ ] Agent memory: conversation history, summarization, context window management
- [ ] Error recovery: what happens when a tool call fails mid-plan
- **Build:** "Mynu Kitchen Agent" — a conversational agent that can search your recipes, check what ingredients you have, and suggest what to cook tonight (multi-tool orchestration)

### Week 19: Agentic AI — Advanced Patterns
- [ ] Multi-step planning: ReAct pattern (Reason + Act), chain-of-thought
- [ ] MCP (Model Context Protocol): connecting agents to external services
- [ ] Guardrails and safety: input validation, output filtering, cost caps
- [ ] Human-in-the-loop: when to pause and ask for confirmation
- [ ] Evaluation: how to measure if your agent is actually good
- **Build:** "Mynu Meal Prep Agent" — give it constraints ("family of 4, $80 budget, no dairy"), it plans the week's meals, generates a shopping list, and can refine based on your feedback across multiple turns

---

## Phase 5 — System Design & DevOps (Weeks 20–23)

*Goal: Production-grade thinking.*

### Week 20: Docker & Containerization
- [ ] Dockerfile for Next.js app
- [ ] Docker Compose for local dev (app + PostgreSQL + Redis)
- [ ] Multi-stage builds for production
- [ ] Environment variable management
- **Build:** Fully containerized Mynu dev environment

### Week 21: CI/CD & GitHub Actions
- [ ] Linting and formatting (ESLint, Prettier) in CI
- [ ] Automated testing on PR
- [ ] Build and deploy pipeline
- [ ] Preview deployments for PRs
- **Build:** GitHub Actions workflow: lint → test → build → deploy

### Week 22: Deployment & Monitoring
- [ ] Deploying to Vercel (seamless Next.js integration)
- [ ] Database already on Supabase — connecting production environment
- [ ] Sentry for error tracking and performance monitoring
- [ ] PostHog for product analytics: track feature usage, funnels, user behavior
- **Build:** Mynu live on the internet with error tracking + analytics dashboards

### Week 23: System Design Concepts
- [ ] Caching strategies (Upstash Redis, HTTP caching, ISR)
- [ ] Rate limiting with Upstash Ratelimit
- [ ] Database indexing and query optimization
- [ ] Scaling patterns: horizontal vs. vertical, read replicas
- [ ] System design practice: "Design a recipe sharing platform at scale"
- **Build:** Add Upstash Redis caching to search, implement rate limiting on API

---

## Phase 6 — Portfolio Polish & Advanced Topics (Weeks 24–26)

*Goal: Ship it and show it off.*

### Week 24: Performance & UX Polish
- [ ] Lighthouse audit and fixes
- [ ] Core Web Vitals optimization
- [ ] Skeleton loaders, transitions, animations (Framer Motion)
- [ ] Mobile responsiveness deep-dive
- [ ] PWA basics (offline recipe viewing)

### Week 25: Advanced Patterns
- [ ] WebSockets or Server-Sent Events (real-time collaborative menu editing)
- [ ] Background jobs (recipe import queue)
- [ ] API versioning
- [ ] Multi-tenancy concepts (household/team recipe collections)

### Week 26: Portfolio & Career Prep
- [ ] README, architecture diagram, demo video
- [ ] Write 2–3 blog posts about what you learned (AI integration, system design decisions)
- [ ] Open-source contribution practice
- [ ] Technical interview prep using Mynu as your "tell me about a project" story

---

## Phase 7 — Mobile App (Future, post-Week 26)

*Goal: Bring Mynu to phones with a native app.*

> **Prerequisite:** Complete the web app first. Your backend (Supabase, API, Clerk) stays the same — you're only building a new frontend.

### Option A: React Native + Expo (Recommended)
- [ ] Expo setup and project scaffold
- [ ] React Native primitives (`View`, `Text`, `ScrollView`, `FlatList`)
- [ ] Navigation with Expo Router
- [ ] Reusing your existing API and auth
- [ ] Camera integration for recipe photo capture
- [ ] App Store / Play Store submission basics
- **Build:** Mynu mobile app — browse recipes, view menus, add recipes from your phone

### Option B: PWA (Already partially covered in Week 24)
- [ ] Service worker for offline recipe viewing
- [ ] App manifest for home screen install
- [ ] Push notifications for shared menus
- No app store needed — works in the browser on any device

*Timeline and scope TBD after the web app is complete.*

---

## Session Format

Each time we meet:

1. **Check-in** — Where did you leave off? Any blockers?
2. **Concept intro** — I explain the next topic, applied to Mynu
3. **You build** — Write the code yourself (AI-assisted is fine, but you explain it back)
4. **Review** — We look at your code together, I push on "why" questions
5. **Homework** — A small task to complete before next session

---

## Rules of Engagement

1. **No copy-paste without understanding.** If you use AI to generate code, be ready to explain every line.
2. **Break things on purpose.** Change a variable, remove a line, see what happens. That's how you learn.
3. **Ask "why" before "how."** Understanding *why* a pattern exists matters more than memorizing syntax.
4. **Ship ugly, then polish.** Working code > perfect code. We iterate.
5. **Document as you go.** Future you will thank present you.

---

## Mynu Feature Backlog (for reference)

| Feature | Phase | Concepts Taught |
|---------|-------|----------------|
| Recipe CRUD | 2 | REST, DB, ORM, validation |
| User auth | 2 | Clerk, OAuth, sessions, middleware, webhooks |
| Menu builder | 3 | Relations, aggregation, drag-and-drop |
| Search & filter | 3 | Full-text search, pagination, debounce |
| Image uploads | 3 | Cloudflare R2, presigned URLs, optimization |
| AI recipe import | 4 | LLM APIs, structured output, parsing |
| Semantic search | 4 | Embeddings, pgvector via Supabase, RAG |
| AI meal planner | 4 | Prompt engineering, constraints, streaming |
| Kitchen agent | 4 | Tool use, agent loops, multi-step reasoning |
| Meal prep agent | 4 | MCP, guardrails, human-in-the-loop, evaluation |
| Dockerized setup | 5 | Containers, compose, env management |
| CI/CD pipeline | 5 | GitHub Actions, automated testing |
| Production deploy | 5 | Vercel, Sentry, PostHog, Upstash |
| Real-time collab | 6 | WebSockets, concurrency |
| Mobile app | 7 | React Native, Expo, cross-platform UI |
