# 🚀 [保姆级教程] 研究生双系统血泪史：Win11 + Ubuntu 22.04 + OpenEB 全流程避坑指南

大家好！作为一名事件相机（Event Camera）视觉研究的科研打工人，这几天我经历了一场与电脑底层的“殊死搏斗”。

为了在华硕笔记本上搭建一个稳定高效的 C++ 算法开发环境，我硬生生把 Windows 11 + Ubuntu 22.04 双系统安装，以及 Prophesee OpenEB 驱动库的编译流程全部踩了一遍。这里面充满了极其隐蔽的“坑”，比如系统磁盘被锁定、CMake 依赖疯狂报错等。

为了让后来者少掉几根头发，我把整个流程和所有踩坑记录整理成了这份保姆级指南。如果你也是机器人、自动驾驶或计算机视觉方向的同学，这篇教程绝对能帮你省下大半天的时间！

## 🛠️ 第一阶段：Windows 11 环境准备（防坑重灾区）
在动刀子切分硬盘之前，必须先解除 Windows 的几道封印，否则后续安装必定蓝屏或卡死。

1. 核心封印解除

关闭 BitLocker (设备加密):

路径: 设置 -> 隐私和安全性 -> 设备加密 (或在开始菜单直接搜索“管理 BitLocker”)。

操作: 将其关闭并等待解密完成。如果带着加密状态安装 Linux，可能会导致重启后 Windows 需要输入冗长的恢复密钥，甚至导致 Ubuntu 无法正确挂载和读取硬盘分区。

关闭快速启动 (Fast Startup):

路径: 控制面板 -> 硬件和声音 -> 电源选项 -> 选择电源按钮的功能。

操作: 点击“更改当前不可用的设置”，取消勾选“启用快速启动”，保存修改。开启快速启动会导致 Windows 休眠而非真正关机，这会让 Ubuntu 无法访问 Windows 的磁盘分区，也可能引发引导冲突。

2. 磁盘分区与“不可移动文件”刺客

我们需要在 Windows 的“磁盘管理”中为 Ubuntu 腾出空闲空间（建议 150GB 以上）。

为 Ubuntu 腾出空闲空间:

右键点击桌面底部 Windows 开始菜单图标，选择“磁盘管理”。

找到剩余空间较大的盘（如 D 盘或 C 盘），右键点击选择“压缩卷”。

在“输入压缩空间量”中，输入你需要分配给 Ubuntu 的大小（例如输入 153600 即分配约 150GB）。

点击“压缩”。完成后，磁盘管理中会多出一块黑色的“未分配” (Unallocated) 空间。千万不要在这里给它新建卷或格式化，保持黑色未分配状态即可。

避坑警告： 你可能会遇到“明明有 250GB 剩余空间，但只能压缩出 3GB”的诡异现象。这是因为 Windows 的虚拟内存、休眠文件等“不可移动文件”像路障一样挡在了磁盘中间。

完美解法： 别折腾系统命令了，直接下载免费的 傲梅分区助手 或 DiskGenius。右键选择调整分区大小，强行把空间挤出来，傻瓜式操作，安全无损。在磁盘末尾留下一块黑色的“未分配”空间即可（千万不要新建卷！）。

## 💽 第二阶段：制作 Ubuntu 22.04 启动盘
1. 准备镜像与工具

去 Ubuntu 官网下载 22.04.4 LTS 桌面版镜像。

下载地址: https://releases.ubuntu.com/22.04/

选择文件: 页面中点击下载 ubuntu-22.04.4-desktop-amd64.iso 文件。

准备一个 8GB 以上的 U 盘（提前备份数据，会被清空）。

下载 Rufus 烧录工具。

下载地址: https://rufus.ie/

选择文件: 在网页的“下载”区域选择标准版（如 rufus-4.x.exe）。

2. 烧录设置

分区类型： GPT

目标系统类型： UEFI (非 CSM)

文件系统： FAT32（保持默认，UEFI 必须用 FAT32 引导）

点击“开始”，如果弹出 ISOHybrid 提示，请选择“以 ISO 镜像模式写入 (推荐)”。

如果提示需要下载 GRUB 或 Syslinux 等更新文件，请点击“是”。

最后会警告清除 U 盘数据，确认无误后点击“确定”。等待进度条走完，点击“关闭”。

## 💻 第三阶段：华硕 TUF BIOS 设置
关机状态下插上刚制作好的 U 盘。

按下电源键，疯狂敲击 F2 进入 BIOS。

按 F7 进入 Advanced Mode（高级模式）。

关键一步： 找到 Security 选项卡，将 Secure Boot (安全启动) 设置为 Disabled。（如果不关，以后装 NVIDIA 显卡驱动和各类相机传感器驱动会让你欲哭无泪）。

在 Boot 选项卡中，把带有 UEFI 和你 U 盘品牌的选项移到第一启动项。

按 F10 保存并重启。

