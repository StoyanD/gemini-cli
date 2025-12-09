# Phase 4: Full Stack Refactoring

**Task:** Consolidate Read File Tool UI (Issue #11487) **Goal:** Standardize
`read_file` and `read_many_files` to return structured data and render a unified
UI component. **Focus:** `packages/core` (Tools), `packages/cli` (UI/Ink).

## Checklist

### 1. Core Changes

- [ ] **Branch:** `git checkout -b feat/read-file-ui`.
- [ ] **Define Interface:**
  - Edit `packages/core/src/tools/tools.ts` (or relevant types file).
  - **Architecture:** Create a `FileReadSummary` **interface** (not a class).
    ```typescript
    export interface FileReadSummary {
      filePath: string;
      size: number;
      contentPreview?: string;
      // ...
    }
    ```
- [ ] **Update Tools:**
  - Modify `read_file.ts` to return `FileReadSummary` object instead of raw
    string string.
  - Modify `read_many_files.ts` to return `FileReadSummary[]`.
  - **Testing:** Update `read-file.test.ts` and `read-many-files.test.ts` to
    assert the new object structure.

### 2. UI Implementation

- [ ] **Create Component:**
  - Create `packages/cli/src/ui/components/messages/FileReadMessage.tsx`.
  - **Architecture:** Use a **Functional Component** with Hooks if needed.
  - `export const FileReadMessage = ({ summary }: { summary: FileReadSummary }) => { ... };`
- [ ] **Test Component:**
  - Create `packages/cli/src/ui/components/messages/FileReadMessage.test.tsx`.
  - **Library:** Use `render` from `ink-testing-library`.
  - **Assertion:** Check `lastFrame()` output.
- [ ] **Integrate Component:**
  - Edit `packages/cli/src/ui/components/messages/ToolMessage.tsx` (or parent
    component).
  - Add logic to render `FileReadMessage` when the tool output matches
    `FileReadSummary`.

### 3. Verification

- [ ] **Manual Test:**
  - Run `npm start`.
  - Use the agent to read a file (`read_file file_path="README.md"`).
  - Verify the new UI component appears.
- [ ] **Automated Test:** `npm run preflight`.

### 4. Submit

- [ ] **Commit:**
      `feat(ui): standardize read file tool output and visualization`.
- [ ] **PR:** Open PR linked to Issue #11487.
