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
- **A seven-day cooldown** on major, minor, and patch releases from npm, Go, and GitHub, so a malicious release has time to be found and pulled. Only those carry a release timestamp, so other datasources and digest updates have no cooldown, and a repository that wants one sets it. Vulnerability fixes skip it.
- **Never `ignoreTests` or `internalChecksAsSuccess`.** Either one lets a repository merge with nothing having tested it.

## Adding an exception

An exception belongs in the repository it applies to, with a comment saying why. It is justified by a fact a check cannot cover: a tool that only runs in a job pull requests never reach, a change that is a judgment, a planned migration, or a merge that would not stick. "It is a major version" is not one of these. That is what tests are for.
