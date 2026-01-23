# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Bluesky Social app - a React Native application built on the AT Protocol (atproto). The codebase supports iOS, Android, and Web platforms using Expo. It connects to the AT Protocol network and implements the `app.bsky.*` Lexicon schemas.

## Essential Commands

### Development
- `yarn` - Install dependencies
- `yarn web` - Start web development server
- `yarn ios` - Run on iOS simulator (requires Xcode and `npx expo prebuild`)
- `yarn android` - Run on Android emulator (requires Android Studio and `npx expo prebuild`)
- `yarn start` - Start Expo dev client
- `npx expo prebuild` - Generate native projects (required after changes to `app.json` or native dependencies)

### Testing
- `yarn test` - Run Jest unit tests
- `yarn test-watch` - Run tests in watch mode
- `yarn test-ci` - Run tests in CI mode with JUnit output
- `yarn e2e:mock-server` - Start E2E mock server
- `yarn e2e:metro` - Start metro bundler for E2E tests
- `yarn e2e:run` - Run Maestro E2E tests

### Linting & Type Checking
- `yarn lint` - Run ESLint (caches results)
- `yarn typecheck` - Run TypeScript type checking (uses `tsconfig.check.json`)
- `yarn lint-native` - Lint native Swift/Kotlin code

### Internationalization
- `yarn intl:extract` - Extract strings for translation (English only)
- `yarn intl:extract:all` - Extract strings for all locales
- `yarn intl:compile` - Compile translations (required after `.po` file changes)
- `yarn intl:build` - Extract and compile translations

### Building
- `yarn build-web` - Build web app for production
- `yarn build` - Build native apps using EAS
- `npx expo prebuild` - Regenerate native folders (required after native changes)

## Architecture

### Module Organization

The codebase uses path aliases configured in `tsconfig.json` and `babel.config.js`:
- `#/*` maps to `./src/*` - use `#/` prefix for all absolute imports
- Platform-specific files use `.native.ts` or `.web.ts` extensions (web version should be the default without extension)

Key directories:
- `src/screens/` - Top-level screen components organized by feature
- `src/components/` - Reusable UI components
- `src/state/` - State management (React Query, context providers, persisted state)
- `src/lib/` - Utilities, hooks, and business logic
- `src/alf/` - Atomic Layout & Foundations design system (transitioning to `@bsky.app/alf` package)
- `src/view/` - Legacy view components (being migrated to `components/`)
- `src/platform/` - Platform-specific implementations
- `modules/` - Custom native modules (Expo modules)

### State Management

Uses **React Query** (TanStack Query) for server state with persistence:
- Query definitions in `src/state/queries/` organized by domain
- Stale time constants in `src/state/queries/index.ts`
- Persisted to AsyncStorage for offline support
- Context providers in `src/state/` for global state (session, preferences, modals, etc.)

### Navigation

Uses **React Navigation** with nested navigators:
- Main config in `src/Navigation.tsx`
- Bottom tab navigator with: Home, Search, Notifications, Messages, Profile
- Deep linking configured for `bsky.app` URLs
- Route types in `src/lib/routes/types.ts`

### Styling

Two styling systems coexist:
1. **ALF (Atomic Layout & Foundations)** - Modern design system in `src/alf/` and `@bsky.app/alf`
   - Theme-aware, uses React context
   - Typography, tokens, atoms, breakpoints
2. **Legacy styles** - StyleSheet-based styles in older components

Prefer ALF for new components. Text must be wrapped in ALF components (enforced by `bsky-internal/avoid-unwrapped-text` ESLint rule).

### Internationalization

Uses **Lingui** for i18n:
- Translations in `src/locale/locales/{locale}/messages.po`
- Extract strings with `yarn intl:extract`, compile with `yarn intl:compile`
- Use `msg` macro for strings: `msg\`Translation text\``
- Run `yarn intl:build` after changing translatable strings

### Forms & Validation

Uses **Zod** for schema validation throughout the codebase.

### Platform Detection

Import from `#/platform/detection`:
- `isWeb`, `isNative`, `isIOS`, `isAndroid`
- Never use web-only or native-only APIs without platform checks

### AT Protocol Integration

