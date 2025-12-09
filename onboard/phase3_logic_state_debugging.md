# Phase 3: Logic & State Debugging

**Focus:** State management, Date handling, Error handling, File system.

## Task 1: Quota Reset Failure (Issue #14750)

**Goal:** Ensure the daily quota usage resets correctly when a new day starts.

### Reproduction Steps

**Scenario:** Daily quota does not reset at the start of a new day.

1. **Setup:** Manually edit `~/.gemini/state.json` (or equivalent persistence
   file) to set a high usage value for "today".
2. **Action:** Wait for the next day (or simulate it by changing system time).
3. **Execute:** Run any Gemini command.
4. **Expected:** Usage counter resets to 0.
5. **Actual:** Usage counter remains high, potentially blocking the user if they
   were near the limit. _(Note: Since waiting is impractical, use unit tests
   with `vi.useFakeTimers()` to reproduce reliably.)_

### Checklist

- [ ] **Branch:** `git checkout -b fix/quota-reset`.
- [ ] **Locate Logic:** Find where quota state is managed (search "quota" or
      check `~/.gemini/state.json` handling in `packages/core` or
      `packages/cli`).
- [ ] **Debug:**
  - Analyze the date comparison logic.
  - Check if it uses UTC or Local time consistently.
  - Ensure "last reset date" is stored and read correctly.
- [ ] **Fix:** Update the logic to correctly detect a new day and reset the
      counter.
- [ ] **Test (Vitest):**
  - Create or update a test file (e.g., `quota.test.ts`).
  - **Use Fake Timers:** Use `vi.useFakeTimers()` to simulate the passage of
    time and crossing the day boundary.
  - Example:
    ```typescript
    vi.useFakeTimers();
    vi.setSystemTime(new Date('2023-10-01T23:59:59'));
    // ... perform action ...
    vi.advanceTimersByTime(2000); // Advance to next day
    // ... verify quota reset ...
    vi.useRealTimers();
    ```
- [ ] **Verify:** `npm run test` and `npm run preflight`.
- [ ] **Commit:** `fix(core): ensure quota resets correctly on new day`.

## Task 2: Windows "Screen Reader Mode" Persistence (Issue #14807)

**Goal:** Handle "Access is denied" errors when writing the "nudge" file.

### Reproduction Steps

**Scenario:** CLI crashes or malfunctions on Windows when unable to write to the
temp directory.

1. **Setup:** On a Windows machine (or mocked environment), set the
   `.gemini/tmp` directory (or the specific nudge file) to "Read Only" or deny
   write permissions to the current user.
2. **Execute:** Run the Gemini CLI.
3. **Expected:** CLI starts normally, perhaps logging a silent warning.
4. **Actual:** CLI crashes with an `EACCES` or "Access is denied" error, or
   forcefully toggles the screen reader setting incorrectly.

### Checklist

- [ ] **Branch:** `git checkout -b fix/windows-nudge-crash`.
- [ ] **Locate Logic:** Find code writing to
      `.gemini/tmp/seen_screen_reader_nudge.json`.
- [ ] **Refactor:**
  - Wrap the `fs.writeFile` (or equivalent) in a `try/catch` block.
  - In the `catch` block, log a debug warning.
  - **Crucial:** Do NOT crash the application.
- [ ] **Test (Vitest):**
  - Create a test case that mocks `fs` to throw an error.
  - **Mocking:**
    ```typescript
    import fs from 'fs/promises';
    vi.mock('fs/promises');
    // In test:
    vi.mocked(fs.writeFile).mockRejectedValueOnce(
      new Error('EACCES: permission denied'),
    );
    // Call function and expect NO throw
    ```
- [ ] **Verify:** `npm run preflight`.
- [ ] **Commit:**
      `fix(cli): handle permission error for screen reader nudge file`.
