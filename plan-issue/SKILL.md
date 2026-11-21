---
name: plan-issue
description: >-
  Plan the resolution of a GitHub issue in the project repo using the gh CLI. Use this skill whenever a request invokes it (e.g. via the /plan-issue slash command or an explicit mention) together with an issue number (optionally prefixed with #) to plan.
---

# Plan issue

Derive an implementation plan for a GitHub issue, get the user's approval,
write the plan back to the issue, and label it `planned`.

## Procedure

1. Extract the issue number from the request text: the first integer found
   (strip a leading `#`). If no number is present, ask the user for the issue
   number.
2. Fetch the issue from the repo root (so `gh` targets the `origin` remote:

   ```sh
   gh issue view <number> --json state,title,body,labels
   ```

3. Check the state. If it is not `OPEN`, do not plan anything. Notify the user
   that the issue is closed and stop.
4. Check the labels. If the issue already has the label `planned`, do not plan
   anything. Notify the user that the issue is already planned and stop.
5. Read the issue body carefully and determine whether it describes a defect or
   a feature/enhancement. Investigate the codebase as needed to ground the plan
   in the actual code (find the affected files, functions, and tests).
6. Draft a plan with the user. Present it in this structure:
   - A brief summary of the proposed solution.
   - An ordered list of the steps of the solution. Reference concrete file
     paths and symbols in the codebase.
   - If there are many and/or complex steps, arrange the steps into phases,
     where each phase can be compiled and committed separately.
   - If the issue is a defect, the first step of the plan must be to implement
     a regression test that reproduces the defect (it must fail before the fix
     and pass after it).
7. Ask the user for approval of the plan. If the user asks for changes, revise
   the plan accordingly and ask for confirmation again. Do not write anything
   to GitHub before explicit approval.
8. Once the user confirms the plan, write it into the issue. Append it as a
   comment (do not replace the original body):

   ```sh
   gh issue comment <number> --body-file - <<'EOF'
   ## Plan

   <plan text>
   EOF
   ```

9. Mark the issue with the label `planned`. Labels must exist before they can
   be applied; check with `gh label list --search planned` and create a missing
   one with `gh label create planned --color 0e8a16`. Then:

   ```sh
   gh issue edit <number> --add-label planned
   ```

## Sub-issues per phase

If the plan is divided into several phases (step 6), create one sub-issue per
phase after step 9:

1. For each phase, create an issue whose title is the phase title, prefixed
   with `Issue #<number>:` (the parent issue number) and `Phase <n>:`, e.g.
   `Issue #2: Phase 1: Add regression test`, so the title shows to which main
   issue the sub-issue belongs. Its body is that phase's steps, plus a link
   back to the parent issue (`Part of #<number>`).
2. Link each new issue as a sub-issue of the parent using the GraphQL
   `addSubIssue` mutation:

   ```sh
   gh api graphql -f query='
     mutation($parent: ID!, $child: ID!) {
       addSubIssue(input: { issueId: $parent, subIssueId: $child }) {
         issue { url }
       }
     }' -f parent=<parent-node-id> -f child=<child-node-id>
   ```

   Obtain the node IDs with
   `gh issue view <number> --json id` (or from the output of `gh issue create`,
   which returns the new issue's URL; convert it to an ID via
   `gh api graphql` if needed).

3. If sub-issue linking fails (e.g. the feature is unavailable), the `Part of
#<number>` line in the body is the fallback linkage; report this to the
   user.

## Reporting

- End with a short summary: the issue number and URL, whether the plan was
  written as a comment, the `planned` label applied, and the URLs of any
  sub-issues created.
- If `gh` fails because it is not authenticated, tell the user to run
  `gh auth login`; do not retry blindly.
