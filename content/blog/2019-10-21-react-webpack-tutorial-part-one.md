+++
title = "React And Webpack Tutorial - Part 1"
description = "A React and Webpack tutorial, starting from the very bottom."

[taxonomies]
categories = ["React Webpack Tutorial"]
tags = ["react", "webpack"]
+++

# Introduction

When you're first starting to learn React, many tutorials will tell you to use create-react-app. This handy tool allows you focus on learning React, without getting bogged down in Webpack or the additional JavaScript tooling and ecosystem. This is great for people to get familiar with the language and SPA concepts, but chances are at some point you'll eventually need to dip down into webpack and make changes. The aim of this tutorial is to hopefully explain React and it's relationship with Webpack from the ground up. This tutorial (hopefully) makes few assumptions about the readers previous knowledge of React, Webpack and JavaScript tooling, but assumes a baseline knowledge of general front-end development and JavaScript.

<!-- more -->

# Tooling

- Ensure you have [node](https://nodejs.org/en/) installed.
- The above will install npm for you, nodes package manager. However, I strongly recommend you use [yarn](https://yarnpkg.com/lang/en/) instead. All commands in the guide will use yarn, but there are npm equivalents for them all.
- (Optional) Ensure [git](https://git-scm.com/) is installed. This isn't a prerequisite for developing, but it's good practice to always make regular commits while developing. As a bonus, you can download any given commit for a section if you get lost.

# Setup

To start with, we're going to do `git init` to create a repository in our directory. This should give the following output.

```
>git init
Initialized empty Git repository in <directory>/.git/
```

Follow with an `npm init -y` to start our npm project. If all went well you should see something similar to the following in your command line.

```
>npm init -y
Wrote to <folder>\package.json:

{
  "name": <folder>,
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
}
```

# Our first code

Now we're all setup, we can make our first commit.

```
>git add .
>git commit -m "Initial Commit"
```

Thanks to git you can follow along with the tutorial as we write code in case you get lost, or just want a point of reference. For example you can see the commit where we are right now by [clicking here](https://gitlab.com/Worble/react-webpack-tutorial/commit/bd64892e489001bd60ef4dc66ceb325f6660a5ba#7030d0b2f71b999ff89a343de08c414af32fc93a). The full repository can be seen by [clicking here](https://gitlab.com/Worble/react-webpack-tutorial)

Ok, so it's time to start actually making your react project. For starters, we need a folder to place our source code. You could put this anywhere you wanted, including the root of our project, but this would be an organisational nightmare and make it a lot harder later on when we have a large amount of files. For now, we're going to place all of our source code in a folder called `src`.

Inside that folder we can make our first file, `index.js`. This file will be known as our "entrypoint": all code for the application will start here and flow down.

So, let's go to Reacts website and steal their [smallest react example](https://reactjs.org/docs/hello-world.html). Let's place this in our `index.js`:

```tsx
ReactDOM.render(<h1>Hello, world!</h1>, document.getElementById("root"));
```

Unfortunately for us, this is using `.jsx` syntax. We'll come onto that later, but this will need compiling into raw JavaScript for the browser to parse it (Webpack will take of that for us eventually). For now we'll make a small modification so we can run it.

```js
ReactDOM.render(
  React.createElement("div", null, "Hello World"),
  document.getElementById("root")
);
```

We will also need to install `react` and `react-dom` as dependencies:

```
>yarn add react react-dom
```

And then import them into our `index.js` file:

```js
import React from "react"; //new line
import ReactDOM from "react-dom"; //new line

ReactDOM.render(
  React.createElement("div", null, "Hello World"),
  document.getElementById("root")
);
```

Now you may be wondering how to actually execute this script we've made. Usually in front-end development, you would create a HTML file, and place any js within <script></script> tags and then simply view that in the browser. If you tried that here however, you'd likely get a `Uncaught SyntaxError: Cannot use import statement outside a module` error. Because we're going to making use of the compilation, minification and other -ations of webpack, we just write our JavaScript, and webpack will take care of the rest (you can and we will be making HTML files later, but for now this is the simplest example). So let's move onto that.

# Setting up Webpack

First of all, webpack is a npm package, so we'll add it as a dependency first. We will also need to add the CLI tool. We use --dev to signify it's a dependency only for development, and won't be needed in the final source.

```
>yarn add webpack webpack-cli --dev
```

Note that we could globally install webpack, which would mean we would not longer need to install it individually for each of our projects, however installing it as a dev dependency has a couple of advantages:

1. The version is pinned: even if Webpack version 357 is released, our project will always resolve the version of webpack we know works, and any breaking changes won't affect us until we're ready to migrate
2. Anyone can pick up the project, and with a `yarn` or `npm i` can work on it, without having to figure which external tools they do or don't have installed

For our very first example, this is all we need to do. You can simply run `yarn webpack` in the command line from the root of your project and watch the magic.

```
>yarn webpack
yarn run v1.17.3
$<folder>\node_modules\.bin\webpack
Hash: 14370ea36c1d2d237762
Version: webpack 4.41.2
Time: 2399ms
Built at: 10/22/2019 1:44:55 PM
  Asset     Size  Chunks             Chunk Names
main.js  129 KiB       0  [emitted]  main
Entrypoint main = main.js
[3] ./src/index.js 174 bytes {0} [built]
    + 7 hidden modules

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/configuration/mode/
Done in 6.10s.
```

You may have noticed after running webpack, a new folder has appeared called `dist` in the root of the project. Since we didn't specify an input or an output, webpack made a best guess that our code is in `src/index.js` and that it should output the compiled code to `dist`. Looking inside of `dist` you should be able to see a `main.js`. Again, since we didn't specify what to call it, Webpack just made a best guess.

Looking inside of `dist/main.js` you should be able to see a large amount of rather ugly code. This is what we wrote, only minified to make the binary size as small as possible. It may look like there's far more code here than what was written in our `index.js`, but this minified code is a single bundle that contains all of our dependencies as well, which at this point is React and React-Dom. Since we didn't specify whether we wanted a production or a development build it assumed we wanted a production one and minified the contents.

You can play with some of the options to get a feel for how webpack works, for example, you can pass `-o <directory>` to change the output directory, or pass `--mode development` to see an unoptimized development build. You can see a full list of commands with `yarn webpack --help`. Don't feel you have know the CLI intimately however; we will soon be passing off most of these options to a webpack config file we will create in the future, and simply point webpack to that.

So what now? All we have is still just a `.js` file. Well, if we quickly create an `index.html` in our `dist` folder, and populate it like so:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Document</title>
  </head>
  <body>
    <div id="root"></div>
    <script src="./main.js"></script>
  </body>
</html>
```

Opening `dist/index.html` in your browser, you should be able to see "Hello World!". Congratulations, you've just gotten your first React app converted by webpack into something readable by the browser!

# Finishing Up

You may be tempted to make your second git commit here, but hold on just a sec! Our build process at the moment outputs our distributable files into our `dist` folder, however if we committed now we'd also be pushing all of these up to git! Not to mention our `node_modules` folder, which holds _every single dependency in our application_. Now this isn't _necessarily_ a problem, however for anyone cloning our repository, this potentially adds an incredibly large download for any contributers (not to mention the poor host who has to hold them all), especially when they can build the source themselves, retrieving dependencies from their own cache and get a reproducible build. So now, we're going to add a `.gitignore` to the root of the project and populate it with some basic defaults:

```
/node_modules/**
/dist/**
```

And now we can safely commit our next set of changes.

```
>git add .
>git commit -m "added index.js"
```

[Click here to view the commmit](https://gitlab.com/Worble/react-webpack-tutorial/commit/188101d367dcd4085ac70b2f9b9aee9a591a2ade).

You might think that running Webpack for every change is a pain, as is having to create and open a html file. And you'd be right! As we move on, we'll let Webpack do more and more of the heavy lifting.

Next time we'll look into using React's templating language JSX, creating a custom Webpack config file, and tying the two together so webpack compiles the JSX into browser readable JavaScript.

[Click here for Part 2!](@/blog/2019-10-22-react-webpack-tutorial-part-two.md)
