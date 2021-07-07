# Typescript Basics

This note will be an "as i go" documentation of typescript in an example project.

## The Goal

1. Set up [typescript](https://github.com/microsoft/TypeScript) for compiling TS to JS
2. Set up [ExpressJS](https://github.com/expressjs/express)
3. Set up [nodemon](https://github.com/remy/nodemon) and [ts-node](https://github.com/TypeStrong/ts-node) to JIT compile
4. Set up [prettier](https://github.com/prettier/prettier)
5. Set up [ESLint](https://github.com/eslint/eslint)
6. Integrate all the things with npm scripts through the cli, and VSC plugins
7. Complete an express boilerplate with types
8. Set up some tests using [jest](https://github.com/facebook/jest)

[Source code](https://github.com/RolandWarburton/learningTS) can be found here.

## Installing Dependencies

Assuming you have a project directory already created.

```bash
# its a good idea to have these globally available
npm install -g prettier typescript nodemon
```

Next install nodemon and typescript into the project locally as well, along with all the other dependencies.

* **@types/express** will provide types for express
* **@types/node** will provide types for nodes APIs
* **ts-node** will provide a JIT compiler for us to use with nodemon, so we don't need to rely on the `tsc` command to run, instead ts-node will work in the background to compile our project for us as we develop.

```none
npm install -D nodemon typescript @types/express @types/node ts-node
```

## Set up TypeScript for Compiling

First lets create the basic directories that we will need.

```bash
# source ts files go here that we will watch and edit
mkdir src

# production ready output js files go here for serving in production
mkdir dist

# tsconfig will go here
touch tsconfig.json

# our entry point typescript file
touch src/index.ts
```

Next lets configure `tsconfig.json`. You can run `tsc --init` to generate this file, i have uncommented the parts that are useful and deleted almost the rest.

The only value i left in commented was **allowSyntheticDefaultImports** because ive had some issues with `import x from y` (ESM) vs `const x = require(y)` (CJS) imports.

```json
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig.json to read more about this file */

    /* Basic Options */
    "target": "ES2020",                                /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', 'ES2021', or 'ESNEXT'. */
    "module": "commonjs",                           /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'es2020', or 'ESNext'. */
    "sourceMap": true,                           /* Generates corresponding '.map' file. */
    "outDir": "./dist",                              /* Redirect output structure to the directory. */
    "rootDir": "./src",                             /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */
    "downlevelIteration": true,                  /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */

    /* Strict Type-Checking Options */
    "strict": true,                                 /* Enable all strict type-checking options. */
    "noImplicitAny": true,                       /* Raise error on expressions and declarations with an implied 'any' type. */
    "strictNullChecks": true,                    /* Enable strict null checks. */
    "strictFunctionTypes": true,                 /* Enable strict checking of function types. */
    "strictBindCallApply": true,                 /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
    "strictPropertyInitialization": true,        /* Enable strict checking of property initialization in classes. */
    "noImplicitThis": true,                      /* Raise error on 'this' expressions with an implied 'any' type. */
    "alwaysStrict": true,                        /* Parse in strict mode and emit "use strict" for each source file. */

    /* Additional Checks */
    "noUnusedLocals": true,                      /* Report errors on unused locals. */
    "noImplicitReturns": true,                   /* Report error when not all code paths in function return a value. */
    "noFallthroughCasesInSwitch": true,          /* Report errors for fallthrough cases in switch statement. */
    "noImplicitOverride": true,                  /* Ensure overriding members in derived classes are marked with an 'override' modifier. */
    "noPropertyAccessFromIndexSignature": true,  /* Require undeclared properties from index signatures to use element accesses. */

    /* Module Resolution Options */
    "moduleResolution": "node",                  /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */
    "esModuleInterop": true,                        /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
    // "allowSyntheticDefaultImports": true,        /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */

    /* Advanced Options */
    "skipLibCheck": true,                           /* Skip type checking of declaration files. */
    "forceConsistentCasingInFileNames": true        /* Disallow inconsistently-cased references to the same file. */
  }
}
```

Our project structure should look like this

```none
├── dist
├── node_modules
│   └── ...
├── package.json
├── package-lock.json
├── src
│   └── index.ts
└── tsconfig.json
```

## Configure package.json

Next we need to add some scripts to use in `package.json`.

* **start** think of this as running the production ready build
* **dev** we will use this command most of the time for developing
* **build** think of this as bundling the app for production in webpack

The use of the **dev** command allows us to make use of `ts-node` to compile and run our typescript without worrying about compiling too much. ts-node will use the tsconfig.json that we created earlier and will handle things in a just-in-time (JIT) compiler that compiles code as you go.

```json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "nodemon src/index.ts --delay 500 --exec ts-node src/index.ts",
    "build": "tsc -p ."
  },
}
```

## Simple Boilerplate

Lets write some boilerplate express code in `src/index.ts`.

Note how we can use the ESM import `import x from y` to import express (the default export), and the data types that we need for our application to provide type checking with (Application, Request, Response, NextFunction).

```ts
import express, { Application, Request, Response, NextFunction } from 'express';

const app: Application = express();

app.get("/", (req: Request, res: Response, next: NextFunction) => {
    res.status(200).json({message: "hello world"});
})

app.listen(3000, () => {console.log("server running");});
```

Then run `npm run dev` and use the browser [http://localhost:3000](http://localhost:3000) to see our work live.

## Prettier

### Method One

To format the project with prettier, We should already have prettier installed globally via `npm i -g prettier`, also install prettier locally into the project as well. Also, install the `onchange` and `concurrently` packages.

```none
npm i prettier onchange concurrently
```

* The **"prettier:watch"** script watches and formats on file change.
* The **"dev"** script runs prettier:watch and nodemon at the same time

```json
{
  "scripts": {
    "dev": "concurrently \"npm run prettier:watch\" \"nodemon src/index.ts --delay 500 --exec ts-node src/index.ts\"",
    "prettier:watch": "onchange 'src/**/*.ts' -- prettier --write {{changed}}"
  },
}
```

Next lets copy in a .prettierrc file to configure prettier a bit.

```json
// .prettierrc
{
  {
    "useTabs": true,
    "tabWidth": 4,
    "printWidth": 100
  }
}
```

### Method Two

If you are using an editor like VSCode, you can install the prettier plugin, and enable it for typescript files by pressing `f1` and then searching for "*Format Document*" and setting the formatted to prettier when prompted. Also see these settings options to format on save in `settings.json` (if the f1 -> format method doesn't work).

```json
{
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "editor.formatOnType": true,
  "[javascript]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
}
```

## ESLint

Despite a lot of tutorials that ive read that still use tslint, this is deprecated, ESLint is what should be used instead.

We need 3 new dependencies to make ESLint work with TS.

* **eslint** the package itself
* **@typescript-eslint/parser** the parser than enables eslint to read typescript
* **@typescript-eslint/eslint-plugin** the rule set used for linting typescript
* **eslint-config-prettier** plugin for integrating with prettier formatting rules

We will create rules in the rules object, for example setting "[no-console](https://eslint.org/docs/rules/no-console)" to 0 to disable it, this is because we want to use console.log for software that isn't sent to the client.

Remember...

* "off" means 0 (turns the rule off completely)
* "warn" means 1 (turns the rule on but won't make the linter fail)
* "error" means 2 (turns the rule on and will make the linter fail)

```none
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-config-prettier
```

Then create a basic config for eslint.

```none
touch .eslintrc
```

```json
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "plugins": [
    "@typescript-eslint"
  ],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "no-console": 0
  }
}
```

```none
touch .eslintignore
```

```json
node_modules
dist
```

Next add a lint script in `package.json`.

```json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "concurrently \"npm run prettier:watch\" \"nodemon src/index.ts --delay 500 --exec ts-node src/index.ts\"",
    "build": "tsc -p .",
    "lint": "eslint . --ext .ts", // <-- Add this
    "lint:fix": "eslint . --ext .ts --fix" // <-- Add this
  }
}
```

### ESLint plugins

Lets add a plugin as well, [eslint-plugin-node](https://github.com/mysticatea/eslint-plugin-node). More plugins can be found [here](https://github.com/dustinspecker/awesome-eslint).

```none
npm install -D eslint-plugin-node
```

Then add "eslint-plugin-node" to the array of plugins in .eslintrc.

```json
{
  "root": true,
  "env": {
    "node": true,
    "commonjs": true,
    "es6": true
  },
  "parser": "@typescript-eslint/parser",
  "plugins": [
    "@typescript-eslint",
    "eslint-plugin-node" // <-- add this
  ],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "plugin:@typescript-eslint/recommended",
    "prettier", "eslint:recommended"
    "plugin:node/recommended" // <-- add this
  ],
  "rules": {
    "no-console": 0,
    "no-unused-vars": 2,
    "node/file-extension-in-import": ["error", "always"] // <-- now we can add these rules
  }
}
```

### Final ESLint Config

Heres a copy paste friendly "final" config for eslintrc.

```json
{
  "root": true,
    "env": {
    "node": true,
    "commonjs": true,
    "es6": true
  },
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 2020
  },
"globals": {
    "Atomics": "readonly",
    "SharedArrayBuffer": "readonly"
  },
  "plugins": [
    "@typescript-eslint",
    "eslint-plugin-node"
  ],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:node/recommended",
    "prettier", "eslint:recommended"
  ],
  "rules": {
    "no-console": 0,
    "no-unused-vars": 2,
    "node/no-missing-import": 0,
    "node/no-unsupported-features/es-syntax": 0,
    "node/file-extension-in-import": 0,
    "node/prefer-global/buffer": ["error", "always"],
    "node/prefer-global/console": ["error", "always"],
    "node/prefer-global/process": ["error", "always"],
    "node/prefer-global/url-search-params": ["error", "always"],
    "node/prefer-global/url": ["error", "always"],
    "node/prefer-promises/dns": "error",
    "node/prefer-promises/fs": "error"
  }
}
```

### ESLint Run Automatically

Just add this to package.json using the before utilized "onchange" package.

```json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "concurrently \"npm run lint:watch\" \"nodemon src/index.ts --delay 500 --exec ts-node src/index.ts\"",
    "build": "tsc -p .",
    "lint": "eslint . --ext .ts",
    "lint:fix": "eslint . --ext .ts --fix",
    "lint:watch": "onchange 'src/**/*.ts' -- npm run lint" // <-- Add this
  }
}
```

### Integrate ESLint with VSC to run ESLint Automatically

Add the following to VSC settings.json using `F1 -> Open Settings (JSON)` and adding the code snippet.

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.validate": ["javascript"]
}
```

## Intermediate boilerplate

Expanding on the simple boilerplate, the next section will describe a more advanced boilerplate. The intermediate boilerplate will implement the following.

* An index.ts entrypoint that imports an express app to run
* An app.ts module that creates an express server (register middleware and routes)
* Use the typescript interface to **describe** a generic route
* A generic route to **implement** the route interface
* A simple controller to do something when the route is hit
* Implement middleware to catch errors and implement cors

You can find the source for the intermediate project [here](https://github.com/RolandWarburton/learningTS/tree/boilerplate).

The final structure will look like this.

```none
src/
├── index.ts
├── app.ts
├── controllers
│   └── index.controller.ts
├── exceptions
│   └── HttpException.ts
├── interfaces
│   └── routes.interface.ts
├── middleware
│   └── error.middleware.ts
└── routes
    └── index.route.ts
```

![tsDiagram.png](https://i.imgur.com/MWdVzuo.png)

## Testing

### Jest Tooling

I've had **MAJOR** issues getting the VSC jest plugin working. Instead of relying on an plugin for VSC i just use `jest --watch` to watch for new tests (compared to previous commit) or `jest --watchAll` to watch all my tests from a terminal. I Always have a separate (not VSC intergrated) terminal open next to my project because it spreads out information across the screen.

![Picture of a dedicated terminal](https://i.imgur.com/txzX04y.png)

### Mocking Controllers

Lets walk through some basic unit tests for the following controller.

```ts
class IndexController {
  public index = (req: Request, res: Response, next: NextFunction): void => {
    try {
      // check if there was a .error field in the request body
      if (!req?.body?.error) {
        res.status(200).json({ hello: "world" });
      } else {
        next(new HttpException(500, "you messed up"));
      }
    } catch (error) {
      next(error);
    }
  };
}
```

Here are the unit tests to just assert test the basic functionality of the above controller.

* We get 200 status back and a JSON object that contains expected data.
* We can call the `next()` middleware function on an error.
* We do not throw an error when passing JSON in the Request body.

With these simple tests you should be able to research and develop future actual practical tests for a real application.

```ts
import IndexController from "../controllers/index.controller";
import { Request, Response, NextFunction } from "express";

describe("test", () => {
  // declare these partials outside of the test scope so they are available within the closure of the test at all times.
  // A partial is a typescript feature that means the mocked object will be a subset of the object it is pretending to be
  // 		Later on when we pass the mockRequest and mockResponse to the controller we coerce them into their
  // 		type using the (mockRequest as Request), "as" keyword so the controller is happy with the types
  let mockRequest: Partial<Request>;
  let mockResponse: Partial<Response>;
  let nextFunction: Partial<NextFunction>;

  // we also create these objects to store the result of the .status and .json that express gives to us when we mock
  // 		the implementation of the .status and .json functions that exist inside the controller
  let resultJson = {};
  let resultStatus = {};

  beforeEach(() => {
    // reset the request and response objects before each test
    mockRequest = {};
    mockResponse = {};

    // reset the objects that store results in them
    resultJson = {};
    resultStatus = {};

    // The mockImplementation catches the .status function when it is called in express,
    // 	then runs the below test code to "implement" what the function will do.
    //
    // To test express there is a special requirement, that is to return the "mockResponse" object every time,
    // 		this mimics the "function chaining" behavior of express where we call .status().json({}) in the controller
    //
    // the .mockImplementation((result) => {} "result" variable is given to us by express and represents
    // 		what the controller and express passed to the .status and .json parts of the chain,
    // 		essentially we just steal that and keep it for our assertions later on in the test
    mockResponse.status = jest.fn().mockImplementation((result) => {
      resultStatus = result;
      return mockResponse;
    });

    mockResponse.json = jest.fn().mockImplementation((result) => {
      resultJson = result;
      return mockResponse;
    });

    // Create a jest function that we can pass as the nextFunction to the controller
    // 		If the controller reaches an error state, then the next() function will be called
    // 		and we will be able to assert that by calling the .toHaveBeenCalledTimes() method
    nextFunction = jest.fn();
  });

  it("returns 200 and the expected json body", () => {
    const controller = new IndexController().index;
    // run the index controller with the req of {} and wait for the response
    controller(mockRequest as Request, mockResponse as Response, nextFunction as NextFunction);

    // when the res.status is called we expect it to be passed 200
    expect(resultStatus).toBe(200);

    // when the res.json is called we expect it to have the body json from the controller
    expect(resultJson).toEqual({ hello: "world" });
  });

  it("calls the next function when the json body.error field is passed to it", () => {
    const controller = new IndexController().index;

    mockRequest.body = { error: "please" };
    controller(mockRequest as Request, mockResponse as Response, nextFunction as NextFunction);

    // check that the next function has been called
    expect(nextFunction).toHaveBeenCalledTimes(1);
  });

  it("does not throw an error when json is passed to it", () => {
    const controller = new IndexController().index;

    mockRequest.body = { noError: "please" };
    controller(mockRequest as Request, mockResponse as Response, nextFunction as NextFunction);

    // check that the next function has been called
    expect(nextFunction).toHaveBeenCalledTimes(0);
  });
});
```

Lets look at the beginning of the suite. A `Partial` is a typescript feature that allows us to say "this object will contain a subset a class".

* A partial of a `Request` and `Response` from express will be used to mock the req and res
* We will also create a `next` function thats a partial of NextFunction from express

The last line `jest.fn()` is a mock function, think of jest.fn as wrapping the function in jest to let you make queries about the state of that function when it is called, for example what arguments was passed to it `next.mock.calls[0]`, and how many times it was called in the function that you are testing `next.mock.calls.length`.

```ts
describe("test", () => {
  let mockRequest: Partial<Request>;
  let mockResponse: Partial<Response>;
  const nextFunction: Partial<NextFunction> = jest.fn();
```

#### A bit More on Mocking Implementations

The purpose of `jest.fn()` is to create a function that we can mock by exposing parts of the application controller to our test.
In the simple example where we call .mockImplementation we are literally adding an extra hidden step that the regular express controller does not see, in other words the real express `.status()` is called, our `jest.fn().mockImplementation` is also called and is passed the exact same data that the real `.status()` is, therefore allowing us to steal or spy on the data coming from the controllers method.

Also for the record a mocked/mock function == function spies as far as jest is concerned.

```ts
mockResponse.functionWeWantToReturn = jest.fn().mockImplementation((result) => {
  // We have some logic here for our test that has access to the `result` that the real controller
  // passed to the .functionWeWantToReturn method.

  // In this case we want to just capture the result passed to us
  resultStatus = result;

  // Also we need to add this line to the test mocked implementation
  //    because it needs to mock how the real express controller actually acts.
  //    The real express .functionWeWantToReturn method (think of .json or .status) are chained methods
  //    and a chain method returns itself through the chain, so thats what the below line does to make the test work.
  return mockResponse;
}
```

An alternate solution (that you should not use) is below. How this works (i think) is that instead of cracking open the entire function to write some code and steal the result from it we instead do these two things.

* We get the mockResponse (right hand side returned from express) and return it (to satisfy the chained nature of express)
* At the same time steal the .status from the and store it in our own (left hand side) `mockResponse.status` which works because before jest.fn() ran the real `express.status` ran and populated it with the correct status code.

The reason why **NOT** to use this code is because of the "recursive-ish" parameter passing, which makes it way more confusing than building a constant object in general.

```ts
//            because of express running before jest.fn this object already has .status in it
//                                                 |
mockResponse.status = jest.fn().mockReturnValue(mockResponse);
//             |
//          now we can steal what the mocked return value contains
//          and store it in our own local .status
```

#### A bit More on Jest functions

Here is another simpler example using just `jest.fn()` by itself. In this example we convert a typed function into a jest wrapped function using `jest.fn`.

```ts
// the function we want to test
const foo = (): string => {
  return "hello roland";
};

// convert foo into mockedFoo
const mockedFoo = jest.fn(foo);

test("my mocked function", () => {
  // Run the function!
  const result = mockedFoo();

  // expect that it returned the string
  expect(result).toBe("hello roland");

  // expect that we called the function once
  expect(mockedFoo.mock.calls).toHaveLength(1);

  // This function was instantiated exactly once
  expect(mockedFoo.mock.instances.length).toBe(1);
});
```

## OpenAPI and Swagger

OpenAPI is a YAML and JSON standard for standardizing APIs. Swagger developed the OpenAPI standard for public use and makes money by developing tools for generating OpenAPI spec files.

OpenAPI resources that i looked at for setting up API documentation.

* [Swagger UI Express](https://www.npmjs.com/package/swagger-ui-express) To serve the swagger UI based on an OpenAPI spec.
* [Swagger JSDoc](https://www.npmjs.com/package/swagger-jsdoc) To convert jsdoc comments into an OpenAPI spec.
* [Swagger Inspector](https://inspector.swagger.io) Create OpenAPI definitions semi-automatically through HTTP requests and analysing the response.
* [Stoplight Studio](https://stoplight.io/studio/) A paid for (with a free tier) tool for graphically creating an OpenAPI spec.

### Setting up Swagger UI Express

We will add a new step in `app.ts` that initializes Swagger. Make sure to place the docs before the routes initializers to fix any routing issues.

```ts
import openApiSettings from "./openApiSettings";
// eslint-disable-next-line node/no-unpublished-import
import swaggerUi from "swagger-ui-express";
// eslint-disable-next-line node/no-unpublished-import
import swaggerJsdoc from "swagger-jsdoc";

class App {
  public app: express.Application;
  public port: string | number;
  public env: boolean;

  constructor(routes: Route[]) {
    this.app = express();
    this.port = process.env["PORT"] || 3000;
    this.env = process.env["NODE_ENV"] === "production" ? true : false;

    this.initializeMiddlewares();
    this.initilizeDocs(); // <-- Add this
    this.initializeRoutes(routes);
    this.initializeErrorHandling();
  }

  // Create the swagger UI for openapi documentation
  // This step MUST come before the initialize routes
  private initilizeDocs() {
    // use swagger-jsdoc to get openapi scheme
    const openapiSpecification = swaggerJsdoc(openApiSettings);

    // or import an openapi spec
    // const fs.readFileSync("../swagger.json", {format: "utf-8"})

    this.app.use("/api", swaggerUi.serve, swaggerUi.setup(openapiSpecification));
  }

}
```

### Using a Vanilla Swagger Spec JSON

If we were not relying on swagger-jsdoc to generate our spec file based on route comments. We instead must just load in a `swagger.json` we wrote ourselves, for example.

**However** there is a better way! By suing [Stoplight Studio](https://stoplight.io/studio/) you can graphically and much easily create spec files and generate documentation much faster thanwriting by hand. Also OpenAPI is the biggest waste of time to manually write because its complicated, long-winded, and barely designed for human readability in my opinion.

```json
{
  "openapi": "3.0.3",
  "info": {
    "description": "My Example App",
    "version": "1.0.0",
    "title": "Example App",
    "contact": {
      "name": "Roland",
      "email": "abc@gmail.com"
    }
  },
  // etc etc
  "paths": {
    "/todos": {
      "get": {
        "summary": "Get all the tasks",
        "description": "Get all the tasks",
        "produces": ["application/json"],
        "parameters": [],
        "responses": {
          "200": {
            "description": "successful operation",
            "schema": {
              "type": "array",
              "items": {
                "$ref": "#/definitions/todosResponse"
              }
            }
          },
          "400": {
            "description": "Invalid status value",
            "schema": {
              "$ref": "#/definitions/InvalidResponse"
            }
          }
        }
      }
    }
  }
}
```

### Swagger JSDoc

**NOTE** I decided not to use Swagger-JSDoc because i was not able to get *"Components"* and *"$ref"* working with the plugin.

Here is what `openApiSettings.ts` looks like, we load this in as a base config for swagger-jsdoc.

```ts
export default {
  definition: {
    openapi: "3.0.1",
    info: {
      title: "Hello World",
      version: "1.0.0",
    },
  },
  tags: [
    {
      name: "users",
      description: "operations available to everyone",
    },
  ],
  servers: [
    {
      url: "localhost:3000/api",
      description: "development server",
      variables: {
        port: {
          default: "3000",
        },
      },
      basePath: {
        default: "v1",
      },
    },
    {
      url: "example.rolandw.dev/api",
      description: "Production server",
      variables: {
        port: {
          enum: ["443", "80"],
          default: "443",
        },
      },
      basePath: {
        default: "v1",
      },
    },
  ],
  host: "localhost:3080",
  basePath: "/api",
  apis: ["**/routes/*.ts"],
};
```

Next we use `@openapi` yaml in each of our routes for swagger-jsdoc to collect and use to create the full spec.

```ts
// /src/routes/index.route.ts

import { Router } from "express";
import IndexController from "../controllers/index.controller";
import Route from "../interfaces/routes.interface";

/**
 * @openapi
 * /:
 *   get:
 *     summary: returns root with help
 *     description: returns root with help
 *     tags:
 *       - users
 *     responses:
 *       200:
 *         description: Returns a mysterious string.
 *         content:
 *           application/json:
 */
class IndexRoute implements Route {
  public path = "/";
  public router = Router();
  public indexController = new IndexController();

  constructor() {
    this.initializeRoutes();
  }

  private initializeRoutes() {
    this.router.get(`${this.path}`, this.indexController.index);
  }
}

export default IndexRoute;
```

### OpenAPI Practical Example

After downloading the stoplight app and taking a look at the example project. Lets remove all that and start by creating a new spec first, then implement it into our express app.

For example lets create a request that can be describes as follows:

* The request is a POST to the "/user" endpoint
* The request body must by "application/json"
* This request should be tagged with "general" because its general use (tags just group requests together in documentation)
* The request body must contain an "id" property containing a number
* The response body must by "application/json"
* The response body for HTTP 200 must contain an "id: number", "name: string", and "likes: string" property

This whole process is pretty self explanatory apart from resolving these errors for tagging.

* Operation should have non-empty `tags` array.
* Operation tags should be defined in global tags.

![errors picture](https://i.imgur.com/H1mCHUY.png)

To resolve this use the "tag button" on both the request you are creating, and the "api overview" itself.

* By adding the tag to the "api overview" you are adding it to a global tag list that describes all the possible tags. This resolves the `Operation tags should be defined in global tags` error.
* By adding the tag to the specific "/user" request you are resolving the `Operation should have non-empty tags array` error.

Here is a photo of the request in the GUI. Notice how i used a "model" for the response. OpenAPI supports components like models that allow for code reuse.

![request](https://i.imgur.com/vUEwyo2.png)

Heres a photo of the model.

![model](https://i.imgur.com/vgKGwLq.png)

And finally just in case, heres the fully generated spec.json for easy use.

```json
{
  "openapi": "3.1.0",
  "info": {
    "title": "my test api",
    "version": "1.0",
    "summary": "this is the summary",
    "description": "this is the description",
    "contact": {
      "name": "Roland",
      "email": "warburtonroland@gmail.com"
    }
  },
  "servers": [
    {
      "url": "http://localhost:3000",
      "description": "development"
    }
  ],
  "paths": {
    "/user/:id": {
      "parameters": [],
      "get": {
        "summary": "",
        "operationId": "get-user-:id",
        "responses": {
          "200": {
            "description": "OK",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/User"
                },
                "examples": {
                  "example-1": {
                    "value": {
                      "id": "142",
                      "name": "Roland",
                      "likes": "Chocolate"
                    }
                  }
                }
              }
            }
          }
        },
        "description": "returns a user details based on their ID",
        "tags": [
          "general"
        ]
      }
    }
  },
  "components": {
    "schemas": {
      "User": {
        "title": "User",
        "type": "object",
        "description": "",
        "examples": [
          {
            "id": 142,
            "name": "Roland",
            "likes": "Chocolate"
          }
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Unique identifier for the given user."
          },
          "name": {
            "type": "string"
          },
          "likes": {
            "type": "string"
          }
        },
        "required": [
          "id",
          "name",
          "likes"
        ]
      }
    },
    "securitySchemes": {}
  },
  "tags": [
    {
      "name": "general"
    }
  ]
}
```

Next lets see what codegen we can make using Swagger Hub Codegen. Convert the JSON from Stoplight into YAML, paste it into the editor, and generate some JS code.

```output
.
├── api
│   └── openapi.yaml
├── controllers
│   └── General.js
├── index.js
├── package.json
├── README.md
├── service
│   └── GeneralService.js
└── utils
    └── writer.js

4 directories, 7 files
```

I wont even post the generated code here because i have learnt that it does not do a very good job for typescript projects. After all its not even TS generated code (its vanilla JS). To be fair i did not expect a one click solution that automatically programs the entire API because thats nigh impossible.

TLDR this code looks very bare-bones, thats because it is. It seems that swagger-codegen has barely interpreted our spec. So we should definitely not be relying on this tool for any actual programming. Instead lets implement the `/user/:id` endpoint ourselves.

First we need to define the request shape in `src/models/requests/user_id.ts`.

The request will use a custom type created first called *UserRequest*, and *JSONSchemaType* takes that *UserRequest*.

The purpose of a JSONSchemaType from AJV is to allow a typescript safe schema for AJV to read later (in other words it would be a vanilla object in normal JS). The *JSONSchemaType* ensures that we have properties like the ajv "type" and "properties" field.

Using the "UserRequest" as a generic for "JSONSchemaType", we can also ensure that the "properties" contains only expected fields supported by the "UserRequest" type.

```ts
// src/models/requests/user_id.ts

import { JSONSchemaType } from "ajv";

type UserRequest = {
  id: string;
};

// the "UserRequest" of type JSONSchemaType<UserRequest> is used by JSONSchemaType as a "generic"
// A generic is a way of capturing a user passed in Type (<UserRequest>) that JSONSchemaType can use internally.
// JSONSchemaType will be defined in the Ajv library as something like `interface JSONSchemaType<Type> {...}`
// 		Now JSONSchemaType can use the <UserRequest> type. EG. interface JSONSchemaType<Type> { myArg: Type }
// You can also use generics for functions and classes https://www.typescriptlang.org/docs/handbook/2/generics.html
const schema: JSONSchemaType<UserRequest> = {
  type: "object",
  properties: {
    id: { type: "string", nullable: false },
  },
  required: ["id"],
  additionalProperties: false,
};

// We can now use this schema in the middleware of routes/user.route.ts to validate that a user request is an object that matches...
// {
//  id: string
// }
export default schema;
export { UserRequest }
```

And a model for our response as well. We will just use it to store our "database" of users.

```ts
// src/models/responses/user.ts

import Ajv, { JSONSchemaType } from "ajv";

// interface for implementation of the User object
type User = {
  id: string;
  name: string;
  likes: string;
};

// type guard checking for the user object
const ajv = new Ajv();

const schema: JSONSchemaType<User> = {
  type: "object",
  properties: {
    id: { type: "string", nullable: false },
    name: { type: "string", nullable: false },
    likes: { type: "string", nullable: false },
  },
  required: ["id", "name", "likes"],
  additionalProperties: false,
};

const validateSchema = ajv.compile(schema);

export default User;
export { validateSchema };
```

Then lets create new middleware to assist in checking the body of the request has the shape we specified in the OpenAPI GUI creator.

Here we will implement our own generic `<Type>` to accept an argument of `schema: JSONSchemaType<Type>`. The `<Type>` is defined by the user when calling the function like `validateRequest<UserRequest>("params", userRequestSchema)`, then that type `UserRequest` will be used in place (replacing `<Type>` with `UserRequest`).

```ts
// src/middleware/validateReq.middleware.ts

import { NextFunction, Request, Response, RequestHandler } from "express";
import { ValidateFunction } from "ajv";
import HttpException from "../exceptions/HttpException";

type Value = "body" | "query" | "params";

// when we call this we need to tell it the type for the validate function
// for example in user.route.ts: `validateRequest<UserRequest>("params", new Ajv().compile(mySchema);)`
function validationMiddleware<Type>(
  value: Value,
  validate: ValidateFunction<Type>
): RequestHandler {
  // This is sort of like a factory pattern that returns some middleware function that can validate any <Type> of schema passed to it.
  return (req: Request, res: Response, next: NextFunction) => {
    // run a validation check of the request ("body"/"query"/"params") against the schema we gave it
    if (validate(req[value])) next();
    else next(new HttpException(400, `wrong body from validate middleware`));
  };
}

export default validationMiddleware;

```

Lastly lets create a simple "user lookup" route and its controller to handle some simple request that relies on a `req.body.params.id` to be present.

```ts
// user.route.ts

// This is just ONE route. A new file is created for every route
// The Route contains an userController which is a standard-ish MVC controller
// 		The job for userController is to actually run some code for the request

import { Router } from "express";
import UserController from "../controllers/user.controller";
import Route from "../interfaces/routes.interface";
import userRequestSchema, { UserRequest } from "../models/requests/user_id";
import validateRequest from "../middleware/validateReq.middleware";
import Ajv from "ajv";

// The "userRequestSchema" is a JSONSchemaType for AJV to consume
// The UserRequest is a typescript <Type> for JSONSchemaType

// We need "userRequestSchema" object to compile the AJV validator function
// We need "UserRequest" type to tell the validateRequest middleware
//    what type the validate function should use

class UserRoute implements Route {
  public path = "/user/:id";
  public router = Router();
  public userController = new UserController();
  private validator = new Ajv().compile(userRequestSchema);

  constructor() {
    this.initializeRoutes();
  }

  private initializeRoutes() {
    this.router.get(
      `${this.path}`,
      validateRequest<UserRequest>("params", this.validator),
      this.userController.user
    );
  }
}

export default UserRoute;

```

And our controller that handles our route.

```ts
// src/controllers/user.controller

// This file is a controller that runs some code when a route is hit
// This file is referenced by the user.route.ts ROUTE which uses UserController (this) as its CONTROLLER

import { NextFunction, Request, Response } from "express";
import HttpException from "../exceptions/HttpException";
import User from "../models/responses/user";

class UserController {
  public user = (req: Request, res: Response, next: NextFunction): void => {
    // our user database
    const users: Array<User> = [
      {
        id: "1",
        name: "roland",
        likes: "chocolate",
      },
    ];

    // by the time this controller is called we expect that the values passed in have been validated
    // 		so we can put some very simple logic in here to just find the user and return them
    // if the user is 404 then we send next() a HttpException and Express will pass that to the error middleware
    // 		because HttpException is an Error (extends the error class)
    try {
      const user = users.find((user) => user.id == req.params["id"]);
      if (!user) next(new HttpException(404, "user not found"));
      else res.status(200).json(user);
    } catch (error) {
      // the HTTP exception middleware will read this even if its a standard Error and not a HttpException
      // then if its NOT a HTTP exception it just gets passed along the chain, to either express to handle the error,
      // 		or if you enable the general Error (Error class) middleware, then this will handle it instead.
      next(error);
    }
  };
}

export default UserController;
```

Ok great! Now we have achieved the following things in summary.

1. We created a model of our request
2. We created middleware to validate our request
3. We created a new route that uses provides our model to the validate middleware
4. Our route returns a User object based on our defined model

### Unit and Intergration tests

So far we have written only unit tests. That is that we test just a controller, or a route on its own. But in the real application there is a flow to the code. EG...

* A route is created, **then**
* The route has, and calls its middleware, **then**
* A route calls its controller, **then**
* The controller calls its own ExpressJS next() function, **then**
* The next() function continues to pass the the request/response object down the chain until eventually the client gets a response of some kind

Each one of these dot points can (and should) be tested individually (even next(), by checking it was called by the controller with specific parameters etc).

**(Important)** Because we can only test one unit of code at a time, it is hard to test an entire lifecycle of the request/response for a particular route because we would need to
mock every next() call and that would be tedious and wasteful of our time.

This is what intergration testing aims to solve by combining two "units" of code together.

I like to think about the testing ecosystem within the architecture of C4. The intergration of units come together to form "components". Then we can slot these components together to compose our full application or container. And finally this container of software can make up and ecosystem or "software system".

![c4ModelDiagram](https://res.infoq.com/articles/C4-architecture-model/en/resources/c4-3-1529934729500.jpg)

#### The Rest of The Testing Types

Based on [atlassion] there are also other types of tests that you could research and implement into your software.

* Unit Tests (done)
* Intergration tests (done)
* Functional Tests - Tests that assess business level input and output
* End-to-End Tests - Replicate user behavior in a browser (or whatver the user sees/uses)
* Acceptance Tests - A business checklist to verify business requirements are met
* Performance Tests - Benchmarking software speed and performance

### Unit tests for the User Route

Ok lets also write some unit tests to cover this code.

```ts
import UserController from "../controllers/user.controller";
import User from "../models/responses/user";
import HttpException from "../exceptions/HttpException";
import { Request, Response, NextFunction } from "express";

describe("Test the user controller", () => {
  let mockRequest: Partial<Request>;
  let mockResponse: Partial<Response>;
  let nextFunction: Partial<NextFunction>;

  let resultJson = {};
  let resultStatus = {};

  // This is the same boilerplate that i explained above
  beforeEach(() => {
    // reset the request and response objects before each test
    mockRequest = {};
    mockResponse = {};

    // reset the objects that store results in them
    resultJson = {};
    resultStatus = {};

    // mock the response status
    mockResponse.status = jest.fn().mockImplementation((result) => {
      resultStatus = result;
      return mockResponse;
    });

    // mock the response response
    mockResponse.json = jest.fn().mockImplementation((result) => {
      resultJson = result;
      return mockResponse;
    });

    nextFunction = jest.fn();
  });

  it("returns returns user object", () => {
    const controller = new UserController().user;

    // input to the controller
    mockRequest.params = { id: "1" };

    // expected response
    const expected: User = { id: "1", name: "roland", likes: "chocolate" };

    // run the controller method
    controller(mockRequest as Request, mockResponse as Response, nextFunction as NextFunction);

    // when the res.status is called we expect it to be passed 200
    expect(resultStatus).toBe(200);

    // expect the response to be this user
    expect(resultJson).toEqual(expected);
  });

  it("returns user not found error", () => {
    const controller = new UserController().user;

    // input to the controller
    mockRequest.params = { id: "not a user" };

    // run the controller method
    controller(mockRequest as Request, mockResponse as Response, nextFunction as NextFunction);

    // when the res.status is called we expect it to be passed 200
    expect(resultStatus).toBe(404);

    // expect the response to be empty (IE not modified by the mock)
    expect(Object.entries(resultJson).length).toBe(0);

    // expect the response.json to NOT have been called (by checking is mock properties)
    expect(mockResponse.json).toHaveBeenCalledTimes(0);

    // expect the next middleware to have been called due to the user not being found
    expect(nextFunction).toHaveBeenCalledTimes(1);

    // expect that the next middleware was called with this error
    expect(nextFunction).toHaveBeenCalledWith(new HttpException(404, "user not found"));
  });

  // Note that this logically should not happen because there is no route to the controller that allows no user to be passed
  //      as /user/:id requires a parameter to be set (otherwise the path is just /user)
  // Its up to you if you want to write these tests to handle future situations
  //      when you might decide to implement a route that will practically use this test
  test("nothing is passed", () => {
    const controller = new UserController().user;

    // input to the controller
    mockRequest.params = {};

    // run the controller method
    controller(mockRequest as Request, mockResponse as Response, nextFunction as NextFunction);

    // when the res.status is called we expect it to be passed 200
    expect(resultStatus).toBe(404);
  });
});
```

And then some intergration tests using [SuperTest](https://www.npmjs.com/package/supertest) as an agent to run the server in memory and make requests against it, asserting that collections of routes (the /user/:id route) and controllers (the user controller) act correctly as a whole.

```ts
// src/__tests__/intergrations/user.controller.ts

// NOT DONE
```
