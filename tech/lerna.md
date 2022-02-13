# Lerna

<img src="https://user-images.githubusercontent.com/645641/79596653-38f81200-80e1-11ea-98cd-1c6a3bb5de51.png" alt="logo" width="500"/>

> Lerna is a tool for managing JavaScript projects with multiple packages.

## Basic setup

Install lerna globally:

```none
npm i -g lerna
```

## Authenticate to the Registry

To correctly use lerna to publish packages, you need credentials for the registry you would like to publish to.

The two main registries are NPM and GitHub.

* [https://registry.npmjs.org](https://registry.npmjs.org)
* [https://npm.pkg.github.com/](https://npm.pkg.github.com/)

For this example i am using [npm.pkg.github.com](https://npm.pkg.github.com/)

Create a github PAT (personal access token) with `write:packages` permissions.

Next you need to configure your `$HOME/.npmrc` file with your token, and prefferfed registry.

**Make sure** you replace `TOKEN` and `@rolandwarburton` with your actual token and github username.

```none
//npm.pkg.github.com/:_authToken=<TOKEN>
@rolandwarburton:registry=https://npm.pkg.github.com/
```

Next you can now run the `npm login` command and enter your PAT for the password.

* username: rolandwarburton
* password: PAT
* email: EMAIL@DOMAIN.TLD

**Pay close attention to NPM warnings** when logging in. NPM will tell you if there are issues with your configuration file.

## Lerna Setup

Init lerna with `lerna init`.

Then create two packages.

```none
lerna create core
lerna create cli
```

We need to modify the package.json files for these new packages. Edit `./packages/{core/cli}/package.json` and change the following lines.

```json
{
  "name": "@rolandwarburton/cli",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com/@rolandwarburton"
  },
}
```

## Publishing

When it comes to publishing, all your changes should be committed and pushed to the remote first.

Then run `lerna publish --registry=https://npm.pkg.github.com/` to publish the packages.

Answer the questions and thats it.

On github you can see a new tagged release, and new packages published under the repo.

## Bootstrapping

Here we get down to the important stuff.

`lerna bootstrap` links dependencies from packageA to packageB.

### Modify Code For Bootstrap

Lets modify the *core* and *cli* package such that cli requires code from the core package.

```js
function core() {
  console.log("core");
}

export { core };
```

```js
import {core} from "@rolandwarburton/core";
function cli() {
  core()
}

cli();
```

Also because i am using ESM, the core and cli package.json need a modification to add the `type: module` field.

```json
{
  "type": "module",
}
```

Next we need to modify the *cli* package.json so that it requires the "core" module we are importing.

```json
{
  "dependencies": {
    "@rolandwarburton/core": "^0.0.1"
  }
}
```

Now we can run `lerna bootstrap`.

Lerna has handled the import and symlinked the "core" package into "cli"s node_modules folder.

```output
❯ tree packages/cli/
packages/cli/
├── lib
│   └── cli.js
├── node_modules
│   └── @rolandwarburton
│       └── core -> ../../../core
├── package.json
├── README.md
└── __tests__
    └── cli.test.js
```

Lets test out our linked package with `node packages/cli/lib/cli.js`. This will import the core module and print out "core".

## Deleting Packages and Tags

If you decide to remove a package you may do so by going to githubs website and clicking on the
"packages" button above the list of packages, then selecting a specific package, package settings, delete this package.

Similar, you can delete tags by going to the tags button next the the branches button, selecting a tag, and removing it.

## Lerna Topics

Specific lerna related things. You might want to read this before running `lerna publish` to fine tune
how lerna publishes packages.

### Fixed/Locked Vs Independent

Fixed packages are versioned packages that all use the same version number.
When you publish all packages get a version bump.

Independent packages are packages that have unique version numbers.
When you publish you decide on a per package basis which packages get a version bump.

By default, lerna will publish fixed packages.

To publish Independent packages, you can change `lerna.json` to set the version to independent.

```json
{
  "packages": [
    "packages/*"
  ],
  "version": "independent"
}
```

### Other Lerna Config Options

From the docs:

* `version` - The version of the repo.
* `npmClient` - The npm client to use. (npm or yarn)
* `command.publish.ignoreChanges` - Array of globs that wont be included in changelogs
* `command.publish.message` - A custom commit message to use when publishing
* `command.publish.registry` - The registry to publish to. (npm or github)
* `command.bootstrap.ignore` - Array of globs that wont bootstrapped
* `command.bootstrap.npmClientArgs` - Array of strings that are passed to npm when running the bootstrap command
* `packages` - Array of globs used to identify package locations

This is a good example config adapted from the docs.

```json
{
  "version": "independent",
  "npmClient": "npm",
  "command": {
    "publish": {
      "ignoreChanges": ["*.md"],
      "registry": "https://npm.pkg.github.com"
    },
    "bootstrap": {
      "ignore": "component-*",
    }
  },
  "packages": ["packages/*"]
}
```

### To Publish or Not to Publish

If you don't want to publish the package, add `"private": true` to the package.json file in the package you wish to not publish.
This will still generate a git tag (but this is ok/intended).

## Lerna Commands

Lets do the basics first.

* `lerna create` - add a new package
* `lerna bootstrap` - link dependencies
* `lerna init` - initialize a new lerna project

To add a dependency you can run `lerna add <package> packages/NAME --dev`. Omit the dev flag if its not a dev dependency.

To run a command in all packages at once run `lerna run COMMAND`.
Commonly you can run `lerna run build` to build all packages.
