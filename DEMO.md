# Canon A Demonstration Guide

This guide provides step-by-step instructions for demonstrating the Canon A cross-attestation system.

## Overview

The demonstration shows how two AI companions (Canon A and Canon B) can establish mutual trust through cryptographic attestations without centralized control.

## Prerequisites

### Required Setup

1. **GitHub Account Access**
   - Access to QuietWire-Civic-AI organization
   - Fine-grained PAT with appropriate scopes
   - Repository secrets configured

2. **Repository Access**
   - canon-a repository: `QuietWire-Civic-AI/canon-a`
   - canon-b repository: `QuietWire-Civic-AI/canon-b`

3. **Local Environment**
   - Git CLI installed
   - GitHub CLI (gh) installed
   - jq for JSON processing

### Verification Steps

```bash
# Verify GitHub CLI authentication
gh auth status

# Check repository access
gh repo view QuietWire-Civic-AI/canon-a
gh repo view QuietWire-Civic-AI/canon-b

# Verify local tools
which git jq
```

## Demonstration Scenarios

### Scenario 1: Basic Cross-Attestation

**Objective:** Show Canon A creating an attestation about Canon B

#### Step 1: Prepare the Environment

```bash
# Clone both repositories
git clone https://github.com/QuietWire-Civic-AI/canon-a.git
git clone https://github.com/QuietWire-Civic-AI/canon-b.git

cd canon-a
```

#### Step 2: Review Current State

```bash
# Show Canon A's self-claims
echo "=== Canon A Claims ==="
cat canon/claims/claim-A-0001.txt

echo "\n=== Current Attestations ==="
ls -la canon/attestations/
```

#### Step 3: Examine Canon B's Claims

```bash
# Review what Canon A will attest about
echo "=== Canon B Claims (Target) ==="
cat ../canon-b/canon/claims/claim-B-0001.txt

# Calculate claim hash
echo "\n=== Claim Hash ==="
sha256sum ../canon-b/canon/claims/claim-B-0001.txt
```

#### Step 4: Create Attestation Request

```bash
# Open an issue for attestation
gh issue create --title "Demo: Attestation Request" \
  --body "Requesting attestation of Canon B for demonstration purposes."

# Get the issue number
ISSUE_NUM=$(gh issue list --limit 1 --json number --jq '.[0].number')
echo "Created issue #$ISSUE_NUM"
```

#### Step 5: Trigger Attestation Workflow

```bash
# Submit attestation command
gh issue comment $ISSUE_NUM \
  --body "/attest QuietWire-Civic-AI/canon-b \"Canon A\""

echo "Attestation request submitted. Monitoring workflow..."
```

#### Step 6: Monitor Workflow Execution

```bash
# Watch for workflow runs
echo "Watching for workflow execution..."
gh run list --limit 5

# Get the latest run ID and monitor
RUN_ID=$(gh run list --limit 1 --json databaseId --jq '.[0].databaseId')
gh run watch $RUN_ID
```

#### Step 7: Verify Attestation Creation

```bash
# Check for new PR in canon-b
echo "Checking for attestation PR in canon-b..."
gh pr list --repo QuietWire-Civic-AI/canon-b --state open

# Get PR details
PR_NUM=$(gh pr list --repo QuietWire-Civic-AI/canon-b --limit 1 --json number --jq '.[0].number')
if [ ! -z "$PR_NUM" ]; then
  echo "Found PR #$PR_NUM"
  gh pr view $PR_NUM --repo QuietWire-Civic-AI/canon-b
fi
```

#### Step 8: Review Attestation Content

```bash
# Show the attestation files in the PR
if [ ! -z "$PR_NUM" ]; then
  echo "=== Attestation Files ==="
  gh pr diff $PR_NUM --repo QuietWire-Civic-AI/canon-b
fi
```

### Scenario 2: Mutual Attestation

**Objective:** Show bidirectional trust establishment

#### Step 1: Complete Canon A → Canon B Attestation

```bash
# Merge the attestation PR in canon-b
if [ ! -z "$PR_NUM" ]; then
  gh pr merge $PR_NUM --repo QuietWire-Civic-AI/canon-b --squash
  echo "Canon A's attestation merged into Canon B"
fi
```

#### Step 2: Request Reverse Attestation

```bash
# Create issue in canon-b for reverse attestation
gh issue create --repo QuietWire-Civic-AI/canon-b \
  --title "Demo: Reverse Attestation Request" \
  --body "Requesting Canon B to attest Canon A"

B_ISSUE_NUM=$(gh issue list --repo QuietWire-Civic-AI/canon-b --limit 1 --json number --jq '.[0].number')
echo "Created issue #$B_ISSUE_NUM in canon-b"
```

#### Step 3: Trigger Reverse Attestation

```bash
# Submit reverse attestation command
gh issue comment $B_ISSUE_NUM --repo QuietWire-Civic-AI/canon-b \
  --body "/attest QuietWire-Civic-AI/canon-a \"Canon B\""

echo "Reverse attestation request submitted"
```

#### Step 4: Monitor and Complete

