# Week 1: TypeScript & JavaScript Fundamentals

> **Phase:** 0 — Rust Removal
> **Goal:** Shake off the cobwebs. No Mynu code yet — just drills.
> **Time:** ~10 hours
> **Deliverable:** A CLI recipe calculator in TypeScript

---

## What You'll Learn This Week

By the end of Week 1, you should be able to:
- Write TypeScript with proper types, interfaces, and generics
- Use modern JS patterns (arrow functions, destructuring, spread/rest)
- Work with Promises and async/await confidently
- Transform data with array methods (`map`, `filter`, `reduce`)
- Structure code with ES modules

Everything this week is taught through the lens of recipe data — so when we start building Mynu for real in Week 3, these patterns will feel natural.

---

## Day 1–2: TypeScript Basics (~3 hours)

### 1.1 — Why TypeScript?

TypeScript is JavaScript with a type system bolted on. It catches bugs *before* your code runs. This matters for Mynu because:
- A recipe has a specific shape (title, ingredients, steps) — types enforce that shape
- When you refactor later, the compiler tells you what broke
- Your IDE gives you autocomplete for everything

### 1.2 — Core Types

```typescript
// Primitives
let recipeName: string = "Pasta Carbonara";
let servings: number = 4;
let isVegetarian: boolean = false;
let notes: string | null = null; // union type — could be string or null

// Arrays
let tags: string[] = ["Italian", "Quick", "Pasta"];
let ratings: number[] = [4.5, 3.8, 5.0];
let mixed: (string | number)[] = ["eggs", 3]; // avoid this — usually means you need a better type
```

**🧠 Why it matters for Mynu:** Every recipe field needs a type. If `servings` accidentally becomes `"four"` instead of `4`, your scaling math breaks silently in JavaScript. TypeScript catches that.

### 1.3 — Interfaces & Type Aliases

```typescript
// Interface — describes the shape of an object
interface Ingredient {
  name: string;
  quantity: number;
  unit: string;
}

// Type alias — can describe anything
type CuisineType = "Italian" | "Mexican" | "Japanese" | "Indian" | "Other";

// Combining them
interface Recipe {
  id: string;
  title: string;
  description: string;
  cuisine: CuisineType;          // uses our union type
  servings: number;
  prepTimeMinutes: number;
  cookTimeMinutes: number;
  ingredients: Ingredient[];      // array of Ingredient objects
  steps: string[];
  tags: string[];
  isPublic: boolean;
  createdAt: Date;
}
```

**🧠 Why it matters for Mynu:** This `Recipe` interface is essentially what your Prisma schema will look like in Week 6. You're learning the shape of your data before you build the database.

### 1.4 — Optional Properties & Readonly

```typescript
interface Recipe {
  id: string;
  title: string;
  description?: string;          // optional — might not exist
  imageUrl?: string;             // optional — not every recipe has a photo
  readonly createdAt: Date;      // can't be changed after creation
}

// Using optional properties
const quickRecipe: Recipe = {
  id: "1",
  title: "Toast",
  createdAt: new Date(),
  // description and imageUrl are omitted — that's fine
};
```

### 1.5 — Enums

```typescript
enum DifficultyLevel {
  Easy = "easy",
  Medium = "medium",
  Hard = "hard",
}

enum MealType {
  Breakfast = "breakfast",
  Lunch = "lunch",
  Dinner = "dinner",
  Snack = "snack",
  Dessert = "dessert",
}

interface Recipe {
  title: string;
  difficulty: DifficultyLevel;
  mealType: MealType;
}

const pancakes: Recipe = {
  title: "Fluffy Pancakes",
  difficulty: DifficultyLevel.Easy,
  mealType: MealType.Breakfast,
};
```

**🧠 Why it matters for Mynu:** Enums prevent typos. Without them, someone might type `"breakfst"` and your filter breaks. With enums, TypeScript won't even let you compile.

### 1.6 — Generics (the basics)

```typescript
// A function that works with ANY type
function getFirst<T>(items: T[]): T | undefined {
  return items[0];
}

const firstTag: string | undefined = getFirst<string>(["Italian", "Quick"]);
const firstRating: number | undefined = getFirst<number>([4.5, 3.8]);

// A generic API response wrapper — you'll use this pattern A LOT
interface ApiResponse<T> {
  data: T;
  success: boolean;
  error?: string;
}

// Now it works for any data type
type RecipeResponse = ApiResponse<Recipe>;
type RecipeListResponse = ApiResponse<Recipe[]>;
```

**🧠 Why it matters for Mynu:** Every API endpoint you build in Week 7 will return data in a consistent wrapper. Generics let you type that wrapper once and reuse it everywhere.

---

### ✅ Checkpoint 1

