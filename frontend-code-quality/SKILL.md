---
name: frontend-code-quality
description: General frontend coding discipline for JavaScript, TypeScript, React, React Router, and Next.js implementation work. Use this skill for substantive frontend component, hook, utility, state, form, UI logic, refactor, or review tasks where code quality choices matter. Do not use this skill for IQF frontend Zod schema generation or backend-to-frontend schema mapping; use iqf-fe-schema-generator for those tasks when available. It helps keep FE code simple, idiomatic to the existing codebase, minimally exported, type-safe through inference, aligned with React best practices, and explicit about low-quality files encountered during edits. For React component or Next.js tasks, also use vercel-react-best-practices when available. For React Router routing, route module, loader, action, fetcher, navigation, pending UI, SSR/SPA/pre-rendering, or upgrade tasks, also use react-router when available.
---

# Frontend Code Quality

Use this skill when writing or changing frontend code in JavaScript or TypeScript. The goal is not to make code clever or over-abstracted; the goal is to make the smallest correct change that feels native to the project.

Prefer code that a teammate can read once and safely modify later. Avoid adding API surface, names, helpers, types, or React state unless they clearly pay for themselves.

Apply KISS: keep it simple, stupid. Start with the most direct solution that satisfies the requirement, then add structure only when the current problem proves it is needed. Do not build for hypothetical future variants, extension points, or reuse that the task does not actually require.

## First Read The Codebase

Before changing code, inspect nearby files and similar existing implementations.

Use the dominant local style as the default for:

- File organization and naming
- Import ordering and path aliases
- Component structure
- State management patterns
- Form, validation, data fetching, and error handling patterns
- Styling conventions
- Test style, if tests are relevant

Do not introduce a new pattern just because it is generally good. Introduce it only when it solves the current problem better than the patterns already used in this frontend.

## Handle Refactor Requests By Scope

When the user explicitly asks for a refactor, first decide whether the request is broad or concrete.

For a broad, ambiguous, architectural, or multi-file refactor request, first inspect the relevant code and propose a refactor plan that applies this skill's rules: KISS, minimal API surface, narrow exports, TypeScript inference, React best practices, local feature organization, existing utilities, and quality-debt awareness. Wait for the user's confirmation before editing.

For a concrete scoped refactor where the user clearly asked for implementation, such as removing duplicated validation in one component, simplifying a hook, or cleaning up a named file, inspect nearby code and proceed directly. Keep the change narrow and mention any larger refactor ideas only as follow-up notes.

The goal of a refactor is almost never abstraction or unification by itself. Do not chase DRY mechanically. Do not turn straightforward JSX into generic field configuration arrays, factories, render registries, or shared abstractions just because several lines look similar. Repetition is often cheaper than an abstraction that hides behavior.

Focus refactors almost exclusively on code quality:

- Simplify control flow
- Make state flow easier to understand
- Remove unnecessary effects, refs, state, exports, and types
- Move stable non-closure code out of components
- Give complex local UI sections clear names
- Keep side effects and data fetching isolated and cleanup-safe
- Make the component easier to read without changing behavior

For broad refactors, the proposal should include:

- Files that would be changed
- Main structural changes
- What complexity would be removed
- Any behavior that should stay unchanged
- Risks or open questions
- Quality-debt files noticed during inspection

When proposing the plan, explicitly say what you will not abstract. This prevents the refactor from drifting into generic architecture work.

After presenting a broad refactor proposal, wait for the user's confirmation before editing files. Do not start a broad refactor immediately unless the user explicitly asked you to proceed without approval.

If the user asks for a small concrete change, bug fix, or implementation task, do not block on a refactor proposal. Make the requested change directly and keep any unrelated refactor ideas as optional follow-up notes.

## Track Low-Quality Files You Touch

When editing or reading a file for the current task, notice whether the file is materially low quality: tangled responsibilities, repeated logic, unsafe typing, unclear state flow, excessive effects, fragile abstractions, or code that makes the requested change risky.

Do not derail the current task into a broad cleanup unless the cleanup is necessary for the requested change. Instead:

