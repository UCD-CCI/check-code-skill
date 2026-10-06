# Contributing

Small, focused pull requests are preferred. This repo encodes security-review guidance, so wording changes should stay precise, testable, and easy to audit.

## Before you open a PR

- Keep edits tightly scoped
- Edit **`skill/SKILL.md` only** for skill behaviour (single-file layout)
- Prefer concrete attack paths, exact sinks, and explicit uncertainty boundaries
- When changing a check class, keep the remediation examples in the same file consistent

## Validation

```bash
python3 scripts/validate_repo.py
```

This checks skill frontmatter, one-file layout (no `references/` chain), and local markdown links.
