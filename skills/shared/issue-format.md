# Issue Format

When generating issue markdown chunks or files, format each issue in the following way:

1. **Issue Name**: this should have the "problem" word first.
  a. Example: `Malformed HTML in button component`
  b. Example: `Missing accessible name for input`
  c. Example: `Incorrect use of aria-label attribute`
2. **Description**:
  a. describe the error.
  b. provide a list of files (and line/cols) where the error is found.
3. **How to fix**:
  a. Recommend using an HDS component if it exists.
  b. Select the safest approved remediation pattern from the remediation library. If no approved pattern applies, do not invent a fix. Mark the finding as `needs_expert_review` and add it to the jira labels.
  c. Subsequent recommendations should start with `alternatively, you could`
  d. Recommend HTML elements instead of div/span elements with role attributes added.
  e. Use HTML for markup, CSS for styling. Do not provide any "how to fix" suggestions that conflict with this instruction.
4. **References**: make a bulleted list of WCAG success criteria/criterion that is failing, and include links to the related "Understanding" per the list in `understand-links.md`.
5. **Labels**: add the list of labels per the `jira-labels.md` file instructions.
