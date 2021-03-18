## React and Electron
Although this is specifically for React and electron, this could also apply to similar situations using other front end libraries.

---

### Importing ipcRenderer in React
When I encountered this problem, it was specifically ipcRenderer I was trying to import. However, this could apply to any number of other libraries.

**Preload the Library:**

Create a file called `preload.js` in the same folder as your main electron file. (The name and location is not important, but the example code used later assumes a file with this name exists at this location). Add the following code to the file:
```
const { ipcRenderer } = require('electron');
window.ipcRenderer = ipcRenderer;
```
You can load any other libraries that may not work loading into a react app here as well. Assign the imported librarys to `window.someUniqueName`.

**Add the Preloaded Library:**

When initializing a new BrowserWindow, add a `webPreferences` property to the config as in the following example:
```
mainWindow = new BrowserWindow({ 
        width: 900, 
        height: 680, 
        webPreferences: {
            nodeIntegration: true,
            contextIsolation: false,
            preload: path.join(__dirname, 'preload.js')
        }
    });
```
Many sites and forums mention `nodeIntegration` and can't agree on whether or not it's necessary. However, few mention `contextIsolation`, and this is the important part. You must disable this or the preloaded libraries will not be available to the renderer.

**Configure React to Work With Electron:**

Add this `target` property to your webpack config:
```target: "electron-renderer"```

If you're using create-react-app, you can read the instructions on how to [modify webpack config](create-react-app#webpack) to see how to add this property.