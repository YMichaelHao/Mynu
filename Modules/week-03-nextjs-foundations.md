# Week 3: Next.js Foundations

> **Phase:** 1 — Mynu MVP: Frontend
> **Goal:** Start building Mynu for real. Scaffold the project, learn routing, layouts, and the mental model behind Server vs. Client Components.
> **Time:** ~10 hours
> **Prerequisite:** Week 2 (React refresher)
> **Deliverable:** Mynu project scaffold with working navigation between Home, Recipes, Menus, and Settings pages

---

## What You'll Learn This Week

By the end of Week 3, you should be able to:
- Set up a Next.js project with TypeScript, Tailwind, and the App Router
- Understand file-based routing and how folders map to URLs
- Build layouts that persist across page navigations
- Navigate between pages with the `Link` component
- Explain the difference between Server Components and Client Components — and choose correctly
- Use loading and error states at the route level

This is the week where Mynu stops being exercises and becomes a real project.

---

## Day 1: Project Setup & Mental Model (~2 hours)

### 1.1 — What Is Next.js and Why Use It?

React by itself is a UI library — it renders components. But to build a real app you need routing, server-side rendering, API endpoints, image optimization, and more. Next.js gives you all of that in one framework.

**What Next.js adds on top of React:**

| Concern | Plain React | Next.js |
|---------|------------|---------|
| Routing | You install `react-router`, configure it manually | File-based — create a file, get a route |
| Server rendering | You set up your own SSR pipeline | Built-in — Server Components are the default |
| API endpoints | You build a separate Express server | API Route Handlers live in the same project |
| Code splitting | You configure it manually with `React.lazy` | Automatic per-route |
| Image optimization | You handle it yourself or use a CDN | `next/image` does it automatically |
| Deployment | You figure it out | `vercel deploy` and you're live |

**🧠 Why it matters for Mynu:** You could build Mynu with plain React + a separate backend, but Next.js lets you start with everything in one project. Later (Phase 5), we'll split things out when you learn microservice patterns. For now, one project = faster learning.

### 1.2 — Creating the Mynu Project

```bash
npx create-next-app@latest mynu --typescript --tailwind --app --eslint --src-dir --import-alias "@/*"
cd mynu
npm run dev
```

**What each flag does:**
- `--typescript` → TypeScript support (you already know why)
- `--tailwind` → Tailwind CSS pre-configured (Week 4 deep-dive)
- `--app` → Use the App Router (not the older Pages Router)
- `--eslint` → Linting out of the box
- `--src-dir` → Puts your code in `src/` for cleaner organization
- `--import-alias "@/*"` → Import from `@/components/Button` instead of `../../../components/Button`

After running this, open `http://localhost:3000` — you should see the Next.js welcome page.

### 1.3 — Project Structure

```
mynu/
├── src/
│   ├── app/                    # App Router — this is where routing lives
│   │   ├── layout.tsx          # Root layout (wraps EVERY page)
│   │   ├── page.tsx            # Home page (/)
│   │   ├── globals.css         # Global styles
│   │   ├── recipes/
│   │   │   ├── page.tsx        # Recipes list page (/recipes)
│   │   │   └── [id]/
│   │   │       └── page.tsx    # Recipe detail page (/recipes/abc123)
│   │   ├── menus/
│   │   │   └── page.tsx        # Menus page (/menus)
│   │   └── settings/
│   │       └── page.tsx        # Settings page (/settings)
│   ├── components/             # Reusable UI components
│   │   ├── Navbar.tsx
│   │   └── RecipeCard.tsx      # Your Week 2 card, upgraded
│   ├── lib/                    # Utilities, helpers, configs
│   │   └── types.ts            # Shared TypeScript types
│   └── data/                   # Mock data (until we add a real DB in Week 6)
│       └── recipes.ts
├── public/                     # Static files (images, icons)
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

**🧠 Key insight:** In the App Router, **the file system IS your router.** A file at `src/app/recipes/page.tsx` automatically becomes the `/recipes` URL. No route configuration needed.

---

## Day 1–2: File-Based Routing (~3 hours)

### 2.1 — Pages

Every `page.tsx` file inside `src/app/` becomes a route.

```tsx
// src/app/page.tsx — renders at /
export default function HomePage() {
  return (
    <div>
      <h1>Welcome to Mynu</h1>
      <p>Your recipes, your menus, your way.</p>
    </div>
  );
}
```

```tsx
// src/app/recipes/page.tsx — renders at /recipes
export default function RecipesPage() {
  return (
    <div>
      <h1>My Recipes</h1>
      <p>All your recipes in one place.</p>
    </div>
  );
}
```

```tsx
// src/app/menus/page.tsx — renders at /menus
export default function MenusPage() {
  return (
    <div>
      <h1>My Menus</h1>
      <p>Organize recipes into custom menus.</p>
    </div>
  );
}
```

### 2.2 — Dynamic Routes

What about `/recipes/abc123` where `abc123` is a recipe ID? Use square brackets in the folder name.

```tsx
// src/app/recipes/[id]/page.tsx — renders at /recipes/:id

