## Node version management

### Node release schedule

Node.js has defined and predictable release plan. Depending on the Node release version number, the Node release can go through 3 different phases: current, active(LTS) and maintenance. Be aware that odd-numbered versions will not go through active and maintenance phases, while even numbered will go through them all.

#### Current phase

Releases within this phase incorporate most non-major changes that end up on `nodejs/node` main branch. New major releases are branched out every six months. Even-numbered versions are scheduled for release in April, while odd ones are scheduled for October.

#### Active Long Term Support phase

Releases within this phase include new features, bug fixes and updates that have been audited and determined to be stable for the release. With each odd-numbered release, previous even-numbered release will transition to LTS phase. Versions in this phase will receive active support for 12 months after they entered LTS phase.

**You should always aim to use latest LTS version available on your project**

#### Maintenance phase

Releases within this phase will receive critical bug fixes and security updates for 18 months in case of LTS(even-numbered) and 2 months in case of regular(odd-numbered) releases after entering this phase.

### Ensuring same version of Node is being used on all environments

It is very important to keep the same version of Node across the environments. When you update the version of Node in your project, you should take appropriate actions that the new version is also propagated/updated on build/production servers by notifying people responsible for those environments. By not running the same versions you could run into unexpected problems with failing builds, incorrect dependencies etc.

### Managing versions

There are couple of ways for managing versions of Node. This section lists couple of options and generally, all options are fine as long as you are proficient at installing, updating and switching Node versions using that method. These probably aren't all available version managers on the internet, but are probably most popular.

#### n

[n](https://github.com/tj/n) is both powerful and easy to use version manager for Node and should be preferred method for managing Node versions.
You can create a `.n-node-version`, `.node-version` or `.nvmrc` file containing a Node version number. Running `n auto` will look for version specified in one of those files in that order and switch to it.

```bash
#Install n by running script
npm install -g n

#Install latest Node version
n latest

#Install specific version of Node by specifying version or use an alias
n <version>
n lts

#Execute to check Node version currently in use and available versions
n

#Uninstall Node version(this does not affect cached versions)
n uninstall <version>

#Uninstall cached Node version
n rm <version>
```

#### nvm - Node version manager

[Node version manager](https://github.com/nvm-sh/nvm) is a version manager similar to `n` and a good alternative. You can create a `.nvmrc` file containing a Node version number. Running `use`, `install`, `which` etc. commands will use the version specified in the file if none was provided in the command line. When using `nvm` and installing global packages, those packages will only be global for that specific version. eg. You currently use Node 16 and install `polyglot-cli`, if you switch to v18, the package won't be available there until you install it.

```bash
#Install NVM by running script
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash

#Install latest Node version
nvm install node

#Install specific version of Node by specifying version or use an alias
nvm install <version>
nvm install <alias>

#Check Node version currently in use and available versions
nvm ls

#Set default version
nvm alias default <version>

#Switching between Node versions
nvm use <version>
nvm use default

#Uninstall Node version
nvm uninstall <version>
```

#### nvm - Node version manager

[Node version manager](https://github.com/nvm-sh/nvm) is a version manager similar to `n` and a good alternative. You can create a `.nvmrc` file containing a Node version number. Running `use`, `install`, `which` etc. commands will use the version specified in the file if none was provided in the command line. When using `nvm` and installing global packages, those packages will only be global for that specific version. eg. You currently use Node 16 and install `polyglot-cli`, if you switch to v18, the package won't be available there until you install it.

```bash
#Install NVM by running script
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash

#Install latest Node version
nvm install node

#Install specific version of Node by specifying version or use an alias
nvm install <version>
nvm install <alias>

#Check Node version currently in use and available versions
nvm ls

#Set default version
nvm alias default <version>

#Switching between Node versions
nvm use <version>
nvm use default

#Uninstall Node version
nvm uninstall <version>
```

#### Using homebrew

For this method you need to have [homebrew](https://brew.sh/) installed on your Mac(which you probably do). There might be additional steps required when switching versions using `brew` as switching is not supported, so you might need to uninstall and unlink the old version, and install and link new one. Using this method for managing Node versions is discouraged as it is likely that issues will occur at some point if multiple versions are needed.

```bash
#Install Node
brew install node

#Install specific version of node
brew install node@<version>

#Update Node
brew upgrade node

#Uninstall Node
brew uninstall node
```

#### Using direct binary install

Considering previously mentioned options, there is no particular advantage why you would use binary installation. You can still opt in to do so if you wish.

#### avn - Automatic Version Switching

[avn](https://github.com/wbyoung/avn) is a package for automatic version switching of Node using one of your Node version manager package. Currently supported version managing packages are `n`, `nvm` and `nodebrew`. This package is not mandatory and will only work in conjunction with one of the version managing tools. When you `cd` into a directory containing `.node-version` file, `avn` will automatically detect the change and use version manager of your choice to switch to the specified version. Given the `.node-version` and `avn` support, it would be recommended to use `n` as version manager.

```bash
#Install the packages
npm install -g avn avn-nvm avn-n
avn setup
```

#### Specifying version in package.json

You can specify the version of node that your stuff works on by providing Node versions in package.json. If you don't specify this, it means any version will do.

```json
{
  "engines": {
    "node": ">= 18.12"
  }
}
```