Before moving on, you should be able to answer:
1. What's the difference between `interface` and `type`? When would you use each?
2. Why use `string[]` instead of `Array<string>`? (Trick question — they're the same)
3. What happens if you try to assign a `number` to a `string` variable in TypeScript?
4. Write an interface for a `User` that has an `id`, `email`, `name`, optional `avatarUrl`, and a `recipes` array

---

## Day 2–3: Functions & Modern JS Patterns (~3 hours)

### 2.1 — Arrow Functions & Type Annotations

```typescript
// Regular function
function calculateTotalTime(prep: number, cook: number): number {
  return prep + cook;
}

// Arrow function — same thing, shorter syntax
const calculateTotalTime = (prep: number, cook: number): number => {
  return prep + cook;
};

// Even shorter for one-liners
const calculateTotalTime = (prep: number, cook: number): number => prep + cook;

// Typing a function that takes a callback
function processRecipes(
  recipes: Recipe[],
  callback: (recipe: Recipe) => void
): void {
  recipes.forEach(callback);
}
```

### 2.2 — Destructuring & Spread

```typescript
// Object destructuring — pull out specific fields
const recipe = { title: "Tacos", servings: 4, cuisine: "Mexican" };
const { title, servings } = recipe;
console.log(title); // "Tacos"

// With renaming
const { title: recipeName, servings: serves } = recipe;

// Spread — copy and extend objects
const updatedRecipe = { ...recipe, servings: 6 }; // override servings
const withTags = { ...recipe, tags: ["Quick", "Family"] }; // add new field

// Array spread
const allTags = [...recipe.tags, "New Tag"];

// Rest parameters — collect remaining args
function logRecipeInfo(title: string, ...tags: string[]): void {
  console.log(`${title}: ${tags.join(", ")}`);
}
logRecipeInfo("Tacos", "Mexican", "Quick", "Easy");
```

**🧠 Why it matters for Mynu:** You'll use destructuring constantly in React (`const { title, ingredients } = recipe`) and spread for immutable state updates (`setRecipe({ ...recipe, title: "New Title" })`).

### 2.3 — Closures

```typescript
// A closure is a function that remembers variables from its outer scope
function createRecipeScaler(baseServings: number) {
  // This inner function "closes over" baseServings
  return function (targetServings: number, quantity: number): number {
    return (quantity / baseServings) * targetServings;
  };
}

const scaleFrom4 = createRecipeScaler(4);
console.log(scaleFrom4(2, 400)); // 200g — scaled from 4 servings to 2
console.log(scaleFrom4(8, 400)); // 800g — scaled from 4 servings to 8

const scaleFrom2 = createRecipeScaler(2);
console.log(scaleFrom2(6, 100)); // 300g — scaled from 2 servings to 6
```

**🧠 Why it matters for Mynu:** Closures power React hooks under the hood. When you write `useState`, you're using closures. Understanding them prevents subtle bugs.

---

## Day 3–4: Async JavaScript (~2 hours)

### 3.1 — Promises

```typescript
// Simulating an API call to fetch a recipe
function fetchRecipe(id: string): Promise<Recipe> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id === "1") {
        resolve({
          id: "1",
          title: "Spaghetti Bolognese",
          servings: 4,
          ingredients: [],
          steps: [],
          // ... other fields
        } as Recipe);
      } else {
        reject(new Error(`Recipe ${id} not found`));
      }
    }, 1000); // simulate network delay
  });
}

// Using the promise
fetchRecipe("1")
  .then((recipe) => console.log(recipe.title))
  .catch((error) => console.error(error.message));
```

### 3.2 — Async/Await (the better way)

```typescript
// Same thing, but readable
async function getRecipeAndDisplay(id: string): Promise<void> {
  try {
    const recipe = await fetchRecipe(id);
    console.log(`Found: ${recipe.title} (serves ${recipe.servings})`);
  } catch (error) {
    if (error instanceof Error) {
      console.error(`Failed: ${error.message}`);
    }
  }
}

// Fetching multiple recipes in parallel
async function getAllRecipes(ids: string[]): Promise<Recipe[]> {
  const promises = ids.map((id) => fetchRecipe(id));
  const recipes = await Promise.all(promises);
  return recipes;
}

// Promise.all fails if ANY promise fails
// Use Promise.allSettled if you want partial results
async function getRecipesSafely(ids: string[]): Promise<Recipe[]> {
  const results = await Promise.allSettled(ids.map((id) => fetchRecipe(id)));
  return results
    .filter((r) => r.status === "fulfilled")
    .map((r) => (r as PromiseFulfilledResult<Recipe>).value);
}
```

**🧠 Why it matters for Mynu:** Every database query, every API call, every AI request is async. If you don't understand `async/await` and error handling, your app will break in confusing ways.

---

## Day 4–5: Array Methods (~2 hours)

These are the workhorses of data transformation. You'll use them everywhere in Mynu.

### 4.1 — map, filter, find

```typescript
const recipes: Recipe[] = [
  { id: "1", title: "Tacos", cuisine: "Mexican", servings: 4, prepTimeMinutes: 20, cookTimeMinutes: 15, isPublic: true },
  { id: "2", title: "Sushi", cuisine: "Japanese", servings: 2, prepTimeMinutes: 45, cookTimeMinutes: 0, isPublic: false },
  { id: "3", title: "Pizza", cuisine: "Italian", servings: 8, prepTimeMinutes: 30, cookTimeMinutes: 25, isPublic: true },
  { id: "4", title: "Curry", cuisine: "Indian", servings: 6, prepTimeMinutes: 15, cookTimeMinutes: 40, isPublic: true },
] as Recipe[];

// map — transform each item
const titles: string[] = recipes.map((r) => r.title);
// ["Tacos", "Sushi", "Pizza", "Curry"]

const recipeSummaries = recipes.map((r) => ({
  title: r.title,
  totalTime: r.prepTimeMinutes + r.cookTimeMinutes,
}));
// [{ title: "Tacos", totalTime: 35 }, ...]

// filter — keep items that match a condition
const publicRecipes = recipes.filter((r) => r.isPublic);
// [Tacos, Pizza, Curry]

const quickRecipes = recipes.filter((r) => r.prepTimeMinutes + r.cookTimeMinutes <= 30);
// [Tacos]

// find — get the first match (or undefined)
const sushi = recipes.find((r) => r.title === "Sushi");
```

### 4.2 — reduce

```typescript
// reduce — accumulate a single value from an array
const totalServings = recipes.reduce((sum, r) => sum + r.servings, 0);
// 20

// Group recipes by cuisine
const byCuisine = recipes.reduce((groups, recipe) => {
  const key = recipe.cuisine;
  if (!groups[key]) {
    groups[key] = [];
  }
  groups[key].push(recipe);
  return groups;
}, {} as Record<string, Recipe[]>);
// { Mexican: [Tacos], Japanese: [Sushi], Italian: [Pizza], Indian: [Curry] }
```

### 4.3 — Chaining Methods

```typescript
// Real Mynu scenario: Get titles of public recipes that take under 30 min, sorted alphabetically
const quickPublicTitles = recipes
  .filter((r) => r.isPublic)
  .filter((r) => r.prepTimeMinutes + r.cookTimeMinutes <= 35)
  .map((r) => r.title)
  .sort();
// ["Tacos"]

// Generate a shopping list from selected recipes
interface ShoppingItem {
  name: string;
  totalQuantity: number;
  unit: string;
}

function generateShoppingList(selectedRecipes: Recipe[]): ShoppingItem[] {
  const allIngredients = selectedRecipes.flatMap((r) => r.ingredients);

  const grouped = allIngredients.reduce((map, ingredient) => {
    const key = `${ingredient.name}-${ingredient.unit}`;
    if (map.has(key)) {
      map.get(key)!.totalQuantity += ingredient.quantity;
    } else {
      map.set(key, {
        name: ingredient.name,
        totalQuantity: ingredient.quantity,
        unit: ingredient.unit,
      });
    }
    return map;
  }, new Map<string, ShoppingItem>());

  return Array.from(grouped.values());
}
```

**🧠 Why it matters for Mynu:** The shopping list generator above is almost exactly what you'll build in Week 10. Array methods are how you transform raw data into useful UI-ready output.

---

## Day 5: ES Modules & Project Structure (~1 hour)

### 5.1 — Import / Export

```typescript
// ---- types/recipe.ts ----
export interface Ingredient {
  name: string;
  quantity: number;
  unit: string;
}

export interface Recipe {
  id: string;
  title: string;
  ingredients: Ingredient[];
  servings: number;
}

// Default export — one per file
export default interface User {
  id: string;
  name: string;
  email: string;
}

// ---- utils/scaling.ts ----
import { Ingredient, Recipe } from "../types/recipe";

export function scaleIngredient(
  ingredient: Ingredient,
  originalServings: number,
  targetServings: number
): Ingredient {
  const factor = targetServings / originalServings;
  return {
    ...ingredient,
    quantity: Math.round(ingredient.quantity * factor * 100) / 100,
  };
}

export function scaleRecipe(recipe: Recipe, targetServings: number): Recipe {
  return {
    ...recipe,
    servings: targetServings,
    ingredients: recipe.ingredients.map((i) =>
      scaleIngredient(i, recipe.servings, targetServings)
    ),
  };
}

// ---- main.ts ----
import { Recipe } from "./types/recipe";
import { scaleRecipe } from "./utils/scaling";

const carbonara: Recipe = {
  id: "1",
  title: "Pasta Carbonara",
  servings: 4,
  ingredients: [
    { name: "Spaghetti", quantity: 400, unit: "g" },
    { name: "Eggs", quantity: 4, unit: "whole" },
    { name: "Pecorino", quantity: 100, unit: "g" },
    { name: "Guanciale", quantity: 200, unit: "g" },
  ],
};

const scaledFor2 = scaleRecipe(carbonara, 2);
console.log(scaledFor2.ingredients);
// [{ name: "Spaghetti", quantity: 200, unit: "g" }, ...]
```

**🧠 Why it matters for Mynu:** This is exactly how your Next.js project will be organized. Types in one folder, utilities in another, components in another. Get used to this pattern now.

---

## 🏗️ Week 1 Exercise: CLI Recipe Calculator

**Build a command-line tool that:**
1. Reads a JSON file containing a recipe (title, servings, ingredients with name/quantity/unit)
2. Takes a target serving count as a command-line argument
3. Scales all ingredient quantities proportionally
4. Prints the scaled recipe to the console in a readable format
5. Handles edge cases: missing file, invalid JSON, negative servings, zero servings

**Starter structure:**
```
week-1-exercise/
├── data/
│   └── carbonara.json
├── src/
│   ├── types.ts        # Recipe & Ingredient interfaces
│   ├── scaling.ts       # scaleRecipe function
│   ├── display.ts       # formatRecipe function (pretty console output)
│   └── index.ts         # main entry — reads file, parses args, orchestrates
├── package.json
└── tsconfig.json
```

**Setup commands:**
```bash
mkdir week-1-exercise && cd week-1-exercise
npm init -y
npm install typescript ts-node @types/node --save-dev
npx tsc --init
```

**Sample `carbonara.json`:**
```json
{
  "id": "1",
  "title": "Pasta Carbonara",
  "servings": 4,
  "ingredients": [
    { "name": "Spaghetti", "quantity": 400, "unit": "g" },
    { "name": "Eggs", "quantity": 4, "unit": "whole" },
    { "name": "Pecorino Romano", "quantity": 100, "unit": "g" },
    { "name": "Guanciale", "quantity": 200, "unit": "g" },
    { "name": "Black Pepper", "quantity": 2, "unit": "tsp" }
  ],
  "steps": [
    "Boil pasta in salted water",
    "Crisp the guanciale in a pan",
    "Mix eggs and cheese",
    "Toss hot pasta with guanciale, then add egg mixture off heat",
    "Season with black pepper"
  ]
}
```

**Expected output:**
```
$ npx ts-node src/index.ts data/carbonara.json 2

🍝 Pasta Carbonara (scaled to 2 servings)
──────────────────────────────────────────

📦 Ingredients:
  • 200g Spaghetti
  • 2 whole Eggs
  • 50g Pecorino Romano
  • 100g Guanciale
  • 1 tsp Black Pepper

📋 Steps:
  1. Boil pasta in salted water
  2. Crisp the guanciale in a pan
  3. Mix eggs and cheese
  4. Toss hot pasta with guanciale, then add egg mixture off heat
  5. Season with black pepper
```

---

## 🧪 Stretch Goals (if you finish early)

1. **Multiple recipes:** Accept a folder of JSON files and let the user pick which one to scale
2. **Unit conversion:** If scaling makes a quantity awkward (e.g., 0.33 cups), convert to a friendlier unit (e.g., ~5 tbsp)
3. **Shopping list merge:** Given 2+ recipe files, generate a combined shopping list that sums duplicate ingredients
4. **Input validation:** Use a simple validation function to check the JSON matches your `Recipe` interface at runtime (preview of Zod, which you'll learn in Week 5)

---

## 📚 Resources

- [TypeScript Handbook (official)](https://www.typescriptlang.org/docs/handbook/)
- [TypeScript Playground (try code in browser)](https://www.typescriptlang.org/play)
- [JavaScript.info — Modern JS Tutorial](https://javascript.info/)
- [Array Methods Cheat Sheet](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

---

## ✅ Week 1 Checklist

Before moving to Week 2, make sure you can:

- [ ] Explain the difference between `interface` and `type`
- [ ] Write a function with proper TypeScript type annotations
- [ ] Use `async/await` with `try/catch` error handling
- [ ] Transform an array using `map`, `filter`, and `reduce` without looking up syntax
- [ ] Structure a TypeScript project with multiple files using `import/export`
- [ ] Run your CLI recipe calculator and get correct output
- [ ] Explain every line of your code back to me

---

## 🔜 Next Week Preview

**Week 2: React Refresher** — We take these TypeScript skills into the browser. You'll build a visual recipe card component using React, learning `useState`, `useEffect`, and component composition. The types you defined this week become your component props.
