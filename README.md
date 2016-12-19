# Personal notes about Feathers js and React

The intention of this little handbook is to write about my journal to how to use feathers js with React on the client and server side and also to provide a short and concise guide to how to start doing things with feathers js and React fast and easy.

> Note: I will use the feathers js cli tool to generate the feathers js related code however there is another way to do it, and that is to use the yeamon generator

## Requirements

The next list is the needed stuff that you need to install first in order to work properly with this way of work

- flow from [flowtype](https://flowtype.org/)
- vscode from [visual studio code](https://code.visualstudio.com)
- yarn from [yarnpkg](https://yarnpkg.com/)


## How to install the CLI tool

Open a command prompt and type the next command:

```cmd
npm i -g feathers-cli
```

## How to init a feathers js project

Go to the directory that you want to create the app, then type:

```cmd
feathers generate
```

After the installation process we just need to run the next command to make it works

```cmd
npm start
```

## List of useful libraries

This list of libraries can be helpful and I recommend to install it after the feathers installation get completed.

> I will use from now on the yarn tool instead of npm

This packages will be the normal dependencies

```cmd
yarn add react react-dom react-router redux react-redux redux-thunk babel-polyfill
```

This packages will be used as a dev dependencies

```cmd
yarn add --dev eslint babel-cli babel-eslint babel-loader babel-plugin-transform-flow-strip-types babel-preset-latest babel-preset-react babel-preset-stage-3 chokidar ejs eslint-import-resolver-webpack eslint-plugin-flowtype eslint-plugin-import eslint-plugin-react nodemon redux-logger rimraf typescript webpack webpack-dev-middleware webpack-hot-middleware file-loader url-loader css-loader style-loader
```

## Configure eslint 

Using the `.eslinrc` file configure eslint to provide right sintax linting

```json
{
    "parser": "babel-eslint",
    "env": {
        "browser": true,
        "commonjs": true,
        "es6": true,
        "node": true
    },
    "parserOptions": {
        "ecmaFeatures": {
            "classes": true,
            "jsx": true,
            "experimentalObjectRestSpread": true
        },
        "sourceType": "module"
    },
    "rules": {
        "no-const-assign": "warn",
        "no-this-before-super": "warn",
        "no-undef": "warn",
        "no-unreachable": "warn",
        "no-unused-vars": "warn",
        "constructor-super": "warn",
        "valid-typeof": "warn",
        "no-console": "warn",
        "semi": "error",
        "quotes": [
            "error",
            "single",
            {
                "allowTemplateLiterals": true
            }
        ]
    },
    "plugins": [
        "react",
        "import",
        "flowtype"
    ],
    "extends": [
        "eslint:recommended",
        "plugin:react/recommended",
        "plugin:import/errors",
        "plugin:import/warnings",
        "plugin:flowtype/recommended"
    ],
    "settings": {
        "import/resolver": "webpack",
        "flowtype": {
            "onlyFilesWithFlowAnnotation": true
        }
    }
}
```

## Add flow init file

To work with flow we need to add the `.flowconfig` file, to do that just run this command (we are assumming that you already have flow-bin installed):

```cmd
flow init
```

## Configure webpack

Next copy and past the next configuration for webpack into a file with the name of `webpack.config.js` into the root of the feathers project

```js
// @flow
const webpack = require('webpack');
const path = require('path');

module.exports = {
  entry: [
    'webpack-hot-middleware/client?reload=true',
    'babel-polyfill',
    __dirname + '/app/'
  ],
  output: {
    path: path.resolve('public'),
    filename: 'bundle.js',
    publicPath: '/'
  },
  module: {
    loaders: [{
      test: /\.(js|jsx)$/,
      exclude: /node_modules/,
      loaders: ['babel-loader']
    }, {
      test: /\.css/,
      exclude: /node_modules/,
      loaders: ['style', 'css']
    }]
  },
  externals: {
    'socket.io': 'io'
  },
  resolve: {
    extensions: ['', '.js', '.jsx']
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin(),
  ]
};
```

## Configure vscode for the javascript project

If for some reason vscode doesn't create the `jsconfig.json` file for you, here is a copy that you can use:

```json
{
	// See https://go.microsoft.com/fwlink/?LinkId=759670
	// for the documentation about the jsconfig.json format
	"compilerOptions": {
		"target": "es6",
		"module": "commonjs",
		"allowSyntheticDefaultImports": true
	},
	"exclude": [
		"node_modules",
		"bower_components",
		"jspm_packages",
		"tmp",
		"temp"
	]
}
```

## Delete the index.html

Inside the public folder there is a `html` file with the name of `index.html`, delete it.

## Add the views folder to the root of the feathers project

Create a folder with the name `views` and create a file with the name of `index.ejs`.

Here is a sample of the code of that file

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Respuestas - QA System</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/1.5.1/socket.io.min.js"></script>
</head>
<body>
  <div id="main"><%- markup %></div>
  <script>window.__PRELOADED_STATE__ = <%- JSON.stringify(preloadedState)  %></script>
  <script src="bundle.js"></script>
</body>
</html>
```

## Add ejs template system to the server

Go to the root of the feathers project > src > app.js and add this lines to that file

```js
app.set('view engine', 'ejs');
```

## Add React support to be render on the server

First go to the feathers project root > src > middleware > index.js and add this code

```js

// to the top of the others import paste this
import * as React from 'react';
import { renderToString } from 'react-dom/server';
import { match, RouterContext } from 'react-router';
import { Provider } from 'react-redux';
// import the route file
// import the configurestore function for redux

// then before the first middleware

app.get('*', (req, res) => {

  const store = configureStore();
  match({
    routes,
    location: req.url
  }, (err, redirectLocation, props) => {
    if (err) {
      // something went badly wrong, so 500 with a message
          res.status(500).send(err.message);
        } else if (redirectLocation) {
          // we matched a ReactRouter redirect, so redirect from the server
          res.redirect(302, `${redirectLocation.pathname}${redirectLocation.search}`);
        } else if (props) {
          // if we got props, that means we found a valid component to render
          // for the given route
          const markup = renderToString(<Provider store={store}>
            <RouterContext {...props} ></RouterContext>
          </Provider>);

          const preloadedState = store.getState();

          res.render('index', { markup, preloadedState });
        } else {
          // no route match, so 404. In a real app you might render a custom
          // 404 view here
          res.sendStatus(404);
        }
      });
  });

```

## Configure Babeljs

Create a file in the root directory of the feathers app with the name of `.babelrc` file and paste the next snippets

```json
{
  "presets": [
    "react",
    "latest",
    "stage-3"
  ],
  "plugins": [
    "transform-flow-strip-types"
  ]
}
```