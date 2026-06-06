title: Artix Linux + dinit 開機只要10秒！
author: Kawa
tags:
  - Artix
  - dinit
  - Linux
categories:
  - Linux
date: 2026-06-05 11:00:00
---
[更新：2026-06-05 11:00:00]

- 筆電規格 laptop spec：
  - CPU: i7-11800H （4.60 GHz）
  - Intel 內顯 (iGPU) 及 NVIDIA 獨顯 (dGPU)
  - RAM: 32 G


從 Grub 按下 Artix Linux 到 tty 不到 7 秒；到 lightdm 只要 10 秒。
但在 CachyOS + systemd，開機極限只能在 34 秒左右。
```bash
systemd-analyze
Startup finished in 9.982s (firmware) + 8.661s (loader) + 897ms (kernel) + 3.413s (initrd) + 11.924s (userspace) = 34.879s
graphical.target reached after 11.920s in userspace. 
```

dinit is blazing fast!!