interface RecipeDetailPageProps {
  params: Promise<{ id: string }>;
}

export default async function RecipeDetailPage({ params }: RecipeDetailPageProps) {
  const { id } = await params;

  return (
    <div>
      <h1>Recipe Detail</h1>
      <p>Showing recipe: {id}</p>
      {/* In Week 8, this will fetch from your real database */}
    </div>
  );
}
```

**🧠 Why `params` is a Promise:** In Next.js App Router, `params` is async because the framework may need to resolve them at request time. This matters for Server Components — you'll understand why shortly.

### 2.3 — Route Groups (Organizing Without Affecting URLs)

Sometimes you want to organize files without creating URL segments. Use parentheses.

```
src/app/
├── (marketing)/          # Doesn't add /marketing to the URL
│   ├── about/
│   │   └── page.tsx      # /about (not /marketing/about)
│   └── pricing/
│       └── page.tsx      # /pricing
├── (app)/                # Doesn't add /app to the URL
│   ├── recipes/
│   │   └── page.tsx      # /recipes
│   └── menus/
│       └── page.tsx      # /menus
```

This is useful later when your marketing pages and app pages need different layouts. For now, just know it exists.

### 2.4 — Navigation with `Link`

Never use `<a>` tags for internal navigation — use `Link` from Next.js. It does client-side navigation (no full page reload) and prefetches the target page.

```tsx
import Link from "next/link";

function Navbar() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/recipes">Recipes</Link>
      <Link href="/menus">Menus</Link>
      <Link href="/settings">Settings</Link>
    </nav>
  );
}
```

**Dynamic links:**

```tsx
import Link from "next/link";

interface RecipeCardProps {
  id: string;
  title: string;
}

function RecipeCard({ id, title }: RecipeCardProps) {
  return (
    <Link href={`/recipes/${id}`}>
      <div className="recipe-card">
        <h3>{title}</h3>
      </div>
    </Link>
  );
}
```

### 2.5 — Programmatic Navigation

Sometimes you need to navigate in response to an event (form submission, button click). Use `useRouter` — but note this requires a Client Component.

```tsx
"use client"; // Required because useRouter is a hook

import { useRouter } from "next/navigation";

function CreateRecipeButton() {
  const router = useRouter();

  const handleCreate = () => {
    // After creating a recipe, redirect to it
    router.push("/recipes/new-recipe-id");
  };

  return <button onClick={handleCreate}>Create Recipe</button>;
}
```

**Other `useRouter` methods:**
- `router.push("/path")` — Navigate forward (adds to history)
- `router.replace("/path")` — Navigate without adding to history
- `router.back()` — Go back
- `router.refresh()` — Re-fetch the current page's data

---

### ✅ Checkpoint 1

Before moving on, you should be able to answer:
1. What file path creates the URL `/recipes/chicken-tikka`?
2. What's the difference between `<Link>` and `<a>` in Next.js?
3. Why does `useRouter` require `"use client"` at the top of the file?
4. Create a `not-found.tsx` file — where would you put it, and when does it render?

---

## Day 2–3: Layouts (~2 hours)

### 3.1 — Root Layout

The root layout wraps every single page in your app. It's where you put things that should always be visible: navbar, footer, global styles.

```tsx
// src/app/layout.tsx
import type { Metadata } from "next";
import { Inter } from "next/font/google";
import "@/app/globals.css";
import Navbar from "@/components/Navbar";

