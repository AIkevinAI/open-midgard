# AGENTS.md — OpenMidgard 项目导航

> 本文件为 AI 代理提供项目快速上下文。人类开发者请参阅 `README.md`。

## 项目性质

OpenMidgard 是 **Ragnarok Online（仙境传说）客户端的开源重制版**，用现代 C++17 + CMake 重建，刻意锚定 **2008 年 pre-Renewal** 时期的客户端体验、画面风格与封包行为。

- **封包目标**：`packet_ver 23` / `2008-09-10aSakexe`
- **语言**：C++17
- **构建系统**：CMake 3.20+
- **当前状态**：活跃开发中，无自动化测试套件

## 技术栈

- **核心依赖**：zlib v1.3.1（FetchContent 自动拉取）
- **渲染后端**（多后端可选）：
  - DirectX 7（原始客户端原生后端）
  - Direct3D 11 / Direct3D 12（仅 Windows MSVC，选项 `RO_ENABLE_NATIVE_D3D11` / `RO_ENABLE_NATIVE_D3D12`）
  - Vulkan（Linux/macOS 主路径，Windows 需 `VULKAN_SDK`）
- **UI 后端**：
  - 经典原生 UI（默认，`src/ui/`）
  - Qt 6 / QML（可选，选项 `RO_ENABLE_QT6_UI`；非 Windows 平台强制开启）
- **遗留商业库**（运行时可选加载）：Miles Sound System、Bink Video、Granny 3D、Intel JPEG Library
- **Windows 系统库**：winmm、dxguid、ole32、oleaut32、psapi、msimg32

## 模块架构

源码位于 `src/`，按 `CMakeLists.txt` 中 `add_subdirectory` 顺序组装为静态库，最终链接为 `open-midgard` 可执行文件。

| 模块（库） | 目录 | 职责 |
|---|---|---|
| `ro_core` | `src/core/` | Timer、Hash、SettingsIni、GPak/Grf 文件、File、Globals、Xml、DllMgr、Locale |
| `ro_cipher` | `src/cipher/` | 网络封包加解密 |
| `ro_network` | `src/network/` | Connection、GronPacket、MapSendProfile、PacketQueue |
| `ro_res` | `src/res/` | Bitmap、GndRes、ImfRes、ModelRes、PaletteRes、Sprite、ActRes、WorldRes、EzEffectRes、Ijl |
| `ro_audio` | `src/audio/` | Miles Sound System 运行时加载 |
| `ro_input` | `src/input/` | 输入处理 |
| `ro_session` | `src/session/` | Session、SkillActionInfo |
| `ro_gamemode` | `src/gamemode/` | LoginMode、GameMode、View、CursorRenderer、GameModePacket |
| `ro_world` | `src/world/` | World、GameActor、3dActor、RagEffect、MsgEffect、Granny |
| `ro_render` | `src/render/` | Renderer、DrawUtil、DC、Prim（2D 原语） |
| `ro_render3d` | `src/render3d/` | RenderBackend、RenderDevice、GraphicsSettings、Device、D3dutil |
| `ro_ui` | `src/ui/` | 经典 UI 窗口系统（40+ 窗口类）+ UIWindowMgr |
| `ro_qtui` | `src/qtui/` | Qt6/QML 现代 UI 运行时：QtUiRuntime、QtPlatformWindow、QtUiSpikeBridge |
| `ro_skill` | `src/skill/` | 技能系统 |
| `ro_item` | `src/item/` | 物品系统 |
| `ro_pathfinder` | `src/pathfinder/` | 寻路算法 |
| `ro_lua` | `src/lua/` | Lua 脚本桥接（LuaBridge） |
| `ro_security` | `src/security/` | 安全相关 |
| `ro_main` | `src/main/` | 应用入口 |

## 入口点

- **Windows**：`src/main/WinMain.cpp`（标准 `WinMain`，对应反编译原始偏移 `0x68bcd0`）
- **非 Windows**：`src/main/AppMain_stub.cpp`（stub 入口）
- **Windows 资源**：`src/main/OpenMidgard.rc`（图标 ID = 101）
- 最终目标：`add_executable(open_midgard ...)`，输出名 `open-midgard`，Windows 下为 `WIN32` 子系统 GUI 程序

