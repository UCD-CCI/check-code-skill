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
- **Uncertainty Boundary:** runtime/config unknowns that could change impact  

### Remediation
Concrete fixed code in the same language. See `04-remediation.md`.

### References
- Authoritative link (OWASP, CWE, or similar)

---

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

1. Repository-relative file path  
2. CWE id  
3. Normalized sink snippet (collapse whitespace; keep key function/operator names)  
4. Hash with SHA-256 (or SHA-1); keep first 10–12 hex chars  
5. Emit `APPSEC-<hash>`  

Same issue in the same file should keep the same ID across wording changes.

## Confidence

- **HIGH** — clear source→sink, straightforward exploitation  
- **MEDIUM** — likely, but depends on config/upstream not fully in scope  
- **LOW** — suspicious; may be a false positive — still report with uncertainty  

Order findings highest practical risk first.
