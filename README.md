# list_apps

在已 root 的 Android 設備上，列出所有已安裝應用的**包名**與**應用名稱**，並保存為 txt。

應用名稱透過 `aapt dump badging` 從 APK 讀取，優先取繁體中文，其次簡體中文、中文，最後是預設名稱。讀不到時退回顯示包名。

## 環境需求

- Android 設備，已 root（需要 `su`）
- [Termux](https://github.com/termux/termux-app)
- `aapt`：`pkg install aapt`
- 儲存權限：`termux-setup-storage`

## 用法

```sh
sh list_apps.sh                        # 全部應用（含系統）
sh list_apps.sh -3                     # 只列第三方
sh list_apps.sh -3 -e                  # 只列已啟用的第三方
sh list_apps.sh -o /sdcard/apps.txt    # 指定輸出檔案
sh list_apps.sh -o ~/out/              # 指定輸出目錄（以 / 結尾，或已存在的目錄）
sh list_apps.sh -q                     # 不顯示進度
```

| 旗標 | 說明 |
|------|------|
| `-3` | 只列第三方應用 |
| `-s` | 只列系統應用 |
| `-e` | 只列已啟用的 |
| `-d` | 只列已停用的 |
| `-u` | 含已卸載但保留資料的 |
| `-o 路徑` | 指定輸出檔案或目錄 |
| `-q` | 安靜模式，不顯示進度 |

旗標可組合使用。

> 腳本放在 `/storage/emulated/0/` 下無法直接執行（sdcard 為 noexec），請用 `sh` 呼叫，或複製到 Termux 家目錄後 `chmod +x`。

## 輸出

執行時畫面會在同一行顯示進度（`[123/751] 包名`），跑完後一次印出結果，同時保存到檔案。預設位置：

```
/storage/emulated/0/TM/apps_YYYYMMDD_HHMMSS.txt
```

用 `-o` 指定檔案時，若該檔案已存在會被覆蓋。進度只輸出到螢幕，不會混進檔案。

每行一個應用，**Tab 分隔**，依包名排序：

```
com.android.chrome	Chrome
org.telegram.messenger	Telegram
```

用 Tab 而不是空格對齊，是因為中文名稱在固定寬度下會錯位。可直接貼進試算表，或用 `column -t -s "$(printf '\t')" 檔案.txt` 預覽。

## 運作方式

腳本會生成一個臨時腳本並以 root 執行，一次提權完成全部工作：用 `cmd package list packages -f` 取得每個應用的 APK 路徑，再以 root 直接讀取原 APK 取名稱，不複製檔案。臨時腳本在結束時自動刪除。輸出路徑只在 Termux 這層使用，不會進入 root 的命令字串。

## 已測試環境

| 項目 | 內容 |
|------|------|
| 設備 | OnePlus PJD110 |
| 系統 | Android 16（API 36），ColorOS / OxygenOS |
| root | KernelSU（LKM 模式） |
| Termux | GitHub 版 |
| 結果 | 751 個應用，全量約 1 分鐘；103 個（約 13.7%）顯示包名 |

Magisk、APatch 等其他 root 方案理論上相容，但未驗證，歡迎回報。

## 已知限制

- 僅支援 Termux + root，其他環境未測試。
- 不支援 `-i`（輸出多出 `installer=` 欄位會破壞解析）。
- 全量執行需要對每個 APK 呼叫一次 `aapt`，約 750 個應用耗時 1 分鐘左右；因為要排序，結果在跑完後才一次印出，期間只顯示進度。
- 進度行使用 ANSI 控制碼刷新，個別終端可能顯示異常，可用 `-q` 關閉。
- 約一成多的條目會顯示包名而非名稱，主要是資源覆蓋層（`*.overlay.*`、`*_rro`）、無桌面圖示的系統背景服務，以及少數 label 無法被 `aapt` 解析的 APK。
- 若被 `kill -9` 強制結束，`/data/data/com.termux/files/usr/tmp` 可能殘留 `.gen_apps_*.sh`，可手動刪除。

## 授權

MIT
