# CS:R Launcher (Decompiled from `app.asar`)

> [!IMPORTANT]
> Code has been formatted so it's easier to read. For malware analysis, check [`SECURITY_NOTES.md`](./SECURITY_NOTES.md).

## Quick Start (Windows)

Launcher was designed to run only on Windows as you can notice by PowerShell calls inside [`main.js`](./main.js).

```bat
REM 1. Install dependencies (requires Node.js 18+)
npm install

REM 2. Run without (re)building .exe (uses electron)
npx electron .
```

## Original README

For original README, check [`OG_README.md`](./OG_README.md).