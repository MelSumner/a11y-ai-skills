---
name: html-validate
description: "Run HTML5 spec compliance validation on Ember templates (.gts, .gjs, .hbs) using html-validate-ember. Use for detecting content model violations, ARIA/accessibility rule failures, form correctness issues, and invalid attribute values — with diagnostics pointing at exact source positions. Keywords: html-validate, html-validate-ember, Ember templates, HTML spec, content model, ARIA, form validation, attribute validity, GTS, GJS, HBS, HTML linting, Glimmer, validate-gts."
argument-hint: "Provide the Ember project root path, specific directories or files to validate (e.g. app/components, app/templates), and any known exclusions."
user-invocable: true
---

# HTML Spec Compliance for Ember Templates

## What This Skill Produces

- A required starting intake question: **"Are you a developer or an auditor?"**
- A categorized findings register mapping every violation to its rule ID, file path, and line/column.
- A suppression ledger of all `{{!-- [html-validate-disable ...] --}}` directives found in templates.
- A false positive registry (`.html-validate-fps.json`) that the user can update to mark known false positives; the skill creates or updates this file on request.
- A prioritized remediation guide grouped by rule family.
- **Developer mode:** a Markdown report file written to the project root that separates real violations from user-acknowledged false positives, flags stale false positive entries, and includes Markdown links to the relevant specification or WCAG success criterion for each finding.
- **Auditor mode:** grouped Jira-ready issue chunks plus file-based bulk-upload and per-issue artifacts so findings can be filed directly in Jira without converting from tables.
- A clean working directory on request: when the user is done, the skill removes generated report/artifact files.

## When To Use

- You want to check whether an Ember project's rendered HTML is spec-correct and accessible.
- You need to detect content model violations, invalid ARIA, form rule failures, or illegal attribute values in `.gts`, `.gjs`, or `.hbs` template files.
- You are auditing a project for HTML5 spec compliance as a complement to `ember-template-lint` (which checks Ember idioms) or `wcag-audit` (which covers WCAG manual checks).
- Run the `a11y-config` skill first to ensure required packages and configuration files are in place before using this skill.
- You need outputs prepared for either developer remediation flow or auditor Jira filing flow.

## Required Inputs

- Project root path: the Ember app or addon directory containing `package.json`.
- Scope: directories or files to validate. Default is `app/`, `tests/`, and `addon/` when present.
- Default exclusions: always skip `node_modules/` and `dist/`.
- Package manager: `pnpm`, `npm`, or `yarn` — detected automatically; ask the user if detection is inconclusive.
- Glint mode: whether to enable TypeScript-aware component resolution (default: on; use `--no-glint` as fallback if the project lacks `@glint/ember-tsc`).
- Reporting mode: ask at start — `developer` or `auditor`.
- Auditor product context (auditor mode): product name for Jira labeling; ask if not already provided.

## Procedure

Before Step 0, always ask this exact question: **"Are you a developer or an auditor?"**

- If the answer is **developer**: keep the existing workflow and report shape.
- If the answer is **auditor**: keep the same detection logic, but switch final outputs to grouped Jira issue chunks, bulk-upload artifacts, and per-issue files (no findings table output).

0. Run Preflight Discovery

- Locate `package.json` in the project root. If multiple `package.json` files are found, ask the user which one to use.
- Detect the package manager by checking for `pnpm-lock.yaml`, `yarn.lock`, and `package-lock.json`. If detection is inconclusive, ask the user.
- Check `devDependencies` for `html-validate-ember` and `html-validate`. Record each as present or missing.
- Check whether `node_modules/.bin/validate-gts` exists (confirms installed and runnable).
- If `html-validate-ember`, `html-validate`, or `validate-gts` are missing: stop and tell the user to run the `a11y-config` skill first to install and configure the required packages, then return here.
- Check for `.htmlvalidate.json` in the project root. Record as present or missing.
- If `.htmlvalidate.json` is missing: stop and tell the user to run the `a11y-config` skill first to generate the required config, then return here.
- If `.htmlvalidate.json` is present, read and record its contents (extends, plugins, transform, rules).
- Enumerate all inline suppress directives in template files:
  - Long form: `{{!-- [html-validate-disable ...] --}}`
  - Short form: `{{! [html-validate-disable ...] }}`
  - Record every suppressed rule and the file/line where it appears.
