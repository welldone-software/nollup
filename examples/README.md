# Examples

## Getting Started

* Install packages inside ```/examples``` with ```npm install```.
* Install packages inside the example you want to run.
* Start the example inside the example directory with ```npm start```.
* You can also build the project with Rollup using ```npm run build```.

## Preact Example

* Uses Preact for components.
* Has a few basic components with CSS.
* Uses Express to serve content along with Nollup Dev Server.
* Build script that uses Rollup with the same configuration.
* Custom Rollup plugin for loading styles from a link tag.
* Styles supports Hot Module Replacement (HMR).
* A rough version of how to support Hot Module Replacement for JavaScript is demonstrated.

## React Hot Loader Example

* Uses React for components.
* For HMR supports, uses React Hot Loader.
* Uses a custom CommonJS plugin that works with React Hot Loader.

## React Hot Loader Issues

As the RHL example demonstrates, it's possible to use React Hot Loader with Nollup's HMR support, but there is issues.
If you look inside the ```plugins``` directory, there's a custom CommonJS plugin. The standard ```rollup-plugin-commonjs``` is not
compatible with React Hot Loader. Here's a list of the problems that are mostly solved by the custom CommonJS plugin:

### Conditional Requires

React libraries in general use an entry script that checks a condition to see whether or not to load the development
or production version of the library. Assuming you're using ```rollup-plugin-commonjs```, it will detect require calls
but will include both versions of the library, because it does not check for dead branches. This can cause some conflicts to occur. 
The custom plugin solves this by statically checking if the ```require``` function is inside an ```if``` statement.

### Variable Replacement

React libraries typically reference non-existent variables to determine not only what library to load, but also what
functionality to enable. The most commonly referenced variable is ```process.env.NODE_ENV```. This variable does not
exist in the browser so when executed the script will fail. Another issue is with ```rollup-plugin-commonjs```, where
it will detect dynamic require calls, and stub them out with an error message. HMR requires dynamic ```require``` calls using
module ids. Using ```rollup-plugin-replace```, you can replace these variables. The custom CommonJS plugin does not stub
any require calls. 

    replace({
        'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV)
    })

### Babel Plugin

RHL requires that you use a Babel plugin. This Babel plugin modifies all of your component files, and automatically
registers them with RHL, so it can do a clean replacement where it will try to preserve state as much as possible.
Unfortunately, this Babel plugin assumes that you're using a bundler that allows you to put in ```require``` calls
inside ES modules that are using ```import```. The ```rollup-plugin-commonjs``` has a check to see that if it's reading
an ES module by checking for the presence of ```import``` and ```export``` statements. If it finds those, it will
skip parsing the module, so the ```require``` calls that the Babel plugin injects will be ignored and will break.
The custom CommonJS plugin will detect ```require``` calls in every file, including ES modules.

## Opinion about Web Tooling

This is just my personal opinion here, but I believe there's a huge problem with how we're 
approaching tooling in the web community. The original intent of tools like Browserify, was to re-use existing JS code
in both the server and client. Some workarounds were implemented to handle NodeJS specific idioms. With ES modules, we now
have a proper standard that we can statically parse and that can be supported by both the server and client, so in theory it should make
creating tools a lot easier. 

However, because modern tools like Webpack and Parcel have support for legacy Node syntax, developers have opted to keep 
using those legacy features to make configuration easier for the developer. This causes a problem, because it means that new 
tools will always have to support those legacy features. While this doesn't seem like a big challenge at first, it does quickly
turn into a mess. Bundlers have to use static analysis to determine what files to include, but Node allows you to dynamically
include modules. This is fundamentally incompatible. Whatever strategies bundlers implement, they will always have flaws. 

Here's what I believe we need to do.

* Frameworks like React need to start outputting ES modules, and avoid using ```require``` and ```module.exports``` which are now legacy.
* Frameworks like React need to stop using NodeJS idioms like conditional requires, which have no ES modules equivalent. Statically parsing conditional requires reduces performance, and will never be perfect because it's being statically analyzed. 
* Frameworks need to stop assuming developers are using Babel. Some of us prefer using lighter transpilers such as Bublé in order to minimize file size.
* The Rollup CommonJS plugin needs an overhaul, first by not checking for ES modules, and secondly by not stubbing out require calls. 
* We need to stop hiding configuration from the developer. Let's be honest, the main reason people hate configuration is because Webpack's configuration system is a nightmare. While the idea of zero-configuration is cool, it's not really useful for more serious projects where you need to implement custom bundling optimisations that work for your use case. What we should be aiming for is configuration that's easy to understand, which is why I love Rollup. Because Rollup doesn't try to do a lot, it actually makes it easier to work with.