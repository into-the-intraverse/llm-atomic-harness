---
name: factcheck
description: Use when the user runs /atomic-wiki:factcheck, asks to verify atoms they wrote ("check my atoms", "did I get this right?"), hands over a source (article, video, link) to check their atoms against, or when freshly extracted atoms need checking before commit.
---

# Factcheck

Referee for atoms: verify each atom's claims, attach the references you find, propose corrections. Core rule: **a verification that does not land in the atom file is lost** — chat replies are not storage.

## When to use

- User wrote atoms in their own words and asks whether they got it right.
- User hands over a source (URL, article, video, file in `raw/`) and wants their atoms checked against it.
- Atoms freshly extracted by `/atomic-wiki:ingest`, before commit.

Not for wiki pages — that is `/atomic-wiki:lint` territory. Factcheck reads atoms.

## Inputs

Atoms to check (explicit paths, a branch, or "everything uncommitted" via `git status`), plus an optional source. Read URLs live (WebFetch); never download a copy into `raw/` — the URL itself is the reference.

## Procedure

1. **Normalize.** Hand-written atoms may be sloppy: check frontmatter fields, slug rules, one-claim-per-atom (propose a split when two claims cohabit).
2. **Check against the source**, when one is given: read it, then per atom decide — understood correctly / imprecise / wrong — quoting the source segment that decides it.
3. **Verify factual claims** with WebSearch:
   - A mentioned study or research finding → find the actual paper. It must match on both identity (the study the user means) and conclusion (it says what the atom says). Existence alone proves nothing.
   - A mentioned author, talk, video → find the canonical URL.
   - Common-knowledge claims → if the user is wrong, draft a correction stating what is right and why.
   - `type: opinion` atoms: the stance is the user's and is not correctable; check only its factual predicates.
4. **Attach references.** Every confirmed source goes into that atom's `source_ids` (plus an inline citation at the body end when the claim needs precision). Found nothing? Say so in the report — the claim stays unattributed. Never cite anything you did not open.
5. **Report per atom** using the template below.
6. **Apply.** If the user's request already authorized edits («поправь», "fix it"), apply and show what changed. Otherwise wait for approval — the user owns the editorial voice. Fix facts, keep voice: change nothing beyond what the correction requires.

## Report template (all fields required, per atom)

```markdown
### <branch>/<slug> — ✅ correct | ⚠️ imprecise | ❌ wrong | ❓ could not verify
- **Claim:** <the atom's core claim, one line>
- **Checked:** <source found and read, or the searches that came up empty>
- **References → source_ids:** <exact IDs added to the atom> | none found — stays unattributed
- **Correction:** <proposed replacement text> | none
```

## Versioning

- Atom already committed → applying a correction bumps `version:` by 1.
- Atom never committed (no version in HEAD) → it stays `version: 1` no matter how much changes.

## Common mistakes

| Mistake | Fix |
|---|---|
| Claim verified in chat, `source_ids` still `[]` | The verification IS the reference — attach it, or the work is lost |
| Bumping `version:` on a never-committed atom | The hook compares against HEAD; new atoms stay at 1 |
| Citing a study found by title but never opened | Open it; confirm the conclusion matches the atom |
| "Correcting" an opinion atom | Flag factual predicates only; the stance stands |
| Rewriting the user's tone while fixing a fact | Minimal diff: change what is wrong, keep the voice |
