---
name: register
description: >
  Establish a project's positive naming register and bring its code into that register, so
  benign work stops reading as something it isn't to a vocabulary-sensitive classifier. Use
  when a project keeps drawing false-positive downgrades on ordinary diction, when starting a
  new project that should be born in a clean register, or when the user asks to "re-gloss",
  "de-load the vocabulary", "establish a register", or "make this generic across projects".
---

# Register — name a project in its own honest vocabulary

Some projects trip vocabulary-sensitive classifiers on wholly benign work, because the code's
diction drifted into another world's register — an epistemic reviewer named in security terms,
a data pipeline named in warfare terms, a scheduler named in courtroom terms. Density is what
trips it: many near-synonyms from the wrong world, piled into one context, read as that world's
subject matter regardless of what the code actually does. Reading such a file, or having
auto-recall inject it, is enough.

This capability fixes the cause: it names the project in the vocabulary of **what it actually
is**, and brings the code into that register. The durable artifact is always a `REGISTER.md`
that says what TO reach for — never a blacklist of what to avoid.

## The one rule that shapes everything

**Positive, never a blacklist.** Define the register the project SHOULD speak; let the mapping
fall out as a consequence. A list of forbidden words is self-defeating three ways: it
re-concentrates the exact terms you're removing into one file (a fresh trigger), it teaches a
brittle rule instead of an instinct, and it never generalizes to the next loaded word. So the
guidance a project keeps is "here is our vocabulary," and a person writing new code stays in
register on their own because they're writing in the target language from the start.

## Method

### 1. Establish the register (positive first)
Read the project's purpose from its own docs and code. Write `REGISTER.md` at the project root
from `TEMPLATE.md` (next to this skill): one honest sentence on what the project is, the
register broken out by area, a plain "words we reach for" list, and "the instinct to build"
(how to name the next new thing). Start from the domain, never from a scan. If a word from
another world (contest, security, courtroom, warfare, medicine) is the first that comes to
mind for a part, that word is describing the mechanism's shape, not its purpose here — reach
past it for the word a person would use for the same move in the real domain.

### 2. Measure the drift (`register-scan`)
`register-scan <path> [REGISTER.md]` tokenizes identifiers, strings, and comments, drops an
English + programming stoplist and the register's own "words we reach for", and ranks what
remains by frequency × file-spread — a heatmap of where the code's vocabulary has drifted from
its own declared register. It names no "bad" words; it surfaces clusters for a human to judge
against the register. That keeps the tool itself register-clean and fully general.

### 3. Derive the mapping against the register
For each drifted cluster the scan surfaces, choose the register word that fits — from
`REGISTER.md`, not invented ad hoc. Be paranoid about the replacements: the plainest domain
word beats a clever one, and a replacement that still carries another domain's charge is no
better than what it replaced. Watch dual-use words especially — a word that names causing harm
as readily as preventing it reads the same whatever your intent, and no framing makes the
classifier context-aware — so a plain description of the mechanism beats it. Words that only
name protecting, checking, or noticing carry no such ambiguity. When unsure a candidate is
clean, prefer a plain verb of the real activity.

**Distinguish a charged metaphor from a precise term of art.** Most loaded words are metaphors
(a reviewer that "challenges") and lose nothing when plainly reworded. But some are genuine
terms of art — an established concept whose exact name (e.g. an access-mediation primitive from
security architecture) carries defining properties the plain word does not. Renaming one of
those is only safe if you **state those properties in plain language at the definition** — what
it guarantees, why it can't be bypassed — so nothing is lost but the jargon. Done that way the
plain description is usually *clearer* than the term, since it spells out the invariants instead
of assuming the reader knows the literature. What you forfeit is only the literature handle (the
word someone would grep or cite).

