# Open Source Contribution Log

**Project:** LibrePhotos
**Issue:** Remove orphaned thumbnail files when deleting missing photos
**Issue documentation:** [Remove orphaned thumbnail files when deleting missing photos](https://docs.librephotos.com/docs/development/contribution/backend/missing-photos)
**Status:** Phase III complete; ready for Phase IV pull request work

## Proposed Solution

When a `Thumbnail` record is deleted, use a Django `post_delete` signal to delete the files referenced by its three thumbnail fields through their configured storage backend. Add a regression test that verifies both database records and physical thumbnail files are removed when a missing photo is deleted.

## Phase I: Issue Selection

### Why I Chose This Issue

When LibrePhotos deletes missing photos, cascading database deletion removes the related thumbnail record, but Django does not automatically remove files stored by `ImageField` values. The orphaned files continue consuming storage. This issue has a clear, testable outcome and demonstrates how Django model deletion can coordinate with file cleanup.

## Phase II: Reproduction and Solution Planning

**Scope:** This Phase II record concerns thumbnail cleanup, not email-validation issue #695 in the current README. The portal must identify the same issue as the submitted Phase II work.

**Working branch:** [fix/remove-orphaned-thumbnails](https://github.com/stepheng223/librephotos/tree/fix/remove-orphaned-thumbnails). Local inspection on October 6, 2026 confirmed this branch and implementation commit `44080a45`; remote branch availability was awarded in the supplied Phase II review, not rechecked here.

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

Before submitting, review these edge cases with maintainers and add coverage where the agreed behavior requires it:

| Edge case | Why it matters / proposed verification |
| --- | --- |
| Empty thumbnail fields | Skip empty names; deleting the record must not call storage deletion with an empty path. |
| File already missing | Confirm the configured storage backend's missing-file behavior; cleanup should tolerate an already-removed asset where supported. |
| Database transaction rollback | The current receiver deletes files immediately. A later rollback could restore rows without their files; assess deferring cleanup with `transaction.on_commit()` and test the agreed lifecycle. |
| Shared filename | Check whether another thumbnail record can reference the same stored name before deleting it; record deletion must not damage another photo's assets. |
| Storage permission/network failure | Decide how to log, report, or retry cleanup errors. Do not silently report successful cleanup when files remain. |
| Video preview files | The square fields may reference MP4 assets; delete their recorded names rather than assuming every file ends in WebP. |
| Other user's photos | Ensure the deletion job and receiver leave surviving users' records and files intact. |

These are investigative findings and planned checks, not claims that every case is already covered. The existing regression test covers the normal path for all three fields.

#### Evaluate

Acceptance criteria: deleting a missing photo removes its `Photo` and `Thumbnail` rows and all three referenced files, while surviving photos retain their assets. Run the focused cleanup regression and the existing missing-photo test module; inspect job completion/error behavior as well as assertions, since the job catches exceptions internally. The earlier focused test result is recorded in Phase III. Additional edge-case tests and a fresh suite run remain outstanding before claiming broader coverage.

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
- [ ] Push the revised documentation and submit the matching repository link in the Course Portal with **Phase II Complete** selected.

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

Local inspection on October 6, 2026 confirmed the implementation and regression test in commit `44080a45` on this branch. Remote publication and pull-request status were not rechecked during this documentation revision.

Relevant files:

- `apps/backend/api/models/thumbnail.py`
- `apps/backend/api/tests/photos/test_delete_missing_photos.py`

### Testing Strategy and Results

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

## Phase III Completion

**Phase III Complete.** The implementation is working, includes a regression test, and has been validated in the Docker development environment. The next step is Phase IV: commit the changes, push the development branch, open a pull request, and respond to maintainer feedback.

## Phase IV: Pull Request and Review

**Pull request:** Not submitted yet.

### Change Summary

This change prevents orphaned thumbnail files when missing photos are deleted. It keeps storage aligned with database state without changing thumbnail generation or unrelated photo cleanup behavior.

### Maintainer Feedback and Responses

No maintainer feedback yet. This section will be updated after the pull request is opened.

## Submission Checklist

- [x] Implementation summary added.
- [x] Development branch link added.
- [x] Testing strategy and results documented.
- [x] Phase III marked complete.
- [x ] Commit and push the implementation changes.
- [ x] Open a pull request targeting LibrePhotos `dev`.
- [ ] Participate in the required Slack scrum and attach a screenshot to the Phase III submission.
- [ x] Submit the contribution README and mark Phase III complete in the course platform.
