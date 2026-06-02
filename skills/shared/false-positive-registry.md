# False Positive Registry

The false positive registry is stored as a JSON file in the project root. It is created by the skill when the user marks a finding as a false positive and read during preflight/inventory on every subsequent run.

## File Format

```json
{
  "version": 1,
  "entries": [
    {
      "check": "rule-or-criterion-identifier",
      "location": "file-path-or-component-location",
      "issue": "Brief description of the issue",
      "reason": "Human-readable explanation of why this is a false positive",
      "added": "YYYY-MM-DD"
    }
  ]
}
```

## Field Definitions

| Field | Required | Description |
|---|---|---|
| `version` | Yes | Schema version. Always `1` for entries written by this skill. |
| `check` | Yes | The check/rule/criterion identifier exactly as recorded in the finding (e.g., `"ember-template-lint/rule-disabled: no-invalid-interactive"`, `"no-implicit-close"`, `"1.4.3"`). |
| `location` | Yes | The location string exactly as recorded in the finding's location column. Can be a file path, config file entry, component name, or any combination. |
| `issue` | Yes | A short description of the issue. Stored for human readability; not used in matching. |
| `reason` | Yes | A human-readable explanation of why this is a false positive. Required; do not accept an empty reason. |
| `added` | Yes | ISO date (`YYYY-MM-DD`) when the entry was added. |

## Matching Algorithm

A registry entry **matches** a current finding when both of the following are equal (case-insensitive, whitespace-trimmed):

1. `entry.check === finding.check` (or `entry.criterion === finding.criterion`, or `entry.rule === finding.rule`)
2. `entry.location === finding.location` (or `entry.file === finding.file` for line-based matches)

For line-based matches (html-validate), also require:
3. `entry.line === finding.line`

If the location string or line number has changed since the entry was created, the entry will not match and will appear as a **stale entry** in the report. Update the entry's location/line value to match the new location, or unmark and re-mark the finding to refresh it.

## When to Use This Registry

| Approach | Use when |
|---|---|
| Registry entry | The finding is a known false positive that recurs across audit runs and you want a durable, auditable record. Common cases: intentional rule overrides with documented justification, third-party component constraints, rules disabled for a bounded legacy area with a remediation plan, Glint resolution that is correct at runtime. |
| Accepted risk / exception | The finding is a genuine issue that has been formally accepted with documented risk, not a false positive. Use the "accepted exceptions" section of the report for these. |
| Inline suppression | (For html-validate only) You can modify the template and want the suppression colocated with the code. |

## User-Triggered Actions

### Mark as False Positive

When the user says "mark row N as a false positive", "mark finding N as a false positive", or similar:

1. Identify the finding from the current report context using the row number
2. Ask for a reason if not provided (required for every entry)
3. Create or update the registry file in the project root
4. Append the new entry to the `entries` array
5. Set `"added"` to today's date in `YYYY-MM-DD` format
6. Confirm back to the user with the entry details

### Unmark False Positive

To unmark a false positive, remove its entry from the `entries` array in the registry file.