- Check for `.html-validate-fps.json` in the project root. If present, load all entries and record the count. If absent, record "no false positive registry found."
- Produce a preflight baseline summary (package manager, config status, suppression ledger, FP registry status) before proceeding.

1. Define Validation Scope

- Confirm which directories or files to validate: accept user-provided paths or use the default (`app/`, `tests/`, `addon/`).
- Always exclude `node_modules/` and `dist/` from scope.
- Confirm file types in scope: `.gts`, `.gjs`, `.hbs`.
- Record the final validated scope before execution.

2. Run the Validator

- Execute the validator against the confirmed scope:
  - Preferred: `pnpm exec validate-gts <scope-paths>` (or `npx validate-gts` / `yarn exec validate-gts`)
  - If Glint is not available (`@glint/ember-tsc` missing from `devDependencies`): add `--no-glint` flag
  - If `validate-gts` is not available as a binary, run programmatically:
    ```
    node -e "require('./node_modules/html-validate-ember/dist/run.js')" -- <scope-paths>
    ```
- Capture full stdout (diagnostic lines) and stderr, plus exit code.
- If the validator exits with a parse error or plugin load error (not a lint violation), record it as a tool error in the preflight baseline and apply Step 6 (Degraded Mode).
- Do not silently discard non-zero exit codes; treat exit code 1 as "violations found" and anything higher as a tool error.

3. Categorize Findings

Group every violation into one of four rule families. Use the rule ID from the output to classify:

**Content Model** — HTML5 nesting and parse-time structure rules:
- `element-permitted-content`, `element-permitted-parent`, `element-permitted-order`
- `no-implicit-close`, `close-order`, `no-self-closing` (when not in transformer-artifact context)
- `element-required-content`

**ARIA / Accessibility** — ARIA spec and landmark rules:
- All `wcag/*` rules (e.g. `wcag/h32`, `wcag/h37`, `wcag/h67`, `wcag/h71`)
- `aria-label-misuse`, `aria-hidden-body`, `aria-prohibited-attr`
- `no-dup-id`, `unique-landmark-label`
- `input-missing-label`, `element-required-attributes` (when on interactive elements)

**Form Correctness** — Form element and input rules:
- `wcag/h32` (form must have submit), `wcag/h71` (fieldset must have legend)
- `form-dup-name`, `input-attributes`
- `no-implicit-input-type`

**Attribute Validity** — Attribute value and presence rules:
- `attribute-allowed-values`, `attribute-misuse`
- `element-required-attributes` (when on non-interactive elements)
- `no-raw-characters`

For each finding record: rule ID, rule family, file path, line number, column number, severity (error/warning), the diagnostic message, and the applicable spec reference formatted as a Markdown link:
- ARIA/Accessibility findings: the relevant WCAG 2.2 Success Criterion as `[SC X.X.X Title](https://www.w3.org/WAI/WCAG22/Understanding/slug.html)`, or the relevant ARIA spec section as a Markdown link, or explicitly note "none identified."
- Content Model findings: the relevant HTML Living Standard section as `[HTML Living Standard §X.X — section title](https://html.spec.whatwg.org/multipage/page.html#anchor)`, or explicitly note "none identified."
- Form Correctness and Attribute Validity findings: the relevant WCAG 2.2 criterion or HTML spec section as a Markdown link when applicable, or explicitly note "none identified."

In **auditor** mode, additionally classify each finding by accessibility severity for `## Jira Label Generation (Auditor Mode)` label assignment:
- Critical: prevents core task completion or creates critical exclusion (for example: missing accessible name on an interactive element in a primary user journey).
- Serious: substantial friction or likely WCAG failure in key journeys.
- Moderate: meaningful issue with an available workaround.
- Minor: limited impact or an edge case.
Validator-reported severity (`error`/`warning`) is structural; accessibility severity must be assessed independently based on user impact.

After categorizing all findings, cross-reference each violation against the false positive registry (if loaded in Step 0):
- A violation **matches** a registry entry when its `rule`, `file`, and `line` all equal the entry's stored values.
- Move matched violations out of the findings list and into a separate **active false positives** list.
- Any registry entry that did not match any current violation is a **stale entry** — record it separately.
- Violations that did not match any registry entry remain in the findings list as real violations.

