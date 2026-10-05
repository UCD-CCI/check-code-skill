# Output Format

Every finding must include every field below. No field may be omitted.

---

## Finding [N]: [Vulnerability Class]

**Finding ID:** `APPSEC-[stable-id]`  
**File:** `path/to/file.ext`  
**Lines:** [start]–[end]  
**CWE:** CWE-[ID] — [Name]  
**OWASP Category:** [e.g., A03:2021 – Injection]  
**Priority:** P1 | P2 | P3  
**Severity:** CRITICAL | HIGH | MEDIUM | LOW | INFO  
**Confidence:** HIGH | MEDIUM | LOW  

### Vulnerable Code
```[language]
[exact snippet from the file — unmodified]
```

### Description
2–4 sentences: what the issue is, why this code is vulnerable, attacker impact. Name the attack type and the source→sink path.

### Prioritization
Relative remediation priority (exploitability, impact, confidence). If not top priority, note what ranks higher.

### Exploitability Notes
- **Exploit Preconditions:** what must be true for exploitation  
- **Attacker input shape:** one concrete path, parameter, header, body field, or sequence that reaches the sink (required by Pass 3)  
- **Uncertainty Boundary:** runtime/config unknowns that could change impact  
- **Invalidating change:** one concrete code/config change that would make this finding no longer valid (required for P1/P2; recommended for P3)

### Remediation
Concrete fixed code in the same language. See `04-remediation.md`. Match the project’s idioms (SQL placeholders, path APIs, etc.). Prefer safer patterns than the templates when you know a better standard library fix.

### References
- Authoritative link (OWASP, CWE, or similar)

---

## Priority vs severity

**Severity** = inherent impact if exploited (CRITICAL…INFO).  
**Priority** = remediation urgency given exploitability and preconditions. They may differ.

| Priority | Meaning |
|----------|---------|
| P1 | Fix now — easy to reach and/or high impact |
| P2 | Next cycle — real issue, more preconditions or lower blast radius |
| P3 | Backlog — limited exploitability, defense-in-depth, or low impact |

Example: a HIGH severity issue behind strong, unlikely preconditions can be **P2** or **P3**.

## OWASP labels

- **Classic web/app findings:** use **OWASP Top 10:2021** (e.g. `A03:2021 – Injection`, `A01:2021 – Broken Access Control`).
- **LLM / GenAI trust-boundary findings:** use an **OWASP Top 10 for LLM / GenAI** label when one fits (e.g. prompt injection, sensitive information disclosure, excessive agency, improper output handling). You may put that in **OWASP Category**, or pair it with a 2021 web category when both apply.
- Always include a **CWE** as well.

Example LLM-oriented category strings:
- `OWASP LLM01 – Prompt Injection`
- `OWASP LLM02 – Sensitive Information Disclosure`
- `OWASP LLM06 – Excessive Agency`
- `OWASP LLM05 – Improper Output Handling`

## After all findings — Scan Summary

```
## Scan Summary

| Metric          | Value |
|-----------------|-------|
| Files Analyzed  | N     |
| Total Findings  | N     |
| Critical        | N     |
| High            | N     |
| Medium          | N     |
| Low             | N     |
| Info            | N     |

### Key Risks
[2–3 sentences on the most impactful issues.]
```

## Stable Finding IDs

Build a deterministic ID (not a random UUID):

1. **File path as given in the review scope** — repo-relative when reviewing a repository; for uploads or paste-only chats, use the path/name provided with the file  
2. CWE id  
3. Normalized sink snippet (collapse whitespace; keep key function/operator names)  
4. Hash with **SHA-256 only** (not SHA-1); keep the **first 12** hex chars. Compute the hash when a sandbox/tool is available — do not invent IDs.
5. Emit `APPSEC-<hash>`  

Same issue in the same file should keep the same ID across wording changes.

## Confidence

- **HIGH** — clear source→sink, straightforward exploitation  
- **MEDIUM** — likely, but depends on config/upstream not fully in scope  
- **LOW** — suspicious; may be a false positive — still report with uncertainty  

Order findings by Priority first (P1→P3), then by Severity within the same priority.
