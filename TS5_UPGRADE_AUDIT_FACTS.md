# TypeScript 5 Upgrade Proposal — Source-of-Truth Fact Sheet

**Audit scope:** Every claim below was checked directly against the `microsoft/TypeScript` repository's own records via the GitHub REST/GraphQL API on 2026-10-22. No blog posts, secondhand write-ups, or memory were used as evidence.

---

## 1. Verified Fact — The "v5.0.0" release: does it exist?

**Finding: A release/tag named `v5.0.0` does NOT exist in the microsoft/TypeScript repository.**

- `GET /repos/microsoft/TypeScript/releases/tags/v5.0.0` → **404 Not Found**
- `GET /repos/microsoft/TypeScript/git/ref/tags/v5.0.0` → **404 Not Found**

What the repository's release history actually shows for the TypeScript 5.0 line:

| What exists | Release name | Publish date | Prerelease flag |
|---|---|---|---|
| `v5.0-beta` | "TypeScript 5.0 Beta" | 2023-01-27T18:57:34Z | `true` (prerelease) |
| `v5.0-rc` | "TypeScript 5.0 RC" | 2023-03-16T22:39:46Z | `true` (prerelease) |
| **`v5.0.2`** | **"TypeScript 5.0"** | **2023-03-16T22:41:23Z** | **`false` (not a prerelease)** |
| `v5.0.3` | "TypeScript 5.0.3" | 2023-04-05T20:41:33Z | `false` (not a prerelease) |
| `v5.0.4` | "TypeScript 5.0.4" | 2023-04-07T17:53:55Z | `false` (not a prerelease) |

**Implication for the proposal:** Any reference to a `v5.0.0` release or tag is wrong. The first *stable* TypeScript 5.0 release shipped under the git tag **`v5.0.2`**, is named **"TypeScript 5.0"**, was published **2023-03-16T22:41:23Z**, and is **not** flagged as a prerelease (a `v5.0.1` tag/release also does not exist; the RC tag `v5.0-rc` is the closest prior tagged artifact, published 2023-03-16T22:39:46Z and flagged as a prerelease).

---

## 2. Verified Fact — The git tag for the stable 5.0 release

The first stable 5.0 release is tagged **`v5.0.2`**.

- **Tag → commit SHA:** `3ec598c947cdcda4349f7f8615d6344c9673235f`
  - (Note: `v5.0.2` is a *lightweight* tag — the ref points directly at the commit; `GET /repos/microsoft/TypeScript/git/tags/3ec598c...` returns 404 because no annotated tag object exists.)
- **Commit top-level message:** `Update LKG`
- **Commit author date:** 2023-03-15T23:58:05Z
- **Author/committer:** TypeScript Bot (`typescript-bot`), email `typescriptbot@microsoft.com`
- **Commit link:** https://github.com/microsoft/TypeScript/commit/3ec598c947cdcda4349f7f8615d6344c9673235f

---

## 3. Verified Fact — The run-up: three PRs merged most recently in the week ending on the release's publish date

**Week under audit (derived from the release's own publish date, 2023-03-16T22:41:23Z):** 2023-03-10 through 2023-03-16 (7 days ending on publish day). 35 PRs were merged into `microsoft/TypeScript` during that week.

The **three most recently merged** PRs of that week:

| # | PR | Title | Author | Merged at (UTC) | Still merged? | In release notes? |
|---|---|---|---|---|---|---|
| 1 | **#52628** | Add an extra test case related to return statements and annotated return types | @Andarist | 2023-03-16T21:58:04Z | Yes (`merged: true`) | No |
| 2 | **#53297** | Set minimal workflow permissions | @pnacht | 2023-03-16T20:19:41Z | Yes (`merged: true`) | No |
| 3 | **#53117** | Skip asking for jsdoc in checkObjectLiteral for enums in TS | @jakebailey | 2023-03-16T20:14:54Z | Yes (`merged: true`) | No |

**In the release notes? — None of the three appear in the release notes.** The `v5.0.2` release notes link only to milestone-based "fixed issues" queries (`milestone:"TypeScript 5.0.0"`, `"TypeScript 5.0.1"`, `"TypeScript 5.0.2"`), and all three of these PRs are:
- assigned **no milestone**, and
- labeled **"For Uncommitted Bug"** ("PR for untriaged, rejected, closed or missing bug").

Therefore none of them is captured by the milestone queries the release notes reference.

**Merge-status confirmation:** Each PR was fetched individually and each shows `merged: true` with a recorded `merged_at` timestamp, i.e. all three still show as merged.

---

## 4. Verified Fact — The repository's latest release right now

- **Name:** "TypeScript 7.0.2"
- **Tag:** `v7.0.2`
- **Publish date:** 2026-08-20T18:09:49Z
- **Prerelease:** `false` (not a prerelease)
- **Link:** https://github.com/microsoft/TypeScript/releases/tag/v7.0.2

---

## 5. Verified Fact — Community pulse: most recently active discussion thread

**Finding: GitHub Discussions are disabled on `microsoft/TypeScript`.**

- The repository object reports `has_discussions: false`.
- `GET /repos/microsoft/TypeScript/discussions` and the GraphQL discussion list/category queries all return zero results (there are no discussion categories).

**Closest available community pulse — the repository's most recently active issue thread:** [#64895 — "panic: Unhandled case in Node.MemberList: KindVariableDeclarationList on merged namespace/enum element access"](https://github.com/microsoft/TypeScript/issues/64895)

- **Original post (@muh-syaipullah, 2026-10-21T07:20:42Z):** Reports that `tsc` **panics (crashes)** instead of emitting a type error when compiling code that declares a merged namespace and enum named `Foo` where the enum's computed member accesses the namespace's exported constant (e.g. `namespace Foo { export const BAR = 42; } enum Foo { BAZ = Foo["BAR"] }`). The reporter says this regressed **between TypeScript 6.0.3 and 7.0.2**, notes that TypeScript 5.x handles the same code without panicking, that 5.8.3 instead throws a JavaScript TypeError rather than a hard crash, and that it is a variant of earlier issue #64668 (a `KindModuleBlock` panic) triggered by `export const` instead of `export class`, adding that fix PR #64746 targets `KindModuleBlock` and likely doesn't cover `KindVariableDeclarationList`. Labeled "Bug".
- **Commenter (@Andarist, 2026-10-21T08:11:39Z):** Replies that the crash is "already being fixed by https://github.com/microsoft/TypeScript/pull/64746."

**Summary:** The community thread is a regression report about the compiler panicking (hard crash) on merged namespace/enum code — introduced in the TypeScript 7 line — with a maintainer confirming an in-flight fix (PR #64746). Relevance to a TS5 upgrade proposal: it is further evidence that TypeScript 5.x-era behavior is being compared against the current 7.x line, where regressions like this one are being actively triaged.

---

## Method notes

- All lookups performed against the `microsoft/TypeScript` repository through the GitHub API on 2026-10-22.
- "Most recently active" thread determined by latest `updated_at` among issues with at least one comment (Issues are the venue used, since Discussions are disabled on this repository).
- The three run-up PRs were ordered by `merged_at` timestamp (most recent first) within the audited week.