4. Handle False Positive Requests

This step is **user-triggered**. Execute it when the user says "mark row N as a false positive", "mark violation N as a false positive", or similar.

- Identify the violation from the current report context: use the row number to locate the rule, file, line, message, and rule family.
- Ask for a reason if the user did not provide one; a reason is required for every registry entry.
- Create or update `.html-validate-fps.json` in the project root:
  - If the file does not exist, create it with the schema shown in the `## False Positive Registry` section.
  - If the file already exists, append the new entry to the `entries` array.
  - Set `"added"` to today's date in `YYYY-MM-DD` format.
- Confirm back to the user: print the entry that was written and state that subsequent runs will place this violation in the False Positives section.
- To **unmark** a false positive, remove its entry from the `entries` array in `.html-validate-fps.json`.

5. Produce Compliance Report

- Immediately below the report title and before all other content, include a warning that this report was generated with an AI skill and all findings must be verified by a human reviewer.
- Format the report metadata as a Markdown bulleted list including at minimum: validation date, project/repository, scope (directories and file types validated), exclusions, `html-validate-ember` version, `html-validate` version, Glint mode (enabled or disabled), package manager, command(s) executed, exit code(s), suppression inventory, and any tool errors encountered.
- In **developer** mode: present findings in a table where the first column is a sequential row number.
- In **auditor** mode: do not emit findings tables; instead, group findings by rule family and shared fix pattern into Jira issue chunks and generate labels using `## Jira Label Generation (Auditor Mode)`.
- Group findings by rule family (Content Model, ARIA/Accessibility, Form Correctness, Attribute Validity). Only include violations that were not moved to the active false positives list.
- For every ARIA/accessibility finding, include the relevant WCAG 2.2 Success Criterion or ARIA spec reference formatted as a Markdown link:
  - Format each WCAG criterion as a Markdown link on the criterion text itself, for example: `[SC 1.3.1 Info and Relationships](https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships.html)`.
  - Include applicable Techniques and/or Failures references from https://www.w3.org/WAI/WCAG22/Techniques/ as Markdown links, for example: `[H44](https://www.w3.org/WAI/WCAG22/Techniques/H44)`. Note when none are clearly applicable.
  - For ARIA spec references (e.g. role definitions), link to the relevant section of the ARIA spec at https://www.w3.org/TR/wai-aria-1.2/.
  - If no WCAG criterion or ARIA spec section is clearly applicable, state "none identified."
- For every content model finding, include the relevant HTML Living Standard section formatted as a Markdown link on the section text itself, for example: `[HTML Living Standard §4.3.2 — The article element](https://html.spec.whatwg.org/multipage/sections.html#the-article-element)`. If no specific section is clearly applicable, state "none identified."
- For Form Correctness and Attribute Validity findings, include relevant WCAG criterion or HTML spec references as Markdown links where applicable, following the same format as above.
- State a compliance summary: pass (no violations), violations found (list counts per family and FP count), or tool error (explain what could not run).
- In **developer** mode: write the report as `<project-name>-html-validate.md` in the project root. If that file already exists, append `_NN` before `.md` starting at `01` and use the next available number.
- In **auditor** mode: write Jira artifacts following the auditor naming rules in `## Report File Naming` and the label rules in `## Jira Label Generation (Auditor Mode)`.

6. Handle Degraded Mode (When Validator Cannot Run)

- If `validate-gts` cannot be executed for any reason, do not silently pass any checks.
- For each issue that prevented execution, record:
  - what could not run and why,
  - which files/paths are now unvalidated,
  - what manual steps the user can take to resolve and re-run.
- Continue with static analysis (inspect `.htmlvalidate.json` config, enumerate suppressions) and report degraded coverage explicitly.

7. Finish Audit Session

See [shared cleanup procedure](../shared/cleanup-procedure.md) for complete details.

**Skill-specific file patterns:**
- Developer mode: `*-html-validate.md`, `*-html-validate_NN.md`
- Auditor mode: `*-html-validate-jira-chunks.md`, `*-html-validate-jira-bulk.csv`, `*-html-validate-jira-issue-*.md`
- False positive registry to preserve: `.html-validate-fps.json`

## Branching Rules

