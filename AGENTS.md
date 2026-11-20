# AI IDE Rules Kit (sample)

These instructions were checked against official documentation on 2026-11-20. They are a snapshot, not a lifetime-updated policy, and not an official partnership with Cursor, Windsurf/Devin, Vercel, Tailwind, Supabase, or others.

If this repository also has `.cursor/rules/*.mdc` or `.windsurf/rules/*.md`, do not paste the same rule text into both places. Prefer the tool-native rule files for scoped application, and keep this `AGENTS.md` only if you are not loading the duplicated rule files. See `INSTALLATION.md` in the full pack.

## TypeScript strict mode

Checked against the official TSConfig Reference and TypeScript Handbook on 2026-11-20 (handbook pages on typescriptlang.org showed a last-updated date of 2026-11-17). This snapshot does not require a specific TypeScript major beyond what your project already uses. Next.js 16 documents a minimum of TypeScript 5.1.0 if you are in that stack. This is not an official Microsoft partnership.

## Turn on `strict`

In `tsconfig.json`:

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

Official docs: `"strict": true` enables the **strict mode family**. As listed on the TSConfig Reference (related flags under `strict`), that family includes:

- `noImplicitAny`
- `strictNullChecks`
- `strictFunctionTypes`
- `strictBindCallApply`
- `strictPropertyInitialization`
- `strictBuiltinIteratorReturn` (added in TypeScript 5.6; default `true` when `strict` is on)
- `noImplicitThis`
- `alwaysStrict`
- `useUnknownInCatchVariables`

You may disable an individual family flag if you must, but do not "fix" errors by turning `strict` off for the whole project. Official docs also note that future TypeScript versions may add new checks under `strict`.

These flags are documented type-checking options but are **not** part of the `strict` family. Enable them only when you intend the extra strictness:

- `noUncheckedIndexedAccess` — indexed access adds `undefined`
- `exactOptionalPropertyTypes` — distinguish "missing" from `undefined`
- `noImplicitOverride`, `noPropertyAccessFromIndexSignature`, `noFallthroughCasesInSwitch`, `noUnusedLocals`, `noUnusedParameters`

Do not invent compiler flag names. If a flag is not in the official TSConfig Reference, do not add it.

## Do not use casual `any`

Official handbook: a value of type `any` disables further type checking. That is the opposite of this rule.

- Do not annotate values as `any` to silence errors.
- Do not use `as any` or double assertions (`as unknown as T`) unless you can state why a safe alternative is impossible, and keep the assertion in one narrow place.
- With `noImplicitAny` (on under `strict`), parameters and other values that would have been implicit `any` are errors. Add a real type or a valid inference instead.

Prefer `unknown` when the type is not yet known, then **narrow** before use.

## Narrowing (official handbook patterns)

Use control-flow narrowing rather than assertions:

- `typeof` guards (`"string" | "number" | "bigint" | "boolean" | "symbol" | "undefined" | "object" | "function"`). Remember `typeof null === "object"`.
- `instanceof` checks
- Truthiness / equality checks
- Discriminated unions (a shared literal tag)
- Exhaustiveness with `never` in a `default` or remaining-union branch

Example:

```ts
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input
  }
  return padding + input
}
```

Catch variables are `unknown` when `useUnknownInCatchVariables` is on (the `strict` default since TypeScript 4.4). Narrow before reading `.message`:

```ts
try {
  // ...
} catch (err) {
  if (err instanceof Error) {
    console.error(err.message)
  } else {
    console.error(err)
  }
}
```

Do not use the postfix `!` non-null assertion unless you can prove the value is not `null` or `undefined`. Official docs: `!` does not change runtime behavior.

## `fetch` and JSON

Standard library typings for `JSON.parse` and often `Response.json()` are historically loose (`any`). Do not let that `any` spread.

```ts
async function readJson(res: Response): Promise<unknown> {
  const data: unknown = await res.json()
  return data
}

function isUser(value: unknown): value is { id: string; name: string } {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value &&
    typeof value.id === "string" &&
    typeof value.name === "string"
  )
}

const data = await readJson(response)
if (!isUser(data)) {
  throw new Error("Unexpected user payload")
}
// data is now { id: string; name: string }
```

The same pattern applies to `JSON.parse(...)`: assign to `unknown`, then narrow or validate. A schema validator is fine if the project already uses one; do not add a new runtime dependency just to satisfy this rule unless the user asks.

## Other strict-family pitfalls

- `strictNullChecks`: check for `null` and `undefined` (for example, `array.find(...)` returns `T | undefined`).
- `strictPropertyInitialization`: class fields must be initialized or marked definitely assigned, or typed as possibly `undefined`.
- Prefer `string`, `number`, and `boolean` over the boxed `String`, `Number`, and `Boolean` types (official everyday-types guidance).

## Next.js App Router

Checked against official Next.js App Router docs on 2026-11-20. Target **Next.js 16** (stable line as of this date; GitHub published `v16.4.6` on 2026-11-09). This is a documentation snapshot, not a lifetime policy and not an official Vercel partnership.

