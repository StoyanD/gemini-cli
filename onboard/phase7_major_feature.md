# Phase 7: Major Feature

**Task:** Workspace Command Approvals (Issue #14421) **Goal:** Allow users to
"Always allow in this workspace" to reduce approval fatigue. **Focus:**
Security, Persistence, UI.

## Checklist

### 1. Design

- [ ] **Branch:** `git checkout -b feat/workspace-command-approval`.
- [ ] **Architecture:**
  - Define a clean **Schema** for approvals (e.g., in `settings.schema.json` if
    applicable, or a standalone state file).
  - **Avoid Classes:** Implement the approval logic as a module of functions
    (e.g., `src/services/security/approval-service.ts`).
    - `export function approveCommand(...)`
    - `export function isCommandApproved(...)`

### 2. Persistence Layer

- [ ] **Implement Logic:**
  - Use `fs/promises` to read/write the state file.
  - Ensure atomic writes if possible to prevent corruption.

### 3. UI Integration

- [ ] **Update Prompt:**
  - Modify `ConfirmationRequest.tsx` (or similar).
  - Add "Allow always in this workspace" option.
  - **React:** Ensure state updates trigger a re-render/action.

### 4. Enforcement

- [ ] **Update Scheduler:**
  - Hook into `ToolScheduler`.
  - Call `isCommandApproved` before requesting user permission.

### 5. Verification

- [ ] **Test (Vitest):**
  - **Unit Test Logic:** Create `approval-service.test.ts`.
  - **Mocking:** Mock `fs` (or the settings persistence layer) using `vi.mock`
    to verify that approvals are saved and read correctly without touching the
    real disk.
    ```typescript
    vi.mock('fs/promises');
    ```
  - Verify that distinct commands are NOT treated as the same (hashing logic
    check).
- [ ] **Preflight:** `npm run preflight`.

### 6. Submit

- [ ] **Commit:**
      `feat(security): implement workspace-level command approval persistence`.
- [ ] **PR:** Open PR linked to Issue #14421.
