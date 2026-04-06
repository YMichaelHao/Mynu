# Week 2: React Refresher

> **Phase:** 0 — Rust Removal
> **Goal:** Rebuild your React muscle memory. Still no Mynu codebase — just focused drills.
> **Time:** ~10 hours
> **Prerequisite:** Week 1 (TypeScript fundamentals)
> **Deliverable:** An interactive recipe card component with state, effects, and composition

---

## What You'll Learn This Week

By the end of Week 2, you should be able to:
- Build React components with proper TypeScript typing
- Manage state with `useState` and side effects with `useEffect`
- Use `useRef` for DOM access and persisted values
- Handle events, controlled inputs, and conditional rendering
- Compose components together (parent/child, props drilling, children pattern)
- Render lists with proper keys

You already touched React and Vue before your co-op — this week is about making it click again, but with TypeScript this time. Every example uses the recipe types you built in Week 1.

---

## Day 1: Components, JSX & Props (~2 hours)

### 1.1 — What Is a React Component?

A component is just a function that returns JSX (HTML-like syntax). TypeScript adds prop typing.

```tsx
// The simplest component
function Greeting() {
  return <h1>Welcome to Mynu!</h1>;
}
```

### 1.2 — Props with TypeScript

This is where Week 1 pays off — you already know how to write interfaces.

```tsx
// Define the shape of your props
interface RecipeCardProps {
  title: string;
  cuisine: string;
  prepTimeMinutes: number;
  cookTimeMinutes: number;
  isPublic: boolean;
  imageUrl?: string; // optional
}

// Use the interface to type your component
function RecipeCard({ title, cuisine, prepTimeMinutes, cookTimeMinutes, isPublic, imageUrl }: RecipeCardProps) {
  const totalTime = prepTimeMinutes + cookTimeMinutes;

  return (
    <div className="recipe-card">
      {imageUrl && <img src={imageUrl} alt={title} />}
      <h2>{title}</h2>
      <p>{cuisine} · {totalTime} min</p>
      {isPublic && <span className="badge">Public</span>}
    </div>
  );
}

// Using it
<RecipeCard
  title="Pasta Carbonara"
  cuisine="Italian"
  prepTimeMinutes={15}
  cookTimeMinutes={20}
  isPublic={true}
/>
```

**🧠 Why it matters for Mynu:** Every page in Mynu is built from components like this. The recipe list page is just a loop of `RecipeCard` components. The detail page is a bigger composition of smaller components (ingredient list, step list, nutrition info).

### 1.3 — The `children` Pattern

```tsx
// A reusable Card wrapper — doesn't care what's inside
interface CardProps {
  children: React.ReactNode;
  className?: string;
}

function Card({ children, className = "" }: CardProps) {
  return (
    <div className={`rounded-lg shadow-md p-4 ${className}`}>
      {children}
    </div>
  );
}

// Now you can wrap anything in a Card
<Card>
  <h2>Pasta Carbonara</h2>
  <p>A classic Roman pasta dish.</p>
</Card>

<Card className="bg-yellow-50">
  <p>⚠️ This recipe contains eggs and dairy.</p>
</Card>
```

**🧠 Why it matters for Mynu:** In Week 4, you'll build a component library (Button, Card, Modal, Input). The `children` pattern is how you make them flexible and reusable.

### 1.4 — Conditional Rendering

```tsx
interface RecipeStatusProps {
  isPublic: boolean;
  isFavorited: boolean;
  rating: number | null;
}

function RecipeStatus({ isPublic, isFavorited, rating }: RecipeStatusProps) {
  return (
    <div>
      {/* && for simple show/hide */}
      {isPublic && <span className="badge">Public</span>}

      {/* Ternary for either/or */}
      <button>{isFavorited ? "♥ Favorited" : "♡ Add to Favorites"}</button>

      {/* Handle null/undefined */}
      {rating !== null ? (
        <span>⭐ {rating.toFixed(1)}</span>
      ) : (
        <span>No ratings yet</span>
      )}
    </div>
  );
}
```

