---
layout: post
title: "Speeding up your development with Webpack 5 HMR and React Fast Refresh"
date: 2023-04-23 20:00:00 +0400
tags: frontend build-tools
---

#### Part One: Phonecalls

Recently I have received a phone call on my answering machine from my manager: I was assigned to work on a legacy project. It was not a very complex task, but still rather time-consuming. I started working on it with a "quick in-and-out" attitude, not intending to do any serious changes to the project. However, as time went on, I realized that I am spending a lot of time repeating the actions required to get to the UI component I was working on after the page refreshes after a code change. Every project I usually work on has at least HMR setup, but here I was faced with a reload after the slightest CSS change. So I decided to dig in and get HMR and React fast-refresh up and running in order to speed up the process.

#### Part Two: Questions

What even is HMR and React Fast Refresh?

Let's start with the first one - HMR or Hot Module Replacement. This is a [feature of `webpack`](https://webpack.js.org/concepts/hot-module-replacement/) which has been around for quite some time now, it is enabled by default in a [popular bootstrapping package `create-react-app`](https://github.com/facebook/create-react-app). It enables your app to swap modules while it is running (the "Hot" of "HMR"), without a full page reload and losing the app's state. However, it is hard to retain the state of a module when it is something complex, like a stateful React component. That is why a group of wonderful people developed React Fast Refresh.

React Fast Refresh is a younger cousin of another similar feature - Hot Reloading, but it is officially supported by React and is stated to be more reliable on its [README page](https://www.npmjs.com/package/react-refresh). Now, is it possible that someone blatantly lied in the README file? I know I did that a few times, but from my experience, this is not the case with `react-refresh`. It handles even very complex component changes very well. Once again, projects set up with `create-react-app@^4.0.0` have it enabled by default.

#### Part Three: Connections

The project that I was assigned to work had an outdated `webpack` and `react` version so I went ahead and updated `react` to `^17.0.0` and `webpack` with `webpack-dev-server` to `^5.0.0`.

---

#### Disclaimer

While working on legacy projects you have to be aware of the risks that you are taking when making large changes to the codebase (like updating the bundler and core framework to the next major version). If you do not have automated tests set up or do not have enough QA resources to thoroughly test the software after such an update, I strongly urge you to consider other options if it is possible.

---

After fixing a number of dependencies issues and seeing a green light on my CI dashboard, I proceeded to set up HMR and React Fast Refresh.

#### Setting up HMR

##### Packages used

| Package              | Version   |
| -------------------- | --------- |
| `webpack`            | `^5.0.0`  |
| `react`, `react-dom` | `^17.0.0` |

This one could be as simple as slightly editing your `devServer` section of `webpack.config.js`:

```javascript
...
  devServer: {
    ...
    hot: true,
    ...
  },
...
```

This line tells your `webpack-dev-server` to enable HMR. The last step is enabling `webpack.HotModuleReplacementPlugin`. You could do that in the config file manually, but I suggest you to take a safer route and add `--hot` to your `package.json` `start` (or whatever name you prefer to run your project in development mode) script to make sure that the plugin is used only in the development environment:

```javascript
...
  "scripts": {
    ...
    "start": "webpack serve --mode=development --hot",
    ...
  },
...
```

That should do it in most cases and you are set up with flawless reloading of CSS and other assets without any extra work. However, editing a React component will probably result in a full page refresh, and all the app state is still lost.

#### Setting up React Fast Refresh

---

#### Disclaimer

This part uses an **experimental** webpack plugin that might not handle unknown edge cases. From my personal experience, I have had no issues with it at the time of writing. Still, proceed with caution.

---

##### Packages used

| Package              | Version   |
| -------------------- | --------- |
| `webpack`            | `^5.0.0`  |
| `react`, `react-dom` | `^17.0.0` |
| `babel-loader`       | `^8.2.2`  |

The `react-refresh` npm package that I mentioned earlier is intended to be used by bundler authors. If you want to enable React Fast Refresh in your project, you should check out [the `react-fast-refresh-webpack-plugin`](https://github.com/pmmmwh/react-refresh-webpack-plugin). There is an extensive installation and setup guide, but I will go through these steps here as well.

Its setup is a bit more complex than HMR, but should not be a big problem anyway. First of all, you need to install all the required dependencies:

```bash
yarn add -D @pmmmwh/react-refresh-webpack-plugin react-refresh
```

There are 2 parts of enabling this feature:

1. Adding the `react-refresh/babel` to `babel-loader` plugins.
2. Adding the `react-refresh-webpack-plugin` to `webpack` plugins.

Same as with HMR, enabling React Fast Refresh on production is a huge vulnerability, we need to make sure it is enabled only in the development environment. `webpack@^5.0.0` strongly suggests using a `--mode` parameter, so we could use its value as the source of truth to enable the plugins when necessary. To get the `--mode` parameter value we need our `webpack` config file to export a function, so you can just wrap your existing config in an arrow function like this:

```javascript
// the first parameter in a function
// webpack config is "env" [1]
// which is not used in this example
// so its name is set to "_" to indicate
// that a parameter is being passed,
// but we do not use it
module.exports = (_, argv) => {
  const mode = argv.mode;
  const isDevelopment = mode === "development";
  return {
    ...
    // your existing webpack configuration
    ...
  }
};
```

<sup>[1] - More information about the `env` parameter available in [webpack docs](https://webpack.js.org/guides/environment-variables/).</sup>

Now that we have the handy `isDevelopment` constant, we can edit the rule for loading JS files to conditionally include `react-refresh/babel`:

```javascript
...
  rules: [
  {
    test: /\.js$/,
    ...
    use: {
      loader: "babel-loader",
      options: {
        plugins: [
          // this code will evaluate to "false" when
          // "isDevelopment" is "false"
          // otherwise it will return the plugin
          isDevelopment && require("react-refresh/babel")
        // this line removes falsy values from the array
        ].filter(Boolean),
      },
    },
  },
...
```

Then onto the webpack's `plugins` section:

```javascript
...
  plugins: [
    ...
    isDevelopment && new ReactRefreshWebpackPlugin(),
    ...
  ].filter(Boolean),
...
```

Now that we have both parts set up, you should have React Fast Refresh in your project in development mode.

#### Part Four: Answers

Now, whenever I start the project with the following command:

```bash
webpack serve --hot --mode=development
```

I enjoy the development process with as few page reloads as possible, so I can instantly see the changes I made in the code immediately take effect in the app. This made the time-consuming task I was assigned with way less time-consuming and a bit more fun.
