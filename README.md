# How to Setup React as a view layer for AdonisJS, using InertiaJS

Simple guide to setup React as the view layer for AdonisJS using InertiaJS (https://github.com/eidellev/inertiajs-adonisjs).

This is a rough setup guide, I intend to do a more fleshed out blog post style guide in future, or maybe even a generator.

Feel free to open a PR or issue with comments / suggestions.

This repo is a working example, do the following to see it in action:
```
git clone git@github.com:kevinchar93/csr-adonis-inertia-react.git
cd csr-adonis-inertia-react
npm install
node ace serve --watch
```
Navigate to http://127.0.0.1:3333 

Based on:
- https://dev.to/eidellev/getting-started-with-adonisjs-and-inertia-js-2po0
- https://github.com/eidellev/inertiajs-adonisjs
- https://adocasts.com/lessons/adding-inertiajs-to-a-new-adonisjs-project

## 1. Generate a new app:

```
npm init adonis-ts-app@latest csr-adonis-inertia-react

❯ Select the project structure · web
❯ Enter the project name · csr-adonis-inertia-react
❯ Setup eslint? (y/N) · true
❯ Setup prettier? (y/N) · true
❯ Configure webpack encore for compiling frontend assets? (y/N) › true
```

## 2. Install & configure Inertia JS provider for AdonisJS

```
npm i @eidellev/inertia-adonisjs

node ace configure @eidellev/inertia-adonisjs

❯ Enter the edge file you would like to use as your entrypoint · app
❯ Would you like to install the Inertia.js client-side adapter? (Y/n) · true
❯ Would you like to use SSR? (y/N) · false
❯ Which client-side adapter would you like to set up? · @inertiajs/inertia-react
```

## 3. Register Inertia middleware

Add Inertia middleware to `start/kernel.ts`:
<!-- what does kernel.ts in AdonisJS do? -->

```javascript
Server.middleware.register([
  () => import('@ioc:Adonis/Core/BodyParser'),
  () => import('@ioc:EidelLev/Inertia/Middleware'),
]);
```
<!-- can you summarise what the Inertia middleware from EidelLev does, can we link to the source and explain? -->
## 4. Configure Webpack-Encore for React in Typescript 

By default Webpack Encore (AdonisJS asset bundler) is configured for JS,  we want to use TS in our app, let's configure support for TS.

Install `ts-loader` and `@babel/preset-react` so Encore knows how to bundle Typescript files and JSX syntax.
```bash
npm install ts-loader @babel/preset-react --save-dev
```
<!-- what does the TS loader do & what does preset-react do, can we link to more detailed explainer -->

Modify the entrypoint, edit `webpack.config.js` changing the following:
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
<!-- what does each of these Encore methods setup? break it down in detail -->

Rename `./resources/js/app.js` to `./resources/js/app.tsx`.

Create a file `./resources/js/tsconfig.json`, with contents:
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


## 5. Configure the app entry point

Running the config script earlier for the Inertia JS Provdier _(`node ace configure @eidellev/inertia-adonisjs`)_ should have generated an file `resources\views\app.edge` its contents should be:
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

Now configure the app entrypoint file `resources/js/app.tsx` to contain:
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

## 6. Create a test component

Create a test component `resources/js/Pages/Test.tsx`, with contents:
```javascript
import React from 'react'

const Test = ({exampleProp}) => <div>Hello world, from {exampleProp}!)</div>

export default Test
```


## 7. Create a test route
<!-- ## 7. Create a test route -->


Create a test route `start/routes.ts`, contents:
```javascript
import Route from '@ioc:Adonis/Core/Route'

Route.get('/test', async ({ inertia }) => {
  return inertia.render('Test', { exampleProp: 'inertia' })
}) 
```

Boot up your server with `node ace serve --watch` & navigate to http://127.0.0.1:3333/test

Voilà! ...you should now have a page rendered using **React** + **Inertia JS** served by your **Adonis JS** application.
