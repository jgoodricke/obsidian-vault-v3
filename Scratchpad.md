- [x] Maples’ Incident, Injury, Trauma and Illness Record, including the injury photographs and follow-up note.
- [x] The June 2026 email correspondence containing Maples’ account and the CCTV screenshots.
- [x] The July 2026 correspondence concerning the regulatory notification and insurance claim.
- [x] Dr Justin Parr’s letter dated 15 July 2026.
- [x] Our chronology of events.
- [x] Our interim expense schedule 
	- [x] and supporting receipts.
- [x] James’s telephone call record.


## Workflow 
- Herdr
- plannotator


Carebridge login incident — findings summary (22 Jul 2026)

Three independent problems, not one root cause. The DB error the client forwarded is a red herring — harmless, and only visible because the admin was editing accounts to fix them.

1 · Password-wipe bug (Irene + systemic) — HIGH.
On login, the app treats "no password_reset_tokens row" as "first-time login" and silently overwrites the user's password with a temp value, then forces a reset. It's the intended onboarding flow, but keyed off a transient table — so any onboarded user who reaches login without a token row gets their working password destroyed → "credentials don't match."
- Irene (irene.tapu@ibis.care) — resolved. Real password, protected by a token row; the reported irene@ was a typo.
- Wider blast radius: 7 users locked out right now (unreported); 485/976 (~50%) exposed on their next login.
- Fix: durable password_set_at flag instead of the token check, never clobber a valid password, backfill existing users. Remediate the 7 now; plan the 485.

2 · 2FA lockout (Annette) — MEDIUM, cause not yet pinned.
Confirmed: password fine; she authenticates but is stuck at 2FA (session has web-guard login, no user_2fa); no successful 2FA since February. Cause (delivery latency vs stale code vs mis-entry) still needs the prod mail-provider log (no creds locally) or a live repro tomorrow. Fix (confirm-first): explicit tunable expires_at, longer window, clearer resend messaging, add send/failed-attempt logging. Rate-limit + single-use → separate security ticket.

Next actions: remediate 7 locked users + plan the 485 · pull Annette's mail logs / live repro · file P1/P2/P3 + security ticket · client reply (qualitative + proactive).






## Todo
- Make plan
- Discuss plan with Jordan
- Send plan to Vykas for 


## Notes
- Is Supabase DB separate from main DB? how are they stayiung in sinc?

## 1. Whether ngrok is actually necessary

This is probably the highest-value thing to investigate before the call.

Microsoft SQL MCP Server supports both hosted HTTP and local `stdio` transport, and Microsoft recommends `stdio` for local development. Claude Desktop also supports locally configured MCP servers. That suggests that, for a **single user running everything on one workstation**, there may be a way to connect Claude Desktop directly to SQL MCP Server without exposing an HTTP endpoint through ngrok at all. You would need to confirm compatibility with his exact Claude Desktop configuration, but I would raise this immediately. citeturn822141search10turn822141search18turn466678search13

Research:

- Can his current DAB/SQL MCP configuration run using `stdio`?
- Can his Claude Desktop setup consume that local MCP server directly?
- Is he using a **remote custom connector** specifically, which would explain the need for ngrok?
- What functionality, if any, would he lose by switching from HTTP to local `stdio`?

My likely position would be: **for a single-user local setup, eliminate the public tunnel if possible**. For a genuinely shared service, move to a properly hosted and authenticated endpoint rather than treating ngrok as permanent infrastructure.

---

## 2. Exactly what protects the current MCP endpoint

Ask whether the ngrok URL itself requires authentication.

There are two different authentication boundaries:

**Claude/MCP client → SQL MCP Server**

and:

**SQL MCP Server → SQL Server**

Microsoft explicitly supports inbound authentication for SQL MCP Server using JWT/OAuth, including Entra ID, or recommends placing an authenticating gateway in front when another authentication scheme is required. citeturn822141search2

ngrok itself can also apply controls including authentication, IP restrictions and mutual TLS. citeturn460614search3turn460614search18

Research or ask:

- Is DAB currently configured for anonymous access?
- Does possession of the ngrok URL effectively give someone access to the MCP tools?
- Is there OAuth, JWT validation or another authentication mechanism?
- Are ngrok access logs enabled?
- Is the endpoint automatically disabled when not being used?

A **temporary and difficult-to-guess URL is not an authentication mechanism**.

---

## 3. The SQL login is broader than the DAB whitelist

This is one point I would definitely bring into the discussion.

The client says the dedicated login has `db_datareader`. Microsoft documents that members of `db_datareader` can read **all data from all user tables and views** in that database. citeturn882758search0turn882758search6

Therefore:

> The DAB entity whitelist restricts what can normally be accessed through DAB, but the underlying database credential itself is not restricted to those entities.

