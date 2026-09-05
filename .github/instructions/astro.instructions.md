---
description: 'Astro component patterns for pages, layouts, components, and routing'
applyTo: '**/*.astro'
---

# Astro Component Instructions

## Astro Component Patterns

Astro handles everything in the UI: pages, layouts, components, routing, and content. The site is **fully prerendered** (`output: 'static'`) — there is no client-side UI framework and no separate API server. Pages read data **directly in frontmatter** at build time via the Drizzle/Node SQLite data-access helpers in `src/lib/`.

### Component Structure

```astro
---
// Frontmatter: runs at build time (static output)
import Layout from '../layouts/Layout.astro';
import GameCard from '../components/GameCard.astro';
import { getDatabase } from '../lib/db';
import { getAllGames } from '../lib/games';

interface Props {
  title: string;
}

const { title } = Astro.props;
const games = await getAllGames(getDatabase());
---

<Layout title={title}>
  {games.map((game) => <GameCard {game} />)}
</Layout>
```

## Reusable Component Props Documentation

Every reusable `.astro` component (those that accept props) **must document its `Props` interface** to make the component's API self-explanatory:

```astro
---
/**
 * GameCard displays a game summary with title, description, category, and star rating.
 * Used in listing pages and search results.
 */
interface Props {
  game: Game;
  featured?: boolean;
}

const { game, featured = false } = Astro.props;
---
```

Use JSDoc comments on the interface and individual properties to clarify:
- What the prop represents
- Whether it's required or optional (and any default values)
- The expected data shape

**Example with detailed JSDoc:**
```astro
---
/**
 * StarRating renders a visual star rating for a game.
 */
interface Props {
  /** Game rating between 0 and 5, or null if unrated */
  rating: number | null;
  /** Whether to display the rating in compact form (default: false) */
  compact?: boolean;
}

const { rating, compact = false } = Astro.props;
---
```

## Layouts

- Create reusable layout components in `src/layouts/`
- Use `<slot />` for content injection
- Include common elements: `<head>`, navigation, footer
- Import global styles in layouts

### Layout Example

```astro
---
interface Props {
  title: string;
}
const { title } = Astro.props;
---

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>{title}</title>
  </head>
  <body>
    <slot />
  </body>
</html>
```

## Pages

- Create pages in `src/pages/`
- File-based routing: `src/pages/about.astro` → `/about`
- Dynamic routes: `src/pages/game/[id].astro`
- Provide a branded `src/pages/404.astro` — with static output, any URL with no generated page is a real 404.

### Dynamic Routes (static output)

With `output: 'static'`, every dynamic route must enumerate its pages with `getStaticPaths()` and set `prerender = true`. Query data in frontmatter using the data-access helpers:

```astro
---
import type { GetStaticPaths } from 'astro';
import Layout from '../../layouts/Layout.astro';
import { getDatabase } from '../../lib/db';
import { getAllGameIds, getGameById } from '../../lib/games';

export const prerender = true;

export const getStaticPaths = (async () => {
  const ids = await getAllGameIds(getDatabase());
  return ids.map((id) => ({ params: { id: String(id) } }));
}) satisfies GetStaticPaths;

const { id } = Astro.params;
const game = await getGameById(getDatabase(), Number(id));
---

<Layout title="Game Details - Tailspin Toys">
  <!-- Game details -->
</Layout>
```

## Data Access

- Build-time data comes from a local SQLite database via **Drizzle ORM + Node SQLite** (see [`drizzle.instructions.md`](drizzle.instructions.md)).
- Import `getDatabase()` from `src/lib/db.ts` and the typed helpers from `src/lib/games.ts`.
- The database must be migrated and seeded before `astro build`; the `prebuild` npm script (`db:setup`) handles this.

## Client Interactivity (rare)

There is no Svelte/React layer. When a page genuinely needs client behaviour, add a scoped Astro `<script>` using standard DOM APIs. Prefer native interactive elements (`<button>`, `<a href>`) so keyboard and focus behaviour come for free.

## TypeScript

- Use TypeScript for type-safe props
- Define `Props` interface in frontmatter
- Type component imports and helper return values
- Run `npx astro sync` to (re)generate route/content types before linting or type-checking
- `.astro` files are type-checked by `npm run typecheck:astro` (which runs `astro sync` then `astro check`), on the classic `typescript` package. The pure TypeScript in `db/`, `src/lib/`, and `src/types/` is type-checked separately by `npm run typecheck` (the native TS 7 compiler, `tsgo`), which does **not** process `.astro` files.

### TypeScript Formatting and Style Rules

**Explicit types for function parameters and return values:**
```ts
// Good: explicit types for parameters and returns
export async function getGameById(db: Database, gameId: number): Promise<Game | undefined> {
  // ...
}

// Avoid: inferred types at boundaries
export async function getGameById(db, gameId) {
  // ...
}
```

**Use const for immutable bindings:**
```ts
// Good
const games = await getAllGames(db);
const gameTitle = game.title;

// Avoid
let games = await getAllGames(db);
var gameTitle = game.title;
```

**Import/export style:**
- Use named imports/exports for clarity: `export function getGame(...)` and `import { getGame } from ...`
- Keep imports grouped and sorted (modules first, then relative paths)

**Naming conventions:**
- Use camelCase for variables, functions, and properties
- Use PascalCase for types, interfaces, and classes
- Use UPPER_SNAKE_CASE for constants

All TypeScript code must pass ESLint checks (`npm run lint`). ESLint enforces these patterns and catches common errors.

## Best Practices

- Keep data fetching in frontmatter (build time); avoid client-side fetching
- Minimize client-side JavaScript — the default is zero JS shipped
- Import and use global CSS styles from layouts
- Always include a `data-testid` on interactive elements (see `ui.instructions.md`)