- If `html-validate-ember` is not installed or `.htmlvalidate.json` is missing: stop and instruct the user to run the `a11y-config` skill first, then return to this skill.
- If `@glint/ember-tsc` is absent from `devDependencies`: run with `--no-glint`; note in metadata that Glint-powered component-to-element resolution was not available.
- If the project contains multiple `package.json` files (monorepo): ask the user which package to validate; scope the validator to that package's root only.
- If reporting mode is **auditor** and product is not provided: ask which product is being audited before generating Jira labels. If the product name differs from the canonical list, infer the closest mapped label from `## Jira Label Generation (Auditor Mode)`.
- If scope paths produce zero `.gts`/`.gjs`/`.hbs` files: report that no templates were found in the given scope and suggest alternative paths.
- If the validator reports only `no-trailing-whitespace`, `no-self-closing`, `attr-quotes`, or `no-raw-characters` violations: flag these as likely transformer artifact false positives and recommend confirming that the `:recommended` or `:gts-recommended` preset is active in `.htmlvalidate.json`.
- If `.html-validate-fps.json` does not exist: skip false positive matching entirely; the findings list is the complete set of real violations.
- If a false positive registry entry does not match any violation in the current run: mark it as a stale entry in the report; do not treat the absence of a match as an error.
- If reporting mode is **auditor**: group issues by rule family/root-cause clusters whenever possible and output Jira chunks, bulk-upload artifacts, and per-issue files instead of findings tables.
- If the user requests to mark a violation as a false positive: execute Step 4 (Handle False Positive Requests) regardless of which step the skill is currently executing.
- If the user says "I'm done", "finish audit", "wrap up", "clean up for PR", "remove the report", or similar: execute Step 7 (Finish Audit Session) immediately, regardless of which step the skill is currently executing.
- **Finish — No generated files found**: Inform the user that the working directory is already clean; no action needed.
- **Finish — User declines confirmation**: No files are deleted. Confirm explicitly that nothing changed and the generated files remain in place.

## Quality Gates

A run is considered complete only when all are true:

- Preflight baseline is present (package manager, config status, suppression ledger, FP registry status).
- Validation scope and exclusions are explicitly recorded.
- Validator was either run successfully or failure is documented with reason and residual risk.
- Every finding has: rule ID, rule family, file path, line number, column, severity, and message.
- Findings are grouped by rule family (Content Model, ARIA/Accessibility, Form Correctness, Attribute Validity).
- ARIA/accessibility findings include WCAG 2.2 or ARIA spec references formatted as Markdown links, or explicitly state "none identified."
- WCAG criterion references are rendered as Markdown links on the criterion text (not plain text or bare URLs).
- Techniques and Failures references are rendered as Markdown links, or explicitly marked as none identified.
- Content model findings include HTML Living Standard section references formatted as Markdown links on the section text, or explicitly state "none identified."
- Suppression ledger is present listing all `{{!-- [html-validate-disable ...] --}}` directives found, or explicitly marked empty.
- In **developer** mode: findings table uses sequential numbered rows as the first column.
- In **auditor** mode: findings are grouped into Jira issue chunks (no findings table), with one chunk per grouped issue bundle whenever possible.
- In **auditor** mode: every Jira issue chunk includes an impact statement.
- In **auditor** mode: every Jira issue chunk and bulk row uses the required label contract from `## Jira Label Generation (Auditor Mode)`.
- Report includes AI-generated warning directly below the title and before metadata.
- Report metadata is rendered as a Markdown bulleted list.
- Glint mode used (enabled or disabled) is recorded in metadata.
- Compliance summary (pass / violations found / tool error) is stated.
- Compliance summary includes the count of active false positives suppressed this run (or zero if none).
- False positive registry status (present with N entries, or absent) is recorded in report metadata.
- If the registry is present: active false positives table is present in the report (or explicitly marked empty).
- If the registry is present: stale false positive entries are listed (or explicitly marked none).
- Commands executed and their exit codes are recorded in metadata.
- In **developer** mode: report filename follows `<project-name>-html-validate.md` naming convention with `_NN` suffixing for reruns.
- In **auditor** mode: Jira artifacts include `<project-name>-html-validate-jira-chunks.md`, `<project-name>-html-validate-jira-bulk.csv`, and `<project-name>-html-validate-jira-issue-NNN.md` files.

## Output Format

