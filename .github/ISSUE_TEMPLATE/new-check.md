---
name: New Check Proposal
about: Propose a new check item for an existing or new category
title: "[check] "
labels: enhancement
---

## Category

Which reference file does this belong to? (e.g., `security.md`, `naming.md`, or a new category)

## Check Name

Short name for the check (e.g., "Hardcoded Secrets", "Generic Names")

## Severity

- [ ] critical (-20)
- [ ] error (-10)
- [ ] warning (-4)
- [ ] info (-1)

## Detection

How should Claude detect this? (Grep pattern, Glob pattern, Bash command)

```
Example: Grep for `password\s*=\s*["']` in source files
```

## Roast Example

**English:**
> "Your example roast line here"

**Japanese (optional):**
> "日本語のroast例"

## Fix

What's the recommended fix?

## Real-World Evidence

Have you seen this pattern in actual projects? Link examples if possible.
