# Gemini CLI Onboarding Plan

This document outlines a prioritized list of tasks to get onboarded to the
Gemini CLI repository, progressing from simple documentation fixes to complex
logic and UI improvements.

## Prioritized Task List

1.  **[Issue #14782] Docs: Fix Typos and Build Issues**
    - **Goal:** Fix broken links and sidebar configuration to fix the website
      build.
    - **Complexity:** ⭐ (Very Easy)
    - **Why:** Perfect "Hello World" task. Verifies you can build the docs, run
      tests, and submit a PR without touching complex code.

2.  **[Issue #14720] Hooks: Remove Deprecated Aliases**
    - **Goal:** Remove `permissionDecision` and `permissionDecisionReason`
      compatibility aliases from `BeforeToolHookOutput`.
    - **Complexity:** ⭐ (Easy)
    - **Why:** A clean code modification task. Introduces you to the
      `packages/core` structure and TypeScript types without logic risks.

3.  **[Issue #14750] Bug: Quota Reset Failure**
    - **Goal:** Ensure the daily quota usage resets correctly when a new day
      starts.
    - **Complexity:** ⭐⭐ (Medium)
    - **Why:** Dive into the core logic (state management, date handling). A
      contained bug with clear success criteria.

4.  **[Issue #14807] Bug: Windows "Screen Reader Mode" Persistence**
    - **Goal:** Handle "Access is denied" errors when writing the "nudge" file
      so the CLI respects user settings instead of crashing/reverting.
    - **Complexity:** ⭐⭐ (Medium)
    - **Why:** Practical error handling and file system work. Teaches you about
      the settings merging strategy.

5.  **[Issue #11487] Refactor: Consolidate Read File Tool UI**
    - **Goal:** Standardize `read_file` and `read_many_files` to return
      structured data and render a unified UI component.
    - **Complexity:** ⭐⭐ (Medium)
    - **Why:** Touches the full stack: Tool definition (Core) -> Execution -> UI
      Component (React/Ink).

6.  **[Issue #14591] Crash: WASM Memory Access Out of Bounds**
    - **Goal:** Prevent the `yoga-layout` WASM crash by validating inputs to
      `getComputedWidth`.
    - **Complexity:** ⭐⭐ (Medium/Hard)
    - **Why:** High visibility (opened by maintainer `jacob314`). Requires
      debugging the UI layout engine.

7.  **[Issue #13033] Feature: Native Text Selection**
    - **Goal:** Enable native mouse selection and copy in the CLI without
      needing a special mode key.
    - **Complexity:** ⭐⭐⭐ (Hard)
    - **Why:** High-value UX feature requested by `jacob314`. Deep dive into
      terminal events and `ink`.

8.  **[Issue #14421] Feature: Workspace Command Approvals**
    - **Goal:** Allow users to "Always allow in this workspace" to reduce
      approval fatigue.
    - **Complexity:** ⭐⭐⭐ (Hard)
    - **Why:** Significant architectural change involving persistence and
      security logic. Save for later.

---

## Execution Plan

### Phase 1: Environment & Process (Task 1)

**Task:** Docs Fixes (#14782)

1.  **Setup:** Ensure repo is cloned and `npm install` / `npm run build` works.
2.  **Locate:**
    - `docs/changelogs/index.md` (Fix `datacommons.org` link).
    - `docs/sidebar.json` (Fix `docs/changelog/releases` typo and remove
      `/index` suffixes).
3.  **Verify:** Run the docs build script (likely `npm run docs:build` or
    similar, check `package.json`).
4.  **PR:** Submit with title "fix(docs): resolve build errors and broken
    links".

### Phase 2: Codebase Familiarization (Task 2)

**Task:** Remove Aliases (#14720)

1.  **Locate:** `packages/core/src/hooks/types.ts`.
2.  **Edit:** Find `BeforeToolHookOutput` class. Remove getters/setters for
    `permissionDecision` and `permissionDecisionReason`.
3.  **Search:** Grep the codebase for usages of these aliases to ensure no
    breakages.
4.  **Verify:** Run `npm run test` in `packages/core`.

### Phase 3: Logic & State Debugging (Tasks 3 & 4)

**Task:** Quota Reset (#14750)

1.  **Investigate:** Find where quota is stored (likely `~/.gemini/state.json`
    or local storage). Search for "quota", "reset", or "limit".
2.  **Debug:** Check the date comparison logic. Is it using UTC vs Local time
    correctly? Is the "last reset date" being saved?
3.  **Fix:** Ensure the check compares the current date against the stored date
    correctly.
4.  **Test:** Unit test the reset function with mocked dates.

**Task:** Windows Nudge File (#14807)

1.  **Locate:** Code writing to `.gemini/tmp/seen_screen_reader_nudge.json`.
    Likely in `config/settings.ts` or a utility file.
2.  **Fix:** Wrap the `fs.writeFile` (or equivalent) in a `try/catch` block.
3.  **Fallback:** If writing fails, log a debug warning but **do not** crash or
    force the flag to `true`. Proceed with the existing `settings.json` value.

### Phase 4: Full Stack Refactoring (Task 5)

**Task:** Read File UI (#11487)

1.  **Define Type:** Add `FileReadSummary` interface to
    `packages/core/src/tools/tools.ts`.
2.  **Update Tools:** Modify `read_file.ts` and `read_many_files.ts` to return
    this object instead of a string.
3.  **Create Component:** Build `FileReadMessage.tsx` in
    `packages/cli/src/ui/components/messages/`.
4.  **Integrate:** Update `ToolMessage.tsx` to render the new component when it
    sees `FileReadSummary`.

### Phase 5: Advanced & Maintainer Requests (Tasks 6)

**Task:** WASM Crash (#14591)

1.  **Reproduce:** Try to find a scenario (resize window to 0? huge output?)
    that triggers it.
2.  **Guard:** Locate `getComputedWidth` calls. Add checks for `NaN`, negative,
    or infinite values before passing them to Yoga.

### Phase 6: Deep Dive UX (Task 7)

**Task:** Native Selection (#13033)

1.  **Research:** Study how `ink` handles mouse events and "Alternate Buffer
    Mode".
2.  **Prototype:** Experiment with disabling mouse reporting for scroll events
    specifically, or handling "click and drag" events manually to update the
    clipboard.

### Phase 7: Major Feature (Task 8)

**Task:** Workspace Command Approvals (#14421)

1.  **Design Schema:** Define a JSON structure for storing approved command
    signatures/hashes (e.g., in `.gemini/state.json` or a new
    `allowed_commands.json`).
2.  **Persistence Layer:** Implement functions to
    `approveCommand(workspace, command)` and
    `isCommandApproved(workspace, command)`.
3.  **UI Integration:** Update the `ConfirmationRequest` UI to add a new option:
    "Allow always in this workspace".
4.  **Enforcement:** Hook into the `ToolScheduler` or command execution pipeline
    to check this allowlist before prompting the user.
