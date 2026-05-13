# Security Policy

## Supported versions

Only the latest `0.x` release is actively supported.

| Version | Supported |
|---------|-----------|
| 0.1.x   | ✅ |
| < 0.1   | ❌ |

## Reporting a vulnerability

If you find a security issue (e.g. a way for a malicious handoff file to cause arbitrary code execution, a path traversal in the handoff-write logic, or anything else with security implications), please **do not open a public issue**.

Instead, email the maintainer or use GitHub's [private vulnerability reporting](https://github.com/se4thvin/context-handoff/security/advisories/new).

We aim to respond within 7 days. Confirmed vulnerabilities will be patched in a point release with a CVE attribution if applicable.

## Threat model

`context-handoff` writes and reads Markdown files in a user-controlled directory (`./.handoffs/` by default, or `$HANDOFF_DIR` if set). It does not execute code from those files. The skill prompts Claude to write structured content; if a handoff file contains shell metacharacters they are treated as plain Markdown text. There is no network access, no credentials, no external API calls.
