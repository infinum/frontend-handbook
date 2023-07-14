> 🦋 A way to manage your versioning and changelogs with a focus on monorepos

## Features

[`changesets`](https://github.com/changesets/changesets) offer a solution to the problem of organizing package releases and grouping changes effectively. Here's why you should consider using changesets:

- **Capture Changes at the Right Time:** Changesets allow contributors to document changes when submitting a pull request (PR), ensuring fresh and comprehensive information.
- **Distinct Intent to Change:** Changesets represent an "intent to change" separate from changelogs and version bumps, providing clear versioning and changelog details.
- **Efficient Versioning Process:** Add changesets in PRs, then combine them for version bumps, streamlining reviews and increasing confidence in versioning decisions.
- **Streamlined Tooling:** Changesets come with tools for CLI-generated changesets, automated versioning, and surfacing changesets in PRs, reducing manual effort.
- **Compatible with Mono-repos:** Designed for monorepos, changesets handle interdependencies and ensure compatibility, but also benefit single-package repositories.

Find more information about changesets in the [official documentation](http://github.com/changesets/changesets)

## Setup

### Install

First, install the `@changesets/cli` package in the repository you want to use changesets in:

```bash
npm install -D @changesets/cli
```

Once this is done, you need to initialize the repository to use changesets:

```bash
npx changeset init
```

### Github Actions

Even thought the changesets can be used without any CI, it is recommended to use it with a CI to automate the versioning and publishing process. To do so, you can use the [Github Action](https://github.com/changesets/action) provided by the changesets team.

To use it, you need to create a `.github/workflows/release.yml` file. The action can be setup in few different ways, but the one that we use is the one explained under [With Publishing](https://github.com/changesets/action#with-publishing) section in the documentation with minor adjustments. This flow will update the version of the packages that have changed and publish them to npm.

```yml
name: Release

on:
  push:
    branches:
      # Make sure to check the branch name here; usual values are `main` and `master`
      - master

concurrency: ${{ github.workflow }}-${{ github.ref }}

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v3

      # Make sure to use correct Node.js version for your project
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          cache: 'npm'
          # Use the node version specified in the .node-version file if it exists
          node-version-file: '.node-version'
          # otherwise fall back to the version specified here
          # node-version: '18.x'

      - name: Install Dependencies
        run: npm ci

      - name: Create Release Pull Request or Publish to npm
        id: changesets
        uses: changesets/action@v1
        with:
          publish: npm run publish
          # if your repository is using conventional commits, you should use the following option (the message can be customized)
          # commit: 'ci: version packages'
        env:
          # GITHUB_TOKEN is required for creating a pull request and will be provided by the Github Action automatically
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          # NPM_TOKEN is required for publishing to npm and needs to be provided manually
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

      # This step will push tags to the repository after the packages are published and will create a new release on Github
      - name: Push git tag after publish
        if: steps.changesets.outputs.published == 'true'
        run: git push --follow-tags
```

Before you can use the action, you need to create make sure that the checklist is completed:

- Ask the your TL or TD or PE to add the NPM_TOKEN to the repository secrets
- Make sure that the `package.json` file has the `publish` script: `"publish": "changeset publish"`
- Make sure that the package is build before publishing it to npm (optional)
- Make sure that the Node version is correct
- Make sure that the branch name is correct
- Make sure that the `package.json` file has the `main` field
- Make sure that only relevant stuff will be published to npm (e.g. no `__tests__` folder) by adding [a `.npmignore` file or using a `files` field in the `package.json` file](https://docs.npmjs.com/cli/v9/using-npm/developers#testing-whether-your-npmignore-or-files-config-works)
- Make sure that the `package.json` file has the `repository` field (with `directory` if monorepo)

### Changeset Bot

[Changeset bot](https://github.com/changesets/bot) is a Github bot that will help you even more with the automation of the package publishing. It is recommended to use it to make sure that the changesets are added to the PRs. To use it, you need to install it in the repository you want to use it in. You can find more information about it in the [official documentation](https://github.com/apps/changeset-bot). You will need to ask your TL or TD or PE to install it.

## Usage

Once you are done with your changes, you will create a PR as usual. Once a PR is created bot will add a comment where a changeset can be added. This will be a link which will create new file in your PR. This action should be done by repo maintainer or someone who is responsible for the release. The changeset file will look like this:

```md
---
'@infinum/new-package': patch
---

This is a patch release because of a bug fix.
```

Once a PR is merged, the PR will be created by the Github Action and the version will be bumped. The version will be bumped based on the changeset types. It will converge all changesets and bump the version based on the highest change type. The changeset types are following the semver versioning - major, minor, patch. It will also remove generated changesets so you have a clean slate for the next release.

When a release is ready, responsible person will merge the PR and the Github Action will publish the package to npm. It will also create a new tag and a Github release.

## See it in action

We already have the changesets implemented in few repositories. You can check them out to see how it works:

- [js-linters](https://github.com/infinum/js-linters)
- [polyglot-cli](https://github.com/infinum/js-polyglot-cli)

## Resources

- [Changesets](https://github.com/changesets/changesets)
- [Changeset Bot](https://github.com/changesets/bot)
- [Changesets Github Action](https://github.com/changesets/action)