- Remember the exact file path
- Keep a concise reason for why the file is low quality
- Tell the user that the file quality is poor and should be fixed after the current change is complete
- Include the full list of these files in the final response under a clear heading

Only flag meaningful quality debt. Do not complain about minor style differences, old code that is merely unfamiliar, or code that is outside the files you actually inspected.

Common examples of low-quality FE code:

- A component mixes too many responsibilities: form shell, permissions, data fetching, mutation invalidation, field dependency cleanup, action rendering, and the full field layout in one place
- `useEffect` is used to patch derived form state when a clearer event-level update or narrower dependency flow would be possible
- `useRef` stores previous values only to drive business logic, making state transitions harder to reason about
- Hook calls are hidden inside JSX props instead of being named near the top of the component
- Conditional JSX uses ternaries with `null` or empty fragments instead of guarded rendering
- Large inline `actions={...}` or `render...={...}` blocks make the main layout hard to scan
- Many one-off helper abstractions, exported types, or configuration layers are introduced to "clean up" code without actually simplifying behavior
- Copy/pasted form fields are converted into a generic field renderer even though the fields have different labels, options, read-only rules, and behavior

For example, a form component is low quality if it watches multiple form fields, runs effects to clear dependent fields, fetches and copies data from another entity, renders navigation actions, defines every field, and handles permissions all in one component. A good refactor would not start by building a generic form-field abstraction. It would first separate obvious local responsibilities, name complex sections, pull hook results out of JSX, remove ternary-to-null rendering, and reduce effects only where behavior stays clear.

Use this format when reporting it:

```text
Quality debt noticed:
- path/to/file.tsx: short reason
```

## Keep The Change Small

Prefer direct, local code over abstraction. A helper is worthwhile when it removes meaningful duplication, gives a concept a useful name, or hides genuinely tricky details. It is not worthwhile when it only wraps one expression or saves one line.

Avoid overengineering. Do not introduce generic frameworks, factories, strategy maps, polymorphic abstractions, reusable hooks, context providers, or configuration layers unless the current code has enough real complexity to justify them. A plain conditional, local variable, or direct JSX block is often the best solution.

Before creating a custom function, check whether:

- The codebase already has an equivalent helper
- The platform already provides it
- An installed utility library such as lodash already provides it and is used in this codebase
- A simple inline expression is clearer

Do not create many small custom helpers just to make code look organized. Too many names make frontend code harder to follow because the reader has to jump between definitions.

## Exports And File API

Keep module APIs narrow.

Only export values that are used from another file or are intentionally public entry points. Keep implementation details private to the file.

Avoid exporting:

- One-off helper functions
- Local-only constants
- Types used only by one component or one function
- Intermediate schema fragments or internal UI configuration

If something is only needed in the same file, leave it unexported. If a future use appears, export it then.

## TypeScript Discipline

Let TypeScript infer what it can infer cleanly. Explicit types are useful at boundaries, not everywhere.

Prefer inference for:

- Function return types when the return expression is clear
- Local variables
- Inline callbacks
- Component internals
- Derived values

Add explicit types when they improve safety or communication, especially for:

- Public function parameters
- External API payloads
- Form values crossing module boundaries
- Component props when not inferred from an existing schema or library type
- Complex generics where inference becomes misleading

Avoid creating named types for one-off shapes. Inline the type near the use site unless it is reused, clarifies a domain concept, or represents a public boundary.

Do not export types unless another file needs them.

Avoid `as`. Treat assertions as a last resort. Prefer narrowing, parsing, schema validation, discriminated unions, `satisfies`, or better generic constraints. If an assertion is unavoidable, keep it as narrow and local as possible.

## React Code

For React or Next.js work, also apply the `vercel-react-best-practices` skill.

Keep components and hooks simple:

- Derive values during render when possible
- Prefer event handlers for interaction logic instead of effects
- Use `useEffect` only for synchronizing with external systems, subscriptions, timers, browser APIs, or imperative libraries
- Avoid storing derived data in state
- Avoid putting values in refs just to dodge dependencies or re-renders
- Do not define child components inside parent components unless a closure is intentionally needed and worth the tradeoff

