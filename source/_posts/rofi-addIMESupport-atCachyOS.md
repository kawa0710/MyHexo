---
title: 編譯 rofi 使其支援中文輸入法@CachyOS
date: 2025-12-26 15:02:54
categories: Linux
tags: CachyOS niri rofi 中文 輸入法 seatd 鍵盤滑鼠不能用
---
CachyOS 預設安裝的 niri 使用 fuzzle 啟動器，但是目前它還找不到可以使用中文輸入法的方法。現在找到能支援中文輸入法的啟動器是 rofi，使用目前最新的 2.0 版本並且在已安裝 xcb-imdkit 的 Linux 重新編譯就可以打中文了。

**重編譯後 niri 的重要元件 seatd 好像會被停用導致不能使用鍵盤及滑鼠，還需要手動操作加回一些設定才會正常**

[參考官網安裝方法](https://github.com/davatorium/rofi/blob/next/INSTALL.md)
> 官網的套件名稱似乎是 Ubuntu 系的套件名稱，以下皆使用 Pacman 套件庫名稱


### 安裝編譯 rofi 必要套件
```bash
paru -S gcc meson ninja pkgconfig flex bison check wayland-protocols pandoc doxygen cppcheck ohcount
```

### 使輸入法生效的套件
```bash
paru -S libxcb xcb-util xcb-util-wm xcb-util-cursor xcb-imdkit
```

### 應該是不需要的額外套件，但怕影響 rofi 其他功能還是裝
```bash
paru -S pango cairo glib2 gdk-pixbuf2 startup-notification libxkbcommon libxkbcommon-x11 libmpdclient
```

### 用 git 下載最新的原始碼並編譯
```bash
git clone https://github.com/davatorium/rofi.git
cd rofi
meson setup build
ninja -C build
ninja -C build install
```

**重登入，叫出 rofi 就能打中文了**

### 如果在 niri 不能使用鍵盤及滑鼠...
1. 確認 seatd 已安裝
```bash
pacman -Qs seatd
```
若沒有就用以下指令再裝一次
```bash
sudo pacman -S seatd
```

2. 啟用並立刻啟動 seatd
```bash
sudo systemctl enable --now seatd.service
```

3. 檢查登入的帳號是否已加入 seat 群組
```bash
groups
```

輸出結果若沒有包含「seat」，用以下指令將目前登入的帳號加入 seat 群組
```bash
sudo usermod -aG seat $USER
```
4. **重新開機**，登入 niri 後滑鼠鍵盤就能正常使用了。