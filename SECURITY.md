# Security Policy

## Overview

This document outlines the security policies and procedures for the Canon A cross-attestation repository within the GTFO_2 system.

## Token Scopes and Access Control

### GitHub Personal Access Token (PAT) Requirements

The Actions workflows require a fine-grained Personal Access Token with the following scopes:

**Repository Permissions (for both canon-a and canon-b):**
- **Contents:** Read and Write
- **Issues:** Read and Write  
- **Pull Requests:** Read and Write
- **Metadata:** Read
- **Actions:** Read (if workflow management needed)

**Account Permissions:**
- None required

### Allowlist Configuration

The attestation workflow restricts execution to authorized actors only:

```yaml
# Authorized attestation requesters
allowlist:
  - "AshrafAlhajj"     # Primary architect
  - "Raasid"           # Observer/Agent
  # Add new authorized users here
```

**Adding New Authorized Users:**
1. Update the allowlist in `.github/workflows/attest.yml`
2. Ensure the user has legitimate need for attestation privileges
3. Obtain approval from project stewards
4. Test the configuration in a non-production environment

## Security Hardening Measures

### Workflow Security

1. **Input Validation**
   - All user inputs are validated before processing
   - Repository names must match expected patterns
   - Attestor names are restricted to known companions

2. **Concurrency Control**
   - Workflows include concurrency groups to prevent simultaneous executions
   - Race conditions are mitigated through proper locking

3. **Secrets Management**
   - PAT tokens are stored as repository secrets
   - Secrets are never logged or exposed in workflow outputs
   - Token scope is minimized to required permissions only

### Attestation Security

1. **Cryptographic Integrity**
   - All claims are hashed using SHA-256
   - Commit SHAs provide tamper-evident references
   - Timestamps use RFC3339 format for consistency

2. **Signature Validation**
   - Attestations require both JSON data and signature files
   - Signature files use `__sig.json` suffix for clear identification
   - Each signature stores a SHA-256 digest (hex + base64) of the attestation payload for tamper detection

3. **Branch Protection**
   - Attestation branches use unique naming: `attest/by-${owner}/run-${id}`
   - Branch names include monotonic IDs to prevent conflicts
   - All changes require pull request review

## Incident Response

### Security Incident Types

1. **Unauthorized Attestation Attempts**
   - Monitor workflow logs for failed allowlist checks
   - Review issue comments for suspicious attestation requests
   - Alert project stewards immediately

2. **Token Compromise**
   - Immediately revoke the compromised PAT
   - Generate new token with same scope restrictions
   - Update repository secrets in all affected repos
   - Review recent workflow executions for unauthorized activity

3. **Malformed Attestations**
   - Verify-attestation workflow will reject invalid JSON
   - Review PR diffs for unexpected content
   - Investigate source of malformed data

### Response Procedures

1. **Immediate Actions**
   - Document the incident with timestamps
   - Preserve relevant logs and evidence
   - Notify project stewards via secure channels

2. **Investigation**
   - Analyze workflow logs and Git history
   - Check for unauthorized repository access
   - Review recent changes to sensitive files

3. **Remediation**
   - Address identified vulnerabilities
   - Update security configurations as needed
   - Communicate changes to stakeholders

## Vulnerability Reporting

### Responsible Disclosure

To report security vulnerabilities:

1. **Contact:** Open an issue in this repository with the "security" label
2. **Information:** Include detailed description and reproduction steps
3. **Timeline:** We aim to respond within 48 hours
4. **Coordination:** Work with our team on disclosure timeline

### Security Updates

Security patches and updates will be:
- Applied promptly to address identified vulnerabilities
- Documented in commit messages and release notes
- Communicated to all repository stakeholders

## Compliance and Auditing

### Audit Trail

The system maintains comprehensive audit trails:
- All attestations are permanently recorded in Git history
- Workflow executions are logged in Actions history
- Token usage is tracked through GitHub audit logs

### Regular Reviews

1. **Quarterly Security Reviews**
   - Review access permissions and allowlists
   - Audit token scopes and usage patterns
   - Update security configurations as needed

2. **Attestation Validation**
   - Verify integrity of stored attestations
   - Check for unexpected or malformed entries
   - Validate signature file consistency

## Security Best Practices

### For Repository Maintainers

1. **Token Management**
   - Use fine-grained PATs with minimal required scopes
   - Rotate tokens regularly (recommended: quarterly)
   - Monitor token usage through GitHub audit logs

2. **Workflow Configuration**
   - Keep allowlists current and minimal
   - Review workflow files for security implications
   - Test changes in isolated environments first

3. **Access Control**
   - Limit repository admin access to essential personnel
   - Enable branch protection rules
   - Require signed commits where possible

### For Contributors

1. **Issue Comments**
   - Only use attestation commands when authorized
   - Verify target repository names before requesting attestations
   - Report suspicious activity or requests

2. **Pull Requests**
   - Review attestation PRs carefully before merging
   - Verify JSON schema compliance
   - Check signature file presence and format

---

*Security Policy Version: 1.0*  
*Last Updated: 2025-10-06*  
*Next Review: 2026-01-06*
