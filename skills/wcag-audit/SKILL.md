---
name: wcag-audit
description: "Run automated WCAG 2.2 AA accessibility checks for Ember.js codebases using codebase paths as scope. Use for automated issue detection, non-conformance tracking, and remediation guidance in Ember apps and addons. Manual testing is still required — automated results do not constitute a formal compliance determination. Keywords: WCAG 2.2 AA, accessibility audit, Ember.js, Glimmer, ember-template-lint, ember-a11y-testing, .hbs, .gts, .gjs, automated checks, keyboard navigation, screen reader, color contrast, ARIA, focus management."
argument-hint: "Provide the Ember.js codebase paths, app or addon scope, platform context, and any constraints or exclusions."
user-invocable: true
---

# WCAG Accessibility Compliance for Ember.js

## What This Skill Produces

- A required starting intake question: **"Are you a developer or an auditor?"**
- A WCAG 2.2 AA full-audit plan scoped to codebase paths and modules.
- A findings register mapped to success criteria and conformance status.
- A prioritized remediation backlog with severity, owners, and target dates.
- A detailed evidence report for all issues found, plus an automated audit outcome (pass / conditional pass / fail) that requires manual testing to complete.
- A false positive registry (`.wcag-audit-fps.json`) that the user can update to mark known false positives; the skill creates or updates this file on request.
- **Developer mode:** a Markdown report that separates real findings from user-acknowledged false positives, and flags stale false positive entries.
- **Auditor mode:** grouped Jira-ready issue chunks plus file-based bulk-upload and per-issue artifacts for direct Jira filing.
- A clean working directory on request: when the user is done, the skill removes generated report/artifact files.

## When To Use

- You want to run automated WCAG 2.2 AA checks against an Ember.js codebase to surface issues early.
- You need a structured findings register to track and prioritize accessibility issues before manual testing.
- You need remediation guidance with criterion mappings to inform development work on an Ember app or addon.
- Manual testing is required after this skill — automated results cover what tooling can detect, but keyboard, screen reader, visual, and motion checks must still be performed by a human.
- Run the `a11y-config` skill first to ensure required packages and configuration are in place. Keep HTML spec validation in the separate HTML spec workflow.
- You need outputs prepared for either developer remediation workflows or auditor Jira filing workflows.

> **Tip:** Telling the skill this is an Ember codebase when you invoke it produces better results. For example: *"load the wcag-audit skill and run it on this Ember codebase."*

## Required Inputs

- Scope: codebase paths, modules, packages, or component directories to audit.
- Default exclusions: ignore `node_modules` and `dist` folders when they exist.
- Platform: Ember.js application or addon under review.
- Audit type: full coverage (fixed and enforced by this skill).
- Standard target: WCAG 2.2 AA (fixed and enforced by this skill).
- Assistive technology expectations: keyboard only, screen reader targets, zoom/responsive constraints.
- Evidence requirement: detailed evidence report for all issues found (fixed and enforced by this skill).
- Audit artifacts: Ember route map (`app/routes`), component inventory (`app/components`), templates (`.hbs`, `.gts`, `.gjs`), helpers (`app/helpers`), services (`app/services`), shared UI primitives, and code ownership map.
- Coverage dimensions: required state variants (initial, loading, error, empty, populated, open/expanded) and relevant personas/permission levels in scope.
- Package scripts context: project `package.json` scripts so template linting commands can be discovered and executed.
- Suppression inventory: disabled/overridden lint rules, globally disabled scanner rules, inline suppression directives, and skipped tests affecting coverage.
- Tooling availability: which required commands/tools are present plus version outputs.
- Constraints: timeline, known exclusions, and third-party dependencies in scope.
- Reporting mode: ask at start — `developer` or `auditor`.
- Auditor product context (auditor mode): product name for Jira labeling; ask if not already provided.

## Procedure

Before Step 0, always ask this exact question: **"Are you a developer or an auditor?"**

