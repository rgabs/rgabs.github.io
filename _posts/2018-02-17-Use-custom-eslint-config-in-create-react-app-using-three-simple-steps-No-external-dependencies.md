---
layout: post
title: "Use custom ESLint config in create-react-app using 3 simple steps (No external dependencies)"
date: 2019-02-17T01:50:00.000Z
intro: "This post will solve the problem of adding custom ESLint rules while keeping the default ones provided by create-react-app."
categories: front-end
---

# The Problem

[create-react-app](https://facebook.github.io/create-react-app/) is an amazing boilerplate to start building react app. It abstracts all the configurations such as bundle generation, test configuration, ESLint configuration, etc.

While it turns out to be really good for developers starting with react, it makes life difficult for people who want to make some modications in the configuration such as adding some more ESLint rules.

This post will solve the problem of adding custom ESLint rules while keeping the default ones provided by create-react-app.

# More Context

The current set of rules that create-react-app uses are coming from [eslint-config-react-app](https://www.npmjs.com/package/eslint-config-react-app) NPM package. These rules are handpicked by facebook team and are supposed to be the minimal set of rules. 

After digging a bit into the source code, I found out that the facebook team has intentionally disabled the feature of overriding eslint rules.

To see for yourself, head over to [`node_modules/react-scripts/config/webpack.config.js`](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/config/webpack.config.js#L319). It should look something like this:

```js
options: {
    ...
    baseConfig: {
        extends: [require.resolve('eslint-config-react-app')],
        ...
    },
    ignore: false,
    useEslintrc: false, // On line: 323 at the time of writing blog
    ...
},
```

If you change the value of `useEslintrc` to true, and create an `.eslintrc` file in your source directory, your updated rules would be used by the webpack bundler. But that's not recommended(explained in the next section).

_Now why facebook decided to do make `useEslintrc` false?_

They want the browser to fail/warn only when lint violations happen, not when your code style is not matching the convention set by your team.

__Style linting should be handled by CLI or CI-CD, not by webpack/browser.__


# Existing Solutions

1. `npm run eject`: This will remove all the abstraction and will give you control over all the configurations and NPM modules. While it gives you a lot of power, it has its own problems such as unable to update create-react-app, polluting package.json file, etc. Also, I think it doesn't make sense to eject just for adding some ESLint rules.

2. `Use a third party module/hack`: I found a few hacks by smart developers which lets us override exising ESLint config. Although they are pretty smart solutions and get the job done, I think they are hacky and unmaintainable. Moreover I wouldn't want to install and maintain a 3rd party module just to add a few ESLint rules. Here are some examples I found:
    + [Custom eslint config with create-react-app using react-app-rewired](https://medium.com/@adamdziendziel/custom-eslint-config-with-create-react-app-d6f66e8d61)
    + [Gist by [cncolder](https://github.com/cncolder)](https://gist.github.com/cncolder/19354ba59146c55071841f6fa274da66)

3. `Fork create react app`: You could fork the NPM module and change `useEslintrc: true`. But this would mean that you need to maintain a copy of `create-react-app`. Whenever CRA gets an update, you might need to pull the updated changes manually, which is again a tedius process.

# My solution

Let's take a step back and think why would we need to modify eslint rules.

1. __To edit/remove existing rules__: Though it shouldn't happen, if this is the requirment, we can rasie a PR to `create react app` and tell why the rule should be removed/changed.

2. __To add new ESLint rules__: These new rules would be defining the coding-style that you are following in the project(example: indentation, import order, etc.). These rules should be read by IDE along with the default ones coming from CRA.

### We can add new ESLint rules by follwing just 3 simple steps.

1. Create `.eslintrc.js` under the `src` folder of the your project.

2. Add the following lines in the file:

    ```js
    module.exports = {
        "extends": "react-app",
        "rules" : {
            "indent": ["error", 4] // A custom style-related rule for example
            // More custom rules here
        }
    }
    ```

    ^ This way, we will be telling our IDE to extend the base ESLint rules provided by the create-react-app and then follow some rules defined by us.

3. In your package.json, add the following script:
    ```json
    "lint:fix": "eslint src/**/*.js --fix",
    ```
    This is mainly for CLI and CI/CD environment.
    
Now default setup in most of the IDEs are that they would read the configuration from `src/.eslintrc.js`. Hence you should now see errors/warnings from your custom eslintrc file on your IDE as well


### An example of the above setup is located [here](https://github.com/rgabs/react-minimal-calendar)

> __Bonus suggestion__: Use [husky](https://www.npmjs.com/package/husky) for automatically running `npm run lint:fix` before you commit your code on your local machine 😉

# References
+ https://github.com/facebook/create-react-app/issues/808
+ https://eslint.org/docs/developer-guide/nodejs-api#cliengine
+ https://github.com/rgabs/react-minimal-calendar
+ https://medium.com/@adamdziendziel/custom-eslint-config-with-create-react-app-d6f66e8d61
+ https://gist.github.com/cncolder/19354ba59146c55071841f6fa274da66

That's it! I hope the blog was helpful. 🎉

__PS: Don't forget to press recommend icon below to show your support 🍻__

<div style="text-align:center">
  <img src="https://media.giphy.com/media/11yxY9ef0sAI3S/giphy.gif" style="width: 70%;display:inline-block;text-align:'left'" vspace="20">
</div>

