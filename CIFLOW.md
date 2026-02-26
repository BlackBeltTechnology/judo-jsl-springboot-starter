# CI/CD Flow — Development Versioning and Branch Handling

This document describes the GitFlow-based branching strategy, version numbering policies, and GitHub Actions workflows used by this project.

## Branches

The branching model follows [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) with these branch types:

| Branch | Purpose | Lifetime |
|--------|---------|----------|
| `develop` | Main integration branch with latest development sources | Permanent |
| `master` | Latest released sources of the active version | Permanent |
| `feature/JNG-xxx_summary` | New features, branched from `develop` | Temporary |
| `release/x.y-betaN` | Stabilization before a release, branched from `develop` | Temporary |
| `bugfix/JNG-xxx_summary` | Fixes applied to release branches during testing | Temporary |
| `support/JNG-xxx_summary` | Minor changes to a previous release, branched from release | Temporary |
| `hotfix/JNG-xxx_summary` | Urgent fixes applied to both release and `master` | Temporary |

### Branch Flow

The following diagram shows how branches interact over a typical development cycle. Features merge into `develop`, release branches stabilize the code, and releases merge into `master`.

```mermaid
gitGraph
    commit id: "initial"
    branch develop order: 1
    checkout develop
    commit id: "dev-start"

    branch feature/JNG-1 order: 2
    commit id: "feat-1a"
    commit id: "feat-1b"
    checkout develop
    merge feature/JNG-1 id: "merge-feat-1"

    branch feature/JNG-2 order: 3
    commit id: "feat-2a"
    checkout develop
    merge feature/JNG-2 id: "merge-feat-2"

    branch release/1.0-beta1 order: 4
    commit id: "stabilize"

    branch bugfix/JNG-4 order: 5
    commit id: "bugfix"
    checkout release/1.0-beta1
    merge bugfix/JNG-4 id: "merge-bugfix"

    checkout develop
    merge release/1.0-beta1 id: "merge-release-to-dev"

    checkout main
    merge release/1.0-beta1 id: "release-1.0" tag: "v1.0.0"
```

## Version Numbers

Version numbers follow semantic versioning with these rules:

| Event | Version Change | Example |
|-------|---------------|---------|
| Start a feature branch | No change | stays `1.1.0-SNAPSHOT` |
| Start a release branch | Increment 2nd number on `develop` | `develop` goes from `1.1.0` to `1.2.0` |
| Bugfix on release branch | No change | stays at release version |
| Start a support branch | Increment 3rd number | `1.0.0` becomes `1.0.1` |
| Start a hotfix branch | Increment 4th number | `1.0.0` becomes `1.0.0.1` |

### Development vs. Release Versions

On `develop` and `increment/*` branches, CI produces dynamic snapshot versions:
```
major.minor.qualifier.YYYYMMDD_HHMMSS_commitId_branchName
```

On `master` and `release/*` branches, versions are taken directly from `pom.xml` (without `-SNAPSHOT`).

## GitHub Actions Workflows

The project uses 13 GitHub Actions workflows. The four core workflows form a connected pipeline:

### Core Pipeline

```mermaid
flowchart TD
    subgraph "Triggers"
        PUSH_DEV["Push to develop"]
        PR["PR on develop / master /<br/>increment/* / release/*"]
        MANUAL["Manual trigger<br/><i>release.yml</i>"]
        PUSH_MASTER["Push to master"]
        TAG["Push merge-pr/* tag"]
    end

    subgraph "build.yml"
        B_VERSION["Calculate version<br/><i>dynamic or from pom.xml</i>"]
        B_BUILD["Build & deploy to Nexus"]
        B_TAG["Create git tag v&lt;version&gt;"]
        B_MERGE_TAG["Create merge-pr/&lt;version&gt; tag"]
        B_RELEASE["Create GitHub prerelease<br/>with changelog"]
    end

    subgraph "merge-pr-tagged.yml"
        M_CHECK{"Version format?"}
        M_MASTER["Merge PR to master"]
        M_DEVELOP["Squash PR to develop"]
    end

    subgraph "create-release-on-master.yml"
        C_LOG["Build changelog"]
        C_RELEASE["Create GitHub release<br/><i>final</i>"]
    end

    subgraph "release.yml"
        R_VERSION["Calculate release &<br/>next version"]
        R_PR_MASTER["Create PR on master<br/>with release version"]
        R_PR_DEVELOP["Create PR on develop<br/>with next version"]
    end

    PUSH_DEV --> B_VERSION
    PR --> B_VERSION
    B_VERSION --> B_BUILD --> B_TAG
    B_TAG -->|"increment/*, release/*"| B_MERGE_TAG
    B_TAG -->|"develop"| B_RELEASE
    B_MERGE_TAG --> TAG

    TAG --> M_CHECK
    M_CHECK -->|"major.minor.qualifier"| M_MASTER
    M_CHECK -->|"other"| M_DEVELOP
    M_MASTER --> PUSH_MASTER
    M_DEVELOP --> PUSH_DEV

    PUSH_MASTER --> C_LOG --> C_RELEASE

    MANUAL --> R_VERSION --> R_PR_MASTER
    R_VERSION --> R_PR_DEVELOP
    R_PR_MASTER --> PR
    R_PR_DEVELOP --> PR
```

### Workflow Details

#### build.yml — Main Build Pipeline

- **Triggers:** Push to `develop`, PRs on `develop`/`master`/`increment/*`/`release/*`
- **Runner:** `judong` (custom), 30-minute timeout
- **JDK:** Zulu 21
- **Steps:**
  1. Calculate version (dynamic for develop, from pom.xml for releases)
  2. Build with `mvn deploy -Psign-artifacts -Prelease-judong`
  3. Create git tag `v<version>`
  4. For `increment/*`/`release/*`: create `merge-pr/<version>` tag (triggers merge workflow)
  5. For `develop`: build changelog and create GitHub prerelease

#### release.yml — Manual Release Trigger

- **Trigger:** Manual dispatch with optional version input (`auto` uses pom.xml version)
- **Steps:**
  1. Determine release version and calculate next version (qualifier + 1)
  2. Create PR targeting `master` with release version
  3. Create PR targeting `develop` with next version
  4. Both PRs trigger `build.yml`

#### merge-pr-tagged.yml — Automated PR Merge

- **Trigger:** Push of `merge-pr/*` tags
- **Steps:**
  1. Extract version from tag name
  2. If version is `major.minor.qualifier` format: merge PR to `master` (triggers release on master)
  3. Otherwise: squash PR to `develop` (triggers build)
  4. Delete the `merge-pr/*` tag

#### create-release-on-master.yml — Final Release

- **Trigger:** Push to `master`
- **Steps:**
  1. Build changelog from commit history
  2. Create final GitHub release (non-prerelease)

### Supporting Workflows

| Workflow | Purpose |
|----------|---------|
| `bump-version.yml` | Manually increment version in pom.xml, creates a PR |
| `delete-old-draft-releases.yml` | Cleans up old draft GitHub releases |
| `build-dependabot.yml` | Skips builds for Dependabot PRs |
| `jira-description-to-pr.yml` | Copies JIRA ticket description into PR body |
| `sync-labels.yml` | Syncs GitHub labels from `.github/labels.yml` |

## Development Rules

> **Important:** There is no commit without a ticket number. Every commit and PR must include a `JNG-xxx` JIRA reference.

Issue tracking is managed in [JIRA](https://blackbelt.atlassian.net/jira/dashboards).