- If the answer is **developer**: keep the existing workflow and report shape.
- If the answer is **auditor**: keep the same audit logic, but switch final outputs to grouped Jira issue chunks, bulk-upload artifacts, and per-issue files (no findings table output). Use [shared issue format rules](../shared/issue-format.md) as the canonical issue structure.

0. Run Preflight Discovery

- Build an Ember-specific inventory before running checks: enumerate routes (`app/routes`), components (`app/components`), templates (`.hbs`, `.gts`, `.gjs`), helpers (`app/helpers`), and services (`app/services`).
- Enumerate and record all suppressions:
  - template-lint disabled rules and ignored paths,
  - scanner rule suppressions (for example axe rule overrides),
  - inline suppression directives from other tooling,
  - inline suppression directives and skipped tests.
- Treat suppressions as risk evidence, not pass evidence.
- Verify required tool/script availability and record exact versions and any missing tools.
- Produce a preflight baseline summary (scope map, suppression ledger, known uncovered areas) before proceeding.
- Check for `.wcag-audit-fps.json` in the project root. If present, load all entries and record the count. If absent, record "no false positive registry found."
- If missing tools/scripts are found in preflight, apply Step 8 (Degraded Mode) before continuing with remaining steps.

1. Define Audit Charter

- Confirm WCAG target and scope boundaries.
- Confirm Audit coverage scope: all Ember template and markup files (`.hbs`, `.html`, `.gts`, `.gjs`).
- Exclude generated and dependency directories from audit scope: `node_modules` and `dist` (if present).
- Identify high-impact journeys first, using the Ember route map as the basis: authentication, checkout, forms, navigation, media, and error states.
- Record explicit exclusions and why they are out of scope.

2. Build Compliance Test Matrix

- Map each page or component to likely WCAG criteria areas: perceivable, operable, understandable, robust.
- Track test methods per item: automated checks, keyboard checks, screen reader checks, visual checks.
- Mark legal or business critical flows as highest priority.
- Define pass/fail decision rule for each criterion tested.

3. Run Automated Baseline Checks

- Execute available a11y linters and page scanners.
- Do not consume HTML spec validation results in this skill; that workflow is handled separately.
- Always discover and run any template linting scripts defined in the package scripts section (for example `lint:templates`, `template-lint`, `lint:hbs`, or equivalent project-specific template lint script names).
- Check `package.json` for `ember-a11y-testing` and record whether it is present or missing.
- If no template lint script exists, explicitly record that none was found.
- Record any disabled scanner/lint rules discovered and include them in findings as coverage risk items.
- Record skipped tests that remove audit coverage from routes/components.
- Capture repeatable findings first: missing labels, invalid ARIA, low contrast, heading structure, landmark gaps.
- Run checks across relevant state variants (not only initial render): loading, error, empty, open/expanded, populated.
- Do not treat automated pass as compliance; use it as triage input.

4. Perform Manual Validation

- Keyboard-only navigation:
  - Verify logical tab order, visible focus, no keyboard traps, and operable interactive controls.
  - Verify focus transitions on route changes and when opening/closing overlays, sidebars, modals, and portals.
  - Verify global keyboard shortcuts do not conflict with focused input/editor/terminal contexts.
- Screen reader behavior:
  - Verify accessible names, roles, values, state changes, announcements, and semantic structure.
- Visual and zoom checks:
  - Validate contrast, text resize, reflow, spacing tolerance, and orientation behavior.
- Media and motion checks:
  - Confirm captions/transcripts where needed and reduced motion compatibility.
- Error handling and forms:
  - Ensure errors are identified, associated, announced, and recoverable with clear guidance.
- Complex widgets and dynamic surfaces:
  - Validate keyboard and AT behavior for data visualization (SVG/D3), editors (e.g. CodeMirror), terminal widgets (e.g. XTerm), and streaming/live-update surfaces.
  - Confirm status updates are announced where appropriate (for example live regions/log roles).
