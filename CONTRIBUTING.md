# Contributing

Thanks for your interest in contributing! This guide applies to every repository
owned by [@cravingmaker](https://github.com/cravingmaker) that doesn't have its
own. If a repository has its own contributing guide, follow that instead.

## Ways to contribute

- Report bugs and suggest features through the issue forms.
- Improve documentation, from typos to missing explanations.
- Fix bugs or implement accepted features with a pull request.
- Help others by answering questions in issues.

## Before you start

- For anything beyond a small fix, open an issue first so we can agree on the
  approach before you spend time on it.
- Comment on an issue before you start working on it, to avoid duplicate effort.
- Check the repository's README for setup instructions and project-specific
  conventions.

## Making changes

1. Fork the repository and create a branch from the default branch.
2. Keep the change focused: one fix or feature per pull request.
3. Add or update tests and documentation where it makes sense.
4. Run the project's checks locally before pushing, as described in its README.
5. Open a pull request and fill in the template.

## Commit messages and pull request titles

Most repositories use [Conventional Commits](https://www.conventionalcommits.org/)
and squash merging, so the pull request title becomes the commit message. For
example:

- `fix: handle an empty configuration file`
- `feat: add a rule for import ordering`
- `feat!: drop support for Node.js 20` (the `!` marks a breaking change)

Release tooling uses these titles to choose the next version and to write the
changelog.

## Review process

- Pull requests are reviewed as time allows, usually within a week.
- Automated checks must pass before a pull request is merged.
- If you're asked for changes, push them to the same branch.
- Inactive pull requests may be closed after a while. You're welcome to reopen
  them later.

## AI-assisted contributions

You may use AI tools, but you're responsible for everything you submit:

- Review, understand, and test every change before you open a pull request.
- Say in the pull request how you used AI tools.
- Don't submit anything you can't explain or defend in review.

Low-effort or unreviewed AI-generated contributions will be closed.

## Licensing

By contributing, you agree that your contributions are licensed under the
license of the repository you contribute to.

## Code of conduct and security

Everyone who takes part must follow the
[code of conduct](https://github.com/cravingmaker/.github/blob/main/CODE_OF_CONDUCT.md).
Report security vulnerabilities privately, as described in the
[security policy](https://github.com/cravingmaker/.github/blob/main/SECURITY.md),
and never in a public issue or pull request.
