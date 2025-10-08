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

To test vulnerability scanning, you need to add or update packages to versions with known vulnerabilities. Here are specific examples for each dependency file:

#### Example 1: Testing with package.json (Node.js/npm)

Update your `package.json` to include vulnerable versions:

```json
{
  "dependencies": {
    "lodash": "4.17.10",
    "minimist": "1.2.5",
    "axios": "0.21.1",
    "express": "4.16.0"
  }
}
```

**What this tests:**
- `lodash 4.17.10` - Has prototype pollution vulnerabilities (CVE-2019-10744, CVE-2020-8203)
- `minimist 1.2.5` - Has prototype pollution vulnerability (CVE-2021-44906)
- `axios 0.21.1` - Has SSRF vulnerability (CVE-2021-3749)
- `express 4.16.0` - Has open redirect vulnerability (CVE-2018-3717)

#### Example 2: Testing with requirements.txt (Python)

Add or update to vulnerable Python package versions:

```
django==2.2.0
flask==0.12.2
requests==2.6.0
pyyaml==5.3.1
Pillow==8.1.0
```

**What this tests:**
- `django 2.2.0` - Multiple SQL injection and XSS vulnerabilities
- `flask 0.12.2` - Has denial of service vulnerability
- `requests 2.6.0` - Multiple security issues in older versions
- `pyyaml 5.3.1` - Arbitrary code execution vulnerability (CVE-2020-14343)
- `Pillow 8.1.0` - Multiple image processing vulnerabilities

#### Example 3: Testing with go.mod (Go modules)

Update to older vulnerable versions in `go.mod`:

```go
module github.com/actions-testing-danielhardej/DRA-test-sandbox

go 1.21

require (
	github.com/gin-gonic/gin v1.6.0
	github.com/gorilla/mux v1.7.0
	golang.org/x/crypto v0.0.0-20190308221718-c2843e01d9a2
	gopkg.in/yaml.v2 v2.2.2
)
```

**What this tests:**
- `gin-gonic/gin v1.6.0` - May have older vulnerabilities
- `gorilla/mux v1.7.0` - Older version that may have security issues
- `golang.org/x/crypto` (old version) - May have cryptographic vulnerabilities
- `gopkg.in/yaml.v2 v2.2.2` - Older YAML parser with potential issues

After making these changes, create a PR and observe the Dependency Review Action detecting vulnerabilities

### Testing License Compliance

The Dependency Review Action can detect license violations based on allow-lists and deny-lists. Here are specific examples:

#### Example 1: Testing allow-licenses (Packages that should FAIL)

Add packages with licenses NOT in the allow-list (MIT, Apache-2.0, BSD, ISC) to `package.json`:

```json
{
  "dependencies": {
    "readline-sync": "^1.4.10",
    "strongloop": "^6.0.5"
  }
}
```

**What this tests:**
- `readline-sync` - Uses GPL-3.0 license (not in allow-list, will trigger warning)
- `strongloop` - Uses Artistic-2.0 license (not in allow-list, will trigger warning)

For Python, add packages with non-allowed licenses to `requirements.txt`:

```
readline==6.2.4.1
gplearn==0.4.2
```

**What this tests:**
- `readline` - GPL license (not allowed)
- `gplearn` - GPL-3.0 license (not allowed)

#### Example 2: Testing deny-licenses (Packages that should FAIL)

Add packages with GPL or other copyleft licenses to `package.json`:

```json
{
  "dependencies": {
    "node-gpl": "^1.0.0",
    "gpl3-library": "^1.0.0"
  }
}
```

For Python `requirements.txt`:

```
pylint==2.0.0
gpl-package==1.0.0
```

**What this tests:**
- Packages with GPL-2.0, GPL-3.0, LGPL, or AGPL licenses
- The deny-licenses workflow will fail and report these violations

#### Example 3: Testing with safe licenses (Packages that should PASS)

To verify that properly licensed packages pass, use these examples:

In `package.json`:
```json
{
  "dependencies": {
    "express": "^4.18.0",
    "react": "^18.2.0",
    "lodash": "^4.17.21"
  }
}
```

**What this tests:**
- All have MIT licenses
- Should pass both allow-licenses and deny-licenses checks

In `requirements.txt`:
```
requests==2.31.0
flask==3.0.0
numpy==1.24.0
```

**What this tests:**
- `requests` - Apache-2.0 license
- `flask` - BSD license
- `numpy` - BSD license
- All should pass license checks

#### Example 4: Testing allow-dependencies-licenses with deny-licenses (Exception Override)

