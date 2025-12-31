title: 安裝 PikaOS + niri 以及中文輸入法 Fcitx5
author: Kawa
tags:
  - PikaOS
  - niri
  - 中文輸入法
  - Fcitx5
categories:
  - Linux
date: 2025-12-25 15:56:00
---
[更新：2025-12-31 12:33:00]

# 安裝 PikaOS + niri
- `PikaOS 的 ISO 以 rEFInd 啟動`，以 VM 安裝時要選 UEFI 啟動
![VMWare UEFI](images/PikaOS-niri-TW/VMWare UEFI.png)
[參考：Hyper-V UEFI](https://blog.vnet.tw/hyper-v-%E4%BD%BF%E7%94%A8uefi-%E6%A8%A1%E5%BC%8F%E5%AE%89%E8%A3%9D-ubuntu/)

- 安裝過程可以在 youtube 找到英文及在 bilibili 找到中文語音的安裝過程。數量不多，因為 PikaOS 真的是比較小眾的 distro。

# 安裝 中文輸入法 fcitx5

## 安裝 fcitx5

- 從 Pika 商店安裝


1. 從【PikaOS Welcome】 > Setup Steps > Install more apps from Software Manager
1. 「Lunch」打開 Pika Software Manager
1. 搜尋、安裝 Fcitx5


- 或 指令安裝
```bash
pikman i fcitx5 fcitx5-config-qt im-config
```


## fcitx5 輸入法
[參考 - Chakra@中文](https://chakra-zh.blogspot.com/2015/10/)
- fcitx5-chewing 新酷音
- fcitx5-table-extra 臺繁中、簡中、港繁中使用的各種輸入法
  - 臺灣繁中輸入法有：倉頡（快速倉頡第六代、倉頡大字集、倉頡第三代、倉頡第五代、速成第五代）、嘸蝦米、行列30、行列30大字集、吳語註音法
- fcitx5-chinese-addons 正簡體轉換功能
- fcitx5-rime 中州韻輸入法

```bash
pikman i fcitx5-chewing
pikman i fcitx5-table-extra
pikman i fcitx5-chinese-addons
pikman i fcitx5-rime
```

![fcitx5 臺繁中](images/PikaOS-niri-TW/fcitx_tw.png)
![fcitx5 簡中](images/PikaOS-niri-TW/fcitx_cn.png)
![fcitx5 港繁中](images/PikaOS-niri-TW/fcitx_hk.png)

## 重啟動

## 不能用中文打字 APP（2025/12/31 更新）

目前已知有：

- Super + D 打開的 pikabar-luncher （已解決）
  - 解決方法：編輯 /ect/environment，加入「XMODIFIERS=@im=fcitx」
  - 重啟 PikaOS。
- Pika Software Manager 確認無法使用中文輸入法
  - 它其實是 COSMIC Store，現在 COSMIC 專用的 App 皆不支援輸入法
  - 可能要等 2026 下半年 COSMIC 才有更新


# 心得

相比 Arch Linux、CachyOS，能輕鬆安裝好 niri 的 PikaOS 就很值得一用，不僅 UI 漂亮操作也流暢。


雖然商店不能打中文但並不影響日常甚至是工作的使用。PikaOS 主打「玩遊戲的體驗」，單機遊戲一定是沒問題，就是不曉得要連網互動的遊戲能不能使用中文輸入法...目前還沒有測試過。


近期準備用 PikaOS + niri 開發 Web 前、後端實際體會一段時間，看看是否適合長期使用。