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

It is very important to keep the same version of Node across the environments. When you update the version of Node in your project, you should take appropriate actions that the new version is also propagated/updated on build/production servers by notifying people responsible for those environments. By not running the same versions you could run into unexpected problems with failing builds, incorrect dependencies etc. In case you are using server-side rendering, for example Next.js or Angular Universal you need to make sure that the same Node version is used on build server during the build step and on the production server when running the app.

### Managing versions

There are a couple of ways to manage versions of `Node.js`. All options are fine as long as you are proficient in installing, updating, and switching Node versions using your chosen method. However, the recommended approach is to use `Corepack` and `n`.

#### Corepack

`Corepack` is a tool that comes bundled with Node.js starting from version `16.10.0`. It provides a way to manage package managers (such as `npm`, `yarn`, and `pnpm`) without needing to install them globally. Corepack ensures that the specific package manager version specified in your project is used, which helps maintain consistency across different development environments.

**How Corepack works**

* **Bundling**: Corepack is included with `Node.js` distributions. You can enable it using the command corepack enable.
* **Specifying Package Managers**: In your project's package.json, you can specify which package manager and version your project should use.
* **Automatic Installation**: Corepack will automatically download and install the specified version of the package manager when you run related commands like `pnpm install`.

**Specifying package manager in package.json**

To ensure your project uses specific version of and `pnpm`, you should provide the `packageManager` field in your `package.json` file.

```json
{
  "packageManager": "pnpm@9.4.0",
}
```

**How Corepack manages versions**

By specifying the `packageManager` version in your `package.json`, Corepack will:

* **Check the package.json**: When you run commands that involve the package manager (e.g., installing dependencies), Corepack reads the `packageManager` field in your `package.json`.
* **Download and Install**: If the specified package manager version is not already installed, Corepack will automatically download and install it.
* **Use the Specified Version**: Corepack ensures that the commands are executed using the specified version of the package manager, ensuring consistency across different environments.

This process helps manage the appropriate versions of the package manager for your project, ensuring everyone working on the project uses the same versions.

**Using Corepack in Docker**

Using Corepack in your Docker environment ensures consistency between development and production environments. By leveraging Corepack:

* You avoid potential version mismatches by locking down the exact version of pnpm used across environments.
* You gain the benefits of pnpm's strict dependency management and efficient disk usage within your containerized application.

To use Corepack in a Dockerfile, you need to ensure that Corepack is enabled and properly configured for your project within the container environment. Here’s a step-by-step guide on how to set up Corepack with pnpm in a Dockerfile:

```dockerfile
# Use an official Node.js image as a base image
FROM node:22-alpine

# Enable Corepack, which is included by default in Node.js versions >= 16.10.0
RUN corepack enable

# Optional: Ensure the specific package manager version you want is installed
RUN corepack prepare pnpm@7.15.0 --activate

# Set the working directory in the container
WORKDIR /app
```

You don't need to specify the package manager in Dockerfile if you've provided `packageManager` field in your `package.json` file.

#### n

[n](https://github.com/tj/n) is a powerful and easy-to-use version manager for `Node.js`. It is highly recommended for managing versions due to its simplicity and effectiveness.

**How to Use n**

1. Install n globally using npm:

```bash
npm install -g n
```

2. Easily switch between different Node.js versions using:

```bash
n <version>
```

For example, to switch to Node.js version 14.17.0:

```bash
n 14.17.0
```

3. Ensure your `package.json` specifies the `Node.js` version:

```json
{
   "engines": {
      "node": "20.15.1"
   }
}
```

4. Automatic Version Switching: Use the `n auto` command to switch `Node.js` versions based on the engines field:

```bash
n auto
```

Specifying `engines` field and using `n` ensures that the correct version of `Node.js` is always used for your project, helping to prevent compatibility issues.
