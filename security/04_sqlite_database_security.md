# 🗄️ FLOWORK SOVEREIGN SECURITY DOCTRINE: SQLITE & LOCAL DATABASE HARDENING

> **Classification**: Database Integrity & Forensic Security Standard  
> **Authority**: Super Admin AWENKAUDICO & TEGUHFX (Level 99)  
> **Core Doctrine**: Mandatory PRAGMA Defense Suite, Absolute Parameterization, Memory Map Lockdown  

---

## 1. THE SQLITE SECURITY SUITE (MANDATORY PRAGMAS)

SQLite is the primary local persistence engine across Flowork OS desktop runtimes, browser stores, and analysis caches. By default, SQLite prioritizes backward compatibility over maximum defense. 

Every database connection opened by Flowork OS **MUST IMMEDIATELY EXECUTE THE SOVEREIGN PRAGMA DEFENSE SUITE** prior to any schema querying or transaction execution:

```sql
-- 1. Enforce strict relational referential integrity
PRAGMA foreign_keys = ON;

-- 2. Mitigate malicious triggers and virtual table exploits
PRAGMA trusted_schema = OFF;

-- 3. Detect binary page corruption and b-tree overflow attacks
PRAGMA cell_size_check = ON;

-- 4. Disable memory mapping to prevent memory-corruption vulnerabilities
PRAGMA mmap_size = 0;
```

---

## 2. DETAILED RATIONALE FOR EACH PRAGMA

### 2.1 `PRAGMA trusted_schema = OFF` (The Trigger Exploit Shield)
- **The Threat**: In default SQLite configurations, SQL functions defined inside views, triggers, and virtual tables are executed with the full privileges of the host application. An attacker supplying a crafted database file can trigger arbitrary code execution simply by tricking the application into reading a poisoned view or running `VACUUM`.
- **The Protection**: Setting `trusted_schema = OFF` ensures that functions with side effects or non-deterministic behaviors cannot be called from within schema elements, neutralizing schema-based RCE payloads.

### 2.2 `PRAGMA cell_size_check = ON` (Anti-B-Tree Corruption)
- **The Threat**: Specially crafted, corrupted SQLite files can cause out-of-bounds reads and memory corruption if a b-tree cell reports an offset that extends beyond the physical page boundary.
- **The Protection**: Enables explicit boundary validation on every b-tree cell read. If an invalid offset is detected, SQLite immediately returns `SQLITE_CORRUPT` rather than crashing the host process.

### 2.3 `PRAGMA mmap_size = 0` (Memory Safety Lockdown)
- **The Threat**: While memory mapping (`mmap`) accelerates read throughput by mapping files directly into process address space, reading untrusted or concurrently mutated databases via `mmap` can trigger hardware segmentation faults (SIGSEGV/SIGBUS) if a file is truncated or page tables become desynchronized.
- **The Protection**: Explicitly disabling `mmap` (`mmap_size = 0`) forces SQLite to read pages through standard, bounds-checked OS read buffers, ensuring process stability.

### 2.4 `PRAGMA query_only = ON` (Read-Only Inspection Mode)
- When opening an existing user database or forensic snapshot for read-only analysis, always execute `PRAGMA query_only = ON`.
- Any inadvertent `INSERT`, `UPDATE`, `DELETE`, or `DROP` will fail closed at the SQLite engine level.

---

## 3. ABSOLUTE QUERY PARAMETERIZATION

### 3.1 The Zero-Concatenation Law
> [!CRITICAL]
> **CONCATENATING VARIABLES INTO SQL QUERY STRINGS IS HARAM.**  
> No matter how benign or internal a variable appears, string concatenation (`"SELECT ... WHERE id = '" + id + "'"`) is an instant fatal audit violation.

### 3.2 Secure Parameter Binding
- Always bind values via positional (`?`) or named (`:param`) parameters:
  ```typescript
  // INSECURE (INSTANT REJECTION)
  db.prepare(`SELECT * FROM artifacts WHERE hash = '${fileHash}'`).all();

  // SECURE (SOVEREIGN STANDARD)
  db.prepare("SELECT * FROM artifacts WHERE hash = ?").all(fileHash);
  ```
- This ensures the SQL lexer treats parameters strictly as literal values, completely neutralizing SQL Injection (SQLi) attacks.
