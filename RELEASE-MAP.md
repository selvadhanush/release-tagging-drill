# Release Map

This maps each structured release tag (created per [`VERSIONING.md`](./VERSIONING.md))
to its commit and what it shipped. This table is the answer to "what's in
production" and "what do I roll back to."

| Version (tag) | Commit | Date | What Shipped |
|---|---|---|---|
| `v1.0.0` | `4febcc6` | 2026-06-06 | Initial project scaffold — `README.md` only. Baseline release. |
| `v1.1.0` | `c17d4c4` | 2026-06-06 | Added the checkout service feature (`src/app.js`). MINOR: new functionality, backward-compatible. |
| `v1.1.1` | `ff1fb9a` | 2026-06-06 | Docs/scaffold cleanup — added `CHANGELOG.md`, `docs/deployment-history.md`, `docs/incident-log.md`, `docs/release-notes-old.md`, `src/README.md`, updated `README.md`. PATCH: no functional changes. |

Current production target: **`v1.1.1`** (most recent tag on this sequence).

## Traceable rollback

With the tags above, identifying and rolling back to the last known-good
release is a fixed procedure, not guesswork:

```bash
# 1. See the full sortable release history, newest first
git tag --sort=-v:refname

# 2. Confirm what the current release contains
git show v1.1.1

# 3. If v1.1.1 fails, the previous known-good release is v1.1.0 —
#    check it out and redeploy
git checkout v1.1.0
```

Because every tag here is annotated, semver-formatted, and created on a
commit whose content matches what the tag claims, `git tag --sort=-v:refname`
alone reconstructs the true release order — unlike the original tag set
audited in `TAG-AUDIT.md`, where `1.5.0` (no `v` prefix) numerically
outranked `v1.4.2` despite being chronologically unrelated to the newest
feature commit, and names like `latest-good` or `stable-build` didn't
actually point at the most recent or most complete state. Rolling back here
means running one `git checkout <tag>` against a tag whose name, order, and
annotation message all agree — no cross-referencing commit messages or
guessing which of several aliases for the same commit is authoritative.
