---
name: a11y-config
description: "Audit accessibility package installation and configuration in Ember.js codebases without executing lint or test commands. Use for inventorying setup gaps and producing remediation guidance for WCAG 2.2 AA-oriented readiness."
argument-hint: "Provide Ember version, selected package.json path when needed, package manager, and scope for static configuration review."
user-invocable: true
---

# Accessibility Configuration

## What This Skill Produces

- A required starting intake question: **"Are you a developer or an auditor?"**
- A configuration-only report of package and setup status.
- A clear list of missing or misconfigured items.
- Practical remediation recommendations.
- A required follow-up step to re-run this configuration check after remediation.
- Missing packages installed as `devDependencies` in the selected `package.json` when the user confirms the install prompt.
- **Developer mode:** a Markdown report file written to the working directory.
- **Auditor mode:** grouped Jira-ready issue chunks plus file-based bulk-upload and per-issue artifacts for direct Jira filing.
- A false positive registry (`.a11y-config-fps.json`) that the user can update to mark known false positives; the skill creates or updates this file on request.
- A clean working directory on request: when the user is done, the skill removes generated report/artifact files.

## When To Use

- You are either a developer remediating issues or an auditor preparing Jira-ready findings.
- Existing a11y checks are inconsistent or missing in local workflows.
- You want a repeatable local configuration audit workflow for WCAG 2.2 AA-oriented development.
- Run this skill **first**, before `html-validate-ember` or `wcag-audit`, to ensure all required packages and configuration files are in place.

## Required Inputs

- Ember context: Ember version, app or addon, and rendering/testing setup.
- Target `package.json` path if the codebase contains multiple package.json files.
- Repository package manager: yarn, npm, or pnpm if detection is inconclusive.
- Reporting mode: ask at start — `developer` or `auditor`.
- Execution target: local workflow for developer remediation or auditor issue filing.
- Scope: `app/`, `addon/`, `tests/`, shared component libraries, and excluded paths.
- Auditor product context (auditor mode): product name for Jira labeling; ask if not already provided.

## Ember Default Package And Script Guidance

Use these as defaults unless the repo already has an established equivalent:

- Core packages:
  - `ember-template-lint`
  - `eslint` and `eslint-plugin-ember`
  - `@ember/test-helpers`
  - An axe integration for Ember tests (for example `ember-a11y-testing`)
  - `html-validate` and `html-validate-ember` for HTML5 spec and ARIA validation of templates
- Default script names:
  - `lint:hbs` as the expected template lint script name
  - `lint:js` as the expected JS/TS lint script name
  - `test:a11y` as the expected focused accessibility test script name
  - `html-validate-ember` runs via `<package-manager> exec validate-gts`; there is no standard `package.json` script name for it

## Separate Package Processes

Run these as distinct workflows with separate outcomes.

### Install Fallback Protocol

Use this protocol whenever you need to run an install or remove command. Apply the steps in order, stopping as soon as one succeeds:

1. **Primary command:** run `<pm> add --save-dev <packages>` (or `<pm> remove <packages>` for moves).
2. **Force retry:** if step 1 fails, retry the same command once with the `--force` flag:
   - pnpm: `pnpm add --save-dev --force <packages>` (or `pnpm remove --force <packages>`)
   - npm: `npm install --save-dev --force <packages>` (or `npm uninstall --force <packages>`)
   - yarn: `yarn add --dev --force <packages>` (or `yarn remove --force <packages>`)
3. **Package.json fallback:** if step 2 also fails (for example, a pnpm store version conflict that `--force` cannot resolve):
   a. Edit `package.json` directly:
      - To add a missing package: insert it into `devDependencies` with the version range matching the version currently in `node_modules` (use `^` prefix, e.g. `"html-validate": "^11.2.0"`).
      - To move a misplaced package: remove it from `dependencies` and add it to `devDependencies` in the same edit.
   b. After editing, format `package.json`:
      - **Sort keys:** run the following node command to sort `devDependencies` (and `dependencies` if also modified) alphabetically and re-write with 2-space indentation:
        ```
        node -e "const fs=require('fs');const p=JSON.parse(fs.readFileSync('package.json','utf8'));if(p.devDependencies)p.devDependencies=Object.fromEntries(Object.entries(p.devDependencies).sort());if(p.dependencies)p.dependencies=Object.fromEntries(Object.entries(p.dependencies).sort());fs.writeFileSync('package.json',JSON.stringify(p,null,2)+'\n');"
        ```
      - **Format with prettier:** if `prettier` is present in `devDependencies` or available as a binary, also run `<pm> exec prettier --write package.json` to apply the project's formatting conventions. If prettier is not available, the node sort step alone produces valid, consistently indented JSON and is sufficient.
   c. Detect the workspace root: if a `pnpm-workspace.yaml` (or `yarn.lock` / `package-lock.json` at a parent level) exists above the current package directory, run the install command from that root directory. Otherwise run from the package directory.
   d. Run `<pm> install` (not `<pm> add`) — this re-links all node_modules from the current store using the existing lockfile.
   e. After `<pm> install` completes, re-check `package.json` to confirm the package is now in `devDependencies` before continuing.

