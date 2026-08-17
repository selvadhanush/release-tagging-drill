# Release Notes

Release notes for each structured tag defined in [`RELEASE-MAP.md`](./RELEASE-MAP.md).
Each entry turns a tag (a commit pointer) into traceability (what changed,
why, and what to roll back to if it fails).

## v1.0.0 — 2026-06-06

**Summary:** Initial baseline release. Establishes the repository scaffold.

- **Added:** `README.md` — project scaffold.

**Rollback target:** none (this is the baseline; there is nothing earlier to
roll back to).

## v1.1.0 — 2026-06-06

**Summary:** First feature release, adding the checkout service.

- **Added:** `src/app.js` — dummy checkout service.

**Rollback target:** `v1.0.0` — if the checkout service causes a failure,
revert to the pre-checkout-service baseline.

## v1.1.1 — 2026-06-06

**Summary:** Documentation and scaffold patch release. No functional/code
changes to the checkout service.

- **Added:** `CHANGELOG.md`, `docs/deployment-history.md`,
  `docs/incident-log.md`, `docs/release-notes-old.md`, `src/README.md`.
- **Changed:** `README.md` updated as part of the same scaffold pass.
- **Fixed:** N/A — this release adds documentation scaffolding, it does not
  fix a defect in `v1.1.0`'s code.

**Rollback target:** `v1.1.0` — since this release is docs-only, rolling
back here does not remove any application functionality, only the added
documentation files.
