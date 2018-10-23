---
title: ubuntu16.04下解决Adobe Flash不是最新版问题
date: 2017-05-22 16:02:11
tags: Linux
toc: true
---

刚刚装好的我的Ubuntu16.04，我是开发人员，需要chrome浏览器（强大的调试工具），打开flash提示‘Adobe Flash不是最新版’

# 解决方法

### 1、到Adobe官网下载对应系统的谷歌浏览器flash插件

<img src='/images/google-adobe-flash.png' />

（点击下载.tar.gz源码文件）

### 2、再当前用户我这里目录下创建文件夹

```bash
sudo mkdir -p ~/.config/google-chrome/PepperFlash/25.0.0.171
```

### 3、解压flash_player_ppapi_linux.x86_64.tar.gz文件中的所有文件，放到 ~/.config/google-chrome/PepperFlash/25.0.0.171文件夹

<img src='/images/google_flash_plugin.png' />

### 4、编辑google-chrome.desktop文件。

```bash
sudo gedit  /usr/share/applications/google-chrome.desktop
```

把其中的`Exec=/usr/bin/google-chrome-stable %U`替换为`Exec=/usr/bin/google-chrome-stable %U --ppapi-flash-path=/home/当前用户/.config/google-chrome/PepperFlash/25.0.0.171/libpepflashplayer.so --ppapi-flash-version=25.0.0.171`

<img src='/images/google.desktop.png' />

### 5、关闭所有浏览器窗口重启chrome浏览器，OK成功了！

<img src='/images/google_flash_success.png' />