## 🐧 第四阶段：Ubuntu 纯手动分区安装 (核心避坑区)
重启后选择 Try or Install Ubuntu。这是整个双系统安装中最核心、最需要谨慎操作的环节。为了确保你未来的控制算法开发、ROS2 运行以及大规模视觉数据的处理极其稳定，我们采用纯手动的“Something else”分区方案。

1. 初始向导设置

欢迎界面 (Welcome): 在左侧语言列表中，强烈建议选择 English！在底层控制工程和 C++ 开发中，中文路径（如 /home/用户名/下载/）极其容易导致编译器（如 CMake）或 ROS2 工作空间报出莫名其妙的路径错误。保持纯英文系统环境是行业最佳实践。

键盘布局 (Keyboard layout): 选择 English (US) -> English (US)。

无线网络 (Wireless): 正常连接 Wi-Fi，用于下载必要的显卡和网卡驱动。

2. 更新与其他软件 (Updates and other software)

选择 Normal installation（正常安装）。

玄学避坑（必须注意）： 千万不要勾选 Download updates while installing Ubuntu！Ubuntu 会在后台连境外的官方源下载几百兆更新包，极其容易导致安装进度条卡死几个小时。

务必勾选： Install third-party software for graphics and Wi-Fi hardware...，这能省去后续各种硬件驱动的烦恼。

3. 安装类型 (Installation type) —— 极其关键

系统会检测到 Windows 11。绝对不要选择第一项“Install Ubuntu alongside Windows Boot Manager”（系统自动分配容易失控）。

请选择最下方的 Something else (其它选项)，点击 Continue。

4. 核心环节：自定义磁盘分区

在列表中仔细寻找你在 Windows 下压缩出来的那块标有 free space (空闲空间) 的区域。选中它，点击左下角的 + 号，依次创建以下四个分区：

第一块：EFI 引导分区 (系统启动的钥匙)

Size: 500 MB

Type: Primary (主分区)

Location: Beginning of this space

Use as: EFI System Partition

第二块：Swap 交换空间 (防止编译崩溃的缓冲)

注：编译复杂的运动学算法或构建庞大的 C++ 视觉库极其耗费内存。充足的 Swap 能防止系统内存溢出死机。

Size: 16384 MB (16GB，建议与物理内存同等大小或稍大)

Type: Logical (逻辑分区)

Location: Beginning of this space

Use as: swap area

第三块：根目录 / (系统与软件环境的家)

注：存放 Ubuntu 系统文件、CUDA、ROS2 核心组件及各类依赖。

Size: 61440 MB (60GB 及以上)

Type: Primary (主分区)

Location: Beginning of this space

Use as: Ext4 journaling file system

Mount point: /

第四块：用户目录 /home (你的个人数据仓库)

注：日常代码、事件相机数据集和传感器 Bag 包都在这里。

Size: 保持默认（向导会自动填入剩余全部空间）

Type: Logical (逻辑分区)

Location: Beginning of this space

Use as: Ext4 journaling file system

Mount point: /home

5. 引导加载器设置 (最易出错点)

在界面最下方的 Device for boot loader installation 下拉菜单中，务必选择你刚刚创建的那个 500MB 的 EFI 分区！
你可以核对列表中 EFI 分区的编号（例如 /dev/nvme0n1p6），在下拉菜单中选中同名设备。绝对不要选择整块硬盘或 Windows 的 EFI 分区。

6. 用户信息收尾

点击 Install Now，设置时区。在填写用户名（Your name / Pick a username）时，强烈建议使用全小写纯英文字母，不要加空格。等待进度条走完重启即可！

## ⚙️ 第五阶段：装机后的“软汉化”与图形算力环境配置
保留了全英文系统，但我们需要打字交流。最完美的方案是安装 Fcitx5 中文输入法。

打开终端 (Ctrl+Alt+T)：

```bash
1. 更新系统源

sudo apt update

sudo apt upgrade -y

2. 安装 Fcitx5 拼音输入法

sudo apt install fcitx5 fcitx5-chinese-addons fcitx5-frontend-gtk3 fcitx5-frontend-gtk4 fcitx5-frontend-qt5 -y
```
安装后在系统的 Language Support 里把键盘输入法系统切换为 fcitx5。

重启后（看到这里可以直接跳到5.图形算力配置部分，等待配置完成后一起重启），右上角会出现一个键盘图标，点击它选择 Pinyin 即可输入中文。




3. 安装之后的小插曲

这里有个小插曲，如果重启后没有拼音输入法，则需要手动唤醒Fcitx5。

点击菜单，打开应用搜索框。

输入 fcitx5，如果你看到一个带有键盘图标的应用，点击运行它。

观察屏幕右上角的系统托盘，看看此时有没有出现一个小键盘图标。

即便 Fcitx5 运行了，它默认可能只激活了英文键盘。我们需要手动把拼音拽进去。

点击菜单，搜索并打开 Fcitx5 Configuration (Fcitx5 配置)。

打开的配置界面分为左右两半：左边是“Current Input Method (当前输入法)”，右边是“Available Input Method (可用输入法)”。

核心动作：取消勾选右侧中间的 Only Show Current Language (仅显示当前语言) 这个复选框。（因为我们系统是全英文的，勾选状态下它会把中文输入法藏起来）。

