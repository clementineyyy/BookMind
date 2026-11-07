# Domain Docs

How engineering skills consume this repo's domain documentation.

## Before exploring, read these

- `GLOSSARY.md` at the repository root, when present.
- Relevant decisions in `docs/adr/`, when present.

If these files do not exist, proceed silently. Create them when domain terms or decisions are actually resolved, rather than as setup scaffolding.

## File structure

This repository uses a single-context layout:

```text
/
├── GLOSSARY.md
├── docs/adr/
└── src/
```

## Use the glossary's vocabulary

Use domain terms as defined in `GLOSSARY.md`. If a needed concept is missing, record the gap for domain modeling.

## Flag ADR conflicts

If a proposal contradicts an existing ADR, identify that conflict explicitly instead of silently overriding it.
