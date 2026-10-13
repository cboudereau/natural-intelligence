# draw.io C4 rules

Reference file of the [`c4-graph`](SKILL.md) skill. Read it before writing any diagram code.

## File shape

One `.drawio` file per diagram, uncompressed XML, one page. The human exports it to `<name>.drawio.svg` and markdown embeds that file: `![System Context — Shop](shop-context.drawio.svg)`.

```xml
<mxfile host="ni">
  <diagram name="System Context — Shop" id="context">
    <mxGraphModel background="#ffffff" grid="1" gridSize="10" page="0">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <!-- elements, boundaries, then relationships -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

`background="#ffffff"` keeps the canvas white in VS Code's dark theme and in the exported SVG. Ids are short slugs (`customer`, `web`, `e1`); every `source` and `target` names an existing vertex id.

## Elements

Every element is an `<object>` carrying the C4 metadata, wrapping one `<mxCell>`. Styles come from the draw.io C4 palette, copied verbatim:

| C4 element | Style | Size |
|---|---|---|
| Person | `html=1;fontSize=11;dashed=0;whiteSpace=wrap;fillColor=#083F75;strokeColor=#06315C;fontColor=#ffffff;shape=mxgraph.c4.person2;align=center;metaEdit=1;points=[[0.5,0,0],[1,0.5,0],[1,0.75,0],[0.75,1,0],[0.5,1,0],[0.25,1,0],[0,0.75,0],[0,0.5,0]];resizable=0;` | 200 × 180 |
| External person | Person style with `fillColor=#6C6477;strokeColor=#4D4D4D` | 200 × 180 |
| Software system | `rounded=1;whiteSpace=wrap;html=1;labelBackgroundColor=none;fillColor=#1061B0;fontColor=#ffffff;align=center;arcSize=10;strokeColor=#0D5091;metaEdit=1;resizable=0;` + `points=[[0.25,0,0],[0.5,0,0],[0.75,0,0],[1,0.25,0],[1,0.5,0],[1,0.75,0],[0.75,1,0],[0.5,1,0],[0.25,1,0],[0,0.75,0],[0,0.5,0],[0,0.25,0]];` | 240 × 120 |
| External system | Software system style with `fillColor=#8C8496;strokeColor=#736782` | 240 × 120 |
| Container | Software system style with `fontSize=11;fillColor=#23A2D9;strokeColor=#0E7DAD` | 240 × 120 |
| Store / queue | `shape=cylinder3;size=15;whiteSpace=wrap;html=1;boundedLbl=1;rounded=0;labelBackgroundColor=none;fillColor=#23A2D9;fontSize=12;fontColor=#ffffff;align=center;strokeColor=#0E7DAD;metaEdit=1;points=[[0.5,0,0],[1,0.25,0],[1,0.5,0],[1,0.75,0],[0.5,1,0],[0,0.75,0],[0,0.5,0],[0,0.25,0]];resizable=0;` | 240 × 120 |
| Boundary | `rounded=1;fontSize=11;whiteSpace=wrap;html=1;dashed=1;arcSize=20;fillColor=none;strokeColor=#666666;fontColor=#333333;labelBackgroundColor=none;align=left;verticalAlign=bottom;labelBorderColor=none;spacingTop=0;spacing=10;dashPattern=8 4;metaEdit=1;rotatable=0;perimeter=rectanglePerimeter;noLabel=0;labelPadding=0;allowArrows=0;connectable=0;expand=0;recursiveResize=0;editable=1;pointerEvents=0;absoluteArcSize=1;` + the same `points=` as Software system | fits its children |
| Relationship | `endArrow=blockThin;html=1;fontSize=10;fontColor=#404040;strokeWidth=1;endFill=1;strokeColor=#828282;elbow=vertical;metaEdit=1;endSize=14;startSize=14;jumpStyle=arc;jumpSize=16;rounded=0;edgeStyle=orthogonalEdgeStyle;` | `<mxGeometry width="240" relative="1" as="geometry"/>` |

Blue marks the system in focus and its containers; grey marks external persons and systems. `jumpStyle=arc` draws a line jump where two relationships cross.

Object attributes, with `placeholders="1"` on every object:

| Element | Attributes | `label` (HTML, XML-escaped in the file) |
|---|---|---|
| Person | `c4Name`, `c4Type="Person"`, `c4Description` | `<font style="font-size: 16px"><b>%c4Name%</b></font><div>[%c4Type%]</div><br><div><font style="font-size: 11px"><font color="#cccccc">%c4Description%</font></div>` |
| Software system | `c4Name`, `c4Type="Software System"`, `c4Description` | same as Person |
| Container, store | `c4Name`, `c4Type="Container"`, `c4Technology`, `c4Description` | `<font style="font-size: 16px"><b>%c4Name%</b></font><div>[%c4Type%: %c4Technology%]</div><br><div><font style="font-size: 11px"><font color="#E6E6E6">%c4Description%</font></div>` |
| Boundary | `c4Name`, `c4Type="SystemScopeBoundary"`, `c4Application="Software System"` | `<font style="font-size: 16px"><b><div style="text-align: left">%c4Name%</div></b></font><div style="text-align: left">[%c4Application%]</div>` |
| Relationship | `c4Type="Relationship"`, `c4Description` (the verb), optional `c4Technology` | `<div style="text-align: left"><div style="text-align: center"><b>%c4Description%</b></div></div>`; with technology add `<div style="text-align: center">[%c4Technology%]</div>` before the last `</div>` |

