# Large Data Explorer

A browser-native data exploration engine designed to interactively process and visualize millions of records while keeping the UI responsive.

> **Goal:** Explore large datasets in the browser without freezing the main thread.

---

## 🚧 Project Status

**Status:** In Development

This project is being developed incrementally to explore the challenges of processing and visualizing large datasets inside the browser.

The implementation starts with a simple JavaScript data model and progressively evolves toward:

* Web Workers
* Query execution
* Query cancellation
* TypedArrays
* ArrayBuffer
* Columnar data
* Streaming
* IndexedDB
* Apache Arrow
* Parquet
* WebAssembly
* DuckDB-WASM

---

## 🎯 Motivation

Displaying a large dataset is not simply a rendering problem.

When the dataset grows from:

```text
10K → 100K → 1M → 10M rows
```

different bottlenecks start appearing:

* JavaScript memory usage
* JSON parsing
* Main-thread blocking
* DOM rendering
* Filtering
* Sorting
* Searching
* Data transfer between threads
* Query cancellation
* Data representation
* Storage
* Query execution

The goal of this project is to investigate these problems step by step and build an architecture capable of handling them efficiently.

---

## 🧠 Core Architecture

```text
┌──────────────────────────────────────────┐
│                 React UI                 │
│                                          │
│ Search / Filter / Table / Query Builder │
└─────────────────────┬────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────┐
│             Data Controller              │
│                                          │
│ Query Lifecycle                          │
│ Cancellation                             │
│ Query State                              │
│ Request / Response Coordination          │
└─────────────────────┬────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────┐
│               Data Engine                │
│                                          │
│ Parse / Filter / Sort / Search / Index  │
└─────────────────────┬────────────────────┘
                      │
                      ▼
               ┌─────────────┐
               │ Web Worker  │
               └──────┬──────┘
                      │
                      ▼
┌──────────────────────────────────────────┐
│              Data Storage                │
│                                          │
│ Objects → TypedArrays → Columnar Data   │
│                                          │
│ IndexedDB / ArrayBuffer / Arrow / ...    │
└──────────────────────────────────────────┘
```

### Architectural Principles

* React is responsible for presentation and user interaction.
* The Data Controller manages query lifecycle and coordination.
* The Data Engine is independent from React.
* CPU-intensive operations should not block the main thread.
* Large datasets should not be represented as unnecessary JavaScript objects.
* Only the data required for rendering should reach the UI.
* Every optimization should be measurable through benchmarks.

---

## 🔍 Query Lifecycle

Every query has a lifecycle:

```text
CREATED
   │
   ▼
QUEUED
   │
   ▼
EXECUTING
   │
   ├──────────────► CANCELLED
   │
   ├──────────────► FAILED
   │
   ▼
COMPLETED
```

Each query receives a unique identifier:

```ts
{
  queryId: 1042,
  query: {...}
}
```

This allows the system to handle concurrent queries and prevent stale results from overwriting newer results.

For example:

```text
Q100 ──────────────────────► result ❌

Q101 ───────────────► result ❌

Q102 ───────► result ✅
```

If `Q102` is the latest query, results from `Q100` and `Q101` are discarded.

---

## 🗃️ Data Representation

The project intentionally evolves through several data representations.

### Phase 1 — Row-based JavaScript objects

```ts
[
  { id: 1, age: 25, salary: 5000 },
  { id: 2, age: 31, salary: 7000 },
  ...
]
```

### Phase 2 — TypedArrays

```ts
const ages = new Int32Array(...);
const salaries = new Float64Array(...);
```

### Phase 3 — Columnar Data

```text
Dataset
│
├── id      → Int32Array
├── age     → Int32Array
├── salary  → Float64Array
└── ...
```

This allows the query engine to operate directly on the columns required by a query instead of processing complete row objects.

---

## ⚙️ Query Engine

The initial query engine will support operations such as:

```text
Filter
Sort
Search
Projection
Limit
AND / OR conditions
```

Example query:

```ts
{
  filter: {
    type: "AND",
    conditions: [
      {
        field: "age",
        operator: ">",
        value: 30
      },
      {
        field: "salary",
        operator: ">",
        value: 5000
      }
    ]
  },
  sort: {
    field: "salary",
    direction: "desc"
  },
  limit: 100
}
```

The long-term goal is to evolve this into a query pipeline:

```text
User Query
    ↓
Parser
    ↓
AST
    ↓
Query Planner
    ↓
Execution Plan
    ↓
Data Engine
    ↓
Result
```

---

## 🧵 Web Worker

CPU-intensive operations should be executed outside the browser's main thread.

```text
Main Thread
────────────────────────────

React
  │
  ▼
Data Controller
  │
  │ Query
  ▼
Web Worker
────────────────────────────
  │
  ▼
Data Engine
  │
  ▼
Dataset
```

The worker communicates with the main thread through a defined message protocol.

Example:

```ts
{
  type: "EXECUTE_QUERY",
  queryId: 1042,
  query: {...}
}
```

Response:

```ts
{
  type: "QUERY_RESULT",
  queryId: 1042,
  result: {...}
}
```

---

## 🚀 Rendering Strategy

The application must never attempt to render millions of DOM nodes.

Instead, the UI uses virtualization:

