recommended setup

be careful how you install global packages, as they will be installed globally per package

how to install which managers, nvm n homebrew, direct binary install

advantages and disadvantages

-add voskas comment

phases
node has 3 main phases - current incorporates most non major changes on main branch, every 6 months
-active lts, new features, bug fixes and updates that are stable for the release. every lts major version is actively maintained for 12 months after it enters lts - versions in maint mode, only critical bug fixes and security updates are included, those versions are maintained for 18 months since they enter the status

you should aim to actively have the latest LTS version available

---

i wouldn't add too many options here, i believe this should recommended by the team
where should you enter which node version should be used
package.json -> engines
.node-version file for avn
.nvm file for nvm
.n file for n

for each write down steps for installing, switching, removing, and using node versions
writing down useful commands, like nvm ls

How to ensure that developer, build server and production server all use the same version of node
^ not really sure what is the correct answer here, is this aimed at that server uses the same version by mentioning it somewhere in the code

and why is that important
to avoid version mismatch, example locally you are using node 14 to install packages, but on server you are using node 16, and the installation might fail
