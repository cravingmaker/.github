# GitHub's `.github`: the special directory and the special repository

A research report and decision guide.

| | |
|---|---|
| **Snapshot date** | 2026-10-22 |
| **Verified against** | docs.github.com (fetched 2026-10-22) and the GitHub Changelog. See [Sources](#9-sources). |
| **Written for** | `cravingmaker`, a **personal** GitHub account. Differences for organizations and enterprises are called out where they matter. |
| **Labels** | **(preview)** means GitHub marks the feature as public preview, so details may change. |

> [!NOTE]
> This guide records the research and decisions behind this repository. GitHub doesn't read it, so it has no effect on other repositories.

**How to use this guide:** read the [TL;DR](#tldr) and the [decision guide](#6-decision-guide) to make your choices. Use the [file-by-file reference](#4-file-by-file-reference) while you implement, and go through the [best practices](#5-best-practices) before going live.

---

## TL;DR

1. **Two different things share the name.** The `.github/` **folder** inside any repository holds that repository's configuration: workflows, templates, `CODEOWNERS`, Dependabot, Copilot files, and so on. The `.github` **repository** is a repo literally named `.github`, owned by a user or an organization. It provides **account-wide defaults**, plus a few features that only organizations get.
2. **The `.github` repository must be public.** Its defaults apply to every repository the account owns, public *and* private, that doesn't have its own file of that type.
3. **Only a fixed list of files is inherited:** `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md`, `SUPPORT.md`, `ACCESSIBILITY.md`, `FUNDING.yml`, `VULNERABILITY_REPORT.yml`, issue templates and their `config.yml`, pull request templates, and discussion category forms. Nothing else is inherited.
4. **Not inherited, despite a common belief:** workflows, `dependabot.yml`, `CODEOWNERS`, `release.yml`, `LICENSE`, `README`, Copilot instructions, agents, skills, and hooks. A workflow in the `.github` repository runs only for the `.github` repository.
5. **Inheritance works per file type, except for issue templates.** If a repository has *anything* valid in its own `.github/ISSUE_TEMPLATE/` folder, even just a `config.yml`, it gets *none* of the default templates.
6. **Defaults are virtual.** They don't appear in other repositories' file browsers, history, clones, downloads, or packages. That's also why a `LICENSE` can't be a default.
7. **Watch the doubled path.** Inside the `.github` repository, templates live in `.github/ISSUE_TEMPLATE/`, so from your parent folder the path is `Public/.github/.github/ISSUE_TEMPLATE/`. `FUNDING.yml` and `VULNERABILITY_REPORT.yml` must also sit in that inner `.github/` folder. The Markdown defaults can sit at the repository root.
8. **Personal accounts get less.** A personal account's `.github` repository only provides community-health defaults. The org profile README (`profile/README.md`), workflow templates (`workflow-templates/`), org-wide Copilot agents (`agents/`), and the member-only README (`.github-private`) are organization features. Your personal profile README lives in `cravingmaker/cravingmaker`.
9. **`.github/` is a privileged area.** It holds code that runs with your tokens (workflows, hooks, agent setup steps) and files that steer reviewers and AI agents. Protect it with `CODEOWNERS` and rulesets, pin your actions, and keep `GITHUB_TOKEN` least-privilege.
10. **Time-sensitive:** on **2026-11-02**, GitHub starts enforcing a default policy that blocks `pull_request_target` workflows in public repositories. The two repositories checked locally (`eslint-config`, `prettier-config`) use `workflow_run`, not `pull_request_target`, so they aren't affected.

---

## Contents

1. [Two things named `.github`](#1-two-things-named-github)
2. [Quick reference: every file GitHub recognizes](#2-quick-reference-every-file-github-recognizes)
3. [The `.github` repository in depth](#3-the-github-repository-in-depth)
4. [File-by-file reference](#4-file-by-file-reference)
5. [Best practices](#5-best-practices)
6. [Decision guide](#6-decision-guide)
7. [Applying this to cravingmaker](#7-applying-this-to-cravingmaker)
8. [Recent changes to watch](#8-recent-changes-to-watch)
9. [Sources](#9-sources)

---

## 1. Two things named `.github`

### 1.1 The `.github/` folder (in any repository)

`.github/` is a folder at the root of a repository. The leading dot hides it in most file managers. GitHub reads configuration from it, and those files affect **only the repository that contains them**.

Files fall into three placement groups:

- **Must be inside `.github/`:** workflows (`.github/workflows/`), `dependabot.yml`, issue templates (`.github/ISSUE_TEMPLATE/`), discussion forms (`.github/DISCUSSION_TEMPLATE/`), `FUNDING.yml`, `VULNERABILITY_REPORT.yml`, `release.yml`, `secret_scanning.yml`, and most Copilot customization files (instructions, prompts, agents, hooks). Skills can also live in `.claude/skills/` or `.agents/skills/`, and `AGENTS.md` can go anywhere.
- **Can be in `.github/`, the root, or `docs/`:** `README`, `CONTRIBUTING`, `CODE_OF_CONDUCT`, `SECURITY`, `SUPPORT`, `ACCESSIBILITY`, `CODEOWNERS`, and pull request templates. If more than one copy exists, GitHub uses the first one it finds, in the order **`.github/` → root → `docs/`**.
- **Must be at the root, not in `.github/`:** `LICENSE`, `CITATION.cff`, `.gitattributes`, and dev container configuration (`.devcontainer/`).

GitHub also reads different files from different branches:

| Read from | Files |
|---|---|
| The default branch | Issue, PR, and discussion templates; `FUNDING.yml`; `dependabot.yml`; `CITATION.cff`; Copilot hooks; `copilot-setup-steps.yml` |
| The pull request's **base** branch | `CODEOWNERS` (each branch can have its own) |
| The pull request's **head** branch | The custom instructions, agent instructions, and skills used by Copilot code review |
| The triggering commit | Most workflows. For example, `pull_request` runs the PR's merge commit, while `pull_request_target`, `workflow_run`, and `schedule` use the default branch. |

### 1.2 The `.github` repository (account level)

This is a repository **named exactly `.github`**, owned by a personal account or an organization. GitHub treats it specially in two ways:

- **Default community health files.** Any repository owned by the same account that lacks its own file of a supported type shows the `.github` repository's version instead. This applies whatever the visibility of the destination repository. The `.github` repository itself must be public.
- **Organization features.** For organizations, it also hosts the public profile README (`profile/README.md`), workflow templates (`workflow-templates/`), and org-wide Copilot custom agents (`agents/`).

Apart from that, it's an ordinary repository. Its own `.github/` folder configures only itself.

### 1.3 Related special repositories

| Repository | Account type | Visibility | Purpose |
|---|---|---|---|
| `USERNAME/USERNAME` | Personal | Public | Your **profile README** (`README.md` at the root). This is not the `.github` repository. |
| `OWNER/.github` | Personal or organization | Public (required) | Account-wide defaults. For organizations, it also holds the public profile README, workflow templates, and org custom agents. |
| `ORG/.github-private` | Organization | Private | The member-only org profile README (`profile/README.md`) and org custom agents (`agents/`). |
| `.github-private` in an org that your enterprise designates | Enterprise | Internal or private | Enterprise custom agents (`agents/`) and Copilot governance settings (`copilot/managed-settings.json`, `copilot/team-mappings.json`, `copilot/teams/*.json`). |

### 1.4 Mental model: two zones inside the `.github` repository

```text
OWNER/.github                          (the special repository; this directory)
│
├── Zone A: read by GitHub on behalf of OTHER repositories
│   ├── CODE_OF_CONDUCT.md, CONTRIBUTING.md, SECURITY.md,
│   │   SUPPORT.md, ACCESSIBILITY.md         (root, .github/, or docs/)
│   ├── pull_request_template.md             (root, .github/, or docs/)
│   ├── .github/ISSUE_TEMPLATE/              default issue templates + config.yml
│   ├── .github/DISCUSSION_TEMPLATE/         default discussion category forms
│   ├── .github/FUNDING.yml                  default Sponsor button
│   ├── .github/VULNERABILITY_REPORT.yml     default private vulnerability report form
│   ├── profile/README.md                    org public profile          (orgs only)
│   ├── workflow-templates/                  org workflow templates      (orgs only)
│   └── agents/                              org Copilot custom agents   (orgs only)
│
└── Zone B: ordinary configuration for THIS repository only
    ├── README.md                            this repository's landing page (not a profile)
    ├── .github/workflows/                   CI for this repository, plus any reusable
    │                                        workflows that other repositories call
    ├── .github/dependabot.yml
    ├── .github/CODEOWNERS
    └── .github/copilot-instructions.md, .github/agents/, .github/skills/, ...
```

Two subtleties:

- **`agents/` and `.github/agents/` are different.** In an organization's `.github` or `.github-private` repository, `agents/` at the root makes agents available org-wide (Zone A). `.github/agents/` makes them usable only inside that one repository (Zone B), and GitHub suggests it as a staging area for testing agents before release.
- **Paths are doubled.** In this checkout, a default bug form would be `Public/.github/.github/ISSUE_TEMPLATE/bug_report.yml`: the outer `.github` is the repository and the inner one is its configuration folder. Likewise, a reusable workflow hosted here is called as `cravingmaker/.github/.github/workflows/ci.yml@<ref>`.

---

## 2. Quick reference: every file GitHub recognizes

The last column answers: "If I put this in the `.github` repository, do my other repositories inherit it?"

### 2.1 Community health and project documents

| File | Where it can live | What GitHub does with it | Inherited? |
|---|---|---|---|
| `README.md` | `.github/` → root → `docs/` | Repository landing page. Content beyond 500 KiB is truncated. | No |
| `CONTRIBUTING.md` | `.github/` → root → `docs/` | Linked when someone opens an issue or PR. Adds a "Contributing" sidebar link and appears on the `/contribute` page. | **Yes** |
| `CODE_OF_CONDUCT.md` | `.github/` → root → `docs/` | Sidebar link and community profile entry. | **Yes** |
| `SECURITY.md` | `.github/` → root → `docs/` | The "Security policy" page in the **Security** tab (now labeled "Security and quality"). Vulnerability reporters see a banner linking to it. | **Yes** |
| `SUPPORT.md` | `.github/` → root → `docs/` | Linked when someone creates an issue. | **Yes** |
| `ACCESSIBILITY.md` | `.github/` → root → `docs/` | An **Accessibility** tab on the repository overview, linked from the About section. | **Yes** |
| `FUNDING.yml` | `.github/` only | The **Sponsor** button. | **Yes** (must be in the `.github` repository's `.github/` folder) |
| `LICENSE` | Root | License detection and display. | **Never.** It must ship with the code. |
| `CITATION.cff` | Root | The "Cite this repository" prompt (APA and BibTeX). | No |
| `GOVERNANCE.md` | Anywhere | Older docs (GitHub Enterprise Server 3.15) listed it as a default. The current GitHub.com list doesn't. | Not listed |

### 2.2 Issues, pull requests, and discussions

| File | Purpose | Inherited? |
|---|---|---|
| `.github/ISSUE_TEMPLATE/*.md` | Markdown issue templates, with YAML front matter (`name`, `about`, `title`, `labels`, `assignees`, `type`) | **Yes**, but all-or-nothing per folder |
| `.github/ISSUE_TEMPLATE/*.yml` | Issue forms with web form fields **(preview)** | **Yes** |
| `.github/ISSUE_TEMPLATE/config.yml` | Template chooser settings: `blank_issues_enabled` and `contact_links` | **Yes** |
| `pull_request_template.md` | Default PR body. Can live in `.github/`, the root, or `docs/`. The filename is case-insensitive, and `.md` or `.txt` both work. | **Yes** |
| `PULL_REQUEST_TEMPLATE/*.md` | Multiple PR templates, in a subfolder of `.github/`, the root, or `docs/`. Only selectable through the `?template=` URL parameter. | **Yes** |
| `.github/DISCUSSION_TEMPLATE/<category-slug>.yml` | Discussion category forms | **Yes** |

### 2.3 Security and supply chain

| File | Purpose | Inherited? |
|---|---|---|
| `.github/VULNERABILITY_REPORT.yml` (or `.yaml`) | A custom form for private vulnerability reports (new on 2026-10-01) | **Yes**, but a repository's own file wins |
| `.github/dependabot.yml` (or `.yaml`) | Dependabot version updates, plus customization of security update PRs | No |
| `.github/secret_scanning.yml` | `paths-ignore`, which closes secret-scanning alerts for matching paths and excludes them from push protection | No |
| `.github/codeql/extensions/<pack>/` | CodeQL model packs that code scanning's default setup picks up automatically **(preview)** | No. Organizations publish packs to the Container registry instead. |
| `.github/workflows/actions.lock` | Workflow dependency lockfile **(preview, 2026-10-14)** | No |

### 2.4 Ownership, review, and releases

| File | Purpose | Inherited? |
|---|---|---|
| `CODEOWNERS` | Automatic review requests and code-owner approvals. Lives in `.github/` → root → `docs/` (first found wins), is read per branch, and must be under 3 MB. | No |
| `.github/release.yml` | Categories and exclusions for automatically generated release notes | No |

### 2.5 GitHub Actions

| File | Purpose | Inherited? |
|---|---|---|
| `.github/workflows/*.yml` / `*.yaml` | Workflows, including reusable workflows. They must sit directly in this folder; subfolders aren't supported. | No. A workflow runs only in the repository that contains it. |
| `.github/actions/<name>/action.yml` | A **convention** (not a requirement) for storing local actions | No |
| `workflow-templates/*.yml` + `*.properties.json` (+ optional `*.svg`), at the root of the `.github` repository | Organization workflow templates, shown in the "New workflow" picker | Organization-only feature |
| `.github/workflows/copilot-setup-steps.yml` | The environment for the Copilot cloud agent | No |
| `.github/workflows/copilot-code-review.yml` | The environment for Copilot code review | No |
| `.github/workflows/<name>.md` + `<name>.lock.yml` | GitHub Agentic Workflows **(preview)** | No |

### 2.6 Copilot and AI agents

| File | Purpose | Inherited? |
|---|---|---|
| `.github/copilot-instructions.md` | Repository-wide custom instructions | No. Organizations set org-wide instructions in settings. |
| `.github/instructions/**/*.instructions.md` | Path-specific instructions, using `applyTo` and `excludeAgent` | No |
| `AGENTS.md` (anywhere; the nearest one wins), `CLAUDE.md` or `GEMINI.md` (root) | Agent instructions | No |
| `REVIEW.md` | Extra instructions read by Copilot code review | No |
| `.github/prompts/*.prompt.md` | Prompt files, used in IDEs only | No |
| `.github/agents/<name>.agent.md` | Custom agents for this repository | No. Organizations use `agents/` in `.github` or `.github-private`. |
| `.github/skills/<name>/SKILL.md` (also `.claude/skills/` and `.agents/skills/`) | Agent skills | No |
| `.github/hooks/*.json` | Hooks for the Copilot cloud agent and Copilot CLI | No |

### 2.7 Things that live elsewhere

These are easy to confuse with `.github/` files:

- **Root of each repository:** `LICENSE`, `CITATION.cff`, `.gitattributes` (Linguist overrides), `.devcontainer/devcontainer.json` or `.devcontainer.json`, `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`.
- **IDE folders:** `.vscode/mcp.json` configures MCP servers in VS Code (other IDEs differ). MCP servers for the Copilot cloud agent and code review are set in the repository's settings, not in a file.
- **Root of a special repository:** `profile/README.md`, `workflow-templates/`, and `agents/` (organizations), and `copilot/managed-settings.json` (enterprise `.github-private`).
- **Your personal profile:** `cravingmaker/cravingmaker/README.md`.

### 2.8 Third-party conventions you'll often see in `.github/`

These aren't GitHub features. A tool reads them, so they only matter if you use that tool:

| Path | Tool |
|---|---|
| `.github/labeler.yml` | `actions/labeler` (its default configuration path) |
| `.github/release-drafter.yml` | Release Drafter |
| `.github/renovate.json` / `.json5` | Renovate (one of its supported configuration locations) |
| `.github/settings.yml` | The Probot Settings app (repository settings as code) |
| `.github/actionlint.yaml` | actionlint |
| `.github/zizmor.yml` | zizmor (part of its configuration discovery) |
| `.github/linters/` | Super-Linter |
| `.github/codeql/codeql-config.yml` | CodeQL advanced setup, passed through the `config-file:` input |
| `.github/dependency-review-config.yml` | `actions/dependency-review-action`, passed through the `config-file:` input |

---

## 3. The `.github` repository in depth

### 3.1 Requirements and setup

1. **Name** the repository exactly `.github` and choose the right account as its owner.
2. **Make it public.** GitHub states: "A repository for default files cannot be private."
3. **Commit** the supported files to the **default branch**, in the required folders (see [1.4](#14-mental-model-two-zones-inside-the-github-repository)).
4. **Create the labels** your templates reference, both in the `.github` repository and in every repository that uses them. Labels that don't exist are skipped silently.

### 3.2 How inheritance works

1. For each supported file type, GitHub first looks in the repository itself (`.github/` → root → `docs/`).
2. If the repository has no file of that type, GitHub uses the `.github` repository's file. It applies the same location order inside the `.github` repository.
3. The fallback applies **regardless of the destination repository's visibility**: public, private, and internal repositories all receive defaults.
4. **Issue templates are all-or-nothing.** If a repository defines valid issue templates *or* a `config.yml` in its own `.github/ISSUE_TEMPLATE/` folder, none of the default `ISSUE_TEMPLATE` contents are used. The two sets are never merged.
5. **Defaults are never copied.** They don't appear in other repositories' file browser or history, and they aren't included in clones, packages, or downloads. When you change a default, the change applies at once to every existing and future repository that relies on it.
6. **A repository's own file always wins.** To opt a repository out of a default, give that repository its own file of the same type.

**Worked example.** Suppose a repository has no `CONTRIBUTING.md`, no `SECURITY.md`, and only a `.github/ISSUE_TEMPLATE/config.yml`:

| File type | What visitors see |
|---|---|
| `CONTRIBUTING` | The default from `.github` |
| `SECURITY` | The default from `.github` |
| Issue templates | **None of the defaults.** The repository's own `config.yml` disables them, so visitors see only "Blank issue" (if enabled) and that file's contact links. |

### 3.3 What each account type gets

| Capability | Personal `.github` | Organization `.github` | Organization `.github-private` |
|---|---|---|---|
| Default community health files | Yes | Yes | No |
| Profile README | No (use `USERNAME/USERNAME`) | Yes: `profile/README.md`, public view | Yes: `profile/README.md`, member-only view |
| Workflow templates (`workflow-templates/`) | Documented for organizations only | Yes | Not documented |
| Copilot custom agents for every repository (`agents/`) | No | Yes **(preview)** | Yes **(preview)** |
| Source repository for organization-wide Discussions | Not applicable | Any org repository can be the source; `.github` is a common choice | Not applicable |

### 3.4 What it does *not* do

| Common expectation | Reality | What to do instead |
|---|---|---|
| "Workflows in `.github` run for all my repositories." | They run only for the `.github` repository. | Call **reusable workflows** from each repository. Organizations can also use workflow templates and org rulesets that require status checks. |
| "`dependabot.yml` is inherited." | It isn't. | Add `.github/dependabot.yml` to each repository. Organizations can cover alerts and security updates with security configurations. |
| "`CODEOWNERS` is inherited." | It isn't. | Add `CODEOWNERS` to each repository. |
| "I can set a default `LICENSE` or `README`." | Neither is supported. | Add them to each repository. |
| "Copilot instructions, skills, and hooks are inherited." | They aren't. | Add them to each repository. Personal Copilot instructions (set in your settings) apply only to your own chats. Organizations get org instructions in settings and org agents in `agents/`. |
| "Templates create the labels they reference." | They don't, and missing labels are skipped. | Create the labels in each repository. Organizations can set default labels for *new* repositories. |
| "Defaults ship inside clones and packages." | They don't. | Keep a per-repository copy of any file that must ship. |
| "Files in `.github` can configure repository settings and rulesets." | GitHub has no native feature for this. | Use rulesets (UI or API), or a third-party settings app. |
| "`release.yml`, `secret_scanning.yml`, and `.github/codeql/` are inherited." | Each works per repository only. | Add them where needed. |

### 3.5 Organization and enterprise extras

- **Organization profile README.** A public view comes from `profile/README.md` in the public `.github` repository. A member-only view comes from `profile/README.md` in a **private** `.github-private` repository; internal visibility doesn't work. Signed-in members can switch between the views. Owners can pin up to six repositories for each view. Public organization profiles aren't available to Enterprise Managed Users.
- **Workflow templates.** These go in `workflow-templates/` (see [4.18](#418-workflow-templates-organizations)).
- **Custom agents (preview).** Agent profiles in `agents/` in the org's `.github` or `.github-private` repository are available to every member of the organization, even members who can't access that repository. Enterprises designate one organization's `.github-private` repository as their "source of governance". It holds enterprise agents in `agents/` and Copilot client policy in `copilot/managed-settings.json`, with per-team overrides in `copilot/team-mappings.json` and `copilot/teams/*.json`. When names conflict, the **repository** agent wins over the organization's, and the organization's wins over the enterprise's.

### 3.6 Arranging files inside the `.github` repository

- **Markdown defaults:** keep them at the **root**, so they're visible when someone browses the repository. Use exactly one location per file. Because `.github/` takes precedence, a stray copy there would silently shadow the root copy.
- **Fixed locations:** `.github/ISSUE_TEMPLATE/`, `.github/DISCUSSION_TEMPLATE/`, `.github/FUNDING.yml`, and `.github/VULNERABILITY_REPORT.yml` must live where stated.
- **The repository's own `README.md`:** explain what the repository is and which defaults it provides. It isn't a profile page. Don't put a `README.md` in `.github/`, because it would replace the root README on this repository's page.
- **A `LICENSE` for this repository's own content:** worth adding. It isn't inherited and covers only this repository. If you adapt Contributor Covenant 3.0, which is licensed CC BY-SA 4.0, keep its attribution and license the adapted file under CC BY-SA 4.0 too.

---

## 4. File-by-file reference

### 4.1 `CONTRIBUTING.md`

- **Where GitHub shows it:** as a link when someone opens an issue or pull request, as a "Contributing" link in the sidebar, and on the repository's `/contribute` page.
- **Location:** `.github/`, the root, or `docs/`, searched in that order. The filename is case-insensitive, and supported formats render as rich text.
- **A good default** is short and generic. It says where to ask questions (link to `SUPPORT`) and how to report bugs and propose features. For setup it points readers to the repository's README, and it gives PR expectations and links to the code of conduct and security policy.
- **Long, project-specific guides** stay in their own repositories. A repository's own `CONTRIBUTING.md` overrides the default.

### 4.2 `CODE_OF_CONDUCT.md`

- **Templates:** GitHub offers code-of-conduct templates. Choose **Add file**, name the file `CODE_OF_CONDUCT.md`, then click **Choose a code of conduct template**.
- **Community profile:** "Code of conduct" counts as present if the file isn't empty and doesn't state that the project has no code of conduct.
- **Before adopting one:** make sure you can and will enforce it, and give a working enforcement contact. Keep the attribution of any text you adapt.
- **To centralize it,** write it project-neutral ("this project") instead of naming a specific package.

### 4.3 `SECURITY.md`

- **Where GitHub shows it:** as the security policy in the **Security** tab and in the sidebar. When someone files a private vulnerability report, a banner links to it.
- **Contents:** supported versions, how to report (private vulnerability reporting or an email address), what to include, expected response times, your disclosure policy, and what's out of scope.
- **Wording that works as a default:** "Use **Report a vulnerability** on the affected repository's **Security** tab." Avoid hard-coded links like `.../eslint-config/security/advisories/new`, which only fit one repository.
- **Private vulnerability reporting is a per-repository setting**, not a file. Enable it under **Settings → Advanced Security** in each public repository. Organizations can enable it at scale with security configurations.

### 4.4 `VULNERABILITY_REPORT.yml` (new, 2026-10-01)

- **Purpose:** customizes the form that security researchers fill in when they report privately. GitHub's built-in form asks for a summary, details, a proof of concept, and impact.
- **Location:** `.github/VULNERABILITY_REPORT.yml` or `.yaml` in a repository, or in the `.github/` folder of the account's `.github` repository as a default. A repository's own form takes precedence.
- **Syntax:** the issue-form schema, with `checkboxes`, `dropdown`, `input`, `markdown`, and `textarea` elements. There's no `upload` element here. Fields can be `required`, and `min_length` works on `input` and `textarea`.
- **Fallback:** if the form can't be parsed or validated, reporters see the default form instead.
- **API reports:** once a custom form exists, reports submitted through the REST API must contain its required sections. A REST endpoint returns the form that applies to a repository.
- **Related setting:** **Require a CWE assignment** (per repository) makes reporters tag at least one CWE.
- **Availability** (per the changelog): public repositories with private vulnerability reporting enabled, on the Free, Pro, Team, and Enterprise Cloud plans.

```yaml
# .github/VULNERABILITY_REPORT.yml
name: Private vulnerability report
description: Information the maintainer needs to assess and fix a vulnerability.
body:
  - type: markdown
    attributes:
      value: |
        Thank you for reporting privately. Please don't disclose this publicly until a fix is released.
  - type: textarea
    id: summary
    attributes:
      label: Summary
      description: What is the vulnerability, and how severe do you think it is?
    validations:
      required: true
  - type: input
    id: affected
    attributes:
      label: Affected package and versions
      placeholder: "package-name 1.2.3"
    validations:
      required: true
  - type: textarea
    id: proof_of_concept
    attributes:
      label: Proof of concept
      description: Exact configuration, code, or steps that reproduce the issue.
    validations:
      required: true
      min_length: 100
  - type: textarea
    id: impact
    attributes:
      label: Impact
      description: Who is affected, and what can an attacker achieve?
    validations:
      required: true
```

### 4.5 `SUPPORT.md`

- **Where GitHub shows it:** as a link when someone creates an issue.
- **Contents:** where to ask questions (for example, Discussions), what doesn't belong in issues, and realistic response expectations. "Best effort" is honest for a solo maintainer.

### 4.6 `ACCESSIBILITY.md`

- **Where GitHub shows it:** an **Accessibility** tab on the repository overview, linked from the About section.
- **Location:** `.github/`, the root, or `docs/`, searched in that order.
- **Suggested contents** (from GitHub's docs): accessibility priorities, supported environments, known barriers, what you expect from contributors, how to report barriers, who owns accessibility, and how it's maintained.
- **Inheritance:** the default-files page lists `ACCESSIBILITY.md` for both organizations and personal accounts. The accessibility page itself describes inheritance only from an organization's `.github` repository. Check that it works on a personal account before relying on it.
- **Relevance:** highest for projects with a UI or a documentation site. For configuration packages, a short statement is enough, or you can skip it.

### 4.7 `FUNDING.yml`

- **Location:** `.github/FUNDING.yml` on the default branch. Repository admins can toggle the button under **Settings → General → Features → Sponsorships**, so check that toggle in each repository where you expect the button.
- **Platform keys:** `github`, `patreon`, `open_collective`, `ko_fi`, `tidelift` (`npm/PACKAGE`), `community_bridge`, `liberapay`, `issuehunt`, `polar`, `buy_me_a_coffee`, `thanks_dev` (`u/gh/USERNAME`), and `custom`.
- **Limits:** one entry per external platform and up to four custom URLs. GitHub Sponsors takes one organization plus up to four developers.
- **Quoting:** custom URLs inside an array must be quoted if they contain `:`.
- **Policy:** funding links are for direct financial support of the project only, not for advertising or for political, community, or charity causes.

```yaml
# .github/FUNDING.yml  (only list platforms you actually use)
github: [cravingmaker]
# custom: ["https://example.com/donate"]
```

### 4.8 Issue templates and issue forms

**Folder and file rules**

- Templates live in `.github/ISSUE_TEMPLATE/` on the **default branch**.
- Markdown templates use `.md`, and forms use `.yml`. Filenames are case-insensitive.
- The chooser lists YAML forms before Markdown templates, each group sorted alphanumerically. To control the order, prefix filenames with numbers (`01-bug.yml`, `02-feature.yml`, and so on). Use a leading zero once you have 10 or more templates.
- A template's `name` must be **more than 3 characters**, or the template isn't shown. Names must also be unique across all templates.
- To earn a checkmark in the community profile, Markdown templates need `name` and `about`, and forms need `name` and `description`.

**Issue form keys (preview)**

- **Top level, required:** `name`, `description`, `body`. **Optional:** `title`, `labels`, `assignees`, `type`, `projects`.
- **Body elements:** `markdown` (display only; it isn't submitted), `input`, `textarea`, `dropdown`, `checkboxes`, and `upload`.
- **Validations:** `required` is **enforced only in public repositories**. `min_length` works on `input` and `textarea`, and `accept` lists allowed extensions for `upload`.
- **`id`:** must be unique and contain only letters, numbers, `-`, and `_`. It's also the key for pre-filling a field through URL query parameters.
- **`textarea.render: <language>`:** formats the answer as a code block, which turns off attachments for that field.
- **`dropdown`:** supports `multiple` and `default` (an index). If you set a default, don't include "None" or "n/a" as options.
- **`upload` limits:** images (`.png`, `.jpg`, `.jpeg`, `.gif`, `.svg`, `.webp`) up to 10 MB; videos (`.mp4`, `.mov`, `.webm`) up to 100 MB; archives (`.zip`, `.gz`, `.tar.gz`), documents (`.pdf`, `.docx`, `.xlsx`, `.pptx`), and text (`.json`, `.py`, `.js`, `.ts`, `.log`, `.txt`, `.csv`) up to 25 MB.

**Gotchas**

- Labels in `labels:` must already exist in the target repository. GitHub's default labels (`bug`, `documentation`, `duplicate`, `enhancement`, `good first issue`, `help wanted`, `invalid`, `question`, `wontfix`) exist in most repositories, so they're the safest choice in shared defaults.
- `type:` assigns an organization-level issue type. Personal accounts don't have issue types, so leave it out there.
- `projects:` only works when the person opening the issue has write access to the project. For outside contributors, use the project's auto-add workflow instead.
- **The chooser (`config.yml`):** `blank_issues_enabled: false` hides blank issues from Read and Triage users. Write, Maintain, and Admin users still see "Blank issue" labeled **Maintainers only**. `contact_links` (each with `name`, `url`, and `about`) send people elsewhere.

```yaml
# .github/ISSUE_TEMPLATE/config.yml  (account-wide default; assumes the default branch is main)
blank_issues_enabled: true
contact_links:
  - name: Questions and help
    url: https://github.com/cravingmaker/.github/blob/main/SUPPORT.md
    about: Where to ask questions and get help.
  - name: Security vulnerabilities
    url: https://github.com/cravingmaker/.github/blob/main/SECURITY.md
    about: Please report vulnerabilities privately; see the security policy.
```

```yaml
# .github/ISSUE_TEMPLATE/01-bug-report.yml  (generic default)
name: Bug report
description: Report something that doesn't work as documented.
labels: ["bug"]
body:
  - type: markdown
    attributes:
      value: |
        Thanks for reporting! Please search existing issues first.
        For security problems, don't use this form; follow the security policy instead.
  - type: input
    id: version
    attributes:
      label: Package version
      placeholder: "1.2.3"
    validations:
      required: true
  - type: textarea
    id: what-happened
    attributes:
      label: What happened?
      description: What did you expect, and what happened instead?
    validations:
      required: true
      min_length: 20
  - type: textarea
    id: reproduction
    attributes:
      label: Steps to reproduce
      description: A minimal configuration or repository that shows the problem.
    validations:
      required: true
  - type: textarea
    id: logs
    attributes:
      label: Relevant output
      render: shell
  - type: checkboxes
    id: conduct
    attributes:
      label: Code of Conduct
      options:
        - label: I agree to follow this project's Code of Conduct.
          required: true
```

### 4.9 Pull request templates

- **A single template** goes in `pull_request_template.md` in `.github/`, the root, or `docs/`. It pre-fills every new PR.
- **Multiple templates** go in a `PULL_REQUEST_TEMPLATE/` subfolder of any of those locations. There's no picker in the UI: a template from the folder is used only when the URL names it, as in `.../compare/main...branch?quick_pull=1&template=release.md`. Keep a single default file as well if you want PRs pre-filled.
- **Branch:** templates must be on the default branch.
- **Other query parameters:** `title`, `body`, `labels`, `milestone`, `assignees`, and `projects`. Each needs the matching permission.

```markdown
<!-- .github/pull_request_template.md  (generic default) -->
## Summary

<!-- What does this change, and why? Link related issues (e.g. "Closes #123"). -->

## Validation

<!-- How did you test it? List commands and results. -->

- [ ] I read the contributing guidelines.
- [ ] Tests and docs are updated where relevant.
```

### 4.10 Discussion category forms

- **Location:** `.github/DISCUSSION_TEMPLATE/<category-slug>.yml`. The filename must match a category's slug; the "Announcements" category, for example, uses `announcements.yml`.
- **Keys:** `body` is required and needs at least one element that isn't `markdown`. `title` and `labels` are optional. The form uses the same schema as issue forms.
- **Limits:** forms aren't supported for polls. Creating one needs write access, and the file must be on the default branch.
- **Organization Discussions** use a **source repository** that you choose, and permissions follow that repository. Changing the source later doesn't move existing discussions.

### 4.11 README

- **Precedence:** `.github/` → root → `docs/`. A `.github/README.md` **replaces** the root README on the repository's page, which surprises people.
- **Size:** content beyond 500 KiB is truncated.
- **Links:** use relative links to other files in the same repository.
- **Inheritance:** a README is never inherited. In the `.github` repository, the root `README.md` is just that repository's own page.

### 4.12 `CODEOWNERS`

- **Location:** `.github/`, the root, or `docs/`. GitHub uses the first one it finds. Each branch has its own file, and review requests use the file on the PR's **base** branch; this also applies to PRs from forks.
- **Syntax:** mostly gitignore-style. These gitignore features **don't** work: `!` negation, `[ ]` character ranges, and escaping a leading `#` with `\`.
- **Matching:** the **last matching line wins.** List multiple owners for one pattern on the same line; otherwise only the last line's owners count.
- **Owners:** each must have explicit **write** access. Teams must also be visible and hold write access themselves. Email addresses work, except for managed user accounts. Paths are **case-sensitive**, even if you develop on macOS.
- **Errors:** invalid lines are skipped and highlighted when you view the file (also available through the API). Owners that don't exist or lack access aren't assigned. A file over **3 MB** isn't loaded at all.
- **Enforcement:** "Require review from Code Owners" (in branch protection or a ruleset) is satisfied by an approval from **any one** of the matching owners. Also give `CODEOWNERS` itself an owner, for example `/.github/` or `/.github/CODEOWNERS`.
- **Dependabot:** the `reviewers` option was **removed** from `dependabot.yml` (changelog, 2025-08-08). Dependabot PRs now get reviewers through `CODEOWNERS`.
- **Solo maintainers:** you can't approve your own PR. If you're the only code owner and require code-owner review, you'll need a bypass, such as the admin role in a ruleset's bypass list, to merge your own changes.

```text
# CODEOWNERS (organization-style example; the last match wins)
*                      @my-org/maintainers
/.github/              @my-org/platform
/.github/workflows/    @my-org/platform @my-org/security
/.github/CODEOWNERS    @my-org/admins
```

For a personal account, `* @cravingmaker` already covers everything.

### 4.13 `dependabot.yml`

- **Location:** `.github/dependabot.yml` or `.github/dependabot.yaml`, on the default branch. It's never inherited.
- **Required keys:** `version: 2` and `updates`. Each entry under `updates` needs `package-ecosystem`, `directory` or `directories`, and `schedule.interval`.
- **GitHub Actions:** for the `github-actions` ecosystem, `directory: "/"` scans `.github/workflows/` and a root `action.yml`. Actions stored elsewhere, such as `.github/actions/*`, need extra entries under `directories`, which accepts globs (`directory` doesn't).
- **Schedules:** `daily`, `weekly`, `monthly`, `quarterly`, `semiannually`, `yearly`, or `cron` (with a `cronjob` expression; put the timezone in `timezone`).
- **Cooldown:** version updates now have a **default 3-day cooldown**, so a new release isn't proposed until it's 3 days old. `cooldown` tunes this with `default-days`, `semver-major-days`, `semver-minor-days`, `semver-patch-days`, `include`, and `exclude`. It doesn't apply to security updates.
- **Other useful keys:** `groups`, `multi-ecosystem-groups`, `ignore`, `allow`, `exclude-paths`, `labels`, `assignees`, `milestone`, `commit-message`, `open-pull-requests-limit`, `target-branch`, `versioning-strategy`, and `registries`.
- **Alerts:** Dependabot alerts are configured in settings, not in this file.

```yaml
# .github/dependabot.yml for the .github repository itself (only useful if it has workflows)
version: 2
updates:
  - package-ecosystem: "github-actions"
    directories:
      - "/"                    # .github/workflows/ and a root action.yml
      # - "/.github/actions/*" # uncomment if you add local actions
    schedule:
      interval: "weekly"
    cooldown:
      default-days: 7
    groups:
      github-actions:
        patterns: ["*"]
```

### 4.14 `release.yml`

`.github/release.yml` shapes automatically generated release notes. The keys are `changelog.exclude.labels`, `changelog.exclude.authors`, and `changelog.categories[]`; each category needs a `title` and `labels`, and can set its own `exclude`. Use `"*"` as a catch-all category. This only matters if you use **Generate release notes**. Tools like release-please write their own notes.

```yaml
changelog:
  exclude:
    labels: ["ignore-for-release"]
  categories:
    - title: Breaking changes
      labels: ["breaking-change"]
    - title: Features
      labels: ["enhancement"]
    - title: Fixes
      labels: ["bug"]
    - title: Dependencies
      labels: ["dependencies"]   # Dependabot applies this label by default
    - title: Other changes
      labels: ["*"]
```

### 4.15 `secret_scanning.yml`

- **Purpose:** `.github/secret_scanning.yml` with `paths-ignore:` closes alerts in matching paths as "ignored by configuration" and also excludes those paths from push protection.
- **Patterns:** use the same filter patterns as Actions.
- **Limits:** only the first 1,000 entries are used, and a file over 1 MB is ignored entirely.
- **Best practice:** keep exclusions minimal and precise, put a comment on each explaining why, and review them regularly.

```yaml
# Test fixtures contain fake, revoked keys (see tests/fixtures/README.md)
paths-ignore:
  - "tests/fixtures/**"
```

### 4.16 CodeQL model packs: `.github/codeql/extensions/` (preview)

Copy a model pack directory, containing a `codeql-pack.yml` and its model `.yml` files, into `.github/codeql/extensions/`. Code scanning's default setup detects and uses it automatically, and advanced setup keeps recognizing it. Supported languages are C/C++, C#, Java/Kotlin, Python, Ruby, and Rust. At organization scale, you publish packs to the Container registry and list them in the org's code scanning settings instead.

### 4.17 Workflows and actions

**Where they live**

- Workflows must be in `.github/workflows/` and end in `.yml` or `.yaml`.
- **Reusable workflows** (`on: workflow_call`) live there too. **Subfolders aren't supported.**

**Calling reusable workflows**

- A caller references one as `OWNER/REPO/.github/workflows/FILE.yml@REF`. For a workflow hosted in the `.github` repository, that becomes `cravingmaker/.github/.github/workflows/ci.yml@<sha>`.
- **Same-repository** calls can use `./.github/workflows/FILE.yml` or the new `$/.github/workflows/FILE.yml` (see below).
- **Visibility:** public callers can only use public reusable workflows. Private callers can use public ones and private ones whose **Access** setting allows it.
- **Limits:** up to 10 levels of nesting, and up to 50 unique reusable workflows called from one workflow file (nested calls included).
- **Environment and permissions:** workflow-level `env` isn't propagated in either direction. A called workflow can only **lower** the `GITHUB_TOKEN` permissions it receives.
- **Renames break callers:** Actions doesn't follow redirects for actions or reusable workflows, so renaming or transferring the host repository breaks every caller.

**Local actions**

- `.github/actions/<name>/action.yml` is a common convention.
- Reference such an action as `$/.github/actions/<name>`. The `$/` prefix means "this repository, at the running commit": no checkout is needed, and the path is pinned to that commit. It isn't available on GitHub Enterprise Server.
- The older `./.github/actions/<name>` form needs a checkout first, and it resolves against the runner's workspace.

**YAML anchors and aliases** (`&name`, `*name`) are supported in workflows.

**Lockfile (preview, 2026-10-14)**

- `gh actions-lock` (a GitHub CLI extension) writes `.github/workflows/actions.lock`, which pins **every** action, including transitive ones, to a verified commit and records the source repository's IDs.
- Workflows covered by the lockfile no longer need SHA pins in their `uses:` lines. Dependabot regenerates entries when it bumps a locked action.
- A **Require lockfile** policy can enforce its use.
- **Watch out:** allowlist entries pinned to tags (like `actions/checkout@v6`) stop matching locked workflows. Use wildcards or full SHAs in allowlists instead.
- **Availability** (per the changelog): Free, Team, and Enterprise Cloud plans.
- **Never edit the lockfile by hand.** Review changes to it the way you'd review dependency bumps.

**Who can change workflow files**

- An OAuth token or classic PAT needs the **`workflow`** scope to add or update workflow files. A GitHub App needs the **Workflows** permission.
- `GITHUB_TOKEN` has no workflows permission at all, so a workflow can't push changes to `.github/workflows/` using it.

For security hardening, see [5.3](#53-security-hardening-of-github).

### 4.18 Workflow templates (organizations)

- **Layout:** in the organization's `.github` repository, add `workflow-templates/NAME.yml`, a matching `NAME.properties.json`, and optionally an SVG icon.
- **Metadata keys:** `name` and `description` are required. `iconName` is optional and is either an SVG in the folder (the filename without its extension) or `octicon <name>`. `categories` is optional and uses starter-workflow categories, Linguist languages, or tech stacks. `filePatterns` is optional: regular expressions matched against files in the repository's root.
- **Placeholder:** `$default-branch` is replaced with the repository's default branch.
- **Visibility:** templates in a **public** `.github` repository are available to every repository type. Templates in an **internal** one are available only to internal and private repositories, and in a **private** one only to private repositories. With a private or internal `.github` repository, users also need read access to it.

```yaml
# workflow-templates/node-ci.yml
name: Node CI
on:
  push:
    branches: [ $default-branch ]
  pull_request:
    branches: [ $default-branch ]
permissions:
  contents: read
jobs:
  ci:
    uses: my-org/.github/.github/workflows/node-ci.yml@v1
```

```json
{
  "name": "Node CI (org standard)",
  "description": "Calls the organization's reusable Node.js CI workflow.",
  "iconName": "octicon package",
  "categories": ["JavaScript", "TypeScript"],
  "filePatterns": ["package.json$"]
}
```

### 4.19 Organization profile README

- **Public view:** `profile/README.md` in the public `.github` repository.
- **Member-only view:** `profile/README.md` in a **private** `.github-private` repository.
- **Which view members see by default:** the member view, if any members-only README or pins exist; otherwise the public view. Members can switch between the two, and everyone else always sees the public view.
- **Personal accounts:** use `USERNAME/USERNAME`. It must be public, with a non-empty `README.md` at its root.

### 4.20 Copilot and AI customization files

| File | Read by | Key rules |
|---|---|---|
| `.github/copilot-instructions.md` | Copilot Chat, cloud agent, and code review on GitHub.com; IDEs; CLI | Natural-language Markdown. Applies to every request in the repository. Keep it short and stable. |
| `.github/instructions/**/*.instructions.md` | Cloud agent and code review on GitHub.com; most IDEs; CLI | Front matter `applyTo: "glob,glob"`. Optional `excludeAgent: "code-review"` or `"cloud-agent"`. |
| `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` | Cloud agent, code review, CLI; VS Code reads `AGENTS.md` | Any number of `AGENTS.md` files, and the nearest one in the tree wins. `CLAUDE.md` and `GEMINI.md` are single files at the root. |
| `REVIEW.md` | Copilot code review | Extra review guidance. |
| `.github/prompts/*.prompt.md` | VS Code, Visual Studio, JetBrains, Xcode | Run on demand from chat. Not used on GitHub.com or in the CLI. |
| `.github/agents/NAME.agent.md` | Cloud agent, CLI, IDEs, the GitHub Copilot app | `description` is required. Optional: `name`, `tools`, `mcp-servers`, `target` (`vscode` or `github-copilot`), `model` (IDEs), `disable-model-invocation`, `user-invocable` (`infer` is retired). The prompt is limited to 30,000 characters. Filenames may contain only `.`, `-`, `_`, letters, and digits. Versions are tracked by commit SHA. |
| `.github/skills/NAME/SKILL.md` (or `.claude/skills/`, `.agents/skills/`) | Cloud agent, code review, CLI, the GitHub Copilot app, and agent mode in VS Code, Visual Studio, and JetBrains | Front matter: `name` (lowercase with hyphens) and `description` are required; `license` and `allowed-tools` are optional. Scripts in the same folder can be used. |
| `.github/hooks/*.json` | Cloud agent (a subset of events), CLI, the GitHub Copilot app | `{"version": 1, "hooks": {...}}` with events such as `sessionStart`, `preToolUse` (which can **deny** a tool call), and `postToolUse`. Default timeout is 30 s. **Must be on the default branch** for the cloud agent. |
| `.github/workflows/copilot-setup-steps.yml` | Cloud agent (and code review unless overridden) | Needs a single job named **`copilot-setup-steps`**. Only `steps`, `permissions`, `runs-on`, `services`, `snapshot`, and `timeout-minutes` (max 59) take effect. Must be on the default branch. Runs on Ubuntu x64 or Windows 64-bit runners only. |
| `.github/workflows/copilot-code-review.yml` | Copilot code review | Same format. When present, it replaces the setup-steps file for code review. |
| `.github/workflows/NAME.md` + `NAME.lock.yml` | GitHub Agentic Workflows **(preview)** | Natural-language workflow. Compile it with `gh aw compile` and commit both files. Read-only by default; writes happen only through declared `safe-outputs`. |

**Precedence and scope**

- Personal instructions take priority over repository instructions, which take priority over organization instructions. Copilot receives all of them, though.
- For custom agents, the repository's agent overrides the organization's, which overrides the enterprise's.

**Important caveat:** when Copilot reviews a pull request, it reads instructions, agent files, and skills **from the PR's head branch**. A PR can therefore change the guidance used to review itself. Treat Copilot review as advisory, not as a security gate.

Minimal examples:

```markdown
<!-- .github/instructions/tests.instructions.md -->
---
applyTo: "**/*.test.ts"
excludeAgent: "code-review"
---
Use Vitest. Prefer table-driven tests. Don't add snapshot tests for configuration objects.
```

```markdown
<!-- .github/agents/docs-writer.agent.md -->
---
name: docs-writer
description: Updates Markdown documentation to match code changes, following the project's style.
tools: ["read", "search", "edit"]
---
You are a documentation specialist. Only edit Markdown files. Keep examples runnable.
```

```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      { "type": "command", "bash": "./scripts/hooks/check-tool.sh", "timeoutSec": 10 }
    ]
  }
}
```

---

## 5. Best practices

### 5.1 Structure and naming

- Use GitHub's canonical names and cases (`CODE_OF_CONDUCT.md`, `CODEOWNERS`, `ISSUE_TEMPLATE`). Some lookups are case-insensitive, but `CODEOWNERS` paths aren't, and consistent names prevent surprises.
- Store each file in **one** location. Copies in `.github/`, the root, and `docs/` shadow each other in that order.
- Keep human-facing Markdown at the root, where it's discoverable, and machine configuration in `.github/`.
- Give issue templates numeric prefixes (`01-`, `02-`) so they appear in a deliberate order.
- Stick to one YAML extension. `.yml` is the one GitHub's docs use for templates.

### 5.2 Writing defaults that work in every repository

- **Be generic.** Say "this project", and never name a specific package.
- **Use absolute links** in defaults that point elsewhere. A relative link in a default file resolves inside the `.github` repository, and template text is copied into issues in *other* repositories.
- **Don't link per-repository pages from defaults.** Write instructions instead: "use **Report a vulnerability** on the repository's **Security** tab."
- **Use GitHub's default label names** (`bug`, `enhancement`, `documentation`, `question`, and so on) in default templates, so the labels exist almost everywhere.
- **Remember these limits:** `required` fields are enforced only in public repositories, `type:` needs organization issue types, and `projects:` needs the reporter to have write access.
- **Overriding issue templates is all or nothing.** When a repository needs one custom template, copy the **whole** default set into it and then adjust.

### 5.3 Security hardening of `.github/`

**Ownership and change control**

- Give `.github/` (and `CODEOWNERS` itself) explicit owners, and require code-owner review through a ruleset. Solo maintainers need a bypass for themselves.
- Protect the default branch with a ruleset: require a pull request, block force pushes, and require status checks.
- Know that "Restrict file paths" push rules apply only to private and internal repositories, so public repositories rely on code-owner review instead.

**Token permissions**

- Keep the default `GITHUB_TOKEN` **read-only**. That's already the default for new personal repositories.
- Declare `permissions:` in every workflow. Set `permissions: {}` at the top and grant the minimum each job needs.
- Keep **Allow GitHub Actions to create and approve pull requests** off unless you need it.

**Pinning third-party actions**

- Pin third-party actions to full-length commit SHAs, or adopt the lockfile (preview).
- Consider turning on **Require actions to be pinned to a full-length commit SHA** in the Actions settings.
- Let Dependabot (`github-actions` ecosystem) keep pins current, with a cooldown.

**Untrusted pull requests**

- Avoid `pull_request_target`. From 2026-11-02, GitHub's default policy blocks it in public repositories that have no explicit event policy.
- Never check out and *execute* PR code in a privileged workflow, whatever its trigger. The `allow-unsafe-pr-checkout: true` input exists for rare, reviewed exceptions.
- Treat artifacts that a `workflow_run` workflow receives from other runs as untrusted data.
- Set fork PR approvals to **Require approval for all external contributors**. The first-time-contributor options can be gamed with one trivial merged PR.

**Workflow code**

- Prevent script injection: pass untrusted context (`github.event.*.title`, `body`, branch names) through `env:` rather than interpolating it into `run:`.
- Use `persist-credentials: false` on `actions/checkout` unless a later step really needs to push.
- Set `timeout-minutes` and `concurrency` on jobs.
- Use **OIDC** instead of long-lived secrets for cloud providers and registries. npm trusted publishing is one example, and your publish workflows appear to use it (`id-token: write`, no npm token secret).
- Scan workflows with CodeQL (default setup covers GitHub Actions), zizmor, actionlint, or OpenSSF Scorecard.
- Keep secret scanning and push protection on. Make `secret_scanning.yml` exclusions narrow and commented.

**Automation and AI agents**

- Plan bots around token scopes. Automation that edits workflow files needs a GitHub App token with the **Workflows** permission (or a token with `workflow` scope). `GITHUB_TOKEN` can't do it.
- Treat AI agent files as code: protect `copilot-instructions.md`, `instructions/`, `agents/`, `skills/`, `hooks/`, and `AGENTS.md` with `CODEOWNERS`.
- Inspect third-party skills before installing them (`gh skill preview`). Don't pre-approve `shell` in `allowed-tools` unless you've reviewed the scripts.
- Review hook scripts the way you'd review workflow code.

**Publicity:** the `.github` repository is **public**. Never put secrets, private contact details, or internal information in it.

Example of a hardened workflow skeleton:

```yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]
permissions: {}                      # nothing by default
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    permissions:
      contents: read                 # grant per job, minimally
    steps:
      - uses: actions/checkout@<full-length-sha> # vX.Y.Z
        with:
          persist-credentials: false
      - uses: actions/setup-node@<full-length-sha> # vX.Y.Z
        with:
          node-version: 24
      - run: npm ci --ignore-scripts
      - run: npm test
```

Script injection, wrong and right:

```yaml
# Unsafe: attacker-controlled text is pasted into the shell script
- run: echo "Issue title is ${{ github.event.issue.title }}"

# Safe: the text reaches the script as data, through an environment variable
- env:
    TITLE: ${{ github.event.issue.title }}
  run: echo "Issue title is $TITLE"
```

### 5.4 Maintenance

- **Test defaults** in a throwaway public repository that has no files of its own. Check the issue chooser, the Contributing link, the Security policy, and the private-report form.
- **Validate YAML before merging.** The issue-form editor and the `CODEOWNERS` view both highlight errors, and GitHub's "Common validation errors when creating issue forms" page lists frequent mistakes.
- **Lint automatically:** actionlint and zizmor for workflows, a YAML linter or Prettier for templates, and markdownlint for documents.
- **Review defaults about every quarter.** Keep the supported versions in `SECURITY.md` current and remove stale `paths-ignore` entries.
- **Watch the GitHub Changelog.** Several features covered here are in preview: issue forms, the form schema, custom agents, lockfiles, CodeQL model packs, and agentic workflows.

### 5.5 AI customization hygiene

- Put short, always-true rules in `copilot-instructions.md`, such as build and test commands, layout, and conventions. GitHub's own generator prompt caps it at about two pages.
- Move task-specific procedures into **skills**, which load only when relevant, or into **custom agents**, which define a role with restricted tools.
- If you use several AI tools, `AGENTS.md` is the most portable choice. Copilot also reads a root `CLAUDE.md` or `GEMINI.md`.
- Avoid conflicting instructions across the personal, repository, and organization levels, because Copilot receives all of them.

---

## 6. Decision guide

### 6.1 The decisions to make

**D1. Which account owns the `.github` repository: your personal account or an organization?**

- *Personal* (`cravingmaker/.github`): community-health defaults for all your repositories. Simple, and you're the only owner.
- *Organization:* also gives you an org profile README, workflow templates, teams in `CODEOWNERS`, org rulesets, issue types, default labels, org Discussions, and org-wide Copilot instructions and agents (with a Copilot plan).
- **The cost of moving repositories into an organization later:**
  - GitHub redirects web and git URLs, but **Actions doesn't follow redirects** for `uses:` references to actions or reusable workflows.
  - Your publish workflows appear to use npm trusted publishing (`id-token: write`, no npm token secret). Trusted publishing is configured against a specific GitHub owner, repository, and workflow file, so each package would need reconfiguring.
  - README badges and links need updating.
- **Recommendation:** start with a personal `.github`. Revisit if you add co-maintainers or need organization-only features. An organization's `.github` would cover only that organization's repositories.

**D2. What should be centralized, and what should stay in each repository?** See the matrix in [6.3](#63-per-file-matrix-for-cravingmaker).

**D3. Should there be default issue templates?**

- *A generic default set* (bug report, feature request, and `config.yml`) for every repository without its own. Repositories that need tailored forms keep a complete set of their own.
- *No defaults:* each repository decides for itself.
- **Recommendation:** the generic set. Remember that a repository with *any* template file of its own gets none of the defaults.

**D4. Should blank issues be allowed in the default chooser?**

- `true`: an escape hatch for reports that don't fit a generic form.
- `false`: everyone must use a form, though Write+ users still see "Blank issue (Maintainers only)".
- **Recommendation:** `false` once the default forms cover bugs, features, documentation, and questions, which is the setup this repository uses. With only one or two generic forms, keep `true` as an escape hatch.

**D5. How should people report vulnerabilities?**

- **Recommendation:** all of the following.
  - Enable private vulnerability reporting in every public repository (a setting).
  - Provide a generic default `SECURITY.md`, for example "only the latest major version receives fixes".
  - Provide a default `.github/VULNERABILITY_REPORT.yml`.
  - Optionally turn on "Require a CWE assignment" per repository.
- Keep a per-repository `SECURITY.md` only where the policy really differs.

**D6. Should there be a Sponsor button?** Add `.github/FUNDING.yml` only if you have a funding channel. One default covers every repository, and a repository's own `FUNDING.yml` overrides it.

**D7. Where should shared CI live?**

- *(a) Copies in each repository* (what you do today). Simple and independent, and Dependabot updates each copy.
- *(b) Reusable workflows in `cravingmaker/.github`.* One well-known place, but CI history gets mixed with community documents, and tags version the whole repository.
- *(c) A dedicated public repository* (for example, `cravingmaker/workflows`). Clean history, independent tags and releases, and its own `CODEOWNERS` and rulesets.
- **Constraints for (b) and (c):**
  - Reusable workflows must sit directly in `.github/workflows/`.
  - Callers should pin `@<sha>` (Dependabot can bump the pin).
  - `env` isn't propagated, and permissions can only be lowered.
  - Before centralizing publishing, check that npm trusted publishing still matches.
- **Recommendation:** stay with (a) for now. Move to (c) once several repositories share identical jobs.

**D8. Where should Copilot and AI agent files live?**

- Personal accounts have no account-wide mechanism, so these files go in each repository. Personal Copilot instructions (in your settings) apply only to your own chats. Use `AGENTS.md` for portability across tools.
- **Recommendation:** keep them per repository, protected by `CODEOWNERS`. You could keep starter copies in an ordinary folder of the `.github` repository, such as `templates/`. GitHub doesn't treat that folder specially.

**D9. Where should the Markdown defaults sit inside the `.github` repository?** At the root (visible), in `.github/` (tidier root), or in `docs/`. **Recommendation:** the root, with one copy per file.

**D10. Should the `.github` repository have its own license?** **Recommendation:** yes. Use the same license as your projects, or CC BY 4.0 for prose. A file adapted from Contributor Covenant 3.0 must keep its attribution and stay under CC BY-SA 4.0.

### 6.2 Where should a file live? (decision tree)

```text
1. Is it on GitHub's default-file list?
   (CODE_OF_CONDUCT, CONTRIBUTING, SECURITY, SUPPORT, ACCESSIBILITY, FUNDING.yml,
    VULNERABILITY_REPORT.yml, issue templates + config.yml, PR templates, discussion forms)
   ├─ No  → Keep it in each repository (share logic with reusable workflows or a sync tool).
   └─ Yes → go to 2.
2. Must it ship inside clones, tarballs, or packages?
   ├─ Yes → Keep it in each repository.
   └─ No  → go to 3.
3. Can one wording serve every repository without naming a specific one?
   ├─ No  → Keep it per repository (optionally also add a generic default for the rest).
   └─ Yes → go to 4.
4. Is it an issue template, and do some repositories need different templates?
   ├─ Yes → Generic default set + COMPLETE per-repository sets where needed.
   └─ No  → Put it in the .github repository; remove per-repository copies you don't need.
```

### 6.3 Per-file matrix for cravingmaker

| Item | Recommendation | Reason |
|---|---|---|
| `CODE_OF_CONDUCT.md` | **Centralize** (project-neutral wording) | Your two copies differ only by package name. |
| `SECURITY.md` | **Centralize** a generic policy; override only where a policy really differs | The current differences are repository URLs and tool names, which generic wording replaces. |
| `.github/VULNERABILITY_REPORT.yml` | **Centralize** | It applies to every public repository with private vulnerability reporting enabled. |
| `SUPPORT.md` | **Centralize** | The same support channels apply everywhere. |
| `CONTRIBUTING.md` | Short **generic default**, and keep the tailored ones | `eslint-config` (1,122 lines) and `prettier-config` (330 lines) are highly specific. |
| Issue templates and `config.yml` | **Generic default set** | `prettier-config` keeps its own set (so it gets no defaults); `eslint-config` and the others inherit. |
| PR template | **Generic default**; keep tailored ones | Defaults fall back per file type, so each repository can override. |
| `.github/FUNDING.yml` | Only if you have a funding channel | No point otherwise. |
| `ACCESSIBILITY.md` | Optional | Most relevant to projects with a UI or docs. |
| Discussion forms | Only where Discussions are enabled | No point otherwise. |
| `LICENSE` | **Per repository** | It can't be a default. |
| `CODEOWNERS`, `dependabot.yml`, workflows, `release.yml` | **Per repository** | They aren't inherited. Reduce duplication with reusable workflows later (D7). |
| Copilot and AI files | **Per repository** | Personal accounts have no account-wide mechanism. |

### 6.4 Pitfalls and troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Default issue templates don't appear in a repository | The repository has its own `.github/ISSUE_TEMPLATE/` contents, even just `config.yml` | Remove them, or copy in the full set |
| A template doesn't appear at all | `name` is 3 characters or fewer, the YAML is invalid, or the file isn't on the default branch | Fix the file and check it in the template editor |
| Labels aren't applied | The label doesn't exist in the target repository | Create it, or use a default label name |
| No defaults apply anywhere | The `.github` repository is private or misnamed, or files are in the wrong folders | Make it public, rename it to exactly `.github`, and check paths (`.github/.github/...`) |
| The Sponsor button is missing | `FUNDING.yml` isn't in `.github/`, or the repository's Sponsorships toggle is off | Move the file, or turn on the toggle |
| A workflow in `.github` doesn't run for other repositories | Working as designed | Use reusable workflows |
| "Reusable workflow not found" | It's in a subfolder, has the wrong path or ref, or is private without Access settings | Use `OWNER/.github/.github/workflows/FILE.yml@REF` and check Access |
| A push that changes workflows is rejected | The token lacks the `workflow` scope or Workflows permission | Use a properly scoped token or GitHub App |
| `CODEOWNERS` doesn't request reviews | The file isn't on the base branch, an owner lacks write access, there are syntax errors, or the file is over 3 MB | Check the errors shown in the file view |
| You can't merge your own PR | Code-owner review is required and you're the only owner | Add yourself (admin) to the ruleset's bypass list |
| Required form fields aren't enforced | The repository is private | Expected: `required` works only in public repositories |
| The custom vulnerability form isn't used | It failed validation, so GitHub falls back to the default | Fix the YAML, and keep it to supported elements (no `upload`) |
| Copilot cloud agent hooks don't run | The file isn't on the default branch, the JSON is invalid, `version: 1` is missing, or the script isn't executable | See the hooks troubleshooting steps |
| `pull_request_target` runs fail after 2026-11-02 | GitHub's default event policy | Switch to `pull_request`, or explicitly allow the event with a policy after hardening |

---

## 7. Applying this to cravingmaker

### 7.1 Current state (observed 2026-10-22)

- **Account:** `cravingmaker` is a **personal (User) account** with 13 public repositories. The GitHub API returned *Not Found* for `cravingmaker/.github`, so the repository doesn't exist yet, or at least isn't public.
- **Local checkouts:** `eslint-config` and `prettier-config` each have:
  - their own `CODE_OF_CONDUCT.md` (identical except for the package name), `CONTRIBUTING.md` (tailored), `SECURITY.md` (repository-specific links), and `LICENSE`;
  - the same `.github/CODEOWNERS` (`* @cravingmaker`);
  - a `dependabot.yml` covering `npm` and `github-actions`, with cooldowns;
  - workflows that pin actions to full SHAs, use a GitHub App token for release-please, and appear to publish through npm trusted publishing (`id-token: write`, no npm token secret).
- **Templates:** `prettier-config` also has its own issue forms, `config.yml`, and PR template. `eslint-config` has none, so it will inherit the defaults.
- **The `pull_request_target` deadline:** `prettier-config`'s `dependabot-effect-reference.yml` uses `workflow_run`, guarded so it only runs for same-repository PRs and only uses trusted code from the default branch. No `pull_request_target` workflows were found in either checkout, so the 2026-11-02 enforcement doesn't affect them. The other public repositories weren't checked.

### 7.2 Proposed initial layout for this directory

```text
Public/.github/                      → becomes github.com/cravingmaker/.github (public)
├── README.md                        what this repository is; which defaults it provides
├── LICENSE                          license for this repository's own content
├── CODE_OF_CONDUCT.md               default (project-neutral)
├── CONTRIBUTING.md                  short generic default
├── SECURITY.md                      generic policy + "Report a vulnerability" instructions
├── SUPPORT.md                       where to get help
└── .github/
    ├── VULNERABILITY_REPORT.yml     default private-report form
    ├── pull_request_template.md     generic default
    ├── ISSUE_TEMPLATE/
    │   ├── config.yml
    │   ├── 01-bug-report.yml
    │   └── 02-feature-request.yml
    └── CODEOWNERS                   * @cravingmaker (protects this repository)
```

Optional later additions: `ACCESSIBILITY.md`, `.github/FUNDING.yml`, `.github/DISCUSSION_TEMPLATE/`, and `.github/workflows/` with a `dependabot.yml`, if you add Markdown or YAML linting to this repository.

### 7.3 Rollout plan

1. Confirm D1 (personal account) and D9 (Markdown at the root).
2. Write the generic files, using sections 4.1–4.9 as a guide. Remove repository-specific names and URLs.
3. Create `cravingmaker/.github` as a **public** repository and push to its default branch.
4. Verify in a throwaway public repository with no files of its own. Check the issue chooser, the Contributing and Security policy links, the PR body, and, with private vulnerability reporting enabled, the report form.
5. Decide whether to delete the per-repository duplicates that would otherwise override the defaults: `CODE_OF_CONDUCT.md` in both repositories, and `SECURITY.md` if the generic policy fits.
6. Make sure private vulnerability reporting is enabled in every public repository.
7. Revisit D7 (shared CI) once several repositories share identical jobs.

---

## 8. Recent changes to watch

| Date | Change | Why it matters here |
|---|---|---|
| 2025-08-08 | The Dependabot `reviewers` option was removed | Use `CODEOWNERS` to route Dependabot PRs. |
| 2026-10-01 | Structured forms for private vulnerability reports | New default-able file: `.github/VULNERABILITY_REPORT.yml`. |
| 2026-10-14 | Workflow dependency lockfiles **(preview)** | `.github/workflows/actions.lock`, `gh actions-lock`, and a **Require lockfile** policy. |
| **2026-11-02** | The default policy blocking `pull_request_target` in public repositories is enforced | Audit any `pull_request_target` workflows before this date. |
| Current docs (exact dates not verified) | `ACCESSIBILITY.md` default and Accessibility tab. Issue-form `upload` element and `min_length`. Copilot "coding agent" renamed **"cloud agent"**. `.github/agents/`, `.github/skills/`, and `.github/hooks/`. `REVIEW.md` and `copilot-code-review.yml`. Org agents in `agents/`. The `$/` self-reference syntax. Dependabot's default 3-day cooldown and its quarterly, semiannual, yearly, and cron schedules. `GOVERNANCE.md` dropped from the GitHub.com default list. The Security tab relabeled "Security and quality". | The file locations and limits in this guide reflect these. |

---

## 9. Sources

Official documentation (docs.github.com, fetched 2026-10-22):

- [Creating a default community health file](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)
- [About community profiles for public repositories](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/about-community-profiles-for-public-repositories)
- [Setting guidelines for repository contributors](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/setting-guidelines-for-repository-contributors)
- [Adding a code of conduct to your project](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/adding-a-code-of-conduct-to-your-project)
- [Adding support resources to your project](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/adding-support-resources-to-your-project)
- [Adding an accessibility page to your repository](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/adding-an-accessibility-page-to-your-repository)
- [Adding a security policy to your repository](https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/configure-vulnerability-reporting/add-security-policy)
- [Configuring private vulnerability reporting for a repository](https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/configure-vulnerability-reporting/configure-for-a-repository)
- [About issue and pull request templates](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/about-issue-and-pull-request-templates)
- [Configuring issue templates for your repository](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository)
- [Syntax for issue forms](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms)
- [Syntax for GitHub's form schema](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-githubs-form-schema)
- [Creating a pull request template for your repository](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository)
- [Using query parameters to create a pull request](https://docs.github.com/en/pull-requests/reference/using-query-parameters-to-create-a-pull-request)
- [Creating discussion category forms](https://docs.github.com/en/discussions/managing-discussions-for-your-community/creating-discussion-category-forms) and [Syntax for discussion category forms](https://docs.github.com/en/discussions/managing-discussions-for-your-community/syntax-for-discussion-category-forms)
- [Enabling or disabling GitHub Discussions for an organization](https://docs.github.com/en/organizations/managing-organization-settings/enabling-or-disabling-github-discussions-for-an-organization)
- [About the repository README file](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)
- [Managing your profile README](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/managing-your-profile-readme)
- [Customizing your organization's profile](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/customizing-your-organizations-profile)
- [Displaying a sponsor button in your repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/displaying-a-sponsor-button-in-your-repository)
- [About CITATION files](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-citation-files)
- [About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [Available rules for rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
- [Automatically generated release notes](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes)
- [About the dependabot.yml file](https://docs.github.com/en/code-security/concepts/supply-chain-security/about-the-dependabot-yml-file) and [Dependabot options reference](https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference)
- [Excluding folders and files from secret scanning](https://docs.github.com/en/code-security/how-tos/secure-your-secrets/customize-leak-detection/exclude-folders-and-files)
- [Editing your configuration of default setup (CodeQL model packs)](https://docs.github.com/en/code-security/how-tos/find-and-fix-code-vulnerabilities/manage-your-configuration/edit-default-setup)
- [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax)
- [Reusing workflow configurations](https://docs.github.com/en/actions/reference/workflows-and-actions/reusing-workflow-configurations) and [Reuse workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows)
- [Creating workflow templates for your organization](https://docs.github.com/en/actions/how-tos/reuse-automations/create-workflow-templates)
- [Secure use reference](https://docs.github.com/en/actions/reference/security/secure-use)
- [Securely using pull_request_target](https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target)
- [About Actions policies](https://docs.github.com/en/actions/concepts/about-actions-policies)
- [Workflow dependency lockfiles](https://docs.github.com/en/actions/concepts/security/workflow-dependency-lockfiles), [Locking action dependencies](https://docs.github.com/en/actions/how-tos/secure-your-work/lock-workflow-dependencies), and [Enforcing lockfiles](https://docs.github.com/en/actions/how-tos/administer/enforce-workflow-lockfiles)
- [Managing GitHub Actions settings for a repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository)
- [Scopes for OAuth apps](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/scopes-for-oauth-apps)
- [Copilot customization cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet) and [Support for different types of custom instructions](https://docs.github.com/en/copilot/reference/custom-instructions-support)
- [Adding repository custom instructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions) and [Adding organization custom instructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-organization-instructions)
- [Custom agents](https://docs.github.com/en/copilot/concepts/customizations/custom-agents), [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration), [Preparing to use custom agents in your organization](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-organization/prepare-for-custom-agents), and [Testing and releasing custom agents](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/test-custom-agents)
- [Creating a `.github-private` repository](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-agents/create-github-private-repo) and [Getting started with enterprise-managed settings](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/use-managed-settings/get-started)
- [Agent skills](https://docs.github.com/en/copilot/concepts/customizations/agent-skills) and [Adding agent skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)
- [Hooks](https://docs.github.com/en/copilot/concepts/customizations/hooks) and [Using hooks with the cloud agent](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/use-hooks)
- [Configure the development environment (copilot-setup-steps)](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/customize-the-agent-environment) and [Using Copilot code review](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/request-a-code-review/use-code-review)
- [Prompt files](https://docs.github.com/en/copilot/concepts/prompting-and-context/prompt-files)
- [GitHub Agentic Workflows](https://docs.github.com/en/copilot/concepts/agent-capabilities/agent-powered-workflows-and-operations/about-github-agentic-workflows) and [Creating GitHub Agentic Workflows](https://docs.github.com/en/copilot/how-tos/github-agentic-workflows/creating-github-agentic-workflows)
- [Managing issue types in an organization](https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/managing-issue-types-in-an-organization) and [Managing default labels](https://docs.github.com/en/organizations/managing-organization-settings/managing-default-labels-for-repositories-in-your-organization)
- [Creating a default community health file (GHES 3.15)](https://docs.github.com/en/enterprise-server@3.15/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file), the older list that still includes `GOVERNANCE.md`

GitHub Changelog:

- [Structured forms for private vulnerability reports (2026-10-01)](https://github.blog/changelog/2026-10-01-structured-forms-for-private-vulnerability-reports)
- [GitHub Actions workflow lockfiles in public preview (2026-10-14)](https://github.blog/changelog/2026-10-14-github-actions-workflow-lockfiles-in-public-preview/)
- [Dependabot reviewers configuration option is replaced by code owners (2025-08-08)](https://github.blog/changelog/2025-08-08-dependabot-reviewers-configuration-option-is-replaced-by-code-owners)

Third-party tools:

- [zizmor configuration](https://docs.zizmor.sh/configuration/)
