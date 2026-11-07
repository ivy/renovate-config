# How long a dependency cooldown needs to be

A cooldown (Renovate's `minimumReleaseAge`) holds a new release back for a fixed time after it is published, so that a malicious release can be found and pulled before we adopt it. This document collects the evidence for how long that time should be. It covers compromised releases of established packages, the case a cooldown defends against; typosquats and brand-new malicious packages are a different population and are kept apart.

Researched 2026-11-07 from public sources, without reference to any `ivy/*` configuration, and every cited source re-read against the claim it supports the same day. Each claim is labelled:

- **[M]** measured across many packages
- **[A]** a single incident, even where its timestamps are primary
- **[O]** opinion or recommendation

## Conclusion

The evidence supports a cooldown of about **3 days**, with 7 days as the conservative end.

- About 1 day covers most incidents since 2024 (13 of 19): a stolen credential or a hijacked CI pipeline publishes to a popular package, and the release is pulled within 1–5 hours.
- About 3–4 days adds the incidents where removal was slow: tj-actions, Gluestack, eslint-config-prettier, Trivy's Docker Hub images, and strong_password.
- 7–14 days adds three cases that ran 5–10 days: rest-client, ctx, and Kong's container image.
- Every remaining case lasted about a month or more, beyond any practical cooldown: PEAR (~1 month), xz (~33 days), event-stream (~52 days), Codecov (~60 days), Webmin (~15 months). Building from source, provenance, and review address those; waiting does not.

Confidence is moderate that most compromises are pulled within 1–3 days today, and low on any precise percentage. No peer-reviewed study measures publish-to-removal time for compromised established packages. The best quantitative source is an unreviewed engineering ADR, and the incident list below is hand-collected, so it leans toward famous, detected cases.

A cooldown is only worth this much when:

- security fixes are exempt from it, because exploitation can follow disclosure within days ([costs](#costs-and-counter-cases));
- the references it gates are immutable, because several attacks moved an existing tag and so arrived looking old;
- it covers transitive dependencies, which a pull-request cooldown alone does not.

## Incidents

### Compromised releases of established packages

The window runs from publish until the version was removed, quarantined, or deprecated, unless the row says otherwise. Every row is [A]. "Not stated" in the Attack column means the source names no method.

| Incident | Ecosystem | Attack | Publish → detect → remove | Window | Sources |
|---|---|---|---|---|---|
| eslint-scope 3.7.2 (2018) | npm | Reused password from another breach, then a new token | 2018‑07‑12 10:40 UTC → issue filed 11:17 → unpublished 12:37 | ~2 h | [ESLint postmortem](https://eslint.org/blog/2018/07/postmortem-for-malicious-package-publishes/) |
| event-stream / flatmap-stream 0.1.1 | npm | Maintainer handoff, malicious transitive dependency | 3.3.6 added the dependency 2018‑09‑09; malicious 0.1.1 published about 2018‑10‑05 (secondary sources) → reported 11‑20 → removed 11‑26 | ~52 days (~78 from 3.3.6) | [npm blog](https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident), [Snyk](https://snyk.io/blog/malicious-code-found-in-npm-package-event-stream/), [The Register](https://www.theregister.com/2018/11/26/npm_repo_bitcoin_stealer/) |
| PEAR go-pear.phar (2018–19) | PHP distribution site | Server compromise | Planted "after the last official file release on 20 December 2018" → announced 2019‑01‑19 (the first notice said "in the past six months") | ≤ ~1 month | [Security Affairs](https://securityaffairs.com/80218/hacking/php-pear-site-hacked.html), [The Hacker News](https://thehackernews.com/2019/01/php-pear-hacked.html) |
| Webmin 1.890+ | Upstream tarballs via SourceForge | Build-server compromise | Inserted April 2018 and again July 2018 → disclosed 2019‑08‑10 → fixed 08‑17 | ~16 months | [Duo (archived)](https://web.archive.org/web/2019/https://duo.com/decipher/backdoor-found-in-webmin-utility), [The Hacker News](https://thehackernews.com/2019/08/webmin-vulnerability-hacking.html) |
| bootstrap-sass 3.2.0.3 | RubyGems | Not stated; maintainers rotated their credentials | 2019‑03‑26 → issue 22:59 GMT → yanked 23:56 GMT | < 1 day | [Snyk](https://snyk.io/blog/malicious-remote-code-execution-backdoor-discovered-in-the-popular-bootstrap-sass-ruby-gem/) |
| strong_password 0.0.7 | RubyGems | Account hijack | 2019‑06‑25 → reported 06‑28 20:00 UTC → yanked "quickly" by RubyGems | ~3–4 days | [withatwist.dev](https://withatwist.dev/strong-password-rubygem-hijacked.html) |
| rest-client 1.6.10–1.6.13 | RubyGems | Account hijack | 2019‑08‑13/14 → reported and yanked 08‑19 | ~5–6 days | [rest-client#713](https://github.com/rest-client/rest-client/issues/713) |
| Codecov Bash Uploader | Vendor script (`curl \| bash`) | Infrastructure compromise | Altered 2021‑01‑31 → found by checksum mismatch and fixed 04‑01 | ~60 days | [Codecov](https://about.codecov.io/security-update/) |
| ua-parser-js | npm | Account hijack | 2021‑10‑22, live 14:15–18:23 CET | ~4 h | [Kaspersky](https://www.kaspersky.com/blog/uaparser-js-infected-versions/42700/), [GHSA-pjwm-rvh2-c87w](https://github.com/advisories/GHSA-pjwm-rvh2-c87w) |
| coa | npm | Not stated; the maintainer recovered access | 2021‑11‑04, live 14:12–15:24 CET | ~1.2 h | [coa#99](https://github.com/veged/coa/issues/99) |
| ctx | PyPI | Expired-domain email takeover; old version 0.1.2 replaced | From 2022‑05‑14 19:18 UTC → taken down around 05‑24 | ~10 days | [The Register](https://www.theregister.com/2022/05/24/pypi_ctx_package_compromised/), [BleepingComputer](https://www.bleepingcomputer.com/news/security/popular-pypi-and-php-libraries-hijacked-to-steal-aws-keys/) |
| Ledger Connect Kit 1.1.5–1.1.7 | npm | Phished former employee's account | 2023‑12‑14 09:49 CET → Ledger alerted 14:18 CET → fixed within 40 min | ~5 h | [Ledger](https://ledger.com/blog/security-incident-report) |
| xz-utils 5.6.0 / 5.6.1 | Upstream tarball, then Debian, Fedora, Arch | Long-term maintainer infiltration | 5.6.0 released 2024‑02‑24, in Debian unstable by 02‑26 → found 03‑28 → Debian rollback 03‑28 | ~33 days (5.6.1: ~19) | [research.swtch.com](https://research.swtch.com/xz-timeline) |
| Ultralytics 8.3.41/.42/.45/.46 | PyPI, via GitHub Actions cache poisoning, then a stolen token | CI hijack | .41: 12‑04 20:51 → reported 12‑05 06:34 → removed 09:15; .42 ~1 h; .45/.46 ~7.5–8 h | 1–12 h | [Woodruff](https://blog.yossarian.net/2024/12/06/zizmor-ultralytics-injection) |
| @solana/web3.js 1.95.6–1.95.7 | npm | Compromised publish-access account | 2024‑12‑03, live 15:20–20:25 UTC | ~5 h | [GHSA-jcxm-7wvp-g6p5](https://github.com/advisories/GHSA-jcxm-7wvp-g6p5) |
| rspack 1.1.7 | npm | Stolen token | Discovered 2024‑12‑19 02:01 UTC, "lasting approximately 1 hour"; deprecated | ~1 h | [rspack v1.1.8](https://github.com/web-infra-dev/rspack/releases/tag/v1.1.8) |
| Kong Ingress Controller 3.4.0 | Docker Hub image | CI pipeline compromise ("Pwn Request") that stole a GitHub token, "rather than a leaked access token to DockerHub" | Pushed 2024‑12‑23 (Kong's timeline heading says 12‑24) → deleted 2025‑01‑02 | ~9–10 days | [Kong](https://konghq.com/blog/product-releases/december-2024-unauthorized-kong-ingress-controller-3-4-0-build) |
| reviewdog/action-setup | GitHub Actions | Not stated | 2025‑03‑11, 18:42–20:31 UTC | ~2 h | [CISA](https://www.cisa.gov/news-events/alerts/2025/03/18/supply-chain-compromise-third-party-tj-actionschanged-files-cve-2025-30066-and-reviewdogaction) |
| tj-actions/changed-files | GitHub Actions, all tags moved | Stolen token | About 03‑14 16:00 UTC → confirmed 17:00 → removed by GitHub 03‑15 14:00 → clean restore 22:00; CISA's audit window is 03‑12 to 03‑15 | ~22 h (CISA: up to ~3.5 days) | [StepSecurity](https://www.stepsecurity.io/blog/harden-runner-detection-tj-actions-changed-files-action-is-compromised), [CISA](https://www.cisa.gov/news-events/alerts/2025/03/18/supply-chain-compromise-third-party-tj-actionschanged-files-cve-2025-30066-and-reviewdogaction) |
| Gluestack / react-native-aria (17 packages) | npm | Token compromise | From 2025‑06‑06 21:33 GMT → all versions deprecated 06‑08 01:22 GMT | ~28 h to deprecation | [SecurityWeek](https://www.securityweek.com/react-native-aria-packages-backdoored-in-supply-chain-attack/), [Gluestack](https://gluestack.io/blogs/public-incident-report) |
| eslint-config-prettier | npm | Phishing | Published and reported 2025‑07‑18 (spotted in an npmdiff.dev diff) → deprecated the same day → removed by npm by 07‑21 | < 1 day to detect, ~3 days to remove | [eslint-config-prettier#339](https://github.com/prettier/eslint-config-prettier/issues/339) |
| num2words 0.5.15 / 0.5.16 | PyPI | Phishing | Publish time not stated → reported 2025‑07‑28 15:59 UTC → removed 16:38 and 16:54 | "Within hours" | [PyPI](https://blog.pypi.org/posts/2025-07-31-incident-report-phishing-attack/) |
| Nx (s1ngularity) | npm | GitHub Actions injection, then token theft | 2025‑08‑26, "active for 4 hours"; noticed by its missing provenance | ~4 h | [Nx](https://nx.dev/blog/s1ngularity-postmortem) |
| chalk, debug and others | npm | Phishing | 2025‑09‑08, Aikido alert 13:16 UTC → maintainer acknowledged 15:15 | ~2 h | [Aikido](https://www.aikido.dev/blog/npm-debug-and-chalk-packages-compromised), [GitHub](https://github.blog/security/supply-chain-security/the-case-for-a-cooldown-why-dependabot-now-waits-before-issuing-version-updates/) |
| Trivy v0.69.4 | GitHub Releases, Docker Hub, GHCR, ECR, and the trivy-action and setup-trivy tags | CI credential theft | Tag 2026‑03‑19 17:43 → binaries 18:26 → Socket alert 19:15 → tag deleted 23:13; action tags restored 03‑20 05:40; Docker Hub exposed 03‑19 18:24 to 03‑23 01:36 | ~5 h (release), ~12 h (action), ~3.3–3.9 days (Docker Hub, to removal and quarantine) | [Snyk](https://snyk.io/articles/trivy-github-actions-supply-chain-compromise/), [Docker](https://www.docker.com/blog/trivy-supply-chain-compromise-what-docker-hub-users-should-know/) |
| LiteLLM | PyPI | Token stolen via Trivy | Upload → reported 1 h 19 min → quarantined 2 h 32 min; 119k downloads in the window | ~2.5 h | [PyPI](https://blog.pypi.org/posts/2026-04-02-incident-report-litellm-telnyx-supply-chain-attack/) |
| telnyx | PyPI | Same campaign | Upload → reported 1 h 45 min → quarantined 3 h 42 min | ~3.7 h | [PyPI](https://blog.pypi.org/posts/2026-04-02-incident-report-litellm-telnyx-supply-chain-attack/) |
| axios 1.14.1 / 0.30.4 | npm, via malicious transitive plain-crypto-js@4.2.1 | Maintainer account hijack | plain-crypto-js 03‑30 23:59:12 → Socket alert 00:05:41 → axios 00:21 / 01:00 → public issue 02:19 → removed 03:25–03:40 UTC on 03‑31 | ~3 h | [Datadog](https://securitylabs.datadoghq.com/articles/axios-npm-supply-chain-compromise/), [Socket](https://socket.dev/blog/axios-npm-package-compromised) |
| Bitwarden CLI 2026.4.0 | npm | CI compromise | 2026‑04‑22, live 5:57–7:30 PM ET | ~1.5 h | [BleepingComputer](https://bleepingcomputer.com/news/security/bitwarden-cli-npm-package-compromised-to-steal-developer-credentials) |
| TanStack (84 versions, 42 packages) | npm, through the legitimate trusted-publisher workflow | `pull_request_target` plus cache poisoning | 2026‑05‑11 19:20:39 → detected 19:46 → deprecated 21:03 → removed 22:13–23:55 UTC | ~1.7 h to deprecate, ~4.5 h to remove | [TanStack](https://tanstack.com/blog/npm-supply-chain-compromise-postmortem) |
| arrayref 0.3.10, internment, append-only-vec | crates.io | Maintainer credentials or machine | Live 86, 90 and 107 minutes on 2026‑08‑20 | ~1.8 h | [The Hacker News](https://thehackernews.com/2026/08/rust-supply-chain-attack-puts-build.html), [linuxiac](https://linuxiac.com/rust-supply-chain-attack-compromises-popular-arrayref-crate/) |

Incidents with no usable window are left out of the counts below: rand-user-agent (flagged 2025‑05‑05, publish time not found, [BleepingComputer](https://bleepingcomputer.com/news/security/supply-chain-attack-hits-npm-package-with-45-000-weekly-downloads)), and both waves of the Shai-Hulud npm worm, which spread over days with no per-version windows published ([GitHub](https://github.blog/security/supply-chain-security/our-plan-for-a-more-secure-npm-supply-chain/), [Wiz](https://www.wiz.io/blog/shai-hulud-npm-supply-chain-attack), [Aikido](https://www.aikido.dev/blog/shai-hulud-2-0-unknown-wonderer-supply-chain-attack)). node-ipc 10.1.1/10.1.2 was sabotage by its own maintainer, on the registry for less than 24 hours ([Snyk](https://snyk.io/blog/peacenotwar-malicious-npm-node-ipc-package-vulnerability/)).

### A different population: new packages and dependency confusion

- **torchtriton**, PyPI dependency confusion: live 2022‑12‑25 to 12‑30, about 5 days ([PyTorch](https://pytorch.org/blog/compromised-nightly-dependency/)). [A]
- **boltdb-go**, a Go module typosquat: undetected for more than three years, served from the Go module proxy's cache after the attacker cleaned the git tag ([InfoWorld](https://www.infoworld.com/article/3817921/malicious-package-found-in-the-go-ecosystem.html)). [A]

### Coverage by cooldown length

Across the 31 incidents above, each counted at its longest exposure and at removal where removal differs from detection. rspack and Gluestack are counted to deprecation, the last action their sources report:

| Cooldown | Incidents covered | Share | 2024–2026 only (19) |
|---|---|---|---|
| ≤ 1 day | 18 | ~58% | 13 |
| ≤ ~4 days | 23 | ~74% | 17 |
| ≤ 7 days | 24 | ~77% | 17 |
| ≤ 14 days | 26 | ~84% | 18 (Kong, ~10 days) |
| Longer (about a month to 16 months) | 5 | ~16% | xz, 33 days |

This is a tally of a hand-picked list. It illustrates the shape and measures nothing.

## Aggregate studies

1. **Ohm et al., "Backstabber's Knife Collection" (DIMVA 2020)** [M] ([arXiv](https://arxiv.org/abs/2005.09535))
   - Population: 174 malicious packages on npm, PyPI and RubyGems, 2015–2019, typosquats and compromises mixed.
   - From upload to public advisory: mean 209 days, median 67, range −1 to 1,216.
   - It predates modern registry scanning and does not describe today.
2. **Guo et al., "An Empirical Study of Malicious Code in PyPI Ecosystem" (ASE 2023)** [M] ([arXiv](https://arxiv.org/abs/2309.11021))
   - Population: 861 malicious PyPI packages.
   - 28% went more than two months from release to discovery, and 40% were still being downloaded more than two months after discovery, per PyPI's download records.
3. **PyPI's own reporting** [M]. This measures report to action, not publish to removal.
   - 2023: most report responses completed in under about 485 minutes ([PyPI](https://blog.pypi.org/posts/2023-09-18-inbound-malware-reporting/)).
   - 2024: since August 2023, over 90% of issues resolved in under 24 hours ([PyPI](https://blog.pypi.org/posts/2024-08-16-safety-and-security-engineer-year-in-review/)).
4. **uzomuzo ADR‑0028 (October 2026)** [M, not peer-reviewed] ([ADR](https://github.com/future-architect/uzomuzo-oss/blob/main/docs/adr/0028-dormant-release-marker.md)). This is the closest match to the population that matters here.
   - Population: OpenSSF malicious-packages npm records for packages at least a year old with at least three earlier benign releases.
   - Measures first malicious publish to earliest recorded detection. Not removal: npm does not record removal times.
   - Per package (n = 1,515): median about 6 hours; 89% within 1 day; 98% within 7 days. Bulk account-compromise waves account for 1,418 of these, 99% of them within 7 days.
   - The 97 packages from one-off incidents: 86% within 7 days and 8% over 30 days.
   - A separately selected set of 27 incidents, those whose first malicious version followed a year with no releases, measured to when the record entered the OpenSSF data rather than to first detection: all 7 spanning several release lines within 3 days. Of the 20 single-line incidents, 16 were within 7 days; the rest took 12.6, 57 and 310 days, and one has no timestamp.
   - event-stream, ua-parser-js, coa and rc are absent from its data.
5. **The 100 most recent npm malware advisories in GitHub's database** [M, small sample] ([DEV](https://dev.to/mag_solutions_ai/we-measured-how-late-a-malware-advisory-arrives-sometimes-95-days-fn1))
   - Mostly new packages: 95% were under 30 days old.
   - Median 0 days from first publish to advisory; 30% over 1 day; maximum 95 days.
6. **Curated incident lists** [A, aggregated]
   - Woodruff: of 10 incidents, 8 had windows under a week and 9 under two weeks ([blog](https://blog.yossarian.net/2025/11/21/We-should-all-be-using-dependency-cooldowns)).
   - daniakash: of 21 incidents, a 7-day cooldown blocks 11 and does not help 8 ([blog](https://daniakash.com/posts/simplest-supply-chain-defense/)). GitHub's case for cooldowns relies on this list, which gives Nx as about 1 day where Nx's postmortem says 4 hours, and xz as "2+ years", counting the maintainer's infiltration rather than the release's exposure.
7. **A study of Dependabot cooldown settings** [M, adoption only] ([arXiv](https://arxiv.org/abs/2609.16605))
   - Across 244 configured ecosystems the median and mode cooldown is 7 days, and 64% use exactly 7.
   - The authors say it is no evidence that cooldowns prevent incidents.

No peer-reviewed study joins publish times with removal times for compromised established packages.

## What tools and practitioners recommend

None of these numbers was derived from a distribution of incidents. Most are a chosen default backed by a few anecdotes.

| Source | Cooldown | Stated basis | Label |
|---|---|---|---|
| Dependabot ([changelog](https://github.blog/changelog/2026-07-14-dependabot-version-updates-introduce-default-package-cooldown/), [rationale](https://github.blog/security/supply-chain-security/the-case-for-a-cooldown-why-dependabot-now-waits-before-issuing-version-updates/)) | 3 days by default since 2026‑07‑14; security updates immediate | The daniakash list; it "pushes you past the window where most of these attacks live"; consistency with other tools | [O] |
| Renovate `security:minimumReleaseAge*` presets ([presets](https://docs.renovatebot.com/presets-security/), [docs](https://docs.renovatebot.com/key-concepts/minimum-release-age/)) | 3 days; 14 days appears only as an example | Time for scanners; npm's and crates.io's unpublish windows | [O] |
| pnpm 11 ([docs](https://pnpm.io/settings/dependency-resolution)) | 1 day | "In most cases, malicious releases are discovered and removed from the registry within an hour", unsourced and contradicted by several incidents above | [O] |
| OpenSSF ([blog](https://openssf.org/blog/2026/05/20/detecting-malicious-packages-using-the-osv-api/)) | At least 3 days | "Most" classified within 3 days, no data | [O] |
| Palo Alto Unit 42 ([blog](https://unit42.paloaltonetworks.com/monitoring-npm-supply-chain-attacks/)) | 24–72 hours | "Most" identified and removed in that window, no data | [O] |
| Datadog Security Labs ([blog](https://securitylabs.datadoghq.com/articles/dependency-cooldowns/)) | 1 week common, 1 day minimum | axios and s1ngularity | [O] |
| Woodruff ([blog](https://blog.yossarian.net/2025/11/21/We-should-all-be-using-dependency-cooldowns), [redux](https://blog.yossarian.net/2025/12/13/cooldowns-redux)) | 7 days as the example; 14 catches 9 of his 10 | His 10 incidents | [O] |
| Snyk ([docs](https://docs.snyk.io/scan-with-snyk/pull-requests/snyk-pull-or-merge-requests/upgrade-dependencies-with-automatic-prs-upgrade-prs)) | 21 days before suggesting an upgrade | Bugs, unpublishing, compromised accounts; no data | [O] |

Cal Paterson argues the other side: a cooldown relies on someone else installing the release first, and registries should hold uploads in a queue instead, as Debian's testing migration already delays packages 2–10 days ([blog](https://calpaterson.com/deps.html)). [O]

## Costs and counter-cases

**Waiting delays security fixes.**

- Average time to exploit was 5 days in 2023 with outliers removed, 47 days without, over 138 vulnerabilities ([Mandiant](https://cloud.google.com/blog/topics/threat-intelligence/time-to-exploit-trends-2023)). [M]
- 32.1% of known-exploited vulnerabilities in the first half of 2025 were exploited on or before the day of disclosure ([VulnCheck](https://www.vulncheck.com/blog/state-of-exploitation-1h-2025)). [M]
- Both measure products rather than library dependencies: VulnCheck's largest categories are content management systems, edge devices and server software. No good data was found for library vulnerabilities.
- So security fixes must bypass the cooldown. Renovate's `vulnerabilityAlerts` does by default ([`minimumReleaseAge: null`](https://github.com/renovatebot/renovate/blob/44.198.3/lib/config/options/index.ts#L2686)), and so do Dependabot's security updates.

**Some attacks arrive looking old.**

- **Moved tags:** tj-actions and trivy-action moved existing tags to malicious commits; Trivy and Kong overwrote image tags. Pinning by commit SHA or digest is what defeats these.
- **Replaced old versions:** ctx replaced an existing version. Since July 2026 PyPI rejects new files on releases older than 14 days ([PyPI](https://blog.pypi.org/posts/2026-07-22-releases-now-reject-new-files-after-14-days/)).
- **Caches outlive removal:** the Go module proxy served boltdb-go for years.
- **Transitive dependencies:** axios pulled in a malicious dependency published 21 minutes earlier. A cooldown on update pull requests does not cover dependencies resolved at install time; an installer-level setting does.
- **Provenance alone is not enough:** TanStack's malicious versions were published through its legitimate trusted-publisher workflow.

**Detection is not removal.** Scanning vendors often flag a release within minutes: Socket flagged axios's malicious dependency within 6 minutes of its publication, before axios itself was published. Removal can lag days: eslint-config-prettier took about 3, and Gluestack and rspack report deprecating their versions, not removing them. A cooldown protects only until the version stops resolving, so it should be sized to removal, not detection.

**The free-rider concern is partly real.** Some releases were caught only because someone ran them: tj-actions and Trivy by CI egress monitoring, xz by benchmarking, coa by a broken CLI. Others were caught by scanning with nobody installing them: axios, chalk, and Nx by its missing provenance. Wide adoption of cooldowns weakens only the first kind.
