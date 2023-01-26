# Setup Adonis + Inertia JS react app

Simple guide to setup React as the view layer for AdonisJS using InertiaJS.

Based on:
- https://dev.to/eidellev/getting-started-with-adonisjs-and-inertia-js-2po0
- https://github.com/eidellev/inertiajs-adonisjs

## Generate a new app:

```
npm init adonis-ts-app@latest csr-adonis-inertia-react

❯ Select the project structure · web
❯ Enter the project name · csr-adonis-inertia-react
❯ Setup eslint? (y/N) · true
❯ Setup prettier? (y/N) · true
❯ Configure webpack encore for compiling frontend assets? (y/N) › true
```

## Install & configure inertia js adapter for adonis js

```
npm i @eidellev/inertia-adonisjs
```

```
node ace configure @eidellev/inertia-adonisjs

❯ Enter the edge file you would like to use as your entrypoint · app
❯ Would you like to install the Inertia.js client-side adapter? (Y/n) · true
❯ Would you like to use SSR? (y/N) · false
❯ Which client-side adapter would you like to set up? · @inertiajs/inertia-react
```

## Register inertia middleware

Add Inertia middleware to `start/kernel.ts`:
<!-- what does kernel.ts in AdonisJS do -->

```javascript
Server.middleware.register([
  () => import('@ioc:Adonis/Core/BodyParser'),
  () => import('@ioc:EidelLev/Inertia/Middleware'),
]);
```
<!-- can you summarise what the Inertia middleware from EidelLev does, can we link to the source and explain? -->
## Configure Webpack-Encore for React in Typescript 

By default Encore (AdonisJS asset bundler) is configured for JS,  we want to use the TS in our app, let's configure support for TS.

Install `ts-loader` and `@babel/preset-react` so encore knows how to bundle Typescript files and JSX syntax.
```bash
npm install ts-loader @babel/preset-react --save-dev
```
<!-- what does the TS loader do & what does preset-react do, can we link to more detailed explainer -->

Modify the entry point, edit `webpack.config.js` changing the following:
```javascript
Encore.addEntry('app', './resources/js/app.js')
```
<!-- what is an entry point & why do we need one for app.js? -->

into:
```javascript
Encore.addEntry('app', './resources/js/app.tsx')
Encore.enableTypeScriptLoader()
Encore.enableReactPreset()
```
<!-- what does each of these Encore methods setup, break it down in detail -->

Rename `/resources/js/app.js` to `/resources/js/app.tsx`.

Create a file `/resources/js/tsconfig.json` and with contents:
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

## Configure the app entry point

Running the config script for the inertia js adapter _(`node ace configure @eidellev/inertia-adonisjs`)_ should have generated a `app.edge` file its contents should be:
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="icon" type="image/png" href="/favicon.ico">

  @entryPointStyles('app')
  @entryPointScripts('app')

  <title>csr-adonis-inertia-react</title>

</head>

<body>
  @inertia
</body>

</html>
```

Now configure the app entrypoint file `resources/js/app.tsx` to contain the following:
```javascript
import { InertiaApp } from '@inertiajs/inertia-react'
import React from 'react'
import ReactDOM from 'react-dom'
import '../css/app.css'

// initial page object with props from server
const root = document.getElementById('app')
const page = JSON.parse(root.dataset.page)

// dynamically load specified page component from "resources/js/Pages/." dir
async function resolver(pageName) {
  const module = await import(`./Pages/${pageName}`)
  return module.default
}

function App() {
  return <InertiaApp initialPage={page} resolveComponent={resolver} initialComponent={''} />
}

ReactDOM.render(<App />, root)
```

## Create a test component

Create a test component `resources/js/Pages/Test.tsx`, with contents:
```javascript
import React from 'react'

const Test = ({exampleProp}) => <div>Hello world, from {exampleProp}!)</div>

export default Test
```


## Create a test route 

Create a test route `start/routes.ts`, contents:
```javascript
import Route from '@ioc:Adonis/Core/Route'

Route.get('/test', async ({ inertia }) => {
  return inertia.render('Test', { exampleProp: 'inertia' })
}) 
```
