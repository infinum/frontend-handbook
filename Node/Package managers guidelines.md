## Package Manager Guidelines

### Overview of Package Managers

A package manager is a tool that automates the process of installing, upgrading, configuring, and managing project dependencies. For JavaScript and Node.js projects, there are three popular package managers: npm, yarn, and pnpm. Each package manager offers unique features and benefits that cater to different development workflows.

At Infinum, we’ve chosen pnpm as the default package manager due to its superior performance, disk space efficiency, and advanced dependency management features.

#### npm

`npm` (Node Package Manager) is the default package manager that comes bundled with Node.js. It has been around for a long time and is the most widely used package manager in the Node.js ecosystem.

**Key Features:**

* Default and widely adopted package manager.
* Comes pre-installed with Node.js.
* Straightforward CLI for installing and managing packages.
* Global package installation support.

**Drawbacks:**

* Uses more disk space due to duplicate installations of packages.
* Slower dependency resolution compared to modern alternatives.

#### Yarn

`yarn` is an alternative to npm created by Facebook to address performance and reliability concerns with npm. It was built to be faster and more deterministic by introducing features like a lockfile and offline caching.

**Key Features:**

* Fast and deterministic installations due to the lockfile mechanism.
* Offline mode support.
* Workspaces feature for managing monorepos.
* Improved security with stricter integrity checks.

**Drawbacks:**

* While fast, it can still suffer from disk space inefficiencies.
* Yarn’s newer features are often outpaced by improvements in pnpm.

#### pnpm (Preferred)

`pnpm` is the latest evolution in package management for Node.js. It introduces a unique approach to storing dependencies that dramatically improves performance and efficiency. Instead of duplicating package files across projects, pnpm uses a content-addressable storage mechanism that links packages from a shared location on your system.

### Why We Chose pnpm as Our Default Package Manager

We have standardized on pnpm for several reasons:

1. **Performance:** pnpm consistently outperforms npm and yarn in terms of installation speed. This results in quicker setup times for developers and faster CI/CD pipelines.
2. **Disk Space Efficiency:** By leveraging a content-addressable file system, pnpm avoids redundant package installations, significantly reducing disk space usage in larger projects.
3. **Stricter Dependency Management:** pnpm enforces a more consistent dependency tree, helping to avoid "dependency hell" that can occur with other package managers.
4. **Monorepo Support:** pnpm’s workspace feature makes it the best choice for monorepo setups, allowing you to manage multiple related packages within a single repository.
5. **Community Adoption:** pnpm is rapidly gaining adoption and is widely supported across popular libraries and frameworks.

#### Phantom Dependencies in npm and Yarn

One common issue that arises with both npm and yarn is the problem of **phantom dependencies**. These are dependencies that your project can access but are not explicitly listed in your `package.json` or defined as part of your direct dependency tree. This can lead to unpredictable behavior and introduces several risks in your project.

**How Phantom Dependencies Occur**

Phantom dependencies typically occur due to the following reasons:

1. **Hoisting Behavior**: Both npm and yarn use a process called *hoisting* where dependencies are moved to a top-level node\_modules directory. As a result, dependencies that are nested (i.e., dependencies of other dependencies) might be hoisted to the root directory. This can make them accessible to your project even though they aren’t directly listed as dependencies.

2. **Implicit Dependency Access**: When a nested dependency is hoisted, it becomes available in your project’s node\_modules, even if it is not explicitly declared. This makes it possible for you to accidentally use a package that your project does not officially depend on.

3. **Incorrectly Configured Dependencies**: Sometimes developers may inadvertently use packages that are brought in as transitive dependencies (dependencies of dependencies). Since these dependencies are not declared in the `package.json` file, your project might rely on them without explicitly managing them.

**Why Phantom Dependencies Are Risky**

Phantom dependencies pose several risks:

1. **Fragile Builds**: Because phantom dependencies are not listed in your `package.json`, they can disappear unexpectedly if they are no longer included by the original package that brought them in. This could lead to sudden build failures or bugs when you least expect them.

2. **Unreliable Environment Replication**: If another developer or a CI/CD pipeline installs your project’s dependencies, the exact hoisting behavior might differ based on subtle differences in the environment, node\_modules structure, or even the order of installations. This makes it hard to guarantee that the same dependencies are available across all environments.

3. **Version Conflicts**: Phantom dependencies can introduce version conflicts. Since they are not explicitly managed, you might end up using an incompatible version of a dependency that was hoisted due to the resolution strategy of npm or yarn.

