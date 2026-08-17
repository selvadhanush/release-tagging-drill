# Versioning Convention

This document defines how the team tags releases going forward, so that a
tag alone answers "what's in this release" and "what do I roll back to"
without archaeology through commit messages. See [`TAG-AUDIT.md`](./TAG-AUDIT.md)
for the problems this replaces.

## 1. Semantic versioning rules

We follow [SemVer 2.0.0](https://semver.org/): `MAJOR.MINOR.PATCH`.

- **MAJOR** — incremented for breaking / incompatible changes (an API
  consumer or deployment process would need to change to keep working).
  Example: `v1.4.2 -> v2.0.0` when the checkout API's request/response shape
  changes in a way old clients can't parse.
- **MINOR** — incremented for backward-compatible new functionality.
  Example: `v1.0.0 -> v1.1.0` when the checkout service feature is added
  without changing any existing behavior.
- **PATCH** — incremented for backward-compatible bug fixes only, no new
  features. Example: `v1.1.0 -> v1.1.1` for a fix to the scaffold/docs that
  doesn't add or remove functionality.

Rule of thumb: if it breaks someone, it's MAJOR. If it adds something new and
nothing breaks, it's MINOR. If it only fixes something that was already
there, it's PATCH.

## 2. Tag naming format

All release tags use the pattern:

```
vMAJOR.MINOR.PATCH
```

- Always lowercase `v` prefix, no exceptions (this is what makes
  `git tag --sort=-v:refname` and `git describe` sort and match reliably).
- No suffixes on a final release tag (no `-final`, no `-FINAL`, no
  descriptive words — see Problem 2 in `TAG-AUDIT.md`).

Example: `v1.1.0`

## 3. Annotated vs. lightweight tags

**The team uses annotated tags for every release.** Annotated tags are real
Git objects that store the tagger, date, and a message — that message is
where the "what changed and why" traceability lives. Lightweight tags are
just pointers with no metadata, which is exactly what made tags like
`v2-final-FINAL` and `patch-new` untrustworthy in the audit (no record of who
cut them or why).

Command used for every release tag:

```
git tag -a vMAJOR.MINOR.PATCH -m "Release MAJOR.MINOR.PATCH: <what changed>"
```

Lightweight tags (`git tag vX.Y.Z` with no `-a`/`-m`) are not used for
releases.

## 4. Pre-release rule

Release candidates and betas are marked with a hyphenated pre-release suffix
per SemVer, e.g. `v1.5.0-rc.1`, `v1.5.0-rc.2`, `v1.5.0-beta.1`.

Ordering: pre-release tags sort **before** their final release. For example:

```
v1.5.0-beta.1 < v1.5.0-rc.1 < v1.5.0-rc.2 < v1.5.0
```

So `git tag --sort=-v:refname` naturally lists the final `v1.5.0` above all
of its release candidates, and a release candidate is never mistaken for the
final, production-ready release.
