# Phase 6: Deep Dive UX

**Task:** Native Text Selection (Issue #13033) **Goal:** Enable native mouse
selection and copy in the CLI without needing a special mode key (Shift/Option).
**Focus:** `ink` library, Terminal events, Mouse reporting.

## Checklist

### 1. Research

- [ ] **Branch:** `git checkout -b feat/native-selection`.
- [ ] **Study Ink:** Investigate how `ink` handles mouse input (`useInput`
      hook).

### 2. Prototype & Implementation

- [ ] **Strategy:** Experiment with disabling mouse reporting for specific
      interactions or using "Alternate Buffer Mode" intelligently.
- [ ] **Architecture:** If introducing new hooks, ensure they follow the "Rules
      of Hooks" (top level, unconditional).
- [ ] **Implementation:** Apply fix in the main CLI entry point or root
      component.

### 3. Verification

- [ ] **Manual Test:**
  - Run `npm start`.
  - Try to select text with the mouse.
  - Try to copy text.
- [ ] **Automated Test:**
  - Testing terminal mouse events is hard. Focus on testing any _logic_ changes
    (e.g., state toggles) using standard Vitest unit tests.
  - Ensure `npm run preflight` passes.

### 4. Submit

- [ ] **Commit:** `feat(cli): enable native text selection`.
- [ ] **PR:** Open PR linked to Issue #13033.