### 1.5 — Rendering Lists with Keys

```tsx
interface Ingredient {
  name: string;
  quantity: number;
  unit: string;
}

interface IngredientListProps {
  ingredients: Ingredient[];
  servings: number;
}

function IngredientList({ ingredients, servings }: IngredientListProps) {
  return (
    <ul>
      {ingredients.map((ingredient, index) => (
        // ❌ BAD: using index as key
        // <li key={index}>

        // ✅ GOOD: using a stable, unique identifier
        <li key={`${ingredient.name}-${ingredient.unit}`}>
          {ingredient.quantity} {ingredient.unit} {ingredient.name}
        </li>
      ))}
    </ul>
  );
}
```

**🧠 Why `key` matters:** React uses keys to track which items changed, were added, or removed. Using array index as a key causes bugs when items are reordered, deleted, or inserted. Always use a stable unique value — in Mynu, that'll be the database `id` field.

---

### ✅ Checkpoint 1

Before moving on, you should be able to answer:
1. What's the difference between `React.ReactNode` and `React.ReactElement`?
2. Why do we destructure props in the function signature instead of using `props.title`?
3. What happens if you render a list without `key` props? What about duplicate keys?
4. Build a `StepList` component that takes an array of step strings and renders them as an ordered list

---

## Day 2–3: State with `useState` (~3 hours)

### 2.1 — Basic State

```tsx
import { useState } from "react";

function ServingsAdjuster() {
  // useState returns [currentValue, setterFunction]
  // TypeScript infers the type from the initial value
  const [servings, setServings] = useState(4);

  return (
    <div>
      <button onClick={() => setServings(servings - 1)} disabled={servings <= 1}>
        −
      </button>
      <span>{servings} servings</span>
      <button onClick={() => setServings(servings + 1)}>
        +
      </button>
    </div>
  );
}
```

### 2.2 — State with Complex Types

```tsx
import { useState } from "react";

interface Ingredient {
  name: string;
  quantity: number;
  unit: string;
}

function IngredientManager() {
  // Explicitly type the state when TypeScript can't infer it
  const [ingredients, setIngredients] = useState<Ingredient[]>([]);
  const [newName, setNewName] = useState("");
  const [newQuantity, setNewQuantity] = useState("");
  const [newUnit, setNewUnit] = useState("g");

  const addIngredient = () => {
    if (!newName || !newQuantity) return;

    const ingredient: Ingredient = {
      name: newName,
      quantity: parseFloat(newQuantity),
      unit: newUnit,
    };

    // IMPORTANT: Never mutate state directly!
    // ❌ ingredients.push(ingredient)
    // ✅ Create a new array with spread
    setIngredients([...ingredients, ingredient]);

    // Reset form
    setNewName("");
    setNewQuantity("");
  };

  const removeIngredient = (indexToRemove: number) => {
    setIngredients(ingredients.filter((_, index) => index !== indexToRemove));
  };

  return (
    <div>
      <div className="input-row">
        <input
          value={newName}
          onChange={(e) => setNewName(e.target.value)}
          placeholder="Ingredient name"
        />
        <input
          type="number"
          value={newQuantity}
          onChange={(e) => setNewQuantity(e.target.value)}
          placeholder="Qty"
        />
        <select value={newUnit} onChange={(e) => setNewUnit(e.target.value)}>
          <option value="g">g</option>
          <option value="ml">ml</option>
          <option value="cups">cups</option>
          <option value="tsp">tsp</option>
          <option value="tbsp">tbsp</option>
          <option value="whole">whole</option>
        </select>
        <button onClick={addIngredient}>Add</button>
      </div>

      <ul>
        {ingredients.map((ing, i) => (
          <li key={`${ing.name}-${i}`}>
            {ing.quantity} {ing.unit} {ing.name}
            <button onClick={() => removeIngredient(i)}>✕</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**🧠 Why it matters for Mynu:** This is basically the "Add Recipe" form you'll build in Week 5 — a dynamic list of ingredients that users can add to and remove from. Understanding immutable state updates here prevents bugs later.

### 2.3 — Controlled Inputs

In React, form inputs are "controlled" — React owns the value, not the DOM.

```tsx
import { useState } from "react";

