# Canon A - Cross-Attestation Repository

[![Attestation Verification](https://github.com/QuietWire-Civic-AI/canon-a/actions/workflows/verify-attestation.yml/badge.svg)](https://github.com/QuietWire-Civic-AI/canon-a/actions/workflows/verify-attestation.yml)

This repository implements the **Canon A** component of the GTFO_2 cross-attestation system, enabling decentralized trust between AI companions through cryptographic attestations.

## Overview

Canon A serves as one half of a two-repository mutual attestation system. It:
- Maintains canonical claims about its own state and intentions
- Creates cryptographic attestations about Canon B's repository
- Validates incoming attestations through automated workflows
- Provides a complete audit trail of all cross-attestations

## Repository Structure

```
canon-a/
├── README.md                    # This file
├── README_AR.md                 # Arabic documentation
├── AUTHORS.md                   # Project contributors
├── SECURITY.md                  # Security policies and procedures
├── RUNBOOK.md                   # Operational procedures
├── DEMO.md                      # Demonstration guide
├── TESTS.md                     # Testing procedures
├── canon/
│   ├── claims/
│   │   └── claim-A-0001.txt     # Canon A's self-claims
│   └── attestations/            # Attestations made by Canon A
└── .github/
    ├── PULL_REQUEST_TEMPLATE.md
    └── workflows/
        ├── attest.yml           # Automated attestation workflow
        └── verify-attestation.yml # Attestation verification
```

## Quick Start (10 Steps)

1. **Clone the repository**
   ```bash
   git clone https://github.com/QuietWire-Civic-AI/canon-a.git
   cd canon-a
   ```

2. **Review the canonical claims**
   ```bash
   cat canon/claims/claim-A-0001.txt
   ```

3. **Check existing attestations**
   ```bash
   ls -la canon/attestations/
   ```

4. **Set up repository secrets** (if using Actions path)
   - Go to repository Settings → Secrets and variables → Actions
   - Add `BOT_TOKEN` with your fine-grained PAT

5. **Create an attestation request** (Actions path)
   - Open an issue in this repository
   - Comment: `/attest QuietWire-Civic-AI/canon-b "Canon A"`

6. **Monitor the workflow**
   - Check the Actions tab for running workflows
   - Verify attestation PR creation in target repository

7. **Review attestation format**
   ```bash
   # Attestations follow this JSON schema:
   cat canon/attestations/example.json
   cat canon/attestations/example__sig.json
   ```

8. **Validate attestation signatures**
   ```bash
   # Verify JSON structure and signature files
   jq . canon/attestations/*.json
   ```

9. **Check verification workflow**
   - Review `.github/workflows/verify-attestation.yml`
   - Ensure all checks pass on attestation PRs

10. **Read the documentation**
    - `SECURITY.md` for security policies
    - `RUNBOOK.md` for operational procedures
    - `DEMO.md` for demonstration scenarios

## Attestation Schema

All attestations follow this JSON structure:

```json
{
  "attestor_repo": "QuietWire-Civic-AI/canon-a",
  "attestor_companion": "Canon A",
  "subject_repo": "QuietWire-Civic-AI/canon-b",
  "subject_commit": "<SHA of main branch on subject>",
  "claim_hash": "<sha256 of target claim file>",
  "assertion": "We attest that <subject_commit> matches the claim & norms.",
  "timestamp_utc": "<RFC3339 timestamp>",
  "schema_version": "0.2"
}
```

Signature files use the same basename with `__sig.json` suffix and contain a SHA-256 digest of the evidence encoded in base64:

```json
{
  "signed_by": "Canon A",
  "signature_type": "sha256-base64",
  "signature_b64": "<base64 sha256 of attestation json>",
  "fingerprint": "sha256:<hex sha256 of attestation json>"
}
```

## Contributing

See [AUTHORS.md](AUTHORS.md) for the list of contributors and their roles.

## License

© QuietWire • Civic-AI-Canon — Contributors listed in AUTHORS.md

## Related Repositories

- [Canon B](https://github.com/QuietWire-Civic-AI/canon-b) - The companion attestation repository
- [GTFO_2 Documentation](../docs/) - Complete system documentation
