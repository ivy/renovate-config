# renovate-config

The shared Renovate defaults for `ivy/*`. A repository uses them with one line, and adds only what is true of it:

```json5
{
  "extends": ["github>ivy/renovate-config"],
  "enabledManagers": ["npm", "github-actions"]
}
```

## The rule

If a repository's checks pass, the update merges, whatever kind of update it is. A repository with no checks does not merge, because that is a gap in its workflows and not a reason for someone to review by hand. [Almanac's principle 12](https://github.com/ivy/almanac/blob/main/docs/responsibilities.md) explains the reasoning.

## What the defaults do

- **Automerge everything.** Renovate merges once every check is green.
- **`platformAutomerge: false`.** No repository has required checks, so GitHub's auto-merge would merge before CI reports. Renovate merging keeps `almanac/trust` able to hold a pull request.
- **A three-day cooldown** on major, minor, and patch releases from npm, Go, and GitHub, so a malicious release has time to be found and pulled. Most compromised releases are pulled within that time, and a longer wait buys little ([evidence](docs/research/dependency-cooldowns.md)). Only those carry a release timestamp, so other datasources and digest updates have no cooldown, and a repository that wants one sets it. Vulnerability fixes skip it.
- **Actions locked by commit.** Workflows pin actions to a commit, either inline as a SHA or through `.github/workflows/actions.lock`, which Renovate regenerates with each update. A custom manager keeps the `gh extension install github/gh-actions-lock --pin` release in workflows current; a repository that sets `enabledManagers` lists `custom.regex` for it.
- **Never `ignoreTests` or `internalChecksAsSuccess`.** Either one lets a repository merge with nothing having tested it.

## Checks

`hk check --all` validates `default.json` and this repository's `renovate.json5` with `renovate-config-validator --strict`, which fails on an unknown or deprecated option, and lints the workflows with [jactionlint](https://github.com/jdx/jactionlint). The Check workflow runs it on every push and pull request. Renovate keeps the validator in `mise.toml` current, so a release that changes the config schema fails here before it reaches the repositories that extend this preset. `default.json` is JSONC: Renovate allows comments in it, and it must keep that name, because it is the only file `github>ivy/renovate-config` loads.

## Adding an exception

An exception belongs in the repository it applies to, with a comment saying why. It is justified by a fact a check cannot cover: a tool that only runs in a job pull requests never reach, a change that is a judgment, a planned migration, or a merge that would not stick. "It is a major version" is not one of these. That is what tests are for.