function RecipeSearchBar() {
  const [query, setQuery] = useState("");
  const [cuisineFilter, setCuisineFilter] = useState("all");

  const handleSearch = () => {
    console.log(`Searching for "${query}" in ${cuisineFilter} recipes`);
    // In Week 8, this will call your real API
  };

  return (
    <div>
      <input
        type="text"
        value={query}           // React controls the value
        onChange={(e) => setQuery(e.target.value)}  // Update state on every keystroke
        placeholder="Search recipes..."
        onKeyDown={(e) => e.key === "Enter" && handleSearch()}
      />

      <select
        value={cuisineFilter}
        onChange={(e) => setCuisineFilter(e.target.value)}
      >
        <option value="all">All Cuisines</option>
        <option value="italian">Italian</option>
        <option value="mexican">Mexican</option>
        <option value="japanese">Japanese</option>
        <option value="indian">Indian</option>
      </select>

      <button onClick={handleSearch}>Search</button>

      {query && (
        <p className="text-sm text-gray-500">
          Showing results for "{query}"
          {cuisineFilter !== "all" && ` in ${cuisineFilter}`}
        </p>
      )}
    </div>
  );
}
```

### 2.4 — State Update Gotcha: Stale Closures

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  const incrementThree = () => {
    // ❌ BUG: All three see the SAME count value (closure!)
    // setCount(count + 1);
    // setCount(count + 1);
    // setCount(count + 1);
    // Result: count goes from 0 to 1, not 0 to 3

    // ✅ FIX: Use the functional form — prev is always fresh
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    // Result: count goes from 0 to 3
  };

  return <button onClick={incrementThree}>Count: {count}</button>;
}
```

**🧠 Why it matters for Mynu:** This closure gotcha bites everyone at least once. When you're updating a shopping list or toggling multiple favorites in rapid succession, functional updates prevent data loss.

---

## Day 3–4: Effects with `useEffect` (~2.5 hours)

### 3.1 — What `useEffect` Is For

`useEffect` runs side effects — things that happen *because* the component rendered, but aren't part of the render itself. Think: fetching data, setting up timers, updating document title.

```tsx
import { useState, useEffect } from "react";

interface Recipe {
  id: string;
  title: string;
  description: string;
}

function RecipeDetail({ recipeId }: { recipeId: string }) {
  const [recipe, setRecipe] = useState<Recipe | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // This runs AFTER the component renders
    const fetchRecipe = async () => {
      try {
        setLoading(true);
        setError(null);
        const response = await fetch(`/api/recipes/${recipeId}`);
        if (!response.ok) throw new Error("Recipe not found");
        const data = await response.json();
        setRecipe(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : "Unknown error");
      } finally {
        setLoading(false);
      }
    };

    fetchRecipe();
  }, [recipeId]); // <-- dependency array: re-run when recipeId changes

  if (loading) return <p>Loading recipe...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!recipe) return <p>No recipe found.</p>;

  return (
    <div>
      <h1>{recipe.title}</h1>
      <p>{recipe.description}</p>
    </div>
  );
}
```

### 3.2 — The Dependency Array

This is where most `useEffect` bugs come from. Understand this well.

```tsx
import { useEffect } from "react";

// Runs on EVERY render (usually wrong)
useEffect(() => {
  console.log("I run every single render");
});

// Runs ONCE on mount (empty array = no dependencies)
useEffect(() => {
  console.log("I run only when the component first appears");
}, []);

// Runs when specific values change
useEffect(() => {
  console.log(`Search query changed to: ${query}`);
}, [query]);

// Runs when ANY of the dependencies change
useEffect(() => {
  console.log("Filters changed — refetch!");
}, [query, cuisineFilter, sortBy]);
```

