# Tutorial
## Introduction
This tutorial aims to go over both Electron and React.js in an attempt to help you set up a desktop application built on electron.js with the frontend being done using React.

This tutorial assumes you already have node and npm installed and running. To ensure both of them are running, run the following commands
``` PS
node -v
npm -v
```
These commands run versions and will check if they are present on the computer. The version of Node.js doesn't amtter since Electron embeds Node.js into its binary meaning that the version running on your computer doesn't matter.

--- 
## Electron



**What is Electron?**

Electron.js is a backend builder which enables you to build applications. Many common desktop apps such as Visual Studio Code, Slack, Messenger are build based off of Electron.

![Apps](./images/apps.png)

Electron relies on Chromium and Node.js and creates cross-platform applications with no native development required. Let's go over some important things to better understand React.

&nbsp;

**Processes In Electron**

Electron has a multi-process architecture (inherited from Chromium) which means that it's very similar to modern web browsers. Having a single process could mean that one website crashing would lead to the crash of an entire browser but with multiple processes, you can have multiple different things going on.

In Chrome, each tab renders its own processs and a signle browser controls these processes. The Chrome process manager manages multiple of processes. 

![Apps](./images/chrome-processes.png)

App developers have control of the renderer and main processes. 

&nbsp;

**The Main Process**

The Main process is responsible for creating instances of `BrowserWindow` and different events we observe in Electron. It registers global shortcuts, creates native menus and dialogs and responds to auto-update events, etc.

The main purpose of this is to create and manage application windows with the `BrowserWindow` module. For ever instance of the `BrowserModule` class, it creates an application window that loads a web page in a spearate renderer process. 
``` JS
//main.js
const { BrowserWindow } = require('electron')

const win = new BrowserWindow({ width: 800, height: 1500 })
win.loadURL('https://github.com')

const contents = win.webContents
console.log(contents)
```
When a `BrowserWindow` isntance is destroyed, it's corresponding renderer process gets terminated as well.

The main process also controls your application's lifestyle through Electron's `app` module. This allows you to add custom behaviors. The main process also has custom APIs that allow you to interact with the OS. More information can be found [here](https://www.electronjs.org/docs/latest/api/app).


&nbsp;

**The Renderer Process**

For each open `BrowserWindow`, there is a separate renderer process which renders the web content and the code running in this will run similar to how it would run on Chrome.

Renderer processes have no direct access to Node.js APIs (including ones like `require`) and you will need same bundler toolchains (ex `webpack` or `parcel`) that you use on the web.

&nbsp;

**Preload Scripts**

These are the scripts that run in a renderer process before the web content begins to load. They run in the context of the renderer but have more privileges by having access to Node.js APIs. A preload script can be attached to the main process in the `BrowserWindow` constructor's `webPreferences` option.

```JS
//main.js
const { BrowserWindow } = require('electron')
//...
const win = new BrowserWindow({
  webPreferences: {
    preload: 'path/to/preload.js'
  }
})
//...
```
The preload script shares a global `Window` interface with renderers and can access Node.js APIs, enabling enhancement of the renderer by exposing arbitary APIs in the `window` global for the web contents to use. 

Preload scripts share a `window` global with the renderer they're attached to but do not directly attach any variables from preload script to `window` due to the `contextIsolation` default.
```JS
//preload.js
window.myAPI = {
  desktop: true
}
```
``` JS
//render.js
console.log(window.myAPI)
// => undefined
```
Context Isolation means preload scripts are isolated from renderer's main function to prevent leaking APIs to the web content of the code. `contextBridge` allows for APIs to be used securely.
``` JS
//preload.js
const { contextBridge } = require('electron')

contextBridge.exposeInMainWorld('myAPI', {
  desktop: true
})
```
``` JS
//renderer.js
console.log(window.myAPI)
// => { desktop: true }
```
This is helpful for 2 main reasons:
1. It exposes `ipcRenderer` helpers to renderer which allows use of inter-process communication to trigger main process tasks from the renderer (and vice versa) 
2. It allows you to add custom properties to the renderer's `window` global that can be used for desktop-only logic on the web client's side which can be really helpful when developer Electron wrappers for existing web apps.

&nbsp;


**Helpful links**
- **[Electron Website](https://www.electronjs.org/)**

- **[Electron Documentation](https://www.electronjs.org/docs/latest)**

- **[Electron Github](https://github.com/electron/electron)**

--- 

## React
**What is React?**
React is a front-end library for building user interfaces. It's open-source and is maintained by Meta. This is not a React tutorial so this will not delve too deep into attempting to explain React.


**Helpful links**
- **[React Website](https://reactjs.org/)**
- **[React Documentation](https://reactjs.org/docs/getting-started.html)**
--- 

