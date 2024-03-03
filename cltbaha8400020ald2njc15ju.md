---
title: "Building Frontend Applications across decades"
seoTitle: "Building Frontend Applications across decades"
seoDescription: "From YOLOing Script Tags to using Task Runners to Module Bundlers"
datePublished: Sun Mar 03 2024 09:07:44 GMT+0000 (Coordinated Universal Time)
cuid: cltbaha8400020ald2njc15ju
slug: building-frontend-applications-across-decades
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1709456613076/10c2775a-f4d8-4463-99ff-830887c53269.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1709456783246/13398293-29c5-452e-988c-7edfff4fa894.png
tags: javascript, webpack, rust, vite, module-bundler

---

Building your Frontend applications is not the same, and will not be the same. Here we will go chapter-wise on how the whole frontend bundling experience is like

* Chapter 1: YOLOing `<script>` tags all the way down
    
* Chapter 2: The Original "G"s : Grunt and Gulp Task Runner
    
* Intermission 1: JS Standards and rise of transpilers and Parsers
    
* Chapter 3: Forgotten in time; Bower and Browserify
    
* Chapter 4: Module Bundlers Mashups (Webpack, Rollup and Parcel)
    
* Intermission 2: Make JS Blazingly fast by not using JS (Esbuild, SWC)
    
* Chapter 5: Rising stars; Rspack, Vite and Turbopack
    

# Chapter 1: YOLOing `<script>` tags all the way down

> **script** tags as UI bundling with YOLOing the prod UI bundle

Consider a situation where you are improving JS scripts from a CDN using the `script` tag. You have a main library like jQuery which has all helper methods for manipulating DOM, AJAX calls etc. For some reason, there is another library that is meant for improving async AJAX calls. Let us call it `ajax-plugin.js`. Since jQuery already comes with AJAX, the `ajax-plugin` overides this behaviour. But since the plugin library did not consider that your were using the same function name as jQuery; hence you find an issue making both of them work. Example can be seen in the following Replit project;

