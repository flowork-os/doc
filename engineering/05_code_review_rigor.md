# 🧐 FLOWORK SOVEREIGN ENGINEERING DOCTRINE: CODE REVIEW RIGOR & CODE SMELLS

> **Classification**: Production Code Quality & Review Standard  
> **Authority**: Super Admin AWENKAUDICO & TEGUHFX (Level 99)  
> **Core Doctrine**: Two-Axis Independent Review, The 12 Fowler Code Smells, Anti-Rubber-Stamp  

---

## 1. THE ANTI-RUBBER-STAMP MANDATE

> [!WARNING]
> **A review that finds nothing and changes nothing is almost always a failed review.**  
> An autonomous agent acting as a code reviewer is not an automated cheerleader. Your role is adversarial quality assurance: uncovering silent regressions, architectural debt, security vulnerabilities, and specification deviations before code reaches production.

---

## 2. THE TWO-AXIS INDEPENDENT REVIEW FRAMEWORK

Code reviews in Flowork OS must evaluate pull requests across **two completely independent axes**. Never merge, average, or cross-rank findings between these two axes:

```text
                  AXIS 1: SPECIFICATION FIDELITY
    [Did the code implement exactly what was requested?]
                             │
                             │
                             ├─────────────── HIGH SPEC FIDELITY, LOW STANDARDS
                             │                (Works, but architecturally unmaintainable)
                             │
─────────────────────────────┼─────────────────────────────
                             │
 HIGH STANDARDS, LOW SPEC   │                SOVEREIGN ACCEPTANCE ZONE
 (Clean code, wrong problem) │                (Exact Spec Fidelity + High Standards)
                             │
                  AXIS 2: ENGINEERING STANDARDS
       [Does the code conform to clean architecture & hygiene?]
```

### Axis 1: Specification Fidelity
- Did the implementation satisfy every requirement outlined in the task contract?
- Are edge cases handled according to the requested functional boundaries?
- Did the implementation introduce unrequested features, speculative bloat, or unintended behavioral drift?

### Axis 2: Engineering Standards & Clean Code
- Does the code adhere to structural best practices, memory safety, and performance constraints?
- Is the code free of code smells, dead paths, and security antipatterns?
- Are tests comprehensive, non-tautological, and passing cleanly with terminal Exit Code 0?

---

## 3. THE 12 FOWLER CODE SMELLS (BASELINE AUDIT CHECKLIST)

When inspecting changes along Axis 2, evaluate against Martin Fowler's canonical code smells:

| # | Code Smell | Description & Production Risk | Remediation Strategy |
|:---:|:---|:---|:---|
| **1** | **Long Method** | A method exceeding 25-30 lines containing multiple levels of abstraction. | Extract Method. Separate orchestration from computation. |
| **2** | **Large Class** | A class trying to do too much, violating Single Responsibility Principle. | Extract Class / Delegate responsibilities to deep modules. |
| **3** | **Primitive Obsession** | Using raw strings/integers for domain concepts (e.g., passing `string` instead of a typed `UserId` or `FileHash`). | Introduce Value Objects / Sealed Types. |
| **4** | **Long Parameter List** | Functions taking more than 3-4 arguments, making calls brittle. | Introduce Parameter Object or Builder. |
| **5** | **Data Clumps** | Bunches of data that hang around together (e.g., `x, y, z` or `street, city, zip`). | Encapsulate into a dedicated record/class. |
| **6** | **Feature Envy** | A method that accesses the data of another object more than its own. | Move Method into the data-owning class. |
| **7** | **Divergent Change** | One class is commonly changed in different ways for different reasons. | Split class along axes of change. |
| **8** | **Shotgun Surgery** | A single conceptual change forces many small edits across numerous files. | Consolidate logic into a single cohesive module. |
| **9** | **Temporary Field** | An object field set only under certain circumstances, leaving it null otherwise. | Extract Class / Null Object pattern. |
| **10**| **Message Chains** | Navigating long chains of accessors (`a.getB().getC().getD().doSomething()`). | Hide Delegate. Enforce Law of Demeter. |
| **11**| **Dead Code** | Unused variables, unreachable branches, commented-out logic, obsolete flags. | Ruthless deletion. Git history preserves past code. |
| **12**| **Speculative Generality** | Designing hooks, abstract factories, and extension points "just in case" they are needed later. | YAGNI (You Aren't Gonna Need It). Strip out abstractions until concrete need arises. |

---

## 4. VERDICT PROTOCOL

Every formal review must conclude with one of three deterministic verdicts:
1. **APPROVED**: Both Axis 1 and Axis 2 pass completely. All tests green (Exit Code 0).
2. **CHANGES REQUESTED**: Concrete, actionable defects identified along Axis 1, Axis 2, or both. Must cite specific line numbers, smell categories, or failed test cases.
3. **REJECTED**: Fundamental architectural mismatch, unrecoverable specification divergence, or security boundary violation.
