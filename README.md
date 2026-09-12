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

Updates npm dependencies, GitHub Actions, and tooling version pins (`.nvmrc`, `.node-version`, `.tool-versions`, mise, `engines.node`). Highlights:

- **Weekly**, Monday before 6am (America/New_York). Security fixes ignore the schedule and open immediately.
- **Major updates do not open PRs.** They're listed on the repo's Dependency Dashboard issue with a checkbox; tick it when you're ready to deal with the upgrade. This keeps breaking majors from rotting as permanently-open PRs.
- **Grouped** so the PR count stays low: one PR for all non-major npm bumps (runtime and dev together), one for all GitHub Actions, and one for tooling. The tooling PR moves Node and the package manager (pnpm, yarn, bun or npm) in lockstep across every file that pins them (`.nvmrc`, `.node-version`, `.tool-versions`, mise, `engines.node`, `packageManager`), and `@types/node` major and minor bumps ride along with it; `@types/node` patches stay in the npm PR.
- **Monthly lockfile maintenance** to pick up transitive dependency fixes.
- **npm updates wait three days** after publish before a PR is raised (`security:minimumReleaseAgeNpm`). This clears pnpm 11's own one-day `minimumReleaseAge` gate — pnpm refuses to install a version younger than that, so a PR for a fresh release fails the build — and gives scanners a window to catch a compromised package. Security fixes are exempt and still open immediately.
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

Automerges non-major devDependency and GitHub Actions updates. Runtime dependencies are never automerge-eligible, and Renovate only automerges a PR when every update in it qualifies — so the grouped npm PR automerges only on weeks where it contains nothing but devDependencies, and the tooling PR is not automerged whenever a Node or pnpm pin is in it. GitHub Actions PRs automerge every time. **Only use this in repos where required status checks actually run tests or a build on pull requests** — otherwise it merges unverified changes.

### `low-activity` — opt in for dormant or content-only repos

```json
{
  "extends": [
    "github>Virtual-Coffee/renovate-config",
    "github>Virtual-Coffee/renovate-config:low-activity"
  ]
}
```

Monthly instead of weekly, no lockfile maintenance, and npm plus tooling-version updates are held behind Dependency Dashboard approval. Security fixes and GitHub Actions updates still come through normally.

## Contributing

Changes here affect every repo that extends these presets, so PRs are validated in CI with `renovate-config-validator`. To check locally:

```sh
npx --yes --package renovate -- renovate-config-validator --strict --no-global \
  default.json automerge.json low-activity.json
```