### Process A: ember-template-lint

- Search the codebase for `package.json` files before package inspection.
- If multiple `package.json` files are found, ask the user which one should be evaluated.
- Use the user-selected `package.json` for the remainder of Process A.
- Detect which package manager the repository is using before suggesting any install command.
- If package manager detection is inconclusive, ask the user whether the repo uses yarn, npm, or pnpm.
- Use the user-provided package manager for the remainder of the skill run.
- Inspect `package.json` scripts for accessibility-related lint command wiring.
- Inspect `package.json` and check both `dependencies` and `devDependencies` for `ember-template-lint`.
- If present in `devDependencies`: no action needed — continue Process A.
- If present in `dependencies` but **not** in `devDependencies`: show the move command and ask the developer to confirm before proceeding:
  > "`ember-template-lint` is in `dependencies` instead of `devDependencies` in `<package.json path>`. Run the following to move it?
  > `<package-manager> remove ember-template-lint`
  > `<package-manager> add --save-dev ember-template-lint`
  > (yes / no)"
  - If confirmed: run each command in sequence. If either command fails, follow the Install Fallback Protocol. Re-check `package.json` to confirm `ember-template-lint` is now in `devDependencies` before continuing.
  - If declined: record the package as misplaced in `dependencies`, note it in the report, and skip the remaining Process A steps.
- If absent from both `dependencies` and `devDependencies`: show the exact install command and ask the developer to confirm before proceeding:
  > "`ember-template-lint` is not in `devDependencies` of `<package.json path>`. Run the following to install it?
  > `<package-manager> add --save-dev ember-template-lint`
  > (yes / no)"
  - If confirmed: run the install command. If it fails, follow the Install Fallback Protocol. Re-check `package.json` to confirm `ember-template-lint` is now present.
  - If declined: record the package as missing, note it in the report, and skip the remaining Process A steps.
- If already present or successfully installed, check for a template-lint config file (`.template-lintrc.js`, `.template-lintrc.mjs`, or `.template-lintrc.cjs`).
- If no template-lint config file is found, report it and stop Process A before rule-level analysis.
- If a template-lint config file exists, compare configured rules against the official a11y rule set:
  - https://github.com/ember-template-lint/ember-template-lint/blob/main/lib/config/a11y.js
- For any a11y rule found set to `'off'` or `'warn'`: update the config file to set it to `'error'` and append an inline comment `// a11y rule — must remain enabled` on the same line; record the change (rule name, old value → `'error'`). Do not report or modify non-a11y rules.
- After the config-file update, search the codebase for inline rule disables.
- Look for template comments that start with `{{!-- template-lint-disable` or `{{! template-lint-disable`.
- Report only inline disable locations where the disabled rule is in the a11y rule set. Do not report inline disables for non-a11y rules.

### Process B: ember-a11y-testing

- Search the codebase for `package.json` files before package inspection.
- If multiple `package.json` files are found, ask the user which one should be evaluated.
- Use the user-selected `package.json` for the remainder of Process B.
- Detect which package manager the repository is using before suggesting any install command.
- If package manager detection is inconclusive, ask the user whether the repo uses yarn, npm, or pnpm.
- Use the user-provided package manager for the remainder of the skill run.
- Inspect `package.json` scripts for accessibility-related test command wiring.
- Inspect `package.json` and check both `dependencies` and `devDependencies` for `ember-a11y-testing`.
- If present in `devDependencies`: no action needed — continue Process B.
- If present in `dependencies` but **not** in `devDependencies`: show the move command and ask the developer to confirm before proceeding:
  > "`ember-a11y-testing` is in `dependencies` instead of `devDependencies` in `<package.json path>`. Run the following to move it?
  > `<package-manager> remove ember-a11y-testing`
  > `<package-manager> add --save-dev ember-a11y-testing`
  > (yes / no)"
  - If confirmed: run each command in sequence. If either command fails, follow the Install Fallback Protocol. Re-check `package.json` to confirm `ember-a11y-testing` is now in `devDependencies` before continuing.
  - If declined: record the package as misplaced in `dependencies`, note it in the report, and skip the remaining Process B steps.
