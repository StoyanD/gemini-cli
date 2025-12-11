# Plan: Fix Command Injection in `run_shell_command`

## Objective

Fix the command injection vulnerability in `run_shell_command` where shell
command substitution syntax (e.g., `$(...)`, `` `...` ``) is executed instead of
being treated as a literal string.

## Context

- **Issue**: #14926
- **Vulnerability**: `run_shell_command` executes shell substitutions found in
  the command string.
- **Goal**: Ensure such syntax is treated literally or sanitized to prevent
  arbitrary code execution.

## Step 1: Reproduction

Create a test case to confirm the vulnerability.

1.  Create a new test add to in `integration-tests/run_shell_command.test.ts` to
    check if injection is blocked.
2.  **Test Case**:
    - Invoke `run_shell_command` with a payload containing command substitution:
      `echo $(touch injection_marker)`.
    - **Assertion**: Check if the file `injection_marker` is created.
    - **Expected Result (Current)**: File is created (Vulnerable).
    - **Expected Result (Fixed)**: File is NOT created; output is literally
      `echo $(touch injection_marker)` or similar.

## Step 2: Implementation

Modify the `run_shell_command` implementation to prevent shell substitution.

1.  **Locate Code**: Identify the core implementation of `run_shell_command`.
    Likely in `packages/core` or `packages/cli`.
2.  **Strategy**:
    - **Option A (Sanitization)**: Escape shell metacharacters in the input
      string before passing it to the underlying shell execution (e.g.,
      `bash -c`).
    - **Option B (Execution Mode)**: If possible, execute the command without
      invoking a shell for interpretation, or use a safer execution method that
      treats arguments as literals. _Note: This might break valid shell usage
      like pipes `|`, so careful consideration is needed._
    - Given the requirement to "treat as literal string", we likely need to
      escape the user's input when it's being wrapped in a shell execution.
3.  **Refinement**: Ensure common valid commands (e.g., `ls -la`,
    `git commit -m "msg"`) still work, but dangerous substitutions are
    neutralized.

## Step 3: Verification

1.  Run the reproduction test to ensure `injection_marker` is no longer created.
2.  Run existing tests (`npm run test`, `npm run test:e2e`) to ensure no
    regression in normal `run_shell_command` functionality.
3.  Verify that legitimate commands still function as expected.

## Step 4: Documentation

Update documentation if the behavior of `run_shell_command` changes
significantly (e.g., if specific syntax is now explicitly forbidden or escaped).

## Step 5: Submission Preparation

Ensure the contribution meets all project guidelines before submitting a Pull
Request.

1.  **Preflight Check**: Run the full suite of checks to validate the changes.

    ```bash
    npm run preflight
    ```

    This command runs builds, tests, type checks, and linting. All checks must
    pass.

2.  **Commit Messages**: Follow the
    [Conventional Commits](https://www.conventionalcommits.org/) standard.
    - Format: `fix(core): Prevent command injection in run_shell_command`
    - Provide a clear description of the "why" and link to the issue
      (`Fixes #14926`).

3.  **CLA**: Ensure the Contributor License Agreement is signed.
