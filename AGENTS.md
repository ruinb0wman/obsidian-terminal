# AGENTS.md — AI Coding Agent Guide for obsidian-terminal

This guide provides clear, actionable instructions for AI coding agents working in the `obsidian-terminal` codebase. This is an Obsidian plugin that integrates consoles, shells, and terminals into Obsidian.

## 1. Project Overview

**obsidian-terminal** is an Obsidian plugin that provides:

- Integrated terminal emulation using xterm.js
- External terminal launching (platform-specific)
- An emulated developer console usable on all platforms
- Support for multiple terminal profiles
- History restoration and terminal state persistence

### Technology Stack

| Component | Technology |
| --------- | ---------- |
| Frontend | TypeScript, Svelte |
| Terminal Engine | xterm.js with addons (canvas, webgl, ligatures, search, unicode11, web-links) |
| Backend (Python) | Python 3.9+, psutil, pywinctl, uvloop/winloop |
| Build Tool | esbuild with custom scripts |
| Testing | Vitest (TypeScript), pytest (Python) |
| Package Manager | pnpm (preferred) or npm |
| Python Environment | uv |

### Key Dependencies

- `@polyipseity/obsidian-plugin-library`: Core library for Obsidian plugin functionality
- `@xterm/xterm` and addons: Terminal emulation
- `svelte`: UI framework
- `i18next`: Internationalization
- `acorn` + `espree`: JavaScript parsing for developer console

## 2. Architecture

### Directory Structure

```text
src/
├── main.ts                    # Plugin entry point (TerminalPlugin class)
├── settings-data.ts           # Settings interfaces and validation (.fix functions)
├── settings.ts                # Settings UI (SettingTab)
├── documentations.ts          # Documentation management
├── icons.ts                   # Icon registration
├── import.ts                  # Dynamic import helpers
├── magic.ts                   # Constants and magic numbers
├── modals.ts                  # Modal dialogs
├── patch.ts                   # Logging interception patches
├── utils.ts                   # Utility functions
├── @types/                    # Type definitions
│   ├── obsidian-terminal.ts   # Public API types
│   └── *.ts                   # Internal type definitions
└── terminal/                  # Terminal implementation
    ├── load.ts                # Terminal feature registration
    ├── view.ts                # Terminal view (Obsidian ItemView)
    ├── emulator.ts            # XtermTerminalEmulator wrapper
    ├── emulator-addons.ts     # Custom xterm.js addons
    ├── pseudoterminal.ts      # Pseudoterminal implementations
    ├── spawn.ts               # Terminal spawning logic
    ├── options.ts             # Terminal options merging
    ├── profile-presets.ts     # Default profile configurations
    ├── profile-properties.ts  # Profile type definitions
    ├── utils.ts               # Terminal utilities
    ├── unix_pseudoterminal.py # Unix PTY helper (Python)
    └── win32_resizer.py       # Windows terminal resizer (Python)

assets/
├── locales.ts                 # Localization setup
└── locales/                   # Translation files (51 languages)
    └── */translation.json

scripts/                       # Build and utility scripts
├── build.mjs                  # Main esbuild script
├── obsidian-install.mjs       # Install to vault script
├── version.mjs                # Version bumping
├── sync-locale-keys.mjs       # Locale synchronization
└── utils.mjs                  # Script utilities

tests/                         # Test suite
├── *.spec.ts                  # Unit tests
├── *.test.ts                  # Integration tests
├── mocks/                     # Test mocks
│   └── obsidian.ts            # Obsidian API mock
├── setup.ts                   # Test setup
├── scripts/                   # Script tests
└── support/                   # Test helpers
```

### Core Classes

1. **TerminalPlugin** (`src/main.ts`): Main plugin class extending Obsidian's `Plugin`
   - Manages lifecycle: language, settings, terminal views
   - Implements `PluginContext<Settings, LocalSettings>`

2. **TerminalView** (`src/terminal/view.ts`): Obsidian view for terminal display
   - Extends `ItemView`
   - Manages xterm.js terminal instance
   - Handles terminal lifecycle (spawn, resize, destroy)

3. **XtermTerminalEmulator** (`src/terminal/emulator.ts`): Wrapper around xterm.js
   - Manages terminal addons
   - Handles input/output piping

4. **Pseudoterminal classes** (`src/terminal/pseudoterminal.ts`):
   - `Pseudoterminal`: Interface for PTY implementations
   - `DeveloperConsolePseudoterminal`: In-app JS console
   - `WindowsPseudoterminal`: Windows shell integration
   - `UnixPseudoterminal` (via Python script): Unix PTY

5. **Settings** (`src/settings-data.ts`):
   - `Settings`: Main plugin settings
   - `Settings.Profile`: Terminal profile definitions
   - `Settings.fix()`: Validation/normalization function

### Profile System

Terminal profiles define how terminals are spawned:

| Profile Type       | Description                           |
| ------------------ | ------------------------------------- |
| `integrated`       | Runs shells inside Obsidian using PTY |
| `external`         | Launches external terminal emulators  |
| `developerConsole` | In-app JavaScript console             |
| `empty`            | Placeholder/deleted profiles          |

