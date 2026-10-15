# 🧼 FLOWORK SOVEREIGN SECURITY DOCTRINE: SECRET DATA SANITIZATION & REDACTION

> **Classification**: Data Privacy & Credential Defense Standard  
> **Authority**: Super Admin AWENKAUDICO & TEGUHFX (Level 99)  
> **Core Doctrine**: Zero-Leakage Logs, WHATWG Authority Cleansing, Safe Exception Serialization  

---

## 1. THE ZERO-LEAKAGE IMPERATIVE

System logs, telemetry metrics, exception stack traces, and user-facing terminal outputs frequently become vectors for inadvertent credential exfiltration (*Credential Harvesting / Accidental Leakage*). 

> [!CRITICAL]
> **NO PRIVATE KEY, ACCESS TOKEN, PASSWORD, OR AUTHENTICATION SECRET MAY EVER APPEAR IN PLAIN TEXT IN ANY LOG FILE, TEST OUTPUT, OR SCREEN DISPLAY.**

---

## 2. AUTOMATIC SECRET REDACTION ENGINE

All logging handlers, terminal stdout formatters, and telemetry pipelines must pass output strings through the **Sovereign Secret Redactor**:

### 2.1 Credential Pattern Registry
The redactor applies deterministic pattern replacement (`[REDACTED_SECRET]`) across the following token classes:
- **API Tokens**: GitHub (`ghp_`, `gho_`, `github_pat_`), OpenAI/Anthropic/Gemini (`sk-`, `AIzaSy...`), Slack (`xoxb-`, `xoxp-`), AWS keys (`AKIA...`).
- **Cryptographic Keys**: PEM blocks (`-----BEGIN (RSA|EC|OPENSSH|PRIVATE) KEY-----`), DER hex signatures.
- **Session & Identity**: Bearer tokens (`Authorization: Bearer <token>`), cookie values (`connect.sid=...`, `session=...`), Basic Auth headers.
- **Passwords & Secrets**: JSON keys named `password`, `secret`, `vault_key`, `access_token`, `private_key` must have their values replaced with `***REDACTED***`.

---

## 3. WHATWG URL AUTHORITY SANITIZATION

URLs parsed from network captures, configuration files, or user inputs frequently embed credentials in the userinfo authority section:

$$\text{https://admin:SuperSecretPassword123@api.internal.network/v1/sync}$$

### 3.1 The Sanitization Rule
- When printing, logging, or reporting any URL, the `username` and `password` authority component **MUST BE STRIPPED OR MASKED**:
  ```typescript
  import { URL } from "node:url";

  export function sanitizeUrl(rawUrl: string): string {
    try {
      const parsed = new URL(rawUrl);
      if (parsed.username || parsed.password) {
        parsed.username = "***";
        parsed.password = "***";
      }
      return parsed.toString();
    } catch {
      // If unparseable by WHATWG standards, redact entirely to prevent leaks
      return "[UNPARSEABLE_URL_REDACTED]";
    }
  }
  ```

---

## 4. SAFE EXCEPTION SERIALIZATION

### 4.1 The Stack Trace Leak Hazard
- Raw exception objects often encapsulate SQL statements with parameter values, internal socket buffers, environment variables, or local filesystem paths containing usernames.

### 4.2 Error Masking Protocol
1. **Internal Log**: Strip sensitive tokens and record the sanitized stack trace only to protected, restricted local storage (`chmod 0600`).
2. **External / User Return**: Return only a high-level error classification code and a unique correlation ID:
   ```json
   {
     "error": "INSUFFICIENT_STORAGE_PERMISSIONS",
     "correlation_id": "err_7f8a9b2c",
     "message": "The storage boundary could not be accessed."
   }
   ```
3. Never echo back the underlying database driver error, internal SQL statement, or machine memory address to the caller.
