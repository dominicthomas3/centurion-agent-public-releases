# Centurion Agent — Public Releases

This repository is **artifact-only**: it hosts signed Windows x64 MSI installers (Azure Trusted Signing) for the Centurion AI desktop application, attached to GitHub Releases.

## What's NOT here

- No source code
- No build scripts
- No tests

Source lives in the private `dominicthomas3/centurion-agent` repo. Builds happen there; signed artifacts get mirrored here.

## Why this repo exists

The desktop application's auto-updater and onboarding docs need a stable public URL for signed MSIs. This repo is that URL: `https://github.com/dominicthomas3/centurion-agent-public-releases/releases/latest` resolves cleanly without exposing source.

## Release flow

1. Tag `v*` in `centurion-agent` triggers its `release.yml`
2. That workflow signs the MSI via Azure Trusted Signing, builds the Velopack delta, uploads to S3, and creates a GitHub Release in `centurion-agent`
3. **Phase 2b TODO:** mirror that release into this repo so `releases/latest` here always resolves to the latest signed MSI

## Automation

- `.github/workflows/claude.yml` — `@claude` mentions in issues/PRs spawn a cloud Claude session (mostly for release-process triage; there's no code to edit here)
