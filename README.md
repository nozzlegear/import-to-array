# import-to-array
A small utility function that converts ES6 import objects to an array of its own property values, complete with TypeScript defintions.

## Installation

Install from NPM with Yarn or NPM:

```bash
yarn add import-to-array

# Or use NPM
npm i import-to-array --save
```

## Usage

The main point of this package is to convert an imported object to an array of its own property values, while filtering out the `__esModule` properties that TypeScript adds, and while maintaining TypeScript definitions for the array values.

```ts
import importToArray from "import-to-array";
import * as SomeObject from "./path/to/module";

// SomeObject looks like { hello: "world", foo: 1, bar: false, __esModule: any }

// Convert the import to an array of its properties
const array = importToArray(SomeObject);

// Array looks like ["hello", 1, false]
// Array has TypeScript type (string | number | bar)[];
```

In this example, the `array` variable would look like `["hello", 1, false]` and wouldn't include the `__esModule` value. If you're using TypeScript, the variable with have type `(string | number | bar)[]`.

## Real world usage

In [Gearworks](https://github.com/nozzlegear/gearworks) we have a `routes` folder that contains separate API route modules for an Express server. Each route module exports a single function called `registerTopicRoutes` that must be called by the server to, well, register those routes. 

The problem was that we had to remember to add an import for each route in the `server.ts` module and then also remember to actually call its function. We simplified the whole process by using [Barrelsby](https://github.com/bencoveney/barrelsby) to automatically export all of those routes from the `routes` folder, which lets us import all of them at once in the `server` module.

That helped a lot, as it let us map the imported object's keys and then iterate over those keys to register the routes:

```ts
import * as routeRegistrations from "./routes";

await Promise.all(Object.keys(routeRegistrations).map(async propName => await routeRegistrations[propName](arg1, arg2, etc)));
```

While this was a big improvement, that presented one other problem: in TypeScript, the `routeRegistrations[propName]` had a type of `any`, which could come back to bite us if we ever changed the signature of one of the route registration functions. Luckily, TypeScript soon introduced the `Record` type, which lets us take an argument and use its properties as a generic type. 

That is to say, we can pass in the `routeRegistrations` object and automatically know the type of all of its property values:

```ts
import * as routeRegistrations from "./routes";
import importToArray from "import-to-array";

await Promise.all(importToArray(routeRegistrations).map(async registerRoute => {
    // registerRoute has type: (path: string, requiresAuth: boolean, routingFunction: RoutingFunction) => Promise<void>;

    return await registerRoute(path, requiresAuth, routingFunction);
}))
```