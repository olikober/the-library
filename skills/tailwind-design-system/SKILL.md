---
name: tailwind-design-system
description: Build scalable design systems with Tailwind CSS, design tokens, component libraries, and responsive patterns. Use when creating component libraries, implementing design systems, or standardizing UI patterns.
---

# Tailwind Design System

Build production-ready design systems with Tailwind CSS.

## When to Use

- Creating a component library with Tailwind
- Implementing design tokens and theming
- Building responsive and accessible components
- Standardizing UI patterns across a codebase
- Migrating to or extending Tailwind CSS

## Workflow

1. Identify the design system scope (tokens, components, or both)
2. Apply patterns from references as needed
3. Validate against best practices

## References (load only if needed)

- `references/design-tokens.md` — Config setup, token hierarchy, utilities, best practices
- `references/component-patterns.md` — CVA, compound components, forms, grids, animations, dark mode

## Output Contract

- Use semantic colors (`primary`, `secondary`) not raw hex values
- Compose variants with CVA for type safety
- Include accessibility (ARIA, focus states)
- Test both light and dark themes
