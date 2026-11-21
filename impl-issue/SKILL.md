---
name: impl-issue
description: >-
  Implement the resolution of a GitHub issue in the project repo using the gh CLI, following the plan written in the issue.
  Use this skill whenever a request invokes it (e.g. via the /impl-issue slash
  command or an explicit mention) together with an issue number (optionally
  prefixed with #) to implement.
---

# Implement issue

Implement a planned GitHub issue according to the plan stored in the issue,
propose a commit message, and close the issue after committing.

## Procedure

1. Extract the issue number from the request text: the first integer found
   (strip a leading `#`). If no number is present, ask the user for the issue
   number.
2. Fetch the issue from the repo root (so `gh` targets the `origin` remote):

   ```sh
   gh issue view <number> --json state,title,body,labels
   ```

3. Check the state. If it is not `OPEN`, do not implement anything. Notify the
   user that the issue is closed and stop.
4. Check the labels. If the issue does not have the label `planned`, do not
   implement anything. Notify the user that the issue is not planned (they may
   want to run the plan-issue skill first) and stop.
5. Find the plan in the issue: it is the `## Plan` comment (or section) written
   by the plan-issue skill. If no plan exists, tell the user and stop.
6. Determine whether the plan is divided into several phases:
   - If it is not, implement the whole plan.
   - If it is, check which phases are already implemented. For a main issue
     with phases, the phases are tracked as sub-issues; list them with
     `gh issue view <number> --json subIssues` (fall back to searching for
     issues titled `Issue #<number>: Phase <n>:` if the sub-issue list is
     unavailable). The next phase to implement is the first sub-issue that is
     still open. If all sub-issues are closed, tell the user all phases are
     already implemented and stop.
7. Implement the phase (or the whole plan) exactly as written in the plan:
   - Investigate the current state of the codebase before each step; the plan
     may reference file paths and symbols that have since changed.
   - If the plan's first step is a regression test, write it first and verify
     it fails before the fix.
   - Follow the project's AGENTS.md conventions while implementing.
   - Validate the work: build and run the relevant tests with
     `cargo nextest run` (see AGENTS.md); run `cargo test --doc` if doctests
     are affected.
8. When done, do **not** commit. Instead:
   - Show the user a summary of the changes (files touched, what was
     implemented).
   - Propose a commit message following the project's commit message style.
   - Ask the user for approval of the commit message. If the user asks for
     changes, revise the commit message and ask for confirmation again. Do not
     commit anything before explicit approval.
9. After the user approves, commit the changes yourself (stage only the files
   belonging to this implementation; do not stage unrelated changes). Do not
   push unless the user explicitly asks.
10. Close the corresponding issue:
    - If the implementation was a phase sub-issue, close that sub-issue
      (`gh issue close <sub-issue-number>`), referencing the commit in the
      close comment.
    - Otherwise close the main issue (`gh issue close <number>`).
11. If the closed issue was the last open sub-issue of a main issue with
    phases, also close the main issue.

## Reporting

- End with a short summary: what was implemented, the commit hash and message,
  and which issue(s) were closed.
- If `gh` fails because it is not authenticated, tell the user to run
  `gh auth login`; do not retry blindly.
