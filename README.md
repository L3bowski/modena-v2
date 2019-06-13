# modena

Domain names are expensive and creating new deployment pipelines/scripts takes a precious time you could be dedicating to something else (e.g. learning [functional programming](https://drboolean.gitbooks.io/mostly-adequate-guide-old/content/)). Instead of having a separate environment for each of your express apps, use modena to expose all of them on a single express server while keeping them isolated from each other.

## What does it do?

Modena scans the apps directory of the project (configurable through the appsPath option) and exposes each existing app in the host app, using the app folder name as unique namespace. Each app can then be accessed using the app name as relative url. Given the following file structure:

```bash
.
├── _apps
│   ├── _app-1
|   |    (...)
|   |    ├── get-express-app.js
|   |    ├── index.js
|   |    ├── package.json
|   |    └── package-lock.json
|   |    (...)
│   ├── _app-N
|   |    (...)
|   |    ├── get-express-app.js
|   |    ├── index.js
|   |    ├── package.json
|   |    └── package-lock.json
|   (...)
├── index.js
├── package.json
└── package-lock.json
```

The apps will then automatically be exposed on the host server by placing the following code on the `index.js` file:

```
const express = require('express');
const { exposeHostedApps, launchServer } = require('modena');

const mainApp = express();

// exposeHostedApps returns a Promise, so that apps can be created asynchronously
exposeHostedApps(mainApp)
    .then(() => launchServer(mainApp, { port: 3000}))
    .catch(console.log);
```

So... that's it? Yes! In a nutshell. Additionally, it offers different methods to access the apps without having to prefix the URLs with the app name, automatically injects environment variables into hosted apps and provides some HTTP/HTTPS functionalities.

## How does it work?

Modena detects all the express applications in the host project folder and uses the express library ability to expose many apps inside another app, as showcased below (on an actual host project, each app is created/defined in its own file and directory, and the app folder name is used as its namespace):

```
const express = require('express');
const express = require('express-session');

/* The app that will actually be listening to network requests*/
const mainApp = express();

const app1 = express();
app1.use(/^\/$/, (req, res, next) => res.send('Application 1'));

const app2 = express();
app2.use(session({ secret: 'keyboard cat' }));
app2.use(/^\/$/, (req, res, next) => res.send(typeof req.session));

mainApp.use(/^\/$/, (req, res, next) => res.send('Main app'));

mainApp.use('/app-1', app1);
mainApp.use('/app-2', app2);

mainApp.listen(3000, error => {
    if (error) {
        console.log(error);
    }
    else {
        console.log('Server listening in port 3000');
    }
});
```

## Getting started

To make an express application able to run on modena, we will need to slightly modify the app code so the creation of the app and the call to the `listen` method happen in separate files (otherwise, when importing the express application, the app itself would start listening on its own).

I recommend creating a file called `get-express-app.js` (modena will search for this file by default, but the name can be configured through the app settings) and move there all the app creation and settings. This file must export a function that returns an express app or a Promise returning an express app.

See the following examples of split app creation and `listen` call files:

-   Regular function (synchronous): https://github.com/L3bowski/modena-v2-examples/tree/master/apps/default-app
-   Promise-returning function (asynchronous): https://github.com/L3bowski/modena-v2-examples/tree/master/apps/promise-app

Once the apps are ready, the first thing we need to do to host them through modena is to create a host project and install modena:

```bash
mkdir new-project
cd new-project
npm init
npm i --save modena
```

Next we need to create an `apps` directory where all the hosted apps will be placed:

```bash
mkdir apps
cd apps
# Place the apps inside the current working folder. E.g: cloning a repo...
git clone <URL of your express app repository>
# ...or copying the files from some other location...
cp <app path> .
# ...or adding the apps as a git submodule (having previously initialized a git repository)...
cd ..
git submodule app <URL of your express app repository> apps/<app name>
```

Finally we just have to create an entry point to run the host app (e.g. `index.js`) and we are ready to go!

```
const express = require('express');
const { exposeHostedApps } = require('modena');

const mainApp = express();

exposeHostedApps(mainApp)
    .then(_ => {
        mainApp.listen(3000, (error) => {
            if (error) {
                console.log('Something went wrong');
            }
            else {
                console.log('Server listening at port 3000');
            }
        })
    })
    .catch(console.log);
```

## App settings

TODO modena.json

## URLs resolution

TODO

-   Domain names mapping (cross access vs isolated)
-   \$modena query string parameter
-   Relative url
-   Default app

## Environment variables injection

TODO App prefixing + link to config-app

## HTTP/HTTPS functionality

## Show me the mone... code!

Please visit the [examples repository](https://github.com/L3bowski/modena-v2-examples) to get a better understanding on how to use modena
