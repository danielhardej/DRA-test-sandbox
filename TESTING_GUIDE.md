# Testing Guide for Dependency Review Action

This guide explains how to use this repository to test the Dependency Review Action.

## Quick Start

1. **Fork this repository** to your own GitHub account
2. **Create a new branch** for your test
3. **Make changes** to dependency files (package.json, requirements.txt, go.mod)
4. **Open a Pull Request** to the main branch
5. **Review the results** in the PR checks and comments

## Test Scenarios

### Testing Vulnerability Scanning

To test vulnerability scanning:

1. Add a package with known vulnerabilities to one of the dependency files
2. For example, add an old version of a package to `package.json`:
   ```json
   "lodash": "4.17.10"
   ```
3. Create a PR and observe the Dependency Review Action detecting vulnerabilities

### Testing License Compliance

To test the allow-licenses feature:

1. Add a package with a license not in the allow-list (e.g., GPL)
2. The workflow `.github/workflows/dependency-review-allow-licenses.yml` will fail
3. Check the PR comments for details about the license violation

To test the deny-licenses feature:

1. Add a package with a denied license (e.g., GPL-3.0)
2. The workflow `.github/workflows/dependency-review-deny-licenses.yml` will fail
3. Check the PR comments for details

### Testing Action Vulnerability Scanning

The workflows in this repository use various GitHub Actions from the marketplace. When you:

1. Modify the workflow files to use different action versions
2. The Dependency Review Action will scan those actions for vulnerabilities
3. This tests whether the action can detect vulnerable marketplace actions

### Testing Different Severity Levels

Different workflows test different severity thresholds:

- **Low**: `.github/workflows/dependency-review-vulnerability.yml`
- **Moderate**: `.github/workflows/dependency-review-comprehensive.yml` (job: vulnerability-scanning)
- **High**: `.github/workflows/dependency-review-comprehensive.yml` (job: combined-checks)
- **Critical**: `.github/workflows/dependency-review-advanced.yml` (job: critical-only)

## Expected Behaviors

### When PR Introduces Vulnerabilities

- Workflows configured with `fail-on-severity` will fail if vulnerabilities meet or exceed the threshold
- PR comments will include:
  - A summary of detected vulnerabilities
  - Severity levels
  - Links to security advisories (GHSA)
  - Remediation suggestions

### When PR Introduces License Violations

- Workflows with `allow-licenses` will fail if a package has a license not in the allow-list
- Workflows with `deny-licenses` will fail if a package has a license in the deny-list
- PR comments will include:
  - List of packages with problematic licenses
  - The detected license for each package
  - Package URLs for reference

### When No Issues Are Detected

- All workflows will pass
- Some workflows may still post comments (depending on `comment-summary-in-pr` setting)
- The comments will indicate no vulnerabilities or license issues were found

## Manual Testing Workflow

1. **Create a test branch**:
   ```bash
   git checkout -b test/add-vulnerable-dependency
   ```

2. **Modify a dependency file**:
   ```bash
   # Add a vulnerable package to package.json
   echo 'Add your changes here'
   ```

3. **Commit and push**:
   ```bash
   git add .
   git commit -m "Test: Add vulnerable dependency"
   git push origin test/add-vulnerable-dependency
   ```

4. **Create Pull Request**:
   - Go to GitHub and create a PR from your test branch to main
   - Wait for the workflows to run
   - Review the results in the Checks tab and PR comments

5. **Verify Results**:
   - Check that the Dependency Review Action detected the issues
   - Review the PR comments for detailed information
   - Verify that appropriate workflows failed or passed as expected

## Troubleshooting

### Workflows Not Running

- Ensure workflows are enabled in your repository settings
- Check that the PR targets the `main` branch
- Verify you have the necessary permissions

### No Comments Posted

- Check the workflow configuration for `comment-summary-in-pr` setting
- Ensure the workflow has `pull-requests: write` permission
- Verify the token has appropriate permissions

### False Positives/Negatives

- Review the GitHub Security Advisory Database for the latest vulnerability information
- Check if the package version is actually vulnerable
- Verify license information in the package's repository

## Advanced Testing

### Testing External Config File

Modify `.github/dependency-review-config.yml` and test the workflow that uses `config-file` parameter:

```yaml
# .github/dependency-review-config.yml
fail_on_severity: 'high'
allow_licenses:
  - 'MIT'
  - 'Apache-2.0'
```

### Testing Custom Scopes

Modify `package.json` to test different dependency scopes:

```json
{
  "dependencies": { /* runtime dependencies */ },
  "devDependencies": { /* development dependencies */ },
  "peerDependencies": { /* peer dependencies */ }
}
```

### Testing Package Exclusions

Use the `deny-packages` configuration to exclude specific packages:

```yaml
deny-packages: |
  pkg:npm/test-package@*
  pkg:pypi/dev-tool@*
```

## Best Practices

1. **Always review the PR comments** from the Dependency Review Action
2. **Address vulnerabilities promptly** by updating to patched versions
3. **Maintain license compliance** by reviewing new dependencies
4. **Keep actions up to date** by using the latest stable versions
5. **Use dependabot** to automate dependency updates
6. **Test in isolation** by creating separate PRs for each test scenario

## Resources

- [Dependency Review Action Documentation](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/configuring-the-dependency-review-action)
- [GitHub Security Advisories](https://github.com/advisories)
- [SPDX License List](https://spdx.org/licenses/)
