# Phase 5: Advanced & Maintainer Requests

**Task:** Crash: WASM Memory Access Out of Bounds (Issue #14591) **Goal:**
Prevent `yoga-layout` WASM crash by validating inputs to `getComputedWidth`.
**Focus:** UI Layout, Error Prevention.

## Reproduction Steps

**Scenario:** `yoga-layout` (used by Ink) crashes with a WASM memory access
out-of-bounds error. **Status:** **Unknown / Difficult to Reproduce
Consistently.** _Potential Triggers:_

1. **Zero Width:** Resizing the terminal window to a width of 0 columns.
2. **Massive Output:** Rendering an extremely long single line of text without
   wrapping spaces.
3. **Invalid Component State:** Passing `NaN` or `Infinity` to layout
   calculation props. _Action:_ Since reproduction is flaky, rely on code
   analysis to prevent the _possibility_ of invalid inputs reaching the WASM
   boundary.

## Checklist

### 1. Investigation

- [ ] **Branch:** `git checkout -b fix/yoga-wasm-crash`.
- [ ] **Locate Code:** Find calls to `getComputedWidth` in `packages/cli`
      (likely in UI measuring logic).

### 2. Implementation

- [ ] **Add Guards:**
  - Before calling `yoga-layout` functions or `getComputedWidth`, check the
    input values.
  - Ensure values are not `NaN`, negative, or `Infinity`.
- [ ] **Architecture:** Implement the guard as a pure helper function if
      complex, or inline if simple. Avoid stateful logic for this check.

### 3. Verification

- [ ] **Test (Vitest):**
  - Add a test case specifically for the guard logic.
  - If the logic is inside a React component/hook, use `render` from
    `ink-testing-library`.
  - If it's a utility function, unit test it with `describe/it`.
  - **Mocking:** You might need to mock `yoga-layout` if you want to verify the
    _call_ is prevented, but testing the guard's return value is often enough.
- [ ] **Preflight:** `npm run preflight`.

### 4. Submit

- [ ] **Commit:** `fix(ui): prevent yoga layout crash on invalid width`.
- [ ] **PR:** Open PR linked to Issue #14591.
