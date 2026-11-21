# Agent skills for handling GitHub issues

- **create-issue** — Turns the text of your request into a GitHub issue in the
  project repo using the `gh` CLI.
- **plan-issue** — Derives an implementation plan for a GitHub issue, gets your
  approval, writes the plan back to the issue, and labels it `planned`. If there are many and/or complex steps, arranges the steps into phases, where each phase can be compiled and committed separately, and creates one sub-issue per phase, linked to the parent issue.
- **impl-issue** — Implements a planned issue according to the plan stored in
  the issue, proposes a commit message, gets your approval, and closes the issue after committing. For phased plans, it works on the next open phase sub-issue and closes the main issue once all phases are done.
