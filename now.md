# Current state

## Lite release follow-up

The No-Node Lite distribution has been merged upstream as an additional
release artifact. The current follow-up changes are being implemented on a new
branch after the merge.

### In this implementation

- Keep the existing Electron packages unchanged.
- Publish the Lite artifact as
  `codex-proxy-<version>-no-node-lite-all-platforms.zip`.
- Use maximum ZIP compression (`7z -tzip -mx=9`, `zip -9`, or the Python
  `zipfile` fallback with compression level 9).
- Do not include `MicrosoftEdgeWebView2Setup.exe` in the archive.
- In explicit WebView2 mode, download the Microsoft Bootstrapper at runtime
  only after user confirmation, verify its Authenticode signature, run it, and
  re-check the Runtime.
- Keep `codex-proxy.cmd` as a Windows diagnostic fallback and retain the
  Windows x86/x64 WebView2 hosts.
- Build and package `codex-tls.linux-x64-musl.node` separately from the
  existing Linux GNU addon, with an Alpine runtime smoke test.
- Treat Lite as an optional release artifact: a Lite build failure should not
  prevent a successful Electron release from being marked latest.

### Deferred discussion

These items are intentionally outside the current implementation and should
be discussed as separate follow-up work:

- Linux ARM64 native addon;
- Windows ARM64 native addon;
- native ARM64 WebView2 host;
- 7z as an additional published format;
- Node.js 18 compatibility probing;
- remaining package-size and native-addon optimizations;
- offline WebView2 Standalone Installer distribution;
- broader architecture matrix and real-device validation.

### Current release automation boundary

The upstream `release.yml` is triggered by `v*` tags or manual dispatch. The
Lite job is part of that workflow, while `lite-ci.yml` remains a manual CI
validation workflow. Both workflows now build the Linux x64 musl addon in the
official NAPI-RS Alpine toolchain container before assembling the Lite ZIP.
The manual Lite workflow additionally runs the packaged musl loader inside an
Alpine Node container. The release workflow must still be checked on a real
tag or a manual dispatch before this follow-up is proposed upstream.
An additional `native-musl-ci.yml` workflow is intentionally limited to the
Linux x64 musl addon build, ELF dependency inspection, and direct native HTTP
smoke test; it does not build the Lite archive or WebView2 hosts.

## Validation evidence

- `npm run build` passed.
- `npm --prefix packages/electron run build` passed.
- Local MSYS2 MinGW x86 and x64 WebView2 hosts built successfully.
- The local package was generated as
  `portable-release/codex-proxy-2.1.1-no-node-lite-all-platforms.zip`.
- The local ZIP was about 3.94 MiB, used Deflate for files, and contained no
  Bootstrapper entries.
- Windows package test passed with the native launcher, both WebView2 hosts,
  no-Node checks, data-path checks, and server smoke checks.
- Portable contract and CI package-boundary tests passed: 23 tests.
- The Linux x64 musl build and Alpine runtime smoke test are delegated to
  GitHub CI; this Windows host has neither Docker nor an Alpine runtime, so
  those two boundaries are currently UNRUN locally.
- Full project test suite was not rerun in this follow-up; the portable-focused
  tests and build checks are the current evidence.

The local Git fetch of the merged upstream `dev` branch timed out, so the
working branch currently starts from the locally available final Lite tree.
Before opening the next upstream PR, compare/rebase it against the current
upstream `dev` to ensure the PR contains only this follow-up.