const inter = Inter({ subsets: ["latin"] });

export const metadata: Metadata = {
  title: "Mynu — Your Recipes, Your Menus",
  description: "Store, organize, and share your favorite recipes",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <Navbar />
        <main className="container mx-auto px-4 py-8">
          {children}
        </main>
      </body>
    </html>
  );
}
```

**🧠 Key concept:** `{children}` is the current page. When you navigate from `/recipes` to `/menus`, the layout stays mounted — only `{children}` swaps out. This means your navbar doesn't re-render on navigation. React state in the layout is preserved.

### 3.2 — Nested Layouts

You can add layouts at any level. They nest inside their parent layout.

```tsx
// src/app/recipes/layout.tsx — wraps all /recipes/* pages
export default function RecipesLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <div className="recipes-sidebar">
        <h2>Recipe Categories</h2>
        <ul>
          <li>All Recipes</li>
          <li>Favorites</li>
          <li>Recently Added</li>
        </ul>
      </div>
      <div className="recipes-content">
        {children}
      </div>
    </div>
  );
}
```

**The nesting chain:** When you visit `/recipes/abc123`:
```
RootLayout (navbar, footer)
  └── RecipesLayout (sidebar)
       └── RecipeDetailPage (the actual content)
```

### 3.3 — The `template.tsx` Alternative

A `template.tsx` works like `layout.tsx` but re-mounts on every navigation (state resets). Use it when you need a fresh state per page — like an animation that should replay on every navigation.

```tsx
// src/app/recipes/template.tsx
// This re-mounts (and re-animates) every time you navigate between recipe pages
export default function RecipesTemplate({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="animate-fade-in">
      {children}
    </div>
  );
}
```

**Rule of thumb:** Use `layout.tsx` by default. Use `template.tsx` only when you explicitly want state to reset between navigations.

---

## Day 3–4: Server Components vs. Client Components (~2 hours)

This is the single most important concept in modern Next.js. Get this right and everything else follows.

### 4.1 — The Default: Server Components

In the App Router, **every component is a Server Component by default.** This means it runs on the server, not in the browser.

```tsx
// src/app/recipes/page.tsx — this is a Server Component (no directive needed)

// You can do things that would be impossible in the browser:
import { readFile } from "fs/promises"; // Read files from the server

