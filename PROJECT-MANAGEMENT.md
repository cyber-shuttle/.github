# Cybershuttle Project Management

This document describes the project management infrastructure across the cyber-shuttle GitHub organization.

## GitHub Project Board

All issues and PRs across the organization are tracked in a single [GitHub Projects V2 board](https://github.com/orgs/cyber-shuttle/projects/2).

### Connected Repositories

- [CS-Bridge](https://github.com/cyber-shuttle/CS-Bridge) — VS Code extension
- [linkspan](https://github.com/cyber-shuttle/linkspan) — Tunnel/mount agent binary
- [frp-server](https://github.com/cyber-shuttle/frp-server) — FRP relay server
- [vscode-ux-metrics](https://github.com/cyber-shuttle/vscode-ux-metrics) — UX metrics collection
- [cs-bridge-admin](https://github.com/cyber-shuttle/cs-bridge-admin) — Admin server

### Project Fields

| Field | Type | Options |
|-------|------|---------|
| Status | Single select | Backlog, Todo, In Progress, In Review, Blocked, Done |
| Priority | Single select | P0-Critical, P1-High, P2-Medium, P3-Low |
| Size | Single select | XS, S, M, L, XL |
| Component | Single select | extension, linkspan, frp, metrics, admin, api |
| Sprint | Iteration | 2-week sprints (configure duration/start in [project settings](https://github.com/orgs/cyber-shuttle/projects/2/settings)) |

> **Note:** The Repository column is built-in to GitHub Projects. Enable it in the project board view settings (web UI).

## Labels

A shared label scheme is applied across all connected repositories:

### Type Labels
| Label | Description |
|-------|-------------|
| `type:bug` | Bug reports |
| `type:feature` | New feature requests |
| `type:enhancement` | Improvements to existing features |
| `type:docs` | Documentation changes |
| `type:chore` | Maintenance, dependencies, CI |
| `type:refactor` | Code restructuring without behavior change |

### Priority Labels
| Label | Description |
|-------|-------------|
| `priority:critical` | Must fix immediately |
| `priority:high` | Important, address soon |
| `priority:medium` | Normal priority |
| `priority:low` | Nice to have |

### Component Labels
| Label | Description |
|-------|-------------|
| `component:tunnel` | Tunnel/networking functionality |
| `component:storage` | File storage and sync |
| `component:auth` | Authentication and identity |
| `component:ui` | User interface and UX |

### Status Labels
| Label | Description |
|-------|-------------|
| `status:blocked` | Blocked by external dependency |
| `status:needs-info` | Needs more information |
| `status:wontfix` | Will not be addressed |
| `good first issue` | Good for newcomers |

## Automation Workflows

Each connected repository has four GitHub Actions workflows that keep the project board in sync automatically.

### 1. Auto-Add to Project (`auto-add-to-project.yml`)

**Trigger:** Issue opened/transferred, PR opened

Automatically adds new issues and PRs to the project board.

```yaml
on:
  issues:
    types: [opened, transferred]
  pull_request_target:
    types: [opened]
```

Uses [`actions/add-to-project@v1.0.2`](https://github.com/actions/add-to-project).

### 2. Issue Status Sync (`issue-status-sync.yml`)

**Trigger:** Issue closed or reopened

| Event | Project Status |
|-------|---------------|
| Issue closed | Done |
| Issue reopened | Backlog |

### 3. PR Status Sync (`project-status-sync.yml`)

**Trigger:** PR opened, reopened, closed, ready for review, converted to draft

| Event | Project Status |
|-------|---------------|
| PR opened (ready) | In Review |
| PR converted to draft | In Progress |
| PR marked ready for review | In Review |
| PR merged | Done |
| PR closed without merge | No change |

### 4. PR Merge Annotation (`pr-merge-annotate.yml`)

**Trigger:** PR merged

When a PR is merged, this workflow finds all linked issues (via `Closes #N`, `Fixes #N`, `Resolves #N` keywords in the PR body) and posts a comment on each linked issue with:
- Which PR resolved it
- Who authored the PR
- A summary of the changes

## Issue-PR Linking

Use GitHub's closing keywords in PR descriptions to link issues:

```
Closes #12
Fixes #15
Resolves #8
```

When a PR with these keywords is merged, GitHub automatically closes the linked issues. The `pr-merge-annotate` workflow then adds a comment to each closed issue with context about the resolution.

## State Transition Diagram

```
Issues:
  New Issue → [auto-add] → Backlog
  Backlog → (manual) → Todo → In Progress → In Review → Done
  Done → (reopen) → Backlog
  Any → (manual) → Blocked

Pull Requests:
  New PR (ready) → [auto-add] → In Review
  New PR (draft) → [auto-add] → In Progress
  In Progress → (ready for review) → In Review
  In Review → (convert to draft) → In Progress
  In Review → (merge) → Done
  In Review → (close without merge) → no change
```

## Setup Requirements

### PAT Token

The workflows require an org-level Personal Access Token stored as `ADD_TO_PROJECT_PAT` in each repository's secrets. The token needs these scopes:
- `repo` — full repo access
- `read:org` — read org membership
- `project` — manage projects
- `workflow` — update workflow files

### Manual Configuration (Web UI)

Some settings can only be configured through the GitHub web interface:

1. **Sprint iteration duration and start day** — Set in [project settings](https://github.com/orgs/cyber-shuttle/projects/2/settings) under the Sprint field configuration
2. **Built-in project workflows** — Enable "Item closed → Done" and "PR merged → Done" in the project's Workflows tab
3. **Repository column** — Enable in the project board view settings to see which repo each item belongs to
4. **Board view grouping** — Configure Status-based columns and swimlanes in the board view settings
