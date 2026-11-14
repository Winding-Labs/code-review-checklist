# Code review comments to save as GitHub saved replies

These are comments for the cases that come up in review every week. Each one says what is wrong, why it matters and what would fix it, so the author can act without asking a follow-up question. Replace the parts in angle brackets before you post.

To save one, open your GitHub Settings, then Saved replies, and add it with the title shown. In any pull request comment box, pick it from the saved replies button above the field ([GitHub Docs](https://docs.github.com/en/get-started/writing-on-github/working-with-saved-replies/using-saved-replies)).

The labels at the start follow [Conventional Comments](https://conventionalcomments.org/).

## Ask for the purpose

```markdown
question (blocking): What should this change do for a user, and how did you check it works? I'd like to review it against that rather than guess. A sentence or two in the description is plenty.
```

## Test that can't fail

```markdown
issue (blocking): This test passes on main as well as on this branch, so it wouldn't catch the bug coming back. Could it assert <the behavior that changed>? Running it once against main is a quick way to check it fails there.
```

## Caller left behind

```markdown
issue (blocking): `<name>` is renamed here, but `<path/to/file>` still calls the old name. A search for `<old name>` across the repo turns up <N> more places. Could you update those in this pull request?
```

## Migration or config left behind

```markdown
issue (blocking): This adds `<migration / setting>`, but `<file that lists or reads it>` wasn't updated, so <what breaks: the release check counts the old number / production starts without the value>. Could you add it there too?
```

## Retry that can run twice

```markdown
issue (blocking): If `<call>` succeeds but the response times out, the caller retries and `<charges / sends / writes>` a second time. Passing `<stable id>` as an idempotency key would make the retry safe.
```

## Lost update under concurrency

```markdown
issue (blocking): Two runs at once can both read `<value>`, add one and write it back, so one increment is lost. Incrementing in the database (`UPDATE ... SET n = n + 1`) avoids the race.
```

## Interpolated query

```markdown
issue (blocking, security): `<variable>` goes into the SQL string directly here. Could it be passed as a bound parameter instead? Then a value with a quote in it can't change the query.
```

## Permission check missing

```markdown
issue (blocking, security): This route loads `<resource>` by ID but doesn't check that it belongs to the caller's `<workspace / org / user>`. Anyone who guesses an ID could read it. Could the lookup filter by owner too?
```

## Nit

```markdown
nitpick (non-blocking): <the small thing>. Fine to leave it if you disagree.
```

## Suggestion with a reason

```markdown
suggestion (non-blocking): <the change>, because <why it is better: shorter, one less query, clearer to the next reader>. Not needed for this pull request.
```

## Out of scope

```markdown
note: This isn't something this pull request needs to fix, but <the thing> looks off. I've opened <#issue> so it doesn't get lost.
```

## Approve with small comments

```markdown
Approving: this makes <the area> better as it is. The comments left are non-blocking, so take them or leave them.
```

## Request changes

```markdown
Requesting changes for the <N> blocking comments above: <one-line summary of each>. Everything else is optional. Happy to look again as soon as they're in.
```