Return results in this order, based on reporting mode:

### Developer mode

1. AI-generated warning (below title, before metadata): this report was generated with an AI skill and all findings must be verified by a human reviewer.
2. Report metadata (Markdown bulleted list): validation date, project/repository, scope, exclusions, `html-validate-ember` version, `html-validate` version, Glint mode, package manager, commands executed with exit codes, tool versions, suppression inventory, unvalidated areas with reason, and residual risk summary.
3. Compliance summary: pass (zero violations), violations found (counts per rule family), or tool error.
4. Findings tables (one per rule family): row number, rule ID, file path, line:column, severity, message, spec reference (Markdown link to the relevant WCAG Understanding page, HTML Living Standard section, or ARIA spec section; "none identified" when not applicable). Only real violations appear here; false positives are excluded.
5. Active False Positives table (when `.html-validate-fps.json` exists): row number, rule ID, file path, line:column, reason, date added. Violations suppressed via the registry appear here instead of in the findings tables. Omit this section entirely if no violations matched any registry entry this run.
6. Stale False Positives (when `.html-validate-fps.json` exists): registry entries that did not match any violation in this run, with the stored file, line, and rule. Each entry should be reviewed — the violation may have been fixed or the file may have moved. Omit this section entirely if there are no stale entries.
7. Suppression ledger: file path, line, suppressed rule(s), for every inline directive found.
8. Remediation guide: quick fixes (single-line changes), structural fixes (template restructuring), and configuration-level mitigations (adding rule exceptions to `.htmlvalidate.json` when appropriate).
9. Re-run instructions: exact command to re-run the validator after fixes are applied.

### Auditor mode (Jira-oriented)

1. AI-generated warning and metadata (same required metadata fields as developer mode).
2. Auditor summary: pass / violations found / tool error with grouped counts by rule family.
3. **Grouped Jira issue chunks** (not tables): group violations by rule-family and shared fix pattern whenever possible. Each chunk must be copy/paste ready and include: issue type, summary/title, severity/priority, labels, components, affected files/locations, impact statement, evidence, WCAG/HTML/ARIA references, WCAG Failure references (when known), and acceptance criteria. Jira summary/title must start with what is wrong (for example: "Incorrect use of ...", "Missing accessible name for ...", "Lack of ...", "Malformed syntax ..."). Labels must be generated using `## Jira Label Generation (Auditor Mode)`.
4. Active and stale false positive sections, expressed as Jira notes/chunks (not findings tables).
5. Jira artifact manifest describing files written for filing, and confirm that chunk, bulk-upload, and per-issue artifacts were generated.
6. Re-run instructions and follow-up notes.

## Jira Label Generation (Auditor Mode)

See [shared Jira label generation rules](../shared/jira-labels.md).

## Report File Naming

See [shared report naming conventions](../shared/report-naming.md). This skill uses:
- Developer mode: `<project-name>-html-validate.md` (with `_NN` suffixing for reruns)
- Auditor mode: `<project-name>-html-validate-jira-chunks.md`, `<project-name>-html-validate-jira-bulk.csv`, `<project-name>-html-validate-jira-issue-NNN.md`

## False Positive Registry

The false positive registry is stored as `.html-validate-fps.json` in the project root. See [shared false positive registry documentation](../shared/false-positive-registry.md) for complete details on file format, matching algorithm, and usage guidance.

**Skill-specific notes:**
- Registry file name: `.html-validate-fps.json`
- Created when user marks a violation as false positive (Step 4)
- Read during preflight (Step 0) on every subsequent run
- Matching requires: `rule`, `file`, and `line` to all match

**Example entry for this skill:**
```json
{
  "rule": "no-implicit-close",
  "file": "app/components/pricing-table.gts",
  "line": 42,
  "message": "Element <p> is implicitly closed by parent </div>",
  "reason": "Glint resolves <PricingRow> to a block-level element; the outer <p> is intentionally ended before it at runtime.",
  "added": "2026-05-19"
}
```

**When to use registry vs inline suppressions:**
- Use `.html-validate-fps.json` when you cannot modify the template file, want a project-level audit trail, or the violation is tied to Glint resolution that is correct at runtime
- Use `{{!-- [html-validate-disable rule] --}}` inline directive when you can modify the template and want the suppression colocated with the code
