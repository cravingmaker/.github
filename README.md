# .github

Default community health files for repositories owned by
[@cravingmaker](https://github.com/cravingmaker).

GitHub uses these files in any of my repositories that don't have their own.
A repository's own file always takes precedence.

## What's included

| File                                                                    | Purpose                                                    |
| ----------------------------------------------------------------------- | ---------------------------------------------------------- |
| [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)                              | Expected behavior, and how to report concerns              |
| [`CONTRIBUTING.md`](CONTRIBUTING.md)                                    | How to contribute                                          |
| [`SECURITY.md`](SECURITY.md)                                            | Supported versions, and how to report vulnerabilities      |
| [`SUPPORT.md`](SUPPORT.md)                                              | Where to get help                                          |
| [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE)                     | Issue forms and the template chooser settings              |
| [`.github/pull_request_template.md`](.github/pull_request_template.md)  | Default pull request description                           |
| [`.github/VULNERABILITY_REPORT.yml`](.github/VULNERABILITY_REPORT.yml)  | Form for private vulnerability reports                     |
| [`.github/FUNDING.yml`](.github/FUNDING.yml)                            | Sponsor button that links to GitHub Sponsors               |

## How it works

- **Per file type:** a repository without its own `CONTRIBUTING.md` uses this
  one, and the same goes for each file above.
- **Issue templates are all or nothing:** a repository with any file in its own
  `.github/ISSUE_TEMPLATE/` folder uses none of these templates.
- **Not inherited:** workflows, `CODEOWNERS`, Dependabot settings, licenses, and
  READMEs have to live in each repository.

For background and the reasoning behind this setup, see
[the guide](docs/DOT_GITHUB_GUIDE.md).

## License

[MIT](LICENSE), except [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md), which is
adapted from Contributor Covenant and licensed under
[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
