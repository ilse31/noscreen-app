# AGENTS.md

Guidance for AI agents working in this repo. noscreen is a **Tauri 2** desktop app: a Svelte 5 + SvelteKit frontend (`src/`) and a Rust backend (`src-tauri/`). Read `README.md` for the product overview and architecture diagram.

## Commands

| Task | Command |
|------|---------|
| Run full app (Rust + webview) | `npm run tauri dev` |
| Frontend dev server only | `npm run dev` (Vite, port **1420**, strict) |
| Typecheck frontend | `npm run check` (runs `svelte-kit sync` → `svelte-check` + tsc) |
| Frontend unit tests | `npm test` (Vitest, **uses `vitest.config.ts`** — see gotcha) |
| Frontend tests, watch | `npm run test:watch` |
| Run a single test | `npm test -- <file-or-name-pattern>` |
| Rust unit tests | `cargo test` (run **inside `src-tauri/`**) |
| Production installer | `npm run tauri build` → `src-tauri/target/release/bundle/` |

There is **no lint or formatter configured** (no eslint/prettier/biome). `npm run check` is the only static-analysis gate.

## Package manager: use npm, not Bun

Both `bun.lock` and `package-lock.json` exist, but the canonical manager is **npm**. CI uses `npm ci`, and `tauri.conf.json` runs `npm run dev` / `npm run build` via `beforeDevCommand` / `beforeBuildCommand`. Don't switch to Bun or you'll desync the build pipeline.

## Testing gotchas

- Tests run against **`vitest.config.ts`**, *not* the `test:` block inside `vite.config.ts`. `vitest.config.ts` deliberately uses the plain `svelte` plugin with `resolve.conditions: ['browser']` instead of `sveltekit()`, because SvelteKit's SSR resolve condition otherwise overrides Svelte's browser build and breaks component tests. If you edit test setup, edit the right file.
- `$app/environment` is aliased to a mock at `src/lib/__mocks__/app-environment.ts` (all flags truthy) so SvelteKit imports resolve under jsdom.
- Frontend tests are browser-only (jsdom); they never exercise Rust commands.

## Frontend is client-only

`src/routes/+layout.ts` sets `ssr = false` and `svelte.config.js` uses `adapter-static` in SPA mode. All app code runs inside the Tauri webview — there is no Node server. Don't add server-side code or rely on SSR.

## Adding a Tauri command = edit three places

