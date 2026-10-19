---
name: navigator-version-bump
description: >
  Move this repository to a new Navigator release — the four CLI pins, the portal's navigator-ux pin, the synced
  `AGENTS.md` and `.agents/skills/` catalog — then verify both halves: the `ci` gate locally and the live checks `cd`
  runs on `main`. Trigger when asked to update, bump, upgrade, or roll this repository to a Navigator version, or to
  check whether it is on the current release. Repository-local: it covers `sample-transactional` alone.
---

# Bump `sample-transactional` to a Navigator release

`version:` in `navigator.yaml` selects the `navigator` binary the reusable workflows install, and that binary *is* the
check: `project gate` and `project portal`. A bump is finished when `cd` publishes on `main`, not when the PR lands —
`ci` and `cd` call the same `project-gate.yml`, and on a pull request its live steps have no token and pass vacuously.

The repository is private and carries **no licence** (`UNLICENSED`, all rights reserved): every `gh` call needs an
authenticated token, and a bump never adds a `LICENSE` or SPDX tag.

## 1. Preflight

```bash
TARGET=<next>
CURRENT=$(sed -n 's|^version: ||p' navigator.yaml)
gh api repos/neon-law/navigator/releases/tags/"$TARGET" --jq '{tag_name, prerelease, body}'   # must exist
navigator --version                                                                         # must equal TARGET
git status --porcelain && git fetch origin && git rev-parse HEAD origin/main                # clean and synced
```

Upgrade first with `brew upgrade neon-law-source-code/navigator/navigator`: a gate run at the old CLI proves nothing
about the new one. Read the release notes, then diff the two reusable workflows between `CURRENT` and `TARGET`; if their
`on:` block moved, the callers need more than a version edit. Quote the `?ref=` URL, since zsh globs the `?`:

```bash
for w in project-gate project-publish; do
  diff <(gh api "repos/neon-law/navigator/contents/.github/workflows/$w.yml?ref=$CURRENT" --jq .content | base64 -d) \
       <(gh api "repos/neon-law/navigator/contents/.github/workflows/$w.yml?ref=$TARGET" --jq .content | base64 -d)
done
```

An unchanged caller contract is not a mechanical bump: the new CLI can add a gate rule that lands as a hard error. Diff
`cli/src/projects/repository.rs` between the two tags too (26.10.5 added the required `format:check` script and two
synced skills that way).

## 2. Rewrite the pins

Four pins, which must agree. Replace **the exact current string**, never a pattern — a loose regex rewrites prose that
merely mentions a version.

| File | Occurrences |
| --- | --- |
| `navigator.yaml` | 1 — `version:` |
| `.github/workflows/ci.yml` | 1 — `project-gate.yml@<v>` |
| `.github/workflows/cd.yml` | 2 — `project-gate.yml@<v>` and `project-publish.yml@<v>` |

```bash
git switch -c chore/navigator-"$TARGET" origin/main
sed -i '' "s|$CURRENT|$TARGET|g" navigator.yaml .github/workflows/ci.yml .github/workflows/cd.yml
grep -rn "$CURRENT" . --include='*.yml' --include='*.yaml'   # expect nothing
```

The callers carry `uses:` and nothing else — no `with:`, no comments. `navigator.yaml` takes no comments at all (Y011).
Leave its `allowed_prefixes` / `allowed_hosts` block alone; each entry is load-bearing.

**navigator-ux.** On a stable release with a matching asset, move the tarball URL in `portal/package.json` to `v$TARGET`
and refresh the lockfile. Hold it on a release candidate or a CLI-only hotfix, and say so in the PR body:

```bash
gh api repos/neon-law/navigator/releases/tags/"$TARGET" --jq .prerelease          # true -> hold
gh release view "v$TARGET" --repo neon-law-source-code/navigator-ux \
  --json assets --jq '.assets[].name'                                            # no .tgz -> hold
pnpm --dir portal install
```

**AGENTS.md and the skill catalog** are byte-exact copies of what the binary compiled in, and no verb syncs them. Copy
from the target tag, reading `SYNCED_SKILLS` in `cli/src/projects/repository.rs` at that tag rather than trusting a
list. A skill the catalog drops is deleted here in the same PR. This skill is repository-local and not in the catalog;
the gate leaves it alone.