The `allow-dependencies-licenses` option allows you to exempt specific packages from license checks, even when they have licenses that would normally be denied. This is useful when you need to make an exception for a particular dependency.

**Scenario:** You want to deny GPL licenses globally, but you need to allow a specific package that has a GPL license.

Create a custom workflow file `.github/workflows/test-license-exception.yml`:

```yaml
name: 'Test License Exception'

on:
  pull_request:
    branches: [ main ]

permissions:
  contents: read
  pull-requests: write

jobs:
  test-license-exception:
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout Repository'
        uses: actions/checkout@v4
        
      - name: 'Dependency Review with License Exception'
        uses: actions/dependency-review-action@v4
        with:
          # Deny GPL licenses globally
          deny-licenses: GPL-2.0, GPL-3.0, LGPL-2.0, LGPL-3.0
          # But allow specific packages with GPL licenses
          allow-dependencies-licenses: 'pkg:npm/readline-sync, pkg:pypi/gplearn'
          comment-summary-in-pr: always
```

Then add GPL-licensed packages to your `package.json`:

```json
{
  "name": "dra-test-sandbox",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.0",
    "readline-sync": "^1.4.10"
  }
}
```

**What this tests:**
- `readline-sync` has a GPL-3.0 license, which is in the `deny-licenses` list
- However, `pkg:npm/readline-sync` is listed in `allow-dependencies-licenses`
- **Expected result:** The workflow should **PASS** because the specific package exemption overrides the global deny-licenses rule
- This demonstrates that `allow-dependencies-licenses` acts as an override for specific packages

**Comparison - Without the exception:**

If you remove `readline-sync` from the `allow-dependencies-licenses` list:

```yaml
with:
  deny-licenses: GPL-2.0, GPL-3.0, LGPL-2.0, LGPL-3.0
  # No allow-dependencies-licenses specified
  comment-summary-in-pr: always
```

**Expected result:** The workflow will **FAIL** because `readline-sync` has a GPL-3.0 license which is denied.

**Key takeaway:** `allow-dependencies-licenses` provides fine-grained control, allowing you to create exceptions for specific packages even when they have globally denied licenses. This is useful for legacy dependencies or required packages where you've accepted the license risk.

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

## Complete Testing Examples

Here are complete, copy-paste ready examples for testing different scenarios:

### Scenario 1: Test Vulnerability Detection (High Severity)

**Goal:** Trigger high severity vulnerability warnings

**File to modify:** `package.json`

Replace the entire dependencies section with:
```json
{
  "name": "dra-test-sandbox",
  "version": "1.0.0",
  "description": "Test repository for Dependency Review Action",
  "dependencies": {
    "lodash": "4.17.10",
    "minimist": "1.2.5",
    "axios": "0.21.1"
  }
}
```

**Expected result:** Dependency Review Action will detect multiple high severity vulnerabilities and fail the workflow.

### Scenario 2: Test License Violations (GPL License)

**Goal:** Trigger license violation for GPL-licensed packages

**File to modify:** `package.json`

Add a GPL-licensed package:
```json
{
  "name": "dra-test-sandbox",
  "version": "1.0.0",
  "description": "Test repository for Dependency Review Action",
  "dependencies": {
    "express": "^4.18.0",
    "gpl-library": "^1.0.0"
  }
}
```

**Expected result:** The deny-licenses workflow will fail because GPL is in the deny-list.

### Scenario 3: Test Multiple Vulnerabilities Across Ecosystems

**Goal:** Test vulnerability detection in all three dependency files

**Modify all three files:**

`package.json`:
```json
{
  "name": "dra-test-sandbox",
  "version": "1.0.0",
  "dependencies": {
    "lodash": "4.17.10",
    "express": "4.16.0"
  }
}
```

`requirements.txt`:
```
django==2.2.0
flask==0.12.2
pyyaml==5.3.1
```

`go.mod`:
```go
module github.com/actions-testing-danielhardej/DRA-test-sandbox

go 1.21

require (
	github.com/gin-gonic/gin v1.6.0
	gopkg.in/yaml.v2 v2.2.2
)
```

**Expected result:** Dependency Review Action will detect vulnerabilities across all three ecosystems.

### Scenario 4: Test Severity Thresholds

**Goal:** Test that only critical vulnerabilities trigger failures

**File to modify:** `package.json`

```json
{
  "name": "dra-test-sandbox",
  "version": "1.0.0",
  "dependencies": {
    "lodash": "4.17.15"
  }
}
```

