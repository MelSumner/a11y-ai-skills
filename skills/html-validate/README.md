# html-validate Skill

## High-Level Summary

The `html-validate` skill runs the `validate-gts` binary against Ember template files (`.gts`, `.gjs`, `.hbs`), surfaces every violation at rule ID / file path / line:column, and maps findings to the relevant HTML Living Standard section or WCAG 2.2 Success Criterion.

It starts by asking whether you are a **developer** or an **auditor**. Developer mode keeps the current report flow; auditor mode outputs Jira-ready grouped issue chunks and bulk-upload artifacts with deterministic labels (`a11y-audit`, mapped product label, `wcag-x-x-x`, exactly one `a11y-sev:*`, optional `wcag-fN`). It covers what the HTML5 spec and ARIA rules can detect statically. It is not a full WCAG audit — run `wcag-audit` after this skill for that.

---

## What It Does

1. Starts with a required role question: "Are you a developer or an auditor?"
2. Runs a preflight check before anything else: verifies `html-validate` and `html-validate-ember` are in `devDependencies`, confirms the `validate-gts` binary is present, and confirms `.htmlvalidate.json` exists — redirects to `a11y-config` if anything is missing.
3. Inventories all inline `{{!-- [html-validate-disable ...] --}}` suppress directives across templates before running the validator.
4. Runs `validate-gts` against the confirmed scope (`app/`, `tests/`, `addon/` by default). Falls back to `--no-glint` when `@glint/ember-tsc` is absent from `devDependencies`.
5. Categorizes every finding into one of four rule families:
   - **Content Model** — HTML5 nesting and parse-time structure (`element-permitted-content`, `no-implicit-close`, etc.)
   - **ARIA / Accessibility** — ARIA spec and landmark rules (`wcag/*` rules, `aria-label-misuse`, `input-missing-label`, etc.)
   - **Form Correctness** — form element rules (`wcag/h32`, `wcag/h71`, `form-dup-name`, etc.)
   - **Attribute Validity** — attribute value and presence rules (`attribute-allowed-values`, `no-raw-characters`, etc.)
6. Maps every ARIA/accessibility finding to the relevant WCAG 2.2 Success Criterion, including Understanding document links and applicable Techniques/Failures references.
7. Maps every content model finding to the relevant HTML Living Standard section.
8. Produces a suppression ledger listing every inline suppress directive found, the rule suppressed, and the file location.
9. Manages a false positive registry (`.html-validate-fps.json`) — violations you acknowledge as intentional are tracked separately across runs, not silently dropped.
10. In **developer mode**, writes a Markdown validation report with findings grouped by rule family.
11. In **auditor mode**, outputs grouped Jira-ready issue chunks (instead of findings tables) plus bulk-upload artifacts, with each Jira issue title starting with what is wrong (for example: "Incorrect use of ...", "Missing accessible name for ...", "Lack of ...", "Malformed syntax ...") and labels generated from the standard contract (`a11y-audit`, product label, `wcag-x-x-x`, single `a11y-sev:*`, optional `wcag-fN`).
12. Cleans up generated report/artifact files on request.

---

## What It Does NOT Do

1. **Does not install or configure packages.** If `html-validate-ember` or `validate-gts` is missing, the skill stops and redirects to `a11y-config`.
2. **Does not cover full WCAG testing.** Keyboard navigation, screen reader behavior, visual/zoom checks, and motion checks are out of scope — run `wcag-audit` for those.
3. **Does not check Ember template idioms.** Ember-specific linting (component usage, angle bracket syntax, etc.) is the role of `ember-template-lint`, not this skill.
4. **Does not silently pass when the validator cannot run.** Tool errors are recorded with explicit coverage gaps and residual risk notes.

---

## Where It Fits

This is **skill 2 of 3** in the accessibility workflow. Run them in order:

1. **`a11y-config`** — verifies packages are installed and configuration files are correct
2. **`html-validate-ember`** ← *this skill* — runs HTML5 spec and ARIA validation against `.gts`, `.gjs`, and `.hbs` template files
3. **`wcag-audit`** — runs automated WCAG 2.2 AA checks and produces a findings report

The `wcag-audit` skill will incorporate results from this skill if both are run in the same session.

---

## Usage Commands

> **Tip:** Explicitly mentioning that this is an Ember codebase when invoking the skill produces better results.

| Action | What to say |
|---|---|
| Start the skill | `"load the html-validate-ember skill and run it on this Ember codebase"` |
| Start auditor-oriented reporting | `"run html-validate-ember and use auditor reporting mode"` |
| Re-run after fixes | `"re-run the html-validate-ember skill on this Ember codebase"` |
| Mark a violation as a false positive | `"mark row N as a false positive"` |
| Finish and clean up before a PR | `"finish audit"` / `"wrap up"` / `"clean up for PR"` |

When marking a false positive, a written reason is required. The entry is saved to `.html-validate-fps.json` and matched by rule ID, file path, and line number on all future runs. If the line number shifts due to template edits, the entry becomes stale and will be flagged in the next report.

---

## Files Produced

| File | Description | Commit to source? |
|---|---|---|
| `<project-name>-html-validate.md` | Developer-mode validation report for this run | No — remove before PR |
| `<project-name>-html-validate_01.md`, `_02.md`, … | Developer-mode reruns are numbered sequentially | No — remove before PR |
| `<project-name>-html-validate-jira-chunks.md` | Auditor-mode copy/paste Jira issue chunks | No — generated artifact |
| `<project-name>-html-validate-jira-bulk.csv` | Auditor-mode Jira bulk upload file | No — generated artifact |
| `<project-name>-html-validate-jira-issue-NNN.md` | Per-issue Jira files (one file per grouped issue chunk) | No — generated artifact |
| `.html-validate-fps.json` | False positive registry | **Yes** — project config, commit with code changes |

The clean-up command removes generated report/artifact files only. The false positive registry is never deleted by clean-up.
