---
title: "Windows Desktop Codex 显示 max 推理档位的修复"
date: "2026-08-13"
category: "troubleshooting"
tags: ["codex", "windows", "reasoning", "configuration"]
---

# Windows Desktop Codex 显示 `max` 推理档位的修复记录

## 现象

Windows Desktop 版 Codex 的模型推理 effort 下拉框最高只有 `xhigh`，没有后端已经返回的 `max`。

本机 Windows Desktop 版本：

```text
26.803.10989.0
```

## 根因

后端模型能力返回了：

```text
low, medium, high, xhigh, max
```

但 Desktop 前端默认的隐藏 allowlist 是：

```text
low, medium, high, xhigh, ultra
```

前端会用 `enabled-reasoning-efforts` 过滤后端返回值，所以 `max` 被过滤掉了。

`max` 不应该映射成 `ultra`：

- `max`：最大推理深度；
- `ultra`：最大推理深度，并可能涉及自动任务委派等 Desktop 行为。

## 正确配置位置

Windows Desktop 使用的 Codex Home 是：

```text
C:\Users\ruijie\.codex
```

对应 WSL 路径：

```text
/mnt/c/Users/ruijie/.codex
```

配置文件：

```text
C:\Users\ruijie\.codex\config.toml
```

注意：该设置不是 WSL CLI 的 `/home/ruijie/.codex/config.toml`，也不需要修改 `app.asar`。

## 修复方法

在 Windows 配置文件的 `[desktop]` 段加入：

```toml
[desktop]
enabled-reasoning-efforts = ["low", "medium", "high", "xhigh", "max", "ultra"]
```

如果 `[desktop]` 段已经存在，只添加这一行，不要重复创建段落。

手动编辑或使用脚本时，必须把这一行放在 `[desktop]` 段内。不要直接追加到文件末尾，因为文件后面可能已经进入了其他 TOML 段。

PowerShell 写法：

```powershell
$config = "$env:USERPROFILE\.codex\config.toml"
$text = Get-Content $config -Raw
$line = 'enabled-reasoning-efforts = ["low", "medium", "high", "xhigh", "max", "ultra"]'
$text = $text -replace '(?m)^(\[desktop\]\r?\n)', "`$1$line`r`n"
[IO.File]::WriteAllText($config, $text)
```

更安全的做法是先备份：

```powershell
Copy-Item $config "$config.bak"
```

## 使配置生效

1. 完全退出 Codex Desktop，包括系统托盘中的 Codex 进程；
2. 重新启动 Codex Desktop；
3. 打开模型推理 effort 下拉框，确认出现 `max`；
4. 新建或继续任务，确认实际请求使用的是 `max`，而不是仅修改了显示名称。

## 恢复方法

如果需要恢复备份：

```powershell
Copy-Item "$env:USERPROFILE\.codex\config.toml.bak" "$env:USERPROFILE\.codex\config.toml" -Force
```

## 本机本次修改

本次实际加入的配置为：

```toml
enabled-reasoning-efforts = ["low", "medium", "high", "xhigh", "max", "ultra"]
```

原配置备份为：

```text
C:\Users\ruijie\.codex\config.toml.backup-max-fix-20260813-1535
```