- Capture criterion-level verdicts per tested page or component.
- For each violation, record the exact failed WCAG 2.2 Success Criterion ID(s) and title(s).
- If a violation cannot be clearly mapped to a single criterion but the markup is invalid HTML, record it as an invalid-HTML finding and map all plausibly related criteria (for example, 1.3.1 and 4.1.2).

5. Classify Non-conformance And Risk

- Classify each issue by severity:
  - Critical (Blocker): prevents core task completion or creates critical exclusion.
  - Serious: substantial friction or likely WCAG failure in key journeys.
  - Major: meaningful issue with available workaround.
  - Minor: limited impact or edge case.
- Group fixes by root cause so one change resolves many instances.
- If issue source is third-party dependency, document workaround and vendor escalation path.
- Assign conformance state per finding: fail, partial, pass with notes.
- Add applicable WCAG Techniques and Failures references from the W3C WCAG 2.2 Techniques set for each finding when relevant.
- Add the corresponding WCAG Understanding document link(s) for each mapped Success Criterion.

After completing classification, cross-reference each finding against the false positive registry (if loaded in Step 0):
- A finding **matches** a registry entry when its `criterion` and `location` both equal the entry's stored values (case-insensitive, trimmed).
- Move matched findings out of the active findings list and into a separate **active false positives** list.
- Any registry entry that did not match any current finding is a **stale entry** — record it separately.
- Findings that did not match any registry entry remain in the active findings list as real non-conformances.

6. Handle False Positive Requests

This step is **user-triggered**. Execute it when the user says "mark row N as a false positive", "mark finding N as a false positive", or similar.

- Identify the finding from the current report context: use the row number to locate the criterion, location, issue description, and severity.
- Ask for a reason if the user did not provide one; a reason is required for every registry entry.
- Create or update `.wcag-audit-fps.json` in the project root:
  - If the file does not exist, create it with the schema shown in the `## False Positive Registry` section.
  - If the file already exists, append the new entry to the `entries` array.
  - Set `"added"` to today's date in `YYYY-MM-DD` format.
- Confirm back to the user: print the entry that was written and state that subsequent runs will place this finding in the False Positives section of the report.
- To **unmark** a false positive, remove its entry from the `entries` array in `.wcag-audit-fps.json`.

7. Re-test And Validate Corrective Actions

- Apply semantic-first fixes before ARIA overlays.
- Re-run automated checks and repeat targeted manual checks for modified areas.
- Validate no regressions in previously passing flows.
- Update finding status to open, mitigated, or closed with verification date.

8. Handle Degraded Mode (When Tools Are Missing)

- If required scripts/tools are unavailable, do not silently pass affected checks.
- For each skipped command/check, record:
  - what could not run,
  - why it could not run,
  - which criteria/surfaces are now partially covered or uncovered.
- Continue with remaining checks and explicitly track residual risk from degraded coverage.

9. Produce Audit Report

