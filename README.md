GLFW
![Build status](https://github.com/glfw/glfw/actions/workflows/build.yml/badge.svg)
![Build status](https://ci.appveyor.com/api/projects/status/0kf0ct9831i5l6sp/branch/master?svg=true)
简介
GLFW 是一个开源、跨平台的 C 语言库，用于 OpenGL、OpenGL ES 和 Vulkan 应用程序的开发。它提供了一个简单且与平台无关的 API，用于创建窗口、上下文和表面，读取输入、处理事件等。
GLFW 主要使用 C99 编写，在 macOS 平台上部分功能使用了 Objective-C。
支持 Windows、macOS 和 Linux，并可在多种类 Unix 系统上运行。在 Linux 上同时支持 Wayland 和 X11 显示协议。
GLFW 采用 zlib/libpng 许可证 发布。
你可以从 官网下载 最新稳定版源码或预编译的 Windows/macOS 二进制文件。每个版本自 3.0 起均在 GitHub Releases 页面提供源码和二进制包。
在线 文档 包含教程、开发指南和完整的 API 参考，也包含在源码包中（GitHub 自动生成的包除外）。
最新版本的 更新日志 列出了新增功能、注意事项和弃用项。
完整的 版本历史 记录了每个版本的所有用户可见变更。
GLFW 的诞生离不开全球众多贡献者的工作——无论是报告问题、提供社区支持、提交代码、测试功能、校对文档，还是提出建议。
特别说明：在 Wayland 下运行 Minecraft
Minecraft（Java 版）默认通过 LWJGL 使用 GLFW 创建窗口。然而，官方 GLFW 在 Wayland 下存在一些限制（如无法设置窗口图标、无法聚焦窗口），会导致游戏启动失败或行为异常。
为此，我们可以 手动编译一个修改版的 GLFW，绕过这些限制，从而让 Minecraft 在原生 Wayland 环境下正常运行。
修改要点（基于 GLFW 3.4+）

    调整平台优先级：确保 Wayland 优先于 X11 被选中。
    屏蔽非致命错误：将 GLFW_FEATURE_UNAVAILABLE 错误降级为警告，避免因不支持的功能（如设置窗口图标、聚焦窗口）导致程序退出。

具体补丁如下：
diff

--- a/src/platform.c
+++ b/src/platform.c
@@ -49,12 +49,12 @@
 #if defined(_GLFW_COCOA)
     { GLFW_PLATFORM_COCOA, _glfwConnectCocoa },
 #endif
-#if defined(_GLFW_X11)
-    { GLFW_PLATFORM_X11, _glfwConnectX11 },
-#endif
 #if defined(_GLFW_WAYLAND)
     { GLFW_PLATFORM_WAYLAND, _glfwConnectWayland },
 #endif
+#if defined(_GLFW_X11)
+    { GLFW_PLATFORM_X11, _glfwConnectX11 },
+#endif
 };

diff

--- a/src/wl_window.c
+++ b/src/wl_window.c
@@ -2109,8 +2109,7 @@
 void _glfwSetWindowIconWayland(_GLFWwindow* window, int count, const GLFWimage* images)
 {
-    _glfwInputError(GLFW_FEATURE_UNAVAILABLE,
-                    "Wayland: The platform does not support setting the window icon");
+    fprintf(stderr, "!!! Ignoring Error: Wayland: The platform does not support setting the window icon\n");
 }

@@ -2353,8 +2352,7 @@
 void _glfwFocusWindowWayland(_GLFWwindow* window)
 {
-    _glfwInputError(GLFW_FEATURE_UNAVAILABLE,
-                    "Wayland: The platform does not support setting the input focus");
+    fprintf(stderr, "!!! Ignoring Error: Wayland: The platform does not support setting the input focus\n");
 }

    💡 注意：上述修改不会真正实现“设置图标”或“聚焦窗口”的功能，但会阻止 GLFW 因这些缺失功能而报错退出，从而使 Minecraft 能继续运行。

编译步骤
bash

# 克隆源码
git clone https://github.com/glfw/glfw.git
cd glfw

# 应用上述补丁（保存为 glfw.patch 后）
git apply glfw.patch

# 配置并编译（启用 Wayland 支持）
cmake -S . -B build -D GLFW_BUILD_WAYLAND=1 -D BUILD_SHARED_LIBS=ON -D GLFW_BUILD_EXAMPLES=no -D GLFW_BUILD_TESTS=no -D GLFW_BUILD_DOCS=no

cd build
make -j$(nproc)
sudo make install  # 默认安装到 /usr/local/lib/libglfw.so

NVIDIA 显卡用户额外设置
如果你使用 NVIDIA 闭源驱动，可能还需要设置以下环境变量以避免 EGL 初始化失败：
bash

export __GL_THREADED_OPTIMIZATIONS=0


更多兼容性信息请参阅 兼容性指南。

使用与贡献

    使用 GLFW：请阅读 官方文档
    报告 Bug：请提交至 GitHub Issues
    贡献代码：请阅读 贡献指南

联系我们

    官网：https://www.glfw.org/
    论坛：https://discourse.glfw.org/
    GitHub：https://github.com/glfw/glfw

    本文档部分内容参考自：Ricardo's Blog - 让Minecraft运行在Wayland下
    修改旨在帮助 Linux 用户在现代 Wayland 桌面环境中流畅运行 Minecraft。
