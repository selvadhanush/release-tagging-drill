# Tag Audit

This document audits the tags that existed in the repository before this
branch's cleanup, based on the output of:

```
git tag
git log --oneline --decorate --all
git tag --sort=-v:refname
```

Raw findings (commit each tag actually points to):

| Tag | Type | Commit | Points at |
|---|---|---|---|
| `version-1.0` | annotated | `28bd110` | docs: add broken deployment history and incident log |
| `release_2` | lightweight | `28bd110` | docs: add broken deployment history and incident log |
| `v1.4.2` | lightweight | `ff1fb9a` | docs: scaffold assignment repository |
| `stable-build` | annotated | `ff1fb9a` | docs: scaffold assignment repository |
| `latest-good` | annotated | `da6b2ed` | docs: add broken deployment history and incident log |
| `1.5.0` | annotated | `86989b6` | docs: add old release notes and changelog |
| `v2-final-FINAL` | lightweight | `82535d9` (HEAD) | feat: add dummy checkout service |
| `patch-new` | lightweight | `82535d9` (HEAD) | feat: add dummy checkout service |

## Problem 1: `release_2` and `version-1.0` are two different names for the same commit

**Evidence:** both tags point at `28bd110`.

**What it means for the team:** there is no single source of truth for what
this commit is called. One engineer might deploy "release_2", another might
reference "version-1.0" in a ticket, and nobody can tell from either name
alone that they are the same artifact.

**Risk:** if an incident report says "roll back to version-1.0" and the
on-call engineer only knows the deployment as `release_2`, the rollback is
delayed while people figure out the two tags are aliases. Auditing which tag
was actually deployed to production becomes guesswork.

## Problem 2: `v2-final-FINAL` is a non-semantic, "final-final" style tag

**Evidence:** `v2-final-FINAL` is a lightweight tag on `82535d9` (current
HEAD).

**What it means for the team:** the name tells you nothing about what
changed relative to any other release — is this a major version 2, or just
"the second time we thought we were final"? It's also lightweight, so it
carries no message, author, or date metadata (`git cat-file -t` shows
`commit`, not `tag`).

**Risk:** there is no annotation to explain why this was cut, and the name
gives false confidence ("FINAL" implies stable/shippable) with zero evidence
behind that claim. Rolling back *from* this tag, or auditing why it was
created, requires digging through unrelated commit messages instead of
reading the tag itself.

## Problem 3: `stable-build` points at a commit that is missing a later feature

**Evidence:** `stable-build` (annotated, message: "Annotated but misleading
stable build tag") points at `ff1fb9a`, a docs-only scaffold commit. The
checkout-service feature (`src/app.js`) is only fully present again at the
later commit `82535d9`, which is *not* what `stable-build` points to.

**What it means for the team:** the name "stable" implies "safe to deploy /
safe to roll back to," but rolling back to `stable-build` would silently
change the feature set on production without a clear changelog entry saying
so.

**Risk:** a rollback done in a hurry during an incident, based on the name
alone, could re-introduce a regression or drop a feature customers depend on,
because the tag name promises stability it doesn't document.

## Problem 4: `latest-good` does not point at the most recent commit

**Evidence:** `latest-good` (annotated, message: "Annotated rollback
candidate, but points to an older commit") points at `da6b2ed`, while `main`
has three commits after it (`86989b6`, and HEAD `82535d9`).

**What it means for the team:** the name claims currency ("latest") but the
tag itself admits it is stale. Anyone trusting the name without reading the
annotation would assume this is the newest verified-good state, when newer
commits (and possibly newer, better-tested releases) already exist.

**Risk:** during an incident, rolling back to what's labeled "latest-good"
could mean rolling back further than necessary, discarding good, later work
unnecessarily, or — worse — the name could be trusted over the actual date,
causing the team to skip checking whether a newer good release exists.

## Problem 5: `1.5.0` and `v1.4.2` mix tag-naming formats and invert chronology vs. version number

**Evidence:** `1.5.0` (no `v` prefix) is on `86989b6`. `v1.4.2` (`v` prefix)
is on `ff1fb9a`, which is chronologically *earlier* in the log than
`86989b6`. Meanwhile the actual newest code (`82535d9`, the checkout-service
feature) is tagged only with the non-semver names `v2-final-FINAL` /
`patch-new`.

**What it means for the team:** there is no single consistent tag format, so
tools that rely on `git tag --sort=-v:refname` or `git describe` to find "the
latest release" don't reliably work — mixed `v`/no-`v` prefixes sort
inconsistently, and the numerically "highest" semver tag (`1.5.0`) is a
docs-only commit, not the commit with the most recent feature work.

**Risk:** an automated deploy or rollback script that trusts the highest
semver tag would deploy `1.5.0` believing it's the newest, most feature-complete
release, when in fact the checkout-service feature in `82535d9` isn't
included in it at all — a silent feature regression with no error to flag
it.

## Problem 6 (bonus): four different informal tags (`stable-build`,
`latest-good`, `release_2`, `patch-new`) exist with no defined ordering rule
between them

**Evidence:** none of these four tags follow semantic versioning, so there is
no command (`git tag --sort=-v:refname` or otherwise) that can tell you which
one is meant to supersede another.

**What it means for the team:** "which of these do I deploy?" has no
objective answer — it depends on tribal knowledge of who made which tag and
when.

**Risk:** this is the core traceability failure the rest of this assignment
fixes: without an ordering convention, "what's in production" and "what do I
roll back to" cannot be answered from the tag list alone, only by manually
re-deriving context from commit messages and dates every time.
