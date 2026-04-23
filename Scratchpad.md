# Good Playgrounds
Booran Reserve

Royal Children's Hospital in Parkville. Maritime Cove Park in Port Melbourne. JJ Holland in Kensington.

Hansen Reserve in West Footscray

Marie Wallace in Bayswater 

Thomas Street Park in Hampton

Chelsea Bicentennial Park

Wombat Bend in Templestowe, Eltham North Adventure playground, Diamond Valley regional playspace in Diamond Creek.

# Househunting
39 Amesbury AvenueWyndham Vale, VIC 3024

51 Vaughan Chase, Wyndham Vale, Vic 3024

8 Walbrook Drive, Wyndham Vale, Vic 3024

9 Ellenborough Crescent, Manor Lakes, Vic 3024

59 Kinglake Drive, Manor Lakes, Vic 3024

76 Eureka Drive, Manor Lakes, Vic 3024

4 Chesterfield Drive, Wyndham Vale, Vic 3024


# AI Workflow Steps
- [ ] Plan Mode
	- [ ] Write a PRD
	- [ ] Write issues from PRD
- [ ] Normal Mode
	- [ ] Epic and Issues
- [ ] Plan Mode
	- [ ] Grill Me about implementation
- [ ] Normal Mode
	- [ ] Implement Solution



I am having trouble adding paid parental leave.
208097455T


ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIA0xI/CO5ViCvlMrol1nC1YcVAsk3k9UaoXlzF/DNNA9 R2-D2

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIZzUUlhinUotLUTFXpfq57rNvURofJZDnHNL7fN4oFm u0_a491@localhost