- Summarize audit charter, tested codebase paths, tools, and assistive technologies.
- Immediately below the report title and before metadata, include a warning that the report was generated with an AI skill, covers only what automated tooling can detect, and that all findings/results must be verified by a human reviewer. Manual testing — including keyboard, screen reader, visual, and motion checks — is required to complete accessibility validation.
- Format the report metadata section as a Markdown bulleted list (not paragraphs or tables), including at minimum: audit date, repository/project, audited file types, WCAG target, audit type, Audit coverage scope: all template/markup files (i.e., .hbs, .html, .gts, .gjs), exclusions, tools used (including template lint scripts executed or none found), `ember-a11y-testing` devDependency status (present or missing), assistive technologies, assumptions, commands/scripts executed with outcomes, tool versions, suppression inventory discovered, untestable areas with reason, and residual risk summary.
- Link each finding to failed Success Criterion ID(s) and title(s), location, reproduction steps, and conformance state. Only include findings not moved to the active false positives list.
- In **developer** mode: present findings in a table where the first column is a sequential row number (`1`, `2`, `3`, ...).
- In **auditor** mode: do not emit findings tables; instead, group findings by criterion/root-cause clusters into Jira issue chunks, format each chunk using `## Issue Format (Auditor Mode)`, and generate labels using `## Jira Label Generation (Auditor Mode)`.
- Include detailed evidence for every issue found, including code references, reproduction steps, expected behavior, actual behavior, and verification status.
- For each finding, include applicable Techniques and/or Failures references from https://www.w3.org/WAI/WCAG22/Techniques/ and note when none are clearly applicable.
- For each finding, include corresponding Understanding link(s) from https://www.w3.org/WAI/WCAG22/Understanding/ for every mapped criterion.
- For invalid-HTML findings with ambiguous impact, state that the issue produces invalid HTML and may relate to multiple criteria, then list each mapped criterion and Understanding link.
- Format each mapped criterion as a Markdown link on the criterion text itself, for example: `[4.1.3 Status Messages](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html)`.
- State the automated audit outcome: pass, conditional pass with accepted risk, or fail. Include the count of active false positives suppressed this run in the summary. Note explicitly that this outcome reflects automated checks only and does not constitute a formal compliance determination; manual testing is required to complete accessibility validation.
- In **developer** mode: write the first report as `<project-name>-a11y-audit.md` in the working directory. If that file already exists, append `_NN` before `.md`, starting at `01`, and use the next available number.
- In **auditor** mode: write Jira artifacts following the auditor naming rules in `## Report File Naming` and the label rules in `## Jira Label Generation (Auditor Mode)`.

10. Finish Audit Session

See [shared cleanup procedure](../shared/cleanup-procedure.md) for complete details.

**Skill-specific file patterns:**
- Developer mode: `*-a11y-audit.md`, `*-a11y-audit_NN.md`
- Auditor mode: `*-a11y-audit-jira-chunks.md`, `*-a11y-audit-jira-bulk.csv`, `*-a11y-audit-jira-issue-*.md`
- False positive registry to preserve: `.wcag-audit-fps.json`

## Branching Rules

- If required tools or scripts are unavailable at preflight: instruct the user to run the `a11y-config` skill first, then return here.
- If the scope spans multiple packages/apps:
  - Audit shared UI primitives first, then app-specific implementations.
- If third-party UI dependencies are in scope:
  - Separate first-party and third-party findings, and document vendor escalation for third-party blockers.
- If exclusions exist:
  - Mark each excluded path explicitly in the report and record resulting residual risk.
- If the UI is an SPA or highly dynamic frontend:
  - Audit each critical flow across multiple state variants (initial, loading, error, empty, populated, open/expanded).
- If behavior differs by role/permissions:
  - Audit each in-scope persona/permission profile separately and report coverage deltas.
- If reporting mode is **auditor** and product is not provided: ask which product is being audited before generating Jira labels. If the product name differs from the canonical list, infer the closest mapped label from `## Jira Label Generation (Auditor Mode)`.
- If `.wcag-audit-fps.json` does not exist: skip false positive matching entirely; the findings list is the complete set of real non-conformances.
- If a false positive registry entry does not match any finding in the current run: mark it as a stale entry in the report; do not treat the absence of a match as an error.
- If reporting mode is **auditor**: group findings into Jira issue bundles by criterion/root-cause cluster whenever possible and output Jira chunks, bulk-upload artifacts, and per-issue files instead of findings tables.
- If the user requests to mark a finding as a false positive: execute Step 6 (Handle False Positive Requests) regardless of which step the skill is currently executing.
- If the user says "I'm done", "finish audit", "wrap up", "clean up for PR", "remove the report", or similar: execute Step 10 (Finish Audit Session) immediately, regardless of which step the skill is currently executing.
- **Finish — No generated files found**: Inform the user that the working directory is already clean; no action needed.
- **Finish — User declines confirmation**: No files are deleted. Confirm explicitly that nothing changed and the generated files remain in place.

