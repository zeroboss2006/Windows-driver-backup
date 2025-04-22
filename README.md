# 📦 Windows 驅動程式備份與還原教學（含各版本腳本）

整合多篇來源教學，提供 **Windows 7 / 8 / 10 / 11** 各版本適用的驅動程式備份與還原方式，包含：

- 一般裝置驅動（通用）
- 印表機驅動（進階）
- 批次腳本 / DISM / GUI 工具

---

## 🧩 支援版本概覽

| 作業系統            | 備份方法                       | 備註                         |
| --------------- | -------------------------- | -------------------------- |
| Windows 7       | 批次檔 + `pnputil`            | 無 DISM `/export-driver` 功能 |
| Windows 8/10/11 | `DISM` 工具                  | 建議使用                       |
| 所有版本            | 第三方工具（如 Double Driver）     | 適用於圖形介面使用者                 |
| Server 系統       | `Print Management` 匯出印表機驅動 | 適用於 Server / AD 環境         |

---

## 🛠 一般驅動程式備份與還原

### ✅ 方法一：DISM（Windows 8 以上）

```cmd
dism /online /export-driver /destination:"D:\DriversBackup"
```

- 匯出所有已安裝驅動到 `D:\DriversBackup`

#### 還原：

```cmd
pnputil /add-driver D:\DriversBackup\*.inf /subdirs /install
```

---

### ✅ 方法二：Windows 7 批次檔腳本

> 建議以系統管理員身份執行 `backup_drivers.bat`

```bat
@echo off
setlocal EnableDelayedExpansion
set BACKUP_DIR=C:\DriversBackup

echo.
echo 🚀 備份驅動程式到 %BACKUP_DIR%
echo.

if not exist "%BACKUP_DIR%" (
    mkdir "%BACKUP_DIR%"
)

pnputil -e > "%BACKUP_DIR%\driverlist.txt"

for /f "tokens=2 delims=:" %%A in ('findstr /i "Published Name" "%BACKUP_DIR%\driverlist.txt"') do (
    set infname=%%A
    set infname=!infname:~1!

    for /d %%D in (%SystemRoot%\System32\DriverStore\FileRepository\*!infname:~0,-4!*.*) do (
        echo 備份：%%D
        xcopy "%%D" "%BACKUP_DIR%\!infname!" /E /I /Y >nul
    )
)

echo.
echo ✅ 備份完成，請查看：%BACKUP_DIR%
pause
```

#### 還原：

```cmd
pnputil /add-driver C:\DriversBackup\*.inf /subdirs /install
```

---

## 📨 印表機驅動備份與還原（進階）

### 適用系統：

- Windows 10/11 Pro
- Windows Server 系列
- 安裝「Print Management Console」

### 📆 備份步驟：

1. 開啟「Print Management」
```
printmanagement.msc
```
2. 展開 `Print Servers > 本機名稱`
3. 右鍵「Printers」 > 選擇「Export Printers to a file」
4. 儲存 `.printerExport` 檔案

### ♻️ 還原步驟：

1. 回到 Print Management
2. 點選「Import Printers from a file」
3. 選取 `.printerExport` 匯入

---

## 💡 小提示

- `pnputil` 自 Vista 起支援，但各版參數功能略有差異
- Server 可搭配 `PrintMig.exe` 或 PowerShell 進行進階備份
- 如需備份 `DriverStore` 完整資料夾，可手動複製 `%SystemRoot%\System32\DriverStore\FileRepository`

---

## 🔗 參考來源

- [Top Password - Backup and Restore Device Drivers](https://www.top-password.com/blog/backup-and-restore-device-drivers-in-windows/)
- [Seagull Scientific - Backup / Import Printer Drivers](https://support.seagullscientific.com/hc/en-us/articles/360008934334-How-to-Backup-Import-Printer-Drivers-through-Windows-Print-Management)
- [SuperUser - Export Drivers Windows 7](https://superuser.com/questions/1196061/how-to-export-installed-device-driver-on-windows-7-for-later-use)

