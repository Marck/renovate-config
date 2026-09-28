# Renovate config

Shared Renovate preset for my repositories. Consume it with:

```json
{
  "extends": ["config:recommended", "github>marck/renovate-config"]
}
```

## What it sets

| Option | Value | Why |
| --- | --- | --- |
| `minimumReleaseAge` | `10 days` | Let a release soak before it reaches a cluster. |
| `minimumReleaseAgeBehaviour` | `timestamp-optional` | Renovate's docker datasource only returns a release timestamp for Docker Hub. On the default `timestamp-required`, every ghcr.io and quay.io image stays pending forever and never gets a branch or a PR. |
| `automergeType` + `ignoreTests` | `branch`, `true` | Minor, patch, pin and digest land without a PR. Override per repo for anything that reaches a cluster unreviewed. |
| `reviewers` | `["Marck"]` | Must be an array. A bare string is a config error and no reviewer is requested. |
| `dependencyDashboard` | `true` | One issue per repo listing everything held back, with checkboxes to force a branch now. |

## Validating a change

```sh
npx --yes --package renovate renovate-config-validator --strict default.json
```

CI runs the same command. Validate before pushing: an invalid preset surfaces as a
"Fix Renovate Configuration" issue in every consuming repo, not in this one.
