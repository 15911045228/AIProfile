# Windows C 盘满盘清理 — 原始笔记（2026-07-14 实战）

## 背景

本机 C 盘 80GB，100% 满（仅 3.4MB 可用），随时可能卡死/无法更新。用户在 Claude Code 中要求排查可清理项，并迁移可移动的软件/游戏到别的盘。

## 盘况（Get-Volume）

- C: 79.1G 总 / 79.1G 用 / 0 可用（清理后恢复到 3.9G 可用）
- D: 311G 总 / 24.9G 可用
- E: 311G 总 / 66.1G 可用
- F: 309.5G 总 / 0 可用（已满，勿往 F 挪）
- G: 158.2G 总 / 65.1G 可用

## 顶层占用（Get-ChildItem 递归 Measure-Object Length -Sum）

- C:\Users 42.4G（其中 AppData 37G）
- C:\Windows 18.9G
- C:\Program Files (x86) 7.4G
- C:\Program Files 3.0G
- 其余（Android、360Safe、Steam 等）极小

## AppData 大头（37G）

- Roaming\Tencent 7.0G（微信/腾讯会议/WeGame 数据）
- Roaming\kingsoft 5.1G（WPS，含 update\down 旧安装包）
- Roaming\LarkShell 1.47G / DingTalk 1.38G / duowan 1.23G
- Local\Microsoft 1.64G（含 FileHistory Catalog ~0.9G）
- Local\Programs 1.25G / AnthropicClaude 1.13G
- DingTalk_108+_91+_133+DingTalk 多版本合计 ~2.5G
- NVIDIA 0.98G（DXCache 644M / GLCache 25M / NvBackend 331M）
- .gradle 0.93G（用户根目录，非 AppData 内）

常规 Temp / INetCache / Edge 缓存加起来 < 1G —— 清垃圾箱没用。

## 工具坑（重要）

在 Git Bash 里执行 `powershell -Command "..."` 时，`$_` / `$t` / `$s` 等 `$` 变量会被 bash 吞掉（变成 `extglob.xxx` 或空），导致 PowerShell 语法报错。
**解决：把所有 PowerShell 逻辑写进 .ps1 文件，用 `powershell.exe -File xxx.ps1` 运行。**

## 执行的安全清理（共释放 ~3.8G，C 恢复到 3.9G 可用）

- WPS 旧版更新安装包（kingsoft\office6\update\down 下 5 个旧 setup*.exe，保留最新 26895）：-1.37G
- 4 个 updater 残留安装包（@genieworkbuddy / canva / coze / obsidian-updater）：-1.43G
- NVIDIA GLCache：-0.03G（DXCache 一个文件被显卡驱动占用未删，~640MB，重启后可清）
- .gradle：-0.93G
- npm-cache：-0.08G

未动：微信/Tencent 聊天数据、Claude 目录、任何软件的程序本体。
ms-playwright(0.42G) / OpenAI Codex(0.39G) 因"仅不使用时删"约定跳过，待确认。

## 游戏/软件迁移排查

- Steam / Epic / Nox(Mumu) 安卓模拟器均不在 C 盘；C 盘游戏相关数据仅 ~2.5G（WeGame 数据 622M + duowan 平台 1.26G + 几个 LocalLow 游戏存档）。
- 结论：游戏已在别的盘，"搬游戏"收益极小。
- 真正能腾 C 盘的是 AppData 数据（Tencent 7G / kingsoft 5G / DingTalk 2.5G）。
- 迁移方法：① 微信自带"设置→文件管理→更改"改数据路径（最安全，收益最大）；② `mklink /J` 把大 AppData 子目录迁到 D/E/G 再建软链（收益最大但需先关软件、谨慎）；③ 重装大程序到 D（程序本体都<2G，性价比低）。

## 其他可选安全回收（待确认）

- 系统"磁盘清理（含系统文件）"：清 WinSxS / 更新缓存，常再省数 G。
- 关闭休眠 `powercfg -h off`：释放 hiberfil.sys（约=内存大小，可能 8–16G），但禁用休眠/快速启动。
