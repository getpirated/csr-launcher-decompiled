# Security Notes - CSR Launcher JS review (2026-08-21) (made by AI slop)

> [!NOTE]
> Tl;dr - The launcher doesn't have anything RAT/malware related. If someone actually bricked their PC or got RATed, then it's probably because of game files which are downloaded from https://download.csrestored.fun/. Good luck with scanning, decompiling or reverse engineering each file...

## Verdict: No malware in JS layer (plain Electron, no obfuscation)
Full scan of all non-node_modules JS (main.js:1, preload.js:1, src/js/app.js:1) found only expected launcher behavior.

## Detailed findings

### 1. Native execution
- `main.js:4` `require('child_process')` used only at:
  - `main.js:280` `spawn(csrExe, launchArgs, {cwd: gameDir})` - args: `-game csgo/csr -tickrate 128 [-login_token <cookies>]` plus user `launchArgs` from settings. No shell, stdio ignore.
  - `main.js:1175` `exec(powershell Get-PSDrive)` - disk free check, drive letter only, no user input beyond `path.parse(gameDir).root`.

### 2. File I/O
- Reads: `app.getPath('userData')/settings.json`, `auth_cookies.json`, lang jsons, manifest, checks `csr.exe` existence.
- Writes: same userData files + `custom_lang/` beside exe + downloaded game files under `csgoDir` (validated). No writes to System32, no autorun.

### 3. Network
- Allowlist domains: `api.csrestored.fun`, `download-api.csrestored.fun`, `download.csrestored.fun`, `csrestored.fun`, `cdn.csrestored.fun`, `cdn.discordapp.com`, `127.0.0.1:3000`, `cdnjs.cloudflare.com`, `fonts.googleapis.com` (CSP at src/index.html:6).
- No raw TCP, no WebSocket to unknown hosts, no crypto miners.

### 4. Downloads (potential supply-chain vector)
- Manifest: `GET https://download-api.csrestored.fun/` -> JSON array `{file, hash (md5), lenght}`.
- Download: `GET https://download.csrestored.fun/<file>` -> stream to disk with redirect follow, retry 3x, MD5 verify, unlink on failure.
- If you distrust this, audit hashes / host. Biggest risk is compromised `download.csrestored.fun` serving malicious `csr.exe`.

### 5. Auth / Cookies
- Login window loads `https://csrestored.fun/login` at main.js:535, captures jwt cookies from `persist:auth` partition on navigate away from /login, copies to `session.defaultSession`, persists to `auth_cookies.json`.
- Uses `net.request` with `Cookie` header to `api.csrestored.fun`. No exfiltration elsewhere. `preload.js:1` exposes only via `contextBridge` with `contextIsolation:true`, `nodeIntegration:false`.

### 6. GSIS local server
- `main.js:205` express on 127.0.0.1:3000, CORS enabled, auth token `csr_launcher_token_2024`. Game posts to `/gsis` -> forwarded to renderer via `webContents.send('game-state-update')`. Not exposed externally.

### 7. What to check next if still suspicious
- Hash `csr.exe` after download and VirusTotal it.
- Monitor `https://download-api.csrestored.fun/` manifest manually: `curl https://download-api.csrestored.fun/ | jq`.
- Run launcher in sandbox (Windows Sandbox / VM) and watch `ProcMon` for writes outside `csgoDir`/`userData`.

