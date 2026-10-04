# Analysis Methodology

Use three passes on every review. Keep deep edge-case hunting proportional to evidence in scope.

## Pass 1 — Triage (attack surface)

1. Identify language(s) and frameworks
2. Map entry points: HTTP routes, APIs, CLI, uploads, jobs, webhooks, and relevant deploy/proxy config
3. Map trust boundaries: user input, other services, filesystem, model/tool outputs
4. Note sensitive operations: authz, DB, files, subprocess, crypto, sessions, LLM tools

Build a mental model only — do not dump this map to the user.

## Pass 2 — Deep scan

For each entry point and sensitive operation:

1. Trace untrusted input → transformations → sinks
2. Check sinks against `02-checks.md` (vuln classes, language pitfalls, crypto, LLM trust boundaries)
3. Look for logic flaws: missing authn/authz, IDOR, workflow bypass
4. Look for disclosure: secrets in source/env defaults, PII in logs, verbose errors

When edge/proxy config is in scope, check for path/authorization mismatch and request-derived redirects/headers (CRLF / open redirect). Do not prioritize proxy nuance over clear app-level sinks.

## Pass 3 — Validate

For each candidate:

1. Is it attacker-reachable?
2. Did you miss sanitization or an upstream gate?
3. Is impact real?
4. For access control: classify missing authentication, function-level authz, object-level authz, or tenancy failure
5. Include one plausible attacker input shape that reaches the sink
6. For top findings: name one concrete change that would invalidate the finding

Order findings by practical risk (exploitability × impact × confidence).  
Valid → format per `03-output-format.md`. Ruled out → discard silently.

## Severity guide

| Severity | Criteria |
|----------|----------|
| CRITICAL | Direct RCE, auth bypass, mass data exfiltration with weak/no preconditions |
| HIGH     | SQLi, stored XSS, IDOR on sensitive data, hardcoded secrets, insecure deserialization, eval on user input |
| MEDIUM   | Reflected XSS, CSRF, open redirect, weak crypto outside core auth |
| LOW      | Info disclosure, missing security headers, verbose errors |
| INFO     | Best-practice issues with negligible exploitability |
