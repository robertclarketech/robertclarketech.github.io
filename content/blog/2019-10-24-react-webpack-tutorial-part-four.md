+++
title = "React And Webpack Tutorial - Part 4"
description = "A React and Webpack tutorial, starting from the very bottom."

[taxonomies]
categories = ["React Webpack Tutorial"]
tags = ["react", "webpack"]
+++

# Previous

- [Part 1](@/blog/2019-10-21-react-webpack-tutorial-part-one.md)
- [Part 2](@/blog/2019-10-22-react-webpack-tutorial-part-two.md)
- [Part 3](@/blog/2019-10-23-react-webpack-tutorial-part-three.md)

You can find the full repository and all previous commits by [clicking here](https://gitlab.com/Worble/react-webpack-tutorial).

Last time we had finished making ourselves a swanky development setup, with a live refreshing server, source maps, and even hot reloading. We're now ready to do some actual developing! But wait, we're still writing in plain old JavaScript. Sure, it might have some fancy new ES2015+ features, but it's still untyped! That just won't do, so today we're going to work on adding TypeScript support to our project.

<!-- more -->

# TypeScript support

## Updating Files

So like everything else, adding TypeScript support is going to involve installing the compiler, a loader for Webpack, and hooking it all up. For starters however, let's just convert our existing code to TypeScript.

First, let's change our file types. `./src/index.jsx` will become `./src/index.tsx`, and `./src/myComponent.jsx` will become `./src/myComponent.tsx`.

Next, let's download the types packages for React so TypeScript knows exactly what types the packages export and so we can get proper IDE support:

```
>yarn add --dev @types/react @types/react-dom
```

Now we need to fix up the files. `index.tsx` will need some small modifications to the imports:

```tsx
import * as React from "react";
import * as ReactDOM from "react-dom";
import MyComponent from "./myComponent";

ReactDOM.render(<MyComponent />, document.getElementById("root"));
```

as will `myComponent.tsx`, along with an extra interface:

```tsx
import * as React from "react";
import { hot } from "react-hot-loader";

interface IMyComponentState {
  name: string;
}

class MyComponent extends React.Component<{}, IMyComponentState> {
  constructor(props: {}) {
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

I won't go into detail about what's happening here, TypeScript and React is a tutorial for a different day, but simply put, we now need an interface to describe our component state, and the way we import certains libs is different.

## Webpack Setup

Now we're done making our files TypeScript compatible, let's actually get Webpack ready to compile them. We're going to use the `awesome-typescript-loader`, along with TypeScript itself:

```
>yarn add --dev typescript awesome-typescript-loader
```

Note that there is a `ts-loader`, which _kind of_ works, however `awesome-ts-loader` seems to work much better with Babel and `react-hot-loader`, so we're going with that.

And let's create a `tsconfig.json` in our root repository. The example here I'm giving is just my config file, there are many options here you can tweak to your liking.

```json
{
  "compilerOptions": {
    "jsx": "preserve",
    "allowSyntheticDefaultImports": true,
    "lib": ["dom", "es2015", "es2016"],
    "target": "es2016",
    "module": "es2015",
    "moduleResolution": "node",
    "noImplicitAny": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "removeComments": false,
    "preserveConstEnums": true,
    "sourceMap": true,
    "skipLibCheck": true
  }
}
```

What's really important here is that `"jsx"` is set to `"preserve"`; this means typescript won't change our code, and will instead let babel do that stage of the compilation for us. You might ask why we're using babel if we don't have to? Why not let TypeScript do all that work for us? Afterall, adding another compilation step is only going to make our build times longer. Well, unfortunately for us, `react-hot-loader` _requires_ that code goes through babel first, so unfortunately we're stuck with this for now. If you don't find hot reloading to be a deal breaker for you, you could remove babel from the project, and tweak your `tsconfig.json` accordingly.

Anyway, let's hook this all up into webpack. We're going to add a new object to rules describing `.tsx`, as well as update our entrypoint name:

```js
module.exports = {
  module: {
    entry: [
      "react-hot-loader/patch",
      path.resolve(__dirname, "src", "index.tsx")
    ],
    //...
    rules: [
      //...
      {
        test: /\.ts(x?)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: "awesome-typescript-loader",
            options: {
              useBabel: true,
              babelCore: "@babel/core"
            }
          }
        ]
      }
    ]
  }
};
```

What we're telling Webpack to do here, which hopefully you can understand by now, is that for all files with a `.ts(x)` extension, run it through our `awesome-typescript-loader` (which will automatically pick up our `tsconfig.json`), and we've also the setup our loader to use babel (the `babelCore: "@babel/core"` is required for Babel7), which as explained earlier is required for `react-hot-loader`. Again, you can see the full options you can give to `awesome-typescript-loader` at it's [relevant documentation page](https://github.com/s-panferov/awesome-typescript-loader#typescript-loader-for-webpack)

Alright, let's give it a whirl:

```
yarn serve
```

And... you'll probably see this error in your console:

```
ERROR in ./src/index.tsx
Module not found: Error: Can't resolve './myComponent' in '<folder>\src'
 @ ./src/index.tsx 7:0-40 8:36-47
 @ multi react-hot-loader/patch ./src/index.tsx
```

So you'll notice way back when we changed the imports to `./src/index.tsx`, we changed the way we import our component: `import MyComponent from "./myComponent";`. This unfortunately has confused Webpack a little bit, since by default the the only extensions it looks for are `.js` files, if you don't provide one in the import. So, in order to tell Webpack what files it should be looking for, we'll add another property to our `resolve` property:

```js
module.exports = {
  //...
  resolve: {
    alias: {
      "react-dom": "@hot-loader/react-dom"
    },
    extensions: [".tsx", ".jsx", ".ts", ".js"]
  }
  //...
};
```

This tells Webpack "Hey, if you see an import without an extension, look for these filetypes". Note that if there are multiple files with the same name, Webpack will resolve the one with the extension listed first in the array and skip the rest. [See the relevant documentation here](https://webpack.js.org/configuration/resolve/#resolveextensions).

For reference, here is our full webpack config after adding TypeScript support:

```js
const path = require("path");

