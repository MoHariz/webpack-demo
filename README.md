# Webpack Demo!

This lab is intended to walk you through the basics of the [**Webpack**](https://webpack.js.org/concepts/) module bundling system. Webpack is the tool most commonly used in the React & Javascript community to _transpile_ and _bundle_ components, and is what is used under the hood by [create-react-app](https://github.com/facebookincubator/create-react-app) (we'll be using this next week!). While that scaffolding tool means you don't need to know how to set up Webpack, it is good to be at least somewhat familiar with the concept (and this exercise will give you further practice working with `npm` modules).

_This tutorial is adapted from one by [Tyler McGinnis](https://tylermcginnis.com/react-js-tutorial-1-5-utilizing-webpack-and-babel-to-build-a-react-js-app-5f804d729d3b#.3v5n4cd1z)._

## What is Webpack?
Webpack is a **build tool**. That is, it is a (command-line) application that is used to automatically take the source code you write and prepare that code to be run/executed, whether for development or deployment. There are numerous such build tools in existence: [Gulp](http://gulpjs.com/) is the major competitor to Webpack, and most IDEs (like jGrasp or IntelliJ) provide them. However, Webpack is favored by React developers because of how easily and speedily it transpiles JSX.

At its core, Webpack is a module loader: it takes source files like JavaScript **modules** and bundles them into a few simplified files that can be part of your webpage. It takes your complicated source code structure (lots of files) and transforms it into a brand-new, "simplified" version (few files).

## Let's get started!

There are two things you need to do to get started:
1. First, we need to create the `package.json` file to store information about the app you're building, including dependencies that Webpack will use. You can do this with the following command:

    ```
    cd path/to/project
    npm init
    ```
You will be prompted for a bunch of information to provide about your app. Give it the following details (**just hit `<enter>` to accept the details on any other prompts**)

- `name` should be "webpack-tutorial"
- `author` should be your name

Once you're finished, you will be asked to confirm your choices (type `yes`), and you'll have a brand new `package.json` file ready to use!

> :bulb: If you just want a blank `package.json` file, use `npm init -y` to skip the prompts!


2. Locally install the `webpack` program. This will allow you to run the program from the command-line and bundle your app. Using the `--save-dev` flag saves the module to your `devDependencies`, which will _*not*_ be bundled in the bundled version of your app.

    ```
    npm install --save-dev webpack
    ```

## `webpack.config.js`
While Webpack can be used to do simple bundling from the command-line (see [the official tutorial](https://webpack.js.org/guides/getting-started//) for an example), it's most common to write down all of your bundling options inside a **configuration file**. This file is basically just a JavaScript file that defines a variable that represents all of the different options you'd want to pass to the `webpack` program. If you name this file **`webpack.config.js`**, Webpack will read in that configuration automatically. Thus "using Webpack" really involves creating this file.

1. Create a new file called **`webpack.config.js`** and open it up in your favorite editor (e.g., VS Code).

2. Inside this file, add the following line of code:

    ```javascript
    module.exports = {}
    ```

    This is the [CommonJS](http://requirejs.org/docs/commonjs.html) version of the ES6 `export default {}`&mdash;you are defining an object (initially empty) that will be exported and used by the Webpack program. **The rest of this tutorial will involve adding properties to this object**.

    - Note that is it _possible_ to use ES 6 style `import` and `export` commands, but you need to name your file `webpack.config.babel.js` and will need to have some Babel libraries installed. See [here](http://stackoverflow.com/questions/31903692/how-to-use-es6-in-webpack-config) for discussion. For this lab, stick to the CommonJS syntax.

Webpack's basic job is to take your source files, changing them in some way, and producing a new version. Thus there are three things you'll need to specify:

1. What files to transform (specifically: what JavaScript file makes your program start)
2. What transformations to make
3. Where to save the transformed files

## `entry` and `output`
We specify the first piece (what files to transform) by giving the exported object an `entry` property:

```javascript
module.exports = {
  //list of entry points
  entry: [
    __dirname + '/src/index.js'
  ]
}
```

This indicates which file (relative to the _folder you are currently in_) is the "start" or "entry point" into your program&mdash;in a way, which file has "main" in it. In our case, `src/index.js` (the `__dirname` is a Node constant referring to the current working directory). Note that the `entry` property is actually an _array_, since it's possible to support multiple entry points.

We'll skip step 2 for a moment and also specify where to save the transformed files. This is specified as the `output` property of the config object:

```javascript
module.exports = {
  entry: [
    __dirname + '/src/index.js'
  ],
  output: {
    filename: "bundle.js",
    path: __dirname + '/dist'
  },
}
```

The `output` property is itself an object with more details about the output (rather than an array of possible outputs). The above example says that we should output into the `dist` folder (in the current directory), combining the code into a file called `bundle.js`.

Adding to your `package.json` file, we'll need to update the `scripts` key:
```
...
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "build": "webpack"
},
...
```

Since we don't have webpack installed globally (which you're welcome to do if you wish `npm install -g webpack`), we'll need to add to our `package.json` file, to tell `npm` what to do with our local files!

Now, if you run `npm run build`, the project will build! Similar to `npm install`, when we run this command, we tell `npm` that, within this project, we want to `run` the `build` script, which will call the local version of `webpack` to run. When `webpack` runs, it looks for the `webpack.config.js` file, and then executes using that file to build your project!

Here is roughly the output you should see:
```
> webpack-demo@1.0.0 build /Users/evanfrawley/Github/webpack-demo
> webpack

Hash: fc5c075a9cfa7ee938c0
Version: webpack 3.8.1
Time: 56ms
    Asset  Size  Chunks             Chunk Names
bundle.js  3 kB       0  [emitted]  main
   [0] multi ./src/index.js 28 bytes {0} [built]
   [1] ./src/index.js 390 bytes {0} [built]
```

This will create a new file `dist/bundle.js`. If you view this file in VS Code, you'll see it contains some extra code that organizes the different **modules** into **functions** (to support variable scoping), one of which is the content of the `index.js` file!


## Loaders

[Look at all this red!](https://kangax.github.io/compat-table/es6/) This is the support that ES6 has among today's modern browsers. What does this mean? While ES6 features are being adopted more and more, there isn't great _*uniform*_ support for it. So, we have to do a little trick called _*transpiling*_ our code so that all browsers know what our Javascript is doing.

To do this, we use what are called **loaders**. A loader is basically a plugin that is used to perform a particular transformation (e.g., transpiling JSX or even ES 6 syntax!). Webpack's strength is its set of loaders that enable it to handle pretty much any kind of file and transformation. (It's of course possible to write your own, but that's well beyond the scope of this tutorial).

The `module` property is used to specify the list of `loaders`:

```javascript
module.exports = {
  entry: [
    __dirname + '/src/index.js'
  ],
  output: {
    filename: "bundle.js",
    path: __dirname + '/dist'
  },
  module: {
    loaders: [
      {
        test: /\.jsx?$/,
        loaders: ["babel-loader"],
      }
    ]
  },
}
```

The `module` property is an object that itself contains a `loaders` property, which is an _array_ of loaders to apply. Each loader is described as an object (see? The nesting really does occur!).

- The `test` property indicates which file types we want the loader to transform. This may look scary and confusion, but that's because it's using a [Regular Expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions) (similar to how you split words in the JavaScript Challenge) to specify the _file extension_ that we want to consider. This particular expression indicates files that **end in** `.js` **or** `.jsx` (the later is often used for JSX React components). How to make sense of this expression and its crazy punctuation:

    - Regular expressions are like Strings, but surrounded by `/ /` instead of `" "`
    - The `.` in `.js` needs to be _escaped_, so has a `\` in front of it. Like `\n` for newlines.
    - The `x` in `.jsx` is _op  tional_ (without it we have `.js`, which is fine), so is written with a `?` after it to indicate that it can be present or not.
    - The last `$` indicates the "end of the line", so means we'll only talk about files that _end_ in `.js` (e.g., `libray.js.css` wouldn't get transformed).

- the `loaders` property is a list of which loaders we want to apply to files whose names match the "test". In our case, we use [Babel](http://babeljs.io/docs/setup/#installation) to transform our JSX, so we'll be using `"babel"` as our loader.


### Babel Loader
Loaders such as Babel need to be installed individually using `npm` as if they were separate programs (since they are in fact separate libraries!). This we will need to install the `babel-loader` package to be able to apply Babel transformations:

```apache
npm install babel-loader --save-dev
```

- (The `--save-dev` argument here is like `--save` in that it saves the dependency into your `package.json` file. However, `--save-dev` lists the dependency as only needed for _development_, not for _deployment_. Thus if you wanted to upload your code to a web server (like on [AWS](https://aws.amazon.com/)), this would let that server know that it doesn't need to install Babel because you've already transpiled the code into a production build).

But because nothing is ever simple, the `babel-loader` actually requires an additional library (`babel-core`, which is the Babel program itself) to do its work. Thus we also need to install:

```apache
npm install babel-core --save-dev
```

Babel is able to perform all kinds of transformations, such as compiling JSX and converting ES6 syntax into older, browser-compatible versions. Babel is very modular, so each transformation we want to apply can be downloaded as an individual libraries called [**presets**](https://babeljs.io/docs/plugins/).

```apache
npm install babel-preset-es2015 --save-dev
```

This installs `babel-preset-es2015` (the transformation for ES 6).

Of course, downloading to preset transformations doesn't automatically tell Babel to use them. To do that, we actually need to create _another file_ that will contain which presets Babel should use. This file is called **`.babelrc`** ("rc" stands for "run commands"; note the leading `.` indicating a hidden file). Create this file **in the same directory as `webpack.config.js`**.

Your `.babelrc` file just contains some JSON indicating which presets to use (other options are possible as well):

```json
{
  "presets": ["es2015"]
}
```

Altogether, you've specified what Babel transformations to apply (in `.babelrc`), and told your Webpack config file to use Babel to modify any JavaScript files.

So finally, you should be able to use `webpack` to build a working version of your React program _next week_! Open the `index.html` file in a Browser to see your lovely app.

Some things to note:

- All your code is inside the `bundle.js` file.
- Whenever you change your code, you will need to "re-build" your application (run `webpack` again). There are further webpack plugins that can help automate this, such as ones that will automatically refresh the page when the source files change.

## Clean up :bath:

Now that we've put in the hard work making webpack bundle our files, let's actually convert our files to proper ES6 synxtax and use correct import statements, given that only Chrome supports importing natively with [special exceptions](http://caniuse.com/#feat=imports).

1. Be sure to export the functions from `src/sorter.js`, `src/looper.js` and `src/printer.js`!
2. Now that they're exported, use ES6 syntax in `src/index.js` to import the functions
> Hint: 3rd syntax in the list [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)
3. Delete the `index.html` `<script>` tags for your files.
4. One last thing: in `src/sorter.js`, we're using an npm package called [`lodash`](https://lodash.com/) to sort the array. Run `npm install --save lodash` (`--save` because we want the module to be used in our final app) and then import `lodash` in the `src/sorter.js` file. Hint: use the second import syntax from the MDN docs

With any luck, running `npm run build` one last time should give you a good output, and finally, opening `index.html` and looking at the console log should have your sorted user array!

## Further Loader Practice
The webpage doesn't look great yet, because there is no styling (CSS) involved. To practice working with webpack loaders, add an `import` for `main.css` stylesheet to your `index.js`, and modify `webpack.config.js` so that it will bundle that file:

1. Add the import to your `index.js` file. The file path should be _relative_ to the `index.js` file.

2. Install the **`style-loader`** and **`css-loader`** webpack loaders (remember to `--save-dev`). Together, these loaders are able to handle CSS files.

3. Add another element to the `module.loaders` array in the `webpack.config.js` file to specify the `style-loader` transformation.

    - The `test` should be a regular expression for files ending in **`.css`**
    - The `loader` itself should be `"style-loader!css-loader"`, which refers to the "css loader" module for the style-loader plugin.

4. Re-build your application using `webpack`. If you reload the page, you should now see it has a gray background!


## Next Steps
There are lots of further configurations and options used by Webpack. For example, you can use  the [webpack-dev-server](http://webpack.github.io/docs/webpack-dev-server.html) to have webpack run a local server that will automatically re-bundle modules when the files change. You are encouraged to check out that example if you have time.
