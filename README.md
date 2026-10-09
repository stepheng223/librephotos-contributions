# Open Source Contribution Log

**Contributor:** [stepheng223](https://github.com/stepheng223)
**Contribution repository:** [librephotos-contributions](https://github.com/stepheng223/librephotos-contributions)
**Project fork:** [stepheng223/librephotos](https://github.com/stepheng223/librephotos)
**Project:** [LibrePhotos](https://github.com/LibrePhotos/librephotos)
**Issue:** Remove orphaned thumbnail files when deleting missing photos
**Related GitHub issue:** [#19 — Manage photos](https://github.com/LibrePhotos/librephotos/issues/19) (historical context; closed)
**Issue documentation:** [Remove orphaned thumbnail files when deleting missing photos](https://docs.librephotos.com/docs/development/contribution/backend/missing-photos)
**Status:** PR merged; Phase IV course check-in pending confirmation
**Pull request:** [#2078](https://github.com/LibrePhotos/librephotos/pull/2078)
**Last verified:** October 9, 2026 (PR status, published acceptance checklist, maintainer mention, and release credit; earlier CI verification is dated below)

**Course submission guidance:** Edits are accepted. Slack posting is not required, as confirmed by the contributor.

**Grading evidence:** [Criterion-by-criterion rubric map](RUBRIC-EVIDENCE.md), including verified evidence and outstanding requirements for Units 1–4.

## Proposed Solution

The initial implementation used a Django `post_delete` signal and field storage deletion. The merged implementation defers cleanup until transaction commit, protects files still referenced by other thumbnails or photos, and reuses the project’s `delete_thumbnail_files(image_hash)` helper. See Phase IV for the change in approach and attribution.

## Phase I: Issue Selection

### Phase I Evidence

Related issue #19 describes original-photo deletion leaving thumbnail/database data behind. PR #2078 addresses the narrower orphaned-thumbnail cleanup problem. This reference was identified during the submission audit; it is not confirmed as the original selected issue, and its broader photo-management request is not claimed as resolved by this PR.

The contribution repository and project fork are public. The task originated in a documentation TODO; a live numbered issue and introductory issue comment were not established. This historical Phase I gap remains even though the contribution merged; email-validation #695 must not be used as its issue reference. Course staff must confirm whether the merged contribution satisfies the original issue-selection requirement.

### Why I Chose This Issue

When LibrePhotos deletes missing photos, database cascade removes thumbnail records but originally left their stored files behind, wasting disk space. I chose this bounded cleanup task to apply my documented Python/Django, Docker Compose, Git, and regression-testing work. My learning goal was to understand Django deletion signals and coordinate database changes with file cleanup. Success means removing assets only after committed deletion and only when no surviving photo still needs them.

## Phase II: Reproduction and Solution Planning

**Scope:** All phases in this README concern thumbnail cleanup, the contribution merged in PR #2078. Email-validation #695 was researched later as an alternative and is preserved in [EMAIL-VALIDATION-CANDIDATE.md](EMAIL-VALIDATION-CANDIDATE.md); it was not implemented in this PR.

**Working branch:** [fix/remove-orphaned-thumbnails](https://github.com/stepheng223/librephotos/tree/fix/remove-orphaned-thumbnails). Local inspection on October 6, 2026 confirmed this branch and implementation commit `44080a45`; GitHub’s PR metadata confirms the published source branch and final head `ca8f03db3562020b0dd04089069e197ca2633c80`.

### Reproduction Process

#### Environment Setup

I used the LibrePhotos Docker Compose development environment on macOS. The backend is located in `apps/backend/`, and the Compose files are in `deploy/compose/`.

Setup path: the project's [Development Installation instructions](https://docs.librephotos.com/docs/development/dev-install/) using Docker Compose, rather than a devcontainer or CI-only inspection. The setup events and test results below are preserved from the earlier contribution log; they were not rerun during this documentation revision.

During setup, Docker initially failed because an existing `frontend` container conflicted with the Compose service name. I confirmed it belonged to the same Compose project, removed only that stale container, and restarted the stack with:

```bash
docker compose \
  -f deploy/compose/docker-compose.yml \
  -f deploy/compose/docker-compose.dev.yml \
  up -d
```

The backend, database, frontend, proxy, and pgAdmin services then started successfully.

#### Steps to Reproduce

1. Start the LibrePhotos development environment.
2. Add a test image to the configured scan directory.
3. Scan the library and wait for thumbnails to be generated.
4. Confirm files exist under `protected_media/thumbnails_big/`, `protected_media/square_thumbnails/`, and `protected_media/square_thumbnails_small/`.
5. Remove the original image from the scan directory.
6. Run the Scan Missing Photos job.
7. Run the Delete Missing Photos job.
8. Inspect the thumbnail directories again.

**Expected result:** The photo, thumbnail record, and all associated thumbnail files are removed.

**Actual result before the fix:** The photo and thumbnail database records were removed, but the physical thumbnail files remained on disk.

### Understanding the Issue

The database cascade removes thumbnail records but originally left separately stored assets behind. Cleanup must also account for transaction rollback and other records referencing the same image hash.

### Solution Approach

#### Understand

Root cause: `Thumbnail.photo` in `apps/backend/api/models/thumbnail.py:20` uses `on_delete=models.CASCADE`, so deleting a photo removes the thumbnail database row. That cascade does not by itself remove the files referenced by the three `ImageField` values. `delete_missing_photos()` in `apps/backend/api/autoalbum.py:217` deletes photo batches with `batch_qs.delete()`, which explains why database cleanup can succeed while files remain.

The hypothesis is supported by inspecting the parent of fix commit `44080a45`: that version of `thumbnail.py` has no thumbnail deletion receiver. The existing fix adds `auto_delete_files_on_delete()` at line 194. This establishes the missing hook in the inspected revision; it does not establish when the bug was first introduced.

#### Match

An analogous existing implementation is `auto_delete_file_on_delete()` in `apps/backend/api/models/face.py:92`. It receives a `post_delete` signal for `Face` and removes its associated image file. That is the same lifecycle problem: database deletion must trigger cleanup of a separately stored asset. For thumbnails, use `field_file.storage.delete(field_file.name)` so the code respects the field's storage backend rather than copying the face receiver's local `os.remove()` assumption.

#### Plan

Modify `apps/backend/api/models/thumbnail.py` to register a `Thumbnail` deletion receiver and clean up `thumbnail_big`, `square_thumbnail`, and `square_thumbnail_small`. Extend `apps/backend/api/tests/photos/test_delete_missing_photos.py` with a regression test that creates all three stored files, invokes `delete_missing_photos()`, and checks both record deletion and file absence. Keep the change focused on thumbnail cleanup; no scan pipeline redesign or schema migration is planned.

#### Implement

Phase III contains the implementation record. Local inspection confirmed that commit `44080a45` already contains the receiver and `DeleteMissingPhotosThumbnailCleanupTest.test_deleting_missing_photo_removes_thumbnail_files`; implementation is no longer merely an uncommitted proposal.

#### Review

The following table records the initial review questions. The final PR resolved rollback and shared-reference risks through maintainer-authored commits described in Phase IV:

| Edge case | Why it matters / proposed verification |
| --- | --- |
| Empty thumbnail fields | Skip empty names; deleting the record must not call storage deletion with an empty path. |
| File already missing | Confirm the configured storage backend's missing-file behavior; cleanup should tolerate an already-removed asset where supported. |
| Database transaction rollback | The initial receiver deleted files immediately. A later rollback could restore rows without their files; assess deferring cleanup with `transaction.on_commit()` and test the agreed lifecycle. |
| Shared filename | Check whether another thumbnail record can reference the same stored name before deleting it; record deletion must not damage another photo's assets. |
| Storage permission/network failure | Decide how to log, report, or retry cleanup errors. Do not silently report successful cleanup when files remain. |
| Video preview files | The square fields may reference MP4 assets; delete their recorded names rather than assuming every file ends in WebP. |
| Other user's photos | Ensure the deletion job and receiver leave surviving users' records and files intact. |

These are investigative findings and planned checks, not claims that every case is already covered. The existing regression test covers the normal path for all three fields.

#### Evaluate

Acceptance criteria: deleting a missing photo removes its `Photo` and `Thumbnail` rows and all three referenced files, while surviving photos retain their assets. Run the focused cleanup regression and the existing missing-photo test module; inspect job completion/error behavior as well as assertions, since the job catches exceptions internally. The earlier focused test result is recorded in Phase III. The final PR adds edge-case tests and passed the full backend suite in CI; see Phase III’s final verification links.

#### Investigation Evidence

Read-only commands run during this revision:

```bash
git -C /Users/stephen/librephotos log -3 --oneline -- apps/backend/api/models/thumbnail.py
git -C /Users/stephen/librephotos show -s --format='%h %ad %s' --date=iso-strict 44080a45
git -C /Users/stephen/librephotos show 44080a45^:apps/backend/api/models/thumbnail.py
```

Finding: fix commit `44080a45` is dated September 23, 2026; its parent lacks the cleanup receiver. The preceding thumbnail-file commit is `02c962fe`, dated August 15, 2026. This bounds the inspected pre-fix state, rather than claiming that August 15 introduced the defect.

#### Phase II Submission

- [x] Working branch, setup path, setup challenges, reproduction, expected/actual behavior, and file locations documented.
- [x] Understand, Match, Plan, Review, and Evaluate sections documented.
- [x] Stretch evidence added: analogous receiver, history inspection, and proactive edge-case analysis.
- [ ] Confirm which issue the course-approved Phase II submission follows.
- [x] Revised documentation pushed to the contribution repository.
- [ ] Submit the matching repository link in the Course Portal with **Phase II Complete** selected.

#### Implementation Plan

1. Add deletion cleanup to `apps/backend/api/models/thumbnail.py`.
2. Delete `thumbnail_big`, `square_thumbnail`, and `square_thumbnail_small` through each field's configured storage backend.
3. Ignore empty fields and allow storage backends to handle files that are already absent.
4. Add a regression test in `apps/backend/api/tests/photos/test_delete_missing_photos.py`.
5. Verify that missing-photo deletion removes both database records and physical thumbnail files.

## Phase III: Implementation and Testing

### Implementation Notes

Implemented the thumbnail cleanup signal in `apps/backend/api/models/thumbnail.py`:

- Registered a Django `post_delete` receiver for `Thumbnail`.
- Deletes the files referenced by all three thumbnail fields.
- Uses each field's storage backend instead of assuming a local filesystem.
- Skips empty file fields.

Added `DeleteMissingPhotosThumbnailCleanupTest` in `apps/backend/api/tests/photos/test_delete_missing_photos.py`:

- Creates a photo and thumbnail record.
- Writes test files to all three thumbnail fields.
- Runs `delete_missing_photos()`.
- Confirms the `Photo` and `Thumbnail` records are deleted.
- Confirms all three physical thumbnail files are deleted.

### Code Changes

Development branch: [fix/remove-orphaned-thumbnails](https://github.com/stepheng223/librephotos/tree/fix/remove-orphaned-thumbnails)

Local inspection on October 6, 2026 confirmed the implementation and regression test in commit `44080a45` on this branch. The published PR subsequently merged with maintainer follow-up commits listed in Phase IV.

Relevant files:

- `apps/backend/api/models/thumbnail.py`
- `apps/backend/api/tests/photos/test_delete_missing_photos.py`

### Implementation Progress

Initial student implementation: [44080a45](https://github.com/LibrePhotos/librephotos/commit/44080a451b058f89b5f5ec570c8ccf4748b49135), September 23, 2026. In the inspected initial checkout, `apps/backend/api/models/thumbnail.py:193` registers the receiver and `apps/backend/api/tests/photos/test_delete_missing_photos.py:46` defines the new cleanup test. The test uses existing `create_test_user()` and `create_test_photo()` helpers from `api.tests.utils`, matching neighboring Django TestCase conventions. Six PR commits are present, but five follow-up commits are Niaz-authored; they do not prove a six-commit student cadence.

### Testing Strategy

Focused Docker test command:

```bash
docker exec -e NO_COVERAGE=1 backend python manage.py test \
  api.tests.photos.test_delete_missing_photos.DeleteMissingPhotosThumbnailCleanupTest
```

Result:

```text
Ran 1 test in 0.276s
OK
```

Additional validation:

- Python compilation check passed for the changed Python files.
- `git diff --check` passed.
- Django system checks passed during the focused test.
- The Docker database service was healthy when the test ran.

The initial test attempt exposed that the production image did not include development dependencies. Installing `apps/backend/requirements.dev.txt` in the running container resolved the missing `coverage` and `faker` packages. The test was then rerun successfully with coverage disabled using `NO_COVERAGE=1`.

### Challenges Faced

The earlier Compose startup failed because an existing `frontend` container conflicted with the service name. After confirming it belonged to the same project, the stale container was removed and the development stack restarted. The focused test also lacked `coverage` and `faker`; installing backend development requirements and rerunning with `NO_COVERAGE=1` resolved that recorded attempt. On October 6, Docker was stopped (socket absent), so a fresh full-suite run could not be performed.

### Final CI Verification

On September 26, 2026, the final PR head `ca8f03db3562020b0dd04089069e197ca2633c80` passed:

- [Full backend suite — Django job](https://github.com/LibrePhotos/librephotos/actions/runs/36229563136/job/108370075600): the successful Run tests step executes `python manage.py test api.tests` with the project’s SQLite test settings.
- [Backend lint and formatting — Ruff job](https://github.com/LibrePhotos/librephotos/actions/runs/36229563194/job/108370079719): runs `ruff check apps/backend` and `ruff format --check apps/backend`.

These are upstream CI results, verified through GitHub’s job/check APIs on October 6; they are not a fresh local run. The prior manual walkthrough documents orphaned files before the fix. No new post-fix browser walkthrough or screenshot is claimed.

## Phase III Completion

**Phase III Complete.** The implementation is working, includes a regression test, and has been validated in the Docker development environment. PR #2078 subsequently merged. The focused result above concerns the initial implementation; the separate final CI evidence above establishes the full-suite pass for the final maintainer-modified version.

## Phase IV: Pull Request and Review

### Pull Request

**Link:** [LibrePhotos/librephotos#2078 — fix: remove orphaned thumbnail files](https://github.com/LibrePhotos/librephotos/pull/2078)

**Status:** Merged

**Submitted:** September 23, 2026

**Merged:** September 26, 2026, by `derneuere`

**Base:** `LibrePhotos/librephotos:dev`, the upstream default branch

**Source:** `stepheng223/librephotos:fix/remove-orphaned-thumbnails`

**Merge commit:** [28dd0b61685102d243a91500035d2918310e11be](https://github.com/LibrePhotos/librephotos/commit/28dd0b61685102d243a91500035d2918310e11be)

**Summary:** The PR cleans up orphaned thumbnail assets after photo deletion. The merged version waits until the transaction commits and protects shared assets before using the existing hash-based deletion helper.

**Release evidence:** [LibrePhotos 1.2.0 release notes](https://github.com/LibrePhotos/librephotos/releases/tag/1.2.0) credit `stepheng223` for removing orphaned thumbnail files. Verified October 9, 2026.

### Change in Approach and Final Implementation

The initial student commit [44080a45](https://github.com/LibrePhotos/librephotos/commit/44080a451b058f89b5f5ec570c8ccf4748b49135) introduced the receiver and a cleanup regression test. The final PR includes additional work authored by Niaz; these changes must not be attributed to the student:

| Date | Commit | Change |
| --- | --- | --- |
| September 26, 2026 | [4aad4586](https://github.com/LibrePhotos/librephotos/commit/4aad458626a70a28a16c8bb632532f533724df06) | Removed the course contribution log from the upstream diff. |
| September 26, 2026 | [62e244fe](https://github.com/LibrePhotos/librephotos/commit/62e244fe18ca9a6ed3d7dc2934ebbaeabe83361b) | Deferred deletion using `transaction.on_commit`, protected shared references, handled storage errors, and added transaction/shared-hash/error tests. |
| September 26, 2026 | [a75ffcfe](https://github.com/LibrePhotos/librephotos/commit/a75ffcfef3a12bb3708c7a9591b019b492c75b6c) | Updated missing-photo documentation. |
| September 26, 2026 | [ca8f03db](https://github.com/LibrePhotos/librephotos/commit/ca8f03db3562020b0dd04089069e197ca2633c80) | Reused `delete_thumbnail_files(image_hash)` for orphan cleanup. |

Phase II's immediate field-storage deletion was the initial plan. The shipped approach instead uses transaction-safe, reference-aware, hash-based cleanup. The final helper handles WebP and MP4 variants under `MEDIA_ROOT`; it should not be described as a generic remote-storage implementation.

### Maintainer Feedback and Responses

GitHub's API showed no issue comments or formal PR reviews when checked October 6, 2026. Therefore there is no written feedback thread or student response commit to log. The contribution was submitted September 23 and subsequently revised through the Niaz-authored commits above, then merged by `derneuere` on September 26. This demonstrates maintainer engagement, but does not establish a prior reviewer request or student-authored revision loop.

| Date | Activity / response | Evidence |
| --- | --- | --- |
| September 23, 2026 | Submitted initial implementation and regression test; no formal review feedback was established by the October 6 audit. | Student commit [44080a45](https://github.com/LibrePhotos/librephotos/commit/44080a451b058f89b5f5ec570c8ccf4748b49135). |
| September 26, 2026 | Maintainer-authored improvements followed by merge; these are not student response commits. | Follow-up commits listed above and merge [28dd0b61](https://github.com/LibrePhotos/librephotos/commit/28dd0b61685102d243a91500035d2918310e11be). |
| October 9, 2026 | Student mentioned `@derneuere`, thanked maintainers, documented the transactional/shared-reference lesson, and reported the completed acceptance checklist. | [Published maintainer mention](https://github.com/LibrePhotos/librephotos/pull/2078#issuecomment-6091218100). This occurred after merge, not as an original review request; no new maintainer response is claimed. |

### PR Description and Acceptance Evidence

The substantive description was published and verified on [PR #2078](https://github.com/LibrePhotos/librephotos/pull/2078) on October 6, 2026. Its local source is [PR-2078-DESCRIPTION.md](PR-2078-DESCRIPTION.md). No numbered issue was established for the original documentation TODO, so no unrelated `Closes #...` reference is claimed. Resolve the issue-reference requirement with the course staff rather than link this merged change to email-validation #695.

The completed compatibility checkbox was published to the live PR body and verified through GitHub's API on October 9, 2026. All acceptance checklist items in that description are now checked.

- [x] PR exists, is not a draft, and merged into the upstream default branch.
- [x] Student implementation and new test identified by commit.
- [x] Final approach and maintainer-authored changes documented.
- [x] Previously recorded focused backend test output included in Phase III.
- [x] Published a substantive PR description with problem context, changes, issue-reference limitation, acceptance checklist, and backend evidence. No project PR template was found in the inspected checkout.
- [ ] Establish a valid issue reference for the thumbnail task if required for resubmission.
- [x] Full backend suite and Ruff style checks passed for the final PR head; CI links recorded above.
- [x] No breaking API or schema changes: inspected the final [PR file diff](https://github.com/LibrePhotos/librephotos/pull/2078/files) on October 9, 2026. The three changed files contain the thumbnail deletion receiver, regression tests, and missing-photo documentation; there are no migrations, model-field changes, endpoint changes, or dependency changes. The intended behavior change is removal of unreferenced thumbnail files after committed deletion. Existing full-suite CI evidence is linked in Phase III; this inspection is not a new test run.
- [x] Maintainer mentioned: [October 9 comment tagging `@derneuere`](https://github.com/LibrePhotos/librephotos/pull/2078#issuecomment-6091218100), posted after merge.
- [x] Revised README pushed to the contribution repository.
- [ ] Submit the course check-in with **Phase IV Complete** selected.

### Resubmission Requirements

The README now includes the PR link, summary, Merged status, problem context, backend evidence, a dated maintainer-activity record, all three reflection subsections, and the documented change from the original plan to the merged implementation. These address the missing README sections identified in the Unit 4 feedback.

The remaining requirements cannot be completed by changing README text alone:

| Requirement | Current evidence and next step |
| --- | --- |
| Valid issue-closing reference | The task came from the project's documentation TODO. Ask course staff whether this source is acceptable instead of a numbered issue; no matching issue has been established. |
| Phase IV course check-in | Submit the repository link in the Course Portal with **Phase IV Complete** selected, then record the submission date or confirmation. |

Course check-in details ready to submit:

- **Repository:** https://github.com/stepheng223/librephotos-contributions
- **Phase:** Phase IV Complete
- **Pull request:** https://github.com/LibrePhotos/librephotos/pull/2078
- **Status:** Merged
- **Summary:** Removes orphaned thumbnail files after committed photo deletion while preserving files still referenced by surviving photos; includes regression tests and passing backend/style CI.

Issue-source clarification for course staff: “My merged PR #2078 implements the orphaned-thumbnail cleanup TODO from the project's missing-photo documentation. I have not established a matching numbered issue. Can that documented task satisfy the issue-reference requirement for this contribution?” This clarification has not been sent and no exception is claimed.

After these items are resolved, request reassessment using this updated repository. The previous grading feedback repeatedly described the PR as not submitted and the README sections as missing; the current README supplies evidence for those sections without claiming the unresolved requirements are complete.

## Learnings & Reflections

### Technical Skills Gained

The contribution illustrates how Django database deletion and stored-file lifecycle are separate responsibilities. A cascade can remove a row while leaving its assets behind. The initial fix and regression test connected those responsibilities; the merged changes show why file cleanup must also respect transaction boundaries and shared ownership.

### Challenges Overcome

The recorded setup required resolving a conflicting Docker container and missing development-test dependencies. The initial implementation also missed rollback and shared-hash cases. The final PR addressed those through maintainer-authored changes, which improved the result and showed the limits of a normal-path test.

### What I'd Do Differently

Establish the live issue, acceptance criteria, PR description, and course check-in evidence before submission. Investigate file naming and transaction behavior before choosing the cleanup hook. Keep the course journal in its own repository, test rollback/shared-reference scenarios early, and record the exact commit and test output for each version.

A useful lesson for future contributors: a deleted database record does not necessarily mean its file is unowned. Check both transaction completion and surviving references before removing shared assets. Also distinguish your own commits from maintainer improvements when explaining what you learned.

## Completion Audit

Verified on October 6, 2026: public contribution repo and fork, substantive phase sections, initial student implementation and regression test, final full-suite/style CI success, upstream merge, final change attribution, and reflections.

Completed October 9, 2026: final-diff compatibility inspection, publication and verification of the completed live PR checklist, a linked maintainer mention, and verification of the official 1.2.0 release credit. Documentation revisions are published in the contribution repository.

Still requiring evidence or action:

- Original numbered issue and introductory comment: absent; request course guidance for this merged documentation-TODO contribution.
- Historical student commit cadence: cannot be created retroactively.
- Slack participation: not required under the contributor’s confirmed course guidance; no post is claimed.
- Course Google Sheet and Phase I/II/IV check-ins: require confirmation or submission in the course accounts. The supplied Phase III review already awards its check-in.

The code contribution is merged and CI verified. Coursework is not fully complete until the outstanding submission requirements are resolved.
