# 🖥️ FLOWORK SOVEREIGN SECURITY DOCTRINE: ELECTRON & WEB RUNTIME HARDENING

> **Classification**: Desktop Client & Web Sandbox Hardening Standard  
> **Authority**: Super Admin AWENKAUDICO & TEGUHFX (Level 99)  
> **Core Doctrine**: The 9 Electron Fuses, Sandboxed Renderers, Preload IPC Whitelisting, Strict CSP  

---

## 1. THE 9 CORE ELECTRON SECURITY FUSES

Electron Fuses are compile-time binary toggles that disable vulnerable debugging, node injection, and packaging behaviors at the machine-code level. For production releases of Flowork OS desktop runtimes, the **9 Core Fuses** must be locked as follows:

| Index | Fuse Name | Sovereign State | Security Rationale |
|:---:|:---|:---:|:---|
| **0** | `RunAsNode` | **DISABLED** | Prevents executing the app binary as a plain Node.js interpreter via `ELECTRON_RUN_AS_NODE=1`. |
| **1** | `EnableCookieEncryption` | **ENABLED** | Forces encryption of stored session cookies via OS DPAPI/Keychain. |
| **2** | `EnableNodeOptionsEnvironmentVariable` | **DISABLED** | Ignores `NODE_OPTIONS` environment variable, preventing `--require` payload injection. |
| **3** | `EnableNodeCliInspectArguments` | **DISABLED** | Disables CLI arguments `--inspect` and `--remote-debugging-port`, closing debugger hijacking in production. |
| **4** | `EnableEmbeddedAsarIntegrityValidation` | **ENABLED** | Enforces cryptographic header validation; modifications to `app.asar` cause immediate startup abort. |
| **5** | `OnlyLoadAppFromAsar` | **ENABLED** | Restricts code loading strictly to signed `app.asar`, blocking unpacked folder substitution attacks. |
| **6** | `LoadBrowserProcessSpecificV8Snapshot` | **ENABLED** | Ensures browser process loads its own hardened V8 snapshot. |
| **7** | `GrantFileProtocolExtraPrivileges` | **DISABLED** | Revokes elevated local file system access from `file://` scheme renderers. |
| **8** | `WasmTrapHandlers` | **ENABLED** | Enforces hardware trap handlers for WebAssembly bounds checking. |

---

## 2. WEBPREFERENCES ISOLATION LOCKDOWN

When creating any `BrowserWindow` or `BrowserView`, the following `webPreferences` configuration is **MANDATORY**:

```typescript
const win = new BrowserWindow({
  webPreferences: {
    // 1. Absolute process boundary between DOM and Node.js
    contextIsolation: true,

    // 2. Disallow raw Node.js primitives inside renderer
    nodeIntegration: false,

    // 3. Force OS-level Chromium sandbox on renderer process
    sandbox: true,

    // 4. Enforce SOP (Same-Origin Policy)
    webSecurity: true,

    // 5. Block mixed HTTP/HTTPS content
    allowRunningInsecureContent: false,

    // 6. Explicitly attach minimal preload script
    preload: path.join(__dirname, "preload.js"),
  },
});
```

---

## 3. PRELOAD & INTER-PROCESS COMMUNICATION (IPC) HARDENING

### 3.1 The Context Bridge Whitelist Pattern
- Preload scripts must NEVER expose raw `ipcRenderer` to the browser `window` object:
  ```typescript
  // INSECURE (INSTANT FATAL VULNERABILITY):
  contextBridge.exposeInMainWorld("api", { ipc: ipcRenderer });

  // SECURE (SOVEREIGN STANDARD):
  contextBridge.exposeInMainWorld("floworkAPI", {
    getWorkspaceStatus: () => ipcRenderer.invoke("workspace:get-status"),
    requestFileAudit: (path: string) => ipcRenderer.invoke("audit:file", path),
  });
  ```

### 3.2 Main Process IPC Admission Validation
- The Main process (`ipcMain.handle`) must treat all arguments received from the renderer process as **untrusted user input**.
- Every IPC handler must validate argument types, lengths, and schemas using the Triad Ingestion Binding before taking any system action.

---

## 4. CONTENT SECURITY POLICY (CSP) STANDARD

Every HTML document served or loaded in Flowork OS runtimes must declare a rigid CSP via HTTP header or `<meta>` tag:

```text
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self'; connect-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none';
```

- **Strictly Prohibited**: `'unsafe-eval'` is banned without exception. Any pattern relying on runtime code generation inside renderers must be refactored to static compiled logic.