- `@atproto/api` package provides typed AT Protocol client
- Agent/session management in `src/state/session/`
- Service config (PDS URL) in `src/state/service-config.tsx`

## Code Standards

### Import Order

ESLint enforces import grouping (use `simple-import-sort` plugin):
1. Side effects
2. Node builtins (`node:*`)
3. React/React Native, then Expo, then other packages
4. Relative imports: `#/lib`, `#/state`, `#/logger`, `#/platform`, `#/locale`
5. Views: `#/view`
6. Screens: `#/screens`
7. ALF: `#/alf`
8. Components: `#/components`
9. Other `#/` imports
10. Relative `.` imports

### TypeScript

- Use inline type imports: `import {type Foo} from './foo'`
- Unused vars starting with `_` are allowed
- Use `#/` prefix for absolute imports, not relative paths

### Component Patterns

- Prefer functional components with hooks
- React Compiler is enabled (Babel plugin) - warnings shown for violations
- Use `observer` from MobX for reactive components (legacy pattern, avoid in new code)

### ESLint Rules

Custom rules (`bsky-internal/*`):
- `avoid-unwrapped-text` - Text must be wrapped in ALF typography components
- `use-exact-imports` - Import from exact paths, not barrel exports
- `use-typed-gates` - Use typed Statsig gates
- `use-prefixed-imports` - Use `#/` imports instead of relative

## Testing

### Unit Tests

- Jest with `jest-expo` preset
- Test utilities in `jest/test-utils.tsx`
- Mock setup in `jest/jestSetup.js`
- Run single test: `yarn test <path>`

### E2E Tests

- Maestro framework for E2E testing
- Test PDS (mock server) in `jest/test-pds.ts`
- E2E-specific files use `.e2e.ts` / `.e2e.tsx` extensions
- Three terminals required: mock server, metro bundler, maestro runner

## Native Development

### Setup Requirements

- Node.js >=20
- Xcode (iOS) with Command Line Tools
- Android Studio with SDK (Android)
- Ruby 2.7.6 via rbenv (for CocoaPods)
- Java 17 (zulu-17) for Android builds

### Native Modules

Custom Expo modules in `modules/`:
- `expo-bluesky-swiss-army` - Native utilities
- `expo-receive-android-intents` - Android intent handling
- `bottom-sheet` - Forked bottom sheet component
- Others for GIF view, emoji picker, notifications, etc.

Changes to native modules require:
1. `npx expo prebuild -p ios` or `npx expo prebuild -p android`
2. Rebuild with `yarn ios` or `yarn android`

### Configuration Files

- `app.config.js` - Expo configuration (app version, bundle IDs, etc.)
- `google-services.json` - Copy from `.example` (Firebase not required for dev)
- `.env` - Copy from `.env.example` (Sentry token optional)

## Common Tasks

### Running Single Test
```bash
yarn test src/path/to/file.test.ts
```

### Running Tests for Specific Pattern
```bash
yarn test --testNamePattern="pattern"
```

### Debugging Native Issues
- iOS: Clean build folder in Xcode, `rm -rf ios`, then `npx expo prebuild`
- Android: `adb reverse tcp:8081 tcp:8081` for metro, `adb reverse tcp:{PORT} tcp:{PORT}` for local services
- Check native logs: Xcode console (iOS) or `adb logcat` (Android)

### Accessing Localhost from Android Emulator
```bash
adb reverse tcp:PORT tcp:PORT
```

### Platform-Specific Code
- Web version: `file.ts` (no suffix)
- Native version: `file.native.ts`
- iOS-specific: `file.ios.ts`
- Android-specific: `file.android.ts`

## Git Workflow

- Pre-commit hooks via Husky run linting and formatting
- Lint-staged runs ESLint and Prettier on changed files
- SVG optimization runs on icon changes
- Main branch: `main`

## Deployment

- EAS (Expo Application Services) for builds
- OTA updates via `eas update` (governed by `runtimeVersion` in `app.config.js`)
- Web deployment: Go server in `bskyweb/` serves production web build
- Sentry sourcemaps uploaded automatically on EAS builds (requires `SENTRY_AUTH_TOKEN`)