- If absent from both `dependencies` and `devDependencies`: show the exact install command and ask the developer to confirm before proceeding:
  > "`ember-a11y-testing` is not in `devDependencies` of `<package.json path>`. Run the following to install it?
  > `<package-manager> add --save-dev ember-a11y-testing`
  > (yes / no)"
  - If confirmed: run the install command. If it fails, follow the Install Fallback Protocol. Re-check `package.json` to confirm `ember-a11y-testing` is now present.
  - If declined: record the package as missing, note it in the report, and skip the remaining Process B steps.
- If already present or successfully installed, inspect accessibility test coverage setup in Ember tests.
- Report whether a dedicated a11y test path exists and whether it is wired to `test:a11y`.

### Process C: html-validate-ember

- Search the codebase for `package.json` files before package inspection.
- If multiple `package.json` files are found, ask the user which one should be evaluated.
- Use the user-selected `package.json` for the remainder of Process C.
- Detect which package manager the repository is using before suggesting any install command.
- If package manager detection is inconclusive, ask the user whether the repo uses yarn, npm, or pnpm.
- Use the user-provided package manager for the remainder of the skill run.
- Inspect `package.json` and check both `dependencies` and `devDependencies` for `html-validate` and `html-validate-ember`.
- A package is correctly placed only when it is in `devDependencies`. Three possible states for each package: in `devDependencies` ✅, in `dependencies` only ⚠️ (misplaced), or absent ❌.
- If any package is misplaced in `dependencies` (but not in `devDependencies`): show the move command and ask the developer to confirm before proceeding:
  > "One or more packages are in `dependencies` instead of `devDependencies` in `<package.json path>`. Run the following to move them?
  > `<package-manager> remove <misplaced-packages>`
  > `<package-manager> add --save-dev <misplaced-packages>`
  > (yes / no)"
  - If confirmed: run each command in sequence. If either command fails, follow the Install Fallback Protocol. Re-check `package.json` to confirm the packages are now in `devDependencies` before continuing.
  - If declined: record the packages as misplaced, note it in the report, and skip the remaining Process C steps.
- If any package is absent from both `dependencies` and `devDependencies`: show the exact install command and ask the developer to confirm before proceeding:
  > "One or more packages are missing from `devDependencies` of `<package.json path>`. Run the following to install them?
  > `<package-manager> add --save-dev html-validate html-validate-ember`
  > (yes / no)"
  - If confirmed: run the install command. If it fails, follow the Install Fallback Protocol. Re-check `package.json` to confirm both packages are now present.
  - If declined: record the packages as missing, note it in the report, and skip the remaining Process C steps.
- If already present or successfully installed, check for `.htmlvalidate.json` in the project root.
- If `.htmlvalidate.json` is missing, report it, provide the minimal config template below, offer to create the file, and stop Process C before config-level analysis:
  ```json
  {
    "extends": ["html-validate:recommended", "html-validate-ember:gts-recommended"],
    "plugins": ["html-validate-ember"],
    "transform": {
      "^.*\\.(gts|gjs|hbs)$": "html-validate-ember"
    }
  }
  ```
- If `.htmlvalidate.json` exists, inspect the config and verify:
  - `extends` includes `html-validate-ember:gts-recommended` (or `html-validate-ember:recommended` for `.hbs`-only projects)
  - `plugins` includes `"html-validate-ember"`
  - `transform` is configured to route `.gts`, `.gjs`, and `.hbs` files through `html-validate-ember`
- Report any missing or misconfigured config keys.
- After the config-file check, search the codebase for inline html-validate suppress directives:
  - Long form: `{{!-- [html-validate-disable ...] --}}`
  - Short form: `{{! [html-validate-disable ...] }}`
- Report every inline suppress location found and identify which rules are being suppressed inline.

## Procedure

Before Step 1, always ask this exact question: **"Are you a developer or an auditor?"**

- If the answer is **developer**: keep the existing workflow and report shape.
- If the answer is **auditor**: keep the same discovery/remediation logic, but switch final outputs to grouped Jira issue chunks, bulk-upload artifacts, and per-issue files (no audit results table output).

1. Inventory Current State