Profile presets are defined in `src/terminal/profile-presets.ts`.

## 3. Developer Workflows

> **Note:** **Always prefer `pnpm` over `npm`** for development workflows. Use `npm` only when `pnpm` is unavailable.

### Setup

```bash
pnpm install           # Install dependencies and set up Git hooks
```

### Build Commands

```bash
pnpm build             # Production build (runs checks then builds)
pnpm run build:dev     # Development/watch build
pnpm run build:force   # Build without checks
```

### Testing

```bash
pnpm test              # Run full test suite (Vitest + pytest)
pnpm run test:vitest   # Run TypeScript tests only
pnpm run test:py       # Run Python tests only
pnpm run test:watch    # Interactive test watcher (not for CI)
```

**Test naming conventions:**

- `*.spec.ts` — Unit tests (BDD-style, fast, isolated)
- `*.test.ts` — Integration tests (TDD-style, may use filesystem)

### Linting and Formatting

```bash
pnpm run check         # Run all checks (ESLint, Prettier, markdownlint, Pyright)
pnpm run check:eslint  # TypeScript/JavaScript linting
pnpm run check:prettier # Format checking
pnpm run check:md      # Markdown linting
pnpm run check:py      # Python linting (ruff + pyright)

pnpm run format        # Auto-fix all formatting
pnpm run format:eslint # Fix ESLint issues
pnpm run format:prettier # Format with Prettier
pnpm run format:md     # Fix Markdown
pnpm run format:py     # Format Python (ruff)
```

### Obsidian Installation

```bash
pnpm obsidian:install /path/to/vault       # Build and install
pnpm run obsidian:install:force /path/to/vault  # Skip checks
```

### Version Management