```md
# Instructions
Complete one of the sub-tasks of portal-e8e

Before doing anything:
- Read the root epic and all comments on the epic.
- Decide what to do next yourself. 

Rules:
- Own Beads task management under this epic.
- You may create or split tasks when the existing breakdown is wrong.
- Any task you create or split out during the iteration must also be added under this same root epic.
- Prioritize tasks already marked in_progress before starting new ready tasks.
- Complete one task per iteration.
- Use TDD to complete the task: start with a failing test or repro, make the minimal change to pass, then refactor if needed.
- Before changing a chosen task, read that task's comments.
- If you cannot complete a task this iteration, mark it blocked and explain why in a concise task comment.
- After making any change to Beads tickets or comments, run `bd dolt push`.
- Commit your changes at the end of every iteration.
- Do not interact with any git remote. Keep all changes local to this repository.
- If the task is incomplete or blocked, still commit the work with an explicit in-progress gitmoji commit message, such as one using :construction:.
- After successfully completing a task, leave a concise progress comment on the epic that includes:
  - task completed and PRD/item reference
  - key decisions and reasoning
  - files changed
  - blockers or notes for next iteration
- Close completed tasks.
- Close the root epic when its scope is actually complete.

Before exiting the iteration, ensure the working tree is committed. Do this after validation and after any task or epic comments are updated.

If you repair the task graph because a task is oversized, you may continue in this same iteration only if you can still finish one resulting task safely. Otherwise, leave the task in the correct status, comment concisely, and make the required in-progress commit before stopping.

Keep comments concise. Sacrifice grammar for concision.


# Git Commit Message Instructions

OVERVIEW
- Header:
  - "<gitmoji> <short summary>"
  - Imperative verb, <=50 characters, no trailing period.
- Blank line
- Body (optional):
  - Wrap lines at 72 characters.
  - Describe the code changes as concisely as possible.
  - Do NOT explain why the changes were made.
  - Use bullet points starting with "- ".

EXAMPLE
✨ add token-refresh endpoint

- Support JWT rotation for long-lived sessions
- Return 401 if refresh token is expired

OTHER
- Do NOT write a body for simple commits. Prefer single-line commit
  messages where possible.
- Do NOT use the 🎉 emoji for any commit other than the first.

RULES FOR USING GITMOJIS
1. Always start the commit subject with exactly one gitmoji, followed by
   a space.
2. Choose the gitmoji that best represents the primary purpose of the
   commit.
3. Do not invent meanings or use emojis outside this list.
4. Write commit subjects in the imperative mood.
5. The commit header must describe the change in a way that matches or
   closely reflects the emoji's meaning. A reader should understand the
   intent without needing to know the emoji.
6. Keep the subject short and focused.
7. Examples:
   - Good: `🐛 Fix crash when saving draft`
   - Good: `♻️ Refactor authentication logic`
   - Avoid: `🐛 Update code`
   - Avoid: `✨ Changes`
8. When unsure, default to clarity over cleverness:
   - Feature -> ✨
   - Bug fix -> 🐛
   - Refactor -> ♻️
   - Docs -> 📝
   - Tests -> ✅
9. Only use ✅ when the commit includes test file changes and no other
   substantive changes. If non-test changes are present, choose the
   gitmoji that best matches those changes instead.
10. Output requirement (strict):
   - Return only the final commit subject and body, with no extra text.

OFFICIAL GITMOJI LIST
- 🎨 Improve structure / format of the code.
- ⚡️ Improve performance.
- 🔥 Remove code or files.
- 🐛 Fix a bug.
- 🚑️ Critical hotfix.
- ✨ Introduce new features.
- 📝 Add or update documentation.
- 🚀 Deploy stuff.
- 💄 Add or update the UI and style files.
- 🎉 Begin a project.
- ✅ Add, update, or pass tests.
- 🔒️ Fix security or privacy issues.
- 🔐 Add or update secrets.
- 🔖 Release / Version tags.
- 🚨 Fix compiler / linter warnings.
- 🚧 Work in progress.
- 💚 Fix CI Build.
- ⬇️ Downgrade dependencies.
- ⬆️ Upgrade dependencies.
- 📌 Pin dependencies to specific versions.
- 👷 Add or update CI build system.
- 📈 Add or update analytics or track code.
- ♻️ Refactor code.
- ➕ Add a dependency.
- ➖ Remove a dependency.
- 🔧 Add or update configuration files.
- 🔨 Add or update development scripts.
- 🌐 Internationalization and localization.
- ✏️ Fix typos.
- 💩 Write bad code that needs to be improved.
- ⏪️ Revert changes.
- 🔀 Merge branches.
- 📦️ Add or update compiled files or packages.
- 👽️ Update code due to external API changes.
- 🚚 Move or rename resources (e.g.: files, paths, routes).
- 📄 Add or update license.
- 💥 Introduce breaking changes.
- 🍱 Add or update assets.
- ♿️ Improve accessibility.
- 💡 Add or update comments in source code.
- 🍻 Write code drunkenly.
- 💬 Add or update text and literals.
- 🗃️ Perform database related changes.
- 🔊 Add or update logs.
- 🔇 Remove logs.
- 👥 Add or update contributor(s).
- 🚸 Improve user experience / usability.
- 🏗️ Make architectural changes.
- 📱 Work on responsive design.
- 🤡 Mock things.
- 🥚 Add or update an easter egg.
- 🙈 Add or update a .gitignore file.
- 📸 Add or update snapshots.
- ⚗️ Perform experiments.
- 🔍️ Improve SEO.
- 🏷️ Add or update types.
- 🌱 Add or update seed files.
- 🚩 Add, update, or remove feature flags.
- 🥅 Catch errors.
- 💫 Add or update animations and transitions.
- 🗑️ Deprecate code that needs to be cleaned up.
- 🛂 Work on code related to authorization, roles and permissions.
- 🩹 Simple fix for a non-critical issue.
- 🧐 Data exploration/inspection.
- ⚰️ Remove dead code.
- 🧪 Add a failing test.
- 👔 Add or update business logic.
- 🩺 Add or update healthcheck.
- 🧱 Infrastructure related changes.
- 🧑‍💻 Improve developer experience.
- 💸 Add sponsorships or money related infrastructure.
- 🧵 Add or update code related to multithreading or concurrency.
- 🦺 Add or update code related to validation.
- ✈️ Improve offline support.
- 🦖 Code that adds backwards compatibility.
```