- Search the codebase for `package.json` files before any other setup work.
- If more than one `package.json` file is found, ask the user which one should be evaluated and use that file for the duration of the skill.
- Inspect the selected `package.json` before any other setup work.
- Detect the repository package manager before suggesting install commands.
- If detection is inconclusive, ask the user to specify yarn, npm, or pnpm and use that answer for the duration of the skill.
- Record package status separately for `ember-template-lint`, `ember-a11y-testing`, `html-validate`, and `html-validate-ember` — noting whether each is in `devDependencies` ✅, in `dependencies` only ⚠️ (misplaced), or absent ❌.
- Identify accessibility-related lint/test script wiring in the selected `package.json`.
- Identify existing accessibility checks in linting and tests by configuration inspection only.
- Capture current false-positive sources and known exceptions.
- Confirm whether shared components live in `app/components` or an addon package.
- Check for `.htmlvalidate.json` in the project root. Record as present or missing.
- If `.htmlvalidate.json` is present, read and record its contents (extends, plugins, transform).
- Enumerate all inline html-validate suppress directives in template files:
  - Long form: `{{!-- [html-validate-disable ...] --}}`
  - Short form: `{{! [html-validate-disable ...] }}`
  - Record every suppressed rule and the file where it appears.
- Check for `.a11y-config-fps.json` in the project root. If present, load all entries and record the count. If absent, record "no false positive registry found."

2. Process A: ember-template-lint

- If `ember-template-lint` is misplaced (in `dependencies` but not `devDependencies`):
  - Show the move command and ask for confirmation:
    > "`ember-template-lint` is in `dependencies` instead of `devDependencies` in `<package.json path>`. Run the following to move it?
    > `<package-manager> remove ember-template-lint`
    > `<package-manager> add --save-dev ember-template-lint`
    > (yes / no)"
  - If confirmed: run each command in sequence. If either fails, follow the Install Fallback Protocol. Re-check `package.json` for `ember-template-lint` in `devDependencies` before continuing Process A.
  - If declined: record the package as misplaced in the report and stop Process A here.
- If `ember-template-lint` is missing from both `dependencies` and `devDependencies`:
  - Show the exact install command and ask for confirmation:
    > "`ember-template-lint` is not in `devDependencies` of `<package.json path>`. Run the following to install it?
    > `<package-manager> add --save-dev ember-template-lint`
    > (yes / no)"
  - If confirmed: run the install command. If it fails, follow the Install Fallback Protocol. Re-check `package.json` for `ember-template-lint` before continuing Process A.
  - If declined: record the package as missing in the report and stop Process A here.
- If `ember-template-lint` is present:
  - Check for a template-lint config file (`.template-lintrc.js`, `.template-lintrc.mjs`, or `.template-lintrc.cjs`).
  - If missing, report and stop Process A at this point.
  - If present, compare configured rules against the official a11y preset rule list. For each a11y rule found set to `'off'` or `'warn'`: update the config file to `'error'`, append an inline comment `// a11y rule — must remain enabled` on the same line, and record the change (rule name, old value → `'error'`). Do not flag or modify non-a11y rules.
  - After the config-file update, search the codebase for comments starting with `{{!-- template-lint-disable` or `{{! template-lint-disable`.
  - Record each inline disable location where the disabled rule is in the a11y rule set. Skip inline disables for non-a11y rules entirely.

3. Process B: ember-a11y-testing

- If `ember-a11y-testing` is misplaced (in `dependencies` but not `devDependencies`):
  - Show the move command and ask for confirmation:
    > "`ember-a11y-testing` is in `dependencies` instead of `devDependencies` in `<package.json path>`. Run the following to move it?
    > `<package-manager> remove ember-a11y-testing`
    > `<package-manager> add --save-dev ember-a11y-testing`
    > (yes / no)"
  - If confirmed: run each command in sequence. If either fails, follow the Install Fallback Protocol. Re-check `package.json` for `ember-a11y-testing` in `devDependencies` before continuing Process B.
  - If declined: record the package as misplaced in the report and stop Process B here.
- If `ember-a11y-testing` is missing from both `dependencies` and `devDependencies`:
  - Show the exact install command and ask for confirmation:
    > "`ember-a11y-testing` is not in `devDependencies` of `<package.json path>`. Run the following to install it?
    > `<package-manager> add --save-dev ember-a11y-testing`
    > (yes / no)"
  - If confirmed: run the install command. If it fails, follow the Install Fallback Protocol. Re-check `package.json` for `ember-a11y-testing` before continuing Process B.
  - If declined: record the package as missing in the report and stop Process B here.
