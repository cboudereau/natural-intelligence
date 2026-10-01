---
name: c4-graph
description: "Use when drawing or converting a software architecture diagram following the C4 model — system context, container, or landscape — for GitLab markdown, VS Code preview, or a README, from a text description, a DOT digraph, or a draw.io file or model. Also use when the user mentions C4, mermaid architecture diagram, crossing arrows, diagram unreadable on dark theme, or asks to convert a .drawio file or a draw.io SVG export."
---
# C4 graph

## When to use
- User asks for an architecture diagram, a C4 diagram, or a system landscape
- User wants a diagram that renders in GitLab markdown or VS Code preview
- User provides a DOT digraph, a draw.io file, or a pasted draw.io model to convert
- Arrows cross heavily, or a diagram is unreadable on a dark theme

## Overview

Produce C4 diagrams as a mermaid `flowchart TB` inside markdown: dagre gives a layered (Sugiyama) layout that minimises crossing arrows, and GitLab plus VS Code render it natively. Native mermaid `C4Context` has no layout engine — notation appendix only, never the deliverable.

Reference: [mermaid-rules.md](mermaid-rules.md). Read it before the first line of diagram code.
Reference: [inputs.md](inputs.md). Read it when converting from a source or validating output.

## Rules

1. Read [mermaid-rules.md](mermaid-rules.md) first; copy its init block and class definitions verbatim.
2. Declare actors first, boundaries in flow order, data stores last inside their boundary — declaration order is the placement hint.
3. Respect C4 levels — one level per diagram, 15 elements at most. Level 1 (System Context): persons and systems only, containers collapsed into their owning system. Level 2 (Container): one diagram per in-focus system — its containers plus direct neighbours. Over 15 elements or mixed levels: split per the Levels section of [mermaid-rules.md](mermaid-rules.md), all diagrams in one markdown file, context first.
4. Draw one-way arrows with verb labels ("Notifies", "Reads"); two-way traffic is two labelled arrows.
5. In convert mode, extract per [inputs.md](inputs.md); the output edge set must equal the source's. Report dangling edges, never drop them silently.
6. Render the result before delivering when a renderer exists ([inputs.md](inputs.md) has the command); never ship an unrendered diagram.
7. Keep sequence flows out of the C4 graph — separate mermaid `sequenceDiagram` blocks.

## Boundaries

This skill never lays out with native `C4Context` and never produces draw.io-editable files; when draw.io round-trip or Google Drive preview is required, say so plainly — that needs a different pipeline. Commits and MR text belong to [`git-conventions`](../git-conventions/SKILL.md); multi-session planning to [`plan`](../plan/SKILL.md).
