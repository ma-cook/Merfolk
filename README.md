# Merfolk Syntax Guide

Merfolk is a 3D relational diagram language used to represent codebase architecture. Merfolk markdown files are parsed by the 3D AST generator and rendered as interactive 3D diagrams in Hoverchart. This guide covers the full syntax specification.

Merfolk is embedded inside markdown files within ` ```merfolk ` fenced code blocks. A diagram title can optionally be added after the fence: ` ```merfolk "Title" `

---

## Table of Contents

1. [Diagram Declaration](#diagram-declaration)
2. [Comments](#comments)
3. [Node Types](#node-types)
4. [Node ID Rules](#node-id-rules)
5. [Connection Types](#connection-types)
6. [Labeled Connections](#labeled-connections)
7. [Face Connections](#face-connections)
8. [Node Properties](#node-properties)
9. [Flow Path Tracking](#flow-path-tracking)
   - [The `flowpath` Directive](#the-flowpath-directive)
   - [The `#tag` Syntax](#the-tag-syntax-on-connections)
   - [Combining Both Approaches](#combining-both-approaches)
10. [Nested Grouping](#nested-grouping)
11. [Container Grouping](#container-grouping)
12. [Complete Example](#complete-example)

---

## Diagram Declaration

Optionally declare a diagram with a title at the top of the Merfolk block:

```merfolk
graph3d "My Application Architecture"
```

or equivalently:

```merfolk
ast3d "My Application Architecture"
```

You can also use standalone `title:` and `description:` lines:

```merfolk
title: "My Application Architecture"
description: "Full system overview including services and data stores"
```

---

## Comments

Two comment styles are supported:

```merfolk
%% This is a comment or section header
// This is also a comment
```

Comments are ignored by the parser and can appear anywhere in the diagram.

---

## Node Types

Nodes are declared with an identifier and a bracket style that determines its type and 3D geometry.

### Bracket Styles

| Syntax | Geometry | Use Case |
| --- | --- | --- |
| `A{Type: name}` | Dodecahedron | Components (default container type) |
| `B[Type: name]` | Cube | Functions, hooks, classes, interfaces, variables, constants |
| `C[[Type: name]]` | Cube | Stores, data models |
| `D((Type: name))` | Tetrahedron | Services, external APIs |
| `E<Type: name>` | Cube | Libraries, datapaths |

The label inside the brackets uses the format `Type: Display Name` where `Type` determines the node's semantic role.

### Node Type Keywords

| Keyword(s) | Alias | Type | Color | Use Case |
| --- | --- | --- | --- | --- |
| `Component` | `comp` | Component | `#2196F3` | UI components, modules |
| `Function` | `func` | Function | `#4CAF50` | Functions, methods |
| `Hook` | — | Hook | `#E91E63` | React hooks, custom hooks |
| `Store` | — | Store | `#9C27B0` | Databases, data stores, state |
| `Service` | `svc` | Service | `#FF9800` | External services, APIs |
| `Library` | `lib` | Library | `#00BCD4` | External libraries, packages |
| `Module` | `mod` | Module | `#9C27B0` | Modules, boundaries |
| `Class` | — | Class | `#F44336` | Classes, data models |
| `Interface` | `iface` | Interface | `#00BCD4` | TypeScript interfaces, contracts |
| `Variable` | `var` | Variable | `#FFEB3B` | Variables, configuration values |
| `Constant` | `const` | Constant | `#795548` | Constants, enums |
| `Datapath` | `data` | Datapath | `#FF9800` | Data pipelines (no 3D object rendered) |
| `Endpoint` | `route` | Function | `#4CAF50` | API endpoints, routes |
| `Guard` | `middleware` | Function | `#4CAF50` | Auth guards, middleware |
| `Boundary` | — | Module | `#9C27B0` | Error boundaries, Suspense boundaries |
| `Model` | — | Store | `#9C27B0` | Database models, schemas |

Any unrecognized type keyword defaults to `Component` (dodecahedron).

---

## Node ID Rules

Node IDs may contain: `A-Z`, `a-z`, `0-9`, `_`, `/`, `.`, `-`

The `@` character is **reserved** as the face separator and cannot appear in node IDs. The scanner replaces leading `@` with `_` for npm scoped packages (e.g. `@scope/package` → `_scope/package`).

```merfolk
%% Package-style IDs
firebase-admin/app[Function: init]
eslint-plugin-react[Library: React Plugin]

%% Scoped package (scanner output)
_scope/package[Function: handler]
```

---

## Connection Types

| Syntax | Type | Arrow Style | Color | Use Case |
| --- | --- | --- | --- | --- |
| `A --> B` | Data Flow | Solid arrow `→` | `#4CAF50` | Data passing between nodes |
| `A -.-> B` | Control Flow | Dashed arrow | `#F44336` | Events, control signals, containment |
| `A --- B` | Association | Solid line | `#607D8B` | General relationships |
| `A == B` | Inheritance | Thick line | `#2196F3` | Inheritance, strong dependencies |
| `A *--> B` | Composition | Filled arrow | `#FF9800` | Ownership, composition |
| `A ..> B` | Dependency | Dotted arrow | `#9C27B0` | Imports, weak dependencies |

---

## Labeled Connections

Two label syntaxes are supported:

**Colon style (Merfolk-native):**
```merfolk
App --> DataService : "uses"
```

**Pipe style (Mermaid-compatible):**
```merfolk
App -->|"uses"| DataService
```

Labels are displayed on the 3D connection lines in the rendered diagram.

---

## Face Connections

Connect to a specific face of a node using the `@face` suffix.

```merfolk
A@front --> B@back : "direct connection"
C@top --> D@bottom : "vertical flow"
```

Available faces depend on the node geometry:

| Geometry | Faces |
| --- | --- |
| **Cube** | `front`, `back`, `top`, `bottom`, `left`, `right` |
| **Dodecahedron** | `face_0` through `face_11` (12 faces) |
| **Tetrahedron** | `front`, `left`, `right`, `bottom` |

---

## Node Properties

Attach inline properties after the node declaration, or use a multi-line property block on the following line.

### Inline Properties

```merfolk
App{Component: Main Application} {color: "blue", scale: "2,1,1"}
DataService[Function: Data Processing] {color: "#4CAF50"}
```

### Multi-Line Property Blocks

```merfolk
UserService{Component: User Service}
{
  codeFilePath: "src/services/userService.ts"
  fileSize: "2.4KB"
  exports: "UserService, createUser"
}
```

Property values support strings, numbers, and arrays:

```merfolk
NodeA[Function: Handler]
{
  color: "#4CAF50"
  opacity: 0.7
  position: [0, 5, 0]
}
```

### Standard Properties

| Property | Type | Description |
| --- | --- | --- |
| `color` | string | Hex color or CSS color name |
| `opacity` | number | 0.0 to 1.0 |
| `scale` | string | Comma-separated x,y,z values |
| `position` | array | [x, y, z] coordinates |
| `codeFilePath` | string | Source file path (emitted by scanner) |
| `fileSize` | string | File size (emitted by scanner) |
| `exports` | string | Exported symbols (emitted by scanner) |

---

## Flow Path Tracking

Flow paths let you define and trace complete data paths that span multiple nodes across your application — not just individual point-to-point connections.

### The `flowpath` Directive

Define a named, multi-hop data path in a single line. This auto-creates tagged connections between each adjacent pair of nodes:

```merfolk
flowpath "userDataFlow" : A --> B --> C --> D
```

This creates 3 connections (A→B, B→C, C→D), all tagged with the `userDataFlow` identifier so the entire path can be queried and traced as a unit.

Full syntax options:

```merfolk
%% Basic flow path
flowpath "name" : NodeA --> NodeB --> NodeC

%% With a custom arrow type (applies to all connections in the path)
flowpath "eventPipeline" (-.->): Input --> Transform --> Output

%% With a description
flowpath "requestLifecycle" : Client --> API --> DB --> API --> Client : "full request cycle"
```

If a `flowpath` references a connection that already exists between two adjacent nodes, it tags the existing connection instead of creating a duplicate.

### The `#tag` Syntax on Connections

Tag individual connections with one or more flow path names using `#`:

```merfolk
A --> B : "payload" #userDataFlow
B --> C #userDataFlow #auditTrail
C --> D #auditTrail
```

This is useful when you want to manually compose flow paths from existing connections rather than auto-generating them.

### Combining Both Approaches

You can freely mix `flowpath` directives with `#tag` connections:

```merfolk
%% Nodes
UI{Component: User Interface}
API[Function: API Handler]
Auth[Function: Auth Service]
DB[[Store: Database]]
Cache[Function: Cache Layer]

%% Explicit connections
UI --> API : "request" #userFlow
API --> Auth : "validate"

%% Flow path reuses existing UI-->API connection, creates the rest
flowpath "userFlow" : UI --> API --> Auth --> DB

%% A separate flow path through the cache layer
flowpath "cachedRead" : UI --> API --> Cache --> DB
```

---

## Nested Grouping

The parser automatically nests functions inside their connected components based on connection relationships:

```merfolk
%% Functions connected to a component are automatically nested inside it
UserService{Component: User Service}
validateUser[Function: User Validation]
authenticateToken[Function: Token Authentication]

validateUser --> UserService : "validates"
authenticateToken --> UserService : "authenticates"
```

**Result:**
- Functions become child nodes inside their connected component
- Parent components scale up to contain children
- Parent-child relationships are tracked in the data structure

The scanner also emits file container nodes (e.g. `App_file[Function: App]{codeFilePath: "..."}`) that wrap a file's exports, connected via `-.->` : `"contains"`.

---

## Container Grouping

The renderer automatically groups root nodes by type into labeled containers: **Services**, **Hooks**, **Stores**, **Backend**, **Libraries**, **Utilities**, **Workers**, **Shaders**, **Classes**, **Interfaces**, **Variables**, and **Constants**.

---

## Complete Example

The following example uses all node types and connection features together:

```merfolk
graph3d "E-Commerce Platform"

%% Components (Dodecahedrons — can become containers for functions)
App{Component: Main Application}
UI{Component: User Interface}
API{Component: API Gateway}

%% Functions (Cubes — can be nested in components)
processData[Function: Data Processing]
validateUser[Function: User Validation]
renderUI[Function: UI Rendering]

%% Hooks (Cubes)
useAuth[Hook: useAuth]
useForm[Hook: useForm]
useTheme[Hook: useTheme]

%% Stores (Cubes via double brackets)
UserDB[[Store: User Database]]
ConfigDB[[Store: Configuration Store]]

%% External Services (Tetrahedrons)
PaymentAPI((Service: Payment Gateway))
EmailService((Service: Email Provider))

%% Libraries (Cubes via angle brackets)
ReactLib<Library: React>
ExpressLib<Library: Express.js>

%% Classes and Interfaces
UserModel[[Class: User Model]]
IAuthService[[Interface: IAuthService]]

%% Variables and Constants
APP_VERSION[Constant: APP_VERSION]
maxRetries[Variable: maxRetries]

%% Labeled connections
processData --> API : "processes requests"
validateUser --> API : "validates tokens"
renderUI --> UI : "renders components"

%% Hook connections
useAuth --> UI : "provides auth state"
useForm --> UI : "manages form state"
useTheme --> UI : "provides theme"

%% Component connections
App --> UI : "renders"
App --> API : "calls"
API --> UserDB : "queries"
API --> PaymentAPI : "payment processing"
API -.-> EmailService : "notifications"

%% Composition and inheritance
UserModel *--> UserDB : "owned by"
AdminUI == UI : "extends"

%% Dependency
processData ..> ReactLib : "imports"

%% Library dependencies
UI --> ReactLib : "uses"
API --> ExpressLib : "uses"

%% Face-specific connections
App@front --> UI@back : "direct rendering"
API@top --> UserDB@bottom : "data flow"

%% Flow paths
flowpath "requestCycle" : App --> API --> UserDB : "full request"
flowpath "cachedRead" : App --> API --> ConfigDB

%% Multi-line properties
EmailService
{
  codeFilePath: "src/services/email.ts"
  fileSize: "3.1KB"
}
```

---

## Output

- Ensure the markdown file contains all relevant nodes and connections to represent the codebase architecture accurately.
- Validate the syntax to ensure compatibility with the 3D AST generator.
- Duplicate node IDs are silently skipped with a console warning.
- References to undefined node IDs in connections produce a validation warning but are non-fatal.
