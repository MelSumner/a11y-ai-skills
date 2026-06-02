# Jira Label Generation (Auditor Mode)

Apply labels to every Jira issue (chunk content, bulk CSV row, and per-issue file) in this order:

1. `a11y-audit`
2. Exactly one product label:
   - HCP Boundary → `a11y-audit:hcp-boundary`
   - Boundary Enterprise → `a11y-audit:boundary-enterprise`
   - HCP Vault → `a11y-audit:hcp-vault`
   - Vault Enterprise → `a11y-audit:ve`
   - Terraform Enterprise → `a11y-audit:tfe`
   - Terraform → `a11y-audit:hcp-tf`
   - HCP Vault Secrets → `a11y-audit:hvs`
   - HCP Waypoint → `a11y-audit:hcp-waypoint`
   - Consul Enterprise → `a11y-audit:consul-enterprise`
   - If the auditor provides a product name not listed above, infer the closest mapped product label from this list.
3. One or more WCAG Success Criteria labels, one per mapped criterion, in dasherized format:
   - `1.1.1` → `wcag-1-1-1`
   - `1.3.1` → `wcag-1-3-1`
4. Exactly one severity label:
   - Critical → `a11y-sev:critical`
   - Serious → `a11y-sev:serious`
   - Moderate → `a11y-sev:moderate`
   - Minor → `a11y-sev:minor`
5. Optional WCAG Failure label when known, in dasherized format:
   - `F1` → `wcag-f1`

Do not emit alternative severity label namespaces. Keep all labels lowercase, deduplicated, and formatted for direct Jira application.
