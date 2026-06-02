# a11y-config Skill

## High-Level Summary

The `a11y-config` skill audits package installation and accessibility tool configuration in Ember.js codebases. It starts by asking whether you are a **developer** or an **auditor**, then uses the same checks with role-specific reporting outputs.

This skill does not run lint or test commands. It is configuration inspection and setup only.

---

## What It Does

1. Starts with a required role question: "Are you a developer or an auditor?"
2. Locates all `package.json` files in the codebase and asks which one to evaluate when multiple are found.
3. Detects whether `ember-template-lint`, `ember-a11y-testing`, `html-validate`, and `html-validate-ember` are installed in the correct place — `devDependencies`, not `dependencies`.
4. Offers to install missing packages or move misplaced ones, with an explicit confirmation prompt for each action.
5. Follows a three-step fallback if an install fails: primary command → `--force` retry → direct `package.json` edit + `<pm> install`.
6. Checks for a template-lint config file (`.template-lintrc.js`, `.template-lintrc.mjs`, or `.template-lintrc.cjs`) and updates any a11y rules set to `'off'` or `'warn'` to `'error'`, adding an inline comment to protect them.
7. Reports every inline `template-lint-disable` comment in templates that targets an a11y rule.
8. Checks for `.htmlvalidate.json`; if it is missing, offers to create it with the correct preset configuration for Ember.
9. Verifies that `.htmlvalidate.json` has the required `extends`, `plugins`, and `transform` entries for Ember template files.
10. Reports every inline `{{!-- [html-validate-disable ...] --}}` suppress directive found in templates.
11. Manages a false positive registry (`.a11y-config-fps.json`) — configuration gaps you acknowledge as intentional are tracked separately across runs.
12. In **developer mode**, writes a Markdown config report with a readiness recommendation (ready / conditional pass / not ready).
13. In **auditor mode**, outputs grouped Jira-ready issue chunks (instead of audit results tables) plus bulk-upload artifacts, with each Jira issue title starting with what is wrong (for example: "Incorrect use of ...", "Missing accessible name for ...", "Lack of ...", "Malformed syntax ...") and labels generated from the standard contract (`a11y-audit`, product label, `wcag-x-x-x`, single `a11y-sev:*`, optional `wcag-fN`).
14. Cleans up generated report/artifact files on request.

---

## What It Does NOT Do

1. **Does not run lint commands.** Running `ember-template-lint` or any other linter is outside the scope of this skill.
2. **Does not run test commands.** Test execution is out of scope.
3. **Does not install, move, or modify anything without your confirmation.** Every install prompt, move prompt, and config file write requires an explicit yes before the skill proceeds.

---

## Where It Fits

This is **skill 1 of 3** in the accessibility workflow. Run them in order:

1. **`a11y-config`** ← *this skill* — verifies packages are installed and configuration files are correct
2. **`html-validate`** — runs HTML5 spec and ARIA validation against `.gts`, `.gjs`, and `.hbs` template files
3. **`wcag-audit`** — runs automated WCAG 2.2 AA checks and produces a findings report

Run this skill first. Both `html-validate` and `wcag-audit` will redirect you here if required packages or config files are missing.

---

## Usage Commands

> **Tip:** Explicitly mentioning that this is an Ember codebase when invoking the skill produces better results.

| Action | What to say |
|---|---|
| Start the skill | `"load the a11y-config skill and run it on this Ember codebase"` |
| Start auditor-oriented reporting | `"run a11y-config and use auditor reporting mode"` |
| Re-run after fixes | `"re-run the a11y-config skill"` |
| Finish and clean up before a PR | `"finish"` / `"wrap up"` / `"clean up for PR"` |

---

## Files Produced

| File | Description | Commit to source? |
|---|---|---|
| `<project-name>-a11y-config.md` | Developer-mode config audit report for this run | No — remove before PR |
| `<project-name>-a11y-config_01.md`, `_02.md`, … | Developer-mode reruns are numbered sequentially | No — remove before PR |
| `<project-name>-a11y-config-jira-chunks.md` | Auditor-mode copy/paste Jira issue chunks | No — generated artifact |
| `<project-name>-a11y-config-jira-bulk.csv` | Auditor-mode Jira bulk upload file | No — generated artifact |
| `<project-name>-a11y-config-jira-issue-NNN.md` | Per-issue Jira files (one file per grouped issue chunk) | No — generated artifact |
| `.a11y-config-fps.json` | False positive registry | **Yes** — project config, commit with code changes |
| `.htmlvalidate.json` | HTML validator config (created if missing, on confirmation) | **Yes** — required config, commit with code changes |

The clean-up command removes generated report/artifact files only. The false positive registry and `.htmlvalidate.json` are never deleted by clean-up.
