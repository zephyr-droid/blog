# CachyOS Post Install

CachyOS的默认设置需要进行调整，以优化其使用体验。

## 目录

- [So Nvidia, fuck you!](#so-nvidia-fuck-you)
- [中文字体](#中文字体)
- [家目录](#家目录)
- [系统设置](#系统设置)
- [社区仓库](#社区仓库)
- [装机必备](#装机必备)
- [游戏](#游戏)

## So Nvidia, fuck you!

我的GTX1060在Wayland下存在严重的兼容性问题，所以不得不使用X11会话。

如果你的独立显卡也是16系以下的老卡，显卡工作时会出现随机卡顿，可以试试看。

首先，你需要安装`Xorg`组件：

```shell
sudo pacman -S --needed plasma-x11-session kwin-x11 xorg
```

重启后，在登录界面的左下角，更换你的会话即可生效。

执行以下命令检查，如果出现`x11`则表示设置成功：

```shell
echo $XDG_SESSION_TYPE
```

## 中文字体

安装系统后，在默认情况下，汉字显示为韩文字形。

要解决这个问题，可以修改字体配置：

```shell
sudo pacman -S --needed inter-font adobe-source-serif-fonts noto-fonts-cjk noto-fonts-emoji ttf-sarasa-gothic
wget https://github.com/szclsya/dotfiles/raw/refs/heads/master/fontconfig/fonts.conf
mkdir -p ~/.config/fontconfig && cp fonts.conf ~/.config/fontconfig
```

或者将中文字形的优先级调整到其他异体字形之前。

在`/etc/fonts/conf.d/`下创建`64-language-selector-prefer.conf`文件：

```shell
touch /etc/fonts/conf.d/64-language-selector-prefer.conf
```

写入：

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
    </prefer>
  </alias>
  <alias>
      <family>serif</family>
      <prefer>
        <family>Noto Serif CJK SC</family>
        <family>Noto Serif CJK TC</family>
        <family>Noto Serif CJK JP</family>
      </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Noto Sans Mono CJK SC</family>
      <family>Noto Sans Mono CJK TC</family>
      <family>Noto Sans Mono CJK JP</family>
    </prefer>
  </alias>
</fontconfig>
```

然后更新字体缓存即可生效：

```shell
fc-cache -fv
```

执行以下命令检查，如果出现`NotoSansCJK-Regular.ttc: "Noto Sans CJK SC" "Regular"`则表示设置成功：

```shell
fc-match -s | grep 'Noto Sans CJK'
```

根据Github的讨论，尽管Flatpak应用程序可以使用系统字型，但却不采纳系统fonts.conf的设定，而是以自己的优先级来显示字型，有时候在Wayland下还会导致字型模糊(跟缩放无关)。

只要开放全部的Flatpak程序读取`xdg-config/fontconfig:ro`就行，等同允许读取`~/.config/fontconfig/`。

```shell
flatpak --user override --filesystem=xdg-config/fontconfig:ro
```

## 家目录

像CachyOS这类，对用户友好的发行版，会在安装的时候，帮你搞定语言的问题。当你选中文安装，家目录下的常用文件夹也会变成中文的。

不过对于经常要用终端的用户来说，肯定是不太方便。跳转常用目录还得切换输入法，很是麻烦。

执行以下命令，将家目录文件夹改成英文：

```shell
LC_ALL=C.UTF-8 xdg-user-dirs-update --force
```

重启后，在Dolphin里把常用位置也改成英文。

## 系统设置

KDE应用的默认配置，不是很符合我个人的使用习惯。下面列出我调整后的选项，希望对你有所帮助：

1. 鼠标和触摸板鼠标：**禁用指针加速度**
2. 锁屏，自动锁定屏幕：**不自动锁屏**
3. 电源管理，空闲时：**无操作**
4. 桌面会话，登录时自动启动应用程序：**启动为空会话**

## 社区仓库

官方仓库的软件包数量有限，我们可以通过添加社区仓库，以此扩充可用的软件包来源。

例如要添加[ArchLinuxCN](https://www.archlinuxcn.org/archlinux-cn-repo-and-mirror/)，执行：

```shell
printf '%s' '[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
' | sudo tee -a /etc/pacman.conf
```

之后，通过以下命令安装`archlinuxcn-keyring`包，以导入GPG密钥：

```shell
sudo pacman -Sy archlinuxcn-keyring
```

## 装机必备

完成以上的基础配置后，我会优先安装那些“装机必备”软件包。你可以参考一下我的选择，也许对你有所帮助。

### 中文输入法

[小企鹅输入法](https://fcitx-im.org/wiki/Fcitx_5/zh-cn)是一个开源的输入法框架。

接下来，我将介绍如何在CachyOS上配置小企鹅输入法5（以下简称fcitx5）。

1. 安装
   基本的fcitx5安装包括：主程序、配置程序、输入法模块和输入法引擎。
   要安装这些组件，执行：
   
   ```shell
   sudo pacman -S fcitx5-im fcitx5-rime
   ```
   
   `fcitx5-im`包组提供主程序、配置程序和输入法模块。
   `fcitx5-rime`包提供[中州韵输入法引擎](https://rime.im/)。

2. 环境变量
   为了让fcitx5能被正确识别并参与输入，需要设置相应的环境变量。执行以下命令，然后重新登录：
   
   ```shell
   printf '%s' 'export XMODIFIERS=@im=fcitx
   export GTK_IM_MODULE=fcitx
   export QT_IM_MODULE=fcitx
   ' | sudo tee -a /etc/profile
   ```

3. 配置词库
   执行以下命令，获取[白霜拼音](https://github.com/gaboolic/rime-frost)并导入：
   
   ```shell
   git clone --depth 1 https://github.com/gaboolic/rime-frost Rime
   cp -r Rime/* ~/.local/share/fcitx5/rime
   ```
   
   按`Ctrl`+空格键切换输入法。右键任务栏的“中州韵”图标，进入“小地球”的菜单，重新部署。

4. 主题美化
   为当前用户安装[Mellow](https://github.com/sanweiya/fcitx5-mellow-themes)主题：
   
   ```shell
   git clone https://github.com/sanweiya/fcitx5-mellow-themes.git
   cd fcitx5-mellow-themes && mkdir -p ~/.local/share/fcitx5/themes
   cp -r ./mellow-* ~/.local/share/fcitx5/themes
   ```
   
   然后前往“系统设置，输入法，配置附加组件，经典用户界面”设置主题。

### Nerd Fonts

Nerd Fonts是包含大量图标的字体，主要用于终端显示。比如：Yazi。

我推荐使用[Maple Mono](https://github.com/subframe7536/Maple-font)，中英文完美对齐且风格统一，应该是绝无仅有了。

要安装Maple Mono，需要先安装AUR助手：

```shell
sudo pacman -S paru
```

然后用AUR助手从ArchLinuxCN社区仓库获取软件包：

```shell

```

### Flatpak

[Flatpak](https://flatpak.org/)是一个Linux桌面程序的构建、分发和沙箱化运行系统。

1. 安装
   执行以下命令，然后重启：
   
   ```shell
   sudo pacman -S flatpak
   ```

2. 配置
   官方源的速度太慢了，建议更换其他镜像源：
   
   ```shell
   sudo flatpak remote-modify flathub --url=https://mirrors.ustc.edu.cn/flathub
   ```

## 游戏

### Minecraft

第三方启动器我比较推荐[Prism Launcher](https://prismlauncher.org/)，它是开源的，且在Linux社区的讨论度较高，基本功能都有。目前用下来唯一的缺点是下载实例的速度较慢。可能是没有像[PCL](https://afdian.com/p/0164034c016c11ebafcb52540025c377)那样用多线程下载。

1. 安装
   
   建议用`flatpak`安装：
   
   ```shell
   flatpak install flathub org.prismlauncher.PrismLauncher
   flatpak install flathub com.github.tchx84.Flatseal
   ```

2. 配置
   在[Flatseal](https://flathub.org/en/apps/com.github.tchx84.Flatseal)里打开Prism Launcher的**GPU加速权限**。

### GameMode

[GameMode](https://github.com/FeralInteractive/gamemode)是一个Linux下的守护进程和库组合，允许游戏请求一组优化暂时应用于主机操作系统或游戏进程。
安装`gamemode`包和`lib32-gamemode`包：

```shell
sudo pacman -S --needed gamemode lib32-gamemode
```

将自己添加到`gamemode`用户组：

```shell
sudo gpasswd --add user gamemode
```

如果没有这个用户组，GameMode用户守护进程将没有权限更改CPU管理器或进程的优先级。

## 参考

- [GameMode](https://wiki.archlinuxcn.org/wiki/GameMode)
- [小企鹅输入法5](https://wiki.archlinuxcn.org/wiki/Fcitx_5)
- [简体中文本地化](https://wiki.archlinuxcn.org/wiki/%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E6%9C%AC%E5%9C%B0%E5%8C%96)
- [Linux下的字体调校指南](https://szclsya.me/zh-cn/posts/fonts/linux-config-guide/)
- [Arch Linux CN软件仓库](https://help.mirrors.cernet.edu.cn/archlinuxcn/)
- [将家目录下的文件夹改成英文](https://ivonblog.com/posts/linux-xdg-user-dirs/)
- [Flatpak程序吃不到系统中文字型设定](https://ivonblog.com/posts/linux-fontconfig/#4-flatpak%E7%A8%8B%E5%BC%8F%E5%90%83%E4%B8%8D%E5%88%B0%E7%B3%BB%E7%B5%B1%E4%B8%AD%E6%96%87%E5%AD%97%E5%9E%8B%E8%A8%AD%E5%AE%9A)
