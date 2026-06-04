# wcag-audit Skill

## High-Level Summary

The `wcag-audit` skill runs automated WCAG 2.2 AA accessibility checks against an Ember.js codebase, maps findings to specific success criteria, classifies issues by severity, and produces structured outputs for remediation and reporting.

It starts by asking whether you are a **developer** or an **auditor**. Developer mode keeps the current findings-report flow; auditor mode outputs grouped Jira-ready issue chunks plus bulk-upload artifacts with deterministic labels (`a11y-audit`, mapped product label, `wcag-x-x-x`, exactly one `a11y-sev:*`, optional `wcag-fN`). Auditor issue chunks follow the shared [issue format rules](../shared/issue-format.md). Manual testing — keyboard navigation, screen reader behavior, visual checks, and motion checks — is still required. **The output of this skill is not a formal compliance determination.**

---

## What It Does

1. Starts with a required role question: "Are you a developer or an auditor?"
2. Inventories all routes, components, and templates in scope before running any checks.
3. Records all suppressed rules, skipped tests, and inline disable directives as risk evidence — not pass evidence.
4. Runs available automated a11y linters and template lint scripts (e.g. `lint:hbs`, `lint:templates`).
5. Incorporates `html-validate-ember` results from earlier in the session when available.
6. Checks across multiple UI state variants: initial render, loading, error, empty, open/expanded, and populated.
7. Maps every finding to the specific WCAG 2.2 Success Criterion it fails, including Understanding document links and applicable Techniques/Failures references.
8. Classifies each issue by severity:
   - **Critical (Blocker)** — prevents core task completion or creates critical exclusion
   - **Serious** — substantial friction or likely WCAG failure on a key journey
   - **Major** — meaningful issue with an available workaround
   - **Minor** — limited impact or edge case
9. Groups fixes by root cause so one change can resolve multiple instances.
10. Manages a false positive registry (`.wcag-audit-fps.json`) — findings you acknowledge as intentional are tracked separately across runs, not silently dropped.
11. In **developer mode**, writes a Markdown audit report with findings tables, detailed evidence, remediation plan, and an automated audit outcome (pass / conditional pass / fail).
12. In **auditor mode**, outputs grouped Jira-ready issue chunks (instead of findings tables) plus bulk-upload artifacts, with issue structure defined by the shared [issue format rules](../shared/issue-format.md), each Jira issue title starting with what is wrong (for example: "Incorrect use of ...", "Missing accessible name for ...", "Lack of ...", "Malformed syntax ..."), and labels generated from the standard contract (`a11y-audit`, product label, `wcag-x-x-x`, single `a11y-sev:*`, optional `wcag-fN`).
13. Cleans up generated report/artifact files on request.

---

## What It Does NOT Do

1. **Does not replace manual testing.** Keyboard navigation, screen reader behavior, visual/zoom checks, and motion checks must be performed by a human.
2. **Does not produce a formal compliance determination.** The automated audit outcome is a triage signal, not a legal or regulatory compliance decision.
3. **Does not install or configure packages.** Run the `a11y-config` skill first for that.
4. **Does not run HTML5 spec validation.** Run the `html-validate-ember` skill for that — its results feed into this skill.
5. **Does not silently pass checks when tools are missing.** Missing scripts or tools are recorded as coverage gaps with explicit residual risk.

---

## Where It Fits

This is **skill 3 of 3** in the accessibility workflow. Run them in order:

1. **`a11y-config`** — verifies packages are installed and configuration files are correct
2. **`html-validate-ember`** — runs HTML5 spec and ARIA validation against `.gts`, `.gjs`, and `.hbs` template files
3. **`wcag-audit`** ← *this skill* — runs automated WCAG 2.2 AA checks and produces a findings report

If the `a11y-config` preflight check finds missing tools or scripts, the skill will redirect you there before continuing.

---

## Usage Commands

> **Tip:** Explicitly mentioning that this is an Ember codebase when invoking the skill produces better results.

| Action | What to say |
|---|---|
| Start the skill | `"load the wcag-audit skill and run it on this Ember codebase"` |
| Start auditor-oriented reporting | `"run wcag-audit and use auditor reporting mode"` |
| Re-run after fixes | `"re-run the wcag-audit skill on this Ember codebase"` |
| Mark a finding as a false positive | `"mark row N as a false positive"` |
| Finish and clean up before a PR | `"finish audit"` / `"wrap up"` / `"clean up for PR"` |

When marking a false positive, a written reason is required. The entry is saved to `.wcag-audit-fps.json` and will be tracked separately on all future runs.

---

## Files Produced

| File | Description | Commit to source? |
|---|---|---|
| `<project-name>-a11y-audit.md` | Developer-mode audit report for this run | No — remove before PR |
| `<project-name>-a11y-audit_01.md`, `_02.md`, … | Developer-mode reruns are numbered sequentially | No — remove before PR |
| `<project-name>-a11y-audit-jira-chunks.md` | Auditor-mode copy/paste Jira issue chunks | No — generated artifact |
| `<project-name>-a11y-audit-jira-bulk.csv` | Auditor-mode Jira bulk upload file | No — generated artifact |
| `<project-name>-a11y-audit-jira-issue-NNN.md` | Per-issue Jira files (one file per grouped issue chunk) | No — generated artifact |
| `.wcag-audit-fps.json` | False positive registry | **Yes** — project config, commit with code changes |

The clean-up command removes generated report/artifact files only. The false positive registry is never deleted by clean-up.
