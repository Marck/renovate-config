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
| `automergeType` | `pr` | Every update gets a PR, so every update has a reviewer and a record. This was `branch`, which merged to the base branch with no PR at all. |
| `ignoreTests` | `false` | CI has to pass before an automerge. This was `true`, which made Renovate return green without even asking the platform. |
| `platformAutomerge` | `false` | Renovate does the merge itself after it has seen a green run. GitHub's native automerge would merge as soon as the PR was mergeable, which on a branch with no required checks is before CI has started. |
| `reviewers` | `["Marck"]` | Must be an array. A bare string is a config error and no reviewer is requested. |
| `assignAutomerge` | `true` | Renovate skips reviewers on PRs it means to automerge unless this is on, which is why automerged PRs arrived with nobody requested. |
| minor / patch / pin / digest | automerge | Gated on `minimumReleaseAge` plus a green run. |
| major | no automerge, label `major-review` | The review request is the notification. |
| `dependencyDashboard` | `true` | One issue per repo listing everything held back, with checkboxes to force a branch now. |

## Validating a change

```sh
npx --yes --package renovate renovate-config-validator --strict default.json
```

CI runs the same command. Validate before pushing: an invalid preset surfaces as a
"Fix Renovate Configuration" issue in every consuming repo, not in this one.

## Repos with no CI

`ignoreTests: false` means a branch has to go green before it automerges. A repo
with no workflow at all never produces a status check or a check run, which
resolves to yellow rather than green, so nothing would ever automerge there.
Those repos set `ignoreTests: true` for themselves and say so in their own
`renovate.json`: today that is `docker` and `argocd-app-of-apps`. Remove the
override when the repo gains CI.