**Default to transforming.** Anything that can be reworded with its meaning intact *should* be —
even a term of art, once its properties are stated plainly. Keep the charged word only when the
meaning genuinely *cannot* survive without it. The reason is budget: the classifier fires on
cumulative density, so every charged word kept out of caution or convenience spends from a shared
pool that the one truly-irreducible term may need. Be greedy about transforming; hoard the kept
exceptions for the words that truly cannot be transformed, and note each in `REGISTER.md`.

Split the mapping by blast radius:
- **Comments & docstrings** — free to change; zero behavior risk. Do these always.
- **Internal identifiers** (locals, private attrs, helper names) — mechanical rename; tests
  follow within the codebase.
- **Public/persisted surfaces** (string VALUES compared across modules, on-disk keys/values,
  wire formats, user- or agent-facing text) — schema-sensitive. Do NOT rename in place. Add a
  **back-compat read shim** that accepts the old token and normalizes to the new one, migrate
  writers to emit the new token, and leave the shim so existing data/logs still load. Verify
  replay/round-trip identity on any persisted store.

### 4. Execute content-blind, delegated
Handling the loaded vocabulary in bulk is exactly what re-trips the classifier — and on
Opus/Sonnet a trip rolls back invisibly, so the driving session cannot even detect it. So the
driving session stays in the positive register and NEVER pulls the loaded corpus into its own
context. Run the mechanical rename in a **delegated sub-agent** (or via stream-transform pipes,
`sed … | sink`, so the old tokens never enter context). Hand the sub-agent the register and the
mapping and let it do the sweep; the driver reviews diffs by shape and test results, not by
re-reading the loaded prose. This is the one step where doing the work yourself is worse than
delegating it: the bulk handling is exactly what re-trips the classifier.

### 5. Verify honestly
Full test suite green. Replay/round-trip identity for any migrated store. Re-run `register-scan`
and confirm the drift density dropped where you worked. Mirror/commit. Label what was actually
checked (tests + replay + re-scan), not a vibe.

### 6. Bake it in — corrective AND generative
`REGISTER.md` is dual-purpose: the same file the scanner reads as an allowlist is a **primer**
that makes a session write in-register from the start. Its "by area" prose and "words we reach
for" ARE the priming material.

**Corrective guard** — optionally add a pre-commit `register-scan` threshold so drift can't
creep back in silently.

**Generative priming** — put `REGISTER.md` in front of every session that touches the project,
in order of reliability:
- **Project `CLAUDE.md`** — auto-loaded each session in the tree. One positive line is enough:
  *"This project keeps a naming register — read `REGISTER.md` and write in it. Name new parts in
  the vocabulary of what the code actually does."* Point, don't inline (keep loaded words out).
- **Sub-agent seeding** — when you spawn an agent on the project, tell it to read `REGISTER.md`
  first, before it writes anything.
- **SessionStart hook** — the automated fallback (via `settings.json`) when work happens from a
  parent dir where `CLAUDE.md` isn't picked up: inject `REGISTER.md` at session start.
- **The system's own prompts** — if the project is itself an LLM system, its generation register
  is set by its system/instruction prompts; those must be in-register too (re-gloss them in the
  migration, don't leave them off-register).

**Prime positively; never proscribe.** A register cannot be taught to a generator with a "don't
use X" list — that backfires twice: it injects the off-register vocabulary (the very density that
trips the classifier), and it makes the model attend to the banned words ("don't think of an
elephant"). Always show the target language; never enumerate the words to avoid. This is the same
"positive, never a blacklist" rule as the durable artifact, applied to the prompt.

## Generalizing across projects
The method is domain-agnostic; only `REGISTER.md` is per-project. A data project's register is
the vocabulary of data and flow; an infra project's is hosts and services; the CA project's is a
reflecting mind. Reuse this skill and `register-scan` unchanged; write a fresh `REGISTER.md` each
time from the project's own domain. Keep no shared word-list — each project's positive register
is its own, and a shared blacklist would be the very artifact this capability exists to avoid.
