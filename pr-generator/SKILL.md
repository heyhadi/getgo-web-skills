---
name: pr-generator
description: Generates two separate files: a PR description and a manual testing plan.
---

# Goal

Analyze the latest `HEAD` commit and generate two distinct artifacts in the workspace.

# Execution Steps

1. **Analyze**: Read the latest commit message and `git diff`.
2. **Classify**: Select the template from `@/PULL_REQUEST_TEMPLATE/` based on the commit prefix (feat, fix, chore).
3. **Run unit tests**: Execute `npm run test` and capture the output.
   - If **any tests fail**: stop immediately. Do NOT create `PR_DESCRIPTION.md` or the PR. Report the failing tests to the user and tell them to fix the failures before retrying.
   - If **all tests pass**: continue to the next step.
4. **Draft PR**: Fill the template using the commit message and code summary. Include the Jira link: `https://getgo.atlassian.net/browse/[TICKET-ID]`.

# Output (Crucial)

Do not just provide text in the chat. **Perform the following steps in order**:

1. **Create `PR_DESCRIPTION.md`** in the project root with the full filled-out PR template.
2. **Create the PR on GitHub** using `gh pr create` with the title from the commit message and the body from `PR_DESCRIPTION.md`. Push the current branch first if it has no upstream. Capture the PR URL from the command output.
3. **Open the PR in the browser** by running `gh pr view --web` so it opens automatically in the default browser.
4. **Delete the coverage folder** by running `rm -rf coverage` from the project root.
5. **Ask about preview build**: Ask the user: _"Does this PR need a preview build? If yes, I'll apply the `preview` label to trigger an Amplify preview deployment."_
   - If **yes**: run `gh pr edit --add-label "preview"` on the newly created PR. Confirm with: _"Preview label applied — Amplify will build and post the preview URL as a PR comment once the build finishes."_
   - If **no**: skip silently. The default cancel workflow will suppress the Amplify build.

# Final Response

After opening the PR, provide a one-line summary confirming the PR was created and include the PR URL.
If tests failed, output a clearly formatted block like this:

---

## ❌ Unit Tests Failed — PR Not Created

The following tests must be fixed before a PR can be raised:
| # | Test File | Test Name | Error |
|---|-----------|-----------|-------|
| 1 | `src/test/...` | `test name here` | brief error message |
Fix the failures above and re-run `/pr-generator`.

---