For a proof of concept this may be acceptable, particularly against a test environment. For production, research whether they could instead:

- expose purpose-built reporting views;
- grant the SQL account `SELECT` only on those views or approved tables;
- exclude sensitive columns at the database boundary;
- use separate credentials for this workload.

This creates another security boundary if the DAB configuration is incorrect or the MCP service itself is compromised.

Also remember that **a restored production database still contains production data**. Calling the database "TEST" does not reduce the sensitivity of the information inside it.

---

## 4. What data actually reaches Claude

You need to understand the complete flow:

**User question → Claude → MCP call → SQL MCP Server → SQL → result → SQL MCP Server → Claude/Anthropic → response**

Find out:

- Does Claude receive entire database rows?
- Could a broad question retrieve thousands of records?
- Are names, email addresses, financial information or other personal information included?
- Are there particularly sensitive ERP tables that should never be exposed?
- Can reporting views return aggregated or minimised data instead?

This matters more than simply whether the SQL connection is read-only.

Your key question should be:

> **What is the most sensitive piece of information Claude could receive through the currently exposed entities?**

Until you know that, it is difficult to judge whether the controls are proportionate.

---

## 5. The exact Claude product and account being used

Do not let the discussion simply say "Anthropic's API". Establish precisely:

- Claude Free, Pro or Max?
- Claude Team or Enterprise?
- Direct Anthropic API?
- A company-managed account or his personal account?

The distinction materially affects the data-handling discussion. Anthropic states that commercial-product inputs and outputs are not used for model training by default, and standard API inputs and outputs are generally deleted from its backend within 30 days, subject to stated exceptions. Zero-data-retention arrangements are also available for qualifying organisations. Consumer Claude products have separate policies. citeturn715848search10turn715848search34turn822141search1turn715848search2

I would **not provide a definitive privacy assessment until you know exactly which Claude product he is using**.

---

## 6. Australian privacy and organisational requirements

Before discussing production data, determine whether the ERP contains personal information and whether the organisation is covered by the Privacy Act.

The OAIC's guidance says that organisations using commercially available AI products need to consider their Privacy Act obligations when personal information is involved. Overseas disclosure may also raise APP 8 considerations depending on the circumstances. citeturn460614search7turn460614search8

Research or ask:

- Does the ERP contain customer or employee personal information?
- Is there sensitive information?
- Does the organisation already have an approved AI usage policy?
- Are employees permitted to submit company data to third-party AI providers?
- Does the organisation have a contract or data-processing agreement with Anthropic?
- Does it have data residency requirements?
- Has whoever owns information security/privacy approved this use case?

You do not need to give legal advice. You can say this requires organisational confirmation rather than treating it as solely a technical security question.

---

## 7. What a proper multi-user version would require

A shared version is substantially different from the current proof of concept.

Microsoft's DAB architecture supports authenticated users, role-based permissions, policies and row-level restrictions. SQL MCP Server also supports inbound authentication. citeturn715848search0turn715848search4turn822141search2

Research these components at a high level:

**Identity**
- Individual users authenticate through something like Entra ID.

**Authorisation**
- Users should not necessarily see every exposed entity or every row.
- Access should reflect their normal ERP permissions where appropriate.

**Hosting**
- A centrally managed SQL MCP Server rather than instances running on employee laptops.

**Network**
- Private/internal hosting where possible, or a strongly authenticated public endpoint.

**Auditability**
- Who asked what?
- Which MCP tools were invoked?
- What database queries were generated?
- What information was returned?

**Secrets**
- Centrally managed credentials rather than database passwords on individual machines.

You probably do not need to design this today. The useful distinction is:

> **The current architecture is a personal proof of concept. A shared deployment needs to become a managed application with identity, authorisation, auditing and operational ownership.**

---

## 8. Query controls and data exfiltration limits

Being read-only prevents modification but does not prevent someone from extracting the entire permitted dataset.

Research or raise:

- maximum records returned per query;
- pagination limits;
- query timeouts;
- rate limiting;
- whether users can perform unrestricted aggregation;
- whether particularly sensitive fields can be excluded;
- whether MCP tools need access to every currently exposed entity.

DAB supports permission-aware access and policies around entities and data access, so there is scope for considerably finer controls than simply exposing an entity as readable. citeturn715848search0turn715848search1

---

## 9. Logging and monitoring

Ask him what is currently logged at each layer:

- SQL Server;
- DAB/SQL MCP Server;
- ngrok;
- Claude.

Then consider an important secondary risk: **do the logs themselves contain ERP data?**

A production solution should make it possible to investigate who accessed what without unnecessarily creating additional copies of sensitive query results.

---

## 10. Accuracy and appropriate usage

