---
name: react-dev
version: 1.0.0
description: Type-safe React development with TypeScript. Use when building React components, typing hooks, handling events, or working with React 18-19 features including Server Components, TanStack Router, or React Router.
---

# React TypeScript Development

## When to Use
- Building typed React components with TypeScript
- Implementing generic components or custom hooks
- Typing event handlers, forms, refs
- Using React 19 features (Actions, Server Components, use())
- Router integration (TanStack Router, React Router)

## Quick Patterns

| Pattern | Reference |
|---------|-----------|
| Component props extension | [references/hooks.md](references/hooks.md) |
| Event handlers | [references/event-handlers.md](references/event-handlers.md) |
| React 19 migration | [references/react-19-patterns.md](references/react-19-patterns.md) |
| Generic components | [examples/generic-components.md](examples/generic-components.md) |
| Server Components | [examples/server-components.md](examples/server-components.md) |

## Core Rules

**ALWAYS:**
- Specific event types (MouseEvent, ChangeEvent, etc)
- Explicit useState for unions/null
- ComponentPropsWithoutRef for native element extension
- Discriminated unions for variant props

**NEVER:**
- any for event handlers
- JSX.Element for children (use ReactNode)
- forwardRef in React 19+

## References (Load Only If Needed)

- [references/hooks.md](references/hooks.md) — useState, useRef, useReducer, useContext
- [references/event-handlers.md](references/event-handlers.md) — All event types, generic handlers
- [references/react-19-patterns.md](references/react-19-patterns.md) — useActionState, use(), useOptimistic
- [references/tanstack-router.md](references/tanstack-router.md) — TanStack Router typed routes
- [references/react-router.md](references/react-router.md) — React Router v7 patterns
- [examples/generic-components.md](examples/generic-components.md) — Table, Select, List patterns
- [examples/server-components.md](examples/server-components.md) — Async components, Server Actions