### 3.3 — Cleanup Functions

Some effects need to be "undone" when the component unmounts or before the effect re-runs.

```tsx
import { useState, useEffect } from "react";

function CookingTimer({ durationMinutes }: { durationMinutes: number }) {
  const [secondsLeft, setSecondsLeft] = useState(durationMinutes * 60);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning || secondsLeft <= 0) return;

    const intervalId = setInterval(() => {
      setSecondsLeft((prev) => prev - 1);
    }, 1000);

    // Cleanup: clear the interval when component unmounts
    // or when isRunning/secondsLeft changes
    return () => clearInterval(intervalId);
  }, [isRunning, secondsLeft]);

  const minutes = Math.floor(secondsLeft / 60);
  const seconds = secondsLeft % 60;

  return (
    <div>
      <p>{minutes}:{seconds.toString().padStart(2, "0")}</p>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? "Pause" : "Start"}
      </button>
      <button onClick={() => setSecondsLeft(durationMinutes * 60)}>
        Reset
      </button>
    </div>
  );
}
```

**🧠 Why it matters for Mynu:** Cleanup prevents memory leaks. If a user navigates away from a recipe detail page while data is still loading, the cleanup ensures you don't try to update state on an unmounted component.

### 3.4 — Common `useEffect` Mistake: Infinite Loops

```tsx
// ❌ INFINITE LOOP — object/array in dependency
useEffect(() => {
  // This creates a NEW array every render
  const filtered = recipes.filter((r) => r.isPublic);
  setFiltered(filtered);
}, [recipes.filter((r) => r.isPublic)]); // New reference every time → effect re-runs → infinite loop

// ✅ Depend on the actual value, compute inside the effect
useEffect(() => {
  const filtered = recipes.filter((r) => r.isPublic);
  setFiltered(filtered);
}, [recipes]); // Only re-runs when recipes reference changes

// ✅ Even better — don't use useEffect at all for derived state!
// Just compute it during render:
const filteredRecipes = recipes.filter((r) => r.isPublic);
```

**Rule of thumb:** If you're using `useEffect` to compute something from existing state/props, you probably don't need `useEffect`. Just compute it directly in the component body.

---

## Day 4: `useRef` (~1.5 hours)

### 4.1 — DOM Access

```tsx
import { useRef, useEffect } from "react";

function RecipeSearchBar() {
  const inputRef = useRef<HTMLInputElement>(null);

  // Auto-focus the search input when the component mounts
  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return (
    <input
      ref={inputRef}
      type="text"
      placeholder="Search recipes..."
    />
  );
}
```

### 4.2 — Persisting Values Without Re-renders

`useRef` stores a value that persists across renders but does NOT trigger a re-render when changed. Think of it as a "box" that holds a mutable value.

```tsx
import { useState, useRef, useEffect } from "react";

function RecipeSearch() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState<string[]>([]);
  const renderCount = useRef(0);
  const previousQuery = useRef("");

  // Track render count without causing re-renders
  renderCount.current += 1;

  useEffect(() => {
    // Compare with previous value
    if (query === previousQuery.current) return;
    previousQuery.current = query;

    // Simulate search
    console.log(`Searching for: ${query} (render #${renderCount.current})`);
  }, [query]);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <p>Component has rendered {renderCount.current} times</p>
    </div>
  );
}
```

**🧠 `useState` vs `useRef`:**
| | `useState` | `useRef` |
|---|---|---|
| Triggers re-render on change? | ✅ Yes | ❌ No |
| Persists across renders? | ✅ Yes | ✅ Yes |
| Use for | UI-visible values | DOM refs, timers, previous values, render counts |

---

## Day 5: Component Composition Patterns (~2 hours)

### 5.1 — Lifting State Up

When two components need to share state, move the state to their closest common parent.

```tsx
import { useState } from "react";

interface Ingredient {
  name: string;
  quantity: number;
  unit: string;
}

