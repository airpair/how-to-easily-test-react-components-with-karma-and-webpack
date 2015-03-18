## Useless background

About 8 months ago, I started looking around for solutions for how to test React components but didn't have any good ideas. I first tried out QUnit, which seemed to work fine, but I wanted it to be run automatically. I tried a qunit runner kind of like how the Ember.js project used to run tests, but that ended up being too cumbersome too. I switched over to Karma to test my components, and spent my time building my test assets and loading them up to Karma, but that ended up being kind of slow because I never cached the builds and ran a clean webpack build for each bundle. Ugh. Facebook went on to release Jest and I switched over. I thought it was great, but I quickly ran into some issues: 1) I don't like the module mocking that makes things look like black boxes, 2) I never actually wanted/needed black boxing, 3) it was slow, 4) it didn't install on all of my peers' computers without problem, 5) the documentation was lacking, 6) I had to come up with a hacky way to call it from Node, 7) it didn't work with io.js when I first tried it, 8) I didn't really know how to debug my tests other than by the really long stack traces, and the biggest thing I didn't like was that 9) it didn't run in a real browser.

This post exists because 1) the guys writing react-router are GODS, 2) every other blog post on testing React components I've seen has been super freaking complicated, and 3) because this setup is the best thing I've found wrt React since pure rendering.

## What does this look like when you've finished it?

![Screen Shot 2015-02-21 at 7.32.17 PM.png](https://qiita-image-store.s3.amazonaws.com/0/42481/062116c4-ac2a-cf2e-fbea-922164ef9c23.png "Screen Shot 2015-02-21 at 7.32.17 PM.png")

## Why would I want this?

![Screen Shot 2015-02-21 at 7.33.57 PM.png](https://qiita-image-store.s3.amazonaws.com/0/42481/463d71c8-4421-0066-1fbb-92c7503f8129.png "Screen Shot 2015-02-21 at 7.33.57 PM.png")

When has testing and debugging with full sourcemaps been this easy???

## Okay, how do I get this?

### Installation

Dependencies you'll probably need:

* karma -- the actual test runner
* karma-cli -- the cli for karma
* karma-mocha -- for using the mocha test framework with karma
* karma-webpack -- for using webpack to actually preprocess sources for karma
* karma-sourcemap-loader -- for loading up sourcemaps into karma
* karma-chrome-launcher -- for launching chrome
* expect -- Michael Jackson's (not to be confused with the famous singer/dancer) amazingly nice assertion library
* babel-loader -- or some kind of loader for your JSX files
* react -- well, this is what you use, right?
* webpack -- webpack, the most amazing browser build tool ever

### Configuration

Karma reads from a `karma.conf.js` file, so let's make sure we set it up right.

```javascript
//karma.conf.js
var webpack = require('webpack');

module.exports = function (config) {
  config.set({
    browsers: [ 'Chrome' ], //run in Chrome
    singleRun: true, //just run once by default
    frameworks: [ 'mocha' ], //use the mocha test framework
    files: [
      'tests.webpack.js' //just load this file
    ],
    preprocessors: {
      'tests.webpack.js': [ 'webpack', 'sourcemap' ] //preprocess with webpack and our sourcemap loader
    },
    reporters: [ 'dots' ], //report results in this format
    webpack: { //kind of a copy of your webpack config
      devtool: 'inline-source-map', //just do inline source maps instead of the default
      module: {
        loaders: [
          { test: /\.js$/, loader: 'babel-loader' }
        ]
      }
    },
    webpackServer: {
      noInfo: true //please don't spam the console when running in karma!
    }
  });
};
```

Next, we need our single file, which will actually use the webpack require API to find the files we need automagically.

```javascript
//tests.webpack.js
var context = require.context('./src', true, /-test\.js$/); //make sure you have your directory and regex test set correctly!
context.keys().forEach(context);
```

### Write some tests

Tests really just need to prove things like `1 === 1`, so let's make sure it's easy to understand.

```javascript
//root-test.js
var React = require('react');
var TestUtils = require('react/lib/ReactTestUtils'); //I like using the Test Utils, but you can just use the DOM API instead.
var expect = require('expect');
var Root = require('../root'); //my root-test lives in components/__tests__/, so this is how I require in my components.

describe('root', function () {
  it('renders without problems', function () {
    var root = TestUtils.renderIntoDocument(<Root/>);
    expect(root).toExist();
  });
});
```

### Run the tests!

Just a simple `karma start` will run the tests once. `karma start --single-run=false` for multiple runs.

## Conclusion

And that's about it! Pretty minimal configuration to get a really nice testing environment set up. Big thanks to the react-router guys for having this up on GitHub to follow through.

## References

React-Router -- https://github.com/rackt/react-router/
Karma -- http://karma-runner.github.io/0.12/index.html
React TestUtils -- http://facebook.github.io/react/docs/test-utils.html
My Example repo -- https://github.com/justinwoo/react-karma-webpack-testing/

## Bonus

Making this work with Travis is trivial. Just do a few things.

### Modify what browser will run in Travis

Travis only comes with Firefox, so we should change that accordingly.

Install the Firefox launcher, `karma-firefox-launcher`, and then change your karma config accordingly to `browsers: [ process.env.CONTINUOUS_INTEGRATION ? 'Firefox' : 'Chrome' ],`. You can also use PhantomJS, if you really want to, using the correct loader.

## Configure Travis

Add a `.travis.yml` with the appropriate settings. You'll have to start up `xvfb` in Travis accordingly to make Firefox work.

```markup
//.travis.yml
language: node_js
node_js:
  - "0.10"
before_install:
  - "export DISPLAY=:99.0"
  - "sh -e /etc/init.d/xvfb start"
```

And then you're good to go!

See Travis in action in my demo repo: https://github.com/justinwoo/react-karma-webpack-testing (build log here: https://travis-ci.org/justinwoo/react-karma-webpack-testing/builds/51681700)
