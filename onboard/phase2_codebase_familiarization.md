# Phase 2: Codebase Familiarization

**Task:** Remove Deprecated Aliases (Issue #14720) **Goal:** Remove
`permissionDecision` and `permissionDecisionReason` compatibility aliases from
`BeforeToolHookOutput`. **Focus:** `packages/core`, TypeScript types.

## Checklist

### 1. Preparation

- [ ] Update main branch: `git checkout main && git pull`.
- [ ] Create feature branch: `git checkout -b chore/remove-hook-aliases`.

### 2. Refactor Code

- [ ] Locate file: `packages/core/src/hooks/types.ts`.
- [ ] Find class/interface `BeforeToolHookOutput`.
- [ ] **Architecture Check:** Ensure you are modifying the existing structure
      without introducing new classes or complex inheritance. Keep the types
      clean.
- [ ] Remove the getter/setter or property definition for `permissionDecision`.
- [ ] Remove the getter/setter or property definition for
      `permissionDecisionReason`.

### 3. Verify Safety

- [ ] Search codebase for usages:
  ```bash
  grep -r "permissionDecision" packages/
  ```
- [ ] Fix any compile errors or broken references found by the search.

### 4. Test

- [ ] Run core tests:
  ```bash
  npm run test -- --project core
  ```
- [ ] **Test Architecture:** Ensure existing tests in `hooks-system.test.ts` (or
      similar) are updated if they relied on these aliases.
- [ ] Run full preflight check: `npm run preflight`.

### 5. Submit Pull Request

- [ ] Commit changes:
  ```bash
  git commit -m "refactor(core): remove deprecated permission aliases from BeforeToolHookOutput"
  ```
- [ ] Push and open PR linked to Issue #14720.
