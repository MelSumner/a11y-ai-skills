# Issue Format

When generating issue markdown chunks or files, format each issue in the following way:

1. **Issue Name**: this should have the "problem" word first.
  a. Example: `Malformed HTML in button component`
  b. Example: `Missing accessible name for input`
  c. Example: `Incorrect use of aria-label attribute`
2. **Description**: describe the error and list where the error is found.
3. **How to fix**: provide modern, conformant best practices.
  a. Always recommend an HDS component first.
  b. Subsequent recommendations should start with `alternatively, you could`
  c. Recommend HTML elements instead of div/span elements with role attributes added.
4. **References**: make a bulleted list of WCAG success criteria/criterion that is failing, and include links to the related "Understanding" per the list in `understand-links.md`.
5. **Labels**: add the list of labels per the `jira-labels.md` file instructions.
