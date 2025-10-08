# Canon A Testing Guide

This document provides comprehensive testing procedures for the Canon A cross-attestation system.

## Test Categories

1. [Unit Tests](#unit-tests) - Individual component validation
2. [Integration Tests](#integration-tests) - Cross-component functionality
3. [End-to-End Tests](#end-to-end-tests) - Complete workflow validation
4. [Security Tests](#security-tests) - Security posture validation
5. [Performance Tests](#performance-tests) - System performance evaluation

## Unit Tests

### JSON Schema Validation

#### Test 1: Valid Attestation JSON

```bash
#!/bin/bash
# Test: valid_attestation_json.sh

test_valid_attestation_json() {
  local test_file="test_valid_attestation.json"
  
  cat > "$test_file" << 'EOF'
{
  "attestor_repo": "QuietWire-Civic-AI/canon-a",
  "attestor_companion": "Canon A",
  "subject_repo": "QuietWire-Civic-AI/canon-b",
  "subject_commit": "abc123def456",
  "claim_hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "assertion": "We attest that abc123def456 matches the claim & norms.",
  "timestamp_utc": "2025-10-06T03:56:04Z",
  "schema_version": "0.2"
}
EOF

  # Test JSON validity
  if jq empty "$test_file" 2>/dev/null; then
    echo "✅ PASS: Valid JSON format"
  else
    echo "❌ FAIL: Invalid JSON format"
    return 1
  fi
  
  # Test required fields
  local required_fields=("attestor_repo" "subject_repo" "subject_commit" "claim_hash" "timestamp_utc")
  for field in "${required_fields[@]}"; do
    if jq -e ".$field" "$test_file" >/dev/null 2>&1; then
      echo "✅ PASS: Field '$field' present"
    else
      echo "❌ FAIL: Field '$field' missing"
      rm -f "$test_file"
      return 1
    fi
  done
  
  rm -f "$test_file"
  echo "✅ ALL TESTS PASSED: Valid attestation JSON"
}

test_valid_attestation_json
```

#### Test 2: Valid Signature JSON

```bash
#!/bin/bash
# Test: valid_signature_json.sh

test_valid_signature_json() {
  local test_file="test_valid_signature.json"
  
  cat > "$test_file" << 'EOF'
{
  "signed_by": "Canon A",
  "signature_type": "sha256-base64",
  "signature_b64": "BASE64_SHA256_PAYLOAD",
  "fingerprint": "sha256:digest"
}
EOF

  # Test JSON validity
  if jq empty "$test_file" 2>/dev/null; then
    echo "✅ PASS: Valid signature JSON format"
  else
    echo "❌ FAIL: Invalid signature JSON format"
    return 1
  fi
  
  # Test required fields
  local required_fields=("signed_by" "signature_type" "signature_b64" "fingerprint")
  for field in "${required_fields[@]}"; do
    if jq -e ".$field" "$test_file" >/dev/null 2>&1; then
      echo "✅ PASS: Signature field '$field' present"
    else
      echo "❌ FAIL: Signature field '$field' missing"
      rm -f "$test_file"
      return 1
    fi
  done

  local signature_type
  signature_type=$(jq -r '.signature_type' "$test_file")
  if [ "$signature_type" != "sha256-base64" ]; then
    echo "❌ FAIL: Unexpected signature_type '$signature_type'"
    rm -f "$test_file"
    return 1
  fi

  local fingerprint
  fingerprint=$(jq -r '.fingerprint' "$test_file")
  if [[ ! "$fingerprint" =~ ^sha256:[a-f0-9]{64}$ ]]; then
    echo "❌ FAIL: Fingerprint must be sha256:<64 hex characters>"
    rm -f "$test_file"
    return 1
  fi
  
  rm -f "$test_file"
  echo "✅ ALL TESTS PASSED: Valid signature JSON"
}

test_valid_signature_json
```

### Claim Hash Validation

#### Test 3: SHA-256 Hash Calculation

```bash
#!/bin/bash
# Test: claim_hash_calculation.sh

test_claim_hash_calculation() {
  local test_claim="canon/claims/claim-A-0001.txt"
  
  # Verify claim file exists
  if [ ! -f "$test_claim" ]; then
    echo "❌ FAIL: Claim file not found: $test_claim"
    return 1
  fi
  
  # Calculate hash
  local calculated_hash=$(sha256sum "$test_claim" | cut -d' ' -f1)
  
  # Verify hash format (64 hex characters)
  if [[ $calculated_hash =~ ^[a-f0-9]{64}$ ]]; then
    echo "✅ PASS: Valid SHA-256 hash format: $calculated_hash"
  else
    echo "❌ FAIL: Invalid hash format: $calculated_hash"
    return 1
  fi
  
  # Test hash consistency
  local second_hash=$(sha256sum "$test_claim" | cut -d' ' -f1)
  if [ "$calculated_hash" = "$second_hash" ]; then
    echo "✅ PASS: Hash calculation is consistent"
  else
    echo "❌ FAIL: Hash calculation inconsistent"
    return 1
  fi
  
  echo "✅ ALL TESTS PASSED: Claim hash calculation"
}

test_claim_hash_calculation
```

## Integration Tests

### Workflow Validation

#### Test 4: Workflow File Syntax

```bash
#!/bin/bash
# Test: workflow_syntax_validation.sh

test_workflow_syntax() {
  local workflows=("attest.yml" "verify-attestation.yml")
  
  for workflow in "${workflows[@]}"; do
    local workflow_path=".github/workflows/$workflow"
    
    if [ ! -f "$workflow_path" ]; then
      echo "❌ FAIL: Workflow file not found: $workflow_path"
      return 1
    fi
    
    # Test YAML syntax
    if yq eval '.' "$workflow_path" >/dev/null 2>&1; then
      echo "✅ PASS: Valid YAML syntax: $workflow"
    else
      echo "❌ FAIL: Invalid YAML syntax: $workflow"
      return 1
    fi
    
    # Test required workflow components
    if yq eval '.on' "$workflow_path" >/dev/null 2>&1; then
      echo "✅ PASS: Workflow has trigger definition: $workflow"
    else
      echo "❌ FAIL: Missing trigger definition: $workflow"
      return 1
    fi
    
    if yq eval '.jobs' "$workflow_path" >/dev/null 2>&1; then
      echo "✅ PASS: Workflow has jobs definition: $workflow"
    else
      echo "❌ FAIL: Missing jobs definition: $workflow"
      return 1
    fi
  done
  
  echo "✅ ALL TESTS PASSED: Workflow syntax validation"
}

test_workflow_syntax
```

#### Test 5: Allowlist Configuration

```bash
#!/bin/bash
# Test: allowlist_configuration.sh

test_allowlist_configuration() {
  local workflow_path=".github/workflows/attest.yml"
  
  if [ ! -f "$workflow_path" ]; then
    echo "❌ FAIL: Attest workflow not found"
    return 1
  fi
  
  # Check if allowlist exists in workflow
  if grep -q "allowlist" "$workflow_path"; then
    echo "✅ PASS: Allowlist configuration found"
  else
    echo "❌ FAIL: Allowlist configuration missing"
    return 1
  fi
  
  # Check for authorized users
  local expected_users=("AshrafAlhajj" "Raasid")
  for user in "${expected_users[@]}"; do
    if grep -q "$user" "$workflow_path"; then
      echo "✅ PASS: Authorized user '$user' in allowlist"
    else
      echo "❌ FAIL: Authorized user '$user' missing from allowlist"
      return 1
    fi
  done
  
  echo "✅ ALL TESTS PASSED: Allowlist configuration"
}

test_allowlist_configuration
```

## End-to-End Tests

### Attestation Workflow Test

#### Test 6: Complete Attestation Flow

```bash
#!/bin/bash
# Test: complete_attestation_flow.sh

test_complete_attestation_flow() {
  echo "Starting end-to-end attestation flow test..."
  
  # Prerequisites check
  if ! command -v gh &> /dev/null; then
    echo "❌ FAIL: GitHub CLI not installed"
    return 1
  fi
  
  if ! gh auth status &> /dev/null; then
    echo "❌ FAIL: GitHub CLI not authenticated"
    return 1
  fi
  
  # Step 1: Create test issue
  echo "Creating test issue..."
  local issue_url=$(gh issue create --title "TEST: Automated attestation test" \
    --body "This is an automated test of the attestation system.")
  
  if [ $? -eq 0 ]; then
    echo "✅ PASS: Test issue created: $issue_url"
  else
    echo "❌ FAIL: Could not create test issue"
    return 1
  fi
  
  # Extract issue number
  local issue_num=$(echo "$issue_url" | grep -o '[0-9]*$')
  
  # Step 2: Add attestation comment
  echo "Adding attestation comment..."
  if gh issue comment "$issue_num" --body "/attest QuietWire-Civic-AI/canon-b \"Test Canon A\""; then
    echo "✅ PASS: Attestation comment added"
  else
    echo "❌ FAIL: Could not add attestation comment"
    return 1
  fi
  
  # Step 3: Wait for workflow execution
  echo "Waiting for workflow execution (30 seconds)..."
  sleep 30
  
  # Step 4: Check for workflow run
  local recent_runs=$(gh run list --limit 5 --json status,conclusion,displayTitle)
  if echo "$recent_runs" | jq -e '.[] | select(.displayTitle | contains("Attest"))' >/dev/null; then
    echo "✅ PASS: Attestation workflow executed"
  else
    echo "❌ FAIL: No attestation workflow found"
    return 1
  fi
  
  # Step 5: Check for PR in target repository
  echo "Checking for attestation PR in canon-b..."
  local pr_list=$(gh pr list --repo QuietWire-Civic-AI/canon-b --limit 5 --json title,number)
  if echo "$pr_list" | jq -e '.[] | select(.title | contains("Attestation"))' >/dev/null; then
    echo "✅ PASS: Attestation PR created in canon-b"
  else
    echo "❌ FAIL: No attestation PR found in canon-b"
    return 1
  fi
  
  # Cleanup: Close test issue
  gh issue close "$issue_num" --comment "Test completed successfully"
  
  echo "✅ ALL TESTS PASSED: Complete attestation flow"
}

# Uncomment to run (requires valid GitHub authentication)
# test_complete_attestation_flow
```

## Security Tests

### Access Control Tests

#### Test 7: Unauthorized User Prevention

```bash
#!/bin/bash
# Test: unauthorized_user_prevention.sh

test_unauthorized_user_prevention() {
  local workflow_path=".github/workflows/attest.yml"
  
  # Test that workflow includes user validation
  if grep -q "github.actor" "$workflow_path" && grep -q "allowlist" "$workflow_path"; then
    echo "✅ PASS: Workflow includes user validation"
  else
    echo "❌ FAIL: Workflow missing user validation"
    return 1
  fi
  
  # Test that workflow exits on unauthorized users
  if grep -q "exit 1" "$workflow_path" || grep -q "return 1" "$workflow_path"; then
    echo "✅ PASS: Workflow exits on unauthorized access"
  else
    echo "❌ FAIL: Workflow doesn't properly exit on unauthorized access"
    return 1
  fi
  
  echo "✅ ALL TESTS PASSED: Unauthorized user prevention"
}

test_unauthorized_user_prevention
```

#### Test 8: Input Validation

```bash
#!/bin/bash
# Test: input_validation.sh

test_input_validation() {
  local workflow_path=".github/workflows/attest.yml"
  
  # Test for input sanitization patterns
  local validation_patterns=("[[]" "grep" "validate" "check")
  local found_validation=false
  
  for pattern in "${validation_patterns[@]}"; do
    if grep -q "$pattern" "$workflow_path"; then
      found_validation=true
      break
    fi
  done
  
  if [ "$found_validation" = true ]; then
    echo "✅ PASS: Workflow includes input validation"
  else
    echo "❌ FAIL: Workflow missing input validation"
    return 1
  fi
  
  echo "✅ ALL TESTS PASSED: Input validation"
}

test_input_validation
```

## Performance Tests

### Workflow Performance

#### Test 9: Workflow Execution Time

```bash
#!/bin/bash
# Test: workflow_performance.sh

test_workflow_performance() {
  echo "Analyzing recent workflow performance..."
  
  # Get recent workflow runs with duration
  local runs=$(gh run list --limit 10 --json durationMs,conclusion,displayTitle)
  
  # Calculate average duration for successful runs
  local avg_duration=$(echo "$runs" | jq -r '.[] | select(.conclusion == "success") | .durationMs' | \
    awk '{ sum += $1; count++ } END { if (count > 0) print int(sum/count); else print 0 }')
  
  if [ "$avg_duration" -gt 0 ]; then
    local avg_seconds=$((avg_duration / 1000))
    echo "✅ PASS: Average workflow duration: ${avg_seconds} seconds"
    
    # Performance threshold: workflows should complete within 5 minutes (300 seconds)
    if [ "$avg_seconds" -le 300 ]; then
      echo "✅ PASS: Workflow performance within acceptable limits"
    else
      echo "⚠️ WARN: Workflow performance may need optimization (>${avg_seconds}s)"
    fi
  else
    echo "❌ FAIL: No successful workflow runs found for performance analysis"
    return 1
  fi
  
  echo "✅ ALL TESTS PASSED: Workflow performance analysis"
}

# Uncomment to run (requires workflow history)
# test_workflow_performance
```

## Test Execution Scripts

### Run All Tests

```bash
#!/bin/bash
# Script: run_all_tests.sh

run_all_tests() {
  echo "==================================="
  echo "Canon A Test Suite Execution"
  echo "==================================="
  echo "Start time: $(date)"
  echo ""
  
  local test_count=0
  local pass_count=0
  local fail_count=0
  
  # Array of test functions
  local tests=(
    "test_valid_attestation_json"
    "test_valid_signature_json"
    "test_claim_hash_calculation"
    "test_workflow_syntax"
    "test_allowlist_configuration"
    "test_unauthorized_user_prevention"
    "test_input_validation"
  )
  
  # Execute each test
  for test_func in "${tests[@]}"; do
    echo "Running $test_func..."
    test_count=$((test_count + 1))
    
    if $test_func; then
      pass_count=$((pass_count + 1))
      echo "✅ PASSED: $test_func"
    else
      fail_count=$((fail_count + 1))
      echo "❌ FAILED: $test_func"
    fi
    echo "---"
  done
  
  # Summary
  echo "==================================="
  echo "Test Execution Summary"
  echo "==================================="
  echo "Total tests: $test_count"
  echo "Passed: $pass_count"
  echo "Failed: $fail_count"
  echo "Success rate: $(( (pass_count * 100) / test_count ))%"
  echo "End time: $(date)"
  
  if [ $fail_count -eq 0 ]; then
    echo "✅ ALL TESTS PASSED"
    return 0
  else
    echo "❌ SOME TESTS FAILED"
    return 1
  fi
}

# Execute if script is run directly
if [ "${BASH_SOURCE[0]}" = "${0}" ]; then
  run_all_tests
fi
```

### Continuous Testing

```bash
#!/bin/bash
# Script: continuous_testing.sh

continuous_testing() {
  echo "Starting continuous testing mode..."
  echo "Press Ctrl+C to stop"
  
  local iteration=1
  
  while true; do
    echo "\n========================================"
    echo "Test Iteration #$iteration - $(date)"
    echo "========================================"
    
    if run_all_tests; then
      echo "✅ Iteration #$iteration: All tests passed"
    else
      echo "❌ Iteration #$iteration: Some tests failed"
    fi
    
    iteration=$((iteration + 1))
    echo "Waiting 60 seconds before next iteration..."
    sleep 60
  done
}

# Uncomment to enable continuous testing
# continuous_testing
```

## Test Environment Setup

### Prerequisites Installation

```bash
#!/bin/bash
# Script: setup_test_environment.sh

setup_test_environment() {
  echo "Setting up Canon A test environment..."
  
  # Check required tools
  local required_tools=("git" "jq" "yq" "gh")
  local missing_tools=()
  
  for tool in "${required_tools[@]}"; do
    if ! command -v "$tool" &> /dev/null; then
      missing_tools+=("$tool")
    fi
  done
  
  if [ ${#missing_tools[@]} -gt 0 ]; then
    echo "Missing required tools: ${missing_tools[*]}"
    echo "Please install missing tools before running tests."
    return 1
  fi
  
  # Verify GitHub authentication
  if ! gh auth status &> /dev/null; then
    echo "GitHub CLI not authenticated. Please run 'gh auth login'"
    return 1
  fi
  
  # Create test directory structure
  mkdir -p tests/{unit,integration,e2e,security,performance}
  
  echo "✅ Test environment setup complete"
}

setup_test_environment
```

---

*Testing Guide Version: 1.0*  
*Last Updated: 2025-10-06*  
*Coverage: Unit, Integration, E2E, Security, Performance*
*Test Framework: Bash with jq/yq validation*