# Inspections
8 Walbrook Drive

22 Federal Drive
- External door
- shade cloth broken

42 Howqua Way
- didn't go inside
- School and playground nearby

15 Olvallia Rd
- Paint bubbling in bathroom


3 Maculata Pl
- 



# TODO
- Make backup of DB before deploying these changes.

# Things to Update
- [x] ApplicationRepresentative.php
	- [x] Convert those arrays to Enums
- [x] 2026_03_31_000001_create_application_representatives_table.php
	- [x] Change the name of the other_epoa_contact_info column to 'notes', rather than making a new column.
- [x] Form formatting
	- [x] Delete button is too large, move into the top corner.
	- [x] Add a prompt before deleting
	- [x] Move title to the next row
	- [x] Remove Representative X title
- [ ] resources/js/pages/applications/components/form/components/epoa-details-section/index.tsx
	- [ ] Rather than using the append method directly, make a seperate helper method to wrap that function.
- [x] In /home/james/Projects/portal/app/Http/Resources/ApplicationResource.php, update the gate to use edit instead of view permission.
- [x] Backfill required EPOA fields during migration — /home/james/Projects/portal/database/migrations/2026_03_31_000001_create_application_representatives_table.php:60-72  
	  This backfill creates every legacy EPOA representative with authority and primary_decision_maker set to null, but the new validation now requires both for any EPOA row. After deploying this migration, any existing application that had legacy EPOA data will fail validation on the next edit unless a user manually repairs those new fields first, even if they were only changing an unrelated field.
- [ ] Tests
	- [ ] Also write tests for edge-cases.

Fix these:
  - [x] Gate premium payload with the view permission — /home/james/Projects/
    portal/app/Http/Resources/ApplicationResource.php:18-18
    This switched the resource gate from application-view-premium-fields to
    application-edit-premium-fields, but the React view still decides whether to
    render the section from the view permission. In any role setup where a user
    may view-but-not-edit premium data, the page will now show -/empty
    representative data because patient_has_capacity_to_make_decisions, notes,
    and representatives are omitted from the Inertia payload. - REVERT IN GIT 
  - [ ] Clear stale other_type when a representative stops being Other — /home/
    james/Projects/portal/resources/js/pages/applications/components/form/
    index.tsx:107-115
    This mapper forwards every representative field unchanged except
    primary_decision_maker. Because other_type is rendered conditionally and the
    form is not using shouldUnregister, a user can pick Other, enter Advocate,
    then switch the same row to EPOA or POA; React Hook Form will keep the hidden
    other_type value and it will be saved. The read-only card then renders
    incorrect labels such as EPOA: Advocate. 
  - [x] Throw on transactional save failures instead of returning strings — /
    home/james/Projects/portal/app/Services/ApplicationService.php:269-269
    The new transaction only rolls back on exceptions, but the failure paths
    inside this closure all return error strings (store() and
    syncRepresentatives() swallow exceptions). If a representative insert fails
    after syncRepresentatives() has deleted the old rows, the request reports an
    error while still committing the delete and any earlier inserts, leaving the
    application partially updated. 



