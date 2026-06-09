# Jira Label Generation (Auditor Mode)

Apply labels to every Jira issue (chunk content, bulk CSV row, and per-issue file) in this order:

1. `a11y-audit`
2. Exactly one product label:
   - Boundary Enterprise → `a11y-audit:boundary-enterprise`
   - Consul Enterprise → `a11y-audit:consul-enterprise`
   - HCP Boundary → `a11y-audit:hcp-boundary`
   - HCP Platform → `a11y-audit:hcp-plat`
   - HCP Vault → `a11y-audit:hcp-vault`
   - HCP Vault Secrets → `a11y-audit:hvs`
   - HCP Waypoint → `a11y-audit:hcp-waypoint`
   - Nomad → `a11y-audit:nomad`
   - Packer → `a11y-audit:packer`
   - Terraform → `a11y-audit:hcp-tf`
   - Terraform Enterprise → `a11y-audit:tfe`
   - Vault Enterprise → `a11y-audit:ve`
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

## other instructions

- Do not emit alternative severity label namespaces.
- Keep all labels lowercase.
- If the user provides a product name not listed above, infer the closest mapped product label from the existing product lists in #2.
- Multiple WCAG Succeess Criteria labels may apply