This project uses [`changesets`](https://github.com/changesets/changesets) for changelog management:

```bash
pnpm run version       # Bump version using changesets
```

## 4. Coding Conventions

### TypeScript

- **No `any` type**: Prefer `unknown` with type guards
- **No `as` casting**: Use runtime type guards instead
- **Prefer `interface` for object shapes**: Better for incremental TypeScript performance
- **Strict null checks enabled**: Always handle null/undefined cases
- **Use `.fix()` functions for validation**: See `Settings.fix()`, `LocalSettings.fix()` patterns

Example:

```typescript
// Good: Interface for object shapes
interface TerminalOptions {
  fontSize?: number;
  cursorStyle?: 'block' | 'bar' | 'underline';
}

// Good: Type guard instead of casting
function isProfile(obj: unknown): obj is Settings.Profile {
  return typeof obj === 'object' && obj !== null && 'type' in obj;
}
```

### Imports

- **Always use top-level static imports** where possible
- Place imports at the very top of the file (after any file-level doc comment)
- Use `import type` for type-only imports
- Dynamic imports (`await import()`) only when necessary (module isolation, mocks)

### Python

- **Minimum version**: Python 3.9+
- **Type checking**: Pyright in strict mode
- **Linting**: Ruff with line-length 88
- **Every module must declare `__all__`** (tuple, not list)
- **Windows-specific dependencies**: psutil, pywinctl, typing_extensions

Example:

```python
__all__ = ("spawn_terminal", "resize_terminal")

async def spawn_terminal() -> None:
    ...
```

### Localization

The plugin supports 51 languages. When adding new text:

1. Add to `assets/locales/en/translation.json` first
2. Use the i18n key pattern: `category.subcategory.key`
3. Access via: `i18n.t("category.subcategory.key")`
4. Run `pnpm run sync-locale-keys` to propagate to other languages

Example:

```json
// assets/locales/en/translation.json
{
  "settings": {
    "terminal-options": "Terminal options"
  }
}
```

```typescript
// Usage
const text = i18n.t("settings.terminal-options");
```

## 5. Testing Guidelines

### Vitest Best Practices

- Use `vi.fn()` for spies and stubs
- Use `vi.mocked()` for typed module mocks
- Use `vi.useFakeTimers()` for timer-based tests
- Reset mocks in `afterEach`: `vi.restoreAllMocks()`

Example:

```typescript
import { vi, describe, it, expect, afterEach } from "vitest";

afterEach(() => {
  vi.restoreAllMocks();
});

it("should call handler", () => {
  const handler = vi.fn();
  triggerEvent(handler);
  expect(handler).toHaveBeenCalledOnce();
});
```

### Test File Organization

Follow the **one test file per source file** convention:

| Source                            | Test                                         |
| --------------------------------- | -------------------------------------------- |
| `src/magic.ts`                    | `tests/src/magic.spec.ts`                    |
| `src/settings-data.ts`            | `tests/src/settings-data.spec.ts`            |
| `src/terminal/emulator-addons.ts` | `tests/src/terminal/emulator-addons.spec.ts` |

### Obsidian API Mocking

Use the provided mock at `tests/mocks/obsidian.ts`:

```typescript
// In test file
vi.mock("obsidian", async () => await vi.importActual("../mocks/obsidian.js"));

// Setup test data
const app = new App();
// ... use mocked Obsidian API
```

## 6. Build System

The build is managed by `scripts/build.mjs` using esbuild:

- **Entry points**: `src/main.ts`, `src/styles.css`
- **Output**: `main.js`, `styles.css` (in project root)
- **Format**: CommonJS (for Obsidian compatibility)
- **Target**: ES2018
- **External**: obsidian, electron, node:* modules, @codemirror/*, @lezer/*
- **Special loaders**:
  - `.json` — bundled as JSON
  - `.md`, `.py` — bundled as lazy-loaded text

### Development Mode

```bash
pnpm run build:dev
```

Watches files and rebuilds automatically with inline source maps.

### Production Mode

```bash
pnpm build
```

Runs checks, formats code, and creates optimized bundles.

## 7. Commit Message Convention

All commits must follow [Conventional Commits](https://www.conventionalcommits.org/):

```text
type(scope): short summary (≤72 chars)

Optional body (wrap at 100 chars)

Optional footer (BREAKING CHANGE, Refs, etc)
```

**Types:**

- `feat`: New feature
- `fix`: Bug fix
- `chore`: Maintenance/build/tooling
- `docs`: Documentation only
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `test`: Adding or fixing tests
- `style`: Formatting changes
- `perf`: Performance improvement

**Scopes:**

- `terminal`: Terminal functionality
- `settings`: Settings UI/data
- `build`: Build scripts
- `deps`: Dependencies
- `i18n`: Translations
- `profiles`: Profile system

Example:

```text
feat(terminal): add support for custom color schemes

- Added theme picker to profile settings
- Implemented color scheme validation

Refs: #123
```

Run `pnpm run commitlint` to validate commit messages locally.

## 8. Integration Points

### Obsidian API

- Peer dependency via `@polyipseity/obsidian` (type definitions)
- Runtime API from Obsidian app
- View types registered via `registerView()`

### @polyipseity/obsidian-plugin-library

Provides:

- `LanguageManager` — i18n management
- `SettingsManager` — Settings persistence
- `PluginContext` — Plugin interface contract
- UI helpers — Modals, notices, settings UI
- Utility functions — Platform detection, DOM helpers

### xterm.js

Terminal emulation with these addons:

- `@xterm/addon-canvas` — Canvas renderer
- `@xterm/addon-webgl` — WebGL renderer
- `@xterm/addon-fit` — Auto-fit to container
- `@xterm/addon-search` — Find in terminal
- `@xterm/addon-ligatures` — Font ligatures
- `@xterm/addon-unicode11` — Unicode 11 support
- `@xterm/addon-web-links` — Clickable links
- `@xterm/addon-serialize` — Terminal state serialization

## 9. Security Considerations

1. **Command Injection**: Profile executables and arguments are user-configured. The plugin passes these directly to `child_process.spawn()`.
2. **Python Scripts**: The plugin embeds Python helper scripts (`unix_pseudoterminal.py`, `win32_resizer.py`) that execute with user-specified Python interpreter.
3. **Developer Console**: Evaluates arbitrary JavaScript in the Obsidian context. This is intentional but sandboxed from system access.
4. **Module Exposure**: The `exposeInternalModules` setting patches `require()` to expose Obsidian internals for the developer console.

## 10. Common Tasks

### Adding a New Setting

1. Add to `Settings` interface in `src/settings-data.ts`
2. Add default value in `Settings.DEFAULT`
3. Add validation in `Settings.fix()`
4. Add UI in `src/settings.ts`
5. Add translation keys in `assets/locales/en/translation.json`
6. Add tests in `tests/src/settings-data.spec.ts`

### Adding a Terminal Addon

1. Install: `pnpm add @xterm/addon-<name>`
2. Import in `src/terminal/emulator.ts` or `src/terminal/emulator-addons.ts`
3. Load addon in `XtermTerminalEmulator` or custom addon class
4. Add tests

### Adding a Translation

1. Copy `assets/locales/en/` to `assets/locales/<lang>/`
2. Update `assets/locales/<lang>/language.json` with native name
3. Translate `translation.json`
4. Add to `PluginLocales.RESOURCES` in `assets/locales.ts`
5. Run `pnpm run sync-locale-keys` to validate

## 11. Troubleshooting

### Build Issues

- **esbuild errors**: Check `package.json` for correct dependency versions
- **Type errors**: Run `pnpm run check:eslint` and `pnpm run check:py`
- **Lockfile issues**: Delete `node_modules` and `pnpm-lock.yaml`, then `pnpm install`

### Runtime Issues

- **Terminal not spawning**: Check Python installation (3.9+) and executable path in profile settings
- **Windows terminal resize not working**: Ensure `pywinctl` and `psutil` are installed
- **Developer console not evaluating**: Check browser console for syntax errors in evaluated code

### Testing Issues

- **Mock not working**: Ensure `vi.mock()` is at the top of the test file
- **Type errors in tests**: Check `tests/tsconfig.test.json` configuration

---

For additional context, see:

- `README.md` — User-facing documentation
- `CHANGELOG.md` — Release history
- `.agents/instructions/` — Detailed instruction files for specific topics
