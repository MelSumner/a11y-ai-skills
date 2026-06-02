# Report File Naming

## Developer Mode

- **First report file**: `<project-name>-<skill-suffix>.md` in the project root
- **Subsequent reports**: Append `_NN` before `.md`, starting at `01`, and use the next available number

Examples:

- `nomad-a11y-config.md` (first run)
- `nomad-a11y-config_01.md` (second run)
- `nomad-html-validate.md` (first run)
- `nomad-a11y-audit.md` (first run)

## Auditor Mode

Write Jira artifacts in the project root as:

- `<project-name>-<skill-suffix>-jira-chunks.md` (copy/paste issue chunks)
- `<project-name>-<skill-suffix>-jira-bulk.csv` (bulk import)
- `<project-name>-<skill-suffix>-jira-issue-NNN.md` (per-issue files, one file per grouped issue chunk)

Examples:

- `nomad-a11y-config-jira-chunks.md`
- `nomad-a11y-config-jira-bulk.csv`
- `nomad-a11y-config-jira-issue-001.md`
- `nomad-html-validate-jira-chunks.md`
- `nomad-a11y-audit-jira-chunks.md`

## Project Name Derivation

Derive `<project-name>` from the folder name of the selected `package.json` (or the repository name if at the root).

## Skill Suffixes

- `a11y-config` for configuration audits
- `html-validate` for HTML spec validation
- `a11y-audit` for WCAG accessibility audits