// Parent owns the state
function RecipeBuilder() {
  const [servings, setServings] = useState(4);
  const [ingredients, setIngredients] = useState<Ingredient[]>([
    { name: "Spaghetti", quantity: 400, unit: "g" },
    { name: "Eggs", quantity: 4, unit: "whole" },
    { name: "Pecorino", quantity: 100, unit: "g" },
  ]);

  // Scale ingredients based on servings
  const scaledIngredients = ingredients.map((ing) => ({
    ...ing,
    quantity: (ing.quantity / 4) * servings,
  }));

  return (
    <div>
      <h1>Recipe Builder</h1>
      {/* Both children react to the same servings state */}
      <ServingsControl servings={servings} onServingsChange={setServings} />
      <IngredientDisplay ingredients={scaledIngredients} />
    </div>
  );
}

// Child 1: Controls the servings
interface ServingsControlProps {
  servings: number;
  onServingsChange: (newServings: number) => void;
}

function ServingsControl({ servings, onServingsChange }: ServingsControlProps) {
  return (
    <div>
      <button onClick={() => onServingsChange(Math.max(1, servings - 1))}>−</button>
      <span>{servings} servings</span>
      <button onClick={() => onServingsChange(servings + 1)}>+</button>
    </div>
  );
}

// Child 2: Displays the (scaled) ingredients
interface IngredientDisplayProps {
  ingredients: Ingredient[];
}

function IngredientDisplay({ ingredients }: IngredientDisplayProps) {
  return (
    <ul>
      {ingredients.map((ing) => (
        <li key={ing.name}>
          {ing.quantity} {ing.unit} — {ing.name}
        </li>
      ))}
    </ul>
  );
}
```

**🧠 Why it matters for Mynu:** The recipe detail page in Mynu will work exactly like this — a servings adjuster at the top that scales the ingredient list below it. Same pattern, real feature.

### 5.2 — Callback Props (Child → Parent Communication)

```tsx
interface RecipeFilterProps {
  onFilterChange: (filters: { cuisine: string; maxTime: number }) => void;
}

function RecipeFilter({ onFilterChange }: RecipeFilterProps) {
  const [cuisine, setCuisine] = useState("all");
  const [maxTime, setMaxTime] = useState(60);

  // Notify parent whenever filters change
  const handleApply = () => {
    onFilterChange({ cuisine, maxTime });
  };

  return (
    <div>
      <select value={cuisine} onChange={(e) => setCuisine(e.target.value)}>
        <option value="all">All</option>
        <option value="italian">Italian</option>
        <option value="mexican">Mexican</option>
      </select>

      <input
        type="range"
        min={10}
        max={120}
        value={maxTime}
        onChange={(e) => setMaxTime(Number(e.target.value))}
      />
      <span>{maxTime} min max</span>

      <button onClick={handleApply}>Apply Filters</button>
    </div>
  );
}

// Parent uses the filter values
function RecipeListPage() {
  const [filters, setFilters] = useState({ cuisine: "all", maxTime: 60 });

  return (
    <div>
      <RecipeFilter onFilterChange={setFilters} />
      <p>Showing {filters.cuisine} recipes under {filters.maxTime} min</p>
      {/* Recipe list would go here, filtered by these values */}
    </div>
  );
}
```

### 5.3 — Splitting Components (When and Why)

```tsx
// ❌ One giant component — hard to read, test, and reuse
function RecipePage() {
  return (
    <div>
      <div className="header">...</div>
      <div className="ingredients">
        {/* 50 lines of ingredient rendering */}
      </div>
      <div className="steps">
        {/* 40 lines of step rendering */}
      </div>
      <div className="reviews">
        {/* 60 lines of review rendering */}
      </div>
    </div>
  );
}

