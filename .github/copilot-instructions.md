# Tailspin Toys Crowd Funding Development Guidelines

This is a crowdfunding platform for games with a developer theme. The application is a single **Astro 7** site (fully prerendered/static output) styled with **Tailwind CSS v4**. Data is stored in a local SQLite database accessed at build time through **Drizzle ORM + Node.js's built-in SQLite driver**; pages query the database directly in frontmatter — there is no separate backend API or client-side UI framework. Please follow these guidelines when contributing:

## Agent notes

- Explore the project before beginning code generation
- Create todo lists for long operations
  - Before each step in a todo list, reread the instructions to ensure you always have the right directions
- Always use instructions files when available, reviewing before generating code
- Do not generate summary markdown files upon completion of a task
- Always use absolute paths when running scripts and BASH commands
- **NEVER commit or push to main automatically unless explicitly instructed to do so**

## Code standards

### Required Before Each Commit

#### Testing guidelines

- **Always run tests and lint through the `quality-checks` skill — never invoke `npm run test:unit`, `npm run test:e2e`, or `npm run lint` directly.** The skill wraps environment setup, ordering, and troubleshooting. (Starting the app for manual validation is not a quality check — run `npm run dev` directly for that.)
- Run Vitest unit tests to verify the data layer and transforms, and Playwright tests to verify e2e and frontend functionality
- Run ESLint to check frontend code quality before committing
- Review the existing tests to ensure we're not duplicating efforts
- Test code should be of the same quality as the rest of the project, and follow DRY principles
- For frontend changes, verify the build (`npm run build`) directly, and run the end-to-end tests through the `quality-checks` skill, to ensure everything works correctly
- When changing the data layer (schema, helpers, transforms), update and run the corresponding unit tests

#### Project guidelines

- When updating the database schema, generate and commit the drizzle-kit migration (`npm run db:generate`)
- When adding new functionality, make sure you update the README
- Make sure all guidance in the Copilot Instructions file is updated with any relevant changes, including to project structure and scripts, and programming guidance

### Code formatting requirements

- Use TypeScript with explicit types for function parameters and return values, especially in the data layer (`db/`, `src/lib/`)
- Frontend code (TypeScript, Astro) must pass ESLint checks (`npm run lint`)

### Data Layer Patterns (Drizzle + Node SQLite)

- Define tables in `db/schema.ts`; manage schema changes with drizzle-kit migrations - see `drizzle.instructions.md`
- Keep data-access helpers in `src/lib/` with an **injectable `db`** argument so they're testable
- Keep CSV/seed logic as pure functions in `db/transforms.ts`
- Seed-derived values must be deterministic (no `Math.random`) so static builds are reproducible
- **Star ratings**: The `Game` type includes a `starRating` field (number 0-5 or `null`). Display using the `StarRating` component which formats the rating with star symbols and shows "No rating yet" when null.

#### Star Rating Implementation

The rating system includes:
- **Game type** (`src/types/game.ts`): includes `starRating: number | null`
- **StarRating component** (`src/components/StarRating.astro`): accepts `rating: number | null`, displays stars or "No rating yet"
- **GameCard component** (`src/components/GameCard.astro`): integrates StarRating display

Example usage in an Astro component:
```astro
---
import StarRating from './StarRating.astro';
const { game } = Astro.props;
---

{game.starRating !== null ? (
    <StarRating rating={game.starRating} />
) : (
    <span class="text-slate-500 text-sm">No rating yet</span>
)}
```

Rating values are between 0 and 5. The StarRating component uses the `formatStarRating()` helper to render star symbols (★) and `clampRating()` to normalize the value.

### Astro Patterns

- **Astro Pages/Components**: routing, layouts, content, and components are all `.astro` - see `astro.instructions.md`
- Query data directly in page frontmatter via the `src/lib/` helpers (build-time, static output)
- Dynamic routes use `getStaticPaths()` + `export const prerender = true`
- Provide a branded `404.astro` (unknown routes are real 404s under static output)
- Only add a scoped Astro `<script>` when genuine client interactivity is required

### Styling

- Use Tailwind CSS utility classes exclusively - see `style.instructions.md`
- Dark theme colors: slate palette (`bg-slate-800`, `text-slate-100`, etc.)
- Rounded corners and modern UI patterns
- Follow modern UI/UX principles with clean, accessible interfaces

### GitHub Actions workflows

- Follow good security practices
- Make sure to explicitly set the workflow permissions
- Add comments to document what tasks are being performed

## Scripts

- The project uses **npm scripts** for all development tasks — there is no `scripts/` directory.
- **Skills take precedence.** Before running a command directly, check whether a skill covers the task (e.g. the `quality-checks` skill wraps tests and lint). If one applies, follow it.
- Key npm scripts:
  - `npm run dev` — start the Astro dev server (`predev` migrates + seeds the local SQLite database)
  - `npm run build` — build the static site (`prebuild` migrates + seeds the local SQLite database)
  - `npm run preview` — serve the built `dist/` output
  - `npm run lint` — ESLint
  - `npm run test:unit` — Vitest unit tests
  - `npm run test:e2e` — Playwright E2E tests (builds + previews first)
  - `npm run typecheck` — type-check the pure TypeScript with `tsgo` (TypeScript 7 native compiler, via `@typescript/native-preview`) using `tsconfig.tsgo.json`
  - `npm run typecheck:astro` — type-check `.astro` files with `astro check` (classic TypeScript package)
  - `npm run typecheck:all` — run both type-check scripts (used by the CI `type-check` job)
  - `npm run db:generate` / `db:migrate` / `db:seed` / `db:setup` — Drizzle schema/migration/seed tasks

> [!NOTE]
> TypeScript 7 (`tsgo`) is adopted **side-by-side** for type checking only; it does not affect linting. ESLint + `typescript-eslint` and `astro check` still resolve the classic `typescript` package (kept at v6) because the native compiler's API isn't ready for them yet. Do **not** bump the classic `typescript` package to 7 (a Dependabot `ignore` holds it) until `typescript-eslint` + `@astrojs/check` support the native API. `tsgo` is `--noEmit` only; the site is still built by `astro build`.

## Repository Structure

The application lives at the repository root:

- `db/`: Drizzle schema, migrations, transforms, seed, and `games.csv`
- `src/lib/`: Node SQLite client (`db.ts`) and data-access helpers (`games.ts`, `ratings.ts`)
- `src/components/`: reusable `.astro` components
  - `GameCard.astro`: displays game summary with title, category, publisher, description, and star rating
  - `StarRating.astro`: renders formatted star rating or shows "No rating yet" for unrated games
- `src/layouts/`: Astro layout templates
- `src/pages/`: Astro page routes (`index.astro` listing, `game/[id].astro`, `404.astro`, `about.astro`)
- `src/styles/`: CSS and Tailwind configuration
- `src/types/`: TypeScript interfaces (Game, Publisher, Category)
- `e2e-tests/`: Playwright E2E tests (home, games, accessibility)
- `drizzle.config.ts`, `vitest.config.ts`, `astro.config.mjs`, `playwright.config.ts`: tooling config
- `README.md`: Project documentation
