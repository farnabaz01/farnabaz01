## Contribution

### 2. Fork & Clone

1. Determine whether the current working directory is already a clone of the target repository:

   * Verify with `git rev-parse --is-inside-work-tree`.
   * Verify the repository matches the target project.

2. If the working directory is not a valid clone of the target repository:

   * Fork the repository:

     ```bash
     gh repo fork <owner>/<repo> --clone=false
     ```

   * Clone the fork into the working directory.

3. If cloning or fetching over SSH fails:

   ```bash
   git config --global url."https://github.com/".insteadOf git@github.com:
   ```

   Retry the operation.

4. Verify git remotes:

   * `origin` → fork repository
   * `upstream` → original repository

5. Fetch all refs:

   ```bash
   git fetch origin --prune
   git fetch upstream --prune
   ```

6. Identify the upstream default branch and ensure it is fully up to date:

   ```bash
   git checkout <default-branch>
   git fetch upstream
   git reset --hard upstream/<default-branch>
   ```

7. Always create a **fresh dedicated fix branch** from the latest upstream default branch (usually main). Do not reuse existing local or remote feature branches.

   ```bash
   git checkout -b <fix-branch-name>
   ```

   Verify the new branch is based on the current tip of `upstream/<default-branch>` before making any changes.

---

### 3. Explore the Codebase

1. Read the security report completely before making changes.
2. Search source files only:

   * Prefer `src/`, `lib/`, and other authored code.
   * Do **not** analyze generated output (`dist/`, `build/`, `.next/`, etc.) unless explicitly required.
3. Locate all relevant code paths using search tools (`grep`, `rg`, etc.).
4. Read the surrounding implementation before editing.
5. Trace the complete data flow:

   * Input sources
   * Validation/sanitization layers
   * Business logic
   * Outputs/sinks
6. Identify every affected surface, not just the reported example.

---

### 4. Implement the Fix

1. Confirm the root cause before changing code.
2. Fix the underlying vulnerability, not only the observed symptom.
3. Review similar code paths for the same issue.
4. Minimize behavioral changes unrelated to the fix.
5. Preserve backward compatibility unless the security issue requires otherwise.
6. Ensure the implementation follows existing project conventions.

---

### 5. Testing

1. Review existing tests covering the affected area.
2. If the repository requires generated configuration:

   ```bash
   pnpm dev:prepare
   ```
3. Reproduce the issue with a failing test whenever possible.
4. Add regression tests demonstrating:

   * The vulnerability exists before the fix.
   * The vulnerability is prevented after the fix.
5. Add coverage for adjacent edge cases discovered during investigation.
6. Run the relevant test subset during development.
7. Before submission, run the full test suite:

   ```bash
   pnpm vitest run
   ```
8. Do not submit changes with failing tests unless failures are pre-existing and documented.

---

### 6. Commit & Push

1. Create separate commits for:

   * The code fix
   * Regression tests
2. Use clear commit messages describing intent.
3. Push using an authenticated HTTPS remote:

   ```bash
   git remote set-url origin https://x-access-token:$(gh auth token)@github.com/<fork>/<repo>.git
   git push origin <branch>
   ```
4. Restore the original remote URL after the push succeeds.
5. Verify the branch exists on the fork before creating a PR.

---

### 7. Create the Pull Request

1. Confirm the fork owner:

   ```bash
   gh repo list
   ```

2. Create the PR against the upstream repository:

   ```bash
   gh pr create \
     --repo <upstream-owner>/<repo> \
     --head <fork-owner>:<branch> \
     --base main
   ```

3. Include in the PR description:

   * Root cause summary
   * Description of the fix
   * Test coverage added
   * Any compatibility or migration considerations

4. If formal review requests are unavailable due to permissions, leave a PR comment instead.

5. Always leave the following comment on the PR:

   ```
   This PR is ready for review by @farnabaz.
   ```

6. Record the PR URL in the final task output.
