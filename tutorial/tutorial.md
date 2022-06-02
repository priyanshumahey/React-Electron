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

## Starting off (Setting Up Electron w/o React)
The first part of this tutorial is dedicated to explaining how you would set up Electron. This will only allow you to work with HTML/CSS/JS and not React. We need to start off by creating a folder and initializing an npm package.
``` PS
mkdir Method1
cd Method1
npm init
```
Within `package.json`, we need to make a couple of important changes:
- `entry point` should be `main.js`
- `author` and `description` are required but can be anything
So it can end up looking something like this:
``` JSON
{
  "name": "my-electron-app",
  "version": "1.0.0",
  "description": "Hello World!",
  "main": "main.js",
  "author": "Jane Doe",
  "license": "MIT"
}
```
Finally, in the `scripts` field of `package.json`, we add a `start` command like:
``` JSON
{
  "scripts": {
    "start": "electron ."
  }
}
```
The `start` command will let you open your app in development mode. For now this will not run anything.

&nbsp;


**Run the Main Process**

The `main` script is the entry point of the Electron application and it runs in the Node.js environment. Like it's name suggests, it controls the main process and is responsible for controlling the app's lifecycle, displaying native interfaces, performing privileged operations, and managing renderer processes. 

In the root folder, you'll need to create a few files:
- index.html
- main.js
- preload.js

index.html contains the content that will be loaded onto the window we create. This is the html that will be rendered.
```html
<--index.html-->
<!--index.html-->

<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.
  </body>
</html>
```

&nbsp;

**Opening your web page in a browser window**

We will load the web page into an application window using the following Electron modules:
- The `app` module controls the app's event lifecycle
- The `BrowserWindow` module is reponsible for creating and managing app windows
Since main.js is running in the Node environment, we can use Node modules such as require. 

In Node.js, `require()` is a native function that allows you to bring in external separate files and in this case, we will use it to bring in Electron.
```JS
const { app, BrowserWindow } = require('electron')
```
Then, we add a function called `createWindow` that loads `index.html` into a new `BrowserWindow` instance. 
``` JS
const createWindow = () => {
  const win = new BrowserWindow({
    width: 960,
    height: 720
  })

  win.loadFile('index.html')
}
```
Next, we'll need to actually open the function. Browser windows can only be created after the `app` module's `ready` event is fired. To wait for this, we see the `app.whenReady()` API which upon activating, we can use to start the window.
``` JavaScript
app.whenReady().then(() => {
  createWindow()
})
```

&nbsp;

**Manage your window's lifecycle**

&nbsp;

**Access Node.js from the renderer with a preload script**

&nbsp;

**Add functionality to your web contents**

&nbsp;

**Package and distribute your application**

&nbsp;

---

## Full Setup of Electron + React App