## Configuration

Configure your global gitignore file, usually in your home directory named `.gitignore_global`. It should contain ignore rules specific for your operating system and editor of choice. You can find these ignore rules [here](https://github.com/github/gitignore/tree/master/Global) and add them to your global gitignore file.

After you have created your global gitignore file, update the global git configuration to use your global ignore rules:

`git config --global core.excludesfile ~/.gitignore_global`


### _OPTIONAL: Color highlighting_

Depending on your terminal color profile, the default Git diff coloring might not be optimal. The following colors are suggested instead:

```bash
git config --global color.ui true

git config --global color.diff-highlight.oldNormal    "red bold"
git config --global color.diff-highlight.oldHighlight "red bold 52"
git config --global color.diff-highlight.newNormal    "green bold"
git config --global color.diff-highlight.newHighlight "green bold 22"

git config --global color.diff.meta       "yellow"
git config --global color.diff.frag       "magenta bold"
git config --global color.diff.commit     "yellow bold"
git config --global color.diff.old        "red bold"
git config --global color.diff.new        "green bold"
git config --global color.diff.whitespace "red reverse"
```

## Flow

We use the [Git Flow](https://nvie.com/files/Git-branching-model.pdf) workflow model.

### Master
The `master` branch is one of the two branches with an infinite lifetime. The source code of `HEAD` always reflects a production-ready state.

### Develop

`develop` is the second branch with an infinite lifetime. The `develop` branch source code of `HEAD` always reflects a state with the latest delivered development changes for the next releases.

When the source code in `develop` branch reaches a stable point and is ready to be released, all of the changes should be merged back into `master` and tagged with a release number.

### Feature

Next to the main `master` and `develop` branches, the Flow model uses a variety of supporting branches to aid parallel development between team members, ease tracking of features, prepare for production releases, and assist in fixing live production problems quickly.
Unlike the two main branches, these branches have a limited lifetime.

Feature branches may branch off from `develop` and must be merged back into `develop`. Feature branch naming convention is as follows: `feature/*`, for example, `feature/login-page`.
It is advisable to put a unique task identificator into the branch name. Usually, that would be a task ID or some other unique task property, for example `feature/8712-login-page`

Feature branches are used to develop new features for the upcoming or a distant future release. These branches exist as long as the feature is in development. Eventually, they are merged back into `develop` or discarded.


### Hotfix

Hotfix branches are created if you need to immediately act upon an undesired state of a live production version. If a critical bug in production must be resolved immediately, a hotfix branch can be branched off from the `master` branch. Similarly to the feature branches, the hotfix branches are named `hotfix/*`. A hotfix branch should be merged back into `master` and `develop`.


## Committing

A commit should round up only the related changes. For example, fixing two different bugs should produce two separate commits. Small, atomic commits make it easier for other developers to understand the changes and roll them back if needed.

Committing will often keep your commits small and help you commit only related changes. It will also allow you to share your code more frequently with others. Furthermore, it will help avoide merge conflicts.

Don't use `git add -A` to stage changes. If you have a lot of changes you can filter through them with `git add -p` and stage only relevant changes.


### Commit messages

Writing good commit messages is important. A clear commit log will be easier to navigate through and make it easier for other people to follow along. You can read on how to write proper commit messages [here](https://chris.beams.io/posts/git-commit/).


## Pull requests

Avoid creating pull requests that contain [a lot of changes](https://www.urbandictionary.com/define.php?term=KPR). If you are implementing a feature that requires a lot of file changes, try breaking it up into multiple smaller features which results in multiple pull requests.

Smaller pull requests are easier to review and will therefore be merged sooner.

## Optional

### Codeowners

You can use a CODEOWNERS file to define individuals or teams that are responsible for code in a repository. Code owners are automatically requested for review when someone opens a pull request that modifies the code they own. [More about code owners](https://help.github.com/articles/about-code-owners/).
