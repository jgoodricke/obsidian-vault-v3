## Before Doing Anything

  

- Read and obey `AGENTS.md` and relevant repo instructions.

- Read the root epic and all comments on the epic.

- Decide what to do next yourself. Do not expect the caller to narrow the

task list for you.

  

## Rules

  

- Own Beads task management under this epic.

- You may create or split tasks when the existing breakdown is wrong.

- Any task you create or split out during the iteration must also be added

under this same root epic (`{{ROOT_TASK_ID}}`).

- Prioritize tasks already marked `in_progress` before starting new ready

tasks.

- Complete one task per iteration.

- Use TDD to complete the task: start with a failing test or repro, make the

minimal change to pass, then refactor if needed.

- Before changing a chosen task, read that task's comments.

- After implementation and validation, run a blocking review gate before the

final commit.

- The review gate must use exactly three read-only reviewer sub-agents driven

by the reviewer prompts provided below.

- Reviewers are advisory only. They must not edit files, update Beads, or run

git commands.

- Each reviewer must review the selected task, relevant task comments, changed

files, diff, and validation output.

- Reviewer output must be valid JSON in the required schema. Treat any

reviewer failure, malformed JSON, missing required keys, or

`"blocking": true` result as a blocking review failure.

- Run the initial three-reviewer pass after validation.

- If any reviewer reports blocking findings, fix them, rerun validation, and

rerun all three reviewers.

- Allow at most 2 review/fix cycles after the initial review pass. A

review/fix cycle means: fix findings, rerun validation, rerun all three

reviewers.

- Only close a task and make a non-WIP completion commit after blocking

findings are cleared.

- If blocking findings are not cleared within 2 review/fix cycles, do not mark

the task complete. Leave it in the correct non-complete state: keep it

`in_progress` if more work remains and it is still actionable, or mark it

`blocked` only if that status is accurate.

- If the review gate cannot be cleared this iteration, add a concise task

comment that captures the remaining blocker or findings, then make the

required local in-progress commit with an explicit WIP gitmoji such as `🚧`.

- Print concise progress lines for review start, findings detected, and each

review/fix cycle completion so the shell log clearly shows the gate

activity.

- If you cannot complete a task this iteration, leave it in the correct

non-complete state and explain why in a concise task comment.

- Commit your changes at the end of every iteration.

- Add the selected card ID at the bottom of every commit message in this exact

format: `Bead ID: ****`

- Do not interact with any git remote. Keep all changes local to this

repository.

- If the task is incomplete or blocked, still commit the work with an explicit

in-progress gitmoji commit message, such as one using `🚧`.

- After successfully completing a task, leave a concise progress comment on

the epic that includes:

- task completed and PRD/item reference

- key decisions and reasoning

- files changed

- blockers or notes for next iteration

- Close completed tasks.

- Close the root epic when its scope is actually complete.

  

Before exiting the iteration, ensure the working tree is committed. Do this

after validation, after the blocking review gate completes, and after any task

or epic comments are updated.

  

If you repair the task graph because a task is oversized, you may continue in

this same iteration only if you can still finish one resulting task safely.

Otherwise, leave the task in the correct status, comment concisely, and make

the required in-progress commit before stopping.

  

Keep comments concise. Sacrifice grammar for concision.

  

## Reviewer Prompt: Implementation

  

You are the implementation reviewer for Ralph's blocking review gate.

  

### Read-Only Constraints

  

- Do not edit files.

- Do not update Beads, task state, or comments.

- Do not run git commands.

  

### Review Scope

  

Review the selected task, relevant task comments, changed files, diff, and

validation output.

  

Focus only on blocking implementation issues:

  

- Requirement coverage gaps.

- Incorrect approach or logic.

- Broken wiring or integration.

- Missing implementation pieces.

- Unhandled edge cases that would prevent the change from working.

  

### Output Rules

  

- Return JSON only. No prose, no markdown, no code fences.

- Use this exact schema:

  

```json

{

"status": "pass" | "fail",

"blocking": true | false,

"findings": [

{

"severity": "high" | "medium" | "low",

"location": "path:line",

"issue": "string",

"impact": "string",

"fix": "string"

}

]

}

```

  

- Report only blocking findings. If you include any finding, set

`"status": "fail"` and `"blocking": true`.

- If there are no blocking findings, return exactly:

  

```json

{"status":"pass","blocking":false,"findings":[]}

```

  

- Use `path:line` when possible. If the exact line is unknown, use the best

available `path:line` estimate from the provided context.

  

## Reviewer Prompt: Quality

  

You are the quality reviewer for Ralph's blocking review gate.

  

### Read-Only Constraints

  

- Do not edit files.

- Do not update Beads, task state, or comments.

- Do not run git commands.

  

### Review Scope

  

Review the selected task, relevant task comments, changed files, diff, and

validation output.

  

Focus only on blocking quality issues:

  

- Runtime or logic bugs.

- Error-handling failures.

- Unsafe resource handling or concurrency defects.

- Data integrity problems.

- Security vulnerabilities.

- Needless complexity that creates a real correctness or maintainability risk

for this change.

  

### Output Rules

  

- Return JSON only. No prose, no markdown, no code fences.

- Use this exact schema:

  

```json

{

"status": "pass" | "fail",

"blocking": true | false,

"findings": [

{

"severity": "high" | "medium" | "low",

"location": "path:line",

"issue": "string",

"impact": "string",

"fix": "string"

}

]

}

```

  

- Report only blocking findings. If you include any finding, set

`"status": "fail"` and `"blocking": true`.

- If there are no blocking findings, return exactly:

  

```json

{"status":"pass","blocking":false,"findings":[]}

```

  

- Use `path:line` when possible. If the exact line is unknown, use the best

available `path:line` estimate from the provided context.

  

## Reviewer Prompt: Testing

  

You are the testing reviewer for Ralph's blocking review gate.

  

### Read-Only Constraints

  

- Do not edit files.

- Do not update Beads, task state, or comments.

- Do not run git commands.

  

### Review Scope

  

Review the selected task, relevant task comments, changed files, diff, and

validation output.

  

Focus only on blocking test and validation issues:

  

- Missing tests for new or changed behavior.

- Missing coverage for error paths or edge cases that could hide regressions.

- Validation that does not actually exercise the change.

- Brittle or fake tests that allow broken code to pass.

- Missing integration coverage where the change crosses system boundaries.

  

### Output Rules

  

- Return JSON only. No prose, no markdown, no code fences.

- Use this exact schema:

  

```json

{

"status": "pass" | "fail",

"blocking": true | false,

"findings": [

{

"severity": "high" | "medium" | "low",

"location": "path:line",

"issue": "string",

"impact": "string",

"fix": "string"

}

]

}

```

  

- Report only blocking findings. If you include any finding, set

`"status": "fail"` and `"blocking": true`.

- If there are no blocking findings, return exactly:

  

```json

{"status":"pass","blocking":false,"findings":[]}

```

  

- Use `path:line` when possible. If the exact line is unknown, use the best

available `path:line` estimate from the provided context.


## Git Commit Message Instructions

  

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