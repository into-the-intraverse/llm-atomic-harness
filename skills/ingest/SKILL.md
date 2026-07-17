---
name: ingest
description: Extract knowledge atoms from raw source material. Use when the user runs /atomic-wiki:ingest, or when new files appear in raw/ and the user asks you to process them.
---

# Ingest

Read raw material from `raw/` (or any path the user names), classify each segment as `extract` / `skip` / `deferred`, and write the `extract` segments as atoms under `atoms/<branch>/`.

## When to use

- User runs `/atomic-wiki:ingest <file-or-folder>`.
- User drops new material into `raw/` and asks you to process it.
- During the lifecycle of any operation that produces new knowledge worth retaining (e.g., a Query that surfaces a synthesis the user wants captured).

Not for checking atoms the user already wrote about a source — that is `/atomic-wiki:factcheck` with the source as input. Do not re-extract what the user has written themselves.

## Procedure

1. **Read the source.** Treat `raw/` as read-only. Never write back into it.
2. **Classify each segment.** Per segment, mark:
   - **extract** — contains a knowledge point, view, experience, or method that stands alone. Even out of original context, this segment has teaching or reference value.
   - **skip** — pure social interaction, restating others' views, filler agreement, action items, pure emotion.
   - **deferred** — potentially valuable but uncertain, or requires surrounding context to understand. Skip in the first pass; revisit after the full batch.
3. **Extract atoms.** For each `extract` segment:
   - One atom equals one claim. If a passage contains two views, and removing one leaves the other intact, split into two atoms.
   - Refine, don't copy. Strip filler from the source; preserve the author's voice and stance.
   - Place atoms under the matching `atoms/<branch>/` folder.
   - If no branch fits, list the segment as a deferred candidate and surface it to the user. See the branch-design criteria below — **do not create a new branch without user approval.**
4. **Write the atom file.** Use the frontmatter format below. New atoms get `version: 1`.

## Atom frontmatter

```yaml
---
id: <branch>/<descriptive-slug>
type: explanation | opinion | tutorial | myth-busting | case-study | comparison
depth: beginner | intermediate | advanced
source_type: post | reply | thread | transcript | article | note | screenshot | audio | video
source_ids: ["<stable-id-or-url>"]
reuse_score: high | medium | low
tags: []
version: 1
---
```

## Images in sources

Sources may carry images — keyframes woven into a video transcript, screenshots beside a note.
An image belongs to the segment it appears in:

- If the image anchors the segment's claim (a chart, diagram, slide, table, code screenshot),
  the atom **embeds it**: `![<alt>](../../raw/<path-to-image>)`. Atoms live two levels below the
  repo root, so the prefix is always `../../raw/`. Reference the file where it lives — `raw/`
  being read-only means you never *write* there; linking to it is the intended use.
- If the text stands alone without the image, leave the image out.
- For video sources, anchor `source_ids` to the moment: append the segment's timestamp to the
  URL, e.g. `["https://www.youtube.com/watch?v=ID&t=252"]` for a claim made at 04:12. Write the
  URL byte-for-byte with a plain `&` — never HTML-escape it to `&amp;`.

## Filename

`atoms/<branch>/<slug>.md` — all lowercase, hyphens only, 3–6 words. No date prefix. Slug must be unique within the branch and must match the `id:` field.

## Constraints

- One atom equals one claim.
- Use the frontmatter format above. Do not invent fields.
- Preserve the author's voice. Personal knowledge base, not neutral encyclopedia.
- Tag sources via `source_ids` — atoms without source attribution are not auditable.
- An image that anchors a claim travels with its atom as a `../../raw/` embed — never copied,
  never moved.
- If a passage doesn't pass the "extract" bar (pure action items, unannotated news restatement, time-sensitive ephemera, pure emotion), skip it.

## After ingest

- Surface to the user: how many atoms extracted, which branches received them, any deferred candidates, any segments that didn't fit existing branches.
- Run the `/atomic-wiki:factcheck` pass over the freshly extracted atoms (verify claims, attach references) and present its per-atom report alongside the atom list.
- The user reviews: they decide corrections, deferred candidates, and any new-branch approvals.
- Commit only after the user approves the batch. The pre-commit hook will enforce `version: 1` on new atoms.

## Branch-design criteria

Add a new branch only when ALL four of the following hold:

1. **Independence** — the topic doesn't fit cleanly under any existing branch.
2. **Scale** — you expect 5+ atoms in this branch.
3. **Clear boundary** — you can state in one paragraph what belongs and what doesn't.
4. **Teaching independence** — the branch could anchor a 30-minute talk on its own.

If only 1–2 atoms fit a candidate topic, use `tags` instead of creating a new branch. Still require user approval before creating any branch.

See `${CLAUDE_PLUGIN_ROOT}/reference/SCHEMA.md` and `${CLAUDE_PLUGIN_ROOT}/METHODOLOGY.md` (Phase 2 + Phase 3) for the full reasoning.
