# Finish Audit/Config Session

This step is **user-triggered**. Execute it when the user says "I'm done", "finish", "finish audit", "wrap up", "clean up for PR", "remove the report", or similar.

## Procedure

1. **Identify files to remove** based on reporting mode:
   - **Developer mode**: Find report files matching the skill-specific pattern (e.g., `*-a11y-config.md`, `*-html-validate.md`, `*-a11y-audit.md`) including any `_NN` suffix variants
   - **Auditor mode**: Find Jira artifact files matching the skill-specific pattern (e.g., `*-a11y-config-jira-chunks.md`, `*-a11y-config-jira-bulk.csv`, `*-a11y-config-jira-issue-*.md`)
   - Ask the user if they want to remove the false-positive file(s).

2. **If no matching files are found**: Tell the user the working directory is already clean and nothing needs to be removed.

3. **If files are found**: List each file by name and ask for explicit confirmation before deleting. Use this exact phrasing:
   > "Found N generated accessibility file(s) to remove:
   > - [file1]
   > - [file2]
   > ...
   > Deleting these will prepare a clean working directory for your PR. Your false positive registry (`.X-fps.json`), if present, will be kept — it is a project configuration file that should be committed with your code changes. Confirm deletion? (yes / no)"

4. **On yes**: Delete each listed file. Confirm each deletion by name. After all files are removed, remind the user that the false positive registry is safe to commit alongside the code changes.

5. **On no**: Make no changes. Confirm explicitly that no files were deleted and the working directory is unchanged.

## Branching Rules

- **Finish — No generated files found**: Inform the user that the working directory is already clean; no action needed.
- **Finish — User declines confirmation**: No files are deleted. Confirm explicitly that nothing changed and the generated files remain in place.
