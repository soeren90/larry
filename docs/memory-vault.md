# The memory vault

The memory vault is a folder of plain Markdown files that every agent reads
before a task and writes to after. It doubles as a personal knowledge base
(openable in any notes app). No database, no embeddings — and at this scale,
none needed.

## Why plain Markdown

- **Diff-able.** Every change to what the system "knows" shows up in `git diff`.
- **Inspectable.** You can read the memory with your eyes. There's no opaque
  vector blob to debug.
- **Portable.** It's just files. It survives tool changes and outlives any one
  runtime.
- **Good enough.** Retrieval is by an index + filenames + frontmatter tags, not
  similarity search. For a personal operations system, that's plenty.

## The shape of a memory

One fact per file, with light frontmatter so relevance is easy to judge:

```markdown
---
name: short-kebab-slug
description: one-line summary used to decide relevance at recall time
type: user | feedback | project | reference
---

The fact itself. For feedback/project facts, follow with why it matters and
how to apply it. Link related notes with [[other-slug]].
```

The four types:

| Type | Holds |
| --- | --- |
| `user` | who the operator is — role, preferences, working style |
| `feedback` | guidance on *how* the system should work, with the reasoning |
| `project` | ongoing work and constraints not derivable from the data |
| `reference` | pointers to external resources |

## The index

A single index file is loaded every session — one line per memory, a title and
a hook. It's the map of what the system knows, so an agent can decide what to
pull before acting. The index never holds memory *content*, only pointers.

## Read before, write after

- **Before** a task: pull the notes relevant to it.
- **After** a task: write down what was learned — a customer fact, a project
  constraint, a corrected preference.
- **Before saving:** check for an existing file that already covers it and
  update that, rather than creating a duplicate.

## Hard rules

- **No secrets.** No passwords, credentials, or payment data, ever.
- **Append-only on wired paths.** Some folders are referenced by scripts;
  maintenance consolidates additively and never renames or moves them.
- **Facts, not transcripts.** If something only mattered to one conversation,
  it doesn't belong here.

## Maintenance

A monthly job (see [scheduled-tasks.md](scheduled-tasks.md)) consolidates the
vault — merges duplicates, flags stale facts, fixes dead links, closes index
gaps. Cosmetic and unambiguous fixes happen directly; anything that changes
meaning is written as a proposal for human approval.