%[https://replit.com/@AymanArif1/01ScriptLoading] 

Here in `index_script_overide.html`, we have loaded the scripts like this:

```html
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="ajax-plugin.js"></script> <!-- Include the custom plugin -->
```

When you run this, you can see the jQuery loads before the culprit JS file `ajax-plugin.js` that overides the `ajax` function of jQuery; which creates a Maximum Stack error:

In `index.html`, jQuery is loaded at the end;

```html
<script src="ajax-plugin.js"></script> <!-- Include the custom plugin -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
```

This will work, but the solution is **YOLO (You Only Live Once)** as a production grade application's script dependencies is a cobbled web which will be impossible to untangle.

Hence, ordering matters in script imports for Javascript files which is a big headache if done manually. Hence, there needed to be a solution that made sure that these type of issues do not come.

# Chapter 2: The Original "G"s : Grunt and Gulp Task Runner

> Grunt Gulp galore gangsters

## Grunt

\==demo not possible; provide psedo code==

`Gruntfile.js`

```javascript
module.exports = function(grunt) {
  // 1. Project configuration
  grunt.initConfig({
    // Tasks configuration
    concat: {
      options: {
        separator: ';',
      },
      dist: {
        src: ['src/*.js'],
        dest: 'dist/bundle.js',
      },
    },
    uglify: {
      dist: {
        files: {
          'dist/bundle.min.js': ['<%= concat.dist.dest %>'],
        },
      },
    },
    watch: {
      scripts: {
        files: ['src/*.js'],
        tasks: ['concat', 'uglify'],
        options: {
          spawn: false,
        },
      },
    },
  });

  // 2. Load plugins
  grunt.loadNpmTasks('grunt-contrib-concat');
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-watch');

  // 3. Register task(s)
  grunt.registerTask('default', ['concat', 'uglify', 'watch']);
};
```

## Gulp

`gulpfile.js`

```javascript
const gulp = require('gulp');
const concat = require('gulp-concat');
const uglify = require('gulp-uglify');
const rename = require('gulp-rename');

// TODO: Add task for browserify

// Concatenate and minify JavaScript files
gulp.task('scripts', function() {
  return gulp.src('src/*.js')
    .pipe(concat('bundle.js'))
    .pipe(gulp.dest('dist'))
    .pipe(rename('bundle.min.js'))
    .pipe(uglify())
    .pipe(gulp.dest('dist'));
});

// Watch for changes in JavaScript files
gulp.task('watch', function() {
  gulp.watch('src/*.js', gulp.series('scripts'));
});

// Default task
gulp.task('default', gulp.series('scripts', 'watch'));
```

Task runner lead to *Big Ball of Mud* with no good abstractions on how to load a JS and CSS. It was all custom scripts which were brittle and hard to extend. That is why tools such as Rollup and Webpack came and became dominant in building JS applications!

# Intermission: JS Standards and rise of transpilers and Parsers

> IE deviation, ES6 migration and dread of polyfills

## **ES6 Migration and the Need for Babel**

With the evolution of JavaScript, ECMAScript 6 (ES6), introduced significant enhancements to the language, providing developers with powerful features to write more concise, readable, and maintainable code. However, the widespread adoption of ES6 posed a challenge for developers, particularly those working on projects with existing codebases or targeting older browsers such as IE11 that lacked support for ES6 features out-of-the-box.

To address this compatibility issue, developers turned to transpilers like Babel. This converted non-ES6 code into ES6 with help of polyfills

# Chapter 3: Forgotten in time

> You will be remembered for your service:
> 
> 1. Bower
>     
> 2. Browserify
>     

## Bower

It was released in early 2010s (around 2012). Node.js was 3 years old in 2009 and npm was released in 2010. JS import such as **CommonJS** were becoming the standard on server-side Node.js. Frontend world were not reaping the benefit of a package manager like NPM and *CommonJS*.

Bower solved these 2 problems. It created things:

1. A *package manager* that works on the web.
    
2. Making *CommonJS* work on the web
    

#### Package Manager

When you install frontend packages in Bower it would go to `bower_components` folder (not the usual `node_modules` folder which is the norm now!)

#### RequireJS

Bower folder structure

```bash
bower_project/
├── bower_components/
│   ├── package1/
│   │   ├── dist/
│   │   ├── src/
│   │   └── bower.json
│   ├── package2/
│   │   ├── dist/
│   │   ├── src/
│   │   └── bower.json
│   └── ...
├── build/
├── src/
├── bower.json
└── .bowerrc
```

Downfall of Bower was its own usage of modules folder. Also NPM picked up support for module resolution and usage of frontend libraries which eventually made Bower useless.

## Browserify

It is a spiritual successor to Bower. It didn't create its own Package manager but allowed to write **CommonJS** imports on the browser.

Browserify allowed Node-speicfic library packages such as [events, stre](http://nodejs.org/docs/latest/api/events.html)[am, path](http://nodejs.org/docs/latest/api/stream.html)[, ur](http://nodejs.org/docs/latest/api/path.html)[l, a](http://nodejs.org/docs/latest/api/url.html)[ssert, buff](http://nodejs.org/docs/latest/api/assert.html)[er, util](https://github.com/feross/buffer)[, qu](http://nodejs.org/docs/latest/api/util.html)[erystring, http](http://nodejs.org/docs/latest/api/querystring.html)[, vm](https://github.com/jhiesey/stream-http)[,](https://github.com/browserify/vm-browserify) and [crypto](https://github.com/crypto-browserify/crypto-browserify) into the browser.

The architecture of Browserify did not consider features such as

1. Multiple entry points per-page
    
2. ES Modules as Browserify was built on making Node's CommonJS available to browser
    

# Chapter 4: Module Bundlers Mashups

> Fourth way of building UI (after Yolo Scripts, Task Runners and Third wave building tools)
> 
> 1. Webpack
>     
> 2. Rollup
>     
> 3. Parcel
>     

## Webpack

> This is the tool I have worked with the most. But even Webpack teaches me something new when I back to it!

> *Core Strength:* **Plugin ecosystem.**

Webpack is the defacto Module bundler for frontend applications. Angular uses it under the hood. Create-React-App uses Webpack which can be modifiend when you run the `npm run eject` command.

Webpack was created in mid-2010s to solve the issue of Big-ball of mud codebases that were created when using task runners such as Gulp and Grunt. It uses a better abstraction model of using plugins and loaders for trasnpiling source code into production build. It also provided a `webpack-dev-server` which just works without manually configuring like it used to be in Grunt via `watch mode`

Webpack Example:

<iframe src="https://codesandbox.io/p/devbox/quiet-shape-9ghj28?embed=1&file=%2Fwebpack%2Fwebpack.dev.js" style="width:100%;height:500px;border:0;border-radius:4px;overflow:hidden" sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"></iframe>

## Rollup

Second-place in popularity and maturity. It has a rich ecosystem of APIs to hook into, which is why Vue uses it for its production build.

## Parcel

Less popular than Rollup and Webpack but still used today.

# Intermission 2: Make JS Blazingly fast by not using JS

Before we go to the final section of modern modern bundlers, we need to talk about what improvementsare being done in ther JS ecosystem. It is all about reduing bloat that were used in the past due to differfent browser suppot, usage of lighter and fast runtimes such as Rust, Golang, Zig etc

A Logan would say;

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709455107506/29644303-e096-41ee-9388-3c89009e93c6.jpeg align="left")

## ESBuild

Go based bundler that is faster than Webpack, Rollup and Parcel

It is used in Vite in the pre-bundling process.

## SWC

Rust based Babel alternative Used by Next.js, Parcel and Deno. Next.js uses SWC for 2 things:

1. Replacement of Babel for transpiling Typescript to Javascript
    
2. Replacement for [TerserPlugin](https://webpack.js.org/plugins/terser-webpack-plugin/) (Minifies JS for smaller bundle sizes)
    

In the following simple Hello World React + Typescript example, you can see the performance improvements of using SWC and ESBuild

%[https://replit.com/@AymanArif1/02swcesbuildtsc] 

| Tool | Time | Speed difference |
| --- | --- | --- |
| tsc | 15.77s | base |
| swc | 2.85s | 5.5 times faster than tsc |
| esbuild | 1.68s | 9 times faster than tsc |

## Vite

> **Fast** module bundler that combines all the great modern browser featres

Vite is a amalgamation of a lot of modern JS technologies such as Esbuild, Rollup, ES Modules

### Blazingly fast Dev Mode

Levereges ESBuild and ES Modules to make vite HMR (aka Livereloading) blazingly fast

1. ESBuild usage in Vite:
    

Vite uses ESBuild for pre-bundling `node_modules` dependencies. 2. ES Modules usage in Vite: Vite uses ESM do development code inside project.

`Vite Local Start -> Esbuild for libraries -> ESModule for local project files -> Browser`

### Blazingly fast Prod build

Leverages Rollup to make builds faster. There are plans by Vite to create its own bundler **Rolldown** in the future.

### Blazingly fast Test execution

Vite ecosystem provides a Jest compatible Vitest for running Unit test cases faster.

Vite example:

<iframe src="https://codesandbox.io/p/devbox/webpack-react-typescript-w8fyy2?embed=1&file=%2Findex.html" style="width:100%;height:500px;border:0;border-radius:4px;overflow:hidden" sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"></iframe>

## Turbopack

> Vercel'answer to build tools with blazingly fasst builds for Next.js, Svelete and other Vercel-supported platforms

%[https://www.youtube.com/watch?v=_w0Ikk4JY7U&t=86s] 

## Rspack

> Blazingly fast Webpack written in **Rust** @ Bytedance

%[https://www.youtube.com/watch?v=rgLMfSYpn5s]