# Layer 2 PR Review Architecture

Layer 2 is the review system that sits between local implementation agents and merge. It is designed to reduce PR spam, preserve high-quality review, and avoid running expensive model reviews on every tiny intermediate commit.

## Target Architecture

```text
Layer 1 local agent
  -> branch: claude/<workstream> or codex/<workstream>
  -> one draft PR with review packet
  -> GitHub PR router
  -> Layer 2 Claude Code reviewer runner
  -> advisor decision + labels/checks/comments
  -> Layer 1 fixes same branch if needed
  -> merge only after gate passes
```

## Components

### PR Router

File: `.github/workflows/pr-router.yml`

Runs on PR open, synchronize, label changes, draft/ready transitions, and edits.

Responsibilities:

- Create standard review labels.
- Classify risk by changed file paths.
- Mark low-risk PRs as router-only when allowed.
- Mark release or cross-repo PRs as `agent-review:run`.
- Clear stale approvals when the PR head changes.
- Post or update a routing comment.

### Required Gate

File: `.github/workflows/layer2-gate.yml`

This is the branch-protection check. It fails closed until the PR is approved or explicitly classified as router-only low risk.

Required branch protection:

- Require status check: `layer2-gate / gate`.
- Require existing CI checks.
- Prevent direct pushes to `main`.
- Require conversation resolution.
- Do not let Layer 1 agents merge their own PRs.

### Local Claude Code Reviewer Runner

Canonical implementation handoff: `docs/LAYER2_CLAUDE_CODE_HANDOFF.md`.

Initial deployment can be a local full-time runner on Dominic's machine or a small VPS/self-hosted runner. It watches the three repos for `agent-review:run`, builds a review packet, asks reviewer models for analysis when configured, and posts one advisor decision.

The runner should not edit source repos. It should only read repository and PR data, run safe verification in disposable worktrees when useful, post comments, and update labels/checks.

## Reviewer Roles

Advisor:

- Primary adjudicator.
- Intended model: Claude Opus via Claude Code for the first version.
- Produces final decision: approve, changes requested, needs human, or comment only.

GPT reviewer:

- Secondary thoroughness pass for missed edge cases, release safety, and regression risk.

Gemini reviewer:

- Third-platform perspective for broad consistency and release workflow drift.

Specialists:

- Release reviewer for binaries, checksums, signing, installer, updater, and public distribution.
- Contract reviewer for platform download metadata and agent build provenance.

## Decision Labels

- `agent-review:pending`: PR is waiting for review.
- `agent-review:run`: reviewer runner should process the PR.
- `agent-review:router-only`: low-risk PR can pass without expensive model review.
- `agent-review:approved`: Layer 2 approved the current head commit.
- `agent-review:changes-requested`: Layer 2 found blocking issues.
- `agent-review:needs-human`: human decision required.
- `agent-review:disputed`: Layer 1 disputes a finding.

## False Positive Handling

Layer 2 must include enough evidence for every blocking finding. Layer 1 may dispute with evidence. The advisor reviews the dispute and either clears the label or keeps the block.