- If `ember-a11y-testing` is present:
  - Inspect Ember rendering/application tests for a11y coverage usage.
  - Ensure a dedicated `test:a11y` path exists for focused accessibility checks.
  - Ensure the a11y test path is correctly wired in configuration.

4. Process C: html-validate-ember

- If `html-validate` or `html-validate-ember` is misplaced (in `dependencies` but not `devDependencies`):
  - Show the move command and ask for confirmation:
    > "One or more packages are in `dependencies` instead of `devDependencies` in `<package.json path>`. Run the following to move them?
    > `<package-manager> remove <misplaced-packages>`
    > `<package-manager> add --save-dev <misplaced-packages>`
    > (yes / no)"
  - If confirmed: run each command in sequence. If either fails, follow the Install Fallback Protocol. Re-check `package.json` for both packages in `devDependencies` before continuing Process C.
  - If declined: record the packages as misplaced in the report and stop Process C here.
- If `html-validate` or `html-validate-ember` is missing from both `dependencies` and `devDependencies`:
  - Show the exact install command and ask for confirmation:
    > "One or more packages are missing from `devDependencies` of `<package.json path>`. Run the following to install them?
    > `<package-manager> add --save-dev html-validate html-validate-ember`
    > (yes / no)"
  - If confirmed: run the install command. If it fails, follow the Install Fallback Protocol. Re-check `package.json` for both packages before continuing Process C.
  - If declined: record the packages as missing in the report and stop Process C here.
- If both packages are present:
  - Check for `.htmlvalidate.json` in the project root.
  - If missing, report and stop Process C at this point.
  - If present, inspect the config and verify:
    - `extends` includes `html-validate-ember:gts-recommended` (or `html-validate-ember:recommended` for `.hbs`-only projects)
    - `plugins` includes `"html-validate-ember"`
    - `transform` routes `.gts`, `.gjs`, and `.hbs` files through `html-validate-ember`
  - Report any missing or misconfigured config keys.
  - After the config-file check, enumerate inline html-validate suppress directives in template files:
    - Long form: `{{!-- [html-validate-disable ...] --}}`
    - Short form: `{{! [html-validate-disable ...] }}`
  - Record each inline suppress location and the suppressed rule names.

5. Select Tooling Layers

- Static checks:
  - Verify `ember-template-lint` accessibility rules for templates are configured.
  - Verify `html-validate-ember` is installed and `.htmlvalidate.json` is configured for HTML5 spec and ARIA validation.
  - Verify ESLint accessibility checks for JavaScript/TypeScript where applicable.
- Runtime checks:
  - Verify accessibility scanning integration is configured in Ember rendering/application tests (for example with axe-based helpers).
- Route/page checks:
  - Verify browser-level scan tooling configuration for critical Ember routes and user journeys.
- Ensure configuration indicates at least one static layer and one runtime layer.

6. Define Policy And Thresholds

- Classify violations by severity: critical, serious, moderate, minor.
- Define expected local scan behavior by severity:
  - Block on critical and serious.
  - Report moderate and minor.
- Define temporary exceptions with owner and expiration date.

7. Implement Local Developer Experience

- Verify lint command wiring for changed files and full-project runs.
- Verify fast a11y test command wiring for pre-commit validation.
- Ensure commands are mapped to Ember scripts so contributors can run them via the package manager.
- Add documentation for how to reproduce configuration issues found by this skill.

8. Verify End-To-End

- Confirm package/configuration checks produce consistent results from static inspection.
- Validate report output, ownership mapping, and exception expiry checks.

9. Handle False Positive Requests

This step is **user-triggered**. Execute it when the user says "mark row N as a false positive", "mark finding N as a false positive", or similar.

- Identify the finding from the current report context: use the row number to locate the check name, location, issue description, and status.
- Ask for a reason if the user did not provide one; a reason is required for every registry entry.
- Create or update `.a11y-config-fps.json` in the project root:
  - If the file does not exist, create it with the schema shown in the `## False Positive Registry` section.
  - If the file already exists, append the new entry to the `entries` array.
  - Set `"added"` to today's date in `YYYY-MM-DD` format.
- Confirm back to the user: print the entry that was written and state that subsequent runs will place this finding in the False Positives section of the report.
- To **unmark** a false positive, remove its entry from the `entries` array in `.a11y-config-fps.json`.

10. Produce Config Report

