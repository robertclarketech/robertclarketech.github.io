+++
title = "React And Webpack Tutorial - Part 3"
description = "A React and Webpack tutorial, starting from the very bottom."

[taxonomies]
categories = ["React Webpack Tutorial"]
tags = ["react", "webpack", "babel"]
+++

# Previous

- [Part 1](@/blog/react-webpack-tutorial/2019-10-21-part-one.md)
- [Part 2](@/blog/react-webpack-tutorial/2019-10-22-part-two.md)

You can find the full repository and all previous commits by [clicking here](https://gitlab.com/Worble/react-webpack-tutorial).

Alright, last time we had a working Webpack config and a single JSX file. Today we're going to work on making ourselves a dev environment, with an auto-reloading dev server.

<!-- more -->

# Setting up a Development Webpack Config

First off, I'm basically going to be covering the very helpful [development guide over at Webpack website](https://webpack.js.org/guides/development/). At this point, you should understand the ecosystem well enough to be able to follow this yourself. You can [skip ahead to hot reloading](#hot-reloading) if you prefer to follow that one instead, but we're going to forge ahead here anyway.

As an aside, you may notice at this point that we're going to spending most of our time setting up Webpack loaders and addons, and you'd be right. Once you're initially set up, that's all Webpack is - chaining together existing toolchains (such as typescript, sass and babel compilation, minification, and compression) into one pipeline.

# Mode

So first of all, since we're building for development, we're going to tell Webpack that that is our aim; add mode: 'development' to our webpack config:

```js
const path = require("path");

module.exports = {
  mode: "development"
  //...
};
```

Essentially, this line is an alias for a number of Webpack optimizations, as well as letting any plugins we're using know what environment we're in so they can react accordingly as well. You can see the full list of changes that setting the mode changes at [Webpack's documentation page](https://webpack.js.org/configuration/mode/).

# Source Maps

Since we're compiling our code from ES2015+ syntax, when we view it when developing, it's going to be hard for anyone (including ourselves!) to understand and debug it. That's where source maps come in. Source maps "map" the code to what was originally written, and the browser will automatically show you the original code. To enable this, we'll add another option in our Webpack config `devtool: "eval-source-map"`

```js
const path = require("path");

module.exports = {
  //...
  devtool: "eval-source-map"
};
```

There are a number of different mapping schemes you can use: some are recommended for development purposes, others for production. You can see the full list and their details on the [relevant Webpack configuration page](https://webpack.js.org/configuration/devtool/). For now, `eval-source-map` is satisfactory for us.

# Dev Server

If you read the above link, Webpack recommend three tools for testing development changes:

- webpack's Watch Mode
- webpack-dev-server
- webpack-dev-middleware

The pros and cons are listed in the article, but we're just going to focus on webpack-dev-server since it's the most featureful while still being easy to setup.

First, let's install it:

```
> yarn add --dev webpack-dev-server
```

Then we'll edit our `./webpack.config.js` to use it, telling it where it should serve our built files from:

```js
const path = require("path");

module.exports = {
  //...
  devServer: {
    contentBase: path.join(__dirname, "dist")
  }
};
```

**UPDATE 2019/10/28** - This line isn't actually necessary, you can leave any devServer configuration off for now

As always, there are a bunch of options here we aren't using, for a full reference of `devServer` see [Webpacks documentation](https://webpack.js.org/configuration/dev-server/). And now we can run:

```
>yarn webpack-dev-server --open
```

This should cause your browser to open, and run our code. You may notice that control of the terminal is not given back either; webpack-dev-server is still running, and watching our code. Go ahead and make a simple change in our `./src/index.jsx` file:

```tsx
import React from "react";
import ReactDOM from "react-dom";

//change anything in the tags here
//                       ||
//                       \/
ReactDOM.render(<h1>Hello, worlds!</h1>, document.getElementById("root"));
```

Save your file, and voila! You should notice your changes are picked up by the terminal, they will be rebuilt, and your browser will refresh to automatically reflect the changes.

It's great that we no longer have to build every time, but it's a pain to write out `yarn webpack-dev-server --open` every time, and more importantly, how would someone joining our project know they have to run this line? To solve this, let's make a quick script in our `./package.json` file to alias this for us. Under "scripts", add: `"serve": "webpack-dev-server --open"` (note we don't need the `yarn` here, that's automatically inferred for us). Now we can simple call `yarn serve` and get the same effect. While we're here, we can also add a script for building without the dev server: `"build": "webpack"`, and update our "main" to be "index.jsx":

```json
{
  "name": "react-webpack-tutorial-part-one",
  "version": "1.0.0",
  "main": "index.jsx",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "serve": "webpack-dev-server --open",
    "build": "webpack"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "react": "^16.10.2",
    "react-dom": "^16.10.2"
  },
  "devDependencies": {
    "@babel/core": "^7.6.4",
    "@babel/preset-env": "^7.6.3",
    "@babel/preset-react": "^7.6.3",
    "babel-loader": "^8.0.6",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.9",
    "webpack-dev-server": "^3.9.0"
  },
  "keywords": [],
  "description": ""
}
```

This seems like a good place for a commit, so let's do one here:

```
>git add .
>git commit -m "setup development environment"
```

[View the commit here](https://gitlab.com/Worble/react-webpack-tutorial/commit/e6c7cba3c6e17443a5867c22aade18403cba799c).

# Hot Reloading

Having the browser refresh on code change is _nice enough_, but doesn't it still suck to lose all your stateful data? In order to demonstrate this, let's expand our React a little bit.

Let's create a new component called `myComponent.jsx` in our src folder, and we'll just create a very simple component:

```tsx
import React from "react";

export default class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      name: ""
    };
  }
  render() {
    return (
      <div>
        <div>
          <label>
            Name:
            <input
              onChange={event => this.setState({ name: event.target.value })}
            ></input>
          </label>
        </div>
        <div>Hello {this.state.name}!</div>
      </div>
    );
  }
}
```

And let's add this to our `index.jsx`:

```tsx
import React from "react";
import ReactDOM from "react-dom";
import MyComponent from "./myComponent.jsx";

ReactDOM.render(<MyComponent />, document.getElementById("root"));
```

And let's start up our dev server with `yarn serve`. Playing with our new component, you can see that as we type, the text gets saved to Reacts state, and is then inserted to the line below as "Hello <name>!". But, if we decide to change our file, for example if we change "Name:" to "Names:", lo and behold, our dev server picks up the changes, refreshes the browser, and our input is gone. Very annoying. Well, thanks to [react-hot-reloader](https://github.com/gaearon/react-hot-loader), we can avoid this!

First of all, let's install the necessary dependencies:

```
>yarn add react-hot-loader
>yarn add --dev @hot-loader/react-dom
```

And update our `.babelrc` file with `"plugins": ["react-hot-loader/babel"]`:

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": ["react-hot-loader/babel"]
}
```

Now, we need to mark our component that it should be hot reloaded. In `./src/myComponent.jsx`:

```tsx
import React from "react";
import { hot } from "react-hot-loader";

class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      name: ""
    };
  }
  render() {
    return (
      <div>
        <div>
          <label>
            Name:
            <input
              onChange={event => this.setState({ name: event.target.value })}
            ></input>
          </label>
        </div>
        <div>Hello {this.state.name}!</div>
      </div>
    );
  }
}

