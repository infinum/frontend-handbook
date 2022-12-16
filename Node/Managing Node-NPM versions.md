## Node version management

### Node release schedule

Node.js has defined and predictable release plan. Each node version goes through 3 phases: current, active(LTS) and maintenance. Be aware that odd-numbered versions will not go through active nad maintenance phases.

#### Current phase

Incorporates most non major changes that end up on `nodejs/node` main branch. New major releases are branched out every six months. Even-numbered versions are scheduled for release in April, while odd ones are scheduled for October.

#### Active Long Term Support phase

Versions in this phase include new features, bug fixes and updates that have been audited and determined to be stable for the release. With each odd-numbered release, previous even-numbered release will transition to LTS phase. Versions in this phase will receive active support for 12 months after they entered LTS phase.

**You should always aim to use latest LTS version available on your project**

#### Maintenance phase

Versions in this phase will receive critical bug fixes and security updates for 18 months after it enters this phase.

### Managing versions

There are couple of ways for managing versions of Node. I will list couple of options and generally, all options are fine as long as you are proficient at installing, updating and switching Node versions using that method.

(Discuss if any of the method should be default/preferred)

#### nvm - Node version manager

.nvmrc file

#### n

.n file

#### avn - Automatic Version Switching

#### Using homebrew

For this method you need to have `homebrew` installed on your Mac(which you probably do). There might be additional steps required when switching versions using `brew` as switching is not supported, so you might need to uninstall and unlink the old version, and install and link new one.

```bash
#Install Node
brew install node

#Install specific version of node
brew install node@14

#Update Node
brew upgrade node

#Uninstall Node
brew uninstall node
```

#### Using direct binary install

advantages and disadvantages

recommended setup

where should you enter which node version should be used
package.json -> engines
.node-version file for avn

for each write down steps for installing, switching, removing, and using node versions
writing down useful commands, like nvm ls

How to ensure that developer, build server and production server all use the same version of node
^ not really sure what is the correct answer here, is this aimed at that server uses the same version by mentioning it somewhere in the code

and why is that important
to avoid version mismatch, example locally you are using node 14 to install packages, but on server you are using node 16, and the installation might fail
