---
date: 2026-05-02
tags: [birge, git, github, workflow]
status: active
---

# Git Commit and Push Rules

> [!NOTE]
> These rules apply to the BIRGE repositories, especially `birge-ios`. The goal is to keep `main` stable, make PRs reviewable, and avoid mixing unrelated work.

## Core Rules

1. Never push directly to `main`.
2. Always create a branch from the latest `main`.
3. Keep each branch focused on one type of change.
4. Commit only the files that belong to that change.
5. Use conventional commit messages.
6. Run the relevant build/test/check before pushing.
7. Push the branch and open a PR into `main`.
8. Do not merge your own PR until CI passes or the failure is understood.

## Branch Naming

Use clear prefixes:

| Change type | Branch format | Example |
|---|---|---|
| Feature | `feature/<short-name>` | `feature/ride-request-api` |
| Bug fix | `fix/<short-name>` | `fix/driver-target-name` |
| Docs | `docs/<short-name>` | `docs/readme-update` |
| CI/CD | `ci/<short-name>` or `feature/ci-cd` | `feature/ci-cd` |
| Refactor | `refactor/<short-name>` | `refactor/api-client-live` |
| Tests | `test/<short-name>` | `test/otp-flow` |

## Commit Message Format

Use:

```text
type(scope): description
```

Examples:

```text
ci: add iOS and Vapor GitHub Actions workflows
docs: add badges, getting started, and roadmap to README
fix(ios): rename driver target to BIRGEDriver
feat(passenger): wire ride request API
test(auth): add OTP retry coverage
refactor(core): move API client dependencies into BIRGECore
```

Common types:

| Type | Use for |
|---|---|
| `feat` | New user-facing behavior |
| `fix` | Bug fixes |
| `docs` | README, Obsidian, docs |
| `ci` | GitHub Actions, build pipelines |
| `test` | Test additions or test-only fixes |
| `refactor` | Internal restructuring without behavior change |
| `chore` | Maintenance tasks |

## Standard Workflow

```sh
git switch main
git pull --ff-only origin main
git switch -c <branch-name>
```

Make changes, then inspect carefully:

```sh
git status -sb
git diff --check
git diff
```

Stage only related files:

```sh
git add <file-1> <file-2>
```

Commit:

```sh
git commit -m "type(scope): description"
```

Push:

```sh
git push -u origin <branch-name>
```

Open a PR into `main`.

## Verification Before Push

For iOS Passenger:

```sh
cd /Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger
xcodebuild test \
  -project BIRGEPassenger.xcodeproj \
  -scheme BIRGEPassenger \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro'
```

For iOS Driver:

```sh
cd /Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger
xcodebuild build \
  -project BIRGEPassenger.xcodeproj \
  -scheme BIRGEDriver \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro'
```

For Vapor:

```sh
cd /Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/birge-vapor
swift build
swift test
```

## PR Rules

Every PR should include:

- What changed
- Why it changed
- How it was verified
- Any known blockers or follow-ups

Keep PRs small:

- CI workflow files go in a CI PR.
- README/docs go in a docs PR.
- Xcode target renames go in a fix/refactor PR.
- API wiring goes in a feature PR.

Do not combine CI + README + app code + backend changes in one PR unless they are inseparable.

## Special Rule: GitHub Actions Workflow Files

> [!WARNING]
> Pushing `.github/workflows/*` requires a GitHub token with `workflow` scope. If the token lacks that scope, GitHub rejects the push even if normal code pushes work.

If pushing CI changes fails with:

```text
refusing to allow a Personal Access Token to create or update workflow
```

then regenerate/update the GitHub token with `workflow` scope, update the remote credentials, and retry:

```sh
git push -u origin feature/ci-cd
```

## Current Known Branches

| Branch | Purpose | Status |
|---|---|---|
| `docs/readme-update` | README badges, setup, architecture, roadmap | PR opened |
| `fix/driver-target-name` | Rename `BIRGEDrive` to `BIRGEDriver` | PR opened |
| `feature/ci-cd` | iOS + Vapor GitHub Actions workflows | Local commit exists; push blocked by missing `workflow` token scope |

## Obsidian Links

- [[GitHub_CI_Readme_2026-05-02]]
- [[UI_Design_System_2026-05-02]]
- [[iOS_Agent_Context]]
- [[iOS_Sprint_1]]

## Tags

#birge #git #github #workflow #commits #pull-requests
