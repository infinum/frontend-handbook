## Requirements

We wanted to use git commit hash inside our application. Our use case was to display it next to the version of the app.

## First attempt

We tried to execute `git rev-parse --short HEAD` command in our `next.config.js` file but got the following error:

```text
Error: Command failed: git rev-parse --short HEAD
fatal: not a git repository (or any parent up to mount point /)
```

## Why?

[Mina](https://github.com/mina-deploy/mina) deletes the `.git` directory after it clones the repository. You can check the implementation [here](https://github.com/mina-deploy/mina/blob/master/tasks/mina/git.rb).

## Solution

If we provide the commit argument to `mina deploy` command as suggested in [Deployment chapter](https://infinum.com/handbook/books/frontend/react/next/deployment) the git commit hash is saved to `.mina_git_revision` file ([code](https://github.com/mina-deploy/mina/blob/master/tasks/mina/git.rb#L33)).

```js
// next.config.js
const fs = require('fs');
const path = require('path');

const revisionFilePath = path.join(__dirname, '.mina_git_revision');

let GIT_COMMIT_HASH = '';

if (fs.existsSync(revisionFilePath)) {
  GIT_COMMIT_HASH = fs.readFileSync(revisionFilePath, 'utf-8').trim();
}
```
