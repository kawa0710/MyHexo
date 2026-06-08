title: BTRFS 啟用 zswap 及休眠
author: Kawa
tags:
  - BTRFS
  - zswap
  - hibernate
  - Linux
categories:
  - Linux
date: 2026-06-06 11:31:48
---
[更新：2026-06-08 17:57:55]

> 我把 /home 也放到 BTRFS subvolume，所以...swap.img 造成建 snapshot 出錯...[點我跳到解決方法](#把-swap-檔案移到沒有快照的-subvolume)

雖然我的工作筆電 RAM 有 32G，但平常工作要開發網頁系統前、後端，必開瀏覽器、2 個 Antigravity IDE、LibreOffice 以及常駐通訊軟體 Discord、Bottle + LINE，隨便就能吃超過 16G。

這裡面最恐怖的是瀏覽器，根本吃 RAM 怪獸...不管 Zen Browser 還是 Helium，開啟來後慢慢的從 8XX MB 吃到 2、3G，多幾個活動中分頁還直奔 5G 多...

必定要開的 IDE 加上連帶要執行的 LSP Server、Dev Tools，他們一開起來也是吃了 1、2G，AI agent 再 RUN 起來也會吃 RAM；工作要發布站台，最省事是透過綁死 Windows 的 WebDeploy 三步驟無腦發布流程，這時候就要開 Winboat 進 Windows 開 VSCode 發布，而專屬 Windows 的 WinForm 程式再開 Visual Studio 來發布...同樣地，VSCode、VS 都是一開就要吃 1、2G 的 RAM。

上星期某天同時要開發需求、緊急改 BUG 並發布，沒注意到 RAM 的用量，等到感覺操作卡頓時 RAM 已經用了 97%，再回覆個話就來不及了~~電腦硬碟指示燈已經一直在亮，滑鼠及鍵盤都沒反應了...這時候才意識到，RAM 用爆了，而且我沒建 swap 分區...還好資料都儲存了，按了電源鍵等個 10 幾秒，Linux 開始 Kill Process 然後關機了...不像 Windows 都沒反應只能強迫斷電。

本來想說 32G 這麼大不用 swap 分區，但發生了這事有空就開始研究是否必要建立 swap 分區以及要用 zswap、zram 哪種技術。綜合找到的資料及詢問 AI 的回覆，最後我選擇不建 swap 分區，用像 Windows 使用的交換檔，並且用 zswap，原因是「看起來有點難」：直接挑戰大魔王，萬一失敗了再做比較簡單設置的 zram。另外，也是因為不想再重裝 Linux 或縮小分區所以採用建立 swap 檔，而且還有一個好處：真要修改的話，檔案絕對比分割區方便。

## 確認有設定 swap.img 及啟用 zswap 

安裝前確認都沒有用避免衝突；設置 swap image 及 zswap 成功後也需要再確認。

> 例如：CachyOS 預設啟用 zram，而 zram 雖然不使用 swap 但也會標記有使用 swap，且大小等於 RAM 的大小。

### 1. 確認 swap.img 是否有成功設定並啟用
```bash
swapon --show
```

有啟用：
```Plaintext
NAME       TYPE SIZE USED PRIO
/swap.img  file   4G   0B   -2
```

沒有啟用： 什麼都沒有顯示（完全空白），代表目前系統沒有啟用任何 Swap 空間。


### 2. 確認 zswap 是否已經啟用
```bash
cat /sys/module/zswap/parameters/enabled
```

結果：
- 顯示 Y：代表 zswap 已經成功啟用。
- 顯示 N：代表 zswap 處於關閉狀態。


如果啟用了 zswap（顯示 Y），可以輸入這行指令檢查壓縮狀態：
```bash
sudo grep -r . /sys/kernel/debug/zswap/ 2>/dev/null
```

輸出結果：
```Plaintext
/sys/kernel/debug/zswap/stored_incompressible_pages:0
/sys/kernel/debug/zswap/stored_pages:0
/sys/kernel/debug/zswap/pool_total_size:0
/sys/kernel/debug/zswap/written_back_pages:0
/sys/kernel/debug/zswap/decompress_fail:0
/sys/kernel/debug/zswap/reject_compress_poor:0
/sys/kernel/debug/zswap/reject_compress_fail:0
/sys/kernel/debug/zswap/reject_kmemcache_fail:0
/sys/kernel/debug/zswap/reject_alloc_fail:0
/sys/kernel/debug/zswap/reject_reclaim_fail:0
/sys/kernel/debug/zswap/pool_limit_hit:0
```

其中：
- stored_pages：目前壓縮後儲存在記憶體裡的頁面數量。如果數值大於 0，代表 zswap 真的有在運作。
- pool_limit_hit：如果這個數值一直增加，代表 zswap 快取容量滿了，資料開始轉去硬碟的 swap.img。


## BTRFS 檔案系統設定休眠及建立 swap file 的方法

### 步驟一：建立 BTRFS 專用的 swap file

我的 RAM 有 32G，加點冗餘量避免意外發生。
（純被蛇咬經驗...32*1024^3 超過 34G，加上電腦數字我偏好偶數，故取 36G）。

1. 用 BTRFS 指令 mkswapfile 建立一個 36G 的 swap.img，它會自動處理好 Btrfs 所有的特殊限制（關閉 CoW、不壓縮、完全連續的磁碟空間）。

```bash
sudo btrfs filesystem mkswapfile --size 36G /btrfs-swap/swap.img
```

啟用 swap file：
```bash
sudo swapon /swap.img
```

輸入 `swapon --show`，應該就能順利看到 36GB 的 /swap.img 了。

### 步驟二：設定開機自動掛載

在 /etc/fstab 檔案最底部加入這行：
```bash
swap.img  none  swap  defaults  0  0
```
可以用下面指令一次完成
```bash
printf '/swap.img  none  swap  defaults  0  0' | sudo tee -a /etc/fstab
```

### 步驟三：調整使用 swap 的時機

#### 以下引用自 [Ivon的部落格](https://ivonblog.com/posts/linux-zswap-and-zram/)
vm.swappiness 這個數值可以定義 Linux 核心什麼時候要使用到 swap 檔案，避免系統明明還有 RAM 能用卻把程式放到 swap 檔案導致系統變慢。數字越小，Linux 核心越不會去使用 swap。若設定為 0，就只有防止 OOM 的時候才會使用 swap。

編輯 sysctl 設定
```bash
sudo nano /etc/sysctl.conf
```

設定 swappiness 數值（重開機後才會套用變更，在此先不重開機）
```bash
vm.swappiness=10
```

### 步驟四：配置核心參數（取得 BTRFS 專屬 Offset）

#### 1. 取得根目錄（/）硬碟 UUID

> 如果不需要休眠可以略過此步驟。

輸入以下指令：
```bash
findmnt -no UUID /
```
輸出結果可能像這樣：abcdef12-3456-7890-abcd-ef1234567890

#### 2. 取得 BTRFS 專屬的 Offset

> 如果不需要休眠可以略過此步驟。

輸入以下指令：
```bash
sudo btrfs inspect-internal map-swapfile -r /swap.img
```
輸出結果可能像這樣的數字：15869184

#### 3. 修改 GRUB 核心參數

開啟 GRUB 設定檔：
```bash
sudo nano /etc/default/grub
```

找到 GRUB_CMDLINE_LINUX_DEFAULT="..."，加入 `zswap.enabled=1 zswap.max_pool_percent=60 zswap.zpool=zsmalloc` 啟用 zswap。

> zswap.enabled：啟用 zswap。
> zswap.max_pool_percent：設定為60，偏好將分頁留在RAM，而非移動到SWAP檔案（數字越大越不會使用到SWAP檔案）。
> zswap.zpool：分配器使用 zsmalloc 取得最好壓縮比。
> 　　　— 參考 [Ivon的部落格](https://ivonblog.com/posts/linux-zswap-and-zram/)



加入剛才查到的 UUID 及 Offset 啟用「休眠」：

> 如果不需要休眠可以略過此步驟。
```plaintext
resume=UUID=你的UUID resume_offset=你的BTRFSOffset"
```

儲存離開後，更新 GRUB 設定（以下兩個 sudo 指令擇一執行）：
```bash
sudo update-grub 
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### 步驟五：配置 initramfs 鉤子 (Hook)

> 如果不需要休眠可以略過此步驟。

確保系統開機時載入休眠恢復模組。
```bash
sudo nano /etc/mkinitcpio.conf
```

找到 HOOKS=(...) 那一行，在 filesystems 之前加上 resume：
```plaintext
HOOKS=(base udev autodetect modconf kms keyboard keymap console block resume filesystems fsck)
```

儲存離開後，重新生成 initramfs：
```bash
sudo mkinitcpio -P
```

### 🎉 重啟與測試

重新啟動電腦使設置生效：
```bash
sudo reboot
```
> 如果不需要休眠可以略過此步驟。

開機後，驗證休眠：
```bash
loginctl hibernate
```

## 把 swap 檔案移到沒有快照的 subvolume

我居然把 /home 也放到 BTRFS subvolume！！放是建 snapshot 時報錯了...BTRFS 禁止對 swap file 建 snapshot。好吧...

要把 /home 移出 BTRFS subvolumes 感覺有點恐怖，所以先從最簡單的作法：把 swap file 放到新的子卷，新子卷沒設定要執行 snapshot 就不會報錯了。

```bash
sudo swapoff /swap.img
sudo rm /swap.img
```

> 上面文章是用「swap.img」，依實際的檔名替換

接下來就可以開始建立新 subvolume 及 swap file。

### 步驟一：掛載 root 分區以建立新 subvolume

我的 root 「/」掛在 /dev/nvme1n1p5，如果不知道實際的代號可以用 `lsblk -f` 查詢。

```bash
sudo mkdir -p /mnt/btrfs-root
sudo mount /dev/nvme1n1p5 /mnt/btrfs-root -o subvol=/
```

建立一個專門給 Swap 用的 subvolume，名稱為 @swap。

```bash
sudo btrfs subvolume create /mnt/btrfs-root/@swap
```

建立完就可以卸載 root 分區了

```bash
sudo umount /mnt/btrfs-root
```

### 步驟二：掛載新 subvolume 並建立 swap file

```bash
sudo mkdir -p /btrfs-swap
sudo mount /dev/nvme1n1p5 /btrfs-swap -o subvol=@swap,nodatacow,noatime

sudo btrfs filesystem mkswapfile --size 36G /btrfs-swap/swap.img
sudo swapon /btrfs-swap/swap.img
```

### 步驟三：開機自動掛載新 subvolume

```bash
sudo nano /etc/fstab
```

先刪掉原本最下方的

```plaintext
/swap.img  none  swap  defaults  0  0
```

再加入

```plaintext
UUID=你的UUID  /btrfs-swap  btrfs  subvol=@swap,nodatacow,noatime  0  0
/btrfs-swap/swap.image                     none        swap   defaults                        0  0
```

> 用 `lsblk -f` 可以查到 UUID。

存檔離開。

再次建 snapshot 就會成功了！
```bash
sudo snapper create -d "Bug fixed snapshot"
```

> 得找個長時間把 /home 移除 subvolume，順便規劃快照 ~/.config，或...改過的設定放到有快照的 subvolume 再 ln -s 回 .config？也許也要放到 github...