4. **Security and Compliance Risks**: Unmanaged dependencies increase the chance of missing critical security updates, as you are not actively monitoring or managing these packages. Additionally, if your project is audited for licensing or security, phantom dependencies might introduce unexpected issues.

**How pnpm Avoids Phantom Dependencies**

Unlike npm and yarn, pnpm uses a unique approach where each package has its own isolated node\_modules structure. This prevents packages from accessing dependencies that are not explicitly declared in their own `package.json` file. If a package tries to access a dependency that is not listed, pnpm throws an error, enforcing strict and clear dependency management.

This strictness ensures that your project is always explicit about its dependencies, reducing the likelihood of unpredictable bugs, version conflicts, or build failures due to phantom dependencies.

#### Deppelgangers

In the context of npm and yarn, "doppelgangers" are a type of issue that arises due to the way these package managers handle dependency installations. Doppelgangers refer to situations where different versions of the same dependency are installed multiple times across a project, leading to duplication, inconsistencies, and potential conflicts. This problem is particularly relevant when dealing with large or complex dependency trees.

**Understanding Doppelgangers in npm and Yarn**

When using npm or yarn, dependencies are resolved and installed in a way that can lead to multiple versions of the same package being installed in different parts of the node\_modules structure. This happens because each package in the tree may have its own specific version of a dependency that doesn’t match other versions requested by different packages.

For example:

* Package A requires `lodash@4.17.0`.
* Package B requires `lodash@4.16.0`.

In npm or yarn, both versions may be installed separately, leading to redundant copies of lodash scattered across the project. This duplication can waste disk space, slow down installations, and lead to inconsistencies, especially if different versions of the same package behave differently.

**Why We Chose pnpm to avoid Doppelgangers**

pnpm takes a fundamentally different approach to dependency management that directly addresses the doppelganger problem. Instead of installing dependencies in a flat or deeply nested structure, pnpm uses a content-addressable storage system where all versions of a package are stored globally on the filesystem and linked into your project’s node\_modules using hard links or symlinks.

This approach offers several key advantages:

* **No Redundant Installations**: With pnpm, different versions of the same package are never redundantly installed. If a version of a package is already present in the global store, it is simply linked to where it’s needed. This eliminates the doppelganger issue entirely.
* **Consistent Dependency Resolution**: pnpm enforces a strict dependency tree that ensures only the correct and intended versions of packages are used. This minimizes the risk of conflicts that can arise from duplicate installations.
* **Efficient Disk Usage**: By storing each version of a package only once and linking it as needed, pnpm drastically reduces the disk space used by node modules, even in large projects with many dependencies.
* **Improved Performance**: Because pnpm avoids redundant downloads and installations, it’s faster to install dependencies, especially in projects with complex dependency trees.

### Getting Started with pnpm

To start using pnpm, you first need to install it globally:

```bash
npm install -g pnpm
```

You can then use pnpm just like any other package manager, but with some enhancements.

#### Installing Dependencies

To install all dependencies listed in your package.json, run:

```bash
pnpm install
```

#### Adding a New Package

To add a new package as a dependency, use:

```bash
pnpm add <package-name>
```

For example:

```bash
pnpm add express
```

To add development dependency:

```bash
pnpm add -D jest
```

To add dependency with exact latest version instead of one with caret:

```bash
pnpm add -E storybook
```

#### Running scripts

Scripts defined in your package.json can be run using:

```bash
pnpm run <script-name>
```

For example:

```bash
pnpm run build
```

You can also use shorthand:

```bash
pnpm build
```

#### Managing Workspaces (Monorepos)

If you’re working with a monorepo, pnpm makes it easy to manage multiple packages using workspaces. You define your workspace in a pnpm-workspace.yaml file:

```yml
packages:
  - 'packages/*'
```

This setup allows you to run commands across all packages and share dependencies efficiently.

#### Checking Dependency Health

To audit your project’s dependencies for vulnerabilities:

```bash
pnpm audit
```

### Conclusion

While npm and yarn are popular choices, pnpm offers significant advantages that make it the ideal package manager for our development workflow. Its speed, efficiency, and robust dependency management align with our goals of building scalable and maintainable software.

By adopting pnpm as our default, we ensure that our projects are fast, efficient, and consistent across all environments, ultimately improving productivity and reducing friction in our development process.
