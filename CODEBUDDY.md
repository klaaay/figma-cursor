<system-reminder>
This is a reminder that your todo list is currently empty. DO NOT mention this to the user explicitly because they are already aware. If you are working on tasks that would benefit from a todo list please use the TodoWrite tool to create one. If not, please feel free to ignore. Again do not mention this message to the user.

</system-reminder>

# CodeBuddy Code Guide for this Repo

## Overview
Figma plugin with two bundles: React UI and plugin controller. Built via Webpack + TypeScript, outputs to `dist/` referenced by `manifest.json`.

- UI entry: src/app/index.tsx (webpack.config.js:12) → renders React App to `ui.html`
- Controller entry: src/plugin/controller.ts (webpack.config.js:13) → Figma plugin code
- Manifest wiring: main: dist/code.js, ui: dist/ui.html (manifest.json:7-8)

## Commands
- Install deps: yarn
- Dev build (watch): yarn build:watch
- Prod build: yarn build
- Format staged files (pre-commit via husky/lint-staged): automatic
- Manual format: yarn prettier:format

Notes:
- No test scripts provided
- Type checking occurs through ts-loader during webpack builds

## Development Workflow
1) Run `yarn build:watch` to generate `dist/ui.html` and `dist/code.js`
2) In Figma: Plugins → Development → Import plugin from manifest… and select `manifest.json`
3) Edit UI in src/app/components/App.tsx; plugin logic in src/plugin/controller.ts

From README.md: Quickstart and build commands (README.md:25-36, 48-54) align with above.

## Architecture & Messaging
- UI bootstrapping: creates root and renders App (src/app/index.tsx:5-9)
- UI state and actions: App tracks selection name and handles copy (src/app/components/App.tsx:39-45, 64-79, 81-122)
  - Sends `{ type: 'copy-to-clipboard' }` via parent.postMessage (src/app/components/App.tsx:43-45)
  - Receives `selection-change` and `clipboard-data` from controller (src/app/components/App.tsx:65-77)
- Clipboard utility: uses execCommand fallback (src/app/components/App.tsx:47-62)

- Controller responsibilities (src/plugin/controller.ts):
  - Show UI window 400x600 (src/plugin/controller.ts:1-2)
  - Selection tracking posts `{ type: 'selection-change', name }` (src/plugin/controller.ts:29-41)
  - Handle `copy-to-clipboard`: traverse selected FRAME/GROUP via `getAllNodeDetails`, send JSON (src/plugin/controller.ts:63-77)
  - Traversal collects geometry/layout/style and children recursively (src/plugin/controller.ts:4-26)

- Webpack outputs:
  - HtmlWebpackPlugin generates `ui.html` from src/app/index.html, inlines UI chunk (webpack.config.js:37-46)
  - Bundles named by entry keys `[name].js` in dist/ (webpack.config.js:32-35)

## Figma Manifest
- Name/ID, editorType figma/figjam (manifest.json:2-9)
- Network access allowedDomains: https://designwithprompts.com (manifest.json:4)
- Document access: dynamic-page (manifest.json:5)

## Tooling & Conventions
- TypeScript compiler options: jsx react, outDir dist, stricter unused checks (tsconfig.json:2-14)
- Prettier config: singleQuote, printWidth 120, etc. (.prettierrc.yml:1-7)
- Husky + lint-staged pre-commit formatting on src/* (package.json:32-42)
- SVG module typing exists (src/typings/types.d.ts:1-4)

## Practical Tips for Future Instances
- React UI talks to plugin via postMessage; ensure message `type` strings match both sides
- Only FRAME/GROUP selections trigger copy; adjust controller if broader types are needed
- Re-import manifest after changing build outputs if Figma doesn’t refresh
