# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) configuration presets for Virtual Coffee repositories.

Dependency-update policy lives here instead of being copy-pasted into every repo, so a change to the schedule, grouping, or noise level is one PR against this repo rather than ten.

## Usage

Add a `renovate.json` to the root of your repo:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>Virtual-Coffee/renovate-config"]
}
```

Then ask an org owner to add the repo to the [Renovate GitHub App](https://github.com/apps/renovate) installation.

## Presets

### `default` — the baseline

```json
{ "extends": ["github>Virtual-Coffee/renovate-config"] }
```

Updates npm dependencies, GitHub Actions, and `.nvmrc`/`engines.node`. Highlights:

- **Weekly**, Monday before 6am (America/New_York). Security fixes ignore the schedule and open immediately.
- **Major updates do not open PRs.** They're listed on the repo's Dependency Dashboard issue with a checkbox; tick it when you're ready to deal with the upgrade. This keeps breaking majors from rotting as permanently-open PRs.
- **Grouped** so the PR count stays low: one PR for non-major runtime dependencies, one for non-major devDependencies, one for all GitHub Actions, one for Node.
- **Monthly lockfile maintenance** to pick up transitive dependency fixes.
- **No automerge** — every PR needs a human.

### `automerge` — opt in if CI gates your PRs

```json
{
  "extends": [
    "github>Virtual-Coffee/renovate-config",
    "github>Virtual-Coffee/renovate-config:automerge"
  ]
}
```

Automerges non-major devDependency and GitHub Actions updates. Runtime dependency PRs are **not** automerged even with this preset — those still need a human. **Only use this in repos where required status checks actually run tests or a build on pull requests** — otherwise it merges unverified changes.

### `low-activity` — opt in for dormant or content-only repos

```json
{
  "extends": [
    "github>Virtual-Coffee/renovate-config",
    "github>Virtual-Coffee/renovate-config:low-activity"
  ]
}
```

Monthly instead of weekly, no lockfile maintenance, and npm updates are held behind Dependency Dashboard approval. Security fixes and GitHub Actions updates still come through normally.

## Contributing

Changes here affect every repo that extends these presets, so PRs are validated in CI with `renovate-config-validator`. To check locally:

```sh
npx --yes --package renovate -- renovate-config-validator --strict --no-global \
  default.json automerge.json low-activity.json
```
