# JS/TS 桌面应用打包指南

本文档介绍 JavaScript/TypeScript 桌面应用的打包方案，重点介绍 Electron 的替代品。

---

## 目录

- [方案对比](#方案对比)
- [Tauri](#tauri) - 推荐，Rust 后端
- [Neutralinojs](#neutralinojs) - 轻量级
- [NW.js](#nwjs) - Electron 前身
- [Electron](#electron) - 传统方案
- [Wails](#wails) - Go 后端
- [Flutter](#flutter) - Dart 方案
- [选择建议](#选择建议)

---

## 方案对比

| 方案 | 语言 | 包体积 | 内存占用 | 性能 | 成熟度 | 推荐指数 |
|------|------|--------|----------|------|--------|----------|
| **Tauri** | Rust + JS/TS | ~3-10MB | 低 | 高 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Neutralinojs** | C++ + JS/TS | ~2-5MB | 低 | 中 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **NW.js** | JS/TS | ~100MB+ | 中 | 中 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Electron** | JS/TS | ~150MB+ | 高 | 中 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Wails** | Go + JS/TS | ~10-15MB | 低 | 高 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Flutter** | Dart | ~15-20MB | 中 | 高 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

## Tauri

### 简介

Tauri 是目前最推荐的 Electron 替代品：
- 使用系统 WebView（macOS 用 WebKit，Windows 用 WebView2，Linux 用 WebKitGTK）
- 后端用 Rust，性能极高
- 包体积极小（3-10MB）
- 安全性强，默认禁用不安全 API

### 安装

```bash
# 安装 Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 安装 Tauri CLI
pnpm add -g @tauri-apps/cli

# 或在项目中安装
pnpm add -D @tauri-apps/cli
```

### 创建项目

```bash
# 使用脚手架
pnpm create tauri-app

# 或在现有前端项目中添加
pnpm tauri init
```

### 项目结构

```
my-app/
├── src/              # 前端代码
├── src-tauri/        # Rust 后端
│   ├── src/
│   │   └── main.rs   # Rust 入口
│   ├── tauri.conf.json
│   └── Cargo.toml
├── package.json
└── vite.config.ts
```

### 配置文件

`src-tauri/tauri.conf.json`:

```json
{
  "build": {
    "beforeDevCommand": "pnpm dev",
    "beforeBuildCommand": "pnpm build",
    "devPath": "http://localhost:5173",
    "distDir": "../dist"
  },
  "package": {
    "productName": "My App",
    "version": "1.0.0"
  },
  "tauri": {
    "allowlist": {
      "all": false,
      "fs": { "all": true, "scope": ["$APP/*", "$DOCUMENT/*"] },
      "shell": { "open": true },
      "dialog": { "all": true }
    },
    "windows": [
      {
        "title": "My App",
        "width": 800,
        "height": 600,
        "resizable": true,
        "fullscreen": false
      }
    ],
    "security": {
      "csp": null
    },
    "bundle": {
      "active": true,
      "targets": "all",
      "identifier": "com.myapp.app",
      "icon": [
        "icons/32x32.png",
        "icons/128x128.png",
        "icons/icon.icns",
        "icons/icon.ico"
      ]
    }
  }
}
```

### 前端调用 Rust

`src-tauri/src/main.rs`:

```rust
#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]

#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![greet])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

前端调用：

```typescript
import { invoke } from '@tauri-apps/api/tauri'

const result = await invoke('greet', { name: 'World' })
console.log(result) // "Hello, World!"
```

### 开发与打包

```bash
# 开发模式
pnpm tauri dev

# 打包生产版本
pnpm tauri build

# 输出位置
# macOS: src-tauri/target/release/bundle/dmg/
# Windows: src-tauri/target/release/bundle/msi/
# Linux: src-tauri/target/release/bundle/deb/
```

### 常用 API

```typescript
// 文件系统
import { writeTextFile, readTextFile } from '@tauri-apps/api/fs'
await writeTextFile('test.txt', 'Hello World')
const content = await readTextFile('test.txt')

// 对话框
import { open, save } from '@tauri-apps/api/dialog'
const filePath = await open({ multiple: false })

// Shell
import { Command } from '@tauri-apps/api/shell'
const command = Command.sidecar('my-sidecar')
await command.execute()

// HTTP
import { fetch } from '@tauri-apps/api/http'
const response = await fetch('https://api.example.com', { method: 'GET' })

// 通知
import { sendNotification } from '@tauri-apps/api/notification'
await sendNotification({ title: 'Hello', body: 'World' })

// 剪贴板
import { writeText, readText } from '@tauri-apps/api/clipboard'
await writeText('Hello')
const text = await readText()

// 进程
import { exit, relaunch } from '@tauri-apps/api/process'
await exit(0)
await relaunch()
```

### 自动更新

```rust
// src-tauri/src/main.rs
use tauri::updater::UpdaterExt;

fn main() {
    tauri::Builder::default()
        .setup(|app| {
            let handle = app.handle();
            tauri::async_runtime::spawn(async move {
                match handle.updater().check().await {
                    Ok(update) => {
                        if update.is_update_available() {
                            update.download_and_install().await.unwrap();
                        }
                    }
                    Err(e) => eprintln!("Update check failed: {}", e),
                }
            });
            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### 优点

- 包体积极小（3-10MB vs Electron 150MB+）
- 内存占用低
- 启动速度快
- 安全性强
- 原生 API 丰富
- 活跃的社区

### 缺点

- 需要学习 Rust（但前端部分可以完全用 JS/TS）
- 生态不如 Electron 成熟
- 某些复杂功能需要自己用 Rust 实现

---

## Neutralinojs

### 简介

Neutralinojs 是一个轻量级跨平台框架：
- 使用系统 WebView
- 后端用 C++
- 包体积极小（2-5MB）
- 配置简单

### 安装

```bash
# 安装 CLI
pnpm add -g @neutralinojs/neu

# 创建项目
neu create my-app
cd my-app
```

### 项目结构

```
my-app/
├── resources/        # 前端代码
│   └── index.html
├── neutralino.config.json
└── package.json
```

### 配置文件

`neutralino.config.json`:

```json
{
  "applicationId": "com.myapp.app",
  "version": "1.0.0",
  "defaultMode": "window",
  "port": 0,
  "documentRoot": "/resources/",
  "url": "/",
  "enableServer": true,
  "enableNativeAPI": true,
  "tokenSecurity": "one-time",
  "logging": {
    "enabled": true,
    "writeToLogFile": true
  },
  "nativeAllowList": [
    "app.*",
    "os.*",
    "debug.*",
    "filesystem.*",
    "storage.*",
    "computer.*"
  ],
  "modes": {
    "window": {
      "title": "My App",
      "width": 800,
      "height": 600,
      "minWidth": 400,
      "minHeight": 300,
      "fullScreen": false,
      "alwaysOnTop": false,
      "icon": "/resources/icons/appIcon.png",
      "enableInspector": true,
      "borderless": false,
      "maximize": false,
      "hidden": false,
      "resizable": true,
      "exitProcessOnClose": true
    }
  },
  "cli": {
    "binaryName": "myapp",
    "resourcesPath": "/resources/",
    "extensionsPath": "/extensions/",
    "clientLibrary": "/resources/js/neutralino.js",
    "binaryVersion": "4.15.0",
    "clientVersion": "3.12.0"
  }
}
```

### 前端调用原生 API

```javascript
// 文件系统
await Neutralino.filesystem.writeFile('test.txt', 'Hello World')
const content = await Neutralino.filesystem.readFile('test.txt')

// 对话框
const filePath = await Neutralino.os.openFileDialog()
const savePath = await Neutralino.os.saveFileDialog()

// 剪贴板
await Neutralino.clipboard.writeText('Hello')
const text = await Neutralino.clipboard.readText()

// 通知
await Neutralino.computer.notify('Hello', 'World')

// 进程
await Neutralino.app.exit()
await Neutralino.app.restart()
```

### 开发与打包

```bash
# 开发模式
neu run

# 打包
neu build

# 输出位置
# dist/my-app/
```

### 优点

- 包体积极小
- 配置简单
- 学习曲线低
- 纯 JS/TS 开发

### 缺点

- 社区较小
- 原生 API 不如 Tauri 丰富
- 扩展性有限

---

## NW.js

### 简介

NW.js（原名 node-webkit）是 Electron 的前身：
- 内置 Node.js 和 Chromium
- 可以直接调用 Node.js API
- 成熟稳定

### 安装

```bash
# 下载 SDK 版本
# https://nwjs.io/downloads/

# 或使用 npm
pnpm add -D nw
```

### 项目结构

```
my-app/
├── package.json
├── index.html
└── main.js
```

### 配置文件

`package.json`:

```json
{
  "name": "my-app",
  "main": "index.html",
  "node-main": "main.js",
  "window": {
    "title": "My App",
    "width": 800,
    "height": 600,
    "resizable": true,
    "icon": "icon.png"
  }
}
```

### 前端调用 Node.js

```html
<!DOCTYPE html>
<html>
<head>
  <title>My App</title>
</head>
<body>
  <script>
    // 直接使用 Node.js API
    const fs = require('fs')
    fs.writeFileSync('test.txt', 'Hello World')
    
    const content = fs.readFileSync('test.txt', 'utf-8')
    console.log(content)
  </script>
</body>
</html>
```

### 打包

```bash
# 使用 nw-builder
pnpm add -D nw-builder

# 打包命令
npx nwbuild -p win64,osx64,linux64 -o dist .
```

### 优点

- 成熟稳定
- 直接使用 Node.js API
- 社区活跃

### 缺点

- 包体积大（100MB+）
- 内存占用较高
- 不如 Tauri 现代

---

## Electron

### 简介

Electron 是最成熟的桌面应用框架：
- 内置 Chromium 和 Node.js
- 生态最完善
- 大量成功案例（VS Code, Slack, Discord 等）

### 安装

```bash
pnpm create electron-app my-app
cd my-app
```

### 项目结构

```
my-app/
├── main.js          # 主进程
├── preload.js       # 预加载脚本
├── src/             # 渲染进程（前端）
├── package.json
└── electron-builder.yml
```

### 主进程

`main.js`:

```javascript
const { app, BrowserWindow, ipcMain } = require('electron')
const path = require('path')

function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      contextIsolation: true,
      nodeIntegration: false
    }
  })
  
  win.loadFile('src/index.html')
}

app.whenReady().then(() => {
  createWindow()
  
  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow()
    }
  })
})

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

// IPC 通信
ipcMain.handle('greet', async (event, name) => {
  return `Hello, ${name}!`
})
```

### 预加载脚本

`preload.js`:

```javascript
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld('electronAPI', {
  greet: (name) => ipcRenderer.invoke('greet', name),
  
  // 文件对话框
  openFile: () => ipcRenderer.invoke('dialog:openFile'),
  
  // 通知
  notify: (title, body) => ipcRenderer.send('notify', { title, body })
})
```

### 渲染进程

```javascript
// 前端调用
const result = await window.electronAPI.greet('World')
console.log(result) // "Hello, World!"
```

### 打包

`electron-builder.yml`:

```yaml
appId: com.myapp.app
productName: My App
directories:
  output: dist
files:
  - "**/*"
  - "!**/node_modules/*/{CHANGELOG.md,README.md,README,readme.md,readme}"
  - "!**/node_modules/*/{test,__tests__,tests,powered-test,example,examples}"
  - "!**/node_modules/*.d.ts"
  - "!**/node_modules/.bin"
  - "!**/*.{iml,o,hprof,orig,pyc,pyo,rbc,swp,csproj,sln,xproj}"
  - "!.editorconfig"
  - "!**/._*"
  - "!**/{.DS_Store,.git,.hg,.svn,CVS,RCS,SCCS,.gitignore,.gitattributes}"
  - "!**/{__pycache__,thumbs.db,.flowconfig,.idea,.vs,.nyc_output}"
  - "!**/{appveyor.yml,.travis.yml,circle.yml}"
  - "!**/{npm-debug.log,yarn.lock,.yarn-integrity,.yarn-metadata.json}"

mac:
  category: public.app-category.utilities
  target:
    - dmg
    - zip

win:
  target:
    - nsis
    - portable

linux:
  target:
    - AppImage
    - deb
```

打包命令：

```bash
# 安装 electron-builder
pnpm add -D electron-builder

# 打包
pnpm electron-builder

# 或在 package.json 中配置 scripts
# "build": "electron-builder"
pnpm build
```

### 优点

- 最成熟稳定
- 生态最完善
- 文档最全
- 大量成功案例

### 缺点

- 包体积大（150MB+）
- 内存占用高
- 启动速度较慢

---

## Wails

### 简介

Wails 是 Go 语言的桌面应用框架：
- 后端用 Go
- 前端用任意框架
- 包体积小（10-15MB）

### 安装

```bash
# 安装 Go
# https://go.dev/dl/

# 安装 Wails
go install github.com/wailsapp/wails/v2/cmd/wails@latest

# 创建项目
wails init -n myapp
cd myapp
```

### 项目结构

```
myapp/
├── main.go           # Go 入口
├── app.go            # 应用逻辑
├── frontend/         # 前端代码
│   ├── src/
│   └── package.json
└── wails.json
```

### Go 后端

`app.go`:

```go
package main

type App struct {
    ctx context.Context
}

func NewApp() *App {
    return &App{}
}

func (a *App) startup(ctx context.Context) {
    a.ctx = ctx
}

func (a *App) Greet(name string) string {
    return fmt.Sprintf("Hello, %s!", name)
}
```

`main.go`:

```go
package main

import (
    "embed"
    "github.com/wailsapp/wails/v2"
    "github.com/wailsapp/wails/v2/pkg/options"
)

//go:embed all:frontend/dist
var assets embed.FS

func main() {
    app := NewApp()
    
    err := wails.Run(&options.App{
        Title:  "My App",
        Width:  800,
        Height: 600,
        AssetServer: &options.AssetServer{
            Assets: assets,
        },
        OnStartup: app.startup,
        Bind: []interface{}{
            app,
        },
    })
    
    if err != nil {
        println("Error:", err.Error())
    }
}
```

### 前端调用

```typescript
import { Greet } from '../wailsjs/go/main/App'

const result = await Greet('World')
console.log(result) // "Hello, World!"
```

### 开发与打包

```bash
# 开发模式
wails dev

# 打包
wails build

# 输出位置
# build/bin/
```

### 优点

- 包体积小
- 性能高
- Go 语言简洁

### 缺点

- 需要学习 Go
- 社区较小

---

## Flutter

### 简介

Flutter 是 Google 的跨平台框架：
- 使用 Dart 语言
- 自渲染引擎
- 一套代码多平台

### 安装

```bash
# 安装 Flutter SDK
# https://docs.flutter.dev/get-started/install

# 创建项目
flutter create myapp
cd myapp
```

### 项目结构

```
myapp/
├── lib/
│   └── main.dart
├── windows/          # Windows 平台代码
├── macos/            # macOS 平台代码
├── linux/            # Linux 平台代码
└── pubspec.yaml
```

### 代码示例

`lib/main.dart`:

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'My App'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text('You have pushed the button this many times:'),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### 开发与打包

```bash
# 开发模式
flutter run -d windows  # Windows
flutter run -d macos    # macOS
flutter run -d linux    # Linux

# 打包
flutter build windows
flutter build macos
flutter build linux

# 输出位置
# build/windows/x64/runner/Release/
# build/macos/Build/Products/Release/
# build/linux/x64/release/bundle/
```

### 优点

- 一套代码多平台（移动端 + 桌面端 + Web）
- 性能高
- UI 组件丰富
- 热重载

### 缺点

- 需要学习 Dart
- 包体积比 Tauri 大
- 桌面端生态不如移动端成熟

---

## 选择建议

### 推荐 Tauri 的情况

- 追求最小包体积
- 追求最低内存占用
- 需要高性能
- 愿意学习一点 Rust
- **首选推荐！**

### 推荐 Neutralinojs 的情况

- 项目简单
- 不想学习新语言
- 追求最简单配置

### 推荐 Electron 的情况

- 团队熟悉 Node.js
- 需要最成熟的生态
- 不在意包体积

### 推荐 Wails 的情况

- 团队熟悉 Go
- 需要高性能
- 追求小包体积

### 推荐 Flutter 的情况

- 需要同时开发移动端和桌面端
- 团队愿意学习 Dart
- 需要丰富的 UI 组件

---

## 快速对比表

| 需求 | 推荐方案 |
|------|----------|
| 最小包体积 | Tauri / Neutralinojs |
| 最低内存 | Tauri |
| 最快开发 | Electron |
| 最高性能 | Tauri / Wails |
| 最成熟生态 | Electron |
| 移动端 + 桌面端 | Flutter |
| 不学新语言 | Electron / Neutralinojs |

---

## 总结

对于新项目，**强烈推荐 Tauri**：
- 包体积极小（3-10MB）
- 内存占用低
- 性能高
- 安全性强
- 社区活跃
- 文档完善

如果团队完全不想学习 Rust，**Neutralinojs** 是不错的选择。

如果需要最成熟的生态和最多的第三方库，**Electron** 仍然是可靠的选择。
