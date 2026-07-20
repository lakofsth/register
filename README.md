# register

Name a codebase in its own honest vocabulary — so benign work stops reading as another
domain's subject matter to a vocabulary-sensitive classifier, and so the names simply read
truer.

## The problem

Language models, left to their own defaults, reach for a vivid metaphor when they name things —
a reviewer that *questions* a claim becomes one that *challenges* it, a check that *screens* input
becomes one that *guards* against it, and so on across the whole lexicon of contest and conflict.
Pile enough of one borrowed domain's vocabulary into a project and the code starts to read *as*
that domain — even when it does nothing of the kind. A checker whose job is honesty, but whose
names are all borrowed from security, reads like security tooling. That misleads a human skimming
it, and it can trip an automated classifier that judges by vocabulary, not intent.

The fix is not to police words. It is to name each part for **what it actually does, in the
vocabulary of its real domain** — which is also just better naming.

## The one rule

**Positive, never a blacklist.** The durable artifact is a `REGISTER.md` that says what a
project's vocabulary *is* — the words to reach for — never a list of words to avoid. A blacklist
defeats itself three ways: it re-concentrates the very terms it's trying to remove into one file,
it teaches a brittle rule instead of an instinct, and it never generalizes to the next case.
Everything here follows from that.

## What's in here

- **`REGISTER.md`** *(you write one per project — see `TEMPLATE.md`)* — a short, positive
  statement of the project's register: what it is in one sentence, the vocabulary by area, a
  plain "words we reach for" list, and the instinct for naming the next new thing. It is
  dual-purpose: a primer that makes a session write in-register from the start, and the allowlist
  the scanner reads.

- **`register-scan`** — a dependency-free (stdlib) drift detector. It carries **no** word-list of
  its own; it ranks the tokens in your code that are **far from your declared register**, so it
  points at drift without ever naming a "bad" word. Two modes:

  ```sh
  # whole-tree: where has a codebase drifted from its register?
  register-scan path/to/project REGISTER.md

  # 'new material': check only what a change touched (the leading effectiveness signal)
  git diff --name-only HEAD~5 | register-scan --stdin REGISTER.md
  ```

- **`SKILL.md`** — the method, as a Claude Code skill: establish the register, scan for drift,
  map fixes by blast radius (comments are free; identifiers rename with their references; on-disk
  and cross-module string values need a back-compat read shim), migrate the loaded material out of
  the working context (delegate it), verify, and bake the register in.

## Two directions

The same `REGISTER.md` works both ways:

- **Discriminative** — `register-scan` catches drift after the fact (run it in a pre-commit hook
  or periodically).
- **Generative** — put `REGISTER.md` in front of a session (a project `CLAUDE.md` pointer, a
  sub-agent's first read, or a session-start hook) and it writes in-register from the start.
  **Prime positively; never proscribe** — a "don't say X" prompt both injects the vocabulary you
  want gone and makes the model fixate on it.

## The dual-use note

When you name a mechanism, prefer a plain description over a vivid term of art — *especially* when
the vivid word names causing harm as readily as preventing it. Such words read the same whether
you build a thing or guard against it, so the plain description is clearer and less easily mistaken
for what it isn't. Words that only name protecting, checking, or noticing carry no such ambiguity.
Where a project genuinely works in a charged domain, that domain's own terms are its plain words —
the instinct is against importing a *foreign* domain's drama, not against a domain's native
vocabulary.

## Install (Claude Code)

```sh
ln -s "$PWD" ~/.claude/skills/register          # the skill
ln -s "$PWD/register-scan" ~/bin/register-scan  # the detector on PATH
```

Then, in any project: write a `REGISTER.md` from `TEMPLATE.md`, and point your work at it.
