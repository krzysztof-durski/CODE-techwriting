# Code comments

This document shows seven comments from the [watchpapa.tv source code](https://github.com/watchpapa/watchpapa). Each example is a code block copied without changes. It includes the code that the comment describes, so you can see how the two fit together.

> [!NOTE]
> The code blocks come from the original watchpapa.tv repository at commit `4786d6c`, which is not part of this repository. The line numbers refer to that commit.

## Record a decision as a contract

The comment: `worker/test/ics.test.js`, lines 4 to 11

```js
// Contract for the ICS serializer (RFC 5545) — decided 2026-10-25:
// - every line ends with CRLF
// - a line is at most 75 octets (UTF-8 bytes); longer lines are folded by inserting CRLF + one space
// - a multi-byte character is never split across two lines
// - TEXT values escape backslash, semicolon, comma and newline
// - all-day events use DTSTART;VALUE=DATE:YYYYMMDD
// - the serializer is pure: `now` is passed in, so the same input always gives the same bytes, whatever the machine's timezone
// - invalid input throws a RangeError
```

The code that follows two of its rules: `worker/src/calendar/ics.js`, lines 3 to 17

```js
const TEXT_ESCAPES = {
  "\\": "\\\\",
  ";": "\\;",
  ",": "\\,",
  "\r\n": "\\n",
  "\r": "\\n",
  "\n": "\\n",
};

export function escapeText(text) {
  if (typeof text !== "string") {
    throw new RangeError("text must be a string");
  }
  return text.replace(/\\|;|,|\r\n|\r|\n/g, (match) => TEXT_ESCAPES[match]);
}
```

The comment lists the rules of the iCalendar (ICS) serializer, the part of the code that writes calendar files. The date shows when they were decided. Two of the rules appear in the code below it. The rule "TEXT values escape backslash, semicolon, comma and newline" is the `TEXT_ESCAPES` table. The rule "invalid input throws a RangeError" is the `throw new RangeError` line. The comment lives in the test file because the tests check each rule.

## Explain a security choice and what happens when it fails

The comment: `Frontend/src/lib/pwned.js`, lines 1 to 10

```js
// Breached-password check against HaveIBeenPwned's "Pwned Passwords" range API.
//
// Replaces Supabase's Pro-only "leaked password protection". k-anonymity: only
// the first 5 hex chars of the password's SHA-1 leave the browser; the match is
// done locally against the ~800 suffixes HIBP returns. The plaintext password
// and the full hash are never sent.
//
// Fails OPEN — a HIBP outage, timeout or odd response must never block a signup
// or password reset. Pure and dependency-injected (`fetch`) so it's unit-tested
// in plain node (tests/unit/pwned.test.js).
```

The code that implements the "fails open" rule: `Frontend/src/lib/pwned.js`, lines 65 to 70

```js
    return await Promise.race([request, timeout]);
  } catch {
    return 0;
  } finally {
    clearTimeout(timer);
  }
```

The comment answers two questions that a reviewer would ask: what leaves the browser, and what happens when the breach check breaks. The code backs up the second answer. If anything goes wrong, the `catch` block returns `0`, which means "no breach found," so the sign-up continues. Without the comment, that `return 0` could look like a bug.

## Explain a one-line choice

The comment and the function: `Frontend/src/lib/followGate.js`, lines 10 to 17

```js
// Local YYYY-MM-DD, not toISOString() (which is UTC and can be a day off for
// users west/east of UTC around midnight).
export function todayKey() {
  const d = new Date();
  const mm = String(d.getMonth() + 1).padStart(2, "0");
  const dd = String(d.getDate()).padStart(2, "0");
  return `${d.getFullYear()}-${mm}-${dd}`;
}
```

A developer might replace this function with `toISOString()`, which looks shorter. The comment explains why that would be wrong: `toISOString()` uses Coordinated Universal Time (UTC), so the date can be off by one day for users in other time zones around midnight. The function builds the date from `getFullYear()`, `getMonth()` and `getDate()`, which use the local time zone.

## Say why a check reads the database

The comment and the start of the function: `worker/src/auth.js`, lines 45 to 51

```js
// Requires profile.role === 4. DB-checked (a crafted JWT cannot spoof it).
export async function requireAdmin(c, next) {
  const user = c.get("user");
  const sql = getSql(c);
  try {
    const rows = await sql`SELECT role FROM public.profile WHERE id = ${user.id}`;
    if (Number(rows[0]?.role) !== 4) return c.json({ error: "Forbidden" }, 403);
```

The comment says that the admin role is checked in the database and that a forged token cannot fake it. The code does exactly that. It ignores the token and runs `SELECT role FROM public.profile`, then compares the result with `4`. Someone who sees this query might want to remove it to save a database call. The comment explains why they should not.

## Explain why a setting exists

The comment and the setting: `worker/src/db.js`, lines 27 to 37

```js
    // Parse int8/bigint as JS numbers (tmdb ids + our bigint PKs are all well
    // within Number.MAX_SAFE_INTEGER). Without this, postgres.js returns them as
    // strings, which breaks numeric comparisons like `role !== 4`.
    types: {
      bigint: {
        to: 20,
        from: [20],
        parse: (x) => (x === null ? null : Number(x)),
        serialize: (x) => x.toString(),
      },
    },
```

The setting `types.bigint` converts large integers from the database into JavaScript numbers. The comment gives the reason: without it, the library returns them as strings, and a comparison such as `role !== 4` stops working. It also explains why the conversion is safe, because the ids in this project stay well within the safe number range.

## Explain why the order matters

The comment and the function: `worker/src/tmdb/client.js`, lines 30 to 43

```js
// `language` is set right after `api_key`, before `params`, on every call site —
// keeping that position stable matters because Cloudflare's edge cache keys on
// the full URL, so a caller-supplied `language` (or any `params` key) naturally
// partitions the cache without any extra work.
function buildUrl(path, params, apiKey, language = "en-US") {
  const u = new URL(BASE + path);
  u.searchParams.set("api_key", apiKey);
  u.searchParams.set("language", language);
  for (const [k, v] of Object.entries(params ?? {})) {
    if (v === undefined || v === null || v === "") continue;
    u.searchParams.set(k, String(v));
  }
  return u.toString();
}
```

The code sets `language` right after `api_key` and before the other parameters. That order looks like a style choice but it is not. The comment says that Cloudflare's edge cache uses the full uniform resource locator (URL) as its key, so a stable position keeps the cache working. A developer who reorders those lines could break the cache without any error message.

## Explain why a variable lives outside the function

The comment and the code: `worker/src/auth.js`, lines 4 to 18

```js
// Verify the Supabase access token locally against the project JWKS (ES256).
// No network round-trip to Supabase Auth per request — the JWKS is fetched once
// and cached by jose (module-level singleton keyed by SUPABASE_URL).

let jwks = null;
let jwksUrl = null;

function getJwks(supabaseUrl) {
  const url = `${supabaseUrl}/auth/v1/.well-known/jwks.json`;
  if (!jwks || jwksUrl !== url) {
    jwks = createRemoteJWKSet(new URL(url));
    jwksUrl = url;
  }
  return jwks;
}
```

The variables `jwks` and `jwksUrl` sit outside `getJwks`, so they survive between requests. The comment gives the reason: the key set is fetched once and then reused, so a request does not need a network call to Supabase. The `if` statement in the function builds the key set on first use and again only if the project URL changes.

## Summary

Each comment above answers a question that the code leaves open: why this rule, why this order, why this extra step. Each one sits next to the code that it explains. When you write a comment, ask what a reader would wonder about the code, and answer that.
