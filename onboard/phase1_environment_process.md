# Phase 1: Environment & Process

**Task:** Docs Fixes (Issue #14782) **Goal:** Fix broken links and sidebar
configuration to fix the website build. **Prerequisites:**

- Node.js ~20.19.0
- Git

## Reproduction Steps

**Scenario:** The documentation build fails due to broken links and
configuration errors.

1. Run `npm run preflight` (or the specific docs build command).
2. **Expected:** Build passes successfully.
3. **Actual:** Build fails with errors pointing to `docs/changelogs/index.md`
   (broken link) and `docs/sidebar.json` (invalid path/typo).

## Checklist

### 1. Setup Environment

- [ ] Verify Node.js version: `node -v` (Should be ~20.19.0).
- [ ] Install dependencies: `npm install`.
- [ ] Verify build: `npm run build`.

### 2. Locate & Fix Issues

- [ ] **Fix Link:** Open `docs/changelogs/index.md`.
  - Find the broken link to `datacommons.org`.
  - Correct it (ensure it's a valid URL).
- [ ] **Fix Sidebar:** Open `docs/sidebar.json`.
  - Locate the entry `docs/changelog/releases`.
  - Correct the typo if necessary (ensure it points to the correct file path
    relative to `docs/`).
  - Remove `/index` suffixes from paths if present (e.g., change `folder/index`
    to `folder/`).

### 3. Verify Changes

- [ ] Run documentation build/check.
  - Command: `npm run preflight` (This runs linting, formatting, and tests. It
    is the gold standard for verification).
- [ ] Preview changes locally if possible (check `package.json` for a docs
      server command).

### 4. Submit Pull Request

- [ ] Create a new branch: `git checkout -b fix/docs-build-errors`.
- [ ] Stage changes: `git add docs/changelogs/index.md docs/sidebar.json`.
- [ ] Commit with Conventional Commits:
  ```bash
  git commit -m "fix(docs): resolve build errors and broken links"
  ```
- [ ] Push branch: `git push origin fix/docs-build-errors`.
- [ ] Open PR on GitHub linked to Issue #14782.
