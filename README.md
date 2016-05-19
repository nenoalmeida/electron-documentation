# Application Distribution

To distribute your app with Electron, the folder containing your app should be named app and placed under Electron’s resources directory (on OS X it is Electron.app/Contents/Resources/ and on Linux and Windows it is resources/), like this:

On OS X:

```sh
electron/Electron.app/Contents/Resources/app/
├── package.json
├── main.js
└── index.html
```

On Windows and Linux:

```sh
electron/resources/app
├── package.json
├── main.js
└── index.html
```

Then execute Electron.app (or electron on Linux, electron.exe on Windows), and Electron will start as your app. The electron directory will then be your distribution to deliver to final users.

* Packaging Your App into a File

Apart from shipping your app by copying all of its source files, you can also package your app into an asar archive to avoid exposing your app’s source code to users.

To use an asar archive to replace the app folder, you need to rename the archive to app.asar, and put it under Electron’s resources directory like below, and Electron will then try to read the archive and start from it.

On OS X:

```sh
electron/Electron.app/Contents/Resources/
└── app.asar
```

On Windows and Linux:

```sh
electron/resources/
└── app.asar
```

More details can be found in Application packaging.

Rebranding with Downloaded Binaries

After bundling your app into Electron, you will want to rebrand Electron before distributing it to users.

Windows
You can rename electron.exe to any name you like, and edit its icon and other information with tools like rcedit.

OS X
You can rename Electron.app to any name you want, and you also have to rename the CFBundleDisplayName, CFBundleIdentifier and CFBundleName fields in following files:

Electron.app/Contents/Info.plist
Electron.app/Contents/Frameworks/Electron Helper.app/Contents/Info.plist
You can also rename the helper app to avoid showing Electron Helper in the Activity Monitor, but make sure you have renamed the helper app’s executable file’s name.

The structure of a renamed app would be like:

```sh
MyApp.app/Contents
├── Info.plist
├── MacOS/
│   └── MyApp
└── Frameworks/
    ├── MyApp Helper EH.app
    |   ├── Info.plist
    |   └── MacOS/
    |       └── MyApp Helper EH
    ├── MyApp Helper NP.app
    |   ├── Info.plist
    |   └── MacOS/
    |       └── MyApp Helper NP
    └── MyApp Helper.app
        ├── Info.plist
        └── MacOS/
            └── MyApp Helper
```
  
Linux
You can rename the electron executable to any name you like.

Rebranding by Rebuilding Electron from Source

It is also possible to rebrand Electron by changing the product name and building it from source. To do this you need to modify the atom.gyp file and have a clean rebuild.

grunt-build-atom-shell
Manually checking out Electron’s code and rebuilding could be complicated, so a Grunt task has been created that will handle this automatically: grunt-build-atom-shell.

This task will automatically handle editing the .gyp file, building from source, then rebuilding your app’s native Node modules to match the new executable name.

Quick Start

Electron enables you to create desktop applications with pure JavaScript by providing a runtime with rich native (operating system) APIs. You could see it as a variant of the Node.js runtime that is focused on desktop applications instead of web servers.

This doesn’t mean Electron is a JavaScript binding to graphical user interface (GUI) libraries. Instead, Electron uses web pages as its GUI, so you could also see it as a minimal Chromium browser, controlled by JavaScript.

Main Process
In Electron, the process that runs package.json’s main script is called the main process. The script that runs in the main process can display a GUI by creating web pages.

Renderer Process
Since Electron uses Chromium for displaying web pages, Chromium’s multi-process architecture is also used. Each web page in Electron runs in its own process, which is called the renderer process.

In normal browsers, web pages usually run in a sandboxed environment and are not allowed access to native resources. Electron users, however, have the power to use Node.js APIs in web pages allowing lower level operating system interactions.

Differences Between Main Process and Renderer Process
The main process creates web pages by creating BrowserWindow instances. Each BrowserWindow instance runs the web page in its own renderer process. When a BrowserWindow instance is destroyed, the corresponding renderer process is also terminated.

The main process manages all web pages and their corresponding renderer processes. Each renderer process is isolated and only cares about the web page running in it.

In web pages, calling native GUI related APIs is not allowed because managing native GUI resources in web pages is very dangerous and it is easy to leak resources. If you want to perform GUI operations in a web page, the renderer process of the web page must communicate with the main process to request that the main process perform those operations.

In Electron, we have several ways to communicate between the main process and renderer processes. Like ipcRenderer and ipcMain modules for sending messages, and the remote module for RPC style communication. There is also an FAQ entry on how to share data between web pages.

Write your First Electron App

Generally, an Electron app is structured like this:

your-app/
├── package.json
├── main.js
└── index.html
The format of package.json is exactly the same as that of Node’s modules, and the script specified by the main field is the startup script of your app, which will run the main process. An example of your package.json might look like this:

{
  "name"    : "your-app",
  "version" : "0.1.0",
  "main"    : "main.js"
}
Note: If the main field is not present in package.json, Electron will attempt to load an index.js.

The main.js should create windows and handle system events, a typical example being:

const electron = require('electron');
// Module to control application life.
const {app} = electron;
// Module to create native browser window.
const {BrowserWindow} = electron;

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let win;

function createWindow() {
  // Create the browser window.
  win = new BrowserWindow({width: 800, height: 600});

  // and load the index.html of the app.
  win.loadURL(`file://${__dirname}/index.html`);

  // Open the DevTools.
  win.webContents.openDevTools();

  // Emitted when the window is closed.
  win.on('closed', () => {
    // Dereference the window object, usually you would store windows
    // in an array if your app supports multi windows, this is the time
    // when you should delete the corresponding element.
    win = null;
  });
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.on('ready', createWindow);

// Quit when all windows are closed.
app.on('window-all-closed', () => {
  // On OS X it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', () => {
  // On OS X it's common to re-create a window in the app when the
  // dock icon is clicked and there are no other windows open.
  if (win === null) {
    createWindow();
  }
});

// In this file you can include the rest of your app's specific main process
// code. You can also put them in separate files and require them here.
Finally the index.html is the web page you want to show:

<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using node <script>document.write(process.versions.node)</script>,
    Chrome <script>document.write(process.versions.chrome)</script>,
    and Electron <script>document.write(process.versions.electron)</script>.
  </body>
</html>
Run your app

Once you’ve created your initial main.js, index.html, and package.json files, you’ll probably want to try running your app locally to test it and make sure it’s working as expected.

electron-prebuilt
electron-prebuilt is an npm module that contains pre-compiled versions of Electron.

If you’ve installed it globally with npm, then you will only need to run the following in your app’s source directory:

electron .
If you’ve installed it locally, then run:

./node_modules/.bin/electron .
Manually Downloaded Electron Binary
If you downloaded Electron manually, you can also use the included binary to execute your app directly.

Windows
$ .\electron\electron.exe your-app\
Linux
$ ./electron/electron your-app/
OS X
$ ./Electron.app/Contents/MacOS/Electron your-app/
Electron.app here is part of the Electron’s release package, you can download it from here.

Run as a distribution
After you’re done writing your app, you can create a distribution by following the Application Distribution guide and then executing the packaged app.

Try this Example
Clone and run the code in this tutorial by using the atom/electron-quick-start repository.

Note: Running this requires Git and Node.js (which includes npm) on your system.

 Clone the repository
> git clone https://github.com/electron/electron-quick-start
 Go into the repository
> cd electron-quick-start
Install dependencies and run the app
> npm install && npm start

