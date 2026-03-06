# Merfolk Syntax Guide

Merfolk is a DSL for defining nodes and connections in diagrams. It is used by tools such as [3d-ast-generator](https://www.npmjs.com/package/3d-ast-generator) to generate 3D abstract syntax trees and architecture visualisations.

---

## Table of Contents

1. [Comments](#comments)
2. [Node Types](#node-types)
3. [Connection Types](#connection-types)
4. [Labeled Connections](#labeled-connections)
5. [Face Connections](#face-connections)
6. [Node Properties](#node-properties)
7. [Flow Path Tracking](#flow-path-tracking)
   - [The `flowpath` Directive](#the-flowpath-directive)
   - [The `#tag` Syntax](#the-tag-syntax-on-connections)
   - [Combining Both Approaches](#combining-both-approaches)
8. [Nested Grouping](#nested-grouping)
9. [Complete Example](#complete-example)

---

## Comments

Use `%%` to add comments. Comments are ignored by the parser.

```merfolk
%% This is a comment
App{Component: Main Application}  %% inline comment
```

---

## Node Types

Each node is declared with an identifier and a bracket style that determines its type and (in 3D tools) its geometry.

| Syntax | Type | 3D Geometry | Use Case |
|---|---|---|---|
| `A{Component: name}` | Component | Dodecahedron | Components, modules |
| `B[Function: name]` | Function | Cube | Functions, methods |
| `C[[Store: name]]` | Store | Cube | Databases, data stores |
| `D((Service: name))` | Service | Tetrahedron | External services, APIs |
| `E<Library: name>` | Library | Cube | External libraries |
| `F[Hook: name]` | Hook | Cube | React hooks, custom hooks |

The label inside the brackets uses the format `Type: Display Name`. The bracket style determines the node type; the label prefix (e.g. `Component:`, `Function:`, `Hook:`) further refines the type for tools that consume the syntax.

```merfolk
%% Node declarations
App{Component: Main Application}
processData[Function: Data Processing]
useAuth[Hook: useAuth]
UserDB[[Store: User Database]]
PaymentAPI((Service: Payment Gateway))
ReactLib<Library: React>
```

---

## Connection Types

| Syntax | Type | Style | Use Case |
|---|---|---|---|
| `A --> B` | Data Flow | Solid arrow | Data passing between nodes |
| `A -.-> B` | Control Flow | Dashed arrow | Events, control signals |
| `A --- B` | Association | Solid line | General relationships |
| `A == B` | Inheritance | Thick line | Inheritance, strong dependencies |

```merfolk
App --> DataService       %% data flow
App -.-> EventBus         %% control / event flow
ModuleA --- ModuleB       %% association
ChildClass == ParentClass %% inheritance
```

---

## Labeled Connections

Add a quoted label after a colon to describe a connection.

```merfolk
App --> UserInterface : "renders"
App --> DataService : "uses"
DataService --> Database : "queries"
```

---

## Face Connections

Connect to a specific face of a node using the `@face` suffix.

```merfolk
A@front --> B@back : "direct connection"
C@top --> D@bottom : "vertical flow"
```

Available faces depend on the node geometry:

- **Cubes**: `front`, `back`, `top`, `bottom`, `left`, `right`
- **Dodecahedrons**: `face_0` through `face_11`
- Other shapes expose context-appropriate face names.

---

## Node Properties

Attach an inline property object `{key: "value"}` after the node declaration to customise appearance.

```merfolk
App{Component: Main Application} {color: "blue", scale: "2,1,1"}
DataService[Function: Data Processing] {color: "#4CAF50"}
```

---

## Flow Path Tracking

Flow paths let you define and trace complete data paths that span multiple nodes — not just individual point-to-point connections.

### The `flowpath` Directive

Define a named, multi-hop data path in a single line. This auto-creates tagged connections between each adjacent pair of nodes:

```merfolk
flowpath "userDataFlow" : A --> B --> C --> D
```

This creates three connections (`A→B`, `B→C`, `C→D`), all tagged with the `userDataFlow` identifier so the entire path can be queried as a unit.

**Full syntax options:**

```merfolk
%% Basic flow path
flowpath "name" : NodeA --> NodeB --> NodeC

%% With a custom arrow type (applies to all connections in the path)
flowpath "eventPipeline" (-.->): Input --> Transform --> Output

%% With a description
flowpath "requestLifecycle" : Client --> API --> DB --> API --> Client : "full request cycle"
```

### The `#tag` Syntax on Connections

Tag individual connections with one or more flow path names using `#`:

```merfolk
A --> B : "payload" #userDataFlow
B --> C #userDataFlow #auditTrail
C --> D #auditTrail
```

This is useful when composing flow paths from existing connections rather than auto-generating them.

### Combining Both Approaches

You can freely mix `flowpath` directives with `#tag` connections. If a `flowpath` references a connection that already exists, it tags the existing connection instead of creating a duplicate:

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

The Merfolk parser automatically creates nested grouping when functions are connected to components:

```merfolk
%% Define components and functions
UserService{Component: User Service}
AuthService{Component: Authentication Service}

validateUser[Function: User Validation]
authenticateToken[Function: Token Authentication]

%% Connect functions to components — this creates nesting
validateUser --> UserService : "validates"
authenticateToken --> AuthService : "authenticates"
```

Result:
- Functions automatically become nested inside their connected components.
- Components become containers with increased scale.
- Parent-child relationships are tracked in the data structure.

---

## Complete Example

The following example uses all node types and connection features together:

```merfolk
%% Complete Application Architecture

%% Components (Dodecahedrons, can become containers for functions)
App{Component: Main Application}
UI{Component: User Interface}
API{Component: API Gateway}

%% Functions (Cubes, can be nested in components)
processData[Function: Data Processing]
validateUser[Function: User Validation]
renderUI[Function: UI Rendering]

%% Hooks (Cubes)
useAuth[Hook: useAuth]
useForm[Hook: useForm]
useTheme[Hook: useTheme]

%% Stores (Cubes)
UserDB[[Store: User Database]]
ConfigDB[[Store: Configuration Store]]

%% External Services (Tetrahedrons)
PaymentAPI((Service: Payment Gateway))
EmailService((Service: Email Provider))

%% Libraries (Cubes)
ReactLib<Library: React>
ExpressLib<Library: Express.js>

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

%% Library dependencies
UI --> ReactLib : "uses"
API --> ExpressLib : "uses"

%% Face-specific connections
App@front --> UI@back : "direct rendering"
API@top --> UserDB@bottom : "data flow"

%% Flow paths
flowpath "requestCycle" : App --> API --> UserDB : "full request"
flowpath "cachedRead" : App --> API --> ConfigDB
```
