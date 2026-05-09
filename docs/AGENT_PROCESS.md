# Centurion Agent Process

This process applies to the three Centurion repos that ship the platform and desktop product:

- `centurion-platform`: website, accounts, billing, downloads, pairing, APIs, contracts.
- `centurion-agent`: Windows-native Centurion AI desktop agent and local runtime.
- `centurion-agent-public-releases`: signed public binary release mirror.

The goal is commercial-grade engineering flow without creating 50 to 60 small PRs per day.

## Layers

Layer 1 is implementation or release preparation. Local agents such as Claude Code and Codex change files, verify them, and prepare one PR for a coherent workstream.

Layer 2 is review and gatekeeping. A Claude Code reviewer runner watches pending PRs, builds a review packet, asks model reviewers for analysis when configured, posts a decision, and updates GitHub labels/checks. Layer 2 does not merge and does not silently rewrite the PR branch.

## Layer 1 Branch Workflow

1. Start from the correct base branch.
2. Create one branch for the workstream:

```powershell
git checkout main
git pull --ff-only
git checkout -b claude/<workstream>
```

Use `codex/<workstream>` for Codex, `claude/<workstream>` for Claude Code, and `agent/<workstream>` for mixed/manual work.

3. Make as many commits as needed on that branch. Commits are cheap checkpoints.
4. Keep related work together. If a subtask is only useful as part of the same release or fix, keep it in the same branch and PR.
5. Open one draft PR when there is enough context for CI and review.
6. Keep pushing to that branch until the workstream is complete.
7. Mark the PR ready only after self-review and verification.
8. Wait for Layer 2. If changes are requested, update the same branch and same PR.

## Public Release PR Standard

Every release PR must preserve provenance:

- Source repo and commit that produced the artifact.
- Release version.
- Binary names and sizes if applicable.
- Checksums if applicable.
- Signing/notarization status if applicable.
- Download URL or GitHub Release target if applicable.
- Manual verification performed.

Do not invent provenance. If verification is missing, say it is missing and leave the PR blocked.

## One Workstream, One PR

Use one PR for one release, one release-note update, one metadata correction, or one documentation workstream.

Split only when changes can be reviewed, shipped, and reverted independently.

## Risk Routing

Low risk:

- Documentation-only changes that do not alter install behavior, release trust, license terms, signing claims, or download URLs.

Medium risk:

- Release documentation changes that affect user guidance but not trust or artifacts.

High risk:

- Binaries, checksums, signatures, release metadata, download URLs, publishing workflows, GitHub Actions, branch protection, license, terms, or latest-release claims.

Cross-repo risk:

- Any release that depends on a specific `centurion-agent` commit or must be coordinated with `centurion-platform` download/pairing behavior.

## Layer 2 Gate

The GitHub workflow `.github/workflows/pr-router.yml` classifies each PR and labels it.

The workflow `.github/workflows/layer2-gate.yml` is the required merge gate. It passes only when:

- Layer 2 labels the current PR head `agent-review:approved`, or
- The router classifies a non-draft PR as `risk:low` and `agent-review:router-only`.

Branch protection should require the `layer2-gate / gate` check before merge.

## How Layer 1 Consumes Layer 2 Feedback

When Layer 2 requests changes:

```powershell
gh pr view <number> --comments
gh pr checks <number>
gh pr diff <number>
```

Then fix the same branch, push the same PR, and comment with the update and verification result.

If Layer 2 appears wrong, add `agent-review:disputed` and provide evidence. A human or advisor model decides whether to clear the block.

## Merge Rule

Only merge when CI is green, `layer2-gate / gate` passes for the current head commit, and no unresolved blocking review thread remains.

