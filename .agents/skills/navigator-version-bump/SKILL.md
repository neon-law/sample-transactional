---
name: navigator-version-bump
description: >
  Upgrade this source-only Project repository to a published Navigator release, synchronize its canonical agent files,
  and verify the pull-request gate and delivery run. Use for release upgrades or current-release checks in
  sample-transactional.
---

# Upgrade sample-transactional

Use the repository's existing authoritative forge. A release upgrade preserves source history, document identities and
client visibility. Keep the original checkout's unrelated changes separate from the upgrade.

## Establish the release and baseline

Verify the target exists with `gh release view <tag> --repo neon-law/navigator` and install that published CLI.
`navigator --version` must report the target. Read its release notes and check changes to the reusable workflow contract
and Project gate before editing. A source-built binary with different embedded canonical files does not prove a
published release upgrade.

Start from a clean branch based on the latest `origin/main`. Preserve unrelated work rather than stashing or reverting
it. Fetch and signed-rebase with `git fetch origin main` and `git rebase -S origin/main`.

If the release pins and canonical files already match, report that fact and investigate actual gate or delivery
failures. Do not create an empty version-bump PR.

## Make the smallest compatible change

Keep these four pins on the same exact published tag:

| File | Pin |
| --- | --- |
| `navigator.yaml` | Top-level `version` |
| `.github/workflows/ci.yml` | `project-gate.yml@<tag>` |
| `.github/workflows/cd.yml` | `project-gate.yml@<tag>` and `project-publish.yml@<tag>` |

Run `navigator project repository sync-skills --dir .` with the target release. It restores that release's canonical
`AGENTS.md`, skills, permission settings and required ignore rules while retaining custom skills such as this one.
Review its complete diff. Change authored Drafts or Matter Pages only for demonstrated gate incompatibilities. A
source-only Project has no portal application or package build.

Do not run `navigator project sync` merely to update the CLI. It can upload staged documents and synchronize live data.
Document bytes stay in Navigator-managed storage and ignored local folders. A pointer move or rename must preserve its
asset identity, revision chain, SHA-256, provenance and visibility. Compare those fields before and after any necessary
layout repair; never discard an unmatched pointer or staged file.

## Verify the gate and live checks

Run `CI=true navigator project gate --ci` from the repository root. Require zero content, layout and pointer errors.
Missing local CI credentials or an inconclusive live check is not a live verification pass.

Use the deployment's normal OAuth session for local read checks: `navigator site whoami --host <host>` and `navigator
project gate --check`. Resolve a refused or inconclusive live check before claiming deployment readiness. Inspect any
changed pointer rather than committing a server mismatch as an upgrade fix.

Before opening a ready PR, confirm repository Actions is enabled and `main` requires the actual `ci / ci` check from
GitHub Actions. Preserve PR-only squash merging and branch cleanup. Use Navigator's canonical governance reconciler when
these controls drift; review its dry-run first.

## Deliver and verify the exact result

Commit the reviewed change with a signed commit, push the topic branch and open a ready PR. Confirm auto-merge is armed
or the PR has merged. If necessary, use the GitHub auto-merge fallback in `AGENTS.md`; the required gate still governs.

Inspect `ci / ci` on the exact PR head. After merge, obtain its commit with `gh pr view <number> --repo
neon-law/sample-transactional --json mergeCommit` and inspect the `cd.yml` run for that exact commit with `gh run list
--repo neon-law/sample-transactional --workflow cd.yml --commit <merge-sha>`.

A failed or missing delivery run remains unfinished work. Investigate its failed job and retain source and document
history during repairs. A successful no-op publication job alone does not prove source application: verify the relevant
live rows and document revision metadata before reporting that the upgrade was applied.
