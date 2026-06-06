---
name: headatever
description: Use when bumping, releasing, or pushing a Headatever version (head.yymmdd.patch) — touching the VERSION file, cutting a release, bumping major/head/patch/date, or pushing a release tag to the remote. Triggers on phrases like "release", "bump version", "버전 올려줘", "릴리스", "VERSION 파일", "push tag", "태그 푸시", "릴리스 푸시". Runs a bundled shell script that validates, writes VERSION, commits, tags, and can push or create a GitHub release in one step.
---

# Headatever version bump

Bump the project version (`head.yymmdd.patch`) by running the bundled script.
**Do not hand-edit `VERSION`** — the script validates the result, writes the file,
commits it as `chore(release): v<version>`, and creates the annotated `v<version>` tag.

## Usage

Run (path is relative to this skill's directory):

```bash
scripts/headatever.sh <command> [options]
```

| Command | Use when | Effect |
|---|---|---|
| `show` | You only need the current version | read-only |
| `patch` | Cutting another release | same day → patch+1; new day → date=today, patch=0 |
| `major` | Breaking change or milestone | head+1, date=today, patch=0 |
| `date` | Forcing a fresh release day | date=today, patch=0 (errors if already today) |
| `set <version>` | Correcting to an explicit value | sets `head.yymmdd.patch` (validated) |
| `init [head]` | A project has no `VERSION` yet | creates `VERSION` (default head `0`) |
| `push` | Publishing an already-tagged release | `git push --follow-tags` (release commit + `v<version>` tag) |
| `release` | Publishing an already-tagged release to GitHub | push, then `gh release create v<version> --generate-notes` (needs `gh`) |

Options: `--dry-run` (preview, write nothing), `--no-git` (write VERSION only), `--push` (bump, then `git push --follow-tags`), `--release` (bump, push, then create a GitHub release; implies `--push`, needs `gh`).

## Notes

- **Pick the command by intent:** `patch` is the everyday release; reserve `major` for a human-signaled breaking/milestone release. `head` may be `0` (initial development).
- Dates use **local time**. The script reads/writes `VERSION` at the **git root**.
- It refuses to create an existing tag or to move the date backwards, so a bump always produces a unique, spec-valid version.
- **Pushing:** use `--push` to bump and push in one step; use the standalone `push` command to publish a release you already committed and tagged earlier. `push` errors if the `v<version>` tag does not exist locally.
- **GitHub release:** use `--release` to bump, push, and create a GitHub release in one step; use the standalone `release` command to publish a tag you cut earlier. Both require the GitHub CLI (`gh`) installed and authenticated, and error if a release for `v<version>` already exists.
- When unsure what a command will do, run it with `--dry-run` first.
- Spec: `head.yymmdd.patch`, SemVer-compatible — see the repo `README.md`.
