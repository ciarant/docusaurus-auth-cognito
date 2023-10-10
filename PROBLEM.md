# Introduction
This repo is a fork of the one Massoud Maboudi shared in his [Docusaurus Authentication using AWS Cognito](https://iammassoud.net/blog/docusaurus-authentication-using-aws-cognito) blog post. I've forked it to help get to the bottom of an issue I'm having when I introduce [docusaurus-plugin-openapi-docs](https://docusaurus-openapi.tryingpan.dev/) into the equation.

# Reproducing the Problem
## Create an env file
```
cp .env.local.sample .env.local
```
Note: Reproducing the error doesn't require that you change any of the values in the `.env.local`.

## Run development server
The following works just fine.
```bash
yarn install
yarn docusaurus gen-api-specs all
yarn start
```

## Build for production
The following fails.
```bash
yarn build
yarn run v1.22.19
$ docusaurus build
[INFO] [en] Creating an optimized production build...

✔ Client
  

✖ Server
  Compiled with some errors in 1.22m



Error: Not supported
    at Array.map (<anonymous>)
[WARNING] {"moduleIdentifier":"/Users/ciaran/code/docusaurus-auth-cognito/node_modules/aws-crt/dist/native/binding.js","moduleName":"./node_modules/aws-crt/dist/native/binding.js","loc":"92:18-31","message":"Critical dependency: the request of a dependency is an expression","compilerPath":"server"}
[ERROR] Unable to build website for locale en.

● Client █████████████████████████ cache (99%) shutdown IdleFileCachePlugin
 resolve build dependencies

● Server █████████████████████████ cache (99%) shutdown IdleFileCachePlugin
 process pending cache items

[ERROR] Error: Failed to compile with errors.
    at /Users/ciaran/code/docusaurus-auth-cognito/node_modules/@docusaurus/core/lib/webpack/utils.js:180:24
    at /Users/ciaran/code/docusaurus-auth-cognito/node_modules/webpack/lib/MultiCompiler.js:554:14
    at processQueueWorker (/Users/ciaran/code/docusaurus-auth-cognito/node_modules/webpack/lib/MultiCompiler.js:491:6)
    at process.processTicksAndRejections (node:internal/process/task_queues:77:11)
[INFO] Docusaurus version: 2.4.3
Node version: v18.18.0
error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

# Solution
`yarn install` then delete the following files
- `node_modules/@aws-amplify/auth/lib/urlListener.d.ts`
- `node_modules/@aws-amplify/auth/lib/urlListener.js`
- `node_modules/@aws-amplify/auth/lib/urlListener.js.map`
- `node_modules/@aws-amplify/auth/lib/urlListener.native.js`
- `node_modules/@aws-amplify/auth/lib/urlListener.native.js.map`
- `node_modules/@aws-amplify/auth/src/urlListener.native.ts`
- `/node_modules/@aws-amplify/auth/src/urlListener.ts`

Comment out the following lines in these files
- `node_modules/@aws-amplify/auth/lib/Auth.js`
- `node_modules/@aws-amplify/auth/src/Auth.ts`

```
var usedResponseUrls_1 = {};
urlListener_1.default(function (_a) {
    var url = _a.url;
    if (usedResponseUrls_1[url]) {
        return;
    }
    usedResponseUrls_1[url] = true;
    _this._handleAuthResponse(url);
});
```
Add `patch-package` as a dev dependency
- `yarn add --dev patch-package`

Create a patch file
- `npx patch-package @aws-amplify/auth`

Add the following to the `scripts` section in `package.json`
- `"postinstall": "patch-package"`