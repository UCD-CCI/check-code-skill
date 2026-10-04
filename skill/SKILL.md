---
name: check-code
version: 0.3.0
spec_version: 1
description: >-
  Performs focused application security code review: short three-pass methodology,
  OWASP/CWE-oriented checks (including common language/crypto pitfalls and LLM
  trust-boundary issues), structured findings, and concrete remediations. Use when
  analyzing source for security vulnerabilities, reviewing changes for AppSec
  issues, or when the user asks for a security review or secure coding feedback.
---

# Check Code Skill

When this skill is active, you conduct secure code review like a senior application security engineer: map the attack surface, hunt source→sink issues, and report grounded findings with fixes.

## When to use

- Security review of files, directories, or pull requests
- Requests to find vulnerabilities, unsafe patterns, secrets, or crypto misuse
- Structured reporting that matches this skill’s finding schema

## Before touching application code

Read these modules **in order** (paths relative to this skill folder):

1. [references/00-identity.md](references/00-identity.md) — mindset and hard rules  
2. [references/01-methodology.md](references/01-methodology.md) — three-pass review  
3. [references/02-checks.md](references/02-checks.md) — vulnerability, language, crypto, and LLM checks  
4. [references/03-output-format.md](references/03-output-format.md) — finding schema  
5. [references/04-remediation.md](references/04-remediation.md) — fix patterns  

## Invocation examples

**Single file:** load this skill, then analyze `<path/to/file>` for security vulnerabilities.

**Directory:** load this skill, then analyze all source files under `<path/to/dir/>` for security vulnerabilities.

Editors that support the [Agent Skills](https://cursor.com/docs/skills) layout discover this folder as a skill; load [`SKILL.md`](SKILL.md) first, then the `references/` chain it lists.
