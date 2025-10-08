# Canon A Operations Runbook

This runbook provides step-by-step operational procedures for managing the Canon A cross-attestation repository.

## Table of Contents

1. [System Overview](#system-overview)
2. [Daily Operations](#daily-operations)
3. [Attestation Procedures](#attestation-procedures)
4. [Troubleshooting](#troubleshooting)
5. [Maintenance Tasks](#maintenance-tasks)
6. [Emergency Procedures](#emergency-procedures)

## System Overview

### Architecture Components

- **Repository:** QuietWire-Civic-AI/canon-a
- **Companion:** QuietWire-Civic-AI/canon-b
- **Workflows:** Attestation automation and verification
- **Storage:** JSON attestations with signature files

### Key Directories

```
canon/
├── claims/           # Self-claims and assertions
└── attestations/     # Cross-attestations about canon-b
```

### Workflow Components

- `attest.yml`: Automated attestation creation
- `verify-attestation.yml`: Attestation validation

## Daily Operations

### Morning Checklist

1. **Check Workflow Status**
   ```bash
   # Review recent workflow runs
   gh run list --repo QuietWire-Civic-AI/canon-a --limit 10
   ```

2. **Review Open Issues**
   ```bash
   # Check for attestation requests
   gh issue list --repo QuietWire-Civic-AI/canon-a --state open
   ```

3. **Verify Repository Health**
   ```bash
   # Check for pending PRs
   gh pr list --repo QuietWire-Civic-AI/canon-a --state open
   ```

4. **Monitor Attestation Directory**
   ```bash
   # Check new attestations
   git log --oneline --since="24 hours ago" -- canon/attestations/
   ```

### Routine Monitoring

#### Workflow Execution Health

```bash
# Check recent workflow failures
gh run list --repo QuietWire-Civic-AI/canon-a --status failure --limit 5

# Get detailed logs for failed runs
gh run view <run-id> --log
```

#### Attestation Integrity Check

```bash
# Validate JSON format for all attestations
for file in canon/attestations/*.json; do
  echo "Checking $file"
  jq empty "$file" || echo "INVALID JSON: $file"
done

# Check for missing signature files
for json_file in canon/attestations/*.json; do
  sig_file="${json_file%%.json}__sig.json"
  [ -f "$sig_file" ] || echo "MISSING SIGNATURE: $sig_file"
done
```

## Attestation Procedures

### Manual Attestation Request

1. **Open Issue**
   ```bash
   gh issue create --repo QuietWire-Civic-AI/canon-a \
     --title "Attestation Request" \
     --body "Requesting attestation of canon-b repository"
   ```

2. **Submit Attestation Command**
   ```bash
   # Comment on the issue
   gh issue comment <issue-number> --repo QuietWire-Civic-AI/canon-a \
     --body "/attest QuietWire-Civic-AI/canon-b \"Canon A\""
   ```

3. **Monitor Workflow Execution**
   ```bash
   # Watch for new workflow runs
   gh run watch --repo QuietWire-Civic-AI/canon-a
   ```

4. **Verify PR Creation**
   ```bash
   # Check for new PRs in target repository
   gh pr list --repo QuietWire-Civic-AI/canon-b --state open
   ```

### Attestation Validation

#### JSON Schema Validation

```bash
# Validate attestation structure
jq -e '{
  attestor_repo: .attestor_repo,
  attestor_companion: .attestor_companion,
  subject_repo: .subject_repo,
  subject_commit: .subject_commit,
  claim_hash: .claim_hash,
  assertion: .assertion,
  timestamp_utc: .timestamp_utc,
  schema_version: .schema_version
}' canon/attestations/latest.json
```

#### Signature File Validation

```bash
# Validate signature file structure
jq -e '{
  signed_by: .signed_by,
  signature_type: .signature_type,
  signature_b64: .signature_b64
}' canon/attestations/latest__sig.json
```

### Cross-Repository Verification

```bash
# Verify claim hash matches target repository
TARGET_CLAIM=$(curl -s "https://raw.githubusercontent.com/QuietWire-Civic-AI/canon-b/main/canon/claims/claim-B-0001.txt")
EXPECTED_HASH=$(echo -n "$TARGET_CLAIM" | sha256sum | cut -d' ' -f1)
ATTESTATION_HASH=$(jq -r '.claim_hash' canon/attestations/latest.json)

if [ "$EXPECTED_HASH" = "$ATTESTATION_HASH" ]; then
  echo "✅ Claim hash verification passed"
else
  echo "❌ Claim hash mismatch: expected $EXPECTED_HASH, got $ATTESTATION_HASH"
fi
```

## Troubleshooting

### Common Issues

#### Workflow Failures

**Issue:** Attestation workflow fails with "User not authorized"
```bash
# Check allowlist in workflow file
grep -A 10 "allowlist" .github/workflows/attest.yml

# Verify user is in authorized list
# Update allowlist if necessary
```

**Issue:** "Target repository not found"
```bash
# Verify repository name format
echo "Repository should be: QuietWire-Civic-AI/canon-b"

# Check repository accessibility
gh repo view QuietWire-Civic-AI/canon-b
```

**Issue:** "Token permissions insufficient"
```bash
# Verify PAT scopes in repository secrets
# Required: Contents (RW), Issues (RW), Pull Requests (RW)
# Update token if necessary
```

#### JSON Validation Errors

**Issue:** Invalid JSON format
```bash
# Find and fix JSON syntax errors
jq . canon/attestations/problematic.json

# Common fixes:
# - Missing trailing commas
# - Unescaped quotes in strings
# - Invalid UTF-8 characters
```

**Issue:** Missing required fields
```bash
# Check for all required schema fields
REQUIRED_FIELDS=("attestor_repo" "subject_repo" "subject_commit" "claim_hash" "timestamp_utc")
for field in "${REQUIRED_FIELDS[@]}"; do
  if ! jq -e ".$field" canon/attestations/problematic.json >/dev/null; then
    echo "Missing field: $field"
  fi
done
```

### Diagnostic Commands

#### Repository State

```bash
# Check repository status
git status
git log --oneline -n 10

# Verify branch protection
gh api repos/QuietWire-Civic-AI/canon-a/branches/main/protection

# Check repository secrets
gh secret list --repo QuietWire-Civic-AI/canon-a
```

#### Workflow Debugging

```bash
# Get workflow run details
gh run view <run-id> --verbose

# Download workflow logs
gh run download <run-id>

# Check workflow file syntax
yq eval '.jobs' .github/workflows/attest.yml
```

## Maintenance Tasks

### Weekly Tasks

1. **Review Attestation History**
   ```bash
   # Generate attestation summary
   echo "Attestations created this week:"
   git log --since="1 week ago" --pretty="%h %s" -- canon/attestations/
   ```

2. **Validate Repository Integrity**
   ```bash
   # Run comprehensive validation
   ./scripts/validate-repo.sh
   ```

3. **Update Documentation**
   ```bash
   # Check for outdated references
   grep -r "TODO\|FIXME\|XXX" *.md
   ```

### Monthly Tasks

1. **Token Rotation** (if applicable)
   ```bash
   # Generate new PAT and update repository secrets
   gh secret set BOT_TOKEN --repo QuietWire-Civic-AI/canon-a
   gh secret set BOT_TOKEN --repo QuietWire-Civic-AI/canon-b
   ```

2. **Security Review**
   ```bash
   # Review allowlist configuration
   grep -A 20 "allowlist" .github/workflows/attest.yml
   
   # Check for unauthorized access patterns
   gh api repos/QuietWire-Civic-AI/canon-a/events --paginate
   ```

3. **Performance Analysis**
   ```bash
   # Analyze workflow execution times
   gh run list --json createdAt,conclusion,displayTitle,durationMs \
     --jq '.[] | select(.conclusion == "success") | [.displayTitle, .durationMs]'
   ```

## Emergency Procedures

### Security Incident Response

1. **Immediate Actions**
   ```bash
   # Disable workflows if compromised
   gh workflow disable attest.yml --repo QuietWire-Civic-AI/canon-a
   
   # Revoke and regenerate PAT
   # Update repository secrets immediately
   ```

2. **Investigation**
   ```bash
   # Review recent activity
   gh api repos/QuietWire-Civic-AI/canon-a/events --paginate | \
     jq '.[] | select(.created_at > "2025-10-06T00:00:00Z")'
   
   # Check for unauthorized commits
   git log --since="24 hours ago" --all --graph --pretty=fuller
   ```

3. **Recovery**
   ```bash
   # Re-enable workflows after securing
   gh workflow enable attest.yml --repo QuietWire-Civic-AI/canon-a
   
   # Validate system integrity
   ./scripts/full-validation.sh
   ```

### Data Recovery

**Corrupted Attestation Files**
```bash
# Restore from Git history
git checkout HEAD~1 -- canon/attestations/corrupted-file.json
git checkout HEAD~1 -- canon/attestations/corrupted-file__sig.json

# Validate restored files
jq empty canon/attestations/corrupted-file.json
jq empty canon/attestations/corrupted-file__sig.json
```

**Repository Corruption**
```bash
# Clone fresh copy
git clone https://github.com/QuietWire-Civic-AI/canon-a.git canon-a-backup

# Compare and merge necessary changes
diff -r canon-a canon-a-backup
```

---

*Runbook Version: 1.0*  
*Last Updated: 2025-10-06*  
*Next Review: 2025-11-06*
