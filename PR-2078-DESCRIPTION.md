# Published description for PR #2078

The body below was published and verified on PR #2078 on October 6, 2026.

Revision published and verified through GitHub's API on October 9, 2026: completed the compatibility checklist after inspecting the final PR diff. The body below matches the live PR description.

## Why

Deleting missing photos removed their database records but left thumbnail assets on disk, consuming storage. The original implementation had no thumbnail deletion receiver. Investigation also showed that filenames are shared by image hash, so deleting a row does not necessarily make its files safe to remove.

## What does this PR do?

The student implementation adds thumbnail deletion cleanup and a regression test. Maintainer-authored follow-up commits defer cleanup until transaction commit, protect assets still in use, add edge-case coverage, and reuse the existing hash-based deletion helper. Course-only documentation was removed from the upstream diff.

## Issue references

Original task: the missing-photo documentation TODO. No matching numbered GitHub issue has been established. Supply a valid issue reference only after verifying that it covers this contribution; #695 concerns email validation and is unrelated.

## Acceptance criteria

- [x] Adds a new regression test exercising missing-photo thumbnail cleanup.
- [x] Final code includes rollback and shared-reference protections.
- [x] Cleanup reuses the existing project helper.
- [x] Full backend suite passed for the final PR head in [GitHub Actions](https://github.com/LibrePhotos/librephotos/actions/runs/36229563136/job/108370075600).
- [x] Backend Ruff lint/format checks passed in [GitHub Actions](https://github.com/LibrePhotos/librephotos/actions/runs/36229563194/job/108370079719).
- [x] No breaking API or schema changes: the final [PR diff](https://github.com/LibrePhotos/librephotos/pull/2078/files), inspected October 9, changes only thumbnail cleanup, regression tests, and documentation. There are no migrations, model-field changes, endpoint changes, or dependency changes. Removing unreferenced thumbnail files after committed deletion is the intended behavior change; final backend-suite CI evidence is linked above.

## Backend evidence

Before the initial fix, the earlier contribution log records that photo and thumbnail rows were removed while physical thumbnail files remained. The initial cleanup regression writes all three field assets, calls the production deletion job, and asserts absence of both database rows and files.

Previously recorded initial-version focused test:

```text
Ran 1 test in 0.276s
OK
```

This output has not been reproduced for the final merged implementation. Final-version backend suite and style-check success are linked above.

## Review and final status

Merged into `LibrePhotos/librephotos:dev` by `derneuere` on September 26, 2026. Implementation and test commit: `44080a451b058f89b5f5ec570c8ccf4748b49135`; final follow-up: `ca8f03db3562020b0dd04089069e197ca2633c80`. Follow-up authorship is credited in the PR commit history.
