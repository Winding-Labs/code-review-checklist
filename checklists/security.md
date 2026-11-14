# Security code review checklist

Start where trust changes. List what the pull request adds that the outside world can reach (routes, background jobs, webhooks, and any value from a user, a request header or a site setting), then follow each one to where it is used. Check it is bound, validated or escaped at that spot, not only where it came in.

Each item below comes with a real bug from an open-source pull request, and how many of 11 AI reviewers caught it when IonWarp ran them on 13 pull requests with 18 known security issues. The answer key on that set is provisional and the set is small, so the counts show which kinds of bug slip past a first pass, not which reviewer is best. The data is in [Winding-Labs/ai-code-review-benchmark](https://github.com/Winding-Labs/ai-code-review-benchmark).

## Input that reaches a query

- [ ] Every value reaches SQL, a shell or a template engine as a bound parameter or an argument list, never through string interpolation.

A Discourse admin controller built SQL by interpolating host names into the string. It should have bound them as parameters. 5 of 11 reviewers caught it.

## URLs the server fetches

- [ ] A URL that comes from a user or a setting is checked against an allow list, and cannot resolve to localhost, a private range or a cloud metadata address.

A Discourse job called `open()` on a URL from site settings, so whoever could set it could make the server fetch internal addresses. 6 of 11 caught it.

## Input that reaches a page

- [ ] Anything written into HTML, an attribute, a script or a URL is escaped for that exact context.

The same Discourse pull request wrote the Referer header into JavaScript without escaping it, which is cross-site scripting. 4 of 11 caught it.

## Allow lists and block lists

- [ ] Both sides of the comparison are normalized first: lowercased, trimmed, Unicode-normalized.
- [ ] A domain match is anchored, so `example.com` does not also match `evil-example.com` or `example.com.evil.net`.

A case-sensitive lookup let `User@Example.com` past a block on `user@example.com` (10 of 11 caught it), and a regex with no boundary let `evil.example.com` past an entry for `example.com` (2 of 11).

## One-time tokens under load

- [ ] A token, code or coupon that may be used once is spent in a single atomic write, such as an update with a condition or a transaction, not read, checked in memory and written back.

Cal.com checked 2FA backup codes in memory and then wrote them back, so two parallel logins could spend the same code. 3 of 11 caught it.

## Login and OAuth flows

- [ ] The OAuth `state` value is random per request and checked on the way back.
- [ ] Redirect targets after login are checked against an allow list.

A Sentry GitHub OAuth flow used a static signature as its state value, not a random one per request. 4 of 11 caught it.

## Response headers

- [ ] Framing, content-type and CORS headers are as strict as before, or stricter.
- [ ] No decision trusts a header any client can set, such as Referer.

Discourse set `X-Frame-Options` to `ALLOWALL` and trusted the Referer header, so clickjacking protection was off. 3 of 11 caught it.

## Permission logic (read this part yourself)

- [ ] Every new route, job or action checks who is calling and what they may touch.
- [ ] A cached permission is invalidated when the grant is revoked, and grants and denials are cached by the same rule.
- [ ] A lookup by ID also checks the owner, tenant or workspace it belongs to.

Grafana trusted cached grants but rechecked cached denials, so a stale grant could reach a revoked resource. A Keycloak check looked up a resource by the wrong owner, ignoring per-client permissions. None of the 11 reviewers caught either. Seeing them takes knowing how the system names and caches its resources, so a person who knows the system should read every permission change.

## Secrets

- [ ] No token, key or password in the code, a test fixture, a log line, an error message or the client bundle.
- [ ] Secret scanning runs on the repository, so a pattern a person misses is still caught.

## Further reading

The long version of all of this is OWASP's [Code Review Guide](https://owasp.org/www-project-code-review-guide/).
