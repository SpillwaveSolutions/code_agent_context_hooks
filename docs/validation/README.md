# CCH Validation Evidence

This directory contains validation evidence for CCH releases following the IQ/OQ/PQ framework.

## Directory Structure

```
docs/validation/
├── README.md           # This file
├── iq/                 # Installation Qualification evidence
│   └── {date}/
│       ├── macos-arm64/
│       ├── macos-intel/
│       ├── windows/
│       └── linux/
├── oq/                 # Operational Qualification evidence
│   └── {date}/
│       ├── test-results.json
│       ├── scenarios/
│       └── evidence/
├── pq/                 # Performance Qualification evidence
│   └── {date}/
│       ├── benchmarks/
│       ├── load-tests/
│       └── stability/
└── sign-off/           # Validation sign-off reports
    └── v{version}-validation-report.md
```

## Validation Framework

See the [Project Constitution](.speckit/constitution.md#validation-framework-iqoqpq) for full details.

### IQ: Installation Qualification
Verifies software installs correctly across all target platforms.

### OQ: Operational Qualification
Verifies features function correctly in operational environments.

### PQ: Performance Qualification
Verifies system meets performance requirements under realistic load.

## Quick Reference

### Running Validation Tests

```bash
# Integration tests (OQ subset) - REQUIRED before release
task integration-test

# Unit tests
cd cch_cli && cargo test

# Pre-flight checks (includes integration tests)
.opencode/skill/release-cch/scripts/preflight-check.sh
```

### Creating Validation Evidence

```bash
# Create dated evidence directory
DATE=$(date +%Y-%m-%d)
mkdir -p docs/validation/iq/$DATE/{macos-arm64,macos-intel,windows,linux}
mkdir -p docs/validation/oq/$DATE/{scenarios,evidence}
mkdir -p docs/validation/pq/$DATE/{benchmarks,load-tests,stability}
```

### Evidence Naming Convention

- `iq-{platform}-{date}.md` - IQ reports
- `oq-{test-id}-{date}.json` - OQ test results
- `pq-benchmark-{platform}-{date}.csv` - PQ metrics

## Validation Gates

| Gate | Requirement | Evidence |
|------|-------------|----------|
| PR Merge | Integration tests pass | CI logs |
| Release Candidate | Full IQ + OQ on all platforms | Evidence files |
| Production Release | IQ + OQ + PQ with sign-off | Signed report |

## Reference Documents

- [IQ/OQ/PQ Integration Testing Guide](../IQ_OQ_PQ_IntegrationTesting.md)
- [Project Constitution](../../.speckit/constitution.md)
- [Integration Test README](../../test/integration/README.md)
