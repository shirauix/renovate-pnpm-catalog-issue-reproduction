# Catalog version not applying to all packages in pnpm workspace when shared-workspace-lockfile = false #36508

A repository that reproduces a Renovate bug where, under a pnpm workspace environment with shared-workspace-lockfile = false, the catalog version is not applied to all packages.

Link to the discussion:

https://github.com/renovatebot/renovate/discussions/36508


## File structure

```
.
├── packages
│   ├── no-catalog
│   │   ├── package.json  // uses husky without catalog
│   │   └── pnpm-lock.yaml
│   └── use-catalog
│       ├── package.json  // uses zod with catalog
│       └── pnpm-lock.yaml
├── package.json  // uses zod with catalog
├── pnpm-lock.yaml
├── pnpm-workspace.yaml  // zod version defined via catalog
├── README.md
└── renovate.json
```

## Self-hosted Runner Configuration

GitHub Actions workflow:

```yaml
      - name: Self-hosted Renovate
        uses: renovatebot/github-action@v42.0.5
        with:
          configurationFile: '.github/renovate.json'
          renovate-version: 40.50.0
          token: ${{ steps.app-token.outputs.token }}
        env:
          LOG_LEVEL: 'debug'
```

Config:

```renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:best-practices",
    "mergeConfidence:all-badges"
  ],
  "branchPrefix": "renovate/",
  "labels": [
    "renovate"
  ],
  "platform": "github",
  "prCreation": "not-pending",
  "repositories": [
    "<Org>/<Repo>"
  ],
  "username": "renovate-bot[bot]",
  "allowedCommands": [
    "^pnpm install.*$"
  ]
}
```

## Current Behavior

Renovate generates the following PRs:

### chore(deps): update dependency zod to v3.25.64

This includes updates to the following files:

- `pnpm-lock.yaml`
- `pnpm-workspace.yaml`

Since the `use-catalog` package also references `zod` using `catalog:`, the following file should also be updated, but it is not:

- `packages/use-catalog/pnpm-lock.yaml`

### chore(deps): update dependency husky to v9.0.11

This includes updates to the following files:

- `packages/no-catalog/package.json`
- `packages/no-catalog/pnpm-lock.yaml`

This behavior is as expected.

## Expected Behavior

As described above, in the PR that updates `zod`, the `pnpm-lock.yaml` inside the `use-catalog` package should also be updated.

# Workaround

In my environment, the updates were performed correctly after adding the following configuration to the repository’s `renovate.json`.

This is likely because `pnpm install` recursively updates all packages in the repository by default.

```renovate.json
  "postUpgradeTasks":{
    "commands": ["pnpm install --lockfile-only"],
    "executionMode": "branch",
    "fileFilters": [
      "pnpm-workspace.yaml",
      "**/package.json",
      "**/pnpm-lock.yaml"
    ]
  }
```