## Skill Split Guidance

- Keep this skill focused on automated accessibility checks and structured findings that inform (but do not replace) manual testing.
- Create a separate skill for PR accessibility reviews because the scope, speed, and outputs differ.
- Create a separate design-system skill only if you need component-library governance and documentation conformance as a primary workflow.

## Quality Gates

A scope is considered complete only when all are true:

- No open Critical (Blocker) accessibility issues.
- All Serious issues have fixes or approved risk acceptance.
- Keyboard and screen reader checks pass for critical journeys.
- Contrast and zoom/reflow checks pass for all audited scope.
- Findings register includes conformance state and current status for every issue.
- Every finding includes failed WCAG 2.2 Success Criterion mapping.
- Applicable Techniques/Failures references are included for every finding, or explicitly marked as none identified.
- Every mapped criterion includes a corresponding Understanding link.
- Understanding links are rendered as Markdown links on the criterion text (not plain URLs).
- In **developer** mode: findings table uses numbered rows as the first column.
- In **auditor** mode: findings are grouped into Jira issue chunks (no findings table), with one chunk per grouped issue bundle whenever possible.
- In **auditor** mode: every Jira issue chunk and bulk row uses the required label contract from `## Jira Label Generation (Auditor Mode)`.
- Report includes an AI-generated warning directly under the title and before metadata.
- Report metadata at the top is rendered as a Markdown bulleted list.
- Invalid-HTML ambiguous findings explicitly note multi-criterion linkage when applicable.
- Detailed evidence is present for every issue found.
- Suppression ledger is present and includes all disabled/overridden rules, inline suppressions, and skipped tests affecting coverage.
- Every critical route/component has state-variant coverage, or each untested variant is explicitly documented with reason.
- Every required persona/permission profile in scope is tested, or explicitly documented as untested with reason.
- Template linting scripts were executed when present, and outcomes are included in the report.
- The `ember-a11y-testing` devDependency presence was checked and the result is included in the report.
- Commands/tools expected by the procedure are each marked as executed-with-result or skipped-with-reason.
- Untestable/unresolved items list is present (or explicitly marked empty).
- Residual risk summary is present and aligned with uncovered/skipped checks.
- In **developer** mode: report filename follows `<project-name>-a11y-audit.md` with `_NN` suffixing for subsequent runs.
- In **auditor** mode: Jira artifacts include `<project-name>-a11y-audit-jira-chunks.md`, `<project-name>-a11y-audit-jira-bulk.csv`, and `<project-name>-a11y-audit-jira-issue-NNN.md` files.
- Report is updated with unresolved items, accepted exceptions, and next actions.
- False positive registry status (present with N entries, or absent) is recorded in report metadata.
- Automated audit outcome summary includes the count of active false positives suppressed this run (or zero if none).
- If the registry is present: active false positives table is present in the report (or explicitly marked empty).
- If the registry is present: stale false positive entries are listed (or explicitly marked none).

## Output Format

Return results in this order, based on reporting mode:

### Developer mode