To expose a new backend call to the frontend:
1. Define `#[tauri::command]` in `src-tauri/src/commands.rs`.
2. Register it in the `tauri::generate_handler![...]` list in `src-tauri/src/lib.rs`.
3. Add a typed wrapper in `src/lib/tauri.ts` (all `invoke()` calls are centralized there — don't call `invoke()` directly from components/stores).

## Svelte 5 notes

- Svelte 5 Runes are in use. Files using runes outside a `.svelte` component must use the **`.svelte.ts`** extension (e.g. `src/lib/stores/settings.svelte.ts`).
- Two store styles coexist: `src/lib/stores.ts` (Svelte 4 writable stores, still tested) and `src/lib/stores/` (runes). Don't assume one global pattern.
- `$lib` aliases to `./src/lib`.

## Backend architecture & platform conditionals

- `src-tauri/src/lib.rs` wires everything: window creation, plugins, hotkeys, and the `invoke_handler`. `commands.rs` holds all commands; `protection.rs` applies capture-exclusion per window.
- **Several modules are Windows-only** (`ghost_typing.rs`, `stt.rs`, parts of `copilot/`) via `#[cfg(windows)]` and the `windows` crate. macOS is partially supported; Linux is not. On non-Windows these are stubbed, so `cargo test`/build behavior differs by platform — don't expect every command to compile identically everywhere.
- **Multi-window**: the main UI is window `"ai-view"` (hub); also `"settings"`, `"copilot-card"`, and pre-created hidden service windows `svc-gpt` / `svc-claude` / `svc-translate`. In debug builds windows load `http://localhost:1420/<route>`; in release all load `index.html` (SPA routing). `frontendDist` is `../build`.
- Service webviews get an anti-bot fingerprint script (`INJECT_BOOTSTRAP` in `lib.rs`) injected on load.

## Data & persistence

- SQLite via `rusqlite` with the **`bundled`** feature — no system SQLite needed. DB file is `profile.db` in the OS app-data dir.
- "Migrations" are additive `CREATE TABLE IF NOT EXISTS` statements in `db::open()` (`execute_batch`). Adding a column/table means editing that batch; there is no separate migration tool.
- App config (hotkey, window position, autostart, site overrides) is a JSON file in app-data dir, read/written by `config.rs`.

## Release / CI

- Releases are triggered by pushing a **`v*` git tag**, which runs `.github/workflows/release.yml` (GitHub Actions): builds Windows x64 + macOS arm64/x64 via `tauri-apps/tauri-action`, creates a **draft** GitHub Release.
- `.gitlab-ci.yml` and `appveyor.yml` are Windows-only alternates (tag/web triggered); GitHub Actions is the primary path.


<claude-mem-context>
# Memory Context

# [noscreen-app] recent context, 2026-09-22 4:07pm GMT+7

Legend: 🎯session 🔴bugfix 🟣feature 🔄refactor ✅change 🔵discovery ⚖️decision 🚨security_alert 🔐security_note
Format: ID TIME TYPE TITLE
Fetch details: get_observations([IDs]) | Search: mem-search skill

Stats: 50 obs (16,965t read) | 887,087t work | 98% savings

### Sep 21, 2026
17698 9:18a 🔵 cidre 0.29 TapDesc initialization methods have unresolved Option wrapping bug
17699 " 🔵 cidre's define_weak_cls! macro causes TapDesc alloc() to return Option wrapper
17700 " 🔴 Work around cidre TapDesc convenience wrapper by manual allocation + initialization
17701 9:19a 🔵 Feature-gated cidre dependency fails to compile due to internal cidre library bugs
17702 " 🔵 cidre's api_available macro automatically converts define_cls to define_weak_cls
17703 " 🔵 cidre api_available macro creates dual implementations (regular + weak) for version-gated classes
17704 " 🔵 cidre 0.29 TapDesc errors persist even with MACOSX_DEPLOYMENT_TARGET=14.2
17705 " 🔵 libswift_Concurrency.dylib missing from Swift runtime and Xcode toolchain
17706 9:22a 🔵 TapDesc convenience factory methods chain directly on alloc() without unwrapping Option
17707 " 🔴 Patch cidre TapDesc convenience methods to unwrap Option from alloc()
17708 " ✅ Configure Cargo to use vendored, patched cidre via [patch.crates-io]
17709 " 🔵 macOS audio loopback implementation compiles successfully with patched cidre
17710 9:23a 🔵 All copilot unit tests pass successfully with macOS audio implementation
17711 " 🔵 All 45 library unit tests pass including copilot audio and database integration
17712 " 🔵 Built tauri-app binary times out on startup attempting to load Swift runtime
17713 9:25a ✅ Update README.md to document macOS Copilot audio loopback with Process Tap API requirement
17714 9:26a ✅ Hide Windows-only STT backend option on macOS in settings UI
S5278 Dashboard feature enhancement request - add conversation history and system status panels to existing Dashboard component (Sep 21 at 9:42 AM)
17715 9:43a 🟣 AI model selection from test connection response
17716 " 🟣 Bulk conversation deletion with selection UI
17717 " 🟣 Test coverage for AI settings and chat deletion
17718 " ✅ Version 1.1.0 released across all platforms
17719 " 🔵 All platform builds and tests passed for v1.1.0
S5279 Implement dashboard with recent conversation history and system status metrics, including token usage and latency tracking (Sep 21 at 11:34 AM)
17727 11:36a ✅ Added chat usage tracking database schema and statistics functions
17728 11:37a 🔵 Duplicate test module definition in db.rs causes compilation failure
17729 " 🔴 Resolved duplicate test module definition by merging new usage tests
17730 11:38a ✅ Wired usage stats retrieval and conversation navigation into frontend
17731 " 🟣 Created dashboard formatting helpers with comprehensive test coverage
17732 " 🟣 Implemented Dashboard with recent conversations and system status panels
17733 11:39a 🔵 Dashboard test suite identified dash character assertion ambiguity
17734 " 🔵 Test suite requires SvelteKit build artifacts to execute
17735 " 🔵 All tests pass after excluding stale worktree directory
S5280 Dashboard implementation with recent conversation history and system status metrics, including real token usage and latency tracking (Sep 21 at 11:40 AM)
S5281 Implement server compatibility fallback for stream_options parameter rejection and token estimation when server doesn't report usage (Sep 21 at 11:42 AM)
S5282 Error handling strategy and risk assessment for stream_options fallback and token estimation feature completeness (Sep 21 at 11:42 AM)
S5283 Version bump to 1.2.0, verification pass, commit dashboard feature, and trigger release build workflow (Sep 21 at 11:45 AM)
S5284 Remove Claude co-author trailer from v1.2.0 commit, re-trigger build workflow with amended commit, and establish user preference for no Claude attribution in future commits (Sep 21 at 11:48 AM)
S5285 Investigate why v1.2.0 build completed successfully but release is not published (Sep 21 at 11:51 AM)
17736 12:06p 🔵 v1.2.0 release stuck in Draft status despite successful build
17737 12:07p 🔵 Release workflow configured to create drafts by default via releaseDraft flag
S5286 Publish v1.2.0 release and modify workflow to auto-publish future releases instead of creating drafts (Sep 21 at 12:07 PM)
17738 " ✅ v1.2.0 published and workflow reconfigured to auto-publish future releases
### Sep 22, 2026
18023 3:49p 🔵 Repository state and recent feature history
18024 " 🔵 noscreen-app architecture exploration - UI components and backend commands
18025 3:50p 🔵 Security architecture and keyboard shortcut configuration
S5313 Code audit and feature exploration for noscreen-app — identify gaps and plan enhancements (Sep 22 at 3:50 PM)
18027 4:00p 🔵 Copilot database schema with session and transcript tracking
18028 " 🔵 TranscriptBuffer implements sliding-window transcript management with time-based purging
18029 4:01p 🟣 Copilot session summary persistence and custom preset support in database layer
18030 " 🟣 Expanded copilot presets with domain-specific coaching templates
18031 " 🔵 Test assertion mismatch: three_builtin_presets expects 3 presets but 8 now exist
18032 4:02p 🟣 User-defined custom copilot presets with full CRUD operations
18033 " ✅ Expanded builtin preset library from 3 to 8 domain-specific templates
18034 " 🔄 Exported unused copilot command handlers and added comprehensive test coverage
18035 4:03p 🟣 Integrated custom preset management and session summarization to frontend API layer
18036 4:04p 🟣 Added session summarization UI with async button state and error handling
18037 " ✅ Styled copilot session history with responsive layout and summary display
18038 " ✅ Added copilot custom preset management imports to HubSettings
18039 " 🟣 Implemented custom copilot preset management logic in HubSettings component

Access 887k tokens of past work via get_observations([IDs]) or mem-search skill.
</claude-mem-context>