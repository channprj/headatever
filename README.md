# Headatever

> **`head.yymmdd.patch`** — a calendar‑stamped, SemVer‑compatible versioning scheme where only the **head** needs a human. The rest is just the date and a daily counter.

[한국어 (Korean)](./README.ko.md)

**Headatever** is the day‑precision sibling of [HeadVer](https://github.com/line/headver). It keeps HeadVer's central promise — *only the head number is worth maintaining by hand* — but swaps HeadVer's ISO **year‑week** field for a full **calendar date** (`yymmdd`).

HeadVer decided that week precision was good enough. Headatever stamps the exact day instead — **whatever**, the day is cheaper to reason about, impossible to misread, and free of ISO week‑numbering edge cases. The name is a portmanteau of **Head**Ver + Cal**Ver** + "what**ever**".

---

## TL;DR

- **Format:** `head.yymmdd.patch`, released and tagged as `vhead.yymmdd.patch`.
- **`head`** is the only field a human ever touches. Bump it for a breaking change or a milestone release.
- **`yymmdd`** is generated from the release date; **`patch`** is a zero‑based counter that resets every day.
- Every valid Headatever version is also a valid [Semantic Version](https://semver.org) and **sorts identically** — existing SemVer tooling needs no changes.
- You never argue about whether a change is "minor" or "patch" again. The calendar and the counter decide.

---

## Format

```
  head  .  yymmdd  .  patch
   │         │          └── zero-based daily release counter (0, 1, 2, …)
   │         └───────────── 2-digit year (2000s) + zero-padded month + zero-padded day
   └─────────────────────── manually incremented release line (≥ 0)
```

| Component  | Source     | Example  | Meaning                                            |
| ---------- | ---------- | -------- | -------------------------------------------------- |
| `head`     | **manual** | `1`      | The release line; bump for breaking/milestone work |
| `yymmdd`   | automatic  | `260424` | The calendar date `2026-04-24`                      |
| `patch`    | automatic  | `0`      | The 1st release of that day (zero‑based)            |

> **`1.260424.0`** → head `1`, released on **2026‑04‑24**, the `0`th (first) release that day.
> Canonical version: `1.260424.0` · Git tag: `v1.260424.0`

---

## Fields

### `head`

- A **non‑negative integer** (`≥ 0`) with **no leading zeros** (a lone `0` is allowed).
- Incremented **manually** to signal a breaking change or a milestone release — analogous to SemVer's `major` and HeadVer's `head`.
- **Monotonically non‑decreasing** over the project's lifetime; it is never reset.
- `0` denotes **initial development**, exactly as in SemVer's `0.x` and HeadVer's zero‑based head: a `0.*` line makes no stability promises. Bump to `1` for your first stable or milestone release.
- When adopting Headatever on existing software, start `head` at (or above) your current major version.

### `yymmdd`

A single 6‑digit integer built from the release date:

- **`yy`** — the last two digits of the year, interpreted in the **2000s** (`26` → `2026`). Range `00`–`99`.
- **`mm`** — zero‑padded month, `01`–`12`.
- **`dd`** — zero‑padded day of month, `01`–`31`.

Because it is a plain integer, it increases monotonically with the calendar:
`260424` < `260425` < `261231` < `270101`.

> **Timezone:** the date SHOULD be computed in a single, documented timezone — **UTC is RECOMMENDED** — so the daily counter stays consistent across build machines and contributors.

### `patch`

- A **non‑negative integer** (`≥ 0`) with **no leading zeros** (a lone `0` is allowed).
- A **zero‑based daily counter**: `0` for the first release of a given `yymmdd`, then `+1` for each subsequent release that same day.
- MUST reset to `0` whenever `yymmdd` changes.
- Typically incremented **automatically** by the build/release pipeline, so duplicate versions are impossible even if a manual bump is skipped.

---

## Rules

The key words **MUST**, **MUST NOT**, **SHOULD**, and **MAY** follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

1. A version **MUST** match `head.yymmdd.patch`.
2. `head` **MUST** be `≥ 0` and **MUST NOT** contain leading zeros (a lone `0` is allowed).
3. `yymmdd` **MUST** be a structurally valid date: `mm` in `01`–`12`, `dd` in `01`–`31`. Full calendar validity (e.g. rejecting `260230`) **SHOULD** be enforced in code.
4. `yy` **MUST** be interpreted as a year in the 2000s (`26` → `2026`).
5. `patch` **MUST** be `≥ 0`, **MUST** be `0` for the first release of a day, **MUST** increment by exactly `1` for each additional same‑day release, and **MUST** reset to `0` when `yymmdd` changes.
6. The canonical version **MUST** live in a repo‑root **`VERSION`** file, stored **without** a leading `v`.
7. `VERSION` **MUST** be updated before a release tag is created.
8. Git release tags **MUST** be `v{VERSION}` (i.e. `vhead.yymmdd.patch`); any manual tags or pipeline inputs **MUST** match this exactly.
9. A version **MAY** carry SemVer pre‑release / build‑metadata suffixes (e.g. `1.260424.0-rc.1`, `1.260424.3+sha.9f2a`) without ceasing to be a valid Headatever version.

---

## Source of truth

- The canonical version lives in the repo‑root **`VERSION`** file.
- `VERSION` stores the version **without** the leading `v`.
- Git release tags use **`v{VERSION}`**.

| Artifact      | Value          |
| ------------- | -------------- |
| `VERSION`     | `1.260424.0`   |
| Git tag       | `v1.260424.0`  |
| CLI / display | `1.260424.0`   |

---

## Ordering & SemVer compatibility

Headatever borrows SemVer's three‑number layout exactly. Precedence is determined by comparing `head`, then `yymmdd`, then `patch`, each **numerically**:

```
0.260424.0  <  1.260424.0  <  1.260424.1  <  1.260425.0  <  1.261231.9  <  2.260101.0
```

- Because `yymmdd` is a monotonically increasing integer, **ordering by date "just works"** under SemVer precedence — no custom comparator needed.
- Every Headatever version (for years `≥ 2010`, where `yymmdd` carries no leading zero) is therefore a **valid Semantic Version** and sorts identically. Existing tooling — npm, Cargo, pip, Go modules, generic sort routines — handles it with zero changes.
- The relationship is one‑directional: **every Headatever version is a SemVer version, but not every SemVer version is a valid Headatever version.** Headatever additionally constrains the middle field to a real date and the last field to a daily counter.

---

## Validation

A structural regular expression (ECMAScript flavor):

```regex
^(?<head>0|[1-9]\d*)\.(?<yymmdd>\d{2}(0[1-9]|1[0-2])(0[1-9]|[12]\d|3[01]))\.(?<patch>0|[1-9]\d*)$
```

> This checks **structure** (month `01`–`12`, day `01`–`31`). It does **not** reject impossible calendar dates such as `260230` (Feb 30) — validate those in code.

---

## Examples

**Valid**

| Version       | Meaning                                            |
| ------------- | -------------------------------------------------- |
| `0.260424.0`  | head 0 — an initial‑development line                |
| `1.260424.0`  | head 1, first release on 2026‑04‑24                 |
| `1.260424.9`  | head 1, tenth release on the same day               |
| `2.261231.0`  | head 2, first release on 2026‑12‑31                 |
| `1.260424.0-rc.1` | release candidate (optional SemVer suffix)      |

**Invalid**

| Version        | Why                                       |
| -------------- | ----------------------------------------- |
| `1.26042.0`    | date is not 6 digits                      |
| `1.261399.0`   | month `13` / day `99` do not exist        |
| `01.260424.0`  | `head` must not have leading zeros        |
| `1.260424.00`  | `patch` must not have leading zeros       |
| `v1.260424.0`  | the `VERSION` file stores no leading `v`  |

---

## Why Headatever?

Three problems, three deliberate choices:

1. **SemVer makes you argue.** Is this change "minor" or "patch"? The debate is endless and the answer rarely matters to anyone downstream. Headatever removes the question — the date and the daily counter are decided automatically. **Only `head` is a human decision**, reserved for the one signal that genuinely matters: "this release is different."

2. **CalVer forgets the breaking‑change signal.** Pure calendar versions communicate *when*, but not *whether you should be careful upgrading*. Headatever keeps a SemVer‑style leading `head` precisely for that.

3. **HeadVer rounds to the week.** HeadVer's `yyww` is compact, but ISO week numbering has rough edges — 53‑week years, locale‑dependent week starts, and a number that humans can't map back to a date at a glance. Headatever stamps the full day: unambiguous, human‑readable, and trivially sortable.

In short, Headatever is **CalVer's date core + SemVer's head + a daily build counter**, arranged so the whole thing is still a valid Semantic Version.

| Trait                         | SemVer            | CalVer              | HeadVer            | **Headatever**       |
| ----------------------------- | ----------------- | ------------------- | ------------------ | -------------------- |
| Format                        | `major.minor.patch` | varies (e.g. `YYYY.MM.DD`) | `head.yyww.build` | `head.yymmdd.patch` |
| Fields a human maintains      | major, minor, patch | usually none      | head               | **head only**        |
| Date precision                | none              | day / month / week  | ISO week           | **calendar day**     |
| Auto‑generated fields         | none              | the date            | yearweek, build    | yymmdd, patch        |
| Breaking‑change signal        | major             | —                   | head               | head                 |
| Sorts as SemVer               | yes               | depends             | yes                | **yes**              |
| "Is this minor or patch?" debate | yes            | no                  | no                 | **no**               |

---

## Claude Code skill

This repo bundles a [Claude Code](https://www.anthropic.com/claude-code) skill — [`headatever/`](./headatever/SKILL.md) — that performs spec‑valid bumps for you: it computes the next `head.yymmdd.patch`, writes `VERSION`, commits it as `chore(release): v<version>`, and creates the annotated `v<version>` tag.

### Install

Install it with the [`skills`](https://github.com/vercel-labs/skills) CLI — no clone required:

```bash
# Project scope → ./.claude/skills/headatever
npx skills add channprj/headatever

# Global scope → ~/.claude/skills/headatever
npx skills add channprj/headatever -g
```

The CLI also works with other agents (Cursor, Codex, opencode, …); target one with `-a`, e.g. `npx skills add channprj/headatever -a claude-code`. List and remove installed skills with `npx skills list` and `npx skills remove headatever`.

### Use

Once installed, ask the agent to "bump the version" / "release" / "릴리스", or run the bundled script directly:

```bash
.claude/skills/headatever/scripts/headatever.sh patch        # everyday release
.claude/skills/headatever/scripts/headatever.sh major        # breaking / milestone
.claude/skills/headatever/scripts/headatever.sh patch --push # bump, then git push --follow-tags
.claude/skills/headatever/scripts/headatever.sh show         # print current version
```

Add `--dry-run` to any command to preview without writing. See [`headatever/SKILL.md`](./headatever/SKILL.md) for the full command reference.

---

## FAQ

**Why day precision instead of HeadVer's week?**
A full date is unambiguous and human‑readable: you always know *exactly* when a build shipped, with none of ISO‑8601's week‑53 or locale‑week complications. HeadVer reasoned that the week was enough; Headatever just stamps the day — *whatever*, it's cheaper to reason about.

**What if I release more than once in a day?**
Increment `patch`: `…​.0`, `…​.1`, `…​.2`, … for that `yymmdd`. It resets to `0` on the next day.

**Which timezone defines "the day"?**
Pick one and document it. **UTC is recommended** so the daily counter is consistent across machines and time zones.

**Is `head` ever reset?**
No. It is monotonically non‑decreasing for the lifetime of the project. `0` simply means you haven't cut your first milestone release yet.

**Can I use pre‑release tags like `-rc.1` or build metadata like `+sha.9f2a`?**
Yes. SemVer suffixes are allowed and keep the version SemVer‑compatible.

**How do I adopt it on an existing project?**
Start `head` at (or above) your current major version (or at `0` for greenfield, initial‑development work), set `VERSION` accordingly, and let your pipeline fill in `yymmdd` and `patch` from then on.

---

## References and Inspired by

- [https://calver.org](https://calver.org)
- [https://github.com/line/headver](https://github.com/line/headver)
- [https://x.com/simnalamburt/status/2047318484137443342](https://x.com/simnalamburt/status/2047318484137443342)
- [https://github.com/violetstair](https://github.com/violetstair)