- In **developer** mode: write the full configuration audit report to a file in the working directory following the `## Report File Naming` convention.
- In **auditor** mode: write grouped Jira chunk, bulk-upload, and per-issue artifacts following the `## Report File Naming` convention and `## Jira Label Generation (Auditor Mode)`.
- Immediately below the report title and before metadata, include a warning that the report was generated with an AI skill and all findings/results must be verified by a human reviewer.
- Format the report metadata section as a Markdown bulleted list (not paragraphs or tables), including at minimum: audit date, project name, selected `package.json` path, detected package manager, `ember-template-lint` status (version in devDependencies / misplaced in dependencies / missing), `ember-a11y-testing` status (version in devDependencies / misplaced in dependencies / missing), `html-validate-ember` status (version in devDependencies / misplaced in dependencies / missing), `html-validate` status (version in devDependencies / misplaced in dependencies / missing), template-lint config file path and name, `.htmlvalidate.json` status (present/missing) and config source (pre-existing or absent), scope, script wiring status (`lint:hbs` and `test:a11y` present or absent), a11y rules re-enabled in the template-lint config (count and rule names, or "none — all a11y rules already at `error`"), template-lint a11y inline disables (count and rule names; non-a11y inline disables excluded), html-validate inline suppress count and rules, and false positive registry status (present with N entries, or absent).
- In **developer** mode: present audit findings in a table where the first column is a sequential row number (`1`, `2`, `3`, ...). Only include findings not moved to the active false positives list.
- In **auditor** mode: do not emit audit results tables; instead, group findings by configuration category and shared root cause into Jira issue chunks and generate labels using `## Jira Label Generation (Auditor Mode)`.
- Include detailed evidence for every failing or warning item, including: current configured value, expected value, file path, code reference (rule setting or inline disable), and recommended fix.
- Cross-reference each finding against the false positive registry (if loaded in Step 1):
  - A finding **matches** a registry entry when its `check` and `location` both equal the entry's stored values (case-insensitive, trimmed).
  - Move matched findings out of the active findings list and into a separate **active false positives** list.
  - Any registry entry that did not match any current finding is a **stale entry** — record it separately.
  - Findings that did not match any registry entry remain in the active findings list as real configuration gaps.
- State a readiness recommendation: ready / conditional pass with accepted risk / not ready. Include the count of active false positives suppressed this run in the recommendation summary.

11. Finish Config Session

See [shared cleanup procedure](../shared/cleanup-procedure.md) for complete details.

**Skill-specific file patterns:**
- Developer mode: `*-a11y-config.md`, `*-a11y-config_NN.md`
- Auditor mode: `*-a11y-config-jira-chunks.md`, `*-a11y-config-jira-bulk.csv`, `*-a11y-config-jira-issue-*.md`
- False positive registry to preserve: `.a11y-config-fps.json`

## Branching Rules

- If many issues are found:
  - Prioritize fixes by severity and user impact, then re-run this configuration skill.
- If no UI test framework exists:
  - Start with `ember-template-lint` plus route-level scanner configuration and add test-layer configuration later.
- If `html-validate-ember` or `html-validate` is missing from `devDependencies` or misplaced in `dependencies`: show the install or move command, ask for confirmation, run it on approval (following the Install Fallback Protocol if it fails), and re-check before continuing Process C (Step 4).
- If `ember-template-lint` is missing from `devDependencies` or misplaced in `dependencies`: show the install or move command, ask for confirmation, run it on approval (following the Install Fallback Protocol if it fails), and re-check before continuing Process A (Step 2).
- If `ember-a11y-testing` is missing from `devDependencies` or misplaced in `dependencies`: show the install or move command, ask for confirmation, run it on approval (following the Install Fallback Protocol if it fails), and re-check before continuing Process B (Step 3).
- If the user declines any install or move prompt: record the package as missing or misplaced, note it in the report, and skip the remaining steps for that process.
- If `.htmlvalidate.json` is missing: report it, offer to create it using the minimal config below, and stop Process C config inspection. If the user accepts, write the file before continuing:
  ```json
  {
    "extends": ["html-validate:recommended", "html-validate-ember:gts-recommended"],
    "plugins": ["html-validate-ember"],
    "transform": {
      "^.*\\.(gts|gjs|hbs)$": "html-validate-ember"
    }
  }
  ```
