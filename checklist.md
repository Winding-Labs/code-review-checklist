### Code review checklist

**Before the diff**
- [ ] I know what this change is for and how the author checked it
- [ ] It is one change: no refactor or rename mixed into a behavior change
- [ ] The design fits the system, and nothing is built for a need that doesn't exist yet

**Tests**
- [ ] A test covers the new behavior or the fixed bug
- [ ] That test would fail on the old code
- [ ] Edge cases are covered: empty, largest, duplicate, timeout

**Outside the diff**
- [ ] Callers of anything renamed, removed or retyped are updated
- [ ] Tests, fixtures and snapshots that name it are updated
- [ ] Migrations, schema files and any list that counts them agree
- [ ] Config, env vars and feature flags exist everywhere the code reads them
- [ ] A second read path (cache, export, public API, job) shows the same change
- [ ] README, docs and agent instructions (CLAUDE.md, AGENTS.md) still match

**Failure paths**
- [ ] Errors are handled or passed up, never swallowed
- [ ] A retry cannot charge, send or write twice
- [ ] Two runs at once cannot lose an update
- [ ] Failure logs help debugging and hold no secret or personal data

**Inputs and access**
- [ ] Queries use bound parameters
- [ ] A URL the server fetches cannot reach an internal address
- [ ] Output written into a page or script is escaped
- [ ] Allow and block lists compare normalized values
- [ ] No secret in code, logs, errors or the client bundle
- [ ] Every new route or action checks who is calling (read by hand)
- [ ] Lookups by ID also check the owner or tenant

**Before approving**
- [ ] Blocking comments say what breaks, when, and the fix; optional ones are labeled
- [ ] Every blocking comment is resolved or answered
- [ ] The change leaves the code better than it found it
- [ ] Anyone else who should see it has been asked

<sub>From [Winding-Labs/code-review-checklist](https://github.com/Winding-Labs/code-review-checklist), CC BY 4.0.</sub>
