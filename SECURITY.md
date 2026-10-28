# Security policy

This policy applies to every repository owned by [@cravingmaker](https://github.com/cravingmaker) that doesn't provide its own.

## Supported versions

Only the latest published release of each project is supported for security fixes. For projects without releases, the default branch is supported. Upgrade older releases to receive fixes. Archived repositories are no longer maintained.

## Reporting a vulnerability

**Don't report vulnerabilities in public issues, pull requests, or discussions.** Report them privately through GitHub's private vulnerability reporting instead:

1. Open the affected repository's **Security** tab.
2. Click **Report a vulnerability**, or go directly to `https://github.com/cravingmaker/<repository>/security/advisories/new`.
3. Complete the report form.

Please include:

- Affected package and version, plus related runtime and tool versions (for example, Node.js and peer dependencies).
- A minimal reproduction or proof of concept against the latest release.
- Expected behavior, actual behavior, and potential impact.
- A suggested fix or mitigation, if you have one.

If private vulnerability reporting isn't available for a repository, email **[cravingmaker.id@gmail.com](mailto:cravingmaker.id@gmail.com)** with "Security" in the subject line instead.

For ordinary bugs and feature requests, use the affected repository's issues.

## What to expect

These projects are maintained by one volunteer, so these are targets rather than guarantees:

- **Acknowledgement** within 5 business days.
- **Initial assessment** within 14 days, including whether the report is accepted as a vulnerability.
- **Coordinated disclosure** within 90 days of the report, or sooner once a fix is released. The timing is agreed with the reporter and may be shortened if the vulnerability is being actively exploited.

Accepted vulnerabilities are fixed privately and disclosed through a GitHub security advisory, with a CVE requested when appropriate. After review, GitHub adds published advisories to the GitHub Advisory Database, which can trigger Dependabot alerts for affected users. Reporters are credited in the advisory unless they prefer to remain anonymous. There's no paid bug bounty.

## Scope

In scope:

- Source code in these repositories and the packages published from them.
- GitHub Actions workflows in these repositories, such as a workflow that could expose secrets or let untrusted code run with write access.

Out of scope:

- Vulnerabilities in third-party dependencies. Report those upstream. You can still report one here if it affects these projects in a way that needs more than a dependency update.
- Vulnerabilities in GitHub or npm themselves. Report those through the [GitHub Security Bug Bounty](https://bounty.github.com/).
- Findings that require an already compromised machine or account, social engineering, or denial-of-service testing.
- Scanner output without a demonstrated, exploitable impact.

## Good-faith research

If you make a good-faith effort to follow this policy, your research is considered authorized, and @cravingmaker won't pursue or support legal action against you. Test only against your own installations, don't access or modify other people's data, avoid degrading services, and allow reasonable time for a fix before any disclosure. This covers only these projects, not systems run by others, such as GitHub or npm.

## AI-assisted reports

AI-assisted research is welcome, but you're responsible for the report. If you used AI tools to find or write up an issue, say so, verify the finding yourself, and include a proof of concept that works against a released version. Speculative or unverified reports may be closed without a detailed response.

## Verifying releases

Packages that these projects publish to npm are released from GitHub Actions using npm trusted publishing, so each release carries a provenance attestation that links it to its source commit and workflow run. To verify the packages installed in your project, run:

```sh
npm audit signatures
```
