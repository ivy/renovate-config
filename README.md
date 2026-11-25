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
- **A three-day cooldown** on major, minor, and patch releases from npm, Go, GitHub, and Node.js, so a malicious release has time to be found and pulled. Most compromised releases are pulled within that time, and a longer wait buys little ([evidence](docs/research/dependency-cooldowns.md)). A datasource joins the rule in `default.json` only if it carries a release timestamp, since one without it would wait for ever; digest updates have no cooldown. Vulnerability fixes skip it.
- **Actions locked by commit.** Workflows pin actions to a commit, either inline as a SHA or through `.github/workflows/actions.lock`, which Renovate regenerates with each update. A custom manager keeps the `gh extension install github/gh-actions-lock --pin` release in workflows current; a repository that sets `enabledManagers` lists `custom.regex` for it.
- **Never `ignoreTests` or `internalChecksAsSuccess`.** Either one lets a repository merge with nothing having tested it.

## Checks

`hk check --all` validates `default.json` and this repository's `renovate.json5` with `renovate-config-validator --strict`, which fails on an unknown or deprecated option, and lints the workflows with [jactionlint](https://github.com/jdx/jactionlint). The Check workflow runs it on every push and pull request, and the actions-lock workflow runs `gh actions-lock --verify` to fail when `.github/workflows/actions.lock` no longer matches the workflows. Renovate keeps the validator in `mise.toml` current, so a release that changes the config schema fails here before it reaches the repositories that extend this preset. `default.json` is JSONC: Renovate allows comments in it, and it must keep that name, because it is the only file `github>ivy/renovate-config` loads.

## Adding an exception

An exception belongs in the repository it applies to, beside the setting, with a comment naming the fact behind it and what would lift it. It is justified by a fact outside the repository that no check can observe: a service that accepts only certain versions of a client, or a forge copy that another system overwrites, so a merge there would not stick.

A major version, a tool that pull requests never run, a change that takes judgment, and a move to a new operating-system release are not exceptions. Each one is a gap in the repository's checks, and the answer is a check that covers it, not a person who reviews the update.
