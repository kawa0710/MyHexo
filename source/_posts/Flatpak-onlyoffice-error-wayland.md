title: flatpak 安裝的 onlyoffice 打不開
author: Kawa
tags:
  - flatpak
  - onlyoffice
  - niri
  - wayland
  - Linux
categories:
  - Linux
date: 2026-06-05 17:31:38
---
[更新：2026-06-05 ]17:31:38
 
不知道為什麼 flatpak 安裝的 onlyoffice 打不開，
改用指令方式開啟 `flatpak run --user org.onlyoffice.desktopeditors` 看有什麼錯誤訊息。

google 了錯誤訊息
```bash
flatpak run qt.qpa.screen:
QXcbConnection: Could not connect to display
Could not connect to any X display.
```

找到了 flatpak Github Repo 的 issue [https://github.com/flatpak/flatpak/issues/5658](https://github.com/flatpak/flatpak/issues/5658)
「pizdjuk on Jul 22, 2024」這樓有正解！

如果有裝 flatseal，在 onlyoffice 關閉「fallback to x11」，
沒有的話下指令關閉
```bash
flatpak override --user --nosocket=fallback-x11 org.onlyoffice.desktopeditors
```

若要檢查可以執行
```bash
flatpak override --user --nosocket=fallback-x11 org.onlyoffice.desktopeditors
```

結果如下：
```bash
[Context]
shared=network;ipc;
sockets=x11;wayland;pulseaudio;
devices=dri;
filesystems=home;/tmp;xdg-run/gvfsd;~/.local/share/gtk-3.0;host;xdg-run/gvfs;
```
在 sockets 沒有了 fallback-x11。

> 搞不懂 app 執行時與 x11 和 wayland 的糾葛...
