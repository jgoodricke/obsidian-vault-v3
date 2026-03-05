---
tags:
  - Knowlege
  - Leaf
---
## Overview
- Header: 
  - `<gitmoji> <short summary>`
  - imperative verb, ≤50 chars, no trailing period
- Blank line
- Body (optional): 
  - wrap at 72 chars
  - Describe the code changes as concisely as possible.
  - Don't explain why the changes were made.
  - Use bullet points

EXAMPLE
feat: ✨ add token-refresh endpoint
- Support JWT rotation for long-lived sessions
- Return 401 if refresh token is expired

OTHER
- Don't write body for simple commits. Prefer single-line commit messages where possible.
- Don't use the 🎉 emoji for any commit other than the first.
---

## Rules for Using gitmojis

1. **Always start the commit subject with exactly one gitmoji**, followed by a space.

   ```
   ✨ Add user profile page
   ```

2. **Choose the gitmoji that best represents the *primary purpose* of the commit**.

3. **Do not invent meanings or use emojis outside this list.**

   * Only use the official gitmojis defined below.

4. **Write commit subjects in the imperative mood**.

   * Good: `Fix login redirect`
   * Bad: `Fixed login redirect`

5. **Keep the subject short and focused**.

   * Details belong in the commit body if needed.

6. **When unsure, default to clarity over cleverness**.

   * Feature → ✨
   * Bug fix → 🐛
   * Refactor → ♻️
   * Docs → 📝
   * Tests → ✅

7. **Output requirement (strict)**
   When GitKraken AI generates a commit message, return **only**:

   * the final commit subject (and body if explicitly requested)
   * already prefixed with the correct emoji

---

## Official gitmoji List

### Code & Features

* 🎨 Improve structure / format of the code
* ⚡ Improve performance
* 🔥 Remove code or files
* 🐛 Fix a bug
* 🚑 Critical hotfix
* ✨ Introduce new features
* ♻️ Refactor code
* 💩 Write bad code that needs to be improved
* 🧠 Add or update business logic
* 🧵 Multithreading or concurrency changes
* 🦖 Add backwards compatibility
* 🧪 Add a failing test

### UI / UX

* 💄 Add or update UI and style files
* ♿ Improve accessibility
* 🚸 Improve user experience / usability
* 📱 Responsive design work
* 💫 Animations and transitions

### Documentation & Text

* 📝 Add or update documentation
* 💬 Add or update text and literals
* 💡 Add or update comments in source code
* ✏️ Fix typos
* 🌐 Internationalisation and localisation

### Testing & Quality

* ✅ Add, update, or pass tests
* 🚨 Fix compiler / linter warnings
* 🥅 Catch errors
* 🩺 Add or update healthchecks
* 🦺 Validation-related code

### Build, CI & Tooling

* 👷 Add or update CI build system
* 💚 Fix CI build
* 🔧 Add or update configuration files
* 🔨 Add or update development scripts
* 📦 Add or update compiled files or packages
* 🧰 Infrastructure-related changes
* 🧑‍💻 Improve developer experience

### Dependencies

* ⬆️ Upgrade dependencies
* ⬇️ Downgrade dependencies
* ➕ Add a dependency
* ➖ Remove a dependency
* 📌 Pin dependencies to specific versions

### Versioning, Releases & Deployment

* 🚀 Deploy stuff
* 🔖 Release / version tags
* 💥 Introduce breaking changes
* 🎉 Begin a project

### Files, Assets & Data

* 🚚 Move or rename resources
* 🗃️ Database-related changes
* 🧺 Deprecate code that needs cleanup
* ⚰️ Remove dead code
* 🥚 Add or update an easter egg
* 🧱 Architectural changes
* 🧪 Experiments / spikes
* 🌱 Seed files
* 🏷️ Add or update types
* 🎏 Feature flags
* 📸 Snapshots
* 🧃 Add or update assets

### Security & Auth

* 🔒 Fix security or privacy issues
* 🔐 Add or update secrets
* 🛂 Authorisation, roles, permissions

### Logging, Monitoring & Analytics

* 📈 Add or update analytics
* 🔊 Add or update logs
* 🔇 Remove logs
* 🧐 Data exploration / inspection

### Project & Meta

* 📄 Add or update licence
* 👥 Add or update contributors
* 🙈 Add or update `.gitignore`
* 🔀 Merge branches
* ⏪ Revert changes
* 🚧 Work in progress
* 🤡 Mock things
* 👽 External API changes
* 💸 Sponsorships or money-related infrastructure
* ✈️ Improve offline support
* 🍻 Write code drunkenly (use sparingly, if ever)