Finally, separate **database retrieval accuracy** from **AI interpretation accuracy**.

Even if SQL MCP retrieves the correct records, Claude can still:

- misunderstand the question;
- summarise results incorrectly;
- draw unsupported conclusions;
- omit relevant qualifications.

Ask what decisions people intend to make from the answers.

For general exploratory reporting, human verification may be sufficient. For financial, regulatory or operational decisions, you may want reproducible reports or the ability to inspect the underlying records behind an answer.





































## Book to Read
- Pandora's star by peter F Hamilton, and Judas Unchained
- Silver Ships by Jucha
- First Colony by Ken LozitoB


## Childcare
- good
	- Everything looks new
	- Kitchen well set up abd good meals
	- Dedicated breakout rooms for when children need a break.
- Not Good
	- No App, all paper-based.
	- Probably a new centre, not at full capacity yet.
	- Shared playground with all age groups.
	- Orientation was a bit short.
- Two orientation days

## Rust Materials
### Learning Material 
https://fasterthanli.me/articles/a-half-hour-to-learn-rust
The Rust Book
Rust by Example
Rustlings
### News
Read Rust
This Week in Rust

Bin Replacement Request:
REQ2026-075276.97O3JL2Q

# Fixes for Thermonuclear code review
- remove the repetition, make it more DRY
- Focus a lot more in tests.


# TODO
- Fix bug where the facility emails are not showing on the maps screen after switching to multiple emails.
- Move the Email field to a seperate row in the table.
# Security Camera Options:
https://www.jbhifi.com.au/products/eufy-eufycam-s4-security-camera-2-pack
https://www.jbhifi.com.au/products/eufy-s4-nvr-poe-24-7-security-system-2-x-ptz-2-x-turret-cameras

Router:
https://www.pccasegear.com/products/64427/tp-link-archer-be550-be9300-tri-band-wifi-7-router
https://www.pccasegear.com/products/69529/tp-link-archer-axe75-axe5400-tri-band-gigabit-wi-fi-6e-router
https://www.pccasegear.com/products/64301/mercusys-ccmr47be-be9300-tri-band-wifi-7-router

# Security System Installation
## Sites
https://www.wirelesscamerasolutions.com.au/
https://www.melbsecurity.com.au/product-and-services/cctv/
https://shreesecurity.com.au/cctv-camera-installtion/
https://www.topnotchsecurity.com.au/cctv.html

## Questions to Ask
- What brand of camera?
- Is there a subscription fee?
- Can you install POIP Cameras?
- How much for installation?




- [x] app/Repositories/Reports/ReportsRepository.php — havingRaw uses string concatenation instead of parameterized bindings, inconsistent with the selectRaw above it and a latent injection risk if $includedStatusIds ever changes source.
- [x] tests/Unit/ReportsRepositoryTest.php — No test for applications with zero enquiries in activeApplicationsCount; the whereHas silently excludes them and there's no test (or confirmed decision) about whether that's intentional.
- [x] tests/Unit/ReportsRepositoryTest.php — No unit test for activeApplicationsCount respecting the date filter; every other repository method has one.
- [x] .gitignore — Comment # Added by code-review-graph is a tool artefact and shouldn't be committed as-is.
- [ ] In @resources/tests/Pages/Reports/ReportsPage.test.tsx around lines 99 - 107, The assertion uses Number.prototype.toLocaleString without a locale causing flakiness; update the test in ReportsPage.test.tsx so the expected value is deterministic —  call (1205).toLocaleString('en-US') in the expect for activeApplicationsCard?.value to match a fixed locale.
- [x] In @tests/Feature/Portal/ReportsPageTest.php at line 111, The assertion in ReportsPageTest is expecting the wrong active applications count; update the assertion in the test method that checks 'props.activeApplications' (the line currently asserting 1) to assert 0 because PLACED and CONTRACT_COMPLETE applications should be excluded from active count per the "excludes placed applications from active applications count" scenario; locate the assertion ->assertJsonPath('props.activeApplications', 1) and change the expected value to 0 so the test reflects the intended behavior.




---

- Need confirmation for NAB regarding the 99/1 split.
- 50/50 split would speed up the process.


- first hoome buyers declaration form.

# Good Playgrounds
Been:
- Harpley Estate Playground
- Booran Reserve
- Marie Wallace in Bayswater

To Go:
- Royal Children's Hospital in Parkville. 
- Maritime Cove Park in Port Melbourne. 
- JJ Holland in Kensington.
- Hansen Reserve in West Footscray
- Wattle Park in Burwood
- Thomas Street Park in Hampton
- Chelsea Bicentennial Park
- Wombat Bend in Templestowe, 
- Eltham North Adventure playground, 
- Diamond Valley regional playspace in Diamond Creek.

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