// ✅ Composed from focused components
function RecipePage() {
  return (
    <div>
      <RecipeHeader recipe={recipe} />
      <IngredientList ingredients={recipe.ingredients} servings={servings} />
      <StepList steps={recipe.steps} />
      <ReviewSection recipeId={recipe.id} />
    </div>
  );
}
```

**Rules for splitting:**
1. If a section has its own state — make it a component
2. If you'd reuse it elsewhere — make it a component
3. If the parent is over ~100 lines — find pieces to extract
4. If it has a clear, nameable responsibility — make it a component

---

## 🏗️ Week 2 Exercise: Interactive Recipe Card

**Build a single-page React app (using Vite + TypeScript) that displays an interactive recipe card.**

### Requirements:

1. **Recipe data:** Hardcode 3–5 recipes as an array of objects (use your Week 1 `Recipe` interface)
2. **Recipe selector:** Buttons or tabs to switch between recipes
3. **Recipe card component:** Displays title, cuisine, difficulty, prep/cook time, ingredients, and steps
4. **Servings adjuster:** +/− buttons that scale all ingredient quantities in real time
5. **Favorite toggle:** A heart icon/button that toggles on and off (state only — no persistence)
6. **Search filter:** A text input that filters the recipe list by title (if you show a list view)

### Component structure:

```
App
├── RecipeSelector (tabs/buttons to pick a recipe)
├── RecipeCard
│   ├── RecipeHeader (title, cuisine, time, favorite button)
│   ├── ServingsAdjuster (+/− buttons)
│   ├── IngredientList (scaled by servings)
│   └── StepList (ordered list of steps)
└── SearchBar (filter recipes by title)
```

### Setup:

```bash
npm create vite@latest week-2-exercise -- --template react-ts
cd week-2-exercise
npm install
npm run dev
```

### What I'll quiz you on:

- Why did you choose `useState` vs `useRef` for a specific value?
- What would break if you removed the `key` prop from your list items?
- Show me where you used the spread operator for immutable state updates
- Why didn't you use `useEffect` for scaling ingredients? (Hint: derived state)
- Walk me through the data flow from ServingsAdjuster → IngredientList

---

## 🧪 Stretch Goals (if you finish early)

1. **Dark mode toggle:** Add a button that toggles between light and dark themes using state (preview of Context API in Week 5)
2. **Cooking timer:** Add a countdown timer for each step that has a time (uses `useEffect` cleanup + `useRef` for interval ID)
3. **Local storage persistence:** Save favorites to `localStorage` so they survive page refreshes (uses `useEffect` for syncing)
4. **Animated transitions:** Add a fade or slide animation when switching between recipes (CSS transitions or a library)
5. **Responsive layout:** Make it look good on both desktop and mobile (you'll go deep on this in Week 4 with Tailwind, but try with plain CSS now)

---

## 📚 Resources

- [React docs — Quick Start](https://react.dev/learn)
- [React docs — useState](https://react.dev/reference/react/useState)
- [React docs — useEffect](https://react.dev/reference/react/useEffect)
- [React docs — useRef](https://react.dev/reference/react/useRef)
- [React docs — Thinking in React](https://react.dev/learn/thinking-in-react) (read this one — it's excellent)
- [Vite — Getting Started](https://vitejs.dev/guide/)

---

## ✅ Week 2 Checklist

Before moving to Week 3, make sure you can:

- [ ] Create a typed React component with an interface for its props
- [ ] Use `useState` with primitives, objects, and arrays
- [ ] Explain when to use `useEffect` and when NOT to (derived state)
- [ ] Write a `useEffect` with proper dependencies and cleanup
- [ ] Explain the difference between `useState` and `useRef`
- [ ] Render a list with proper `key` props and explain why keys matter
- [ ] Pass data down via props and actions up via callback props
- [ ] Build your interactive recipe card and explain every component's responsibility

---

## 🔜 Next Week Preview

**Week 3: Next.js Foundations** — We start building Mynu for real. You'll scaffold the project with `create-next-app`, set up file-based routing with the App Router, and learn when to use Client vs. Server Components. The recipe card you built this week becomes the first real component in Mynu.
