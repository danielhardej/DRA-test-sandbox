# DRA-test-sandbox

Test repository for the GitHub Dependency Review Action. This repository contains workflows and dependency files to test various features of the Dependency Review Action.

## Purpose

This repository is designed to test the following capabilities of the Dependency Review Action:

1. **Vulnerability Scanning**: Detect known security vulnerabilities in dependencies
2. **License Compliance**: Check for license violations using allow-lists and deny-lists
3. **Action Vulnerability Scanning**: Scan GitHub Actions from the marketplace for vulnerabilities

## Workflows

### 1. Dependency Review - Vulnerability Scanning
**File**: `.github/workflows/dependency-review-vulnerability.yml`

Tests the basic vulnerability scanning functionality of the Dependency Review Action. Configured to fail on any vulnerability with severity of `low` or higher.

### 2. Dependency Review - Allow Licenses
**File**: `.github/workflows/dependency-review-allow-licenses.yml`

Tests the `allow-licenses` configuration option. Only dependencies with MIT, Apache-2.0, BSD, or ISC licenses will be allowed.

### 3. Dependency Review - Deny Licenses
**File**: `.github/workflows/dependency-review-deny-licenses.yml`

Tests the `deny-licenses` configuration option. Dependencies with GPL, LGPL, or AGPL licenses will be rejected.

### 4. Test - Actions Vulnerability Scanning
**File**: `.github/workflows/test-actions-vulnerability.yml`

A workflow that uses various GitHub Actions from the marketplace to test whether the Dependency Review Action can scan and report vulnerabilities in Actions themselves.

### 5. Dependency Review - Comprehensive Test
**File**: `.github/workflows/dependency-review-comprehensive.yml`

A comprehensive test workflow that includes multiple jobs testing:
- Basic vulnerability scanning with different severity levels
- License allow-lists
- License deny-lists
- Combined checks with multiple configurations
- Action vulnerability scanning

### 6. Dependency Review - Advanced Configuration
**File**: `.github/workflows/dependency-review-advanced.yml`

Tests advanced and edge-case configurations:
- Critical-only vulnerability scanning
- Dependency scope filtering (runtime, development, etc.)
- External configuration file usage
- Custom base/head ref comparisons
- Strict security combined with license checks
- Package exclusions

### 7. Test - Action Dependencies with Different Versions
**File**: `.github/workflows/test-action-versions.yml`

Tests how the Dependency Review Action handles different action version formats:
- Full SHA references
- Tag-based versions (v4, v5, etc.)
- Pinned versions (v4.0.2)
- Third-party marketplace actions
- Demonstrates action vulnerability scanning across different version formats

## Test Dependencies

The repository includes sample dependency files for testing:

- **package.json**: Node.js dependencies (npm)
- **requirements.txt**: Python dependencies (pip)
- **go.mod**: Go dependencies

These files contain various packages with different licenses and potential vulnerabilities to trigger the Dependency Review Action.

## Configuration Files

- **.github/dependency-review-config.yml**: Sample external configuration file that can be referenced in workflows for centralized configuration management
- **.gitignore**: Prevents build artifacts and dependencies from being committed

## Usage

To test the Dependency Review Action:

1. Create a pull request with changes to any of the dependency files
2. The workflows will automatically run and report findings
3. Check the PR comments for detailed reports from the Dependency Review Action

For detailed testing instructions, see [TESTING_GUIDE.md](TESTING_GUIDE.md).

## Configuration Options Tested

The workflows test the following Dependency Review Action configuration options:

- `fail-on-severity`: Set vulnerability severity threshold (low, moderate, high, critical)
- `allow-licenses`: Specify allowed licenses (mutually exclusive with deny-licenses)
- `deny-licenses`: Specify denied licenses (mutually exclusive with allow-licenses)
- `allow-dependencies-licenses`: Exempt specific packages from license checks (overrides allow-licenses/deny-licenses for listed packages)
- `comment-summary-in-pr`: Post review summaries as PR comments (always, never, on-failure)
- `vulnerability-check`: Enable/disable vulnerability checking
- `license-check`: Enable/disable license checking
- `deny-ghsas`: Deny specific GitHub Security Advisories
- `fail-on-scopes`: Only fail on dependencies in specific scopes (runtime, development, unknown)
- `config-file`: Use an external configuration file
- `base-ref` / `head-ref`: Custom git references for comparison
- `deny-packages`: Exclude specific packages from review
- `retry-on-snapshot-warnings`: Retry if snapshot warnings occur

## Notes

- The `allow-licenses` and `deny-licenses` options are mutually exclusive and cannot be used together in the same workflow
- Workflows trigger on pull requests to the `main` branch
- Some workflows can also be triggered manually via `workflow_dispatch`

## Resources

- [Dependency Review Action on GitHub Marketplace](https://github.com/marketplace/actions/dependency-review)
- [Dependency Review Action Documentation](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/configuring-the-dependency-review-action)