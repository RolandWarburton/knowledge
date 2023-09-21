# User Scripts

This article will be about my opinions on user scripts, and how you can use them to maximum effect!

This writing will cover how to use a bundler to embed JS libraries,
and how to create JSX components in user scripts. 
Plus some extra bits that i think are useful to know.

## What is a User Script

A user script is some amount of javascript (JS) that is loaded adjacent to a websites regular JS,
allowing for user defined code to act in conjunction with the contents of the page to add,
remove, or modify its content.

Scripts can empower you to shape already existing websites to make them more convenient
in the manners mentioned above. The main focus of will be discussing
some of the ways in which you can leverage existing web technologies to
create elaborate user scripts to do interesting things.

## User Script Plugins

A user script requires a plugin to manage and load JS onto webpages as they load,
for chrome this can be done with [tampermonkey](https://www.tampermonkey.net/index.php?locale=en)
which this is based on.

### Getting Started

Lets start by creating a basic project.

```bash
mkdir -p src/{website-one,utils}
```

Our project should not look like this.

```none
src
├── website-one
└── utils
```

Next we can write our first user script.

```bash
touch src/website-one/say-hello.js
```

```js
// ==UserScript==
// @name         website-one/say-hello
// @namespace    roland
// @version      0.1
// @description  says hello world
// @author       Roland Warburton
// @match        https://website-one.com/*
// @grant        none
// ==/UserScript==

document.addEventListener('load', main(), {
  once: true
});

async function main() {
  console.log("running say-hello");
}
```

Save this into tampermonkey, load the website in the `match` block,
and check the console to see your hello world!

A userscript is constructed with a preamble between the `==UserScript==`
fence that appears at the top of the page. You can learn more about what values you can
use [here](https://www.tampermonkey.net/documentation.php?locale=en).

## Bundling

Next lets take our scripts a step further and implement other peoples code by bundling it.

The job of a bundler is to take all dependencies and assets and turn then into something
production ready, usually meaning software deployed to a server, or a websites client code.

We can expand the scope of a bundler beyond these two major uses to suite our user scripts too.

For our user script purposes we need our bundler to do two things.

First, it needs to resolve all the library dependencies,
and be able to bundle them into each user script such each JS file has access to its
required library code when its running.

Secondly it should transform any non browser friendly code
into something our browser can understand.

## The First (plain old) User Script Again

Lets solve bundling with esbuild, which provides a library of its own
which allows us to write code to describe how to build our user scripts.

First we will need to install esbuild.

```bash
# if not done already init your package.json
npm init -y

# install esbuild
npm install -D esbuild

# create a build script
touch build.js
```

Our project should not look like this.

```none
├── build.js
├── package.json
├── package-lock.json
└── src
    ├── website-one
    │   └── say-hello.js
    └── utils
```

Lets write our build script (first iteration).

```js
const fs = require('fs');
const path = require('path');
const { build } = require('esbuild');

function makeTemp(name) {
  if (!fs.existsSync(`./${name}`)) {
    fs.mkdirSync(`./${name}`);
  }
}

// returns a list of files in a directory non-recursively
function readFilesInDirectory(directoryPath, filePaths) {
  const files = fs.readdirSync(directoryPath, { withFileTypes: true });

  for (const file of files) {
    if (file.isFile()) {
      const filePath = path.join(directoryPath, file.name);
      filePaths.push(filePath);
    }
  }
}

// run esbuild on the file
async function buildScript(filepath ) {
  const result = await build({
    entryPoints: [filepath],
    bundle: true,
    write: false,
    format: 'esm',
    jsx: 'preserve',
  }).catch((err) => {
    console.log(err)
    process.exit(1)
  });

  // read the result from esbuild
  const content = result.outputFiles[0].text;
  parsedPath = path.parse(filepath);
  fs.writeFileSync(`temp/${parsedPath.base}`, content);
}

function main() {
// our built files will go here
  makeTemp('temp');
  const filePaths = [];

  const entryPoints = ['./src/website-one'];

  // for each website
  for (const entryPoint of entryPoints) {
    readFilesInDirectory(entryPoint, filePaths);
    // for each script
    for (const p of filePaths) {
      const userScriptContent = extractUserScriptContent(p);
      buildScript(p, userScriptContent);
    }
  }
}

main();
```

In this script we create a folder to put our bundled files `temp`,
and iterate over each domain `website-one`, and process each script for it `say-hello.js`.

This will output some nice bundled JS that looks pretty much the same as our input JS.

```js
// src/website-one/say-hello.js
document.addEventListener("load", main(), {
  once: true
});
async function main() {
  console.log("running say-hello");
}
```

But we have a problem! Our output does not include our user script preamble at the top of the file.
Lets fix that.

Lets modify our build script (second iteration) by adding the `extractUserScriptContent` function.

```js
const fs = require('fs');
const path = require('path');
const { build } = require('esbuild');

function makeTemp(name) {
  if (!fs.existsSync(`./${name}`)) {
    fs.mkdirSync(`./${name}`);
  }
}

function readFilesInDirectory(directoryPath, filePaths) {
  const files = fs.readdirSync(directoryPath, { withFileTypes: true });

  for (const file of files) {
    if (file.isFile()) {
      const filePath = path.join(directoryPath, file.name);
      filePaths.push(filePath);
    }
  }
}

// Add this new function
function extractUserScriptContent(filePath) {
  try {
    const fileContent = fs.readFileSync(filePath, 'utf8');
    const startIndex = fileContent.indexOf('// ==UserScript==');
    if (startIndex !== -1) {
      const endIndex = fileContent.indexOf('// ==/UserScript==', startIndex);
      if (endIndex !== -1) {
        const extractedContent = fileContent.substring(startIndex, endIndex + 18);
        return extractedContent.trim();
      }
    }
    return '';
  } catch (err) {
    console.error('Error reading file:', err);
    return '';
  }
}

async function buildScript(filepath, userScriptContent) {
  const result = await build({
    entryPoints: [filepath],
    bundle: true,
    write: false,
    format: 'esm',
    jsx: 'preserve',
  }).catch((err) => {
    console.log(err)
    process.exit(1)
  });

  const content = result.outputFiles[0].text;
  parsedPath = path.parse(filepath);
  fs.writeFileSync(`temp/${parsedPath.base}`, userScriptContent + '\n\n' + content);
}

function main() {
// our built files will go here
  makeTemp('temp');
  const filePaths = [];

  const entryPoints = ['./src/website-one'];

  // for each website
  for (const entryPoint of entryPoints) {
    readFilesInDirectory(entryPoint, filePaths);
    // for each script
    for (const p of filePaths) {
      const userScriptContent = extractUserScriptContent(p);
      buildScript(p, userScriptContent);
    }
  }
}

main();
```

Now when we run `build.js` we get our preamble.

```js
// ==UserScript==
// @name         website-one/say-hello
// @namespace    roland
// @version      0.1
// @description  says hello world
// @author       Roland Warburton
// @match        https://website-one.com/*
// @grant        none
// ==/UserScript==

// src/website-one/say-hello.js
document.addEventListener("load", main(), {
  once: true
});
async function main() {
  console.log("running say-hello");
}
```

We are now ready to add dependencies.

## Adding Dependencies

Lets install a library and use it.

```js
npm i lorem-ipsum
```

You will notice that esbuild supports ESM so we will import it using `import THING from PLACE`
syntax instead of `const THING = require(PLACE` syntax)`.

```js
// ==UserScript==
// @name         website-one/say-hello
// @namespace    roland
// @version      0.1
// @description  says hello world
// @author       Roland Warburton
// @match        https://website-one.com/*
// @grant        none
// ==/UserScript==

import { LoremIpsum } from "lorem-ipsum";

document.addEventListener("load", main(), {
  once: true
});

async function main() {
  console.log(loremIpsum());
}
```

Have a look at the size of the output file, `wc -l` says its now 584 lines!
We have successfully bundled the lorem-ipsum library allowing us to call
the library functions in our user script.

## Utilities

Currently we wait for the load event on the document, the run our code.

The load event is fired when the whole page has loaded,
including all dependent resources such as stylesheets, scripts, iframes,
and images ([mdm](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event)).

However lots of pages hydrate their DOM with dynamic content,
we need a way to wait for this content to be available.

```js
// waits for an element to be available by css selector
// resolves with the element
function findElement(base, selector) {
  if (!base) {
    throw new Error('base element was not defined');
  }
  return new Promise((resolve) => {
    const checkForElement = () => {
      const e = base.querySelector(selector);
      if (e) {
        clearInterval(intervalId);
        resolve(e);
      }
    };

    const intervalId = setInterval(checkForElement, 100);
  });
}

// example use
await findElement(document, '.my-class')
```

## Working With Frameworks

The last useful tool would be a nice way to create UI.
Tools like preact are great for this use case
as they are small and portable enough to keep entirely inside a single script
file without it becoming too big.

To assist us we will also use a [preact-island](https://github.com/mwood23/preact-island)
to embed multiple "widgets" on the page, each widget will be a preact controlled DOM
that works independently from other widgets and the rest of the page.

```bash
npm install preact preact-island
```

We also need to modify our build script to support our JSX.

```js
async function buildScript(filepath, userScriptContent) {
  const result = await build({
    entryPoints: [filepath],
    bundle: true,
    write: false,
    format: 'cjs',
    jsx: 'transform',
    loader: {
      '.js': 'jsx'
    },
    jsxFactory: 'h',
    jsxFragment: "Fragment"
  }).catch((err) => {
    console.log(err)
    process.exit(1)
  });

  const content = result.outputFiles[0].text;
  parsedPath = path.parse(filepath);
  fs.writeFileSync(`temp/${parsedPath.base}`, userScriptContent + '\n\n' + content);
}
```

Next lets write a widget.

```js
async function testWidget() {
  // find something in the DOM to attach the widget to
  const parentElement = await findElement(document, '#attach-to-me');

  // create a widget parent element
  const islandContainer = document.createElement('div');
  islandContainer.setAttribute('data-island', 'widget');
  islandContainer.id = 'header-data-island';
  headerBar.appendChild(islandContainer);

  // construct UI
  const Widget = () => {
    return (
      <div>
        Hello World!
      </div>
    );
  };

  // render the widget as an island
  const island = createIsland(Widget);
  island.render({
    selector: '[data-island="widget"]'
  });
}
```
