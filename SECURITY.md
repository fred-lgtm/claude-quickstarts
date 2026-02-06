# Security Policy

## Reporting a Vulnerability
Email fred@brickface.com with details. Do not open public issues.

## Supported Versions
Only the latest version on main is supported.

## Security Practices
- All credentials managed via 1Password CLI (op://)
- No plaintext secrets in code or config
- Automated secret scanning via TruffleHog in CI
- Dependency auditing via Dependabot