## 构建命令（CMake Presets）

| 平台 | 配置 | 命令 |
|---|---|---|
| Windows VS2022 Win32 | Release | `cmake --preset vs2022-win32` → `cmake --build --preset build-release --config Release` |
| Windows VS2022 x64 | Release | `cmake --preset vs2022-x64` → `cmake --build --preset build-release-x64 --config Release` |
| Windows MinGW+Qt x64 | 默认 | `cmake --preset mingw-qt-x64` → `cmake --build --preset build-mingw-qt-x64` |
| Windows MinGW+Qt x64 | Debug | `cmake --preset mingw-qt-x64-debug` → `cmake --build --preset build-mingw-qt-x64-debug` |
| Linux Qt+Vulkan | Release | `cmake --preset linux-qt-vulkan` → `cmake --build --preset build-linux-qt-vulkan -j` |

## 运行

构建产物：
- Windows：`open-midgard.exe`
- Linux/macOS：`open-midgard`

运行时需在同目录放置 `data.grf`、`data/clientinfo.xml`、`data/` 等 RO 资源，或设置环境变量：

```
OPEN_MIDGARD_DATA_DIR=/path/to/runtime
```

## 关键 CMake 选项

| 选项 | 作用 |
|---|---|
| `RO_ENABLE_QT6_UI` / `RO_QT_ROOT` / `RO_QT_WINDEPLOYQT_EXECUTABLE` | Qt6 UI 路径 |
| `RO_ENABLE_NATIVE_D3D11` / `RO_ENABLE_NATIVE_D3D12` | Windows 原生 D3D 后端 |
| `RO_ENABLE_MILES_AUDIO` / `RO_ENABLE_BINK_VIDEO` / `RO_ENABLE_GRANNY` / `RO_ENABLE_IJL` | 遗留商业库运行时加载 |
| `RO_ENABLE_DEV_DEPLOY` | 构建后自动拷贝到 `D:/Spel/OldRO` |
| `RO_LOG_VULKAN_STATS` | Vulkan 性能统计日志 |

> 注：x64 首个里程碑构建会强制关闭 Miles/Bink/Granny/IJL（见根 `CMakeLists.txt` 第 54-62 行）。

## 关键配置文件

| 文件 | 作用 |
|---|---|
| `CMakeLists.txt` | 根构建配置：所有 `RO_ENABLE_*` 选项、平台检测、子模块组装、最终可执行目标链接 |
| `CMakePresets.json` | 6 个配置预设 + 8 个构建预设 |
| `cmake/QtSetup.cmake` | Qt6 查找与配置封装 |
| `cmake/DeployQtRuntime.cmake` | Qt 运行时部署（windeployqt） |
| `cmake/BestEffortCopy.cmake` | 构建后拷贝 exe 到本地开发部署目录 |
| `.github/workflows/build-release-clients.yml` | CI 发布构建工作流 |
| `.github/instructions/*.md` | AI 协作规则（ai_rules、implementation_rules、packet_alignment、reference_lookup） |
| `DllMgr.i` | DLL 管理接口定义（SWIG 风格 .i 文件） |

## 目录结构

```
I:\open-midgard\
├── src/                     # 客户端源码（19 个子模块）
├── cmake/                   # CMake 辅助脚本
├── docs/                    # 项目文档（qt 构建、qml 迁移）
├── third_party/qt/          # 第三方 Qt 说明
├── .github/                 # CI 工作流 + AI 规则说明
├── CMakeLists.txt           # 根构建配置
├── CMakePresets.json        # 构建预设
└── README.md                # 人类开发者文档
```

## 平台支持

- **Windows**（VS2022 与 MinGW+Qt）：主开发平台，最完整构建流
- **Linux**：Qt + Vulkan 路径
- **macOS**：支持较初步，需手动配置 CMake

## 测试

项目目前**无自动化测试**（未集成 gtest/Catch2/doctest 等）。验证依赖手动运行客户端。