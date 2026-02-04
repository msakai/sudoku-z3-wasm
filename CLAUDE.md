# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a browser-based Sudoku solver application using the Z3 theorem prover. The application is built with Vite and vanilla JavaScript.

## Development

**Setup:**
```bash
npm install
```

**Running the application:**
```bash
npm run dev
```

Open http://localhost:5173/ in your browser.

**Production build:**
```bash
npm run build
npm run preview
```

The `npm run build` command automatically runs `copy-assets` which copies Z3 WASM files and coi-serviceworker from node_modules to the public directory.

## Deployment

**GitHub Pages:**

- Automatic deployment via GitHub Actions on push to `main`
- Workflow file: `.github/workflows/deploy.yml`
- Live URL: <https://msakai.github.io/sudoku-z3-wasm/>

**coi-serviceworker:**
GitHub Pages cannot set custom HTTP headers. The app uses coi-serviceworker to inject COOP/COEP headers client-side via a service worker, enabling SharedArrayBuffer for Z3.

## Coding Guidelines

**Language:** All text content, comments, and user-facing messages should be written in English unless explicitly instructed otherwise.

## Architecture

**File structure:**

- `index.html`: Main HTML with UI and script loaders
- `src/main.js`: UI and grid management logic
- `src/solver.js`: Z3-based Sudoku solver
- `public/coi-serviceworker.js`: Service worker for COOP/COEP headers (copied from node_modules)
- `public/z3-built.js`: Z3 WASM loader (copied from node_modules)
- `public/z3-built.wasm`: Z3 WASM binary (~33MB, copied from node_modules)
- `vite.config.js`: Vite config with COOP/COEP headers, base path, and global polyfill
- `.github/workflows/deploy.yml`: GitHub Actions workflow for GitHub Pages deployment

**Key components in src/main.js:**

- **Grid management:**
  - `initGrid()`: Dynamically generates 81 input elements for the 9x9 grid
  - `getGrid()`: Converts DOM state to a 9x9 array representation
  - `setGrid()`: Updates DOM from array representation

- **Visual feedback:**
  - `.initial` class: User's original input (gray background)
  - `.solved` class: Auto-generated solution (green background)
  - 3x3 block boundaries use CSS nth-child selectors

**Key components in src/solver.js:**

- `initZ3()`: Initializes the Z3 solver (async, called on page load)
- `solveSudoku(grid)`: Solves the puzzle using Z3 constraints (async)

**Z3 Constraint Encoding:**
- Each cell is an `Int` variable constrained to 1-9
- `Distinct()` constraints for each row, column, and 3x3 block
- Initial values are fixed with equality constraints
- `solver.check()` returns 'sat' if solvable, then `solver.model()` extracts values

**State management:**
- `initialCells` Set tracks which cells were user-provided vs auto-solved
- No framework - direct DOM manipulation

## Technical Notes

**SharedArrayBuffer requirement:**
Z3-solver uses WebAssembly threads which require `SharedArrayBuffer`. This needs special HTTP headers:

- `Cross-Origin-Opener-Policy: same-origin`
- `Cross-Origin-Embedder-Policy: require-corp`

For local development, these are configured in `vite.config.js`. For GitHub Pages (which cannot set custom headers), coi-serviceworker injects these headers client-side via a service worker.

**Script loading order:**
Scripts in `index.html` must be loaded in this order:

1. `coi-serviceworker.js` - Registers service worker for COOP/COEP headers
2. `z3-built.js` - Sets up `window.initZ3` required by z3-solver
3. `./src/main.js` (module) - Application entry point

**Z3 initialization:**
The `z3-built.js` must be loaded via a script tag before ES modules, as it sets up `window.initZ3` required by z3-solver's browser build.

**GitHub Pages base path:**
The `vite.config.js` uses a dynamic base path: `/sudoku-z3-wasm/` when `GITHUB_ACTIONS` env var is set, otherwise `/` for local development.
