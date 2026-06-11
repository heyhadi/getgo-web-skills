---
name: unit-test-last-commit
description: Inspects the last git commit, identifies changed source files, and writes or updates unit test files in src/test/ to cover the new behaviour.
---

# Goal

Analyse the latest `HEAD` commit and produce unit test files for every changed source file.

# Execution Steps

1. **Inspect commit**: Run `git log -1 --stat` and `git diff HEAD~1 HEAD` to see which files changed and exactly what changed.
2. **Map changed files to test files**: For each changed source file derive the mirror test path:
    - `src/utils/foo.ts` → `src/test/utils/foo.test.ts`
    - `src/brand/flagConfig.ts` → `src/test/brand/flagConfig.test.ts`
    - `src/brand/index.ts` → `src/test/brand/brand.test.ts`
    - `src/views/Foo/Foo.svelte` → `src/test/views/Foo/Foo.test.ts`
    - `src/components/X/X.svelte`→ `src/test/components/X/X.test.ts`
      Skip files that are config/build only (rollup.config.js, tsconfig.json, .env.\*, etc.).
3. **Read existing test file** (if it exists) so you do not duplicate or overwrite passing tests — only append new `describe`/`it` blocks for the changed behaviour.
4. **Read the changed source file** in full so you understand the exported symbols and their contracts.
5. **Write tests** that cover the new or modified behaviour introduced in the commit:
    - Use `vitest` + `@testing-library/svelte` (for Svelte components).
    - Follow the project conventions in `CLAUDE.md`:
        - Mock `src/brand` in any test for a component that imports `brand`; the mock must include every `brand` field the component uses.
        - Platform option shape is `{ id: string, label: string }`.
        - Never mock internal helpers — only system boundaries (API clients, stores).
    - Keep tests focused: one `describe` per exported symbol or component, one `it` per observable behaviour.
    - Do not add comments explaining what the test does — the `it` description is enough.
6. **Write/update the file**: If the test file already exists, append only the new `describe` blocks; otherwise create the file from scratch with the correct imports.

# Output

-   Write each test file directly to `src/test/…` (no intermediate files, no markdown reports).
-   After writing, run `npx vitest run <path>` for each file and confirm it passes. If a test fails, fix it before finishing.
-   After all tests pass, delete the `coverage/` directory at the project root if it exists: `rm -rf coverage`.

# Final Response

List each file written/updated and the number of new test cases added. Keep the message to 3–4 lines.
