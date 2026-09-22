# Etemad Bourse - Full Source Archive (v3.8)

The complete, packaged source release (v3.8) of "Etemad Bourse" (اعتماد بورس) - a TSETMC
(Tehran Stock Exchange) stock/option screener and daily-report tool built by the author. It
bundles the Python desktop server + Persian RTL web dashboard, the Go analysis worker, the
native Android WebView client, and the deployment/test tooling into one archive with an
`ARCHIVE_SUMMARY.txt` manifest.

This folder is a **release snapshot of the author's own project**, not third-party or
client-delivered code. `Server_Backend_Frontend/koochin_report.py`, `main.py` and
`app/server.py` are byte-identical to the working copies in
`Documents\Projects\koochin-report\`, which carries the same three PyInstaller `.spec`
files, and `Android_App` declares the same `com.etemad.bourse` application id as
`Documents\Projects\BourseFilterAndroid`. See Notes for which copy is canonical.

**Suggested repo name:** `etemad-bourse`
**Stack:** Python 3.12 + openpyxl (desktop server/CLI), Go 1.22 worker, Java/Android WebView client, PyInstaller packaging, Playwright tests
**Status:** active - packaged release
**Last modified:** 2026-09-20

## What it does

Three sub-projects plus tooling, all for the same product (a Persian stock/option screener whose
core metric is buyer power, "قدرت خریدار", over 30-second TSETMC trade windows):

- `Server_Backend_Frontend/` - the main app.
  - `main.py` starts a local HTTP server and opens the dashboard (`app/index.html`) in a WebView2
    window (falls back to the default browser). Flags: `--browser`, `--no-browser`, `--port 8600`.
  - `app/server.py` - stdlib `ThreadingHTTPServer` with multi-user auth, SQLite per-user isolation,
    session memory, `POST /api/export` (writes filtered rows to Excel) and `POST /api/open-folder`.
  - `koochin_report.py` - the analysis engine + CLI producing the client's RTL Excel report
    (`--symbol`, `--date`, `--watch`, `--history`, `--from-sample`).
  - `worker/` - Go high-performance data worker (`bourse-worker`, built `.exe` and Linux binary).
  - `BourseFilter*.spec` / `EtemadBourse.spec` / `version_info.txt` - PyInstaller packaging (v3.8.0.0).
  - `tests/test_engine.py` - engine unit tests.
- `Android_App/` - the same WebView shell as `BourseFilterAndroid` (near-identical `MainActivity.java`),
  plus a prebuilt `binaries/app-debug.apk`.
- `Deployment_and_Tools/` - `upload.py` (SSH/SCP deploy to the production host),
  `build_unified_app.py` (single-file bundler), `test_ui_baskets_playwright.py` and
  `test_remote_4baskets.py` (live remote API verification), `verify_remote_html.py`.

## Layout

```
ARCHIVE_SUMMARY.txt              manifest describing the 3 parts + production server details
Server_Backend_Frontend/
  main.py                        desktop entrypoint (server + WebView window)
  app/server.py                  HTTP server, auth, export/open-folder endpoints
  app/index.html, login.html     Persian RTL dashboard + login
  koochin_report.py              TSETMC analysis engine + CLI
  worker/main.go                 Go data worker (+ go.mod, built binaries)
  tests/test_engine.py           engine tests
  *.spec, version_info.txt       PyInstaller packaging
Android_App/                     native WebView client (+ binaries/app-debug.apk)
Deployment_and_Tools/            upload.py, build_unified_app.py, Playwright/remote tests
```

## Running it

The parts run independently; commands come from `Server_Backend_Frontend/README.md`:

```bash
cd Server_Backend_Frontend
pip install openpyxl
py main.py                                   # desktop app (WebView2 / browser)
py koochin_report.py --symbol کوچین --out koochin.xlsx    # CLI report
```

Production deployment is scripted by `Deployment_and_Tools/upload.py` (SCP to `<prod-host>:8080`,
`/home/developer/bourse`, service `bourse.service`). The Android app builds with Gradle/Android Studio.

## Notes

- **Author's own code - this is the packaging, not the working tree.** An earlier read of this
  archive treated it as a contractor deliverable purely because `ARCHIVE_SUMMARY.txt` credits
  "Development Team"; that was wrong. The hash match against `Documents\Projects\koochin-report\`
  settles it. The practical question is which copy becomes the repo: this archive is the more
  complete artifact (Android app + Go worker + deploy/test tools in one place, all 41 files),
  while `koochin-report\` is the actively-edited tree but also carries `build\`, `dist\` and
  `data.db` junk. Recommend publishing this archive as `etemad-bourse` and marking
  `koochin-report` the same repo's source location rather than shipping two repos of one product.
- **Already has its own README** in `Server_Backend_Frontend/`. If you split that sub-project into
  its own repo, it needs no new README; the root one here is the archive wrapper.
- **Production endpoint is hard-coded and secret-bearing.** `Deployment_and_Tools/upload.py` holds
  the live host (`<prod-host>`), SSH user, password, and host key; the Android `MainActivity` and
  dashboard point at the same plaintext `http://` IP. Rotate those credentials and strip them from
  the archive before any public push.
- `app/server.py` stores user `password` in SQLite; `main.py`/`server.py` carry default admin
  credentials - review the auth model before exposing the dashboard.
- Prebuilt binaries are committed: `Android_App/binaries/app-debug.apk`, `worker/bourse-worker*`,
  and PyInstaller `dist/` output. These are build artifacts, not source, and should not be published.
- Overlaps with the separate `BourseFilterAndroid` folder (the Android app) - pick one canonical copy.