- If `@glint/ember-tsc` is absent from `devDependencies`: note in the report that Glint-powered component resolution will not be available when the validator is run.
- If reporting mode is **auditor** and product is not provided: ask which product is being audited before generating Jira labels. If the product name differs from the canonical list, infer the closest mapped label from `## Jira Label Generation (Auditor Mode)`.
- If `.a11y-config-fps.json` does not exist: skip false positive matching entirely; the findings list is the complete set of real configuration gaps.
- If a false positive registry entry does not match any finding in the current run: mark it as a stale entry in the report; do not treat the absence of a match as an error.
- If reporting mode is **auditor**: group issues by configuration category/root-cause clusters whenever possible and output Jira chunks, bulk-upload artifacts, and per-issue files instead of audit results tables.
- If the user requests to mark a finding as a false positive: execute Step 9 (Handle False Positive Requests) regardless of which step the skill is currently executing.
- If the user says "I'm done", "finish", "finish audit", "wrap up", "clean up for PR", "remove the report", or similar: execute Step 11 (Finish Config Session) immediately, regardless of which step the skill is currently executing.
- **Finish — No generated files found**: Inform the user that the working directory is already clean; no action needed.
- **Finish — User declines confirmation**: No files are deleted. Confirm explicitly that nothing changed and the generated files remain in place.

## Quality Gates

Setup is complete only when all are true:

- Required a11y packages are installed in the selected `package.json`: `ember-template-lint`, `ember-a11y-testing`, `html-validate`, and `html-validate-ember`.
- Required configuration files and script wiring are present and correctly mapped.
- Severity policy and exception process are documented.
- At least one critical user journey has scanner configuration defined.
- Exception entries include owner and expiry date.
- In **developer** mode: report filename follows `<project-name>-a11y-config.md` with `_NN` suffixing for subsequent runs.
- Report includes an AI-generated warning directly under the title and before metadata.
- Report metadata at the top is rendered as a Markdown bulleted list.
- In **developer** mode: audit results table uses numbered rows as the first column.
- In **auditor** mode: findings are grouped into Jira issue chunks (no audit results table), with one chunk per grouped issue bundle whenever possible.
- In **auditor** mode: every Jira issue chunk includes an impact statement.
- In **auditor** mode: every Jira issue chunk and bulk row uses the required label contract from `## Jira Label Generation (Auditor Mode)`.
- In **auditor** mode: Jira artifacts include `<project-name>-a11y-config-jira-chunks.md`, `<project-name>-a11y-config-jira-bulk.csv`, and `<project-name>-a11y-config-jira-issue-NNN.md` files.
- Detailed evidence is present for every failing or warning item.
- False positive registry status (present with N entries, or absent) is recorded in report metadata.
- Readiness summary includes the count of active false positives suppressed this run (or zero if none).
- If the registry is present: Active False Positives table is present in the report (or explicitly marked empty).
- If the registry is present: Stale False Positives section is present in the report (or explicitly marked none).

## Output Format

Return results in this order, based on reporting mode:

### Developer mode

1. AI-generated warning (below title, before metadata): this report was generated with an AI skill and all findings/results should be verified by a human.
2. Report metadata (Markdown bulleted list): audit date, project name, selected `package.json` path, detected package manager, `ember-template-lint` status (version in devDependencies / misplaced in dependencies / missing), `ember-a11y-testing` status (version in devDependencies / misplaced in dependencies / missing), `html-validate-ember` status (version in devDependencies / misplaced in dependencies / missing), `html-validate` status (version in devDependencies / misplaced in dependencies / missing), template-lint config file path and name, `.htmlvalidate.json` status (present/missing) and config source (pre-existing or absent), scope, script wiring status (`lint:hbs` and `test:a11y` present or absent), a11y rules re-enabled in the template-lint config (count and rule names, or "none — all a11y rules already at `error`"), template-lint a11y inline disables (count and rule names; non-a11y inline disables excluded), html-validate inline suppress count and rules, and false positive registry status (present with N entries, or absent).
3. Audit results table: row number, check name, status (Pass / Fail / Warn), location or file, brief issue description. Only real configuration gaps appear here; false positive-matched rows are excluded.
4. Remediation plan: grouped by category — install-required, config change, test wiring, policy. Include recommended fix and priority for each item.
5. Active False Positives table (when `.a11y-config-fps.json` exists and entries matched): row number, check, location, issue description, reason, date added. Findings suppressed via the registry appear here instead of in the audit results table. Omit this section entirely if no findings matched any registry entry this run.
6. Stale False Positives (when `.a11y-config-fps.json` exists and entries did not match): registry entries that did not match any finding in this run, with stored check and location. Each entry should be reviewed — the issue may have been resolved or the location text may have changed. Omit this section entirely if there are no stale entries.
7. Detailed evidence report: per-item current value, expected value, file path, code reference (rule setting or inline disable directive), recommended fix, and verification notes.
8. Re-scan summary and readiness recommendation: ready / conditional pass with accepted risk / not ready. Include the count of active false positives suppressed this run (or zero if none).
9. Residual risks, accepted exceptions, and next actions.

