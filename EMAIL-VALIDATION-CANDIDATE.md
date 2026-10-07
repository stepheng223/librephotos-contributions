# Open Source Contribution Log

**Contributor:** [stepheng223](https://github.com/stepheng223)

**Project:** [LibrePhotos](https://github.com/LibrePhotos/librephotos)

**Issue:** [#695 — Replace regex-based email validation](https://github.com/LibrePhotos/librephotos/issues/695)

**Contribution repository:** [librephotos-contributions](https://github.com/stepheng223/librephotos-contributions)

**Project fork:** [stepheng223/librephotos](https://github.com/stepheng223/librephotos)

**Status:** Phase I — selection documented; community engagement and check-in pending confirmation

**Last reviewed:** October 6, 2026

## Phase I: Issue Selection

### Why I Chose This Issue

LibrePhotos uses a shared regular expression to validate email addresses, and issue #695 reports maintenance problems and related failures in [#689](https://github.com/LibrePhotos/librephotos/issues/689) and [#637](https://github.com/LibrePhotos/librephotos/issues/637). This matters because validation should accept supported valid addresses while giving users clear feedback for invalid input. My earlier contribution log documents work with Python/Django, Docker Compose, Git branches, and regression testing; I chose this small frontend issue to apply that debugging and testing process while learning React/TypeScript form validation. From reading the thread, I understand that common email formats are the current priority, so my goal is a maintainer-approved replacement with agreed acceptance cases and preserved translated error messages.

### Understanding the Issue

The request is to improve email validation's robustness and readability. It does not specify a replacement library or require support for every address allowed by RFC 5322. [sickelap's comment](https://github.com/LibrePhotos/librephotos/issues/695#issuecomment-1346103701) explains the focus on commonly used formats and invites a better solution; [AudreyBeard's reply](https://github.com/LibrePhotos/librephotos/issues/695#issuecomment-1346874413) identifies this as a possible first TypeScript/frontend contribution. I will ask maintainers to agree on the validation contract before implementing it.

### What Fixed Looks Like

- All consumers of the shared email pattern use the agreed replacement consistently.
- Regression tests accept and reject the concrete email examples agreed with maintainers, including cases from the linked reports where still applicable.
- Invalid input retains the existing translated error feedback.
- Relevant frontend tests, lint, and build checks pass.

### Repository and Issue Evidence

On October 6, 2026, GitHub's unauthenticated API confirmed that this contribution repository is public, #695 is open with no assignees or subtasks, and its labels contain no blocked, wontfix, needs discussion, or question label. The complete issue timeline contained two discussion comments and no linked PR or current work claim. Neither comment is from `stepheng223`, so the required introductory comment remains outstanding. The upstream project received a push on October 6, 2026, within the rubric's six-month maintenance window.

The repository name `librephotos-contributions` describes the project and purpose. The reviewer nevertheless deducted its naming point; the supplied feedback does not state a mandatory naming convention. Confirm the required name with course staff before renaming or changing the submission URL.

Likely code locations:

- [apps/frontend/src/routes/signup.tsx](https://github.com/LibrePhotos/librephotos/blob/dev/apps/frontend/src/routes/signup.tsx): verified in upstream `dev`; imports `EMAIL_REGEX` and uses it in the signup form validator.
- `apps/frontend/src/util/util.ts`: shared utility module imported by signup; inspect the pattern and find all consumers in Phase II.
- [apps/frontend/src/api_client/auth/hooks/useSignUpMutation.ts](https://github.com/LibrePhotos/librephotos/blob/dev/apps/frontend/src/api_client/auth/hooks/useSignUpMutation.ts): inspect signup error handling so frontend and server validation remain consistent.

### Candidate Comparison

| Candidate | Problem / scope | Skills and learning | Activity / availability | Context | Setup | Decision |
| --- | --- | --- | --- | --- | --- | --- |
| [#695: Email validation](https://github.com/LibrePhotos/librephotos/issues/695) | Clear validation problem; size XS, easy | Small TypeScript learning target | Open, unassigned; no claim or linked PR in reviewed timeline | Two comments and related reports | Development guide | Selected: smallest bounded scope |
| [#861: Choose Start of Week Day](https://github.com/LibrePhotos/librephotos/issues/861) | Calendar preference; size S, easy | Frontend learning target | Open, unassigned; timeline/PR availability not checked | No comments; preference storage needs clarification | Same development guide | Backup: less implementation context |
| [#2037: Settings/admin navigation](https://github.com/LibrePhotos/librephotos/issues/2037) | Shortcuts plus links across pages | Frontend learning target | Open, unassigned; recent member-authored request; timeline not checked | Concrete interaction examples | Same development guide | Backup: broader scope |

These notes were checked through GitHub's live API on October 6, 2026. Membership in the course's curated Google Sheet still needs confirmation.

### Six-Point Selection Checklist

| Check | Assessment and evidence |
| --- | --- |
| 1. Understand the problem | Pass: replace difficult-to-maintain validation; success means agreed accepted/rejected inputs and clear feedback. |
| 2. Scope fits four weeks | Pass: upstream labels are good first issue, size XS, and difficulty easy. Limit changes to validation and its consumers. |
| 3. Skills match or can be learned quickly | Prior log documents Python/Django, Docker Compose, Git branches, and regression testing. Apply that process here while learning React/TypeScript; confirm those experience details before submission. |
| 4. Active and claimable | Provisional: open, no assignees, two comments with no active claim, and no linked PR in the reviewed timeline. Labels updated July 22, 2026; recent issue-specific maintainer confirmation is still needed. |
| 5. Helpful context | Pass: description explains the maintenance problem, links related reports, and discusses common email formats. Acceptance examples remain to be agreed. |
| 6. Clear setup documentation | Pass: official guide includes macOS/Linux Docker Compose setup, prerequisites, hot reload, and frontend checks. CONTRIBUTING explains the monorepo and dev target branch. |

**Assessment:** Four checks supported, two provisional. Following the course's 4–5-check guidance, seek mentor confirmation of scope and availability before treating selection as fully confirmed.

References: [Development Installation](https://docs.librephotos.com/docs/development/dev-install/), [CONTRIBUTING.md](https://github.com/LibrePhotos/librephotos/blob/dev/CONTRIBUTING.md), and [Frontend README](https://github.com/LibrePhotos/librephotos/blob/dev/apps/frontend/README.md). Local reproduction belongs in Phase II.

### Community Engagement

**GitHub issue comment:** Pending. Add its permalink here after posting.

Draft to post on #695:

> Hi, I'm Stephen, a CodePath contributor, and I'd like to work on this issue. I found that signup still uses the shared EMAIL_REGEX, and I plan to review its other callers and propose a focused replacement with regression tests. Is this still available? Do you have a preferred validation approach or examples of addresses we should accept and reject? I'll preserve the translated error feedback and confirm scope before implementation.

**Course Google Sheet:** Pending. Confirm #695 is approved and comment on its row: “Stephen / stepheng223 — selecting LibrePhotos #695 (email validation); checking availability and validation expectations with maintainers.”

**Issue-selection Slack draft** for `#dts-fa26-ai301-issue-selection`:

> I'm considering LibrePhotos #695: https://github.com/LibrePhotos/librephotos/issues/695. It is open, unassigned, and labeled good first issue / size XS / difficulty easy. I found the shared regex in signup and want to propose a focused replacement with regression tests. Four checklist items are supported; skill ramp-up and current maintainer confirmation are provisional. Any scope or course-list concerns?

### Phase I Deliverables and Evidence

- [x] Contribution README and sections for all four phases created.
- [x] Live issue link and four-sentence problem/selection summary included.
- [x] Three candidates compared and six selection checks documented.
- [x] Existing public LibrePhotos fork verified through GitHub's API.
- [ ] Confirm GitHub login and CodePath Student Slack access, including `#dts-fa26-ai301-contribute`.
- [x] Contribution repository verified public through an unauthenticated GitHub API request.
- [ ] Resolve the reviewer's repository-name concern with course staff.
- [ ] Push the revised README so the submitted URL renders this version.
- [ ] Confirm issue selection with course staff / curated sheet and maintainers.
- [ ] Post the GitHub interest comment and add its permalink above.
- [ ] Comment on the course Google Sheet issue row.
- [ ] Submit the contribution repository link in the course check-in and select “Phase I Complete.”
- [ ] Announce the milestone in `#dts-fa26-ai301-celebration`.

After completing these steps, change the Status field to **Phase I Complete**. Suggested celebration message:

> 🎯 Phase I Complete — Selected LibrePhotos #695: improve email validation so it is easier to maintain and handles agreed email formats consistently. Issue: https://github.com/LibrePhotos/librephotos/issues/695. Contribution log: https://github.com/stepheng223/librephotos-contributions.

## Phase II: Reproduction and Solution Planning

**Status:** Not started for #695.

The supplied Phase II review awarded all required technical categories for the earlier work; it deducted the Phase II portal check-in and investigative-depth stretch item. The strengthened [thumbnail-cleanup Phase II record](PREVIOUS-CONTRIBUTION-LOG.md#phase-ii-reproduction-and-solution-planning) now includes UMPIRE sections, an analogous cleanup receiver, commit-history findings, and concrete edge cases. Those findings apply to thumbnail cleanup and do not count as reproduction of #695. Confirm the submission's issue before marking Phase II complete.

### Reproduction Process

Review related reports and all regex consumers, run the documented environment, and record concrete input/output examples, expected versus actual behavior, and environment details.

### Solution Approach

Agree on supported formats and a replacement approach with maintainers. Identify affected forms, preserve translated feedback, and plan regression cases. Document the final approach after reproduction.

## Phase III: Implementation and Testing

**Status:** Not started for #695.

### Implementation Notes

Record final changes, branch/commit links, and relevant design decisions after implementation.

### Implementation Progress — Earlier Thumbnail Work

This evidence concerns [fix/remove-orphaned-thumbnails](https://github.com/stepheng223/librephotos/tree/fix/remove-orphaned-thumbnails), not #695. Local inspection verified commit [44080a451b058f89b5f5ec570c8ccf4748b49135](https://github.com/stepheng223/librephotos/commit/44080a451b058f89b5f5ec570c8ccf4748b49135), dated September 23, 2026, with message `fix: remove orphaned thumbnail files`.

| File | Location in inspected checkout | Change |
| --- | --- | --- |
| `apps/backend/api/models/thumbnail.py` | Lines 193–202, `auto_delete_files_on_delete()` | Registers a Thumbnail post-delete receiver and deletes the three referenced assets through each field's storage backend, skipping empty fields. |
| `apps/backend/api/tests/photos/test_delete_missing_photos.py` | Lines 46–73, `DeleteMissingPhotosThumbnailCleanupTest` | Adds a regression test that writes all three assets, calls `delete_missing_photos()`, and checks database and storage cleanup. |
| `contribution_readme.md` | Added in the same commit | Records the contribution process; this is documentation rather than application behavior. |

These line references are from the local checkout inspected October 6, 2026. The commit is after the supplied September 14 deadline; it is evidence for a revised submission, not proof of implementation before that deadline. The grader awarded commit cadence and message quality; this inspection does not independently establish six issue-specific commits or every-other-day activity.

### Challenges Faced

The earlier setup log records a Docker service-name conflict: an existing `frontend` container prevented Compose startup. After confirming it belonged to the same Compose project, the stale container was removed and the development Compose stack was restarted successfully.

The earlier focused test initially failed because the production image lacked development packages, including `coverage` and `faker`. Installing `apps/backend/requirements.dev.txt` in the running backend container and rerunning the test with `NO_COVERAGE=1` resolved that recorded attempt.

During this documentation revision, `docker ps` failed because `/Users/stephen/.docker/run/docker.sock` did not exist. A fresh test run remains blocked until Docker is started; this obstacle is still unresolved and is not presented as a passing verification.

### Testing Strategy

Plan coverage for agreed valid and invalid addresses and consistent behavior across affected forms. Run relevant frontend tests and required lint/build checks; record actual commands and results once executed.

### Testing Evidence — Earlier Thumbnail Work

**Automated case:** `DeleteMissingPhotosThumbnailCleanupTest.test_deleting_missing_photo_removes_thumbnail_files` creates a user/photo with the project's existing `create_test_user()` and `create_test_photo()` helpers, writes each thumbnail field through `storage.save(..., ContentFile(...))`, calls the production deletion job, and asserts that Photo/Thumbnail rows and all three stored files are absent. It follows the neighboring Django `TestCase`, `test_...` naming, and factory-helper patterns. This exercises stored-file cleanup; it does not exercise thumbnail generation or every edge case.

Previously recorded command and result (not rerun today):

```bash
docker exec -e NO_COVERAGE=1 backend python manage.py test \
  api.tests.photos.test_delete_missing_photos.DeleteMissingPhotosThumbnailCleanupTest
```

```text
Ran 1 test in 0.276s
OK
```

**Manual verification:** The earlier log records scanning a test image, removing its original, running missing-photo scan/deletion, and observing orphaned files before the fix. This revision manually inspected the committed receiver and regression assertions. A fresh browser/storage walkthrough after the fix has not been performed or captured here.

**Full suite:** No full-suite pass is claimed. After Docker is running, run the focused module and then the backend suite from the matching checkout:

```bash
docker exec -e NO_COVERAGE=1 backend python manage.py test api.tests.photos.test_delete_missing_photos
docker exec -e NO_COVERAGE=1 backend python manage.py test api.tests
```

Record complete outputs and investigate failures before claiming suite credit.

**Engineering judgment:** Reuse the project's user/photo helpers rather than inventing fixtures, and delete recorded storage names rather than assuming a local filesystem or WebP extension. The Phase II review identifies rollback, shared filenames, video previews, and storage failures as additional cases needing an agreed policy and coverage.

### Phase III Submission Evidence

- [x] Specific files, functions, and implementation commit documented for thumbnail cleanup.
- [x] Recorded setup obstacles and resolutions documented.
- [x] New regression test and project-helper usage identified.
- [ ] Confirm which issue the resubmission follows; this thumbnail evidence cannot establish implementation of #695.
- [ ] Start Docker, run and record the full suite, and capture a fresh manual verification.
- [ ] Push this revised README so the grader can see the evidence.
- [ ] Add a permalink or screenshot of an actual Slack post made within seven days before resubmission.

The supplied review awards the Phase III portal check-in. Confirm the revised submission links the updated README and retains **Phase III Complete** for the same issue.

Suggested Slack update to post yourself after confirming the thumbnail issue remains your submission:

> My thumbnail-cleanup branch adds a post-delete receiver and a regression test using LibrePhotos' existing user/photo helpers. The earlier focused test passed; I'm checking rollback and shared-filename behavior before claiming broader coverage. A fresh suite run is currently blocked because Docker is stopped. Branch: https://github.com/stepheng223/librephotos/tree/fix/remove-orphaned-thumbnails.

## Phase IV: Pull Request and Review

### Pull Request

**Pull request:** Not submitted for #695.

### Change Summary

To be completed after implementation and validation. Target `dev`, as described in CONTRIBUTING.md.

### Maintainer Feedback and Responses

No feedback received for this selection yet. Record comment links, requested changes, and responses here.

## Learnings & Reflections

To be updated throughout the contribution. Reflect on validation design, React/TypeScript learning, regression testing, and maintainer feedback as the work progresses.

## Previous Contribution Work

The earlier thumbnail-cleanup log is preserved in [PREVIOUS-CONTRIBUTION-LOG.md](PREVIOUS-CONTRIBUTION-LOG.md). It was based on a documentation TODO and did not identify a live GitHub issue. Its implementation and testing notes concern that earlier task and do not establish completion of any phase for #695.
