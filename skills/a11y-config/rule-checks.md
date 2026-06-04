# Rule checks

This supports the a11y-config skill so that the AI does not need to spend tokens looking up the current rulesets on a github repo. This also means we can consistently apply rules across products.

## Template Lint

These a11y rules should not be turned off in any template-lint configuration file.

-`link-href-attributes`
-`no-abstract-roles`
-`no-accesskey-attribute`
-`no-aria-hidden-body`
-`no-aria-unsupported-elements`
-`no-autofocus-attribute`
-`no-duplicate-attributes`
-`no-duplicate-id`
-`no-duplicate-landmark-elements`
-`no-empty-headings`
-`no-heading-inside-button`
-`no-invalid-aria-attributes`
-`no-invalid-interactive`
-`no-invalid-link-text`
-`no-invalid-link-title`
-`no-invalid-meta`
-`no-invalid-role`
-`no-nested-interactive`
-`no-nested-landmark`
-`no-obsolete-elements`
-`no-pointer-down-event-binding`
-`no-positive-tabindex`
-`no-redundant-role`
-`no-scope-outside-table-headings`
-`no-unsupported-role-attributes`
-`no-whitespace-for-layout`
-`no-whitespace-within-word`
-`require-aria-activedescendant-tabindex`
-`require-context-role`
-`require-iframe-title`
-`require-input-label`
-`require-lang-attribute`
-`require-mandatory-role-attributes`
-`require-media-caption`
-`require-presentational-children`
-`require-valid-alt-text`
-`table-groups`

## HTML Validate

These rules should be turned off:

- `wcag/h30`
- `wcag/h32`
- `wcag/h36`
- `wcag/h37`
- `wcag/h63`
- `wcag/h67`
- `wcag/h71`
