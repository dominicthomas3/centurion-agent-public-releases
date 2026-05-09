# Handoff To Claude Code: Build Layer 2 PR Reviewer

You are Claude Code acting as the Layer 2 reviewer builder for Centurion. Your job is to build a local runner that watches GitHub PRs across the Centurion repos and performs high-quality PR review before merge.

## Mission

Build a new local project at:

```text
C:\Users\domin\centurion-review-orchestrator
```

The runner reviews PRs for:

- `dominicthomas3/centurion-platform`
- `dominicthomas3/centurion-agent`
- `dominicthomas3/centurion-agent-public-releases`

It should detect PRs labeled `agent-review:run`, review the current head commit, post a structured advisor comment, and update labels so `.github/workflows/layer2-gate.yml` passes or blocks.

## Non-Negotiables

- Do not edit source repos while acting as reviewer.
- Do not push code to PR branches.
- Do not merge PRs.
- Do not approve without reading the diff and changed-file context.
- Do not mark approved if the head SHA changed after review started.
- Every blocking finding needs file evidence, impact, and fix direction.
- No secrets in logs, comments, prompts, or stored packets.

## Build Target

Use a TypeScript CLI unless there is a strong reason not to.

Suggested structure:

```text
centurion-review-orchestrator/
  package.json
  tsconfig.json
  README.md
  .env.example
  src/
    cli.ts
    config.ts
    github/
      client.ts
      labels.ts
      prs.ts
      comments.ts
      checks.ts
    packet/
      buildPacket.ts
      redact.ts
      risk.ts
    reviewers/
      advisorClaude.ts
      gptReviewer.ts
      geminiReviewer.ts
      localHeuristics.ts
    policy/
      decisions.ts
      repoProfiles.ts
    worktrees/
      checkout.ts
      commands.ts
    prompts/
      advisor.md
      reviewer.md
      release.md
```

## Trigger Model

Start with polling. Webhooks can come later.

Commands:

```powershell
node .\dist\cli.js watch --interval 120
node .\dist\cli.js review --repo dominicthomas3/centurion-agent-public-releases --pr 123
```

The watcher should:

1. Query open PRs in all configured repos.
2. Select PRs with `agent-review:run`.
3. Skip draft PRs unless explicitly forced.
4. Capture the PR head SHA.
5. Build the review packet.
6. Run local heuristics and available model reviewers.
7. Ask the advisor to adjudicate.
8. Re-check the head SHA before posting.
9. Post or update a comment with marker `<!-- centurion-layer2-review -->`.
10. Update labels:
    - approve: add `agent-review:approved`, remove `agent-review:run`, `agent-review:pending`, `agent-review:changes-requested`, `agent-review:needs-human`
    - changes: add `agent-review:changes-requested`
    - human: add `agent-review:needs-human`

## Review Packet Contents

Collect:

- Repo, PR number, title, author, base branch, head branch, head SHA.
- PR body and review packet section.
- Changed files and file stats.
- Full diff when small enough.
- Targeted file contents around changed hunks.
- Existing CI status.
- Existing review comments.
- Risk labels from the router.
- Repo profile from `policy/repoProfiles.ts`.

Never paste secrets. Redact `.env`, tokens, keys, private certs, and signing material.

## Reviewer Output Contract

Each reviewer returns JSON:

```json
{
  "reviewer": "gpt|gemini|local|release",
  "decision": "approve|comment|changes_requested|needs_human",
  "grade": 0,
  "confidence": 0,
  "findings": [
    {
      "severity": "blocking|major|minor|nit",
      "file": "path/to/file",
      "line": 1,
      "title": "Short finding title",
      "evidence": "What proves the issue",
      "impact": "What breaks or what risk increases",
      "fix": "Concrete fix direction"
    }
  ],
  "tests": ["commands the reviewer expects or validates"],
  "notes": ["non-blocking observations"]
}
```

The advisor is the only role that posts the final decision.

## Public Release Review Rules

For `centurion-agent-public-releases`, block merge unless release provenance is clear:

- Source `centurion-agent` commit is identified.
- Artifact names and versions match release notes.
- Checksums/signatures are present when claimed.
- Download or latest-release claims match intended release behavior.
- License/terms statements are consistent with the product.

Do not approve binary or checksum changes without provenance.

## GitHub Operations

Prefer `gh` first because it matches Dominic's current workflow.

Useful commands:

```powershell
gh pr list --repo dominicthomas3/centurion-agent-public-releases --state open --label agent-review:run --json number,title,headRefName,headRefOid,isDraft,labels
gh pr view 123 --repo dominicthomas3/centurion-agent-public-releases --json title,body,author,baseRefName,headRefName,headRefOid,labels,statusCheckRollup
gh pr diff 123 --repo dominicthomas3/centurion-agent-public-releases
gh pr comment 123 --repo dominicthomas3/centurion-agent-public-releases --body-file review.md
gh pr edit 123 --repo dominicthomas3/centurion-agent-public-releases --add-label agent-review:approved --remove-label agent-review:run
```

If the GitHub CLI is not authenticated, stop and tell Dominic exactly what auth scope is missing.

## Definition Of Done

- `watch` and `review` commands work.
- Runner can review all three repos.
- Runner refuses to post if the PR head SHA changed mid-review.
- Runner posts one upserted Layer 2 comment.
- Runner updates labels correctly for approve, changes requested, and needs human.
- Runner includes a dry-run mode.
- README explains setup, `.env`, commands, and operating procedure.
- No source repo code is edited by the reviewer.

