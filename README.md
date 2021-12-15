# Master Electron - Course Code

Demo project modified from: https://github.com/electron/electron-quick-start

_Modified for improved screen real estate and for the sake of consistent versioning._

![Master Electron](https://raw.githubusercontent.com/stackacademytv/master-electron/master/splash.png)

## udemy notes

electron: use web technology to build cross-platform applications

electron works with 2 process Types:

1. Main Process: Node.js process (treated as Server-side)
   1. main.js is run as a node.js process
   2. it needs `nodemon` to watch if there is a change then reload the app
2. Renderer Process: Chromium our front end code with the web inspector (treated as Client-side)
   1. can integrate node.js into this process

```js
const { app, BrowserWindow } = require("electron");
// BrowserWindow create new Renderer process
```

basically, develop node.js app by using electron modules

electron-rebuild used to rebuild the installed app against the version of Node.js that your electron project is using.

```
npm i electron-rebuild -g
```

this ensures the module is built against the same version of Node.js as the version used by Electron

```js
mainWindow.webContents.openDevTools(); // 会打开web inspector tool
```

Debug

- debug in vs code: https://www.electronjs.org/docs/latest/tutorial/debugging-vscode
- **debug with chrome dev tools**:
  - 在 code 中加一些`debugger`的位置
  - `electron --inspect=5858 .` node will create a web socket connection to http port for debugging, 这个 port 目前是 5858
    - 如果没有 global 安装 electron, 可以加入到`package.json的script`中
  - app 会打开，然后 open 一个新的 chrome tab, 输入`chrome://inspect`
  - click `configure`, 并输入刚才的 port 的网址: `localhost:5858`
  - click `inspect` 会打开一个新的页面，
  - click `source`
  - 关闭 app 页面，会停在 debugger 位置

## Electron API

- Main Process + share modules, node.js
- Renderer Process + share modules, chromium

### [app](https://www.electronjs.org/docs/latest/api/app)

it is not created, but you juust need to refer

- `ready` event, check `app.isReady()`
- `before-quite` event, Emitted before the application starts closing its windows.
  ```js
  app.on("before-quit", (e) => {
    e.preventDefault(); // this will prevent app quitting
  });
  ```
- `open-file` max os specific
- `browser-window-blur` 知道用户何时离开 app
- `browser-window-focus` 知道用户何时 focus app
- `app.quit()`app 关闭
  ```js
  app.on("browser-window-blur", (e) => {
    console.log("Browser Window Blur");
    setTimeout(() => {
      app.quit();
    }, 3000);
  });
  ```
- `app.getPath("home"/"appData"/”document“/"downloads"....)`:获取一些资源的路径

### [BrowserWindow](https://www.electronjs.org/docs/latest/api/browser-window)

```js
// In the main process.
const { BrowserWindow } = require("electron");

const win = new BrowserWindow({ width: 800, height: 600 });

// Load a remote URL
win.loadURL("https://google.com");

// Or load a local HTML file
win.loadFile("index.html");

win.webContents.openDevTools(); // 打开web inspector
```

2 ways to Show windows

- Using the `ready-to-show` event, using once。 显示任然有白边，显示有延迟
  ```js
  const { BrowserWindow } = require("electron");
  const win = new BrowserWindow({ show: false });
  win.once("ready-to-show", () => {
    win.show();
  });
  ```
- Setting the backgroundColor property, 优先,看不出延迟，因为白边已经被上色了

  ```js
  const { BrowserWindow } = require("electron");

  const win = new BrowserWindow({
    backgroundColor: "#2e2c29",
    width: 600,
    height: 300,
    webPreferences: { nodeIntegration: true },
  });
  win.loadURL("https://github.com");
  ```

### [Parent & Child Windows](https://www.electronjs.org/docs/latest/api/browser-window#parent-and-child-windows)

```js
const { BrowserWindow } = require("electron");

const top = new BrowserWindow();
const child = new BrowserWindow({ parent: top });
child.show();
top.show();
```

- The child window will always show on top of the top window.
- The child window will close as parent window close
- `modal: true` in child window, will make it attached to main window
- Platform notices 根据平台的不同，window 会有一些不同: https://www.electronjs.org/docs/latest/api/browser-window#platform-notices

### [Frameless Window](https://www.electronjs.org/docs/latest/api/frameless-window)

没有 nav bar, menu 等，也没有关闭按钮的窗口，只有窗体内容,
因为没有横条，也无法拖拽

```js
const { BrowserWindow } = require('electron')
const win = new BrowserWindow({ width: 800, height: 600, frame: false }) // titleBarStyle: "hidden",会让title bar悬浮在窗体上
win.show()
// 在html中
<body style="user-select: none; -webkit-app-region:drag;padding-top:20px">
// 窗体不能选内容, 但是窗体自身内容有字的部分是可以用来拖拽移动窗体的
  <input type="range" style="-webkit-app-region:no-drag;"> // 使得内部的range滑块取消了拖拽窗体，可以滑动
</body>
```

### [BrowserWindow Properties, Methods & Events](https://www.electronjs.org/docs/latest/api/browser-window#new-browserwindowoptions)

Properties:

- `minWidth`, `minHeight`, `maxWidth`, `maxHeight`
- `resizable`, `moveable`

Events:

- `focus`, `blur`
- The global "app" instance receives all relevant BrowserWindow events, as they also apply to the application as a whole

Methods:

- [static methods](https://www.electronjs.org/docs/latest/api/browser-window#new-browserwindowoptions)

### [Browser State mgmt use electron-window-state](https://www.npmjs.com/package/electron-window-state)

将会在第二次打开 app 时记住 app 关闭时的位置和大小

### [Browser Window webContents](https://www.electronjs.org/docs/latest/api/web-contents)

webContents:是窗体的内容，也即 Renderer process 的部分

static methods:

```js
const { webContents } = require("electron");
console.log(webContents);
webContents.getAllWebContents();
```

events

- `did-finish-load`
- `new-window`, webContents.on("new-window", e => e.preventDefault())会阻止打开新的窗口
- `wc.on("login", (e, request, authInfo, callback) => { callback("user", "passwd") })`

https://httpbin.org/basic-auth/user/passwd

httpbin helps us test http features

- video control events
  ```js
  <video src=...></video>
  wc.on("media-started-playing", ...), "media-paused"
  ```
- `context-menu`

### [Electron Session](https://www.electronjs.org/docs/latest/api/session)

a session is an object used to store any state data related to web contents of browser window, like, Http cache, cookies, localStoreage

By default, electron create a session which is shared by different windows. the default session

```js
const { BrowserWindow } = require("electron");

const win = new BrowserWindow({ width: 800, height: 600 });
win.loadURL("http://github.com");

const ses = win.webContents.session;
console.log(ses.getUserAgent());
```

a session is used to store any sort of web content data

在 chrome devtool 中的 Application tab 里，有很多 Storage 如 LocalStorage, IndexedDB 等.
可以在这里手动加入一些 value

```js
const defaultSes = session.defaultSession; // use this as default session
```

we can define a custom session and tell the browser to use this session

```js
let customSes = session.fromPartition("part1");
const secWindow = new BrowserWindow({
  width: 800,
  height: 600,
  x: 200,
  y: 200,
  webPreferences: {
    nodeIntegration: true,
    session: customSes,
  },
});
```

现在这个 secWindow 用的 custom session 不同于 app 的 default session
如果没有指明，那么某个 window 还是会用 default session

default session is persist session,就会存在 hard drive 上

```js
let customSes = session.fromPartition("persist:part1"); //定义name有个prefix: persist:就会变成persist session把session的信息保存下来
// 下次打开app就会复用了
```

a short hand for creating and using the custom session

```js
const secWindow = new BrowserWindow({
  width: 800,
  height: 600,
  x: 200,
  y: 200,
  webPreferences: {
    nodeIntegration: true,
    partition: "persist:part1",
  },
});
```

delete data from session

```js
ses.clearStorageData();
ses.clearCache();
```

other

```js
ses
  .enableNetworkEmulation(options) // slow network down for dev
  .setPermissionRequestHandler()
  .getUserAgent()
  .clearAuthCache();
```

### [Cookies](https://www.electronjs.org/docs/latest/api/cookies)

web based storage, a part of session

```js
let ses = session.defaultSession;
ses.cookies.get({}); // get all cookies as Promise

const getCookies = () => {
  ses.cookies
    .get({})
    .then((cookies) => {
      console.log(cookies);
    })
    .catch((e) => console.log(e));
};

mainWindow.webContents.on("did-finish-load", (e) => {
  getCookies(); // after loaded then show the cookies
});
```

create own cookies

```js
let cookie = {
  name: "cookie",
};
ses.cookies.set(cookie).then(() => {
  console.log("cookie1 set");
  getCookies();
});
// our cookie is valid through this session. other session may not have this
// cookie will disappear after the app quit

// persist the cookie
let cookie = {
  name: "cookie",
  expirationDate: Date.now() + 100000,
};
```

read cookie

```js
ses.cookies.get({ name: "cookie1" }).then((cookies) => console.log(cookies));
```

remove a cookie

```js
ses.cookies.remove("https://myasdf.com", "cookie1").then(() => {
  getCookies();
});
```

### [Download Item](https://www.electronjs.org/docs/latest/api/download-item)

```js
const { session } = require("electron");
session.defaultSession.on("will-download", (event, downloadItem, webContents) => {
  downloadItem.setSave("/tmp/save.pdf");
  let filename = downloadItem.getFilename();
  let fileSize = downloadItem.getTotalBytes();
  downloadItem.setSavePath(app.getPath("desktop") + `/${filename}`);
  downloadItem.on("update", (e, state) => {
    let received = downloadItem.getReceivedBytes();
    if (state === "progressing" && received) {
      let progress = Math.round((received / fileSize) * 100); // 可以实时获取load的状态和进度
      webContents.executeJavaScript(`window.progress.value=${prgress}`); // 可以控制下方的window.progress定义的进度条的显示
    }
  });
});
```

download an image by html

```html
<h2><a href="splash.png" download>Download image</h2>
<progress id="progress" value="0" max="100"></progress>

<script>
  window.progress = document.getElementById("progress")// 将progress定义在window上，这样webContents可以接触
</script>
```

[file-examples.com](https://file-examples.com/) for testing download files

### [dialog](https://www.electronjs.org/docs/latest/api/dialog)

```js
const { dialog } = require("electron");
console.log(dialog.showOpenDialog({ properties: ["openFile", "multiSelections"] }));

mainWidnow.webContents.on("did-finish-load", () => {
  dialog
    .showOpenDialog(mainWindow, {
      // will be a child window of mainWindow
      buttonLabel: "select a photo",
      defaultPath: app.getPath("home"),
      properties: ["multiSelections", "createDirectory", "openDirectory"],
    })
    .then((result) => {
      console.log(result);
    });
  dialog.showOpenDialog({
    // ... will be a standard window
  });
  dialog
    .showSaveDialog({}) //展示一个询问保存的对话框
    .then((result) => {
      console.log(result);
    });

  dialog
    .showMessageBox({
      title: "Message Box",
      message: "Please select an option",
      detail: "Message details",
      buttons: ["Yes", "No", "Maybe"],
    })
    .then((res) => {
      console.log("use selected " + res.response);
    });
});
```

### [Accelerators](https://www.electronjs.org/docs/latest/api/accelerator) and [globalShortcut](https://www.electronjs.org/docs/latest/api/global-shortcut)

Accelerators: 描述快捷键组合的字符串

注册快捷键

```js
const { app, globalShortcut } = require("electron");

app.whenReady().then(() => {
  // Register a 'CommandOrControl+Y' shortcut listener.
  globalShortcut.register("CommandOrControl+Y", () => {
    // Do stuff when Y and either Command/Control is pressed.
  });
  globalShortcut.register("G", () => {
    console.log("use pressed G");
    globalShortcur.unregister("G");
  });
  // globalShortcut.isRegistered()
});
```

global 就是无需 focus window 也可以触发的

其他 app 有可能也注册了相同的快捷键，无法避免 clash

### [Menu](https://www.electronjs.org/docs/latest/api/menu) & [MenuItem](https://www.electronjs.org/docs/latest/api/menu-item)

[MenuItem roles](https://www.electronjs.org/docs/latest/api/menu-item#roles)

```js
const { Menu, MenuItem } = require("electron");

let mainMenu = new Menu();
let menuItem1 = new MenuItem({
  label: "Electron",
  submenu: [
    { label: "Item1" },
    { label: "Item2", submenu: [{ label: "Sub Item 1" }] },
    { label: "Item3" },
  ],
});
mainMenu.append(menuItem1);
// ...
Menu.setApplicationMenu(mainMenu);
```

create menu from template:

```js
let mainMenu = Menu.buildFromTemplate([
  {
    lable: "Electron",
    submenu: [
      { label: "Item1" },
      { label: "Item2", submenu: [{ label: "Sub Item 1" }] },
      { label: "Item3" },
    ],
  },
  {
    label: "Action",
    submenu: [
      { label: "DevTools", role: "toggleDevTools" },
      { label: "Action2" },
      {
        label: "Action3",
        click: () => console.log("Hello from Main main"),
        accelerator: "Shift+Alt+G", // menu item accelerator is not global, only be triggered when app focused
      },
    ],
  },
]);
```

refer to js file

```js
// define the menu template in a js file
let mainMenu = Menu.buildFromTemplate(require("./mainMenu"));
```

Custom Context Menus

https://www.electronjs.org/docs/latest/api/menu#menupopupoptions

```js
let contextMenu = Menu.buildFromTemplate([{ label: "Item 1" }, { role: "editMenu" }]);
mainWindow.webContents.on("context-menu", (e) => {
  contextMenu.popup();
});
```

### [Tray](https://www.electronjs.org/docs/latest/api/tray)

task bar, like, right top field of mac, or bottom right in windows

```js
const { app, Menu, Tray } = require("electron");

let tray = null;
app.whenReady().then(() => {
  tray = new Tray("/path/to/my/icon.png"); // mac 16x16 or 32x32 image
  const contextMenu = Menu.buildFromTemplate([
    { label: "Item1", type: "radio" },
    { label: "Item2", type: "radio" },
    { label: "Item3", type: "radio", checked: true },
    { label: "Item4", type: "radio" },
  ]);
  tray.setToolTip("This is my application.");
  tray.setContextMenu(contextMenu);
  tray.on("click", (e) => {
    if (e.shiftKey) {
      app.quit();
    } else {
      mainWindow.isVisible() ? mainWindow.hide() : mainWindow.show();
    }
  });
  tray.setContextMenu(someMenu);
});
```

https://www.electronjs.org/docs/latest/api/native-image

### [powerMonitor](https://www.electronjs.org/docs/latest/api/power-monitor)

Monitor power state changes, like suspend, resume

some platform specific events, like `on-ac` for windows

```js
const { app, powerMonitor } = require(electron);
app.on("ready", () => {
  powerMonitor.on("suspend", (e) => {
    // going to sleep
    console.log("Saving some data");
  });
  powerMonitor.on("resume", (e) => {
    if (!mainWindow) createWindow();
  });
});
```

### [screen](https://www.electronjs.org/docs/latest/api/screen)

when the app is ready, then screen can be used

```js
// Retrieve information about screen size, displays, cursor position, etc.
//
// For more info, see:
// https://electronjs.org/docs/api/screen

const { app, BrowserWindow } = require("electron");

let mainWindow = null;

app.whenReady().then(() => {
  // We cannot require the screen module until the app is ready.
  const { screen } = require("electron");

  // Create a window that fills the screen's available work area.
  const primaryDisplay = screen.getPrimaryDisplay();
  const displays = screen.getAllDisplays();
  console.log([displays[0].size.width, displays[1].size.height]);
  console.log([displays[0].bounds.x, displays[1].bounds.y]);
  // 可以得到不同分屏的大小和位置(主屏左上角点为0,0)
  // 可以从mac - system preferernces - displays - arrangement中理解

  const { width, height } = primaryDisplay.workAreaSize;

  mainWindow = new BrowserWindow({ width, height });
  mainWindow.loadURL("https://electronjs.org");

  screen.on("display-metric-changed", (e, display, metricsChanged) => {});
  screen.getCursorPosition();
});
```

use screen's bound

```js
new BrowserWindow({
  x: primaryDisplay.bounds.x,
  y: primaryDisplay.bounds.y,
  width: primaryDisplay.size.width / 2,
  height: primaryDisplay.size.height / 2,
});
```

## Renderer Process API

### [BrowserWindowProxy](https://www.electronjs.org/docs/latest/api/browser-window-proxy)

- [opening windows from the renderer](https://www.electronjs.org/docs/latest/api/window-open)

open a new browser window, simple: <a href="" target="_blank">but we don't have control on it. Or window.open(url)

```js
const win = window.open("https://www.google.com");
win.close();
win.eval("document.getElementByTagName('h1')[0].style.fontFamily = 'Comic Sand'");
```

window.open(url, frameName, features)

```js
win = window.open(url, "_blank", "width=500,height=450,alwaysOnTop=1"); // always stay on top of other window
```

### [Web Frame](https://www.electronjs.org/docs/latest/api/web-frame)

webframe represents the element of the electron app, the frame web browser loads contents

```js
const { webFrame } = require("electron");
webFrame.getZoomFactor(); // 1 means 100%
webFrame.getZoomLevel();
console.log(webFrame.getResourceUsage());
```

### [DesktopCapture](https://www.electronjs.org/docs/latest/api/desktop-capturer)

```js
const { desktopCapturer } = require("electron");
desktopCapturer.getSources({ types: ["screen"] }).then((sources) => {
  console.log(sources); // the first should be the primary screen
  document.getElementById("screenshot").src = sources[0].thumbnail.toDataURL();
  // <img id="screenshot" > will show the complete screen shot as an image
});
```

Why Node integration in Renderer process is disabled by default?

for security concern

Node integration essentially opens up access to the user's file system directly from within a Renderer process and is therefore disabled by default. This can be enabled by setting the "nodeIntegration" option to "true" on a BrowserWindow's "webPreferences", but must only ever be done when all content in such a window is 100% secure. i.e. loaded and created by the application developer.

Renderer process webFrame/web contents is basicaly a standard Chromium web browser

### ipc Inter-Process Communication

- [ipcRenderer](https://www.electronjs.org/docs/latest/api/ipc-renderer)
- [ipcMain](https://www.electronjs.org/docs/latest/api/ipc-main)

```js
// In main process.
const { ipcMain } = require("electron");
ipcMain.on("asynchronous-message", (event, arg) => {
  console.log(arg); // prints "ping"
  event.reply("asynchronous-reply", "pong");
});

ipcMain.on("synchronous-message", (event, arg) => {
  console.log(arg); // prints "ping"
  event.returnValue = "pong";
});
ipcMain.on("channel1", (e, args) => {
  console.log(args);
  e.sender.send("channel1-response", "Message received on channel1");
});
// the same as
mainWindow.webContents.on("did-finish-load", (e) => {
  mainWindow.webContents.send("mailbox", "You have new mail!");
});
```

```js
// In renderer process (web page).
// NB. Electron APIs are only accessible from preload, unless contextIsolation is disabled.
// See https://www.electronjs.org/docs/tutorial/process-model#preload-scripts for more details.
const { ipcRenderer } = require("electron");
console.log(ipcRenderer.sendSync("synchronous-message", "ping")); // prints "pong"

ipcRenderer.on("asynchronous-reply", (event, arg) => {
  console.log(arg); // prints "pong"
});
ipcRenderer.send("asynchronous-message", "ping");
doucment.querySelector("#talk").addEventListener("click", (e) => {
  ipcRenderer.send("channel1", "Hello from main window"); // this is async, so it exec and move down immediately
});
ipcRenderer.on("channel1-response", (e, args) => {
  console.log(args);
});
```

To send an asynchronous message back to the sender, you can use

- `event.reply(...)`. This helper method will automatically handle messages coming from frames that aren't the main frame (e.g. iframes) whereas
- `event.sender.send(...)` will always send to the main frame.

### Remote Module

except for IPC module to let communicate bewteen main and rederer process, remote module makes it easier

if security is a concern or performance penalty, then disable remote module

```js
const mainWindow = new BrowserWindow({
  width: 1000,
  height: 800,
  webPreferences: {
    nodeIntegration: true,
    enableRemoteModule: true,
  },
});
```

```js
const { remote } = require("electron");
const { dialog } = remote;
setTimeout(() => {
  dialog
    .showMessageBox({
      message: "Dialog from renderer",
      buttons: ["Close"],
    })
    .then((res) => {
      console.log(res);
    });
  let mainWindow = remote.getCurrentWindos(); // will get mainWindow
  mainWindow.maximize();
}, 2000);
```

### Invoke & Handle

- [ipcMain.handle](https://www.electronjs.org/docs/latest/api/ipc-main#ipcmainhandlechannel-listener)
- [ipcRenderer.invoke](electronjs.org/docs/latest/api/ipc-renderer#ipcrendererinvokechannel-args)

remote will be disabled by default in electron 9 or 10.

```js
const { ipcMain } = require("electron");
async function askFruit() {
  let fruits = ["apple", "orange", "grape"];
  let choice = await choicedialog.showMessageBox({
    message: "pick a fruit",
    buttons: fruits,
  });
  return fruits[choice.response];
}

function createWindow() {
  // ...
  setTimeout(() => {
    askFruit().then((answer) => {
      console.log(answer);
    });
  }, 3000);
  // ...
}

ipcMain.on("ask-fruit", (e) => {
  askFruit().then((answer) => {
    e.reply("answer-fruit", answer);
  });
});
```

```js
// renderer
const { ipcRenderer } = require("electron");
document.getElementById("ask").addEventListener("click", (e) => {
  ipcRenderer.send("ask-fruit");
});
ipcRenderer.on("answer-fruit", (e, answer) => {
  console.log(answer);
});
```

simply with invoke and handler

```js
// renderer
document.getElementById("ask").addEventListener("click", (e) => {
  ipcRenderer.invoke("ask-fruit").then((answer) => console.log(answer));
});
```

```js
// Main
ipcMain.handle("ask-fruit", (e) => {
  return askFruit(); // this returns a promise
});
```
## Shared API

### [Process](https://www.electronjs.org/docs/latest/api/process)

Electron's process object is extended from the Node.js process object. It adds some events, properties, and methods:

process is only available in renderer process if nodeIntegration is true


```js
process.version
process.type; // renderer for renderer process, "browser" main process
process.crash();
process.hang(); // 挂起，不会继续向前执行

mainWindow.webContents.on("crash", () => {
  setTimeout(() => {
    mainWindow.reload();
  }, 1000);
})
```

### [Shell](https://www.electronjs.org/docs/latest/api/shell)

`shell.openExternal(url, options)`

Open the given external protocol URL in the desktop's default manner. (For example, mailto: URLs in the user's default mail agent).

使用本地的app打开资源
```js
shell.openExternal("https://electronjs.org")//打开browser
shell.openPath(`${__dirname}/splash.png`) // 会用app默认的图片浏览工具来打开图片

shell.showItemInFolder(`${__dirname}/splash.png`) // 在file explorer 或 mac finder中显示文件位置
shell.moveItemToTrash(`${__dirname}/splash.png`) 
```

### [NativeImage](https://www.electronjs.org/docs/latest/api/native-image)

- [Data URL](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs)

native image used in Tray and screen shot

```js
const { nativeImage } = reqire("electron");
const splash = nativeImage.createFromPath(`${__dirname}/splash.png`);
console.log(splash.getSize());
const toPng = e => {
  let pngSplash = splash.toPNG();
  saveToDeskTop(pngSplash, "png")
}
const toJPG = e => {
  let jpgSpash = splash.toJPED(100);
  saveToDeskTop(jpgSplash, "jpg")
}
async function saveToDeskTop(data, ext){
  let desktopPath = ipcRenderer.invoke("app-path")
  fs.writeFile(`${desktopPath}/splash.${ext}`, data, console.log);
}
```
```js
<img src="" id="preview"/>

const toTag = (e) => {
  const size = splash.getSize();
  let pngSplashURL = splash.resize({ width: size.width / 4, height: size.height/4 }).toDataURL();
  document.getElementById("preview").src = splashURL;
}
```

### [Clipboard](https://www.electronjs.org/docs/latest/api/clipboard)

Perform copy and paste operations on the system clipboard.

```js
const { clipboard } = require("electron");

console.log(clipboard.readText());

document.querySelector("#makeUpper").addEventListener("click", e => {
  let cbText = clipboard.readText();
  clipboard.writeText(cbText.toUpperCase()); // 将把剪贴板内容变为大写
})


<button onClick="showImage()">Show clipboard image</button>
// after image copy, this will be a nativeImage object in clipboard
const showImage = (e) => {
  const image = clipboard.readImage();
  document.getElementById("cbImage").src = image.toDataURL();
}
```

## Features & Techniques

### [Offscreen Rendering](https://www.electronjs.org/docs/latest/tutorial/offscreen-rendering)

load and render content in browser window on a separate thread, since it is not visible, so uses less resources

prefer Software output device, to render in CPU, needs to disable GPU acceeleration

```js
const { app, BrowserWindow } = require('electron')
const fs = require('fs')
const path = require('path')

app.disableHardwareAcceleration()

let win

app.whenReady().then(() => {
  win = new BrowserWindow({ webPreferences: { offscreen: true }, show: false }) // offscreen mode and also do not show the window when load
  win.loadURL('https://github.com')
  // win.webContents.on("did-finish-load", (e) => {
  //   console.log(win.getTitle())
  //   win.close();
  //   win = null;
  // });
  win.webContents.on('paint', (event, dirty, image) => { // get off-screen rendered image
    const screenShot = image.toPNG();

    fs.writeFileSync(app.getPath("desktop") + 'ex.png', screenShot, console.log)// 会不停地pait并覆盖这个路径
  })
  win.webContents.setFrameRate(60)
  console.log(`The screenshot has been successfully saved to ${path.join(process.cwd(), 'ex.png')}`)
})

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow()
  }
})
```

### Network Detection
- [Online and offline events](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/Online_and_offline_events)
- [Navigator.onLine](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine)

```js

const setStatus = (status) => {
  const statusNode = document.getElementById("status")
  statusNode.innerText = status ? "OnLine" : "OffLine"
};
window.addEventListener("online", e => {
  setStatus(true)
})
window.addEventListener("offline", e => {
  setState(false);
});
```

### [Notification](https://www.electronjs.org/docs/latest/tutorial/notifications)
- [MDN web docs - Notification](https://developer.mozilla.org/en-US/docs/Web/API/notification)
- [push.js](https://pushjs.org/)
- [web push](https://github.com/web-push-libs/web-push)
- [Adding Push notification to a web App](https://developers.google.com/web/fundamentals/codelabs/push-notifications/)

```js
setTimeout(() => {
  let notification = new Notification("Electron App", {
    body: "Some notification info!"
  });
  notification.onclick = (e) => {
    console.log(e);
  }
}, 2000);

```
### [Preload Scripts](https://www.electronjs.org/docs/latest/tutorial/security)
当nodeIntegration需要被disabled的时候，preload.js可以被用来在loading前access main process
When "nodeIntegration" needs to be disabled for a given BrowserWindow, a Preload script can provide safe access to the Main process prior to loading that window's content. This allow us to perform tasks that imvolve the Main process or create references to Main process functionality.

A preload script runs in a BrowserWindow just before "nodeIntegration" gets disabled. This is immedaitely prior to loading the webContents of that BrowserWindow in order to have access to both the Node.js API and the Renderer process, but before loading potentially dangerous content.
```js
mainWindow = new BrowserWindow({
  width: 800, height: 600,
  webPreferences: {
    nodeIntegration: false,
    preload: `${__dirname}/preload.js`
  }
})
```
- preload.js  // even without nodeIntegeration, still get access to node.js
- 
```js
window.version = {
  node: process.versions.node,
  electron: process.versions.electron
};

```

### [Progress Bar](https://www.electronjs.org/docs/latest/tutorial/progress-bar)

App 的图标上的进度条
```js
let progress = 0.01
let progressInterval = setInterval(() => {
  mainWindow.setProgressBar(progress)
  if (progress <= 1) {
    progress += 0.01
  } else {
    mainWindow.setProgressBar(-1);// set to negative number will make the progress bar disappear
    clearInterval(progressInterval)
  }

}, 75)
```

## Project

### Overview & Setup

ref:
- [electron-window-state](https://www.npmjs.com/package/electron-window-state)
- [CSS Flex box guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)


