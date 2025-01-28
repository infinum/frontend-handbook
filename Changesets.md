*Changesets* is a lightweight tool that helps you manage versions and generate changelogs for packages in a monorepo (or single-package repository). By adding a small file describing each change, *Changesets* can automatically bump versions, publish packages, and create release notes with minimal friction.

## Why Use Changesets?

1. **Immediate Documentation**: Document your changes while opening a PR instead of waiting for the final release. This ensures you never forget important details.
2. **Clear Separation of Concerns**: Changesets decouple the intent to change (patch, minor, major) from the act of publishing. Your changelogs and version bumps become transparent to the entire team.
3. **Easy Collaboration & Review**: Each pull request can include a changeset file that explicitly states how the package version should be updated and why. This fosters more meaningful code reviews.
4. **Automated Versioning**: When merged, changesets can automatically handle version bumps, changelog generation, and publishing. This saves time and reduces human error.
5. **Monorepo Ready**: Designed for multi-package repositories, Changesets resolve inter-package dependencies, ensuring consistent and reliable versioning across the codebase.

For more info, see the [official Changesets documentation](https://github.com/changesets/changesets).

## Getting started

### Install and initialize

Install the Changesets CLI in your repository:

```bash
pnpm install -D -E @changesets/cli
```

Then, initialize *Changesets*:

```bash
pnpm changeset init
```

This creates a hidden folder, `.changeset/`, with a base configuration file. You’re now ready to track changes in your repo.

### Adding a *Changeset* to every Pull Request

Whenever you open a Pull Request, add a *changeset* to describe how the changes affect the package(s).

1. **Creating a changeset**

After committing your code changes, run:

```bash
pnpm changeset
```

This interactive prompt asks:

* Which packages are affected? (*This question is skipped in single-package repositories*) Use the space bar to select one or more packages in a monorepo.

* Bump type (*patch, minor, major*)?
  * **patch** for backward-compatible bug fixes.
  * **minor** for new features that don’t break existing APIs.
  * **major** for incompatible API changes.

* Summary for this change - Use an impersonal tone, focusing on what changed and why (like commit messages).

2. **Commiting the changeset**

Once you select the affected packages, version bump type and write down summary of changes, a `.md` file is created in the `.changeset/` folder (the file name is automatically generated, e.g. `strange-bees-visit.md`):

```md
---
"@infinum/some-package": minor
---

Introduce a new method `doSomethingAwesome` and fix a small bug in the `init` function.
```

You should commit the `.changeset` file:

```bash
git add .
git commit -m "chore: add changeset for [feature or fix]"
```

And push your branch - this changeset becomes part of the PR for reviewers to see.

> Note: You don’t have to create a dedicated commit for your changeset. Feel free to include the changeset file in the same commit as your code changes. The key point is that the changeset exists for the release automation to reference, regardless of how it’s committed.

### Day-to-day example

1\. Pull Latest

```bash
git pull origin master
```

2\. Create or Switch to a Feature Branch

```bash
git checkout -b feat/improve-logging
```

3\. Make Your Code Changes

(Fix a bug, add a feature, etc.)

4\. Run `pnpm changeset` to create your changeset.

5\. Commit and Push

```bash
git add .
git commit -m "feat(logging): improve error logging format"
git push -u origin feat/improve-logging
```

6\. Open a PR

GitHub will show the changes, including the new `.md` in `.changeset/`.

7\. Review & Merge

Once approved and merged, the CI pipeline will handle version bumps and publishing automatically.

### Continuous Integration Setup

Although changesets can function without continuous integration (CI), it's recommended to use it with a CI system to automate versioning and publishing. You can utilize the [Github Action](https://github.com/changesets/action) provided by the changesets team.

To use the action, create a `.github/workflows/release.yml` file. Follow the instructions under the [With Publishing](https://github.com/changesets/action#with-publishing) section in the documentation, with minor adjustments. This flow updates the versions of changed packages and publishes them to npm registry.

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
        uses: actions/checkout@v4

      # Corepack makes sure to use correct Node.js version for your project, it requires having "engines" specified in package.json file
      # Read more about Corepack and advanced dependencies caching at: https://infinum.com/handbook/frontend/node/managing-node-npm-versions
      - name: 🗃️ Enable corepack
        run: corepack enable
        shell: bash

      - name: Setup Node.js
        uses: actions/setup-node@v4

      - name: Install Dependencies
        run: pnpm install --prod --frozen-lockfile

      - name: Create Release Pull Request or Publish to npm registry
        id: changesets
        uses: changesets/action@v1
        with:
          publish: pnpm ci:publish
          # if your repository is using conventional commits, you should use the following option (the message can be customized)
          # commit: 'ci: version packages'
        env:
          # GITHUB_TOKEN is required for creating a pull request and will be provided by the Github Action automatically
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          # NPM_TOKEN is required for publishing to registry and needs to be provided manually
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

      # This step will push tags to the repository after the packages are published and will create a new release on Github
      - name: Push git tag after publish
        if: steps.changesets.outputs.published == 'true'
        run: git push --follow-tags
```

Before using the action, ensure the following checklist is complete:

* Ask your TL, TD, or PE to add the NPM\_TOKEN to the repository secrets.
* Confirm that the `package.json` file has the `publish` script: `"ci:publish": "changeset publish"`.
* Ensure the package is built before publishing it to npm (optional).
* Verify that the Node version is correct.
* Confirm that the branch name is accurate.
* Ensure the `package.json` file has the `main` field.
* Verify that only relevant items will be published to the registry (e.g., no `__tests__` folder) by adding [a `.npmignore` file or using a files field in the `package.json` file](https://www.npmjs.com/package/npm-packlist) and running [pnpm pack](https://pnpm.io/cli/pack) command.
* Confirm that the `package.json` file has the `repository` field (with `directory` for monorepos).

### GitHub Actions Gotchas

If you actually want to deploy applications with changesets, not just publish new packages to NPM registry, there are 2 gotchas:

* Workflows can’t start other workflows.
* You can't trigger 3+ workflows on `push tags` at the same time.

But those issue can be tackled:

* For triggering new workflows: use a `Fine-Grained Personal Access Token`, so "a real user" is creating new workflows instead of default GitHub Actions user when using `${{ secrets.GITHUB_TOKEN }}`
* For triggering deploy workflows for multiple apps: switch to the `release published` trigger instead of `pushed tag`

Example `release published` trigger:

```yml
on:
  release:
    types: [published]
```

It has some drawbacks — either your own user will be creating all commits, pull requests, tags, and releases needed for deployment, or you'll need to create a dedicated user for that. Some organizations may need to pay for an additional seat, but it's usually not a big deal.

**Creating Fine-grained Personal Access Token**

Required repository permissions:

* **Contents** - Read and Write
* **Pull requests** - Read and Write
* **Metadata** - Read-only (mandatory by default)

> ⚠️ Warning! If possible, create a dedicated user for that, and it should have access only to this single repository. Alternatively, create this token on an organization level and limit repository access as needed.

After you’ve created the Fine-grained PAT, you have to add it to your repository secrets. In the examples below, it’s named `CHANGESETS_GITHUB_PAT`.

**Updated Release workflow**

Example `.github/workflows/release.yml`:

```yml
name: 📢 Release

on:
  push:
    branches:
      - main

concurrency: ${{ github.workflow }}-${{ github.ref }}

jobs:
  release:
    name: 📢 Release
    runs-on: ubuntu-22.04
    steps:
      - name: 📥 Checkout Repository
        uses: actions/checkout@v4
        with:
          token: '${{ secrets.CHANGESETS_GITHUB_PAT }}'

      - name: 💻 Node setup
        uses: ./.github/actions/node-setup

      - name: 🧑‍💻 Configure Git User
        run: |
          git config --global user.name "Bot Kamil"
          git config --global user.email "kamil@infinum.com"

      - name: 🏷️ Create Release Pull Request
        uses: changesets/action@v1
        with:
          publish: pnpm changeset tag
          setupGitUser: false
        env:
          GITHUB_TOKEN: ${{ secrets.CHANGESETS_GITHUB_PAT }}
          HUSKY: 0
```

* Custom `token` in `actions/checkout@v4`: Automates all Git commands (commit, tag, push, etc.) as your custom user (from PAT) instead of the default GitHub Actions user.
* *Node setup* step: Install dependencies.
* *Configure Git User* step: Required so that `changesets/action` can commit version changes (e.g., “Version Packages”).
* `with.publish: pnpm changeset tag`: Use this if you have any private repositories, or if you just want to tag instead of publishing to the NPM registry.
* `with.setupGitUser: false`: Prevents `changesets/action` from overriding your previously configured Git user.
* `GITHUB_TOKEN`: Should use the custom Fine-grained PAT.
* `HUSKY: 0`: Disables any Git hooks — your app should be analyzed and tested before the Release workflow runs.

### Changeset Bot

You can install [Changeset Bot](https://github.com/changesets/bot) to get additional automated PR comments. Once installed, if a PR lacks a changeset, the bot will prompt you to add one. This is highly recommended for teams to maintain consistent usage.

### Pre-Releases (Beta, Alpha, RC)

Pre-releases allow you to publish “unstable” versions (e.g., `1.2.0-beta.1`) for testing before a final release. See the [official docs](https://github.com/changesets/changesets/blob/main/docs/prereleases.md) for details.

> ⚠️ Warning! Prereleases are very complicated! Using them requires a thorough understanding of all parts of npm publishes. Mistakes can lead to repository and publish states that are very hard to fix.

Typical workflow:

1. Enter Pre-Release Mode

   ```bash
   pnpm changeset pre <tag>
   ```

   Usually `<tag>` is `beta`, but you can use `alpha`, `rc`, `next`, etc.

2. Version & Commit

   ```bash
   pnpm changeset version
   git add .
   git commit -m "chore: release beta"
   ```

3. Publish the Pre-Release

   ```bash
   pnpm changeset publish
   ```

   > ⚠️ Important! Use `changeset publish` instead of `pnpm publish` to respect pre-release mode.

4. Push Tags

   ```bash
   git push --follow-tags
   ```

   You can also manually create a GitHub release if desired.

5. Exit Pre-Release Mode

   ```bash
   pnpm changeset exit pre
   git add .
   git commit -m "chore: exit pre-release mode"
   ```

After this, the packages return to normal versioning.

## See it in action

*Changesets* are already implemented in a few repositories. Check them out to see how they work:

* [js-linters](https://github.com/infinum/js-linters)
* [polyglot-cli](https://github.com/infinum/js-polyglot-cli)

## Resources

* [Changesets](https://github.com/changesets/changesets)
* [Changeset Bot](https://github.com/changesets/bot)
* [Changesets Github Action](https://github.com/changesets/action)
