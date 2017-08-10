---
title: Using global style rules in a Vue.js app
categories:
  - JavaScript
tags:
  - Vue.js
  - ES6
  - Webpack
  - CSS
  - Stylus
thumbnail: /images/javascript.png
---

If you are new to vue-cli and the webpack template, you may have encountered the situation where you did not really know how to declare global style rules for your project. This article will show you how to do this, with regular CSS, but also with a CSS preprocessor. For the sake of simplicity, we will use Stylus with the stylus-loader for Webpack...

<!-- more -->

## Global CSS rules

One big strength of Vue.js is its modular nature. Just like a puzzle, Vue allows you to build complex applications using components. Each component is ideally represented in a `.vue` file which contains its own template, script and style. A *single file component* is a convenient way to encapsulate all the code of a specific building block, but what happens if we need some global CSS rules, for instance to set the default font family or background image?

### Unscoped style section in `App.vue`

Style rules declared in `.vue` files are processed by Webpack thanks to **vue-loader** and **PostCSS**. When a style section is scoped, these rules only apply to the component in which they are defined. This means that you will not be able to style the body element from a scoped style section. `<body>` is actually defined in your `index.html` and is not part of any component template.

Suppose we are in `App.vue`. This will not have any effect:

```JavaScript
<style scoped>
  body {
    background-color: black;
  }
</style>
```

Now if we remove the `scoped` attribute, the background color will be modified! An unscoped style section for `App.vue` is therefore a nice place to put some global CSS rules that are not specific to a component. However, `App.vue` is a component itself, which may be a bit counterintuitive...

### Global imports in `main.js`

We have seen that style sections are processed by Webpack, but in fact, you do not need a style section in a `.vue` file to give some work to Webpack. You can simply import your CSS as a  dependency in your `main.js` file:

```JavaScript
import './assets/style.css'
```

It works, but in a Vue.js project, Webpack is mostly used to compile single file components. So when a stylesheet is not related to any specific component, it might be interesting to put it in the `static` folder to distiguish it easily.

### Static stylesheet

In the `static` folder, resources are not processed by Webpack. If you put a `style.css` in there, you will have to reference it manually in your `index.html` like so:

`<link rel="stylesheet" href="/static/style.css">`

## Global Stylus variables

Webpack relies on **loaders** to process resources. If you use a CSS preprocessor like Sass, Less or Stylus, you will need the corresponding loader for Webpack: `sass-loader`, `less-loader` or `stylus-loader`. For this particular example, we will use Stylus which is the most flexible and easiest to configure in a Vue.js project.

First of all, install Stylus and its loader in your project: `npm i stylus stylus-loader -D`. We put them in dev dependencies because they will not be required in production.

If CSS preprocessors are part of your ordinary workflow, you probably like to create global files that contain some variables or mixins. For example, a common use case is to have all colors from a design in a single file using variables. It is useful because you can then use clear variable names in your code instead of a more obscure hexadecimal notation. In your Vue.js project, you may want to create such a file, named `colors.styl`, with the following content:

```CSS
$red: #800000;
$green: #41B883;
$blue: #8ED6FB;
```

So far so good, but... How do you include that in your project?

### Raw `@import` rule in each style section

The most basic approach is to import your `colors.styl` in the style section of each component using the following syntax:

```CSS
@import '../assets/stylus/colors.styl'
```

Problem: it strongly violates the DRY (Don't Repeat Yourself) principle because you will have to write this rule again and again... In large projects where there are lots of components, this is not a viable solution. If you move your file, you will have to modify your import everywhere.

### `@import` rule with a Webpack alias

A better approach is to use a Webpack alias to avoid repetitive modifications in case you move your `styl` file. If you installed vue-router through vue-cli, you probably saw some `@` in import paths. This is actually an alias that corresponds to `src`. We can then apply the same principle to references some global Stylus files. In the `resolve.alias` section of your `build/webpack.base.conf.js`, add this code:

```JavaScript
  styl: path.resolve(__dirname, '../src/assets/stylus')
```

In your components, you should now be able to import your `colors.styl` like so:

```CSS
@import '~styl/colors.styl'
```

This is undoubtedly better, but what happens if you rename `colors.styl` to `variables.styl`? You will have to modify your imports in all components...

### Import in all components from Webpack config

The best solution, by far, is to add a `plugins` section at the end of your `webpack.base.conf.js` to configure an auto-import.

```JavaScript
var webpack = require('webpack') // Do not forget to add this dependency, or else you will get an error

// ...

plugins: [
  new webpack.LoaderOptionsPlugin({
    options: {
      stylus: {
        import: [path.resolve(__dirname, '../src/assets/stylus/colors.styl')]
      }
    }
  })
]
```
