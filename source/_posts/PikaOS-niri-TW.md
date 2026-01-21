title: 安裝 PikaOS + Niri 以及中文輸入法 Fcitx5
author: Kawa
tags:
  - PikaOS
  - Niri
  - 中文輸入法
  - Fcitx5
categories:
  - Linux
date: 2025-12-25 15:56:00
---
[更新：2025-12-31 12:33:00]

# 安裝 PikaOS + Niri
- `PikaOS 的 ISO 以 rEFInd 啟動`，以 VM 安裝時要選 UEFI 啟動
![VMWare UEFI](images/PikaOS-Niri-TW/VMWare UEFI.png)
[參考：Hyper-V UEFI](https://blog.vnet.tw/hyper-v-%E4%BD%BF%E7%94%A8uefi-%E6%A8%A1%E5%BC%8F%E5%AE%89%E8%A3%9D-ubuntu/)

- 安裝過程可以在 youtube 找到英文及在 bilibili 找到中文語音的安裝過程。數量不多，因為 PikaOS 真的是比較小眾的 distro。

# 安裝 中文輸入法 fcitx5

## 安裝 fcitx5

```bash
pikman i fcitx5 fcitx5-config-qt im-config
```
（不建議從 Flatpak 裝，因為 run 在沙盒裡可能不能在非 Flatpak 的 App 使用，而且 Rime 相關的套件可能找不到或裝不起來）

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

![fcitx5 臺繁中](images/PikaOS-Niri-TW/fcitx_tw.png)
![fcitx5 簡中](images/PikaOS-Niri-TW/fcitx_cn.png)
![fcitx5 港繁中](images/PikaOS-Niri-TW/fcitx_hk.png)

## 重啟動

## 不能用中文打字 APP（2025/12/31、2026/01/21 更新）

目前已知有：

- Super + D 打開的 pikabar-luncher （已解決、2026/01/12系統更新後又不能使用）
  - 解決方法：編輯 /ect/environment，加入「XMODIFIERS=@im=fcitx」（2026/01/21 確認已失效）
  - 重啟 PikaOS。
- Pika Software Manager 確認無法使用中文輸入法
  - 它其實是 COSMIC Store，現在 COSMIC 專用的 App 皆不支援輸入法
  - 可能要等 2026 下半年 COSMIC 才有更新


# 心得

相比 Arch Linux、CachyOS，能輕鬆安裝好 Niri 的 PikaOS 就很值得一用，不僅 UI 漂亮操作也流暢。


雖然商店不能打中文但並不影響日常甚至是工作的使用。PikaOS 主打「玩遊戲的體驗」，單機遊戲一定是沒問題，就是不曉得要連網互動的遊戲能不能使用中文輸入法...目前還沒有測試過。


近期準備用 PikaOS + Niri 開發 Web 前、後端實際體會一段時間，看看是否適合長期使用。


（2026/01/21 更新心得）使用了 PikaOS + Niri 超過 10 個工作日，主要集中在：RustDesk、Ksnip(截圖)、Windsurf、Faltpak 安裝的 Line/Discord/OnlyOffice/Draw.io/Trayscale(Tailscale GUI)/dbeaver 等...在 3 個螢幕文書作業、畫 API/UI 文件線稿、製作 Markdown 文件等工作及Zen Browser 收發信/上網/查資料/看PDF/串流影音，PikaOS 本身是穩定的，更不用說 Niri 操作視窗的流暢了。（最近工作內容都在規劃、分析類，除了開資料庫修改 SQL 完全沒有 Coding）

為了解決 pikabar-luncher 不能使用中文輸入法的問題，查到了 Niri 的版本，以及用了較早版本 noctalia shell 客製成 pika shell；我使用 CachyOS + Niri 的時間雖然比在 PikaOS + Niri 多，但 PikaOS + Niri 預設的快速鍵、客製的 shell、自帶的剪貼簿歷史還是比 CachyOS + Niri 順手多，幾乎沒改過快速鍵及裝其他套件。

但是在 CachyOS 頻繁收到 noctalia shell 的更新，noctalia shell 進化得很快，有新功能也有更精緻的 UI，暫時我還會留在 CachyOS + Niri 嘗試新的 noctalia shell。