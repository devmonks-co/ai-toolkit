# D2 Syntax Reference

**Official docs:** https://d2lang.com/tour/intro

## Table of Contents

1. [Shapes and connections](#shapes-and-connections)
2. [Containers](#containers)
3. [Styling and classes](#styling-and-classes)
4. [Configuration](#configuration)
5. [Icons](#icons)
6. [Diagram-type patterns](#diagram-type-patterns)
7. [CLI reference](#cli-reference)
8. [Design best practices](#design-best-practices)

---

## Shapes and connections

Docs: https://d2lang.com/tour/shapes, https://d2lang.com/tour/connections

```d2
server: Backend Server
db: Users {shape: cylinder}
decision: Valid? {shape: diamond}
start: Start {shape: oval}
queue: Jobs {shape: queue}
cloud: AWS {shape: cloud}
doc: Spec {shape: page}

# Markdown label
note: |md
  **Bold** and *italic* text
|

# Connections
a -> b: label            # directed
a -- b                   # undirected
a <-> b                  # bidirectional
a -> b -> c              # chained
```

All shape types: `rectangle`, `square`, `page`, `parallelogram`, `document`, `cylinder`, `queue`, `package`, `step`, `callout`, `stored_data`, `person`, `diamond`, `oval`, `circle`, `hexagon`, `cloud`, `text`, `image`, `sql_table`, `class`, `sequence_diagram`.

Arrowhead shapes (for ERD cardinality): `triangle`, `arrow`, `diamond`, `circle`, `cf-one`, `cf-one-required`, `cf-many`, `cf-many-required`. Set via:

```d2
a -> b: {
  source-arrowhead: {shape: cf-many; label: "*"}
  target-arrowhead: {shape: cf-one-required; label: "1"}
}
```

---

## Containers

Docs: https://d2lang.com/tour/containers

```d2
backend: Backend {
  api: REST API
  worker: Worker
  api -> worker
}

# Cross-container connection
frontend.app -> backend.api: HTTP
```

---

## Styling and classes

Docs: https://d2lang.com/tour/style, https://d2lang.com/tour/vars, https://d2lang.com/tour/classes

```d2
# --- Variables and classes ---
vars: {
  primary: "#0078D4"
}

classes: {
  service: {
    style.fill: "#FFFFFF"
    style.stroke: "#232F3E"
    style.border-radius: 8
    style.shadow: true
  }
}

box: {style.fill: ${primary}}
api: {class: service}

# --- All style properties ---
x: Styled {
  style.fill: "#FF6B6B"
  style.stroke: "#C92A2A"
  style.stroke-width: 2
  style.stroke-dash: 5          # dashed border
  style.border-radius: 8
  style.shadow: true
  style.3d: true
  style.multiple: true
  style.opacity: 0.9
  style.font-size: 16
  style.font-color: "#333"
  style.bold: true
  style.animated: true          # connections only
  style.fill-pattern: dots      # shapes only: dots, lines, grain, paper
}
```

---

## Configuration

Docs: https://d2lang.com/tour/layouts, https://d2lang.com/tour/vars/#d2-config

All configuration belongs in `d2-config` so `.d2` files are self-contained:

```d2
vars: {
  d2-config: {
    layout-engine: elk       # dagre | elk | tala
    theme-id: 3              # 0=neutral, 1=grey, 3=flagship, 8=colorblind, 200=dark-mauve, 300=terminal
    dark-theme-id: 200
    sketch: false
    pad: 100
  }
}

direction: right               # right | left | up | down
```

| Engine | Best for | Notes |
|--------|----------|-------|
| `dagre` | Flat graphs, no nesting | Default. Fast. |
| `elk` | Nested containers, large graphs | Deterministic spacing. Use this for most diagrams. |
| `tala` | Presentation-quality | Proprietary (Terrastruct license). Supports per-container `direction`. |

- `direction: right` — architecture diagrams (left-to-right flow)
- `direction: down` — flowcharts, process diagrams (top-to-bottom)

---

## Icons

Docs: https://d2lang.com/tour/icons

```d2
server: Backend {
  icon: https://icons.terrastruct.com/essentials%2F112-server.svg
}

# Standalone icon (no border)
logo: {
  shape: image
  icon: https://icons.terrastruct.com/dev%2Fdocker.svg
}
```

### Icon resolution

Icons are resolved dynamically — not from a hardcoded list. See `${CLAUDE_SKILL_DIR}/icons.md` for the full resolution guide.

**Priority chain:** Terrastruct → jsdelivr (@mdi/svg) → official provider logo → web search.

Always validate every icon URL with `curl -sL -o /dev/null -w '%{http_code}' "<url>"` before using. Any publicly accessible image URL (SVG, PNG, JPG) works.

---

## Diagram-type patterns

Docs: https://d2lang.com/tour/sql-tables, https://d2lang.com/tour/sequence-diagrams, https://d2lang.com/tour/uml-classes, https://d2lang.com/tour/grid-diagrams

### System architecture

```d2
direction: right

vars: {
  d2-config: {
    layout-engine: elk
    theme-id: 3
  }
}

classes: {
  client: {
    style.fill: "#E3F2FD"
    style.stroke: "#1565C0"
    style.border-radius: 8
    style.shadow: true
  }
  service: {
    style.fill: "#FFFFFF"
    style.stroke: "#232F3E"
    style.border-radius: 8
    style.shadow: true
  }
}

clients: Clients {
  style.fill: "#F5F5F5"
  style.stroke: "#9E9E9E"
  style.border-radius: 10

  web: Web App {
    class: client
    icon: https://cdn.jsdelivr.net/npm/@mdi/svg/svg/monitor.svg
  }
}

infra: AWS {
  style.fill: "#FFF8E1"
  style.stroke: "#FF8F00"
  style.stroke-width: 2
  style.border-radius: 10
  style.stroke-dash: 5

  api: API Server {
    class: service
    icon: https://icons.terrastruct.com/essentials%2F112-server.svg
  }
  db: PostgreSQL {class: service; shape: cylinder}

  api -> db: read/write
}

clients.web -> infra.api: HTTPS
```

### ERD (sql_table)

```d2
users: {
  shape: sql_table
  id: int {constraint: primary_key}
  email: varchar(255) {constraint: unique}
  name: text
}

posts: {
  shape: sql_table
  id: int {constraint: primary_key}
  author_id: int {constraint: foreign_key}
  title: varchar(255)
}

posts.author_id -> users.id
```

Constraints: `primary_key`, `foreign_key`, `unique`. Multiple: `{constraint: [primary_key; unique]}`. Quote SQL reserved words: `"select": varchar`.

### Flowchart

```d2
direction: down

start: Start {shape: oval}
input: Get Input {shape: parallelogram}
check: Valid? {shape: diamond}
process: Process {shape: rectangle}
error: Error {shape: rectangle; style.fill: "#ffcccc"}
end: End {shape: oval}

start -> input -> check
check -> process: Yes
check -> error: No
error -> input: Retry
process -> end
```

### Sequence diagram

```d2
seq: {
  shape: sequence_diagram
  alice: Alice
  bob: Bob
  server: API Server

  alice -> bob: Hello
  bob -> server: GET /data
  server -> bob: 200 OK
  bob -> alice: Here's the data
}
```

Spans (activation): connect to `actor.NestedName`. Groups: container inside the sequence with internal connections only.

### Class diagram

```d2
Animal: {
  shape: class
  +name: string
  +age: int
  -id: int
  +getName(): string
}

Dog: {
  shape: class
  +breed: string
  +bark(): void
}

Animal <-> Dog: inherits
```

Visibility: `+` public, `-` private, `#` protected.

### Grid layout

```d2
grid: {
  grid-rows: 3
  grid-columns: 4
  grid-gap: 10
  cell1; cell2; cell3; cell4
}
```

---

## CLI reference

Docs: https://d2lang.com/tour/install, https://man.d2lang.com

```bash
d2 input.d2 output.svg              # compile to SVG
d2 input.d2 output.png              # compile to PNG
d2 --sketch input.d2 output.svg     # hand-drawn style
d2 fmt input.d2                     # format the file
```

Prefer in-file `d2-config` over CLI flags for theme and layout engine.

---

## Design best practices

### Nesting

- **Max 2 levels deep.** `aws.compute.api` is fine. `aws.compute.backend.api.handler` is not.
- **Group peers, not layers.** Don't nest to show "runs on" — use a connection instead.
- **Never use `shape: cloud` as a container.** Use a rectangle with `style.stroke-dash: 5`.

### Styling

- **Define 2–4 classes** (`client`, `service`, `storage`, `external`) and reuse them.
- **Containers:** light `fill`, darker matching `stroke`, `border-radius: 10`, `stroke-dash: 5` for cloud/infra boundaries.
- **Leaf nodes:** `shadow: true`, `border-radius: 8`.
- **Color sparingly.** Use fills to distinguish groups. Within a group, let the theme handle colors.

### Labels

- **Node labels: 1–3 words.** "API Server", not "Main Backend REST API Server".
- **Connection labels: 1–2 words.** "HTTPS", "read/write". Omit when the relationship is obvious.
- **Container labels: single noun.** "AWS", "Clients", "Backend".

### Connections

- **Minimize crossing.** Order nodes to match the visual flow direction.
- **Declare connections after all nodes** in a grouped section.
- **One style per meaning.** Solid = data flow, dashed = async/optional, animated = real-time.