Escape the label for XML: `<` becomes `&lt;`, `>` becomes `&gt;`, `"` becomes `&quot;`. Descriptions stay under three lines.

## Boundaries and layout

A boundary is a vertex; its members carry `parent="<boundary id>"` and coordinates relative to the boundary's top-left corner. Relationships always carry `parent="1"`.

Claude does not lay out: the human runs Arrange > Layout > Vertical Flow after opening the file, which places the vertices, routes the relationships, and makes the jumps appear. Give every vertex a distinct, non-overlapping position so the file opens readable — persons on top, the in-focus system or boundary below, other systems at the bottom, 80 px apart — and size a boundary to wrap its members with 60 px at the bottom for the title.

## C4 levels and splitting

One level per diagram, 15 elements at most. One `.drawio` file per level, named `<system>-context`, `<system>-containers`, `landscape`; the markdown lists them context first.

| Level | Diagram name | Shows | Never shows |
|---|---|---|---|
| 1 — System Context | `System Context — <system>` | Persons, the in-focus system (blue), the systems it talks to (grey) | Containers, internals |
| 2 — Container | `Containers — <system>` | The containers of one in-focus system inside its boundary, plus the persons and systems that touch them directly | Containers of other systems |
| Landscape (optional) | `System Landscape` | All systems and persons, no containers | Containers |

Collapse rule for Level 1 and the landscape: every container edge rolls up to its owning system. Deduplicate rolled-up edges per (source, target); combine distinct labels with `, ` — never ` / `, a single verb label may itself contain it. Drop self-edges produced by the roll-up.

Ownership is required for collapsing: container → system comes from boundaries (draw.io parent cell, DOT `cluster`, or the user). Without it, ask — never guess ownership.

## Skeleton

Three vertices, one boundary, two relationships; the first label shows the escaping in full, the others are abbreviated:

```xml
<object placeholders="1" c4Name="Customer" c4Type="Person" c4Description="Buys products" label="&lt;font style=&quot;font-size: 16px&quot;&gt;&lt;b&gt;%c4Name%&lt;/b&gt;&lt;/font&gt;&lt;div&gt;[%c4Type%]&lt;/div&gt;&lt;br&gt;&lt;div&gt;&lt;font style=&quot;font-size: 11px&quot;&gt;&lt;font color=&quot;#cccccc&quot;&gt;%c4Description%&lt;/font&gt;&lt;/div&gt;" id="customer">
  <mxCell style="html=1;fontSize=11;dashed=0;whiteSpace=wrap;fillColor=#083F75;strokeColor=#06315C;fontColor=#ffffff;shape=mxgraph.c4.person2;align=center;metaEdit=1;points=[[0.5,0,0],[1,0.5,0],[1,0.75,0],[0.75,1,0],[0.5,1,0],[0.25,1,0],[0,0.75,0],[0,0.5,0]];resizable=0;" vertex="1" parent="1">
    <mxGeometry x="20" y="0" width="200" height="180" as="geometry"/>
  </mxCell>
</object>
<object placeholders="1" c4Name="Shop" c4Type="SystemScopeBoundary" c4Application="Software System" label="<boundary label, escaped>" id="shop">
  <mxCell style="<boundary style>" vertex="1" parent="1">
    <mxGeometry x="0" y="260" width="320" height="380" as="geometry"/>
  </mxCell>
</object>
<object placeholders="1" c4Name="Web App" c4Type="Container" c4Technology="ASP.NET" c4Description="Storefront" label="<container label, escaped>" id="web">
  <mxCell style="<container style>" vertex="1" parent="shop">
    <mxGeometry x="40" y="40" width="240" height="120" as="geometry"/>
  </mxCell>
</object>
<object placeholders="1" c4Name="Orders" c4Type="Container" c4Technology="PostgreSQL" c4Description="Order history" label="<container label, escaped>" id="db">
  <mxCell style="<store style>" vertex="1" parent="shop">
    <mxGeometry x="40" y="200" width="240" height="120" as="geometry"/>
  </mxCell>
</object>
<object placeholders="1" c4Type="Relationship" c4Description="Orders" label="<relationship label, escaped>" id="e1">
  <mxCell style="<relationship style>" edge="1" parent="1" source="customer" target="web">
    <mxGeometry width="240" relative="1" as="geometry"/>
  </mxCell>
</object>
<object placeholders="1" c4Type="Relationship" c4Description="Stores" label="<relationship label, escaped>" id="e2">
  <mxCell style="<relationship style>" edge="1" parent="1" source="web" target="db">
    <mxGeometry width="240" relative="1" as="geometry"/>
  </mxCell>
</object>
```
