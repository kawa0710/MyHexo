title: CachyOS 轉 Artix + Dinit 日常工作 1 個月心得
author: Kawa
tags:
  - CachyOS
  - Artix
  - Dinit
  - chkupd
  - 自動檢查套件可更新數量
  - Linux
categories:
  - Linux
date: 2026-06-05 20:48:28
---
[更新：2026-06-05 20:48:28]


我沒有什麼「信仰」，用 Artix + Dinit 純為了開機速度，而且它也夠穩定，足以做為日常工作的環境。

每日工作舉凡 Antigravity， Bottles + Line， Discord， Zen Browser， Helium， Zed， Winboat, niri, ksnip, gpu screen recoder，Steam 都沒什麼問題。開機 10 秒就是爽！！

原本日常工作都在 CachyOS，轉到 Artix 唯一轉不過來的只有 Btrfs Assistant：強依賴 systemd 執行作業。

不過這是純工作加放假玩 Steam...
除了 input-remapper、syncthing 和 tailscale 沒什麼服務要執行也沒用到 systemd 專用服務，要是哪天要來折騰家用 Server 我還是會使用 systemd 的 linux；systemd 慢是慢，但也是真的方便。

說回 CachyOS，用了超過 7 個月真的是被它慣壞了，Cachy Hello、Cachy Update 以及各種小優化，換到 Antrix 後還特別把 CachyOS 優化後的 bash script 來用，也仿照 Cachy Update 做了個開機檢查套件更新數量的 scirpt。

CachyOS 和 Artix 仍持續交替著用沒有刪除，除了開機速度慢，CachyOS 是最適合我的 distro 第一名，有空要再繼續優化它。


## 自動檢查套件可更新數量
新增檔案 `~/.config/fish/functions/chkupd.fish`

```fish
function chkupd --description '自動檢查套件可更新數量(每日首次自動同步，後續快速檢查)'
    set -l state_dir "$HOME/.local/state/chkupd"
    set -l sync_date_file "$state_dir/last_sync_date" # 將檢查狀態存入檔案

    test -d $state_dir; or mkdir -p $state_dir

    set -l today (date +%Y%m%d)
    set -l last_sync (cat $sync_date_file 2>/dev/null)
    set -l sync_opt -n
    set -l os (grep PRETTY_NAME /etc/os-release | cut -d'"' -f2 | sed 's/Linux//g')

    # 判斷是否需要同步：
    # 1. 使用者手動輸入 'chkupd sync'
    # 2. 或者今天還沒執行過同步 (檔案內容不等於今日日期)

    if test "$argv[1]" = sync -o "$last_sync" != "$today"
        set sync_opt ""
        echo $today >$sync_date_file
    end

    # 1. 檢查 Arch/CachyOS/Artix (使用 Arch 系應該都有的 checkupdate)
    if command -v checkupdates >/dev/null
        set -l a_count (checkupdates 2>/dev/null | count)

        if test $a_count -gt 0
            echo "$os 套件更新數量: $a_count"
            notify-send "套件更新提醒" "$os 套件更新數量: $a_count"
        end
    end

    # 2. 檢查 AUR (使用 paru -Qua 或 yay -Qua)
    set -l a2_count 0
    if command -v paru >/dev/null or command -v yay >/dev/null
        if command -v paru >/dev/null
            set a2_count (paru -Qua 2>/dev/null | count)
        end

        if command -v yay >/dev/null
            set a2_count (yay -Qua 2>/dev/null | count)
        end

        if test $a2_count -gt 0
            echo "AUR 套件更新數量:  $a2_count"
            notify-send "套件更新提醒" "AUR 套件更新數量:  $a2_count"
        end
    end

    # 3. 檢查 Flatpak
    if command -v flatpak >/dev/null
        set -l f_count (flatpak remote-ls --updates 2>/dev/null | count)

        if test $f_count -gt 0
            echo "Flatpak 套件更新數量:  $f_count"
            notify-send "套件更新提醒" "Flatpak 套件更新數量:  $f_count"
        end
    end
end
```
> 也可以到我的 [Github](https://github.com/kawa0710/MyLinuxScripts/blob/main/chkupd.fish) 下載。（英文註釋）

檢查更新放在 fish functions 是因為我主要使用 fish，而當 fish shell 被叫起時就會執行 chkupd.fish 所以也不用特別設定開機服務，省事不少。
