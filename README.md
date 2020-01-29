# i18next-electron-fs-backend
This is an i18next library designed to work with [secure-electron-template](https://github.com/reZach/secure-electron-template). The library is a rough copy of [i18next-node-fs-backend](https://github.com/i18next/i18next-node-fs-backend) but using IPC (inter-process-communication) to request a file be read or written from the electron's [main process](https://electronjs.org/docs/api/ipc-main).

## How to install

### Install the package
`npm i i18next-electron-fs-backend`

### Add into your i18next config
Based on documentation for a [i18next config](https://www.i18next.com/how-to/add-or-load-translations#load-using-a-backend-plugin), import the backend.
```javascript
import i18n from "i18next";
import {
  initReactI18next
} from "react-i18next";
import backend from "i18next-electron-fs-backend";

i18n
  .use(backend)
  .use(initReactI18next)
  .init({
    backend: {
      loadPath: "./app/localization/locales/{{lng}}/{{ns}}.json",
      addPath: "./app/localization/locales/{{lng}}/{{ns}}.missing.json",
      ipcRenderer: window.api.i18nextElectronBackend // important!
    },

    // other options you might configure
    debug: true,
    saveMissing: true,
    saveMissingTo: "current",
    lng: "en"
  });

export default i18n;
```

### Update your preload.js script
```javascript
const {
    contextBridge,
    ipcRenderer
} = require("electron");
const backend = require("i18next-electron-fs-backend");

contextBridge.exposeInMainWorld(
    "api", {
        i18nextElectronBackend: backend.preloadBindings(ipcRenderer)
    }
);
```

### Update your main.js script
```javascript
const {
  app,
  BrowserWindow,
  session,
  ipcMain
} = require("electron");
const backend = require("i18next-electron-fs-backend");
const fs = require("fs");

let win;

async function createWindow() {  
  win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      contextIsolation: true,
      preload: path.join(__dirname, "preload.js")
    }
  });

  backend.mainBindings(ipcMain, win, fs); // <- configures the backend
  
  // ...
}

app.on("ready", createWindow);

app.on("window-all-closed", () => {
  // On macOS it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform !== "darwin") {
    app.quit();
  } else {
    i18nextBackend.clearMainBindings(ipcMain);
  }
});
```

## Options
These are options that are configurable, all values below are defaults.
```javascript
{
    loadPath: "/locales/{{lng}}/{{ns}}.json", // Where the translation files get loaded from
    addPath: "/locales/{{lng}}/{{ns}}.missing.json", // Where the missing translation files get generated
    delay: 300 // Delay before translations are written to file
}
```

## Common problems

### Refused to evaluate a string as Javascript because 'unsafe-eval' is not an allowed source of script...
You may see the following message in the console:

```Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src 'self' 'nonce-GSNC5bhidVKJ+y2K0ENMOA=='".```

This error happens because as part of the backend, missing translation values are written to disk. If you have a strong CSP for your electron app, this error may appear when the i18next backend writes to the missing translation file.

The missing translation keys/files _do_ get written, so you can ignore this error. I am open to a PR to get this error fixed if it really bugs you.