# libfcitx5platforminputcontextplugin

Linux下通过在线安装 Qt 后，QtCreator 无法输入中文，原因在与缺失关键插件 `libfcitx5platforminputcontextplugin` 。由此本插件用于存放该插件并给出具体解决思路。

## 1. 确定 QtCreator 版本

在 QtCreator 的 `帮助` -> `About Qt Creator...` 中存在语段：“Based on Qt xxx (GCC 10.3.1 20210422 (Red Hat 10.3.1-1), x86_64)”，由此可以确定该 QtCreator 由什么版本的 Qt 构建。**这非常关键**，关乎到能否编译成功的插件能否正常使用。

## 2. 复制插件到指定位置

需要注意到 Releases 提供的插件编译条件为：
```cmake
option(ENABLE_QT4 "Enable Qt 4" Off) # 支持 qt4
option(ENABLE_QT5 "Enable Qt 5" Off) # 支持 Qt5
option(ENABLE_QT6 "Enable Qt 6" On)  # 支持 Qt6
option(ENABLE_X11 "Enable X11 support" On) # 支持 X11
option(BUILD_ONLY_PLUGIN "Build only plugin" On) # 仅构建插件
option(BUILD_STATIC_PLUGIN "Build plugin as static" Off) # 构建静态链接库文件
option(WITH_FCITX_PLUGIN_NAME "Enable plugin name with fcitx" On) # 库文件名包含fcitx
option(ENABLE_QT6_WAYLAND_WORKAROUND "Enable Qt6 Wayland workaround" Off) # 支持 Wayland
```
请酌情选择是否自行编译还是直接使用现成，如果选择前者请跳转到第三部分：手动编译。

> ⚠️
>
> 由于自定义安装的Qt位置未知，所以在下面通过 `.../Qt/` 表示 Qt 的安装路径。

```bash
# 赋予插件 可读、可写、可执行 权限
chmod 777 ./libfcitx5platforminputcontextplugin.so
# 复制插件至 Qt Creator 下，使得 QtCreator 能够正常输入中文
cp ./libfcitx5platforminputcontextplugin.so .../Qt/Tools/QtCreator/lib/Qt/plugins/platforminputcontexts/
# 复制插件至 Qt Desingn Studio 下
cp ./libfcitx5platforminputcontextplugin.so .../Qt/Tools/QtDesignStudio/lib/Qt/plugins/platforminputcontexts
# 复制插件至 Qt 插件库中
cp ./libfcitx5platforminputcontextplugin.so .../Qt/6.8.1/gcc_64/plugins/platforminputcontexts/
```

在 Releases 中存放有编译成功的特定 Qt 版本插件，如果符合 QtCreator 要求则可以直接使用，如何不行则需要查看下面的步骤进行手动编译。

## 3. 手动编译

###  QtCreator 无法输入中文

> 问题：当使用 `fcitx5` 时，无法对 `QtCreator` 输入中文。

> 原因：QtCreator 缺乏 ` libfcitx5platforminputcontextplugin.so` 动态链接库文件。

对此，我们需要通过编译 [fcitx 5-qt][1] 获得该动态链接库文件。

### 3.1 安装依赖

```bash
sudo apt-get install libxkbcommon-x11-dev libxcb1-dev libx11-xcb-dev libxcb-glx0-dev libxcb-shape0-dev libxcb-xfixes0-dev extra-cmake-modules libgl1-mesa-dev
```

### 3.2 配置编译环境

> 💡
>
> 以下路径均为笔者自定义 Qt 环境配置，请根据自身情况做出合理更改。
>
> 在使用 Qt 在线软件安装时，默认情况 Qt 安装路径：非 root 账户在 `~/Qt` 下，root 账户在 `/opt/Qt` 下。

#### 3.2.1 下载 `fcitx5-qt`

```bash
git clone https://github.com/fcitx/fcitx5-qt.git
# 或者使用ssh，需要与github进行ssh连接
git clone git@github.com:fcitx/fcitx5-qt.git
```

#### 3.2.2 配置临时环境变量

> 💡
>
> 根据下载的 `QtCreator` 严格确定 Qt 版本。
>
> 可以从 `QtCreator` 的 `帮助->About Qt Creator` 中查看到类似于“Based on Qt xxx (GCC ...)”的语句，确定需要的 Qt 版本。
>
> 一般来说，构建 `QtCreator` 的 Qt 版本比当时最新发行版本低一级。

```bash
# 进入fcitx5-qt文件夹
cd fcitx5-qt
# 创建构建文件夹
mkdir build
# 设置QtCreator库文件环境
export LD_LIBRARY_PATH=/home/aaron/enviroment/qt/Tools/QtCreator/lib/Qt/lib
# 设置Qt环境
export PATH=/home/aaron/enviroment/qt/6.7.3/gcc_64/bin:$PATH
```

#### 3.2.3 定义 `CMake` 构建选项

有一些选项是我们不需要的，可以进行自定义选择。

```bash
vim CMakeLists.txt 
```

在这里，我选择进行以下配置：

```cmake
option(ENABLE_QT4 "Enable Qt 4" Off) # 支持 qt4
option(ENABLE_QT5 "Enable Qt 5" Off) # 支持 Qt5
option(ENABLE_QT6 "Enable Qt 6" On)  # 支持 Qt6
option(ENABLE_X11 "Enable X11 support" On) # 支持 X11
option(BUILD_ONLY_PLUGIN "Build only plugin" On) # 仅构建插件
option(BUILD_STATIC_PLUGIN "Build plugin as static" Off) # 构建静态链接库文件
option(WITH_FCITX_PLUGIN_NAME "Enable plugin name with fcitx" On) # 库文件名包含fcitx
option(ENABLE_QT6_WAYLAND_WORKAROUND "Enable Qt6 Wayland workaround" Off) # 支持 Wayland
```

#### 3.2.4 编译

```bash
# cmake 编译
cmake -DCMAKE_BUILD_TYPE=Release \
-DCMAKE_POSITION_INDEPENDENT_CODE=ON \
-B build/
# make 编译
cd build
make
```

如果没有意外，我们应该可以在 `build` 目录下找到我们所需要的动态链接库文件，具体位置为：

```bash
fcitx5-qt/build/qt6/platforminputcontext/libfcitx5platforminputcontextplugin.so
```

### 3.3 迁移插件

```bash
cd qt6/platforminputcontext/
# 设置权限（建议为完整权限）
chmod 777 ./libfcitx5platforminputcontextplugin.so
# 复制到QtCreator库目录下
cp ./libfcitx5platforminputcontextplugin.so ~/enviroment/qt/Tools/QtCreator/lib/Qt/plugins/platforminputcontexts/
# 复制到对应版本的Qt目录下
cp ./libfcitx5platforminputcontextplugin.so ~/enviroment/qt/6.7.3/gcc_64/plugins/platforminputcontexts/
```

如无出意外， `QtCreator` 应该可以正常输入中文。

---

# 参考

[Qt6 Creator 无法用 Fcitx5 输入中文](https://forum.suse.org.cn/t/topic/16363)

[1]: https://github.com/fcitx/fcitx5-qt.git