```bash
# Watch for PR in canon-a
echo "Waiting for reverse attestation PR..."
sleep 30  # Allow workflow time to execute

REVERSE_PR=$(gh pr list --limit 1 --json number --jq '.[0].number')
if [ ! -z "$REVERSE_PR" ]; then
  echo "Found reverse attestation PR #$REVERSE_PR"
  gh pr view $REVERSE_PR
  gh pr merge $REVERSE_PR --squash
fi
```

### Scenario 3: Validation and Verification

**Objective:** Demonstrate attestation integrity checking

#### Step 1: Validate JSON Structure

```bash
# Check all attestation files
echo "=== Validating Attestation JSON ==="
for file in canon/attestations/*.json; do
  if [ -f "$file" ]; then
    echo "Validating $file"
    jq empty "$file" && echo "✅ Valid JSON" || echo "❌ Invalid JSON"
  fi
done
```

#### Step 2: Verify Signature Files

```bash
# Check signature file presence
echo "\n=== Checking Signature Files ==="
for json_file in canon/attestations/*.json; do
  if [ -f "$json_file" ]; then
    sig_file="${json_file%%.json}__sig.json"
    if [ -f "$sig_file" ]; then
      echo "✅ $sig_file exists"
      jq empty "$sig_file" && echo "✅ Valid signature JSON" || echo "❌ Invalid signature JSON"
    else
      echo "❌ Missing signature file: $sig_file"
    fi
  fi
done
```

#### Step 3: Verify Claim Hashes

```bash
# Validate claim hash accuracy
echo "\n=== Verifying Claim Hashes ==="
for json_file in canon/attestations/*.json; do
  if [ -f "$json_file" ]; then
    echo "Checking $json_file"
    
    # Extract subject repo and claim hash from attestation
    SUBJECT_REPO=$(jq -r '.subject_repo' "$json_file")
    CLAIM_HASH=$(jq -r '.claim_hash' "$json_file")
    
    echo "Subject: $SUBJECT_REPO"
    echo "Claimed hash: $CLAIM_HASH"
    
    # Determine claim file based on subject repo
    if [[ "$SUBJECT_REPO" == *"canon-b" ]]; then
      CLAIM_FILE="../canon-b/canon/claims/claim-B-0001.txt"
    else
      CLAIM_FILE="../canon-a/canon/claims/claim-A-0001.txt"
    fi
    
    if [ -f "$CLAIM_FILE" ]; then
      ACTUAL_HASH=$(sha256sum "$CLAIM_FILE" | cut -d' ' -f1)
      echo "Actual hash: $ACTUAL_HASH"
      
      if [ "$CLAIM_HASH" = "$ACTUAL_HASH" ]; then
        echo "✅ Hash verification passed"
      else
        echo "❌ Hash verification failed"
      fi
    else
      echo "❌ Claim file not found: $CLAIM_FILE"
    fi
    echo "---"
  fi
done
```

## Troubleshooting Demo Issues

### Workflow Not Triggering

```bash
# Check if user is in allowlist
grep -A 10 "allowlist" .github/workflows/attest.yml

# Verify issue comment format
echo "Correct format: /attest QuietWire-Civic-AI/canon-b \"Canon A\""

# Check workflow file syntax
yq eval '.jobs' .github/workflows/attest.yml
```

### Token Permission Issues

```bash
# Verify repository secrets
gh secret list

# Test token permissions
gh api repos/QuietWire-Civic-AI/canon-b/contents/README.md
```

### JSON Validation Failures

```bash
# Manual JSON validation
jq . canon/attestations/problematic.json

# Fix common issues:
# - Escape quotes in strings
# - Remove trailing commas
# - Ensure UTF-8 encoding
```

## Presentation Tips

### Key Talking Points

1. **Decentralized Trust**
   - No central authority validates attestations
   - Each companion validates the other's claims
   - Cryptographic hashes ensure integrity

2. **Transparency**
   - All attestations are publicly visible
   - Complete audit trail in Git history
   - JSON format enables programmatic validation

3. **Automation**
   - GitHub Actions automate the attestation process
   - Human oversight through PR review process
   - Scalable to multiple repositories

### Demo Flow Suggestions

1. **Start with Claims** - Show what each companion claims about itself
2. **Show the Gap** - Explain why external validation is needed
3. **Trigger Attestation** - Demonstrate the automated process
4. **Verify Results** - Show the attestation files and validation
5. **Complete the Circle** - Show mutual attestation

### Common Questions

**Q: How is this different from traditional PKI?**
A: This is more transparent and doesn't require a certificate authority. All attestations are public and auditable.

**Q: What prevents malicious attestations?**
A: The allowlist restricts who can create attestations, and all changes go through PR review.

**Q: How does this scale?**
A: Each repository pair can establish mutual attestation. Networks of trust can emerge organically.

**Q: What about real cryptographic signatures?**
A: This demo currently uses SHA-256 digests (encoded as base64) for tamper detection, but the framework can be extended to support SSH, GPG, or other signature types.

---

*Demo Guide Version: 1.0*  
*Last Updated: 2025-10-06*  
*Target Audience: Technical stakeholders, AI researchers, DevOps teams*
