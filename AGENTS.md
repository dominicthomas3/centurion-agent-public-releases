# Centurion Public Releases Instructions

Read this file before changing this repository. Then read `docs/AGENT_PROCESS.md`.

This repo is the public binary release mirror for Centurion AI desktop. It is not a source-code repo. Treat it as release infrastructure.

## Layer 1 Workflow

- Work on one branch per release or documentation workstream: `codex/<workstream>`, `claude/<workstream>`, or `agent/<workstream>`.
- Make as many commits/checkpoints as needed on that branch.
- Open or update one draft PR for the whole release workstream.
- Do not create multiple PRs for one release.
- Do not merge your own PR. The Layer 2 gate must approve the current PR head before merge.
- If Layer 2 requests changes, update the same branch and same PR.

## Review Packet Required In Every PR

Every PR must include:

- Release/version intent.
- Files changed.
- Binary, checksum, signing, or release-note impact.
- Source repo or build provenance.
- Commands run and verification results.
- Known gaps or manual steps remaining.

## High-Risk Areas

Treat these as high-risk and expect Layer 2 review:

- Any binary artifact, checksum, signature, installer, package, release metadata, or download URL.
- GitHub Actions, release automation, branch protection, or publishing scripts.
- Any statement about trust, signing, license, terms, supported platforms, or latest release.

Docs-only edits can be low risk only when they do not alter release trust, install instructions, or download behavior.

