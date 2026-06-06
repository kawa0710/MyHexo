title: dinit 與 systemd 的不同
author: Kawa
tags:
  - systemd
  - dinit
  - Linux
categories:
  - Linux
date: 2026-06-05 12:43:00
---
[更新：2026-06-05 12:43:00]

## ctl 指令
  - systemctl
  - dinitctl

## 設計哲學差異 (Design Philosophy)：
  - systemd 是「全能型」的服務管理器（包含日誌管理、網路配置、硬體識別等），包山包海，能包全包。
  - dinit 追求的是「Unix 風格（Do one thing and do it well）」，它只專注於處理服務啟動與依賴關係，所以 dinit 啟動速度更快、資源佔用極低。

## dinit 服務描述檔本身不支援直接宣告變數，如果執行要用到變數，要用以下寫法：
```dinit service description
command = /bin/sh -c "export 鍵值=變數 && export 鍵值=變數 && exec 執行檔"
```

## 如果安裝的套件沒有為 dinit 設計，服務要自己寫描述檔
以 input remapper 為例（我用它來對應沒有 Linux 驅動的 SlimBlade Trackball 及 ELECOM HUGE 按鍵）
- 執行路徑在 /usr/bin/input-remapper-service
- 在 /etc/dint.d/ 建立服務描述檔「input-remapper」
```bash
ls /etc/dinit.d/input-remapper
-rw-r--r-- 1 root root 117 Jun  4 14:19 /etc/dinit.d/input-remapper
```
  - 內容如下：
```dinit service description
# description = input-remapper service
command = /usr/bin/input-remapper-service
type = process
depends-on = lightdm
restart = true
```
> 我不會在 tty 使用滑鼠故讓它在 lightdm 後啟動；當然若沒有 lightdm 就要自行替換

- 更多撰寫可用的內容參考：[dinit SERVICE_PROPERTIES](https://davmac.org/projects/dinit/man-pages-html/dinit-service.5.html#SERVICE_PROPERTIES)
- 服務開機啟動用和 systemd 相似的 `dinitctl enable input-remapper`，或手動建軟連結到 boot.d 目錄下
````
sudo ln -s ../input-remapper /etc/dinit.d/boot.d/input-remapper
````

## dinit 系統層級服務放在 /etc/dinit.d, 個人服務放在 ~/.config/dinit.d
Artix 我安裝在個人筆電，因此服務不分系統或個人，全部放在 /etc/dinit.d

## systemd 與 dinit 相似的指令

| 功能描述 | systemd 指令 | dinit 指令 | 備註 / 行為差異 |
| :--- | :--- | :--- | :--- |
| **立即啟動服務** | `systemctl start <service>` | `dinitctl start <service>` | 如果指令失敗就要手動建立服務檔到 /etc/dinit.d |
| **立即停止服務** | `systemctl stop <service>` | `dinitctl stop <service>` |  |
| **重啟服務** | `systemctl restart <service>` | `dinitctl restart <service>` |  |
| **查看服務狀態** | `systemctl status <service>` | `dinitctl status <service>` | dinit 沒有內建日誌故看不到日誌 |
| **啟用開機自啟** | `systemctl enable <service>` | `dinitctl enable <service>` |  如果指令失敗就要手動建立服務檔到 /etc/dinit.d 及 /etc/dinit.d/boot.d |
| **停用開機自啟** | `systemctl disable <service>` | `dinitctl disable <service>` |  |
| **列出所有載入服務** | `systemctl list-units --type=service` | `dinitctl list` | dinit 會用標誌如 `[{+}]`（已啟動）、`[{-}]`（已停止）顯示，比較直覺。 |
| **關機與重啟系統** | `systemctl poweroff` / `reboot` | `dinitctl shutdown` |  |

---

## dinit 與 systemd 不同的功能

systemd 是一個龐大且包山包海的「系統生態系」，而 dinit 僅專注於「Process 與相依性管理」，所以用 systemd 習以為常的「魔法」，在 dinit 要用的話就要找替代方案。

| systemd 獨有功能 / 指令 | 功能描述 | dinit 的替代方案與核心哲學差異 |
| :--- | :--- | :--- |
| **`journalctl -u <service>`** | 查看特定服務的系統日誌 | **dinit 沒有內建日誌系統。** 你必須在服務描述檔中宣告 `logfile = /path/to/log` 導向檔案，或搭配 `socklog`、`s6-log` 等輕量化 syslog 服務。 |
| **`systemctl daemon-reload`** | 重新載入設定檔變更 | dinit 採用動態按需讀取（On-demand）。修改描述檔後再次執行 `start`/`restart` 時它會自動偵測變更並彈出警告；也可以用 `dinitctl reload <service>` 手動強制重載。 |
| **`systemctl mask <service>`** | 強制鎖定服務防止被拉起 | **dinit 沒有 mask 核心指令。** 標準作法是直接刪除、改名描述檔，或在自訂目錄建立一個同名但指向 `/dev/null` 的符號連結，直接讓 dinit 載入失敗。 |
| **`systemctl list-dependencies`** | 以樹狀圖列出服務相依性 | 執行 `suo dinit-check` 看系統服務啟動鏈**列表**，`dinit-check`看個人服務。 |

