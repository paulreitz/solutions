# create-react-app
`create-react-app` is a great tool for getting a react site up and running quickly. But the "black box" nature of the build environment means that it can often be difficult to customize the build for projects that fall outside the standard "cookie cutter" web app.

Surprisingly, as I was looking for solutions to the issues below, I quite often found people on Stack Overflow and other sites that said the only way to solve these issues was to use `eject`. Don't do this! Except as a very last resort.

---
## Contents
1. [Customize jslint](#jslint)
1. [Customize Webpack Config](#webpack)
---

## Customizing jslint <a name="jslint"></a>
**TL;DR - Why I need this**

The default setting for jslint does not allow you to export an undefined object as the default. I do this quite often as a short hand, rather than the three step process of declaring a named constant, assigning it a value, then exporting as the default. 

For example, I may have some object like this (I do this often for static definitions of actions for redux):
```
export default {
    SET_ACCOUNT: 'SET_ACCOUNT',
    UNSET_ACCOUNT: 'UNSET_ACCOUNT'
};
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
