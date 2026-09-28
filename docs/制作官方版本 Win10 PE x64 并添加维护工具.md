
---

## 环境准备

- 操作系统：Windows 10/11 x64
- 下载并安装 [Windows ADK](https://go.microsoft.com/fwlink/?linkid=2337875)
- 下载并安装 [WinPE 附加组件](https://go.microsoft.com/fwlink/?linkid=2337886)

---

## 制作映像

> 以管理员身份运行“部署和映像工具环境”中执行以下命令。



### 1. 创建并挂载映像

#### 1.1 创建目录并导入映像

```
copype amd64 C:\Win10PE_x64
```



#### 1.2 挂载映像

```
dism /Mount-Wim /WimFile:C:\Win10PE_x64\media\sources\boot.wim /Index:1 /MountDir:C:\Win10PE_x64\mnt
```



### 2. 修改映像语言

#### 2.1 添加简体中文字体支持包

```
dism /Image:"C:\Win10PE_x64\mnt" /Add-Package /PackagePath:"%WinPERoot%\amd64\WinPE_OCs\WinPE-FontSupport-ZH-CN.cab"
```



#### 2.2 加载PE注册表单元

```
reg load HKLM\PE_SYS "C:\Win10PE_x64\mnt\Windows\System32\config\SYSTEM"

reg load HKLM\PE_SOFTWARE "C:\Win10PE_x64\mnt\Windows\System32\config\SOFTWARE"
```



#### 2.3 设置默认语言为中文

```
reg add "HKLM\PE_SYS\ControlSet001\Control\Nls\Language" /v Default         /t REG_SZ /d "0804"     /f

reg add "HKLM\PE_SYS\ControlSet001\Control\Nls\Language" /v InstallLanguage /t REG_SZ /d "0804"     /f

reg add "HKLM\PE_SYS\ControlSet001\Control\Nls\Locale"   /ve                /t REG_SZ /d "00000804" /f

reg add "HKLM\PE_SOFTWARE\Microsoft\Windows NT\CurrentVersion\Console\TrueTypeFont" /v "0936"  /t REG_SZ /d "*新宋体"           /f

reg add "HKLM\PE_SOFTWARE\Microsoft\Windows NT\CurrentVersion\Console\TrueTypeFont" /v "00936" /t REG_SZ /d "Microsoft YaHei Mono" /f
```



#### 2.4 卸载注册表单元

```
reg unload HKLM\PE_SYS

reg unload HKLM\PE_SOFTWARE
```



### 3. 加入维护工具并添加启动脚本

#### 3.1 复制工具目录

```
xcopy ".\Tools" "C:\Win10PE_x64\mnt\" /E /I /Y /Q
```



#### 3.2 添加启动项

```
del /f /q "C:\Win10PE_x64\mnt\Windows\System32\startnet.cmd"

(echo @echo off& echo wpeinit& echo call ^"^%SystemDrive^%\Tools\Menu.cmd^") > "C:\Win10PE_x64\mnt\Windows\System32\startnet.cmd"
```



#### 3.3 检查常见依赖

```
if not exist "C:\Win10PE_x64\mnt\Windows\System32\oledlg.dll"    copy /Y "%SystemRoot%\System32\oledlg.dll"    "C:\Win10PE_x64\mnt\Windows\System32\oledlg.dll"

if not exist "C:\Win10PE_x64\mnt\Windows\System32\winhttp.dll"   copy /Y "%SystemRoot%\System32\winhttp.dll"   "C:\Win10PE_x64\mnt\Windows\System32\winhttp.dll"

if not exist "C:\Win10PE_x64\mnt\Windows\System32\hid.dll"       copy /Y "%SystemRoot%\System32\hid.dll"       "C:\Win10PE_x64\mnt\Windows\System32\hid.dll"

if not exist "C:\Win10PE_x64\mnt\Windows\System32\shfolder.dll"  copy /Y "%SystemRoot%\System32\shfolder.dll"  "C:\Win10PE_x64\mnt\Windows\System32\shfolder.dll"

if not exist "C:\Win10PE_x64\mnt\Windows\System32\oleacc.dll"    copy /Y "%SystemRoot%\System32\oleacc.dll"    "C:\Win10PE_x64\mnt\Windows\System32\oleacc.dll"

if not exist "C:\Win10PE_x64\mnt\Windows\System32\oleaccrc.dll"  copy /Y "%SystemRoot%\System32\oleaccrc.dll"  "C:\Win10PE_x64\mnt\Windows\System32\oleaccrc.dll"

if not exist "C:\Win10PE_x64\mnt\Windows\System32\winscard.dll"  copy /Y "%SystemRoot%\System32\winscard.dll"  "C:\Win10PE_x64\mnt\Windows\System32\winscard.dll"

if not exist "C:\Win10PE_x64\mnt\Windows\System32\msimg32.dll"   copy /Y "%SystemRoot%\System32\msimg32.dll"   "C:\Win10PE_x64\mnt\Windows\System32\msimg32.dll"
```



### 4. 保存修改，生成ISO

#### 4.1 卸载并保存映像

```
dism /Unmount-Wim /MountDir:C:\Win10PE_x64\mnt /Commit
```



> 如遇文件占用导致卸载失败，可以先直接构建ISO文件，重启后再通过以下命令强制卸载。

```
// 强制卸载，如果没有报错，无需执行以下命令。
dism /Unmount-Wim /MountDir:C:\Win10PE_x64\mnt /Discard
dism /Cleanup-Wim


//强制卸载后检查是否卸载成功
dism /Get-MountedWimInfo
```



#### 4.2 构建 ISO 文件

```
Makewinpemedia /iso C:\Win10PE_x64 C:\Win10PE_x64\Win10PE_x64.iso
```



---

## 工具菜单

### X:\Tools\Menu.cmd
```
@echo off
cd /d "%~dp0"
chcp 936 >nul 2>&1
title 操作菜单（Win 10 PE x64）

rem ================== 配置区 ==================

set "APP1=%SystemDrive%\Tools\DiskGenius.exe"
set "NAME1=DiskGenius 分区工具"

set "APP2=%SystemDrive%\Tools\WinNTSetup\WinNTSetup.exe"
set "NAME2=WinNTSetup 系统安装工具"

set "APP3=%SystemDrive%\Tools\CPU-Z.exe"
set "NAME3=CPU-Z 处理器检测"

set "APP4=%SystemDrive%\Tools\CrystalDiskInfo\DiskInfo.exe"
set "NAME4=DiskInfo 硬盘检测"

set "APP5=%SystemDrive%\Tools\CrystalDiskMark\DiskMark.exe"
set "NAME5=DiskMark 硬盘跑分"

set "APP6=%SystemDrive%\Tools\DISM++\Dism++x64.exe"
set "NAME6=DISM++ 映像处理工具"

set "APP7=%SystemDrive%\Tools\MemTest64.exe"
set "NAME7=MemTest 内存测试"

set "APP8=%SystemDrive%\Tools\NTPWED07\ntpwedit64.exe"
set "NAME8=NTPWED07 密码重置工具"

rem ================== 配置区结束 ==================

:MainMenu
cls
echo.
echo   ================================================================
echo                        操作菜单（Win 10 PE x64）
echo   ================================================================
echo.
echo     [1]  %NAME1%
echo     [2]  %NAME2%
echo     [3]  %NAME3%
echo     [4]  %NAME4%
echo     [5]  %NAME5%
echo     [6]  %NAME6%
echo     [7]  %NAME7%
echo     [8]  %NAME8%
echo.
echo     [R]  重新启动计算机
echo     [P]  关闭计算机
echo.
echo   ================================================================
echo.

set "KEY="
set /p "KEY=   请输入选项 [1-8]、[R/P] 后按回车: "
goto AfterInput

:AfterInput
if not defined KEY goto MainMenu

set "ap="
if "%KEY%"=="1" set "ap=%APP1%"
if "%KEY%"=="2" set "ap=%APP2%"
if "%KEY%"=="3" set "ap=%APP3%"
if "%KEY%"=="4" set "ap=%APP4%"
if "%KEY%"=="5" set "ap=%APP5%"
if "%KEY%"=="6" set "ap=%APP6%"
if "%KEY%"=="7" set "ap=%APP7%"
if "%KEY%"=="8" set "ap=%APP8%"
if "%KEY%"=="r" goto DoReboot
if "%KEY%"=="p" goto DoShutdown

if not defined ap goto MainMenu

echo.
echo    正在启动：%ap%
echo.
start "" "%ap%"
goto MainMenu


:DoReboot
cls
echo.
echo    正在重新启动计算机...
echo.
wpeutil reboot
pause >nul
goto MainMenu


:DoShutdown
cls
echo.
echo    正在关闭计算机...
echo.
wpeutil shutdown
shutdown /s /t 0 /f
goto MainMenu
```



---

## 第三方软件


**DiskGenius**，[https://www.diskgenius.com/](https://www.diskgenius.com/)

**WinNTSetup**，[https://msfn.org/board/topic/149612-winntsetup-v524/](https://msfn.org/board/topic/149612-winntsetup-v524/)

**CPU-Z**，[https://www.cpuid.com/softwares/cpu-z.html](https://www.cpuid.com/softwares/cpu-z.html)

**CrystalDiskMark**，[https://crystalmark.info/en/software/crystaldiskmark/](https://crystalmark.info/en/software/crystaldiskmark/)

**CrystalDiskInfo**，[https://crystalmark.info/en/software/crystaldiskinfo/](https://crystalmark.info/en/software/crystaldiskinfo/)

**DISM++**，[https://github.com/Chuyu-Team/Dism-Multi-language](https://github.com/Chuyu-Team/Dism-Multi-language)

**NTPWEdit**，[http://www.cdslow.org.ru/en/ntpwedit/](http://www.cdslow.org.ru/en/ntpwedit/)

**Explorer++**，[https://explorerplusplus.com/](https://explorerplusplus.com/)