1. AI-generated warning (below title, before metadata): this report was generated with an AI skill and covers only what automated tooling can detect. All findings and results must be verified by a human reviewer. Manual testing — including keyboard navigation, screen reader behavior, visual/zoom checks, and motion checks — is required to complete accessibility validation. This report does not constitute a formal compliance determination.
2. Report metadata (Markdown bulleted list): audit date, repository/project, audited file types, WCAG target, audit type, Audit coverage scope: all template/markup files (i.e., .hbs, .html, .gts, .gjs), exclusions, tools used (including template lint scripts executed or none found), `ember-a11y-testing` devDependency status (present or missing), assistive technologies, assumptions, commands/scripts executed with outcomes, tool versions, suppression inventory discovered, untestable areas with reason, and residual risk summary.
3. Findings table: row number, issue, impact, severity, failed WCAG 2.2 Success Criterion ID(s)/title(s), Understanding link(s), locations, conformance state, finding status (open/mitigated/closed), applicable Techniques/Failures references. Only real non-conformances appear here; false positives are excluded.
4. Remediation plan: quick wins, structural fixes, owners, and target dates.
5. Active False Positives table (when `.wcag-audit-fps.json` exists): row number, criterion, location, issue description, reason, date added. Findings suppressed via the registry appear here instead of in the findings table. Omit this section entirely if no findings matched any registry entry this run.
6. Stale False Positives (when `.wcag-audit-fps.json` exists): registry entries that did not match any finding in this run, with the stored criterion and location. Each entry should be reviewed — the issue may have been resolved or the location text may have changed. Omit this section entirely if there are no stale entries.
7. Detailed evidence report: per-issue reproduction, code references, expected vs actual behavior, verification notes, failed criterion mapping, corresponding Understanding link(s), applicable Techniques/Failures references, and invalid-HTML multi-criterion notes when applicable.
8. Re-test summary and automated audit outcome (pass / conditional pass / fail). Note that this outcome reflects automated checks only; manual testing is required before drawing any compliance conclusions.
9. Residual risks, accepted exceptions, and follow-up milestones.

### Auditor mode (Jira-oriented)

1. AI-generated warning and metadata (same required metadata fields as developer mode).
2. Auditor summary: pass / conditional pass / fail with grouped counts by severity and criterion cluster.
3. **Grouped Jira issue chunks** (not tables): group issues by shared root cause and criterion overlap whenever possible. Format every chunk using `## Issue Format (Auditor Mode)` and generate labels using `## Jira Label Generation (Auditor Mode)`.
4. Active and stale false positive sections, expressed as Jira notes/chunks (not findings tables).
5. Jira artifact manifest describing files written for filing, and confirm that chunk, bulk-upload, and per-issue artifacts were generated.
6. Re-test and follow-up notes.

## Issue Format (Auditor Mode)

See [shared issue format rules](../shared/issue-format.md). In auditor mode, this is the canonical structure for every Jira issue chunk.

## Jira Label Generation (Auditor Mode)

See [shared Jira label generation rules](../shared/jira-labels.md).

**Note:** This skill uses "Major" severity which maps to `a11y-sev:moderate` label.

## Report File Naming

See [shared report naming conventions](../shared/report-naming.md). This skill uses:
- Developer mode: `<project-name>-a11y-audit.md` (with `_NN` suffixing for reruns)
- Auditor mode: `<project-name>-a11y-audit-jira-chunks.md`, `<project-name>-a11y-audit-jira-bulk.csv`, `<project-name>-a11y-audit-jira-issue-NNN.md`

## False Positive Registry

The false positive registry is stored as `.wcag-audit-fps.json` in the project root. See [shared false positive registry documentation](../shared/false-positive-registry.md) for complete details on file format, matching algorithm, and usage guidance.

**Skill-specific notes:**
- Registry file name: `.wcag-audit-fps.json`
- Created when user marks a finding as false positive (Step 6)
- Read during preflight (Step 0) on every subsequent run
- Matching requires: `criterion` and `location` to both match

**Example entry for this skill:**
```json
{
  "criterion": "1.4.3",
  "location": "app/components/nav.gts — disabled nav link text",
  "issue": "Contrast ratio 3.5:1 on disabled nav link",
  "reason": "Disabled controls are explicitly exempt from contrast requirements per WCAG 2.2 SC 1.4.3 Note 1.",
  "added": "2026-05-19"
}
```

**When to use registry vs accepted risk:**
- Use `.wcag-audit-fps.json` for known false positives that recur across runs (e.g., disabled-control contrast exemptions, third-party limitations, alternate conformance paths)
- Use "accepted exceptions" section for genuine conformance gaps that have been formally accepted with documented risk
