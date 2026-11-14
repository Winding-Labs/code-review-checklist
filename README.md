# Code review checklist

A code review checklist for the person reading a pull request, in the order you read it. Read the description before the diff. Check the tests would fail without the change. Search outside the diff for every caller, test, migration, config and doc that had to change with it. Walk the failure paths. Follow each new input to where it lands, and read permission changes by hand. Then label what blocks the merge and what doesn't, and approve once the change makes the code better, even if it isn't perfect.

The rest of this page explains each step and the real bugs behind it. The same list comes in three forms you can copy:

- [`checklist.md`](checklist.md), short enough to paste into a review comment.
- [`checklists/security.md`](checklists/security.md), the security pass, item by item, with the open-source bug behind each one.
- [`saved-replies.md`](saved-replies.md), review comments for the cases that come up every week, ready to save as GitHub saved replies.

There is also a [GitHub Action](https://github.com/Winding-Labs/code-review-checklist/blob/main/examples/.github/workflows/review-checklist.yml) that posts your team's checklist as a comment on every new pull request, so nobody has to remember it.

IonWarp's team made this, and the numbers in it come from our own graded reviews and the open data behind [ionwarp.com/benchmarks](https://ionwarp.com/benchmarks?utm_source=github&utm_medium=owned&utm_campaign=trusted-sites). Every other claim links to the public source it comes from.

## Why another checklist

Most code review checklists are one long list of questions in no order, and the reviewer is left to guess which ones matter on this pull request. This one is shorter and ordered the way a review actually goes, and each step says what to look for and why. It puts the weight where we have seen reviews fail.

Two AI readers independently graded 267 findings from IonWarp's reviewers, nearly all from 24 to 29 September 2026. Seventeen of them were real merge blockers, and 12 of those 17 were not a wrong line in the diff. They were a test, a migration list, a config value or a second read path that the pull request should have changed and didn't. A reviewer who reads only the diff cannot see them, because they are the lines that aren't there. Our reviewers raised all 17, so the sample leans toward what they look for, but it is the reason step 3 below exists.

People find fewer bugs in review than you might expect, too. In a Microsoft study of its own reviews, only about 15% of reviewers' comments pointed to a possible defect, and at least half were about long-term maintainability ([Czerwonka, Greiler and Tilford, "Code Reviews Do Not Find Bugs", 2015](https://microsoft.com/en-us/research/wp-content/uploads/2015/05/PID3556473.pdf)). A checklist helps a reviewer spend attention on the parts where bugs hide.

## The checklist, step by step

### 1. Read the description before the diff

Know what the change is meant to do, and why, before you judge how it does it. If the description doesn't say, ask for that first: a review of code whose purpose you are guessing at is a review of your guess. Open the linked issue if there is one.

Then ask whether the change belongs here at all. Google's review guide puts design first, before any line of code: do the pieces fit together, does this belong in this codebase or in a library, and is now the right time for it ([Google, What to look for in a code review](https://google.github.io/eng-practices/review/reviewer/looking-for.html)).

- [ ] I know what this change is for and how the author checked it.
- [ ] It is one change. A refactor, a rename and a behavior change are separate pull requests.
- [ ] The design fits the rest of the system, and nothing is built for a need that doesn't exist yet.

### 2. Check that the tests can fail

A test that passes with or without the change proves nothing. Google's guide asks the same question directly: will the tests actually fail when the code is broken? Look for the assertion that would catch the bug the change fixes, or the behavior it adds.

- [ ] A test covers the new behavior or the fixed bug.
- [ ] That test would fail on the old code. If you can't tell, ask the author to run it once against main.
- [ ] Edge cases are covered: empty input, the largest input, a duplicate, a timeout.

### 3. Follow the change outside the diff

This is the step most reviews skip, and the one behind 12 of the 17 blockers above. For anything the pull request renames, removes, retypes or adds, search the repository for the other places that name it. In one of our own pull requests, a release gate went up to 36 migrations and a test still expected 35. In another, a kind of run was hidden from the leaderboard and from events, and the public runs endpoint kept serving it.

- [ ] Every caller of a renamed, removed or retyped function, field or route is updated.
- [ ] Tests, fixtures and snapshots that name it are updated.
- [ ] Migrations, schema files and any list that counts them agree.
- [ ] Config, environment variables and feature flags exist everywhere the code reads them.
- [ ] A second read path (a cache, an export, a public API, a background job) shows the same change.
- [ ] Docs, the README and agent instructions such as `CLAUDE.md` or `AGENTS.md` still match.

### 4. Walk the failure paths

The happy path is the one the author ran. Read what happens when the network times out, the queue redelivers, the row is missing or two requests arrive at once. Google's guide asks reviewers to look for concurrency problems in particular, because they are very hard to find by running the code.

- [ ] Errors are handled or passed up, never swallowed. Nothing logs a failure and carries on as if it worked.
- [ ] A retry cannot charge, send or write twice. A payment or a message takes an idempotency key.
- [ ] Two runs at once cannot lose an update. A counter is incremented in the database, not read, changed and written back.
- [ ] Logs on the failure path say enough to debug it and contain no secret or personal data.

### 5. Follow each new input to where it lands

List what the pull request adds that the outside world can reach: routes, jobs, webhooks, and any value from a user, a header or a setting. Follow each one to where it is used, and check it is bound, validated or escaped there, not only at the edge. [`checklists/security.md`](checklists/security.md) has the full pass with a real bug for each item.

- [ ] Values reach a query as bound parameters, never interpolated into the SQL string.
- [ ] A URL the server fetches cannot point at an internal address.
- [ ] Anything written into a page or a script is escaped for that context.
- [ ] Allow lists and block lists compare in a normalized form (case, trailing dots, subdomains).
- [ ] No token, key or password in code, logs, error messages or the client bundle.

### 6. Read permission changes by hand

We ran 11 AI reviewers on 13 open-source pull requests with 18 known security issues. Two of the issues none of them caught were permission logic: Grafana trusting cached grants while rechecking cached denials, and a Keycloak check that looked up a resource by the wrong owner. Seeing bugs like these takes knowing how the system names and caches its resources, which is what a teammate brings. The labels on that set are provisional, so read it as a pattern, not a ranking.

- [ ] Every new route or action checks who is calling and what they may touch.
- [ ] A cached permission expires or is invalidated when the grant is revoked.
- [ ] A lookup by ID also checks the owner, tenant or workspace.

### 7. Label every comment as blocking or not

Without labels, an author may read every comment as mandatory. Google's guide suggests starting optional comments with "Nit:", "Optional:" or "FYI:" ([Google, How to write code review comments](https://google.github.io/eng-practices/review/reviewer/comments.html)). [Conventional Comments](https://conventionalcomments.org/) goes further, with labels such as `issue:`, `suggestion:`, `question:` and `nitpick:`, plus a `(blocking)` or `(non-blocking)` decoration.

A useful blocking comment names the line, the input that breaks it, what happens and the fix. "This retry can charge the order twice when the response times out; pass the order ID as the idempotency key" can be acted on. "Is this safe?" cannot. When the fix is a few lines, put it in a suggestion block so the author can commit it in one click.

- [ ] Each blocking comment says what breaks, when, and what would fix it.
- [ ] Style points a formatter or linter could catch are left to the formatter or linter.
- [ ] Optional comments are marked as optional.

### 8. Approve once the change makes things better

Google's standard is the one we use: approve a change once it definitely improves the overall health of the code, even if it isn't perfect ([Google, The Standard of Code Review](https://google.github.io/eng-practices/review/reviewer/standard.html)). Holding a good change for polish costs the team more than the polish is worth. Respond fast as well. Google's guide sets one business day as the longest a review request should wait for a first response ([Google, Speed of Code Reviews](https://google.github.io/eng-practices/review/reviewer/speed.html)).

On GitHub, submit the review as Approve, Request changes or Comment. A Comment review neither approves nor blocks ([GitHub Docs, About pull request reviews](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)), so use Request changes when something must change, and say what.

- [ ] Every blocking comment is resolved or answered.
- [ ] The change leaves the code better than it found it.
- [ ] Anyone else who should see it (security, the owning team, the on-call) has been asked.

## When the author is a coding agent

An agent-written pull request gets the same eight steps, with two differences. Ask who read it: a person should have read every line before asking for a review, and the description should say which agent wrote it and what the person checked. And spend longer on step 3. An agent edits the files it opened, so look hardest at the ones it didn't: the caller, the test or the config that should have changed too.

## Where an AI reviewer fits

An AI reviewer is a fast first pass on steps 3 to 5, and it reads every pull request without being asked. It isn't the whole review. On the 50 pull requests in our October code review benchmark, which come from Martian's open code-review-bench with 158 known issues across Cal.com, Discourse, Grafana, Keycloak and Sentry, there were 29 where none of the nine reviewers we ran, apps and models alike, found every known issue. Steps 1, 6 and 8 need someone who knows what the change is for. The per-PR results are open data in [Winding-Labs/ai-code-review-benchmark](https://github.com/Winding-Labs/ai-code-review-benchmark).

## Post the checklist on every pull request

[`examples/.github/workflows/review-checklist.yml`](https://github.com/Winding-Labs/code-review-checklist/blob/main/examples/.github/workflows/review-checklist.yml) is a GitHub Actions workflow that posts a checklist as a comment when a pull request opens or leaves draft, once per pull request. To use it:

1. Copy [`checklist.md`](checklist.md) to `.github/review-checklist.md` in your repository and edit it for your team.
2. Copy the workflow to `.github/workflows/review-checklist.yml`.
3. Merge both to your default branch.

It runs on `pull_request_target` so it can comment on pull requests from forks, and for that reason it never checks out or runs the pull request's code. It reads the checklist from your base branch through the API and needs only `pull-requests: write`.

## Save the comments you write every week

[`saved-replies.md`](saved-replies.md) has review comments for the cases that come up again and again: a test that can't fail, a caller left behind, a retry that can run twice, a nit. To save one on GitHub, open Settings, then Saved replies, and add it with a title. In any pull request comment box you can then pick it from the saved replies button above the field ([GitHub Docs, Using saved replies](https://docs.github.com/en/get-started/writing-on-github/working-with-saved-replies/using-saved-replies)).

## Sources

- Google, [Code review developer guide](https://google.github.io/eng-practices/review/): what to look for, the standard of code review, comment labels, speed.
- Czerwonka, Greiler and Tilford, [Code Reviews Do Not Find Bugs](https://microsoft.com/en-us/research/wp-content/uploads/2015/05/PID3556473.pdf), Microsoft, 2015.
- [Conventional Comments](https://conventionalcomments.org/).
- OWASP, [Code Review Guide](https://owasp.org/www-project-code-review-guide/).
- GitHub Docs on [pull request reviews](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews) and [saved replies](https://docs.github.com/en/get-started/writing-on-github/working-with-saved-replies/using-saved-replies).
- IonWarp's graded findings and benchmark, open data in [Winding-Labs/ai-code-review-benchmark](https://github.com/Winding-Labs/ai-code-review-benchmark) (CC BY 4.0).

## About and license

Made by the team behind [IonWarp](https://ionwarp.com/?utm_source=github&utm_medium=owned&utm_campaign=trusted-sites), an AI code reviewer for GitHub pull requests, at Winding Labs. Corrections and additions are welcome as issues or pull requests; please link the source for any fact.

The checklists and text are [CC BY 4.0](LICENSE): copy them into your own docs, templates and wikis, with credit. The workflow in `examples/` is [MIT](LICENSE-CODE).