module.exports = {
  mode: "development",
  entry: [
    "react-hot-loader/patch",
    path.resolve(__dirname, "src", "index.tsx")
  ],
  devtool: "eval-source-map",
  output: {
    filename: "main.js",
    path: path.join(__dirname, "dist")
  },
  resolve: {
    alias: {
      "react-dom": "@hot-loader/react-dom"
    },
    extensions: [".tsx", ".jsx", ".ts", ".js"]
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
      },
      {
        test: /\.ts(x?)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: "babel-loader",
            options: {
              cacheDirectory: true
            }
          },
          {
            loader: "ts-loader"
          }
        ]
      }
    ]
  }
};
```

Alright, let's try again.

```
>yarn serve
```

And if all went well, we should now have working TypeScript support! Try changing the IMyComponentState properties or accessing a property that doesn't exist to test it out.

Let's make a commit:

```
>git add .
>git commit -m "added typescript support"
```

[You can view the commit here](https://gitlab.com/Worble/react-webpack-tutorial/commit/d161d0ba13f770407364fb576d6535a42cdc694f).

# Building For Production

So you might remember, way back when in [part 1](@/blog/2019-10-21-react-webpack-tutorial-part-one.md) that when we first built our project using Webpack, it defaulted to 'production' mode. We've made a lot of progress since then, including updating to use Webpack config files instead of typing all of our commands on the command line, so let's try and update our project so we can build in production again!

First, let's make a new configuration file. This is the one we'll point Webpack to whenever we want to build for production. Put it in the root of the project and call it `webpack.prod.js`. Now, we _could_ copy over the contents of our current `webpack.config.json` into this one, but would be a pain to manage; if we wanted to add another filetype for example, we would need to make sure we added it in both. The [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle suggests we should find a better way. There's probably a number of ways you could solve this problem, but we're just going to use `webpack-merge` ([documentation](https://github.com/survivejs/webpack-merge)). This will allow us to define multiple configurations and, well, merge them together, surprisingly enough. Let's install it:

```
>yarn add --dev webpack-merge
```

Now, let's refactor our `webpack.config.json` into a `webpack.common.json`. This will hold all the properties that are reused between our development and production configurations.

```js
const path = require("path");

module.exports = {
  entry: [
    "react-hot-loader/patch",
    path.resolve(__dirname, "src", "index.tsx")
  ],
  output: {
    filename: "main.js",
    path: path.join(__dirname, "dist")
  },
  resolve: {
    alias: {
      "react-dom": "@hot-loader/react-dom"
    },
    extensions: [".tsx", ".jsx", ".ts", ".js"]
  },
  module: {
    rules: [
      {
        test: /\.ts(x?)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: "awesome-typescript-loader",
            options: {
              useBabel: true,
              babelCore: "@babel/core"
            }
          }
        ]
      }
    ]
  }
};
```

Now our `webpack.prod.js` can merge it's own configuration into common:

```js
const merge = require("webpack-merge");
const common = require("./webpack.common.js");
const path = require("path");

module.exports = merge(common, {
  mode: "production",
  devtool: "source-map"
});
```

All we care about at the moment is setting our mode to "production" for the inbuilt changes that Webpack makes, as well as changing our source mapping mode. Now we can create something similar for development, let's call it `webpack.dev.js`:

```js
const merge = require("webpack-merge");
const common = require("./webpack.common.js");

module.exports = merge(common, {
  mode: "development",
  devtool: "eval-source-map",
  devServer: {
    hot: true,
    contentBase: path.join(__dirname, "dist")
  }
});
```

Obviously we've seen all this before, the only difference here is that compared to production, we only care about running the dev server in development mode, so we keep this configuration in here. Now, we need to update our `package.json` scripts to point Webpack at the right configurations, which we can do with the `--config` flag:

```json
{
  //...
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "serve": "webpack-dev-server --config webpack.dev.js --open",
    "dev": "webpack --config webpack.dev.js",
    "prod": "webpack --config webpack.prod.js"
  }
  //...
}
```

Note that the development builds script name has changed from "build" to "dev", just to better reflect the build it's making.

A handy tip for `webpack-dev-server`: keeping `--open` at the end means you can pass arguments down to it, even when calling it through yarn or npm. For example, doing `yarn serve firefox` will actually be equivalent to `webpack-dev-server --config webpack.dev.js --open firefox`, allowing you to change what browser you're testing in easily.

Try all these commands now and make sure they work as you expect. Congratulations, your Webpack project is now ready to built for an actual production environment!

Let's make a commit here:

```
>git add .
>git commit -m "added production webpack config"
```

[You can view the commit here](https://gitlab.com/Worble/react-webpack-tutorial/commit/3d71d4913c1b2f40cc8dc8a36b99c8a5720410ed)

These tutorials are going to continue for a little bit, as we continue to customise our Webpack configuration with extra niceities, but hopefully by now the magic spookiness of Webpack should be gone, and you are confident in making your own personal configuration. I highly recommend looking at the [plugins section of the Webpack website](https://webpack.js.org/plugins/), finding a few that you would find useful, and trying to implement some of these yourself.

That's all for this time, next time we'll look at generating our `index.html` using Webpack, cache-busting and serving static files!

[Click here for Part 5!](@/blog/2019-10-25-react-webpack-tutorial-part-five.md)
