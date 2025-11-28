# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MultiPost Markdown Editor is a WeChat-focused Markdown editor that renders documents into WeChat-compatible HTML. It integrates with the MultiPost browser extension to publish content across multiple platforms (知乎, 微博, 小红书, 抖音).

Based on [doocs/md](https://github.com/doocs/md) with custom modifications.

## Dual Build Targets

This project builds **both a web application and a browser extension** from the same codebase:

### Web Application

- **Development**: `npm start` or `npm run dev`
- **Build**: `npm run build` (deploys to `/md` directory)
- **Build for root**: `npm run build:h5-netlify` (sets `SERVER_ENV=NETLIFY`, deploys to `/`)
- **Preview**: `npm run preview`

### Browser Extension

- **Development**: `npm run ext:dev` (Chrome) or `npm run firefox:dev` (Firefox)
- **Package**: `npm run ext:zip` or `npm run firefox:zip`
- Entry point: `wxt.config.ts`
- Extension entrypoints: `src/entrypoints/`

## Development Commands

```sh
# Install dependencies
npm i

# Start dev server (web app)
npm start

# Build web app for /md path
npm run build

# Build web app for root path
npm run build:h5-netlify

# Type check
npm run type-check

# Lint and auto-fix
npm run lint

# Build extension for Chrome
npm run ext:dev    # Development
npm run ext:zip    # Package

# Build extension for Firefox
npm run firefox:dev
npm run firefox:zip

# Analyze bundle size
npm run build:analyze
```

## Architecture

### State Management (Pinia)

Two main stores in `src/stores/index.ts`:

1. **useStore**: Main application state
   - Editor content and CodeMirror instances (`editor`, `cssEditor`)
   - Theme/styling options (`theme`, `fontFamily`, `fontSize`, `primaryColor`, `codeBlockTheme`)
   - Content management (`posts`, `currentPostIndex`)
   - Feature toggles (`isMacCodeBlock`, `isCiteStatus`, `isCountStatus`, `isUseIndent`)
   - Actions for importing/exporting Markdown and HTML

2. **useDisplayStore**: UI visibility state
   - Dialog visibility (`isShowCssEditor`, `isShowInsertFormDialog`, `isShowUploadImgDialog`)

### Rendering Pipeline

The Markdown rendering flow (see `src/utils/renderer.ts`):

1. **Parse front matter** and extract metadata
2. **Convert Markdown to HTML** using marked.js with custom renderer
3. **Apply theme styles** based on user selections (fonts, colors, sizes)
4. **Process special elements**:
   - Code blocks with highlight.js
   - Math equations with KaTeX
   - Mermaid diagrams
   - GFM alerts (via `src/utils/MDAlert.ts`)
5. **Inject custom CSS** from the CSS editor
6. **Sanitize output** with DOMPurify
7. **Add WeChat-specific features**:
   - External link footnotes (when `isCiteStatus` is enabled)
   - Mac-style code block decorations
   - Reading time and word count statistics

### Component Structure

- **Main app**: `src/App.vue` → `src/views/CodemirrorEditor.vue`
- **UI components**: `src/components/ui/` (Radix Vue components with Tailwind)
- **Custom components**: `src/components/` (FormItem, CustomUploadForm, etc.)
- **CodemirrorEditor**: `src/components/CodemirrorEditor/`

### Configuration & Build

- **Vite config**: `vite.config.ts`
  - Uses UnoCSS, auto-imports from Pinia/VueUse
  - Base path: `/md/` (default) or `/` (NETLIFY mode)
  - Polyfills for Node.js modules (path, util, stream, fs)

- **WXT config**: `wxt.config.ts` (browser extension)
  - Custom module: `src/modules/build-extension.ts`
  - Handles inline scripts (CSP compliance), chunking, HTML transformation

- **TypeScript**: Project references (`tsconfig.json` → `tsconfig.app.json`, `tsconfig.node.json`)

- **ESLint**: `eslint.config.mjs` using @antfu/eslint-config

### Auto-imports

The following are auto-imported (no need to import manually):

- Vue 3 APIs (ref, computed, onMounted, etc.)
- Pinia (defineStore, storeToRefs, etc.)
- VueUse composables (@vueuse/core)
- Stores from `src/stores/`
- Toast utilities from `src/utils/toast/`

### Theme System

Themes are defined in `src/config/index.ts` as `themeMap`:

- Each theme has `base`, `block`, and `inline` styles
- Themes are merged with user preferences (font, size, color)
- Custom CSS can override any theme via the CSS editor
- Process: `css2json()` → `customCssWithTemplate()` → `customizeTheme()` → applied to renderer

### Content Storage

- Uses `useStorage()` from VueUse for localStorage persistence
- All keys prefixed via `addPrefix()` function
- Content stored as `posts` array with `{ title, content }`
- CSS editor supports multiple tabs/schemes (`cssContentConfig`)

## Key Files

- `src/stores/index.ts` - State management
- `src/utils/renderer.ts` - Markdown → HTML rendering engine
- `src/utils/index.ts` - Utility functions (CSS parsing, export, formatting)
- `src/config/index.ts` - Theme definitions and constants
- `src/main.ts` - App entry point (web)
- `src/modules/build-extension.ts` - Custom WXT module for extension builds
- `wxt.config.ts` - Browser extension configuration
- `vite.config.ts` - Web app build configuration

## Extension Integration

The browser extension and web app share most code but have different entry points:

- Web: `index.html` → `src/main.ts`
- Extension options page: `index.html` (reused)
- Extension popup: `src/entrypoints/popup/`
- Extension background: `src/entrypoints/background.ts`

Extension build uses custom WXT module to:

- Extract inline scripts for CSP compliance
- Chunk large dependencies (Prettier, highlight.js)
- Handle virtual modules during development

## Working with Markdown Content

When modifying the editor or renderer:

1. Changes to editor update `posts[currentPostIndex].content`
2. Content is auto-saved to localStorage via reactive watchers
3. Call `editorRefresh()` to re-render preview
4. Use `formatContent()` to run Prettier on Markdown
5. Export functions: `exportEditorContent2MD()`, `exportEditorContent2HTML()`

## CSS Customization

The CSS editor (CodeMirror instance) allows users to override theme styles:

- Stored in `cssContentConfig.tabs`
- Supports multiple schemes (tabs)
- Auto-hints enabled for CSS properties
- Format with `Shift-Alt-F`
- Changes apply in real-time via `updateCss()`
