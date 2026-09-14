# Open Source Contribution Log

**Project:** LibrePhotos
**Issue:** ARemove orphaned thumbnail files when deleting missing photos. (https://docs.librephotos.com/docs/development/contribution/backend/missing-photos)
**Status:** Phase I — Issue selection in progress
### Proposed Solution
To be completed.

## Phase I: Issue Selection

### Why I Chose This Issue: When LibrePhotos deletes missing photos, their thumbnail records are removed from the database, 
But the thumbnail files remain on disk and consume storage. 
I chose this issue because it addresses a concrete cleanup problem with a clear outcome: removing thumbnails that are no longer needed.
It also gives me an opportunity to learn how Django database deletion connects with file-system cleanup.

## Phase II: Reproduction and Solution Planning

### Reproduction Process

#### Environment Setup

I set up the LibrePhotos development environment on macOS using Docker
Desktop and Docker Compose. The repository uses a monorepo structure, with
the Django backend located in `apps/backend/` and the Docker Compose files
located in `deploy/compose/`.

During setup, I encountered the following issue:

- Docker failed while extracting an image layer and returned a
  `read-only file system` error.
- [Explain the steps that resolved the error.]
- After resolving the error, I started LibrePhotos and accessed the
  application at `http://localhost:3000`.

#### Steps to Reproduce

1. Start the LibrePhotos development environment.
2. Add a test image to the configured scan directory.
3. Run the LibrePhotos photo scan and wait for the image and its thumbnails
   to be generated.
4. Confirm that thumbnail files exist for the photo under:
   - `protected_media/thumbnails_big/`
   - `protected_media/square_thumbnails/`
   - `protected_media/square_thumbnails_small/`
5. Remove the original image directly from the configured scan directory.
6. Run the **Scan Missing Photos** job in LibrePhotos.
7. Confirm that the photo is identified as missing.
8. Run the **Delete Missing Photos** job.
9. Confirm that the photo and its thumbnail record are removed from the
   database.
10. Inspect the three thumbnail directories again.

**Expected result:** The thumbnail database record and all physical thumbnail
files associated with the deleted photo should be removed.

**Actual result:** The database records are removed, but the thumbnail files
remain in the thumbnail directories and continue consuming storage.

I repeated the process 2 times and observed the same behavior.

#### Reproduction Evidence

Working branch:

https://github.com/stepheng223/librephotos/tree/fix/remove-orphaned-thumbnails



### Solution Approach

#### Implementation Plan

**Understand:**  
When `delete_missing_photos()` deletes a missing `Photo`, Django's cascading
deletion removes the related `Thumbnail` database record. However, deleting
an `ImageField` database record does not automatically remove its physical
file. This leaves orphaned files in the three thumbnail directories.

**Match:**  
LibrePhotos already uses a Django `post_delete` signal in
`apps/backend/api/models/face.py` to remove a face image when its database
record is deleted. I will follow this existing cleanup pattern for the
`Thumbnail` model.

**Plan:**

1. Add thumbnail-file cleanup behavior to
   `apps/backend/api/models/thumbnail.py`.
2. When a `Thumbnail` record is deleted, remove the files referenced by:
   - `thumbnail_big`
   - `square_thumbnail`
   - `square_thumbnail_small`
3. Use each field's configured Django storage backend to delete the file.
4. Ignore empty fields and files that no longer exist so the cleanup does not
   cause the deletion job to fail.
5. Add automated tests to
   `apps/backend/api/tests/photos/test_delete_missing_photos.py`.
6. Verify that deleting a missing photo removes its photo record, thumbnail
   record, and physical thumbnail files.
7. Verify that thumbnails belonging to photos that were not deleted remain
   untouched.

**Implement:**  
Implementation will be completed during Phase III on the
`fix/remove-orphaned-thumbnails` branch.

**Review:**  
I will review the change against LibrePhotos' contribution guidelines. I will
run Ruff, keep the change focused on thumbnail cleanup, and use a concise
commit message such as:

`fix: remove thumbnail files for deleted photos`

**Evaluate:**  
I will run the existing missing-photo deletion tests and the new thumbnail
cleanup tests. I will also repeat the manual reproduction process and confirm
that the three physical thumbnail files no longer remain after using
**Delete Missing Photos**.

## Phase III: Implementation & Testing

### Implementation Notes
To be completed.

### Testing Strategy and Results
To be completed.

## Phase IV: Pull Request & Review

**Pull request:** Not submitted yet.

### Change Summary
To be completed.

### Maintainer Feedback and Responses
To be completed.
