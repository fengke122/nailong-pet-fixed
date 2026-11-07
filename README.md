# 奶龙桌宠 - 修复版（Release 构建可用包）

基于 [mmexile/nailong-pet](https://github.com/mmexile/nailong-pet)（MIT License）源码修改，原作者 mmexile（NKU-mexile）。

## 为什么官方包跑不起来

官方仓库根目录的 `nailong-pet-release.zip` 是 **Debug 构建打包**的：

- `nailong-pet.exe` 的导入表依赖 `ucrtbased.dll`（MSVC 调试版 C 运行时），该 DLL **不可再分发**，只随 Visual Studio 安装，干净的 Windows 上必定报"找不到 ucrtbased.dll"
- 详见上游 issue #1、#2

本包用 **VS Build Tools 2022（MSVC 14.44，v143 工具集）+ SFML 3.1.0 vc17 官方预编译包（x64）** 重新编译了 **Release 版**，标准 VC++ 运行时（MSVCP140/VCRUNTIME140）Win10/11 自带，解压即用。

## 本包相对原版的修改（源码在 source-patch/）

1. **Release 构建修复**（上述）
2. **透明方案重做**：原版用 `SetLayeredWindowAttributes(LWA_COLORKEY)` 纯黑色键做透明——边缘抗锯齿像素混色后不是纯黑，抠不掉，**龙身一圈黑晕**；且素材里纯黑的瞳孔/嘴线会被色键抠成透明洞。
   改为 `UpdateLayeredWindow` 逐像素 alpha 分层窗口：GL 帧缓冲每帧 `glReadPixels` 读入 DIB → 原位预乘 alpha → ULW 上屏。
   效果：边缘平滑无黑晕、瞳孔正常、alpha=0 区域依旧鼠标穿透（与色键行为一致）。已实测 Windows 11 26100+。
   （注：DWM blur-behind 旧方案在 Win11 上会有灰色毛玻璃雾，勿用）
3. **双击触发大笑**：原版为 1.5 秒内连点 5 次，改为 500ms 内双击（单击仍是微笑）
4. **拖拽屏幕边界钳制**：原版拖拽无边界限制，可以把奶龙拖出屏幕"弄丢"；现与飞行物理共用同一套身体包围盒钳制规则，拖不出去

## 运行

解压后双击 `nailong-pet.exe`（必须保持 exe 与 assets 文件夹同目录，资源按相对路径加载）。

- 单击微笑，双击大笑+音效，拖住快速松手可甩飞（带重力弹跳）
- 右键菜单：关闭 / 托盘 / 固定位置 / 禁用大笑 / 静音 / 刷新状态 / 系统信息
- CPU 或内存过载时奶龙会哭；连续使用 1 小时进入疲惫提醒
- `Escape` 或右键菜单退出

## 自己构建

```bash
# 依赖：VS Build Tools 2022（C++ 工作负载，含 CMake）+ SFML 3.1.0 vc17 x64 预编译包
# 下载：https://www.sfml-dev.org/download/sfml/3.1.0/
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 -DSFML_DIR="<SFML-3.1.0>/lib/cmake/SFML"
cmake --build build --config Release
# 产物 build/Release/nailong-pet.exe，与 assets/ 同目录运行
```

## License

上游代码 MIT（见原仓库）；奶龙角色形象版权归原 IP 方所有，仅供娱乐。