**Expected result:** 
- Workflows with `fail-on-severity: low` or `moderate` will fail
- Workflows with `fail-on-severity: critical` may pass if the vulnerability is not critical

### Scenario 5: Test Clean State (No Issues)

**Goal:** Verify that secure, properly-licensed dependencies pass all checks

**File to modify:** `package.json`

```json
{
  "name": "dra-test-sandbox",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.19.0",
    "lodash": "^4.17.21",
    "axios": "^1.6.5"
  }
}
```

**Expected result:** All workflows should pass with green checks.

### Scenario 6: Test License Exception Override (allow-dependencies-licenses with deny-licenses)

**Goal:** Demonstrate that `allow-dependencies-licenses` can override `deny-licenses` for specific packages

**Create a new workflow file:** `.github/workflows/test-license-exception.yml`

```yaml
name: 'Test License Exception Override'

on:
  pull_request:
    branches: [ main ]

permissions:
  contents: read
  pull-requests: write

jobs:
  test-license-exception:
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout Repository'
        uses: actions/checkout@v4
        
      - name: 'Dependency Review with License Exception'
        uses: actions/dependency-review-action@v4
        with:
          deny-licenses: GPL-2.0, GPL-3.0, LGPL-2.0, LGPL-3.0
          allow-dependencies-licenses: 'pkg:npm/readline-sync'
          comment-summary-in-pr: always
```

**File to modify:** `package.json`

```json
{
  "name": "dra-test-sandbox",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.0",
    "readline-sync": "^1.4.10"
  }
}
```

**Expected result:** 
- The workflow will **PASS** even though `readline-sync` has a GPL-3.0 license (which is in the deny-licenses list)
- This is because `pkg:npm/readline-sync` is explicitly listed in `allow-dependencies-licenses`
- The `allow-dependencies-licenses` acts as an exception/override for that specific package

**To compare:** Remove `readline-sync` from the `allow-dependencies-licenses` parameter and the workflow will **FAIL** with a license violation error.

**What this demonstrates:**
- Fine-grained license control for specific packages
- Ability to make exceptions for required dependencies with otherwise denied licenses
- Priority: `allow-dependencies-licenses` overrides global `deny-licenses` rules for specified packages

## Manual Testing Workflow

1. **Create a test branch**:
   ```bash
   git checkout -b test/add-vulnerable-dependency
   ```

2. **Modify a dependency file**:
   ```bash
   # Example: Add a vulnerable version of lodash to package.json
   # Edit package.json manually or use jq:
   jq '.dependencies.lodash = "4.17.10"' package.json > tmp.json && mv tmp.json package.json
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

### Quick Reference: Known Vulnerable Package Versions

Use these versions to quickly test vulnerability detection:

#### Node.js (package.json)
| Package | Vulnerable Version | Known Issue |
|---------|-------------------|-------------|
| lodash | 4.17.10 | Prototype pollution |
| minimist | 1.2.5 | Prototype pollution |
| axios | 0.21.1 | SSRF vulnerability |
| express | 4.16.0 | Open redirect |
| ws | 7.4.5 | ReDoS vulnerability |
| node-fetch | 2.6.6 | Information exposure |

#### Python (requirements.txt)
| Package | Vulnerable Version | Known Issue |
|---------|-------------------|-------------|
| django | 2.2.0 | SQL injection |
| flask | 0.12.2 | DoS vulnerability |
| pyyaml | 5.3.1 | Code execution |
| Pillow | 8.1.0 | Image processing issues |
| cryptography | 2.2 | Cryptographic issues |
| urllib3 | 1.24.1 | CRLF injection |

#### Go (go.mod)
| Package | Vulnerable Version | Known Issue |
|---------|-------------------|-------------|
| github.com/gin-gonic/gin | v1.6.0 | Various issues |
| gopkg.in/yaml.v2 | v2.2.2 | Parsing issues |
| golang.org/x/crypto | v0.0.0-20190308221718 | Crypto vulnerabilities |

### Quick Reference: License Examples

#### Packages with GPL licenses (will fail deny-licenses check)
- **npm:** `readline-sync`, `node-gpl`
- **pip:** `readline`, `gplearn`

#### Packages with permissive licenses (will pass both checks)
- **npm:** `express` (MIT), `react` (MIT), `axios` (MIT)
- **pip:** `requests` (Apache-2.0), `flask` (BSD), `numpy` (BSD)
- **go:** `gin` (MIT), `gorilla/mux` (BSD)

### Advanced Testing

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
