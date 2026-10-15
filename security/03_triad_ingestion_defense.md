# 🛡️ FLOWORK SOVEREIGN SECURITY DOCTRINE: THE TRIAD INGESTION BINDING & BOUNDED BUFFERING

> **Classification**: Input Sanitization & Anti-Injection Architecture  
> **Authority**: Super Admin AWENKAUDICO & TEGUHFX (Level 99)  
> **Core Doctrine**: The Triad Ingestion Binding, Streaming Bounded Buffers, ReDoS Immunity  

---

## 1. THE TRIAD INGESTION BINDING

Every byte of data crossing an external perimeter into Flowork OS (whether arriving via HTTP endpoints, IPC sockets, user CLI parameters, or filesystem reads) **MUST PASS THROUGH THE THREE-LAYER TRIAD BINDING**:

```text
RAW UNTRUSTED BYTES / STREAM
             │
             ▼
┌────────────────────────────────────────┐
│ LAYER 1: INERT PARSER                  │
│ Pure syntactic decomposition.          │
│ ZERO dynamic execution or evaluation.  │
└────────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────┐
│ LAYER 2: STRICT SCHEMA VALIDATOR       │
│ Closed boundary validation.            │
│ Rejects unknown fields & bounds checks.│
└────────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────┐
│ LAYER 3: SEALED STATIC TYPE            │
│ Mapped to verified domain interfaces.  │
│ Safe for internal consumption.         │
└────────────────────────────────────────┘
```

---

## 2. DETAILED BREAKDOWN OF THE THREE LAYERS

### 2.1 Layer 1: Inert Parsing (Zero-Execution Guarantee)
- **The Danger**: Many conventional parsers evaluate input strings as code or deserialize objects dynamically (e.g., Python `pickle.loads()`, Java `readObject()`, YAML `load()` without safe loaders, or JavaScript `eval()` / `Function()`). This grants attackers immediate Remote Code Execution (RCE).
- **The Sovereign Invariant**:
  - Parsers must be inert: transforming bytes into abstract syntax trees (AST) or static dictionaries without triggering constructors or prototypes.
  - JSON parsing must be strictly standard (`JSON.parse` with prototype poisoning sanitization).
  - Web and script bundles must be disassembled via static AST engines (Babel AST, tree-sitter), never evaluated via `vm.runInContext()`.

### 2.2 Layer 2: Strict Schema Validation
- Once parsed into an in-memory structure, data must be validated against a **Closed Schema** (e.g., Zod with `.strict()`):
  - **No Unknown Fields**: Any property not explicitly declared in the schema must trigger immediate rejection (`strip` or `throw`). Permitting arbitrary fields leads to parameter pollution and prototype pollution.
  - **Length & Numeric Bounds**: Every string must have a declared `max_length`. Every number must have valid `min` and `max` constraints.
  - **ReDoS Protection**: Regular expressions applied to input strings must be audited for polynomial or exponential catastrophic backtracking (*Regular Expression Denial of Service*). Disallow nested quantifiers like `(a+)+$`.

### 2.3 Layer 3: Sealed Static Types
- Transform validated schema payloads into sealed, immutable domain models.
- Prevent subsequent application code from modifying fields or injecting untracked side-channel properties.

---

## 3. BOUNDED STREAM BUFFERING & ANTI-EXHAUSTION

### 3.1 The Whole-File Buffering Hazard
- Reading entire files or incoming network streams into RAM in a single synchronous operation (`fs.readFileSync()`, `stream.read()`) exposes systems to instantaneous Denial of Service via gigabyte-sized payloads or infinite `/dev/zero` streams.

### 3.2 Bounded Slicing Heuristics
1. **Maximum Buffer Allocations**:
   - Header Read Ceiling: `MAX_HEADER_BYTES = 16 * 1024 * 1024` (16 MiB).
   - Frame Read Ceiling: `MAX_FRAME_BYTES = 8 * 1024 * 1024` (8 MiB).
   - Immediate Admission Failure: If a declared header indicates an extent exceeding safe budgets, abort the operation immediately before allocating byte arrays.
2. **Streaming Range-Slicing**:
   - For container files (archives, SQLite databases, package bundles), use file descriptor offsets and range reads (`read(fd, buffer, offset, length, position)`).
   - Read only the byte ranges strictly required for the immediate query, leaving the rest unread on disk.
