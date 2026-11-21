---
name: create-issue
description: >-
  Create a GitHub issue in the project repo from the text of the user's request, using the gh CLI. 
  Use this skill whenever a request invokes it (e.g. via the /create-issue slash command or an explicit mention) and contains text describing a bug, feature, or redesign to file — even if the request does not say the words "issue" or "GitHub".
---

# Create issue

Turn the text of the user's request into a GitHub issue in the project repo
using the `gh` CLI.

## Procedure

1. Take the request text that accompanies the skill reference (everything the
   user provided after invoking the skill). Strip the skill invocation itself
   (e.g. `/create-issue`) from the text; everything else is the issue input.
2. Check the first line of the text (after trimming whitespace). If it is
   `defect`, `enhancement`, or `redesign` (case-insensitive, whole line only),
   that line is the issue's flag and must be used as the GitHub label. Use the
   remaining text as the issue body. If the first line is not one of these
   three keywords, the entire text is the body and there is no label.
3. Check the text for conflicting or unclear requirements. If there are any,
   list them, propose a revision of the text and ask the user for approval. If the user asks for changes, revise the text accordingly and ask for approval again. Do not write anything to GitHub before explicit approval.
4. When the user finally approved the text, derive a short title from the body:
   capture the essence of the problem or request in imperative mood, max ~70 characters, no trailing punctuation.
   Do not invent facts that are not in the body.
5. If the body (after stripping the flag line) is empty, do not create an
   issue. Ask the user for the issue text instead.
6. Run `gh` from the repo root so it targets the `origin` remote. 
   Create the issue:

   ```sh
   gh issue create --title "<derived title>" --body-file - <<'EOF'
   <body text>
   EOF
   ```

   Use `--body-file -` with a heredoc for multi-line bodies to avoid quoting
   problems. Single-line bodies may use `--body "..."`.
7. When a flag was given, pass it as a label: add `--label defect` (or
   `enhancement` / `redesign`). Labels must exist before they can be applied;
   check with `gh label list --search <flag>` and create a missing one with
   `gh label create <flag> --color 0e8a16` (pick a distinct color per flag if
   the user has no preference).

## Reporting

- Return the URL of the created issue as the final message.
- If `gh` fails because it is not authenticated, tell the user to run
  `gh auth login`; do not retry blindly.
