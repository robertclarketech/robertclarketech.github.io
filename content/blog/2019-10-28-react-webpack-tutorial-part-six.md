+++
title = "React And Webpack Tutorial - Part 6"
description = "A React and Webpack tutorial, starting from the very bottom."

[taxonomies]
categories = ["React Webpack Tutorial"]
tags = ["react", "webpack"]

+++

# Previous

- [Part 1](@/blog/2019-10-21-react-webpack-tutorial-part-one.md)
- [Part 2](@/blog/2019-10-22-react-webpack-tutorial-part-two.md)
- [Part 3](@/blog/2019-10-23-react-webpack-tutorial-part-three.md)
- [Part 4](@/blog/2019-10-24-react-webpack-tutorial-part-four.md)
- [Part 5](@/blog/2019-10-25-react-webpack-tutorial-part-five.md)

You can find the full repository and all previous commits by [clicking here](https://gitlab.com/Worble/react-webpack-tutorial).

Last time, we started creating our `dist/index.html` file through Webpack, looked at how to ensure users are always getting our latest changes, and how to serve static files. In today's programme, we're going to be covering SASS compilation, and file compression!

<!-- more -->

# CSS

Although I say we're going to SASS compilation, we haven't even added some CSS to our project yet! So let's tackle that first.

Alright, let's make some CSS. We'll put this in a new folder under `./src/` called `css`, and create a new file called `reset.css`. We'll just put a basic CSS reset in here for now:

```css
html {
  box-sizing: border-box;
  font-size: 16px;
}

*,
*:before,
*:after {
  box-sizing: inherit;
}

body,
h1,
h2,
h3,
h4,
h5,
h6,
p,
ol,
ul {
  margin: 0;
  padding: 0;
  font-weight: normal;
}

ol,
ul {
  list-style: none;
}

img {
  max-width: 100%;
  height: auto;
}
```

Alright, now let's import this into our `./src/index.tsx` so it's picked up by Webpack:

```tsx
//...
import "./css/reset.css";
//...
```

If we run Webpack now, we should see that familiar message:

```
ERROR in ./src/css/reset.css 1:5
Module parse failed: Unexpected token (1:5)
You may need an appropriate loader to handle this file type, currently no loaders
are configured to process this file.
```

And it's right! We're going to need two new loaders for css support - the aptly named `css-loader` ([documentation](https://webpack.js.org/loaders/css-loader/)) in order to actually read and parse the styles, and then the `style-loader` ([documentation](https://webpack.js.org/loaders/style-loader/)), which will take the styles and put them in `<style></style>` tags on the DOM.

```
yarn add --dev style-loader css-loader
```

And once again, let's update our `./webpack.common.js`, telling it what it do when it meets a `.css` file:

```js
//...
module.exports = {
  //...
  module: {
    rules: [
      //...
      {
        test: /\.css$/,
        exclude: /node_modules/,
        use: ["style-loader", "css-loader"]
      }
    ]
  }
  //...
};
```

Remember, the loaders are always resolved from _right_ to _left_, so we want it to parse the css first with `css-loader`, and then that css will be injected into the DOM with `style-loader`, so this order is important!

Alright, let's give it a whirl. If everything went right you should now notice less padding on our elements. If you want to be sure it's working, feel free to edit the CSS (and thanks to hot reloading, it's automatically applied, and we don't lose any state!)

Let's make a commit here:

```
git add .
git commit -m "added css support"
```

[You can view the commit here](https://gitlab.com/Worble/react-webpack-tutorial/commit/baad310375c5c3282dc130a037f88803da9045c1) (there are a couple of extra changes in this commit as I was playing around with things and forgot to revert them. Don't worry about adding these in or not, they won't matter).

# Extracting CSS

Now this is all well and good for development purposes, however if you do a dev or production build you'll see that we aren't getting any `.css` files out of it, just JavaScript. Well, what `style-loader` does is create some JavaScript that injects the style into the DOM on load. While this works for development, in production there are many benefits to having actual `.css` files that are referenced in the base HTML. For starters, it'll load quicker since the browser won't need to parse, execute JavaScript and modify the DOM before getting any of the styles. It also means it can be cached by the browser.

So, how do we get our `.css` out of the `.js` file? Well, we can use a different loader for production: `mini-css-extract-plugin` ([documentation](https://webpack.js.org/plugins/mini-css-extract-plugin/)). As the documentation page says, it:

> This plugin extracts CSS into separate files. It creates a CSS file per JS file which contains CSS. It supports On-Demand-Loading of CSS and SourceMaps.

So this allows us to extract our CSS into seperate files, let's install it:

```
yarn add --dev mini-css-extract-plugin
```

Now, we only want to do this in production builds, where it will replace `style-loader`. We want to keep `style-loader` for development builds however, since it facilitates hot module reloading. So let's quickly extract what we put in out `./webpack.common.js` to our `./webpack.dev.js`:

```js
const merge = require("webpack-merge");
const common = require("./webpack.common.js");

module.exports = merge(common, {
  mode: "development",
  devtool: "eval-source-map",
  devServer: {
    hot: true
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        exclude: /node_modules/,
        use: ["style-loader", "css-loader"]
      }
    ]
  }
});
```

And now we'll modify our `./webpack.prod.js` to use `MiniCssExtractPlugin` instead:

```js
//...
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = merge(common, {
  //...
  module: {
    rules: [
      {
        test: /\.css$/,
        exclude: /node_modules/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader
          },
          "css-loader"
        ]
      }
    ]
  },
  plugins: [
    //...
    new MiniCssExtractPlugin({
      filename: "[name].[hash].css",
      chunkFilename: "[id].[hash].css"
    })
  ]
});
```

So here, we give it the new loader, and also setup the plugin itself, just giving names for our output for now. The plugin must be setup for the loader to work.

And if we give it a spin with a `yarn prod`, you should see your new `.css` files are generated! Even better, our HTML plugin recognises it and immediately places it in the head of our `./dist/index.html`. Just magic.

Now we're almost ready to deploy to production, but there's another small problem; our CSS isn't minimised! Well, if we check our old friend [the docs](https://webpack.js.org/plugins/mini-css-extract-plugin/#minimizing-for-production) once again, this is an easy problem to solve. We'll need another plugin: `optimize-css-assets-webpack-plugin` ([documentation](https://github.com/NMFR/optimize-css-assets-webpack-plugin)) so let's install that:

```
yarn add --dev optimize-css-assets-webpack-plugin
```

And update our `./webpack.prod.js` again:

```js
//...
const TerserJSPlugin = require("terser-webpack-plugin");
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");

module.exports = merge(common, {
  //...
  optimization: {
    minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})]
  }
  //...
});
```

TerserJS is the default minimizer in Webpack, we just have to respecify it here if we're applying the `optimization.minimizer` property. And if we run `yarn prod` yet again, we can now see our CSS is nicely minimized! Huzzah. Unfortunately however, it seems we've lost source mapping on our `.js` files, and our `.css` file doesn't have one either. Luckily, these are easy fixes, we just need to specify the right options to the minimizer plugins. As always you can view all the options and documentation on their relevant pages ([Terser](https://webpack.js.org/plugins/terser-webpack-plugin/), [OptimizeCssAssets](https://github.com/NMFR/optimize-css-assets-webpack-plugin)), but it isn't too hard to figure out what to do:

```js
//...
module.exports = merge(common, {
  //...
  optimization: {
    minimizer: [
      new TerserJSPlugin({ sourceMap: true }),
      new OptimizeCSSAssetsPlugin({
        cssProcessorOptions: {
          sourceMap: true,
          map: {
            inline: false,
            annotation: true
          }
        }
      })
    ]
  }
  //...
});
```

Alright, let's run `yarn prod` one last time, and presto! Source maps! Let's make a commit here to commemorate:

```
git add .
git commit -m "css now extracts and minifies in production"
```

[You can view the commit here](https://gitlab.com/Worble/react-webpack-tutorial/commit/611401b5cbefae47c0909ec14fc5e7a0c389d0ad).

# SASS

Alright, this one should be quick; you should know the drill by now. Let's change our `./src/css` folder to `./src/scss`, and change our `reset.css` file to `reset.scss`, and then update our `./src/index.tsx` to point at the new file:

```tsx
import "./scss/reset.scss";
```

Running Webpack will, you guessed it, tell us we need a new loader. We're going to be using `sass-loader` ([documentation](https://webpack.js.org/loaders/sass-loader/)). With `sass-loader`, we'll also need `node-sass` which will actually do the compilation from SASS to CSS.

```
yarn add --dev sass-loader node-sass
```

And let's configure it to run in out `./webpack.dev.js`:

```js
//...
module.exports = merge(common, {
  //...
  module: {
    rules: [
      {
        test: /\.(css|s[ac]ss)$/,
        exclude: /node_modules/,
        use: ["style-loader", "css-loader", "sass-loader"]
      }
    ]
  }
});
```

We update the test so this matches `.scss`, `.sass` and `.css` files, and put `sass-loader` at the end. Remember, loaders go right to left! Running `yarn dev` here should work fine, so let's update `./webpack.prod.js`:

```js
///...
module.exports = merge(common, {
  //...
  module: {
    rules: [
      {
        test: /\.(css|s[ac]ss)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader
          },
          "css-loader",
          "sass-loader"
        ]
      }
    ]
  }
  //...
});
```

And its the same thing again. Run `yarn prod` just to make sure everything works ok. And we're done! SASS added to project. Let's commit here:

```
git add .
git commit -m "added sass compilation"
```

[You can view the commit here](https://gitlab.com/Worble/react-webpack-tutorial/commit/242ded4f1c9665e376a5e7b2d43b1103c00594a0).

# Compression

Alright, last topic: compression! Most webservers and clients these days offer compression out of the box. This cuts down on asset size (sometimes dramatically), so let's go ahead and add that to the project as a final touch. Once again, if you're using an application server, this will probably do this for you, so there's probably no need fo Webpack here, make sure to check your Web frameworks documentation.

We're going to be using `CompressionWebpackPlugin` ([documentation](https://webpack.js.org/plugins/compression-webpack-plugin/)) to compress our assets.

```
yarn add --dev compression-webpack-plugin
```

And since we'll only need this for production builds, let's edit `./webpack.prod.js` to use it:

```js
//...
const CompressionPlugin = require("compression-webpack-plugin");

module.exports = merge(common, {
  //...
  plugins: [
    //...
    new CompressionPlugin()
  ]
});
```

That should be all we need, run `yarn prod` and you should see your build files, plus a copy of all of them gzipped. Creating every file and also having a gzipped copy seems a little redundant, so let's setup the plugin so it removes the original files:

```js
//...
const CompressionPlugin = require("compression-webpack-plugin");

module.exports = merge(common, {
  //...
  plugins: [
    //...
    new CompressionPlugin({
      deleteOriginalAssets: true
    })
  ]
});
```

By default it uses zlib to gzip the files, however if your webserver and clients support it, and you're running Node >= 11.7.0, you can use Brotli which offers superior compression quality.

```js
//...
const CompressionPlugin = require("compression-webpack-plugin");

module.exports = merge(common, {
  //...
  plugins: [
    //...
    new CompressionPlugin({
      deleteOriginalAssets: true,
      algorithm: "brotliCompress"
    })
  ]
});
```

Alright, we're just about done here, let's commit for a final time here:

```
git add .
git commit -m "added compression to production builds"
```

[You can view the commit here](https://gitlab.com/Worble/react-webpack-tutorial/commit/ef4cc7071eec8767d286a5bc79a572e8b38f8d2e).

Alright, we're in a pretty good spot now. We've got a project with TypeScript, React and SASS compilation that minifies, produces source maps and compresses itself. If you came all this way, well done! You should be a master at using Webpack by now. I may also cover creating JavaScript Workers and PWA's at a later point, so keep an eye out for that.

I also hope to eventually make a guide on creating your own Loaders and Plugins, but that'll be done when it's done. Until next time!
