> 🦋 A way to manage your versioning and changelogs with a focus on monorepos

## Features

[`changesets`](https://github.com/changesets/changesets) offer a solution to the problem of organizing package releases and grouping changes effectively. Here's why you should consider using changesets:

- **Capture Changes at the Right Time**: Changesets allow contributors to document changes during pull request (PR) submission, ensuring up-to-date and comprehensive information.
- **Distinct Intent to Change**: Changesets separate the intent to change from changelogs and version bumps, providing clear versioning and changelog details.
- **Efficient Versioning Process**: Add changesets in PRs, then combine them for version bumps, streamlining reviews and boosting confidence in versioning decisions.
- **Streamlined Tooling**: Changesets come with CLI-generated tools, automated versioning, and surfaced changesets in PRs, reducing manual effort.
- **Mono-repo Compatibility**: Designed for monorepos, changesets handle interdependencies and ensure compatibility, benefiting single-package repositories as well.

Find more information about changesets in the [official documentation](http://github.com/changesets/changesets)

## Setup

### Install

First, install the @changesets/cli package in the repository where you want to use changesets:

```bash
npm install -D @changesets/cli
```

Once installed, initialize the repository to use changesets:

```bash
npx changeset init
```

### Github Actions

Although changesets can function without continuous integration (CI), it's recommended to use it with a CI system to automate versioning and publishing. You can utilize the [Github Action](https://github.com/changesets/action) provided by the changesets team.

To use the action, create a `.github/workflows/release.yml` file. Follow the instructions under the [With Publishing](https://github.com/changesets/action#with-publishing) section in the documentation, with minor adjustments. This flow updates the versions of changed packages and publishes them to npm.

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

Before using the action, ensure the following checklist is complete:

- Ask your TL, TD, or PE to add the NPM_TOKEN to the repository secrets.
- Confirm that the `package.json` file has the `publish` script: `"publish": "changeset publish"`.
- Ensure the package is built before publishing it to npm (optional).
- Verify that the Node version is correct.
- Confirm that the branch name is accurate.
- Ensure the `package.json` file has the `main` field.
- Verify that only relevant items will be published to npm (e.g., no `__tests__` folder) by adding [a `.npmignore` file or using a files field in the `package.json` file](https://docs.npmjs.com/cli/v9/using-npm/developers#testing-whether-your-npmignore-or-files-config-works).
- Confirm that the `package.json` file has the `repository` field (with `directory` for monorepos).

### Changeset Bot

[Changeset bot](https://github.com/changesets/bot) is a GitHub bot that further automates package publishing. It's recommended to install it in the repository. Refer to the [official documentation](https://github.com/apps/changeset-bot) for more details. Ask your TL, TD, or PE to install it.

## Usage

When you finish making changes, create a PR as usual. The bot will add a comment with a link to add a changeset file. This action should be performed by a repo maintainer or someone responsible for releases. The changeset file will look like this:

```md
---
'@infinum/new-package': patch
---

This is a patch release because of a bug fix.
```

Once the PR is merged, the GitHub Action will create a new release, bump the version, and publish the package to npm. It removes the generated changesets, preparing for the next release.

When a release is ready, responsible person will merge the PR and the Github Action will publish the package to npm. It will also create a new tag and a Github release.

## Pre-releases

Changesets can be used to create pre-releases. It is not as easy as regular releases, but it is well documented in the [official documentation](https://github.com/changesets/changesets/blob/main/docs/prereleases.md).

> Please proceed with caution when creating pre-releases.

First thing you need to do is to enter the pre-release mode:

```bash
npx changeset pre <tag>
```

Most often for `<tag>` we will use `beta` but you can use any tag you want - e.g. `alpha`, `rc`, `next`, etc.

As a result, a new file will be created in the `.changeset` folder. Next thing to do is to update the version of the package you want to release. You can do that by using the `changeset version` command:

```bash
npx changeset version
```

Now, we need to commit the version change and the changeset file:

```bash
git add .
git commit -m "chore: release <tag>"
```

Once commited, we can publish the package:

```bash
npx changeset publish
```

> IMPORTANT: Make sure to use `changeset publish` command instead of `npm publish` because `npm` does not know that we are in prerelease mode.

After publishing, we can push the tag to the origin and create a new release manually.

```bash
git push --follow-tags
```

Once you are done with the pre-release, you can exit the pre-release mode:

```bash
npx changeset exit pre
```

This will create changes in your code so make sure to also commit them.

```bash
git add .
git commit -m "chore: exit pre-release mode"
```

## See it in action

Changesets are already implemented in a few repositories. Check them out to see how they work:

- [js-linters](https://github.com/infinum/js-linters)
- [polyglot-cli](https://github.com/infinum/js-polyglot-cli)

## Resources

- [Changesets](https://github.com/changesets/changesets)
- [Changeset Bot](https://github.com/changesets/bot)
- [Changesets Github Action](https://github.com/changesets/action)
