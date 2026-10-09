# Rubric evidence for resubmission

This maps the supplied grading criteria to the current submission. “Supported” means evidence is present, not that the grader has awarded credit. The contributor confirms edits are accepted and Slack posting is not required. Historical evidence is still attributed accurately.

## Unit 1

Closest related historical issue: [#19 — Manage photos](https://github.com/LibrePhotos/librephotos/issues/19), which mentions thumbnail/database data remaining after original-photo deletion. It is closed and broader than PR #2078, so it provides context rather than proving an originally open, claimable issue or a valid Closes reference.

| Criterion | Evidence / remaining action |
| --- | --- |
| Professional username | Supported: README header identifies `stepheng223`. |
| Public, clearly named README repo | Public repository verified. Naming deduction remains: ask staff for the required convention before changing the submission URL. |
| Full template structure | Supported: Why I Chose This Issue, Understanding the Issue, Reproduction Process, Solution Approach, Testing Strategy, Pull Request, Learnings & Reflections are present. |
| Project fork | Supported: README links `stepheng223/librephotos`; PR source identifies the fork. |
| Live claimable issue | Gap: thumbnail work originated in a documentation TODO. No matching numbered issue established. #695 is unrelated and cannot supply this credit. |
| Maintained project, bounded issue, setup docs | Project maintenance, small cleanup diff, and development documentation are supported. Live-issue requirement still needs course guidance. |
| Two-sentence introductory issue comment | Gap: no original issue/comment established. Do not manufacture historical evidence. |
| Skill match, learning goal, understanding | Supported: four-sentence Why I Chose This Issue names Python/Django, Docker Compose, Git, regression testing, and deletion-lifecycle learning. Confirm the experience statement is accurate. |
| Two-to-four-sentence problem summary | Supported: Why I Chose This Issue contains four sentences explaining the defect, impact, selection, and intended outcome. |
| Phase I check-in | Needs course-account confirmation/submission. |
| Stretch: files, context, or acceptance criteria | Supported: concrete model/job/test paths and cleanup acceptance criteria. |

## Unit 2

| Criterion | Evidence / remaining action |
| --- | --- |
| Working branch | Supported: README branch link and merged PR source branch. Existing review awarded this item. |
| Real setup challenges and resolutions | Supported: frontend-container conflict and missing development packages, with recorded resolutions. |
| Setup approach | Supported: Docker Compose development-install path explicitly stated and linked. |
| Numbered reproduction steps | Present; existing review awarded this item. |
| Expected versus actual | Explicit lines present; initial focused output and later CI links distinguish versions. |
| Specific files/functions | Supported: Thumbnail relationship/receiver, delete_missing_photos job, and cleanup regression. |
| UMPIRE/equivalent | Supported: Understand, Match, Plan, Implement, Review, Evaluate. |
| Root cause and modification plan | Supported: database cascade versus separate file storage, named modification/test files. |
| Phase II check-in | Needs course-account submission with Phase II Complete selected. |
| Stretch: investigative depth | Supported: parent-commit inspection, analogous Face receiver, rollback/shared-reference/error analysis. |

## Unit 3

| Criterion | Evidence / remaining action |
| --- | --- |
| Meaningful commits after Phase II | Existing review awarded this. Initial student commit is linked; original Phase II timestamp has not been independently verified. |
| Regular student commit cadence | Existing review awarded this. Current PR evidence contains one student implementation commit and five other-author commits; do not claim those establish six student commits. Historical cadence cannot be repaired retroactively. |
| Descriptive messages | Initial message describes thumbnail-file cleanup; follow-up messages describe their specific changes, with authorship separated. |
| Scoped diff | Final maintainer commit removed the course-only log; final application change concerns thumbnail cleanup. |
| Implementation progress: paths/lines/SHAs | Supported in Implementation Progress and Phase IV commit table. Initial line references are labeled as such. |
| Challenges Faced | Supported: recorded obstacles, resolutions, and unresolved fresh-local-run limitation. |
| Manual and automated testing | Initial reproduction and focused test are documented; final CI suite is linked. Fresh post-fix manual walkthrough remains unrecorded. |
| Phase III check-in | Supplied review awarded this; verify the resubmission uses the updated README. |
| New test | Supported: student cleanup regression in commit 44080a45. |
| Existing suite passes | Supported: final-head Django CI runs python manage.py test api.tests successfully. |
| Project test conventions | Supported: Django TestCase and existing create_test_user/create_test_photo helpers. |
| Slack in preceding seven days | Optional stretch item in the supplied rubric; contributor confirms posting is not required. No post or participation credit is claimed. |
| Engineering judgment | Supported: factory reuse, final-plan attribution, transactional/shared-asset lessons and edge cases. |

## Unit 4

| Criterion | Evidence / remaining action |
| --- | --- |
| Upstream default-branch PR | Supported: PR #2078 was non-draft and merged into LibrePhotos/librephotos:dev, the default branch. Ask staff to recognize merged status if they require a currently open PR. |
| PR template/structure | Published description contains Why, What does this PR do, Issue references, Acceptance criteria, evidence, and review status. No project PR template found in inspected checkout. |
| Closes issue reference | Gap: no matching original numbered issue established. Do not use an unrelated issue or create a retroactive claim. |
| Why before what | Supported: published description starts with problem and investigative context. |
| Acceptance checklist fully completed | Live PR description checklist is complete: tests, protections, helper reuse, suite, style, and compatibility. October 9 final-diff inspection found no API/schema/dependency changes; the completed checklist was published and verified through GitHub's API that day. |
| Backend before/after evidence | Initial orphan behavior and focused output documented; final full-suite/style CI links published. |
| README PR link, summary, status | Supported: #2078, cleanup summary, Merged label. |
| Dated feedback/response log | Supported alternative evidence: explicitly records no formal comments/reviews, submission date, maintainer-authored changes, and merge. No student response commits invented. |
| Three reflection subsections | Supported: Technical Skills Gained, Challenges Overcome, What I'd Do Differently. |
| Internal consistency | Supported: all phases track thumbnail cleanup; immediate-storage plan versus final commit-safe hash cleanup is explained. #695 retained as a separate candidate. |
| Phase IV check-in | Needs course-account submission with Phase IV Complete selected. |
| Reviewer mentioned/requested | Supported: [October 9 student comment](https://github.com/LibrePhotos/librephotos/pull/2078#issuecomment-6091218100) mentions @derneuere. This is post-merge outreach, not a historical review request. |
| Stretch: open-source loop or teachable insight | Supported: merged PR and concrete transaction/shared-file lesson; final changes attributed to their authors. |

Additional upstream evidence: [release 1.2.0](https://github.com/LibrePhotos/librephotos/releases/tag/1.2.0) credits stepheng223 for orphaned-thumbnail cleanup, verified October 9, 2026. This supports shipped contribution evidence; it does not supply a missing numbered issue.
