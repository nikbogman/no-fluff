# Comments & Docstrings

Detailed guidance for code comments, inline comments, and docstrings/API
reference comments. Read this when the work involves source files.

## Contents

- [Comments \& Docstrings](#comments--docstrings)
  - [Contents](#contents)
  - [Never delete these](#never-delete-these)
  - [The two litmus tests](#the-two-litmus-tests)
  - [What to delete](#what-to-delete)
  - [What to keep](#what-to-keep)
  - [Check the history first](#check-the-history-first)
  - [Adding missing docs](#adding-missing-docs)
  - [Worked examples](#worked-examples)
  - [Pass flow](#pass-flow)

---

## Never delete these

Some comments are code. They look like noise to a pattern-matcher and deleting
any of them changes behavior.

**Tool directives** — anything a compiler, linter, formatter, type-checker, or
build tool parses:

```python
# noqa: E501
# type: ignore
# pragma: no cover
# fmt: off / # fmt: on
# -*- coding: utf-8 -*-
#!/usr/bin/env python3
```
```ts
// eslint-disable-next-line no-console
// @ts-expect-error
// prettier-ignore
/** @jsx h */
/** @deprecated */
```
```go
//go:embed assets/*
//go:generate mockgen ...
//nolint:gosec
```

Also: `<!-- prettier-ignore -->` and similar in Markdown, `# rubocop:disable`,
`// NOSONAR`, `@SuppressWarnings` and their comment equivalents, license
identifiers like `// SPDX-License-Identifier: MIT`, and framework-significant
annotations in comment form.

**Rule of thumb:** if a comment might be read by a tool rather than a human,
leave it. A redundant-looking directive costs nothing; a deleted one costs a
broken build or a silently re-enabled lint error.

**Executable examples in docstrings** — these are tests, not prose:

```python
def add(a, b):
    """Return the sum.

    >>> add(2, 3)
    5
    """
```
Python doctests, Rust `///` doc examples, and any example block a test runner
executes must not be compressed, reworded, or reformatted. Their exact text is
the assertion.

---

## The two litmus tests

Apply both to every comment — *after* confirming it isn't in
[Never delete these](#never-delete-these):

1. **Disappearance test** — if this comment vanished, would a competent
   developer still understand the code? If yes, delete it.
2. **Why-vs-what test** — does it explain *why*, or only *what*? If only
   *what*, delete it.

Don't replace a useless comment with a shorter useless comment. `// Get the
user` doesn't improve by becoming `// get user`. It improves by being deleted.

If the code can explain it, let the code explain it.

---

## What to delete

**Narration of visible code:**
```ts
// Get the user
const user = getUser(id)

// Check if the user exists
if (user) {

// Loop through all users
for (const user of users) {

// Return the response
return response
```

**Structural labels the syntax already provides:**
```java
// Constructor
public UserService(Repository repo) { ... }

// Getter for name
public String getName() { return name; }

// End of if block
}
```

**Standard language or framework behavior:**
```python
# The __init__ method initializes the class
# This decorator makes it a route handler
# useEffect runs after render
```

Also delete:
- Restatements of the parameter list in prose when types already convey it.
- Unfilled IDE/tool-generated docstring skeletons (`@param x the x`,
  `<summary>Gets or sets the Name.</summary>`).
- Commented-out code (version control has it).
- Changelog comments in source (`// 2019-04-02 JD: fixed bug`) — belongs in git history.
- TODOs with no owner, no date, and no actionable content.
- Section-divider banners that just repeat the next function's name.

---

## What to keep

Keep comments that carry information the code can't:

| Category | Example |
|---|---|
| Why a decision was made | `// Sequential, not parallel — the vendor API rate-limits per connection` |
| Non-obvious business rules | `// Orders under £5 skip fraud check per finance policy 2023-11` |
| External constraints | `// Safari <16 rejects this header; sent as a query param instead` |
| Workarounds | `// Manual retry: the SDK's built-in retry swallows the 429 body` |
| Compatibility requirements | `// Keep Python 3.8 compatible — build servers aren't upgraded yet` |
| Edge cases | `// Empty string is valid here and means "inherit from parent"` |
| Performance reasons | `// Materialised deliberately; the lazy version re-queries per row` |
| Security considerations | `// Constant-time compare to avoid a timing oracle` |
| Still-relevant history | `// Kept for the v1 clients that never migrated; remove after 2027-01` |
| Surprising behavior | `// Returns stale data for up to 60s by design — see cache TTL` |

Also keep **contracts** a caller can't infer from the signature: units, valid
ranges, side effects, error conditions, thread-safety, ownership of returned
objects.

```python
def poll(timeout):
    """Wait for the next event.

    timeout is in milliseconds. Returns None on timeout; raises
    ConnectionLost if the socket closed. Not safe to call concurrently.
    """
```

---

## Check the history first

The odd-looking comment is often the important one. A comment that seems
redundant *but is oddly specific* — a magic number, a named vendor or browser,
a version, a date, a defensive check with no obvious cause — usually got there
for a reason that isn't in the file.

Before deleting one of those:

```bash
git log -S "<distinctive phrase>" --oneline -- <file>
git blame -L <line>,<line> -- <file>
```

Read the commit message. Then decide:

| Comment arrived with | Verdict |
|---|---|
| A bug fix, hotfix, incident response, or revert | **Keep.** The specificity *is* the information. |
| A commit referencing a ticket, CVE, or vendor issue | **Keep**, and make sure the reference survives. |
| The original feature commit, saying nothing the code doesn't | Delete as normal. |
| A bulk reformat or a "add comments" commit | Delete as normal. |

If history is unavailable (shallow clone, no VCS) and the comment is oddly
specific, keep it. Deleting information is irreversible; leaving one extra
line costs almost nothing.

Skip this check for clearly generic comments like `// loop through users` —
it's for the ones that made you pause.

---

## Adding missing docs

Coverage is not the goal — only fill genuine gaps.

Add a minimal docstring when **all** of these hold:
- The symbol is public/exported (part of an interface others call).
- It has *no* documentation at all.
- Its purpose isn't obvious from its name and signature.

Keep additions to one to three lines: what it does, key params/returns if
non-obvious, notable side effects or errors. Then stop.

Do **not** add docstrings to:
- Private/internal helpers with self-explanatory names.
- Trivial accessors, simple wrappers, or obvious constructors.
- Test functions whose names already describe the case.

---

## Worked examples

**Verbose docstring → one line:**
```python
# Before
def get_user(user_id):
    """
    This function is used to get a user.
    It basically takes a user_id and returns the user object.
    Note that if the user is not found, None will be returned.
    """

# After
def get_user(user_id):
    """Return the User with this id, or None if not found."""
```

**Redundant JSDoc in TypeScript:**
```ts
// Before
/**
 * Formats a date.
 * @param {Date} date - The date to format.
 * @param {string} locale - The locale.
 * @returns {string} The formatted date.
 */
function formatDate(date: Date, locale: string): string

// After — types are in the signature; only the non-obvious bit survives
/** Formats as a short date. Falls back to en-GB for unknown locales. */
function formatDate(date: Date, locale: string): string
```

**What survives a block comment:**
```python
# Before
# This function processes the payment. First it validates the amount,
# then it checks the currency is supported, then it calls the gateway.
# We use a 30 second timeout here. The gateway sometimes returns a 200
# with an error body instead of a proper error status, so we have to
# check the body content as well as the status code.

# After
# Gateway returns 200 with an error body on some failures — check the
# body, not just the status.
```
The step-by-step narration is visible in the code below it. The gateway quirk
isn't, so it stays.

---

## Pass flow

1. Read the file top to bottom before editing — you need the whole picture to
   spot duplication between a docstring and the comments beneath it, and to
   infer the docstring convention already in use. Match it; never convert a
   codebase from one style to another as part of a trimming pass.
2. Screen out everything in [Never delete these](#never-delete-these) before
   evaluating anything.
3. For each remaining comment/docstring, apply the two litmus tests.
4. For anything oddly specific, [check the history first](#check-the-history-first).
5. Apply the priority order from SKILL.md: delete > compress > convert >
   move/reference > keep.
6. Check for information that belongs in a doc rather than a comment (setup
   steps, architecture rationale) — move it and link, don't duplicate.
7. Only then consider gaps worth filling.
8. Verify nothing that survived is now stale or contradicts the code.

If a file has nothing worth cutting, leave it untouched and say so. A clean
file is a valid result — don't reword working comments to show effort.

Never change behavior to make a comment removable, beyond the narrow exception
in SKILL.md's Constraints section.