```text
Dataset
10,000,000 rows
       │
       ▼
Virtualization
       │
       ▼
~50 visible DOM rows
```

The table only renders the rows currently visible in the viewport.

---

## 📊 Performance Lab

Performance is treated as a first-class feature of the project.

The system will benchmark:

### Dataset Size

```text
100K
1M
5M
10M
```

### Operations

```text
Parsing
Filtering
Sorting
Searching
Rendering
Data Transfer
```

### Runtime Metrics

```text
Main Thread Time
Worker Time
Memory Usage
Long Tasks
FPS
Query Latency
DOM Node Count
```

Example:

```text
Dataset: 10M rows

Parsing          1.24s
Filtering        180ms
Sorting          720ms
Search           95ms
Worker           890ms
Main Thread      8ms
Memory           620MB
FPS              59
```

---

## 🧪 Benchmark Philosophy

The project will compare different approaches rather than assuming that one architecture is always better.

Examples:

```text
Array<Object>
      vs
TypedArray
```

```text
Row-based
      vs
Columnar
```

```text
Main Thread
      vs
Web Worker
```

```text
Structured Clone
      vs
Transferable ArrayBuffer
```

```text
Full Scan
      vs
Indexed Search
```

```text
JavaScript
      vs
WebAssembly
```

```text
Custom Query Engine
      vs
DuckDB-WASM
```

---

## 🛣️ Roadmap

### M1 — Basic Explorer

* [ ] Next.js + TypeScript
* [ ] Dataset model
* [ ] JSON ingestion
* [ ] 10K / 100K datasets
* [ ] Basic data table
* [ ] Row virtualization

### M2 — Query Engine

* [ ] Query model
* [ ] Filtering
* [ ] Sorting
* [ ] Searching
* [ ] Projection
* [ ] Query Controller

### M3 — Worker Architecture

* [ ] Web Worker
* [ ] Worker protocol
* [ ] Query lifecycle
* [ ] Query cancellation
* [ ] Stale result protection

### M4 — Large Data

* [ ] 1M / 5M / 10M datasets
* [ ] ArrayBuffer
* [ ] TypedArrays
* [ ] Transferable Objects
* [ ] Columnar data
* [ ] Memory benchmarks

### M5 — Data Infrastructure

* [ ] Streaming ingestion
* [ ] IndexedDB
* [ ] Large file support
* [ ] JSON / CSV / NDJSON
* [ ] Apache Arrow
* [ ] Parquet

### M6 — Advanced Execution

* [ ] Query AST
* [ ] Query Planner
* [ ] Execution Plan
* [ ] Indexing
* [ ] WebAssembly
* [ ] DuckDB-WASM

### M7 — Performance Lab

* [ ] Benchmark framework
* [ ] Runtime metrics
* [ ] Query timeline
* [ ] Memory monitoring
* [ ] Worker profiling
* [ ] Performance visualization

---

## 🧱 Planned Project Structure

```text
src/
│
├── app/
│
├── components/
│   ├── data-grid/
│   ├── query-builder/
│   ├── dataset-panel/
│   └── performance-panel/
│
├── controller/
│   └── data-controller/
│
├── data-engine/
│   ├── parser/
│   ├── query/
│   ├── filter/
│   ├── sort/
│   ├── search/
│   ├── index/
│   └── storage/
│
├── workers/
│   └── data-worker/
│
├── domain/
│   ├── dataset/
│   ├── query/
│   └── worker/
│
├── benchmark/
│
└── utils/
```

The exact structure will evolve as the architecture becomes more sophisticated.

---

## 🛠️ Tech Stack

### Core

* Next.js
* React
* TypeScript

### Browser APIs

* Web Workers
* ArrayBuffer
* TypedArrays
* IndexedDB
* Performance API

### Data Technologies

* Apache Arrow
* Parquet
* WebAssembly
* DuckDB-WASM

### UI

* Virtualized Data Grid
* Query Builder
* Performance Dashboard

---

## 🎓 Technical Topics Explored

This project is also a practical exploration of:

* Browser Runtime
* Main Thread vs Worker
* Event Loop
* CPU vs Memory Bottlenecks
* Virtualization
* Large Dataset Rendering
* Query Execution
* Query Planning
* Data Structures
* Binary Data
* TypedArrays
* ArrayBuffer
* Columnar Storage
* Indexing
* Streaming
* Backpressure
* Structured Clone
* Transferable Objects
* WebAssembly
* Apache Arrow
* Parquet
* In-browser Analytics

---

## 📈 Performance Goal

The goal is not simply:

> "Support 10 million rows."

The real goal is:

> **Understand why large datasets make browser applications slow, identify the bottleneck, and design the architecture so that each bottleneck can be measured and addressed independently.**

A successful implementation should keep the UI responsive even while processing large datasets.

---

## 🔬 Engineering Philosophy

This project intentionally avoids premature optimization.

Each major optimization should answer three questions:

1. **What is the bottleneck?**
2. **What architectural change addresses it?**
3. **Did benchmarking prove that the change helped?**

Example:

```text
Problem
  ↓
10M Object rows consume too much memory
  ↓
Hypothesis
  ↓
Columnar TypedArray representation
  ↓
Implementation
  ↓
Benchmark
  ↓
Compare memory + query latency
```

The objective is not to use every modern technology.

The objective is to understand **why and when each technology is useful**.