- [ ] Code review feedback:
	- [ ] Serialize representative fields for premium editors — /home/james/Projects/portal/app/Http/Resources/ApplicationResource.php:41-49  
	  The edit form is hydrated from application.data, but these new premium fields are only sent when the user has application-view-premium-fields. A role can have application-edit-premium-fields without that view permission, and in that case the sidebar opens with patient_has_capacity_to_make_decisions = null, notes = undefined, and representatives = []; submitting any unrelated edit then clears the saved representative data. Please expose these fields whenever the current user is allowed to edit them, or load the edit form from a source that is not gated by the view permission.
	- [ ] Backfill required EPOA fields during migration — /home/james/Projects/portal/database/migrations/2026_03_31_000001_create_application_representatives_table.php:60-72  
	  This backfill creates every legacy EPOA representative with authority and primary_decision_maker set to null, but the new validation now requires both for any EPOA row. After deploying this migration, any existing application that had legacy EPOA data will fail validation on the next edit unless a user manually repairs those new fields first, even if they were only changing an unrelated field.
	- [ ] Throw on save failures inside the new transaction — /home/james/Projects/portal/app/Services/ApplicationService.php:265-280  
	  DB::transaction() only rolls back when the closure throws, but the failure paths here just return error strings. Because store()/syncRepresentatives() catch exceptions and return strings, a representative insert failure after representatives()->delete() or an application_files save failure will still commit the earlier writes. On update that can permanently delete the old representatives even though the request is reported as failed.
# Things to Change
- [ ] Move ApplicationEnquiryReviewController to the Aplication Controller?
- [ ] 


# LLM Skills to Install
grill-me
improve-codebase-architecture

write-a-prd
prd-to-issues


# GitKraken on CLI
- Better log
- rename commits
- squash commits
	- 
- undo commits
- AI commit messages
- Better merge conflict resolution
- move commits up and down
- Create PRs



Car Service:
Wednesday 25th







---

Hi Lisa,

 Thanks for sending through the pen test report. I've updated the cards we discussed in the meeting today. If you could please review them and let me know if anything needs to be added or modified that would be great, thanks. 

ADD_CARDS_HERE

Cheers,
James

- https://linear.app/webres-solutions/issue/CBR-48/allow-re-presentation-of-previously-unsuccessful-patients
- https://linear.app/webres-solutions/issue/CBR-49/allow-hospital-users-to-respond-to-provider-comments
- https://linear.app/webres-solutions/issue/CBR-135/notification-subscriptions

---





Job to Apply for:
https://www.linkedin.com/jobs/view/4377110300/?trackingId=pUHbpsDyXQWpPSdEqlmo2g%3D%3D&refId=hw%2Bv1%2B55F4kyOiRIuhhUVA%3D%3D&midToken=AQEzzsZtjvla8Q&midSig=1h1zF_9OLbNs81&trk=eml-email_job_alert_digest_01-primary_job_list-0-job_posting_0_jobid_4377110300_ssid_15402721724_fmid_35t8t3~mma2qvgq~2g&trkEmail=eml-email_job_alert_digest_01-primary_job_list-0-job_posting_0_jobid_4377110300_ssid_15402721724_fmid_35t8t3~mma2qvgq~2g-null-35t8t3~mma2qvgq~2g-null-null&eid=35t8t3-mma2qvgq-2g&otpToken=MTMwYzFmZTExNDJmYzljMGIzMjQwNGVkNDExY2UyYjU4ZmNmZDE0Mjk4YWE4ODYxNzdjNzA4NmQ0ZTVhNThmMmY2ZGY4NmJkNTVjN2M0ODY2NzlmN2NmODlhMTc1NDMyZGViMWJmOTk1OGNjZGEyNTg2ZmIsMSwx


# Home Loan Application

| Item               | 2024     | 2025      |
| ------------------ | -------- | --------- |
| Taxable Income     | 6,534.00 | 10,216.00 |
| Depreciation       | 1,157.65 | 0         |
| Interest Add Backs | 0        | 0         |
| Operating Expenses | 5,269.09 | 1721<br>  |








Keyboards I like:
- [Charybdis MK2](https://bastardkb.com/product/charybdis-mk2-prebuilt-preorder/)
- [Ergokeyboards Crosses/Bridges Keyboard](https://ergokeyboards.com/products/crosses-modular-keyboard?variant=50272542228762)
- ZSA Moonlander



Good switch upgrades:
- Gateron Baby Kangaroo Switch
- K Pro Banana Switch