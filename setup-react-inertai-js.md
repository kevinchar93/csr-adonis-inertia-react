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
