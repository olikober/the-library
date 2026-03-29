---
name: mermaid-diagrams
description: Create software diagrams using Mermaid syntax. Use for class diagrams, sequence diagrams, flowcharts, ERDs, C4 diagrams, state diagrams, and more. Triggers "diagram", "visualize", "model", "show the flow", "system architecture", database design.
---

# Mermaid Diagramming

Create professional software diagrams using Mermaid's text-based syntax.

## When to use
- Users need to visualize system architecture
- Database schema design or documentation
- API flows, sequence diagrams
- Domain modeling, class diagrams
- User journeys, process flows

## Diagram Type Selection

| Use Case | Diagram Type |
|----------|--------------|
| Domain modeling, OOP | `classDiagram` |
| API flows, interactions | `sequenceDiagram` |
| Processes, decisions | `flowchart` |
| Database schemas | `erDiagram` |
| Architecture (C4) | `C4Context`, `C4Container` |
| State machines | `stateDiagram` |
| Timelines | `ganttChart` |

## Basic Syntax

```mermaid
diagramType
  definition content
```

- First line declares type (e.g., `classDiagram`)
- Use `%%` for comments
- Unknown words break diagrams

## Workflow
1. Identify what to visualize (data flow, structure, process)
2. Select appropriate diagram type from table above
3. Load reference for specific syntax
4. Create focused diagram (split complex ones)

## References (load only when needed)
- `references/class-diagrams.md` - Domain modeling, relationships
- `references/sequence-diagrams.md` - Actors, messages, loops
- `references/flowcharts.md` - Nodes, decisions, subgraphs
- `references/erd-diagrams.md` - Tables, cardinality
- `references/c4-diagrams.md` - Architecture levels
- `references/architecture-diagrams.md` - Cloud, CI/CD
- `references/advanced-features.md` - Themes, styling

## Quick Examples

**Class:** `classDiagram User <|-- Admin`
**Sequence:** `User->>API: Request`
**Flowchart:** `A --> B`
**ERD:** `USER ||--o{ ORDER`

## Best Practices
- Keep focused: one diagram per concept
- Use meaningful names for self-documenting diagrams
- Add titles for context
- Validate in Mermaid Live Editor