export default hot(module)(MyComponent);
```

We import the `hot` method, and instead of exporting the Component directly, we export `export default hot(module)(MyComponent);`. Note that this is the old syntax, the new syntax should allow us to direct export without the `module` word like so: `export default hot(MyComponent);` but I couldn't get this to work ¯\\\_(ツ)\_/¯.

Usually, most React projects will have a single root component that controls the flow of the entire application, and just making this component hot will make your entire application hot, so you only need to do this once.

Next we update our `webpack.config.js`. First update your entry array to also take 'react-hot-loader/patch' like so:

```js
module.exports = {
  entry: ["react-hot-loader/patch", path.resolve(__dirname, "src", "index.jsx")]
  // ...
};
```

(in all honestly I have no idea what this does, just that react-hot-loader requires it). Next, add a new property called "resolve" and alias react-dom to the one we installed earlier:

```js
module.exports = {
  // ...
  resolve: {
    alias: {
      "react-dom": "@hot-loader/react-dom"
    }
  }
};
```

This replaces the "react-dom" package of the same version, but with additional patches to support hot reloading.

And finally, update our devServer property so it runs in hot mode:

```js
module.exports = {
  // ...
  devServer: {
    hot: true,
    contentBase: path.join(__dirname, "dist")
  }
};
```

After all this, your `webpack.config.js` should look like this:

```js
const path = require("path");

module.exports = {
  mode: "development",
  entry: [
    "react-hot-loader/patch",
    path.resolve(__dirname, "src", "index.jsx")
  ],
  devtool: "eval-source-map",
  output: {
    filename: "main.js",
    path: path.join(__dirname, "dist")
  },
  resolve: {
    alias: {
      "react-dom": "@hot-loader/react-dom"
    }
  },
  devServer: {
    hot: true,
    contentBase: path.join(__dirname, "dist")
  },
  module: {
    rules: [
      {
        test: /\.js(x?)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: "babel-loader",
            options: {
              cacheDirectory: true
            }
          }
        ]
      }
    ]
  }
};
```

Alright, let's do another `yarn serve` and load up our app. You should now be able to type into the textbox, and despite any changes you make to the file, your state should be kept! Very cool.

That's all for this time, let's make a quick commit:

```
>git add .
>git commit -m "added hot reloading"
```

[View the commit here](https://gitlab.com/Worble/react-webpack-tutorial/commit/238ea4f55357bb7e00f4d84af8d72137e78ee8cc).

Next time we'll look into getting TypeScript support for our JSX components, as well as setting up a Webpack config for our production environment.

[Click here for Part 4!](@/blog/react-webpack-tutorial/2019-10-24-part-four.md)
