# create-react-app
`create-react-app` is a great tool for getting a react site up and running quickly. But the "black box" nature of the build environment means that it can often be difficult to customize the build for projects that fall outside the standard "cookie cutter" web app.

Surprisingly, as I was looking for solutions to the issues below, I quite often found people on Stack Overflow and other sites that said the only way to solve these issues was to use `eject`. Don't do this! Except as a very last resort.

---
## Contents
1. [Customize jslint](#jslint)
1. [Customize Webpack Config](#webpack)
1. [Exclude Files from Live Reload](#reload)
---

## Customizing jslint <a name="jslint"></a>
**TL;DR - Why I need this**

The default setting for jslint does not allow you to export an undefined object as the default. I do this quite often as a short hand, rather than the three step process of declaring a named constant, assigning it a value, then exporting as the default. 

For example, I may have some object like this (I do this often for static definitions of actions for redux):
```
export default {
    SET_PROJECT: 'SET_PROJECT',
    CLEAR_PROJECT: 'CLEAR_PROJECT',
    TOGGLE_USE_CODES: 'TOGGLE_USE_CODES',
    TOGGLE_USE_ROLES: 'TOGGLE_USE_ROLES'
}
```
The default setting for jslint flags this.

**The Solution:**

Inside `package.json` you'll find this property definition:
```
"eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  }
  ```
You will need to add an `overrides` property to this object. This is an array of jslint rules you want to add/change/remove, and the files this override should be applied to. 

For my example above, my configuration object looks like this:
```
"eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ],
    "overrides": [
      {
        "files": [
          "src/**/*.js"
        ],
        "rules": {
          "import/no-anonymous-default-export": "off"
        }
      }
    ]
  }
  ```

  ---

  ## Modify Webpack Config <a name="webpack"></a>

 ### Step 1:
  Install `react-app-rewired` as a dev dependency

  ```npm i -D react-app-rewired```

  or

  ```yarn add react-app-rewired --dev```

 ### Step 2:
 Add a file named `config-overrides.js` to the root directory of your project, and put the following code inside:
 ```
 module.exports = function override(config, env) {
    // Modify config here.
    return config;
}
```
The `config` variable is the webpack config used by `react-scripts`. Here you can modify it however you like before it's used. You can change any properties, add plugins or compilers, etc. The `env` variable contains the environment so you can change the config based on environment (dev/staging/prod/etc).

### Step 3:
In the scripts section of `package.json`, change `react-scripts` to `react-app-rewired` as in the example below:
```
"scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-scripts eject"
  }
```
Note: This change is not necessary for `eject`.

---

## Exclude Files From Live Reload <a name="reload"></a>
### The problem:
When working in development with create-react-app, the page continuously loads as you're making changes. This is usually a good thing. But there are times when this can cause problems, and even behavior that doesn't reflect how the final product will behave.

For example, I first encountered this as a problem when I included a feature to upload a profile image. There was no need to update the page, but the file saved to the images folder was considered a change. This caused unexpected behavior: such as refreshing the page before the new path was updated in redux - something that would not happen in production.

### The Solution:
Use the method described above to override the default webpack configuration. Put the following code in your `config-overrides.js` file:
```
module.exports = function override(config, env) {
    if (env === 'development') {
        const pluginIndex = config.plugins.findIndex(e => {
            return e.constructor.name === 'ReactRefreshPlugin';
        });
        if (pluginIndex > -1) {
            config.plugins[pluginIndex].options.exclude = [/node_modules/i, /public/i];
        }
    }
    return config;
}
```
### What this is doing:
1. First we check the value of `env` to see if we're in the development environment. This change is only necessary in development.
1. We need to make a change to one of the plugins, which are in an array. So we need to find the index of the plugin we need to modify: `ReactRefreshPlugin`
1. Lastly, we need to change the value of `options.exclude`. This is set to only exclude the `node_modules` folder. In this example we also tell it to exclude the `public` folder.