### Auditor mode (Jira-oriented)

1. AI-generated warning and metadata (same required metadata fields as developer mode).
2. Auditor summary: ready / conditional pass / not ready with grouped counts by configuration category.
3. **Grouped Jira issue chunks** (not tables): group gaps by install-required, config change, test wiring, and policy/root-cause clusters whenever possible. Each chunk must be copy/paste ready and include: issue type, summary/title, severity/priority, labels, components, affected files/locations, impact statement, evidence, WCAG criterion references, WCAG Failure references (when known), and acceptance criteria. Jira summary/title must start with what is wrong (for example: "Incorrect use of ...", "Missing accessible name for ...", "Lack of ...", "Malformed syntax ..."). Labels must be generated using `## Jira Label Generation (Auditor Mode)`.
4. Active and stale false positive sections, expressed as Jira notes/chunks (not audit results tables).
5. Jira artifact manifest describing files written for filing, and confirm that chunk, bulk-upload, and per-issue artifacts were generated.
6. Re-scan and follow-up notes.

## Jira Label Generation (Auditor Mode)

See [shared Jira label generation rules](../shared/jira-labels.md).

## Report File Naming

See [shared report naming conventions](../shared/report-naming.md). This skill uses:
- Developer mode: `<project-name>-a11y-config.md` (with `_NN` suffixing for reruns)
- Auditor mode: `<project-name>-a11y-config-jira-chunks.md`, `<project-name>-a11y-config-jira-bulk.csv`, `<project-name>-a11y-config-jira-issue-NNN.md`

## False Positive Registry

The false positive registry is stored as `.a11y-config-fps.json` in the project root. See [shared false positive registry documentation](../shared/false-positive-registry.md) for complete details on file format, matching algorithm, and usage guidance.

**Skill-specific notes:**
- Registry file name: `.a11y-config-fps.json`
- Created when user marks a finding as false positive (Step 9)
- Read during inventory (Step 1) on every subsequent run

### File format

```json
{
  "version": 1,
  "entries": [
    {
      "check": "ember-template-lint/rule-disabled: no-invalid-interactive",
      "location": ".template-lintrc.mjs — rules['no-invalid-interactive']",
      "issue": "no-invalid-interactive set to 'off'; interactive event handlers on non-interactive elements will not be caught",
      "reason": "Intentional: legacy interactive divs in vendor component; tracked for replacement in Q3.",
      "added": "2026-05-20"
    }
  ]
}
```

### Field definitions

**Example entry for this skill:**
```json
{
  "check": "ember-template-lint/rule-disabled: no-invalid-interactive",
  "location": ".template-lintrc.mjs — rules['no-invalid-interactive']",
  "issue": "no-invalid-interactive set to 'off'; interactive event handlers on non-interactive elements will not be caught",
  "reason": "Intentional: legacy interactive divs in vendor component; tracked for replacement in Q3.",
  "added": "2026-05-20"
}
```

## Hard Constraint

- This skill must not run lint commands.
- This skill must not run test commands.
- This skill **may** run package manager install commands (`pnpm add --save-dev`, `npm install --save-dev`, `yarn add --dev`) when a required package is missing — but only after the developer explicitly confirms the install prompt.
- This skill **may** run package manager remove commands (`pnpm remove`, `npm uninstall`, `yarn remove`) when a required package needs to be moved from `dependencies` to `devDependencies` — but only after the developer explicitly confirms the move prompt.
- This skill **may** retry any failed install or remove command once with the appropriate force flag (`pnpm add --save-dev --force`, `npm install --save-dev --force`, `yarn add --dev --force`, or the equivalent remove with `--force`) when the initial command fails.
- This skill **may**, as a further fallback when both the primary command and the `--force` retry have failed (for example, a pnpm store version conflict), directly edit `package.json` to add or move `devDependencies` entries, then run `<pm> install` from the workspace root to re-link node_modules from the current store.
- When editing `package.json` directly, this skill **may** run a `node` command to sort `devDependencies` and `dependencies` keys alphabetically and re-write the file with 2-space indentation, and **may** run `<pm> exec prettier --write package.json` if prettier is available, to restore correct file formatting.
- This skill **may** write to configuration files in two cases: (1) updating the template-lint config file to re-enable a11y rules set to `'off'` or `'warn'`, and (2) creating `.htmlvalidate.json` when it is absent and the developer confirms.
- Outside of install/remove commands and the permitted config file writes above, this skill inspects package installation status and configuration state only.