在右侧的搜索框里输入 pinyin。

找到 Pinyin (拼音) 后，双击它，或者选中它后点击中间的向左箭头 <，把它移动到左侧的列表中。

确保现在左侧列表中有 Keyboard - English (US) 和 Pinyin 这两项。点击右下角的 Apply (应用) 然后点击 OK 关闭窗口。

现在，系统终于明白你的需求了。除了点击右上角的键盘图标手动切换外，在 Ubuntu 下最地道、最常用的输入法切换快捷键是：Ctrl + Space (按住 Ctrl 敲击空格) 或者 Ctrl + Shift。

你现在尝试按一下 Ctrl + Space，看看能不能顺利打出中文汉字了？

4.调整候选框大小

当你打出汉字的时候，会发现候选框略小，或许有人会不适应，这里提供一种设置方法。

首先打开配置工具： 点击菜单，搜索并打开 Fcitx5 Configuration (Fcitx5 配置)。

切换到附加组件： 在窗口最上方，点击中间的 Addons (附加组件) 标签页。

定位 UI 设置： 在长长的列表中往下滚动，找到名为 Classic User Interface (经典用户界面) 的选项。

进入高级配置： 点击该选项最右侧的 齿轮图标 ⚙️ (配置)。

调大字体字号 (核心步骤)：

在弹出的新窗口中，你会看到 Font (字体) 和 Menu Font (菜单字体) 两个选项。

点击它们右侧当前的字体名称按钮（默认通常显示类似 Sans 10）。

在弹出的字体选择器中，找到底部的 Size (大小) 选项。将数字果断改大，比如从 10 直接修改为 24 或 26。

点击 Select (选择) 确认修改。

应用并保存： 点击右下角的 Apply (应用)，然后点击 OK 关闭所有窗口。

现在，你随便找个可以输入文字的地方（比如打开终端或者浏览器地址栏），按 Ctrl + Space 切换出拼音打几个字试试。因为字体变大了，整个候选框也会被自动撑大，是不是瞬间变得非常清晰舒服了？

5.图形算力配置

事件相机产生的大量异步像素数据，都极度依赖 GPU 算力。Ubuntu 开源的 Nouveau 驱动性能极差，必须安装 NVIDIA 官方闭源驱动。

自动检测推荐驱动：

在终端输入以下命令，系统会列出适合你显卡的驱动版本：

```bash
ubuntu-drivers devices
```
安装推荐版本：

在输出的信息中，找到带有 recommended 字样的那一行（通常是 nvidia-driver-535 或更高版本）。然后输入：

```bash
sudo apt install nvidia-driver-535 -y
(注意：将 535 替换为你终端里显示的 recommended 版本号)
```

验证安装：

重启电脑后，打开终端输入 nvidia-smi。如果出现一个包含 GPU 温度、显存使用率的完整表格，说明显卡驱动安装完美成功。

## 👁️ 第六阶段：最终 Boss 战 - 编译 OpenEB 事件相机库
做事件相机绕不开 Prophesee 的 OpenEB。官方文档的自动化脚本由于版本更新经常失效，导致 CMake 各种花式报错。以下是我总结的纯手动填坑版编译流程。

1. 补齐底层依赖库 (依赖大礼包)

不要等 CMake 报错了再去一个个找，直接一次性全部装好：

```bash
sudo apt install libusb-1.0-0-dev libglew-dev libglfw3-dev libboost-all-dev libopencv-dev libhdf5-dev -y

sudo apt install pybind11-dev python3-pybind11 -y

sudo apt install libprotobuf-dev protobuf-compiler -y
```
(注：libusb 负责硬件通信，pybind11 是 C++ 和 Python 的桥梁，Protobuf 负责高速数据流的打包，这些都是机器人底层的常客。)

2. 克隆源码与编译
```bash
克隆仓库

git clone https://github.com/prophesee-ai/openeb.git

cd openeb

mkdir build && cd build

执行 CMake 配置

cmake .. -DBUILD_TESTING=OFF
```
3. CMake 常见报错排查

报错： Could NOT find LibUSB 或 Could not find pybind11 或 Protobuf

原因： 缺少相应的 C++ 开发头文件。

解法： 检查第 1 步的依赖是否全部安装成功。重要提示： 每次安装完缺失的依赖后，必须执行 rm CMakeCache.txt 清除旧缓存，否则重新 cmake 依然会报错！

4. 启动多线程编译

看到配置成功的提示后，即可释放 CPU 全部核心火力全开：

```bash

make -j$(nproc)

sudo make install
```

至此，你的事件相机驱动已经完美融于你的系统中了！

💡 总结
搭建底层开发环境往往比写上层算法更让人抓狂，但这正是每一个研究生成长的必经之路。把基础打牢，后面的数学推导和 ROS2 节点编写才会顺理成章。

如果你在安装过程中遇到了其他奇葩的报错，欢迎在评论区留言或者提交 Issue，我们一起探讨解决！如果这篇指南帮到了你，别忘了点个 Star ⭐️ 呀！