For JSX conditional blocks, prefer guarded rendering when the `else` branch is only `null` or an empty fragment. Keep a ternary when it matches local style or makes related branches easier to compare.

Prefer this for one-sided conditionals:

```tsx
{
  !!condition && <div />;
}
```

Avoid this by default when there is no meaningful `else` branch:

```tsx
{
  condition ? <div /> : null;
}
{
  condition ? <div /> : <></>;
}
```

Move stable code outside the component when it does not need props, state, context, hooks, or closure variables. Good candidates include constants, option arrays, schemas, simple formatters, regexes, and pure helper functions.

Keep code inside the component when it depends naturally on local closure state or when moving it out would require passing many parameters and make the code less readable.

Use React performance tools deliberately. Do not add `useMemo`, `useCallback`, refs, or memoized components by default. Follow the repo's existing React Compiler and memoization conventions.

When modern React APIs are already part of the project, use patterns such as `startTransition`, `useDeferredValue`, or `useEffectEvent` where they directly match the problem. Do not add them for novelty.

## Practical FE Patterns From Strong Codebases

These patterns are extracted from a production FE codebase with React Router, React 19, Tailwind, `react-intl`, `lodash-es`, `clsx`, and `tailwind-merge`. Treat them as general guidance, not as framework-specific rules. Use the tool-specific parts only when those tools, or equivalent local conventions, are already present in the project.

Keep shared components thin and composable. A component should own the reusable interaction or visual contract, while feature-specific content stays in the route or feature file. Prefer render props or explicit slots only when the caller truly needs to control markup.

Example shape:

```tsx
<Quiz
  quiz={quiz}
  renderQuestion={(question, options) => (
    <Screen>
      <Screen.Heading>{question.question}</Screen.Heading>
      {options}
    </Screen>
  )}
/>
```

Keep feature modules local. If a route or feature has its own config, assets, game logic, or data shape, keep those files next to the feature rather than moving them into global shared folders too early.

Good local grouping:

```text
routes/gas/
- index.tsx
- quiz.tsx
- picture.ts
- game/config.ts
- game/utils.ts
- game/types.ts
```

Use small local components inside a route when they describe real UI sections and reduce noise in the main component. Keep them unexported unless another file imports them.

Prefer centralized utility wrappers for cross-cutting conventions. For class names, use the project's existing `cn`/`clsx`/`twMerge` style helper instead of manually joining strings or adding another class utility. If the project does not use such a helper, do not introduce one for a one-off change.

Example:

```ts
export const cn: typeof clsx = (...params) => {
  return twMerge(clsx(...params));
};
```

Use data/config objects for stable constants and content mappings. Prefer `satisfies` when it preserves literal safety without forcing broad explicit types. Use `as const` only for literal config objects or tuples where the project already uses that style and the literal narrowing is useful.

Example:

```ts
export default {
  GAME_DURATION: 30,
  SPAWN_INTERVAL: 2000,
  SPECIAL_OBJECT_SPAWN_CHANCE: 0.25,
} as const;
```

Prefer hook extraction for real external synchronization, not for every block of logic. Good hook candidates include timers, DOM event listeners, portals, scroll tracking, query-state synchronization, virtualized lists, and browser API integration. Pure render derivations can usually stay inline.

When effects are necessary, make them honest and cleanup-safe:

- Use `AbortController` for DOM listeners when useful
- Cancel throttled or debounced callbacks during cleanup
- Keep dependencies accurate instead of hiding stale closures
- Encapsulate repeated browser synchronization in one hook

Use existing utility libraries for non-trivial collection operations. If `lodash-es` or an equivalent utility library is already installed and used for that category of operation, prefer `groupBy`, `maxBy`, `shuffle`, `throttle`, or similar utilities over hand-rolled equivalents. Do not add or expand a utility dependency for trivial logic.

Use i18n at the boundary where copy is created. Keep message IDs explicit and stable, and avoid scattering hardcoded UI copy when the app already uses `react-intl` or a similar i18n layer.