The App Router (`app/`) is the current documented default for new Next.js apps. Do not present the Pages Router (`pages/`) as the 2026 default. If you are editing a legacy Pages Router app, say so explicitly and do not mix Pages Router data APIs into App Router files.

## Version notes (15 vs 16)

- Next.js 15 introduced async Request APIs (`params`, `searchParams`, `cookies()`, `headers()`, `draftMode()`) with a temporary synchronous compatibility period.
- Next.js 16 **removes synchronous access**. Always `await` these APIs (or use React `use()` in a Client Component where that is the documented option). This style is also the forward-compatible choice on 15.
- Next.js 16 requires **Node.js 20.9+** and **TypeScript 5.1+**. Turbopack is stable and is the default for `next dev` and `next build`. You no longer need `--turbopack` / `--turbo` flags for that default.
- If a project still has a custom webpack config, `next build` (Turbopack by default) can fail; follow the official upgrade guide rather than inventing a workaround.
- To migrate leftover sync access, the official docs mention: `npx @next/codemod@canary next-async-request-api .`

## Server vs Client Components

Layouts and pages are Server Components by default. Use `'use client'` only when you need:

- State or event handlers (`onClick`, `onChange`)
- Lifecycle logic (`useEffect`)
- Browser-only APIs (`window`, `localStorage`, `navigator`)
- Custom hooks that depend on the above

Use Server Components to fetch data close to the source, keep API keys and other secrets on the server, and reduce client JavaScript.

Pattern: fetch in a Server Component, pass **safe, already-sanitized props** into a small Client Component for interactivity. Never pass secret env values, service credentials, or raw privileged tokens into a Client Component (they travel in the RSC payload).

Mark modules that must stay server-only with:

```ts
import 'server-only'
```

Next.js treats this import as a boundary: importing that module into a Client Component is a build-time error. Use the matching `client-only` package for browser-only modules. Installing the npm packages is optional; Next.js documents that it handles these imports internally.

## Async `params` and `searchParams`

Type and await dynamic APIs. Official page example:

```tsx
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  // ...
}
```

- `params` is a Promise in `layout.js`, `page.js`, `route.js`, `default.js`, and metadata image / icon file conventions.
- `searchParams` is a Promise in `page.js`.
- Do not read `params.id` or `searchParams.q` synchronously in Next.js 16.

## `cookies()` and `headers()`

Both are async functions from `next/headers`:

```ts
import { cookies, headers } from 'next/headers'

const cookieStore = await cookies()
const theme = cookieStore.get('theme')

const headersList = await headers()
const userAgent = headersList.get('user-agent')
```

- `headers()` returns a **read-only** Web `Headers` object. You cannot `set` or `delete` outgoing headers through it.
- Cookie `.set()` / `.delete()` only belong in a Server Function (`'use server'`) or a Route Handler, not during Server Component render.
- Prefer `httpOnly`, `secure` (HTTPS), and an explicit `sameSite` when setting auth or session cookies. The only documented default among the options is `path: '/'`.
- These are request-time APIs. Without Cache Components, using them in a layout or page opts the route into dynamic rendering.

## Route Handlers

Use `app/**/route.ts` (or `src/app/**/route.ts`) with the Web Request/Response APIs. Supported methods: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS`.

```ts
import type { NextRequest } from 'next/server'

export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ team: string }> }
) {
  const { team } = await params
  return Response.json({ team })
}
```

- After type generation (`next dev`, `next build`, or `next typegen`), `RouteContext<'/users/[id]'>` is a globally available helper for typed `params`.
- If `OPTIONS` is not defined, Next.js documents that it will implement `OPTIONS` and set `Allow` from the other exported methods.
- Default caching for `GET` handlers changed in the 15 RC from static to dynamic; do not assume GET handlers are statically cached.

## Proxy vs deprecated middleware (Next.js 16)

In Next.js 16 the `middleware` file convention is **deprecated and renamed to `proxy`**. Place `proxy.ts` (or `proxy.js`) at the project root or inside `src/`, at the same level as `app` or `pages`. Export a function named `proxy` (or a default export). Official migration:

```bash
npx @next/codemod@canary middleware-to-proxy .
```

Proxy defaults to the **Node.js runtime**. The official upgrade checklist warns: do not blindly rename Edge middleware to proxy. Read the current "Migration to Proxy" docs if the existing file depends on Edge-only behavior, and preserve required routing behavior.

Without a `matcher`, Proxy runs on every request, including `_next/static`, `_next/image`, and `public/` assets. Use `config.matcher` so auth or redirects do not block CSS, JS, or images.

## Pitfalls to avoid

- Do not leak `process.env` secrets into Client Components or into props you pass across the server/client boundary.
- Do not use Pages Router APIs (`getServerSideProps`, `getStaticProps`, `getInitialProps`) in App Router files.
- Do not invent new `fetch` cache option names. If you need caching or revalidation, follow the official data-fetching docs for the installed Next.js version.
- Prefer the official upgrade path (`npx next upgrade` on 16.1+, or the documented `@next/codemod` flow) over hand-waving version jumps.
