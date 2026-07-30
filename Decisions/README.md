# Decisions

Lightweight records of decisions made for The Forest Network's apps and infrastructure — things that aren't obvious from the code alone, and that we don't want to re-litigate or re-discover from scratch later.

## When to add one

When a non-trivial decision is made that:
- isn't obvious from reading the code or commit history, or
- involved weighing a real tradeoff, or
- future contributors (including future us) are likely to question or accidentally undo.

Small, obvious, or easily-reversible choices don't need a record.

## Format

One file per decision: `NNNN-short-title.md`, numbered sequentially. Each one starts with a **Date** (the date the decision was made, `YYYY-MM-DD`) right under the title, then covers:

- **Context** — what prompted this, what we found.
- **Decision** — what we chose to do.
- **Consequences** — what that means going forward, including anything left as follow-up.

Status isn't tracked separately — if a decision is later reversed or superseded, add a new numbered entry and link back to the one it replaces, rather than editing the original.
