# RainWorld Tomato Clock

![logo](https://raw.githubusercontent.com/zpb911km/RW_Tomato_Clock/refs/heads/main/src-tauri/icons/128x128.png)

提供了番茄钟的(部分)功能和手感。

再辅佐以雨世界降雨计时器的压迫感UI。

在学习时发挥番茄钟功能，产生一种被大雨淋死的紧迫感。

这才是合格的“学者”。

!["学者"](https://raw.githubusercontent.com/zpb911km/RW_Tomato_Clock/refs/heads/main/src-tauri/icons/32x32.png)

鉴定为：玩雨世界玩的。



## 功能特点

- 使用Vue.js框架构建用户界面。
- 利用Tauri进行跨平台打包，支持Windows, macOS和Linux。
- 支持异步任务执行。
- 集成了图片资源导入功能。
- 包含自定义的`RainWorldClock`组件。

## 安装指南

1. 确保已安装Node.js和Vue CLI。
2. 确保已安装Rust和Cargo。
3. 安装Tauri CLI工具：`npm install -g @tauri-apps/cli`
4. 克隆此仓库：`git clone <repository-url>`
5. 进入项目目录：`cd <project-directory>`
6. 安装Vue.js依赖：`npm install`
7. 安装Rust依赖：`cargo install`

## 使用方法

1. 启动开发服务器：
    ```bash
    npm run tauri dev
    ```
2. 构建应用：
    ```bash
    npm run tauri build
    ```
3. 运行构建后的应用（在构建目录中找到相应平台的应用文件并运行）。

## 项目结构

- `src-tauri/Cargo.lock` - Rust依赖锁定文件。
- `src-tauri/src/lib.rs` - Tauri应用的主要入口文件。
- `pnpm-lock.yaml` - pnpm包管理器锁定文件。
- `package-lock.json` - npm包管理器锁定文件。
- `src/components/RainWorldClock.vue` - 自定义组件。
- `src/App.vue` - 应用主组件。

## 依赖

此项目依赖于多个Rust和JavaScript库。具体依赖可以在`Cargo.lock`和`package-lock.json`文件中找到。

## 贡献

欢迎对本项目进行贡献。你可以通过以下方式参与：

- 提交bug报告。
- 提出功能建议。
- 代码审查。
- 提交代码补丁。

