# Mermaid C4 rules

Reference file of the [`c4-graph`](SKILL.md) skill. Read it before writing any diagram code.

## File shape

A markdown file; the diagram sits in a ` ```mermaid ` block wrapped in a white div so dark hosts keep a light canvas:

```markdown
# <Diagram title>

<div style="background:#ffffff; padding:16px">

​```mermaid
%%{init: ...}%%
flowchart TB
    ...
​```

</div>
```

If the host strips the div (markdown security), the init `background` still keeps the diagram legible.

## Init block — copy verbatim

```text
%%{init: {"theme": "base", "themeVariables": {"background": "#ffffff", "lineColor": "#333333", "primaryColor": "#1061B0", "primaryTextColor": "#333333", "primaryBorderColor": "#0D5091", "textColor": "#333333", "edgeLabelBackground": "#ffffff", "personBkg": "#083F75", "personBorder": "#06315C"}, "flowchart": {"htmlLabels": true, "curve": "basis"}}}%%
```

**The white-on-white trap**: `primaryTextColor` drives every `.label`, edge labels included. Set it dark (`#333333`); node text stays white through the classDef `color:#ffffff`, which wins via `!important`. Setting `primaryTextColor` white makes every edge label invisible on the white canvas.

Verified variable table:

| Target | Owner |
|---|---|
| Arrows | `lineColor` |
| Edge label text | `primaryTextColor` (via `.label`) |
| Edge label box | `edgeLabelBackground` |
| Node text | classDef `color` (overrides `.label`) |
| Boundary title | `style <subgraph> ... color:#333333` |
| Canvas | `background` + the div wrapper |

## Elements

| C4 element | Mermaid form | Class |
|---|---|---|
| Person | `id["👤 <b>Name</b><br/>[Person]<br/><i>description</i>"]` | `person` |
| Software system | `id["<b>Name</b><br/>[Software System]<br/><i>description</i>"]` | `system` |
| System in focus | same label form | `focus` |
| Store / queue | `id[("<b>Name</b><br/>[Container: Tech]<br/><i>description</i>")]` | `storeg` or `storeb` |
| Boundary | `subgraph id["Name [Scope]"]` ... `end`, nesting allowed | styled below |

Icons: unicode only (👤 for persons). `fa:fa-*` icons render only where Font Awesome CSS is loaded — not in GitLab or VS Code preview.

Labels: bold name, `[Type: Technology]`, italic description of at most 3 lines.

## Classes and boundary styles — copy verbatim

```text
    classDef person fill:#083F75,stroke:#06315C,color:#ffffff
    classDef focus fill:#00994D,stroke:#06753B,color:#ffffff
    classDef system fill:#1061B0,stroke:#0D5091,color:#ffffff
    classDef storeg fill:#00CC66,stroke:#0E7DAD,color:#ffffff
    classDef storeb fill:#23A2D9,stroke:#0E7DAD,color:#ffffff
    style <each-subgraph-id> fill:none,stroke:#666666,stroke-dasharray:8 4,color:#333333
```

Green (`focus`, `storeg`) marks the system in scope and its containers; blue everything else.

## C4 levels and splitting

One level per diagram, 15 elements at most. All diagrams of a model go in one markdown file, one `##` heading + mermaid block each, context first.

| Level | Title convention | Shows | Never shows |
|---|---|---|---|
| 1 — System Context | `System Context — <system>` | Persons, the in-focus system (green), the systems it talks to | Containers, internals |
| 2 — Container | `Containers — <system>` | The containers of one in-focus system (boundary subgraph), plus the persons and systems that touch them directly | Containers of other systems |
| Landscape (optional) | `System Landscape` | All systems and persons, no containers | Containers |

Collapse rule for Level 1 and the landscape: every container edge rolls up to its owning system. Deduplicate rolled-up edges per (source, target); combine distinct labels with `, ` — never ` / `, a single verb label may itself contain it. Drop self-edges produced by the roll-up.

Ownership is required for collapsing: container → system comes from boundaries (draw.io parent cell, DOT `cluster`, or the user). Without it, ask — never guess ownership.

Level 3 (components) follows the same rules on demand: one in-focus container per diagram.

## Layout

- `flowchart TB`; dagre does the crossing minimisation.
- Declaration order is the only placement control: actors first (top), then boundaries in flow order, stores declared last inside their boundary.
- Feedback loops (a downstream system calling back up) keep a few crossings — accept them; reorder declarations once at most.

## Skeleton

```text
flowchart TB
    customer["👤 <b>Customer</b><br/>[Person]<br/><i>Buys products</i>"]:::person
    subgraph shop["Shop [Platform]"]
        web["<b>Web App</b><br/>[Software System]<br/><i>Storefront</i>"]:::focus
        db[("<b>Orders</b><br/>[Container: PostgreSQL]")]:::storeg
    end
    billing["<b>Billing</b><br/>[Software System]"]:::system
    customer -->|Orders| web
    web -->|Stores| db
    web -->|Charges| billing
```
