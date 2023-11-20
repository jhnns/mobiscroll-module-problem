## What's the problem?

`@mobiscroll/datepicker-react` cannot be used in Node 20 in native ECMAScript modules.

## Steps to reproduce

1. Set up `.npmrc` file with your auth token
2. Run `node index.js`
3. You should see an error:

```sh
import {localeDe} from "@mobiscroll/datepicker-react";
        ^^^^^^^^
SyntaxError: Named export 'localeDe' not found. The requested module '@mobiscroll/datepicker-react' is a CommonJS module, which may not support all module.exports as named exports.
CommonJS modules can always be imported via the default export, for example using:

import pkg from '@mobiscroll/datepicker-react';
const {localeDe} = pkg;

    at ModuleJob._instantiate (node:internal/modules/esm/module_job:122:21)
    at async ModuleJob.run (node:internal/modules/esm/module_job:188:5)
    at async DefaultModuleLoader.import (node:internal/modules/esm/loader:228:24)
    at async loadESM (node:internal/process/esm_loader:40:7)
    at async handleMainPromise (node:internal/modules/run_main:66:12)

Node.js v20.5.1
```

## How to fix

1. Change `scripts` in `package.json` to:

```json
  "scripts": {
    "postinstall": "patch-package"
  },
```

2. Run `npm install` again. This will apply [`patches/@mobiscroll+datepicker-react+5.27.3.patch`](patches/@mobiscroll+datepicker-react+5.27.3.patch) using [patch-package](https://www.npmjs.com/package/patch-package)

## Explanation

Node does not understand the `module` field in the `package.json`. When importing `@mobiscroll/datepicker-react` from ECMAScript modules, it will use the CommonJS build of Mobiscroll. This is basically fine as you can import CJS into ESM. The problem arises because CommonJS modules can't have named exports (such as `localeDe`). [Node tries to statically analyze the module in order to allow named imports](https://2ality.com/2022/10/commonjs-named-exports.html), but this seems to fail with this specific CJS build (maybe because it's essentially UMD and thus harder to analyze statically?).

By specifying the new [`exports`](https://nodejs.org/api/packages.html#conditional-exports) field we can make sure that Node uses the ESM build. However, this will still fail because Node expects all files within `@mobiscroll/datepicker-react` to be CommonJS. By adding a separate `package.json` just inside the `esm5` folder with `"type": "module"`, we can switch the whole `esm5` folder to ESM.