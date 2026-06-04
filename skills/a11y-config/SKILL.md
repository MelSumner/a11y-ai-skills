---
name: a11y-config
description: "Audit accessibility package installation and configuration in Ember.js codebases without executing lint or test commands."
argument-hint: "Provide Ember version, selected package.json path when needed, package manager, and scope for static configuration review."
user-invocable: true
---

# Accessibility Configuration

## What This Skill Produces

- A configuration-only report of package and setup status.
- A clear list of missing or misconfigured items.
- Missing packages installed as `devDependencies` in the selected `package.json` when the user confirms the install prompt.


## When To Use

- You want to make sure your accessibility automation for your Ember app are all installed and configured correctly.
- Existing a11y checks are inconsistent or missing in local workflows.
- You want a repeatable local configuration audit workflow for WCAG 2.2 AA-oriented development.
- Run this skill **first**, before `html-validate` or `wcag-audit`, to ensure all required packages and configuration files are in place.

## Required Inputs

- Ember context: Ember version, app or addon, and rendering/testing setup.
- Target `package.json` path if the codebase contains multiple package.json files.
- Repository package manager: yarn, npm, or pnpm if detection is inconclusive.
- Reporting mode: ask at start — `developer` or `auditor`.
- Execution target: local workflow for developer remediation or auditor issue filing.
- Scope: `app/`, `addon/`, `tests/`, shared component libraries, and excluded paths.
- Auditor product context (auditor mode): product name for Jira labeling; ask if not already provided.

## Ember Default Package And Script Guidance

Use these as defaults:

- Core packages:
  - `ember-template-lint`
  - `eslint` and `eslint-plugin-ember`
  - `@ember/test-helpers`
  - `ember-a11y-testing`
  - `html-validate` and `html-validate-ember`

## Separate Package Processes

Run these as distinct workflows with separate outcomes.

### Install Fallback Protocol

Use this protocol whenever you need to run an install or remove command.
Do not accept a package that is a transitive dependency; it must exist in the package.json.

Apply the steps in order, stopping as soon as one succeeds:

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

### Process:

Repeat this for each of the Core Packages, using specific sub-steps for packages when appropriate:

- Search the codebase for `package.json` files before package inspection.
- If multiple `package.json` files are found, ask the user which one should be evaluated.
- Use the user-selected `package.json` for the remainder of this Process.
- Detect which package manager the repository is using before suggesting any install command.
- If package manager detection is inconclusive, ask the user whether the repo uses yarn, npm, pnpm, or other (allow them to give a different answer).
- Use the user-provided package manager for the remainder of the skill run.
- Inspect `package.json` and check both `dependencies` and `devDependencies` for the Core Packages listed above.
- If present in `devDependencies`: no action needed — continue Process.
- If present in `dependencies` but **not** in `devDependencies` run each command in sequence. If any command fails, follow the Install Fallback Protocol. Re-check `package.json` to confirm each of the Core Packages is now in `devDependencies` before continuing.
- If absent from both `dependencies` and `devDependencies`: run the install command. If it fails, follow the Install Fallback Protocol. Re-check `package.json` to confirm each of the Core Packages is now present.
- Sub-steps for `ember-template-lint`:
  - If already present or successfully installed, check for a template-lint config file (`.template-lintrc.js`, `.template-lintrc.mjs`, or `.template-lintrc.cjs`).
  - If no template-lint config file is found, add it with the recommended configuration.
  - If a template-lint config file exists, compare configured rules against the template-lint a11y rule set in `rule-checks.md`
  - For any a11y rule found set to `'off'` or `'warn'`: update the config file to set it to `'error'` and append an inline comment `// a11y rule — must remain enabled` on the same line; record the change (rule name, old value → `'error'`).
  - Do not report on, or modify, non-a11y rules.
  - After the config-file update, search the codebase for inline rule disables.
  - Look for template comments that start with `{{!-- template-lint-disable` or `{{! template-lint-disable`.
  - Report only inline disable locations where the disabled rule is in the a11y rule set. Do not report inline disables for non-a11y rules.
- Sub-steps for `ember-a11y-testing`:
  - Look for the `test-helper.js` file.
  - If found, ensure that no a11y rules are set to `false`.
  - If a11y rules are set to `false`, set them to `true`.
  - Next, inspect accessibility test coverage setup in Ember tests.
  - Report whether a dedicated a11y test path exists.
- Sub-steps for `html-validate-ember`:
  - If already present or successfully installed, check for `.htmlvalidate.json` in the project root.
  - If `.htmlvalidate.json` is missing, create it with the minimal config template below, and add custom rule setup as defined in `rule-checks.md`:
    ```json
    {
    	"extends": [
    		"html-validate:standard",
    		"html-validate-ember:gts-recommended"
    	],
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
    - `rules` is configured to turn rules off based on the HTML Validate section in `rule-checks.md`
  - Report any missing or misconfigured config keys.
  - After the config-file check, search the codebase for inline html-validate suppress directives:
    - Long form: `{{!-- [html-validate-disable ...] --}}`
    - Short form: `{{! [html-validate-disable ...] }}`
  - Report every inline suppress location found and identify which rules are being suppressed inline.
- Produce Config Report
  - Immediately below the report title and before metadata, include a warning that the report was generated with an AI skill and all findings/results must be verified by a human reviewer.
  - List all changes that were made.
