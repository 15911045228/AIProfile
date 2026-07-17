---
tags: [concept, windows]
status: active
created: 2026-07-14
updated: 2026-07-14
sources:
  - raw/notes/2026-07-14-windows-c-disk-cleanup.md
---

# Windows C 盘空间清理（满盘排查 + 安全清理 + 迁移）

## 概述

在 Windows 小容量系统盘（本机 C 盘仅 80GB）被占满（100%、几 MB 可用）时，如何排查空间去向、执行**零/低副作用的安全清理**，以及真正能腾出空间的**数据迁移**思路。核心认知：**C 盘满盘的主因几乎从来不是"垃圾"，而是 AppData 里的应用数据和 Windows 自身。**

## 核心要点

- **工具坑（高频）：** 在 Git Bash 里执行 `powershell -Command "..."` 时，`$_`、`$t` 等 PowerShell 变量会被 bash 吞掉，导致语法报错。**必须**把所有逻辑写进 `.ps1` 文件，用 `powershell.exe -File xxx.ps1` 运行。
- **排查三步：** ① `df -h`（或 `Get-Volume`）看各盘占用与空闲；② 顶层 `Get-ChildItem -Directory` 定位大目录；③ 递归 `Get-ChildItem -Recurse -File | Measure-Object -Property Length -Sum` 算子目录大小（慢但准）。
- **关键认知：** 程序二进制通常只占 ~15GB，真正的大头是 `C:\Users\xxx\AppData`（本机 37GB：Tencent 7G / kingsoft 5G / DingTalk 多版本 2.5G）。常规 Temp / 浏览器缓存加起来 < 1GB，**只清垃圾箱几乎没用。**
- **安全清理清单（可删，零/低副作用）：**
  - 各软件 `update\down` 下的**旧版安装包**（如 WPS 的 `setup_*_Unified.exe`，保留最新一个）
  - 各 `*updater` 目录里的残留 `installer.exe`（更新时会重新下载）
  - NVIDIA 着色器缓存 `DXCache` / `GLCache`（**保留 `NvBackend`**）
  - 开发缓存 `.gradle`、`npm-cache`（删后首次构建变慢，但自动重建）
  - 回收站、系统「磁盘清理（含系统文件）」清 WinSxS / 更新缓存
  - 关闭休眠 `powercfg -h off` 释放 `hiberfil.sys`（约 = 内存大小）
- **绝对勿动：** `C:\Windows` 目录、微信/Tencent 聊天数据本体、正在运行的 Claude 目录、任何软件的程序本体（只删缓存/残留，不删软件）。
- **迁移才是大头解法：** 游戏（Steam/Epic/安卓模拟器）大多已在别的盘；真正能腾 C 盘的是 AppData 数据。最安全做法是用微信自带「设置 → 文件管理 → 更改」改数据路径；激进做法是用 `mklink /J` 把大 AppData 子目录（Tencent/kingsoft/DingTalk）整体移到 D/E/G 再建目录联接（需先关对应软件，谨慎操作）。

## 详细内容

### 排查命令模板（写进 .ps1 后 `powershell.exe -File` 运行）

```powershell
# 各盘空闲
Get-Volume | Where-Object { $_.DriveLetter } | ForEach-Object {
  "{0}: {1} GB free / {2} GB total" -f $_.DriveLetter,
    [math]::Round($_.SizeRemaining/1GB,1), [math]::Round($_.Size/1GB,1)
}
# 某目录大小
$s = (Get-ChildItem -Path $dir -Recurse -File -ErrorAction SilentlyContinue |
      Measure-Object -Property Length -Sum).Sum
```

### 本机实战数据（2026-07-14）

- 清理前：C 盘 80G / 79.1G 用 / **0 可用**；其他盘 D 24.9G、E 66.1G、G 65.1G 可用，**F 盘已满**。
- 安全清理释放 **~3.8GB**（WPS 旧安装包 1.37G + 4 个 updater 残留 1.43G + NVIDIA GLCache 0.03G + .gradle 0.93G + npm-cache 0.08G），C 盘恢复到 **3.9GB 可用**。
- NVIDIA `DXCache` 一个文件被显卡驱动占用未能删除（~640MB），重启后或系统磁盘清理可清。
- 顶层占用：Users 42.4G（AppData 37G）、Windows 18.9G、ProgFiles(x86) 7.4G、ProgFiles 3.0G。

## 关联页面

- 暂无（可作为"Windows 系统运维"类条目的起点）

## 外部链接

- 无
