# Setup Adonis + Inertia JS react app

## Generate a new app:
npm init adonis-ts-app@latest csr-adonis-inertia-react

❯ Select the project structure · web
❯ Enter the project name · csr-adonis-inertia-react
❯ Setup eslint? (y/N) · true
❯ Setup prettier? (y/N) · true
❯ Configure webpack encore for compiling frontend assets? (y/N) › true

## Install inertia JS 

npm i @eidellev/inertia-adonisjs

## Config inertia JS 

❯ Enter the edge file you would like to use as your entrypoint · app
❯ Would you like to install the Inertia.js client-side adapter? (Y/n) · true
❯ Would you like to use SSR? (y/N) · false
❯ Which client-side adapter would you like to set up? · @inertiajs/inertia-react

## Register inertia middleware
Add Inertia middleware to start/kernel.ts:
<!-- what does kernel.ts in AdonisJS do -->

```javascript
Server.middleware.register([
  () => import('@ioc:Adonis/Core/BodyParser'),
  () => import('@ioc:EidelLev/Inertia/Middleware'),
]);
```
<!-- can you summarise what the Inertia middleware from EidelLev does, can we link to the source and explain? -->
## Configure Webpack-Encore for React in Typescript 

By default Encore (the asset bundler provided to us by Adonis) is configured for Javascript, but since we want to use the same TS throughout our app, let's configure it for that.

Install ts-loader and @babel/preset-react so encore knows how to handle Typescript files and JSX syntax.
```bash
npm install ts-loader @babel/preset-react --save-dev
```
<!-- what does the TS loader do & what does preset-react do, can we link to more detailed explainer -->

Enable support for the 'app.js' as an entry point, edit `webpack.config.js` changing the following into:
```javascript
Encore.addEntry('app', './resources/js/app.js')
```
<!-- what is an entry point & why do we need one for app.js? -->

```javascript
Encore.addEntry('app', './resources/js/app.tsx')
Encore.enableTypeScriptLoader()
Encore.enableReactPreset()
```
<!-- what does each of these Encore methods setup, break it down in detail -->

Rename `/resources/js/app.js` to `/resources/js/app.tsx` to match our previous changes.

Configure typescript for our client side code. Create a file called /resources/js/tsconfig.json and paste this minimal config in it:
```json
{
  "include": ["**/*"],
  "compilerOptions": {
    "lib": ["DOM"],
    "jsx": "react",
    "esModuleInterop": true
  }
}
```

Install react & supporting libraries:
```bash
npm i react react-dom @types/react @types/react-dom
```