For asset-heavy features, collect transformed imports into a local object and pass that object to shared rendering components. This keeps components focused on behavior and layout instead of import plumbing.

Example:

```ts
export default {
  hero: {
    picture: heroPicture,
    placeholder: heroPlaceholder,
  },
};
```

Prefer direct JSX over premature configuration. Use config/data when the thing is data-like; use JSX when the thing is layout, accessibility, or interaction.

## Simplicity Of Functions And Components

Functions and components should do one understandable job. Prefer straightforward control flow and clear data flow.

Use early returns to reduce nesting when that matches the local style. Avoid packing too much logic into chained expressions if a short named local variable would be clearer.

Split code only when the split improves comprehension or reuse. A large component is not automatically worse than a fragmented component tree with unclear boundaries.

Before extracting a component, check whether it has a real identity in the UI, reduces meaningful duplication, or isolates a distinct responsibility. If not, keep it inline.

## Dependencies And Utilities

Use existing dependencies consistently. If the project already uses a utility library for a category of operation, prefer that over writing a custom equivalent.

When a task depends on unfamiliar, subtle, or changing library behavior, study the library's documentation through Context7 MCP if it is available. Resolve the library ID, then query the relevant docs before relying on assumptions about APIs, recommended patterns, edge cases, or migration behavior.

Use Context7 especially when:

- Introducing a new usage of an installed package
- Debugging behavior from a third-party component or hook
- Refactoring integration code around a package API
- Replacing custom code with a library utility
- Touching code copied from or modeled after `node_modules`

For trivial or already-established local package usage, follow nearby code first and do not pause for docs unless something is unclear or risky. Do not inspect package internals as the primary source of truth unless docs are unavailable or insufficient. Prefer documented public APIs over implementation details.

Do not add a new dependency for a trivial helper. Do not import a large utility surface when a direct import or native API is available and matches the project's bundle practices.

## When Reviewing Code

When the user asks for a frontend code review, prioritize defects and risks over style commentary.

Report findings first, ordered by severity. Each finding should include:

- File and line reference when available
- What can break or regress
- Why the issue matters in user-visible or maintenance terms
- A concrete fix direction, without rewriting the whole file unless asked

Look especially for:

- Stale closures, incorrect effect dependencies, missing cleanup, and derived state stored unnecessarily
- Event logic hidden in effects when an event-level update would be safer
- Unsafe type assertions, over-broad types, or validation gaps at API/form boundaries
- New exported helpers, types, config layers, or component APIs that do not have real consumers
- Behavior changes hidden inside a refactor
- Accessibility regressions in interactive JSX
- Tests that no longer cover the changed behavior or are missing for risky logic

Do not fill the review with generic praise. If there are no findings, say so explicitly and mention any residual testing gaps or files you did not inspect.

## Review Checklist

Before finishing, check:

- Does the code match the dominant style of nearby frontend files?
- If the user asked for a broad refactor, did I propose the refactor and wait for confirmation before editing?
- If the user asked for a concrete scoped refactor, did I keep the implementation narrow instead of turning it into a broader rewrite?
- If this was a refactor, did I focus on simplification and code quality rather than abstraction or unification for its own sake?
- Did I avoid unnecessary exports?
- Did I avoid one-off named types and unnecessary explicit return types?
- Did I avoid unsafe `as` assertions or keep them tightly justified?
- Did I reuse existing utilities instead of inventing new ones?
- When relying on unfamiliar or risky package behavior, did I check Context7 docs if available?
- Did I apply KISS and avoid overengineering for hypothetical future needs?
- For React, did I avoid unnecessary `useEffect`, refs, state, memoization, and inline component definitions?
- For JSX, did I prefer guarded rendering for one-sided conditionals unless a ternary was clearer or locally consistent?
- Is the resulting component or function simpler than the alternatives?
- If this was a code review, did I lead with concrete findings and avoid generic praise?
- If I noticed materially low-quality files while working, did I report the exact file paths and reasons?
