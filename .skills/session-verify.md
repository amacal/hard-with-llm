Verify that the most recently written notes file, `.history/` entry, and `.index/sessions/<slug>/` directory conform to every rule in CLAUDE.md. Report each violation specifically — quote the offending text and cite the rule it breaks. No scripts — every check here is `Read`/`Grep`/`Glob` against the actual files, per `.index/schema.yml` and `.history/schema.yml`.

Your prompt should already include the write agent's report: every file it wrote/edited/removed, with paths. Use that to know what changed and where — don't re-derive it by globbing the whole `.index/`/`.history/` tree. Spot-check with targeted `Grep`/`Glob` calls (e.g. `grep -l 'title: "<Title>"' .index/sessions/*/meta.yml`, `grep -rl "<Title>" .index/sessions/*/prerequisites.yml`) rather than reading everything.

## What to check

**Identify the files.** The notes file, `.history/` entry, and `.index/sessions/<slug>/` directory named in the write agent's report.

**Notes file** (skip entirely if this was a datasheet-study session with no notes file — confirm that's actually why it's absent):

1. **Section order.** `# Title`, `## Overview`, bespoke theory/mechanism sections (any names), `## Observable behavior`, `## Internal mechanism`, `## Design rationale`, `## Edge cases` (optional), `## Worked example`. Flag missing/out-of-order/forbidden sections (`## Depends on`, `## Unlocks`).
2. **Prose only.** No bullets/numbered lists anywhere, including the worked example. Flag any `-`, `*`, `1.` list markers.
3. **Math formatting.** All math in `$$...$$` on its own line. Flag inline math (backtick math, parenthetical formulas, `$...$`).
4. **Formula framing.** Every `$$` block needs a preceding sentence (why) and a following sentence (what it means). Flag any missing either.
5. **Paragraph length.** Flag standalone one-sentence paragraphs not attached to a neighbor.
6. **Word count.** `wc -w`. Flag if outside 600–1400 without an explicit justification in Overview or Design rationale.
7. **Worked example.** Non-trivial (not degenerate), mentally verifiable under a minute, entirely prose (no bullet trace), concrete register values/pin signal/CPU steps.
8. **Citation vs. re-derivation.** A concept covered in a prior session's notes must be cited by filename, not re-derived. Flag any full re-derivation of a previously-covered result.
9. **No personal data.** Flag names, emails, or other personal information.

**`.history/` entry:**

1. **Placement and naming.** File exists at exactly `.history/<YYYY-MM>/<YYYY-MM-DD>-<slug>.yml`; its own `date:` field matches the filename's date; its own `title:` field, run through `.index/schema.yml`'s `slug_algorithm`, matches the filename's slug. Flag any mismatch.
2. **Schema completeness.** `date`, `title`, `session` block with every fixed field present: `file`, `source`, `status`, `attempted`, `explored`, `tried`, `corrections`, `bugs_found`, `completed`, `not_completed`, `open_questions`, `notes`. Flag any missing field, even when empty.
3. **Empty-field convention.** Empty = exactly `—`, never an empty list, never omitted. Flag any list mixing `—` with real entries, and flag `bugs_found` not `—` on a datasheet-study entry (no code, no defects possible).
4. **No `Depends on`/`Unlocks`.** Flag any such field or resemblance anywhere — belongs solely in `.index/`.
5. **No register-model/mechanism exposition.** Flag any full register-model description, state-machine derivation, or timing-correctness argument (belongs in the notes file). A short factual reference naming a result ("Compared observed timing against the datasheet's stated settling time") is fine; reproducing the derivation is not.
6. **No personal/psychological content.** Flag personal info, personality description, psychological interpretation, inferred learning style, subjective judgment of intelligence/ability/motivation/behavior. `corrections` must be factual before/after about an assumption or prediction, never an evaluation of the person.
7. **Field discipline.** Spot-check: `completed` items are observable outcomes, not vague ("understood X," "gained insight"); `not_completed` isn't silently empty when the session mentions deferred work; `open_questions` isn't populated with an invented "natural next step" that wasn't actually raised.
8. **No retrospective edits to other entries.** Confirm (via the write agent's file list, or `git status .history/` if needed) that no `.history/` file *other than* the one new entry was touched this close, unless it's one of the immutability-rule exceptions (factual error, formatting, filename/date correction, genuinely-omitted same-session item) — and that exception is explicitly stated in the write agent's report. A change for a newly-discovered dependency/reuse/future-target is a violation — belongs in `.index/` instead.

**`.index/` — consistency and incremental-update discipline:**

1. **Directory shape.** `ls .index/sessions/<slug>/` is exactly the 10 files `.index/schema.yml` enumerates (`meta.yml`, `summary.txt`, and the 8 relationship lists) — none missing, none extra (in particular, no `reuses_code` file should exist).
2. **New entry correctly classified.** `Read` each relationship file and verify it was classified honestly against CLAUDE.md's "Fast index (`.index/`)" tests, comparing against the sibling sessions it actually cites (`Read` those specific files, not the whole tree) — e.g. flag a `prerequisites.yml` entry that's really just chronology, or a harder peripheral listed as prerequisite of a simpler one. Flag a missing cross-platform `related_to` link if this session's title names a chip already paired with an equivalent peripheral on another target.
3. **`prerequisites.yml`/`uses_concepts.yml` resolve.** For each title listed, `grep -l 'title: "<ref>"' .index/sessions/*/meta.yml` finds exactly one file.
4. **`concepts.yml`/`capabilities.yml` reachable.** For each tag, `grep -rl "<tag>" .index/sessions/*/concepts.yml` (or `capabilities.yml`) includes this session's directory.
5. **Date consistency.** `meta.yml`'s `date` matches the `.history/` entry's `date` exactly.
6. **`target` validity.** `meta.yml`'s `target` field is one of `microbit-v2 | cyberpi-esp32 | raspberry-pi-500 | basys3`, and matches the target implied by the `file` field's directory (e.g. `microbit/...` implies `microbit-v2`).
7. **`future_targets` resolved structurally.** If this session's own title was previously a `.index/future-targets/<slug>.yml` file, confirm it's now gone (`rm`'d) — can't be both a completed session and a future target. Flag any bare string flag baked into an identifier — that belongs in the future-target file's `status: not-completed` field.
8. **No unwarranted full re-derivation.** Confirm the write agent's report lists only the new session's own files plus any explicitly-justified touches — not a sweep implying every existing session was re-read or rewritten.
9. **Retrospective revisions reflected correctly, and only in `.index/`.** If an earlier session's file was revised, confirm the write agent's report states a specific reason tied to this new session's evidence, confirm only that one file changed (not a cascade), and confirm it was **not** mirrored into that session's `.history/` entry.

Flag as a violation any of the above left stale or inconsistent.

## Output format

List each violation as: **[File] Rule violated — quoted offending text**. If none found, say so explicitly and give the notes file's word count (or state "datasheet-study, no notes file" if applicable).
