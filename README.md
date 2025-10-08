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

## Test Dependencies

The repository includes sample dependency files for testing:

- **package.json**: Node.js dependencies (npm)
- **requirements.txt**: Python dependencies (pip)
- **go.mod**: Go dependencies

These files contain various packages with different licenses and potential vulnerabilities to trigger the Dependency Review Action.

## Usage

To test the Dependency Review Action:

1. Create a pull request with changes to any of the dependency files
2. The workflows will automatically run and report findings
3. Check the PR comments for detailed reports from the Dependency Review Action

## Configuration Options Tested

The workflows test the following Dependency Review Action configuration options:

- `fail-on-severity`: Set vulnerability severity threshold (low, moderate, high, critical)
- `allow-licenses`: Specify allowed licenses (mutually exclusive with deny-licenses)
- `deny-licenses`: Specify denied licenses (mutually exclusive with allow-licenses)
- `comment-summary-in-pr`: Post review summaries as PR comments
- `vulnerability-check`: Enable/disable vulnerability checking
- `deny-ghsas`: Deny specific GitHub Security Advisories

## Notes

- The `allow-licenses` and `deny-licenses` options are mutually exclusive and cannot be used together in the same workflow
- Workflows trigger on pull requests to the `main` branch
- Some workflows can also be triggered manually via `workflow_dispatch`

## Resources

- [Dependency Review Action on GitHub Marketplace](https://github.com/marketplace/actions/dependency-review)
- [Dependency Review Action Documentation](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/configuring-the-dependency-review-action)