export default async function RecipesPage() {
  // This runs on the server — the user never sees this code
  // In Week 8, this will be a real database query
  const mockRecipes = [
    { id: "1", title: "Pasta Carbonara", cuisine: "Italian" },
    { id: "2", title: "Chicken Tikka", cuisine: "Indian" },
    { id: "3", title: "Sushi Roll", cuisine: "Japanese" },
  ];

  return (
    <div>
      <h1>My Recipes</h1>
      <div className="recipe-grid">
        {mockRecipes.map((recipe) => (
          <div key={recipe.id}>
            <h3>{recipe.title}</h3>
            <p>{recipe.cuisine}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

**What Server Components CAN do:**
- ✅ Access the database directly
- ✅ Read files from the server
- ✅ Use `async/await` at the component level
- ✅ Keep secrets (API keys, DB credentials) safe — code never reaches the browser
- ✅ Reduce JavaScript sent to the browser (better performance)

**What Server Components CANNOT do:**
- ❌ Use `useState`, `useEffect`, `useRef` (no interactivity)
- ❌ Use `onClick`, `onChange` (no event handlers)
- ❌ Use browser APIs (`window`, `document`, `localStorage`)

### 4.2 — Client Components

When you need interactivity, add `"use client"` at the top of the file.

```tsx
// src/components/RecipeSearch.tsx
"use client"; // This directive makes it a Client Component

import { useState } from "react";

export default function RecipeSearch() {
  const [query, setQuery] = useState("");

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search recipes..."
      />
      {query && <p>Searching for: {query}</p>}
    </div>
  );
}
```

### 4.3 — The Decision Framework

Ask yourself: **Does this component need to respond to user interaction?**

| If the component needs to... | Use |
|-----|-----|
| Fetch data, access DB, keep secrets | Server Component |
| Render static content | Server Component |
| Use `useState`, `useEffect` | Client Component |
| Handle clicks, form input | Client Component |
| Use browser APIs (localStorage, etc.) | Client Component |

### 4.4 — The Composition Pattern (Most Important)

You don't make entire pages client-side. You keep the page as a Server Component and drop Client Components into it only where needed.

```tsx
// src/app/recipes/page.tsx — SERVER Component (fetches data)
import RecipeSearch from "@/components/RecipeSearch"; // CLIENT Component
import RecipeCard from "@/components/RecipeCard";     // SERVER Component (just displays)

export default async function RecipesPage() {
  // This runs on the server — fast, secure
  const recipes = await getRecipes(); // DB call in Week 8

  return (
    <div>
      <h1>My Recipes</h1>
      {/* Client Component island — only this part ships JS to browser */}
      <RecipeSearch />
      {/* Server-rendered list — zero JS sent for this */}
      <div className="recipe-grid">
        {recipes.map((recipe) => (
          <RecipeCard key={recipe.id} recipe={recipe} />
        ))}
      </div>
    </div>
  );
}
```

**🧠 Why this matters for Mynu:** Your recipe list page will have a search bar (Client Component — needs `useState`) surrounded by recipe cards (Server Components — just display data). Only the search bar sends JavaScript to the browser. The recipe cards render on the server as plain HTML. This makes your app fast.

### 4.5 — Common Mistake: Making Everything a Client Component

```tsx
// ❌ BAD: Don't slap "use client" on your page just because one child needs it
"use client"; // Now the ENTIRE page is a Client Component

export default function RecipesPage() {
  // Can't access DB directly anymore
  // All this code ships to the browser
  // ...
}

// ✅ GOOD: Keep the page as Server, extract the interactive part
// page.tsx stays as Server Component
// Only the search bar is "use client"
```

**Rule of thumb:** Push `"use client"` as far down the component tree as possible. The less JavaScript you send to the browser, the faster your app.

---

## Day 4–5: Loading & Error States (~1.5 hours)

### 5.1 — Loading UI

Create a `loading.tsx` file alongside any `page.tsx` to show a loading state while the page data is being fetched.

```tsx
// src/app/recipes/loading.tsx
export default function RecipesLoading() {
  return (
    <div>
      <h1>My Recipes</h1>
      <div className="recipe-grid">
        {/* Skeleton cards — preview of Week 24's polish */}
        {[1, 2, 3, 4, 5, 6].map((i) => (
          <div key={i} className="animate-pulse bg-gray-200 rounded-lg h-48" />
        ))}
      </div>
    </div>
  );
}
```

**How it works:** Next.js automatically wraps your page in a `<Suspense>` boundary using this `loading.tsx` as the fallback. When the page is fetching data (async Server Component), users see the loading UI instead of a blank screen.

### 5.2 — Error Handling

Create an `error.tsx` file to catch errors in a route segment.

```tsx
// src/app/recipes/error.tsx
"use client"; // Error boundaries must be Client Components

import { useEffect } from "react";

export default function RecipesError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log error to monitoring service (Sentry in Week 22)
    console.error(error);
  }, [error]);

  return (
    <div>
      <h2>Something went wrong loading recipes</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

### 5.3 — Not Found

```tsx
// src/app/recipes/[id]/not-found.tsx
import Link from "next/link";

export default function RecipeNotFound() {
  return (
    <div>
      <h2>Recipe not found</h2>
      <p>The recipe you're looking for doesn't exist or has been removed.</p>
      <Link href="/recipes">← Back to recipes</Link>
    </div>
  );
}
```

Trigger it from your page:

```tsx
// src/app/recipes/[id]/page.tsx
import { notFound } from "next/navigation";

export default async function RecipeDetailPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const recipe = await getRecipeById(id);

  if (!recipe) {
    notFound(); // Renders the not-found.tsx component
  }

  return <div>{/* recipe detail */}</div>;
}
```

---

## Day 5: Mock Data Setup (~1 hour)

Until we add a real database in Week 6, we need mock data that mimics what the real API will look like.

### 6.1 — Shared Types

```tsx
// src/lib/types.ts
export interface Ingredient {
  id: string;
  name: string;
  quantity: number;
  unit: string;
}

export interface Recipe {
  id: string;
  title: string;
  description: string;
  cuisine: string;
  servings: number;
  prepTimeMinutes: number;
  cookTimeMinutes: number;
  ingredients: Ingredient[];
  steps: string[];
  tags: string[];
  isPublic: boolean;
  imageUrl?: string;
  createdAt: string;
  userId: string;
}
```

### 6.2 — Mock Data

```tsx
// src/data/recipes.ts
import { Recipe } from "@/lib/types";

export const mockRecipes: Recipe[] = [
  {
    id: "1",
    title: "Pasta Carbonara",
    description: "A classic Roman pasta dish with eggs, cheese, and guanciale.",
    cuisine: "Italian",
    servings: 4,
    prepTimeMinutes: 15,
    cookTimeMinutes: 20,
    ingredients: [
      { id: "1a", name: "Spaghetti", quantity: 400, unit: "g" },
      { id: "1b", name: "Eggs", quantity: 4, unit: "whole" },
      { id: "1c", name: "Pecorino Romano", quantity: 100, unit: "g" },
      { id: "1d", name: "Guanciale", quantity: 200, unit: "g" },
      { id: "1e", name: "Black Pepper", quantity: 2, unit: "tsp" },
    ],
    steps: [
      "Boil pasta in generously salted water",
      "Cut guanciale into small strips and crisp in a pan",
      "Whisk eggs with grated pecorino in a bowl",
      "Drain pasta, reserving 1 cup of pasta water",
      "Toss hot pasta with guanciale, then add egg mixture off heat",
      "Add pasta water a splash at a time until creamy",
      "Season generously with black pepper",
    ],
    tags: ["Italian", "Quick", "Pasta", "Classic"],
    isPublic: true,
    createdAt: "2025-01-15T10:30:00Z",
    userId: "user-1",
  },
  // Add 3-4 more recipes here...
];

// Helper functions that mimic what your real API will do
export function getRecipes(): Recipe[] {
  return mockRecipes;
}

export function getRecipeById(id: string): Recipe | undefined {
  return mockRecipes.find((r) => r.id === id);
}

export function searchRecipes(query: string): Recipe[] {
  const lower = query.toLowerCase();
  return mockRecipes.filter(
    (r) =>
      r.title.toLowerCase().includes(lower) ||
      r.cuisine.toLowerCase().includes(lower) ||
      r.tags.some((t) => t.toLowerCase().includes(lower))
  );
}
```

**🧠 Why mock data matters:** These helper functions (`getRecipes`, `getRecipeById`, `searchRecipes`) have the exact same signatures your real database functions will have in Week 6. When we swap in Prisma, you'll just change the implementation — the components won't change at all.

---

## 🏗️ Week 3 Exercise: Mynu Project Scaffold

**Build the foundation of the Mynu app with the following:**

### Requirements:

1. **Project setup:** `create-next-app` with TypeScript, Tailwind, App Router
2. **Pages:**
   - `/` — Home page with a welcome message and links to recipes/menus
   - `/recipes` — Lists all mock recipes as clickable cards
   - `/recipes/[id]` — Detail page showing a single recipe (title, ingredients, steps)
   - `/menus` — Placeholder page ("Coming in Week 10")
   - `/settings` — Placeholder page
3. **Layout:** Root layout with a navbar that highlights the current page
4. **Navigation:** All links use `<Link>`, clicking a recipe card goes to its detail page
5. **Loading/Error:** Add `loading.tsx` for the recipes page, `error.tsx` for error handling, `not-found.tsx` for missing recipes
6. **Mock data:** At least 5 recipes with full ingredient and step data
7. **Server vs. Client:** The recipe list and detail pages should be Server Components. Only interactive parts (if any) should be Client Components.

### Component structure:

```
src/
├── app/
│   ├── layout.tsx              # Root layout with Navbar
│   ├── page.tsx                # Home page
│   ├── recipes/
│   │   ├── page.tsx            # Recipe list (Server Component)
│   │   ├── loading.tsx         # Loading skeleton
│   │   ├── error.tsx           # Error boundary
│   │   └── [id]/
│   │       ├── page.tsx        # Recipe detail (Server Component)
│   │       └── not-found.tsx   # 404 for missing recipes
│   ├── menus/
│   │   └── page.tsx            # Placeholder
│   └── settings/
│       └── page.tsx            # Placeholder
├── components/
│   ├── Navbar.tsx              # Navigation bar (Client Component — needs usePathname)
│   └── RecipeCard.tsx          # Recipe preview card
├── data/
│   └── recipes.ts              # Mock data + helper functions
└── lib/
    └── types.ts                # Shared TypeScript interfaces
```

### What I'll quiz you on:

- Which components are Server vs. Client, and why?
- What happens if you add `useState` to a Server Component?
- Walk me through what happens when a user clicks a recipe card — what renders, what stays?
- Why is the Navbar a Client Component but the recipe cards are not?
- What would change in your code if we swapped mock data for a real database? (Hint: almost nothing)

---

## 🧪 Stretch Goals (if you finish early)

1. **Active link highlighting:** Make the Navbar highlight the current page using `usePathname()` from `next/navigation`
2. **Recipe categories:** Add a sub-navigation on the recipes page that filters by cuisine (Italian, Mexican, etc.)
3. **Breadcrumbs:** Show breadcrumbs on the detail page: Home > Recipes > Pasta Carbonara
4. **Metadata:** Add proper `<title>` and `<meta>` tags for each page using Next.js `metadata` exports
5. **Catch-all route:** Add a custom 404 page at `src/app/not-found.tsx` for any unknown URL

---

## 📚 Resources

- [Next.js Docs — App Router](https://nextjs.org/docs/app)
- [Next.js Docs — Routing Fundamentals](https://nextjs.org/docs/app/building-your-application/routing)
- [Next.js Docs — Server and Client Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
- [Next.js Docs — Layouts and Templates](https://nextjs.org/docs/app/building-your-application/routing/layouts-and-templates)
- [Next.js Docs — Loading UI and Streaming](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming)
- [Next.js Learn (interactive tutorial)](https://nextjs.org/learn)

---

## ✅ Week 3 Checklist

Before moving to Week 4, make sure you can:

- [ ] Create a Next.js project with TypeScript and Tailwind from scratch
- [ ] Add new pages by creating files in the `app/` directory
- [ ] Build a root layout with a persistent navbar
- [ ] Create dynamic routes with `[id]` folders
- [ ] Navigate between pages using `Link` and `useRouter`
- [ ] Explain Server Components vs. Client Components with real examples
- [ ] Add `loading.tsx`, `error.tsx`, and `not-found.tsx` to a route
- [ ] Use mock data that can be easily swapped for a real database later

---

## 🔜 Next Week Preview

**Week 4: Tailwind + UI Components** — Your Mynu scaffold works, but it looks terrible. Next week we fix that. You'll learn Tailwind utility classes, build a reusable component library (Button, Card, Input, Modal), add responsive design, dark mode, and accessibility basics. The recipe list and detail pages get a real visual design.