```bash
get() { gh api "repos/neon-law/navigator/contents/$1?ref=$TARGET" --jq .content | base64 -d > "$2"; }
get cli/src/projects/agent_contract.md AGENTS.md
SK=($(get cli/src/projects/repository.rs /dev/stdout | sed -n '/^const SYNCED_SKILLS/,/^];/p' \
  | grep -o 'skills/[a-z-]*/' | cut -d/ -f2))
for s in "${SK[@]}"; do mkdir -p .agents/skills/$s; get .agents/skills/$s/SKILL.md .agents/skills/$s/SKILL.md; done
```

## 3. Verify the `ci` half locally

```bash
pnpm --dir portal check                 # lint, format:check, typecheck, build, test
CI=true navigator project gate --ci     # offline gate; refuses without CI set
```

Run the gate from the repository root after a build — it is whole-repository only, and Y009 means there is no
`portal/dist/` yet. Expect `0 error(s)` on the `Validated Project repository` line; the closing "live Project check is
inconclusive" line is the live half declining to mint locally. The gate resolves every command a contract names, so a
retired verb in `AGENTS.md`, `README.md` or a portal string is a hard error. It does not read shell scripts, so grep
those yourself.

`format:check` runs `oxfmt` as configured in `portal/.oxfmtrc.json`. If a bump moves the `oxfmt` pin, carry the reformat
in its own commit.

## 4. Pre-flight the `cd` half — before the PR opens

**Opening a ready PR lands it**: `automerge.yml` arms auto-merge as soon as the PR is ready, and a draft is only a
pause. So check the live side first. An expired token needs `! navigator site login --host staging.neonlaw.com`.

```bash
navigator site whoami --host staging.neonlaw.com
rm -rf portal/dist && navigator project gate --check    # what `gate / documents` runs on main
navigator project drift --host staging.neonlaw.com --dir ..
```

`gate --check` must report `fixed 0 file(s)`; any pointer it writes would fail `cd` on `main`, so inspect it with
`navigator site document log <pointer>` and commit it. `publish` cannot be pre-flighted; confirm its configuration
instead, and expect `NAVIGATOR_APP_PUBLISHER_BUCKET` beside the WIF provider and service account:

```bash
gh api repos/neon-law/sample-transactional/actions/variables --jq '.variables[].name'
```

## 5. Commit, push, open

```bash
git commit -S -m "chore: move the Navigator pin to $TARGET"    # the ruleset requires signed commits
git push -u origin chore/navigator-"$TARGET"
gh pr create --repo neon-law/sample-transactional --title "chore: Navigator $TARGET" --body "..."
```

If a push touching a workflow is refused over HTTPS, use SSH; where port 22 is blocked, only the `ssh://` form carries
the port:

```bash
git remote set-url origin ssh://git@ssh.github.com:443/neon-law/sample-transactional.git
```

A green PR left unarmed can be merged by hand with `gh pr merge <n> --squash` — squash is the only method allowed.

## 6. Report both halves

```bash
gh pr checks <n> --repo neon-law/sample-transactional --watch    # `ci / ci` is the required check
SHA=$(gh pr view <n> --repo neon-law/sample-transactional --json mergeCommit --jq .mergeCommit.oid)
gh run list --repo neon-law/sample-transactional --workflow cd.yml --commit "$SHA" --json databaseId,conclusion
```

Select the `cd` run by the merge commit, never "the latest" — that can be the previous release's run. A green `cd` is
seven jobs ending in `publish / publish`. `cd` is not required, so a red one is silent after the merge.

**When `cd` goes red.** `gate / verify` 403 ("not bound to a live project") means the Project row's `repository_url`
does not match this repository; `project drift` names the expected value. `gate / documents` 404 on a pointer is a live
fix with `navigator project sync`, never a repo edit. `publish` failing on `gs:///` means the publisher bucket variable
is unset. Re-run with `gh workflow run cd.yml --repo neon-law/sample-transactional --ref main`.
