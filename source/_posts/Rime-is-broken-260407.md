---
title: Rime 輸入法崩潰了（2026-04-07）
date: 2026-04-07 12:15:25
categories: Linux
tags:
  - Arch Linux
  - CachyOS
  - Rime
  - 中州韻
  - 輸入法
---

最近一次更新後，Rime 輸入法不能正常使用。
執行指令

```bash
fcitx5 --verbose=5
```

查到了錯誤訊息：

```text
Failed to load library for addon rime on "/usr/lib/fcitx5/librime.so". Error: /usr/lib/librime.so.1: undefined symbol: _ZTIN6marisa9ExceptionE
```

，據此 google 到了相關的[解決方案](https://discuss.cachyos.org/t/bug-librime-in-cachyos-extra-v3-repository-symbol-lookup-error-due-to-marisa-library-update/27331)－ 「**裝 extra 庫的 librime**」：

```bash
sudo pacman -S extra/librime
```

或

```bash
paru -S extra/librime
```

；完成後，Rime 輸入法恢復正常。

原因出自 cachyos-extra-v4/librime 的版本，不知道是太新，還是官方 Build 時有問題...

---
這是使用 CachyOS 半年來，第一次遇到「滾動式」更新直接弄壞現有功能，不過好在我還有裝 fcitx5 的倉頡輸入法備用，實際上也沒多大影響，每日工作用 CachyOS 對我來說還是挺穩定的。