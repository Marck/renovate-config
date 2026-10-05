# Renovate config

Shared Renovate preset for my repositories. Consume it with:

```json
{
  "extends": ["local>Marck/renovate-config"]
}
```

The preset extends `config:recommended` itself, so a repo does not need to.

## What it sets

| Option | Value | Why |
| --- | --- | --- |
| `minimumReleaseAge` | `10 days` | Let a release soak before it reaches a cluster. |
| `minimumReleaseAgeBehaviour` | `timestamp-optional` | Renovate's docker datasource only returns a release timestamp for Docker Hub. On the default `timestamp-required`, every ghcr.io and quay.io image stays pending forever and never gets a branch or a PR. |
| `extends` | `config:recommended` | Every consuming repo gets Renovate's standard groups and semantic commits, including repos that only list this preset. |
| `automergeType` | `branch` | A green non-major merges straight to the base branch, so a PR only appears when a human is needed: a major, a red run, or checks still pending after 25 hours (`prNotPendingHours`). This was `branch` once before with `ignoreTests: true`, which merged without running anything; `ignoreTests` is now `false`. |
| `ignoreTests` | `false` | CI has to pass before an automerge. This was `true`, which made Renovate return green without even asking the platform. |
| `platformAutomerge` | `false` | Renovate does the merge itself after it has seen a green run. GitHub's native automerge would merge as soon as the PR was mergeable, which on a branch with no required checks is before CI has started. |
| `reviewers` | `["Marck"]` | Must be an array. A bare string is a config error and no reviewer is requested. |
| `assignAutomerge` | `true` | Renovate skips reviewers on PRs it means to automerge unless this is on, which is why automerged PRs arrived with nobody requested. |
| minor / patch / pin / digest | automerge | Gated on `minimumReleaseAge` plus a green run. |
| major | no automerge, label `major-review` | The review request is the notification. |
| `github-actions`, any update type | automerge, label `github-action` | A workflow action reaches no cluster and migrates no data. A bad one shows up as a red run on the PR that introduced it, which is what blocks the merge anyway, so a major here is not a decision worth queueing. This rule comes after the major rule above, so it wins. |
| major in `release*` / `publish*` workflows | no automerge, label `major-review` | Those run on main only, so branch CI never executes the changed step. |
| `github-runners` major | no automerge, label `major-review` | A runner image swaps the toolchain (Xcode included), it is not an action bump. |
| `dependencyDashboard` | `true` | One issue per repo listing everything held back, with checkboxes to force a branch now. |
| `prConcurrentLimit` / `prHourlyLimit` | `0` / `2` | No cap on open updates, so a manual PR left open cannot starve unrelated ones; creation stays paced at 2 an hour. |

## Validating a change

```sh
npx --yes --package renovate@latest renovate-config-validator --strict default.json
```

CI runs the same command. Validate before pushing: an invalid preset surfaces as a
"Fix Renovate Configuration" issue in every consuming repo, not in this one.

## CI has to run on renovate branches

Branch automerge waits for checks on the `renovate/...` branch itself, before
any PR exists. A workflow that only triggers on `pull_request`, or on
`push: branches: ["*"]` (`*` does not match a `/`), produces no check there, so
the update sits for 25 hours and then opens a PR anyway. Trigger on
`push: branches: ["renovate/**"]` (or `branches-ignore: [main]`).

## New repos

Renovate's onboarding looks for `Marck/renovate-config` itself and, when it
exists, proposes `"extends": ["local>Marck/renovate-config"]` in every
onboarding PR. Nothing to configure.

## Repos with no CI

`ignoreTests: false` means a branch has to go green before it automerges. A repo
with no workflow at all never produces a status check or a check run, which
resolves to yellow rather than green, so every update there would become a PR
after 25 hours.
Those repos set `ignoreTests: true` for themselves and say so in their own
`renovate.json`: today that is `docker` and `argocd-app-of-apps`. Remove the
override when the repo gains CI.
