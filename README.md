# GitHub Streak Automation

[简体中文](README.md) | [English](README.en.md)

让 GitHub Actions 每天自动更新仓库里的日志文件并推送一次提交，无需在自己的电脑上运行定时任务。

[![Daily workflow](https://github.com/Sentryea/Github-Streak-Automation/actions/workflows/daily-commit.yml/badge.svg)](https://github.com/Sentryea/Github-Streak-Automation/actions/workflows/daily-commit.yml)
![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white)

## 快速开始

1. Fork 本仓库，或将项目文件复制到自己的 GitHub 仓库。
2. 确认 `.github/workflows/daily-commit.yml` 已推送到仓库的**默认分支**，且仓库允许运行 GitHub Actions。
3. 打开 **Settings → Actions → General → Workflow permissions**，选择 **Read and write permissions**，保存设置。工作流需要写权限才能提交并推送日志。
4. 打开仓库的 **Actions → Daily GitHub Streak Commit → Run workflow**，手动运行一次，确认配置正常。

之后，工作流会按计划自动运行。若默认分支受到保护规则限制，仍需调整规则，允许此工作流推送提交。

## 工作方式

- **定时运行：** 每天北京时间 08:10（UTC 00:10）触发。
- **更新日志：** 将带有日期、时间和时区的条目追加到 `activity/streak.md`。
- **避免重复：** 当天已有自动生成的条目时跳过；手动运行时可选择 `force=true` 再追加一条。
- **提交变更：** 只有日志文件确实发生变化时，才创建并推送提交。

如果希望这类提交显示在 GitHub 个人贡献图上，`GIT_AUTHOR_EMAIL` 必须是一个能解析到你账号的邮箱。推荐使用**带数字 ID 的 noreply 形式** `<数字ID>+<用户名>@users.noreply.github.com`（例如 `100594650+Sentryea@users.noreply.github.com`，ID 可在 https://github.com/settings/users 查到）。这种形式在你**修改 GitHub 用户名之后依然有效**，而旧式的 `<用户名>@users.noreply.github.com` 会随用户名一并失效，导致历史提交从贡献图上消失。私有仓库的贡献显示还受 GitHub 个人资料设置影响。

## 配置

在仓库 **Settings → Secrets and variables → Actions** 中按需添加变量，放在 **Variables** 或 **Secrets** 下都行，工作流会先读 Variables，再读 Secrets。它们是可选的：未设置时，作者名称和邮箱会使用工作流中的默认值。

| 变量 | 用途 | 示例 |
| --- | --- | --- |
| `GIT_AUTHOR_NAME` | 自动提交显示的作者名称 | `octocat` |
| `GIT_AUTHOR_EMAIL` | 自动提交使用的作者邮箱 | `100594650+octocat@users.noreply.github.com` |

未设置 `GIT_AUTHOR_EMAIL` 时，工作流默认使用 `${{ github.actor_id }}+${{ github.actor }}@users.noreply.github.com`，即带数字 ID 的 noreply 形式——这正是为了在用户名变更后仍能保持提交归属。

计划时间、日志路径和时区在 `.github/workflows/daily-commit.yml` 中设置：

```yaml
cron: "10 0 * * *"         # 每天 UTC 00:10，也就是北京时间 08:10
STREAK_FILE: activity/streak.md
STREAK_TIMEZONE: Asia/Shanghai
```

GitHub Actions 的 `schedule` 使用 UTC。修改 `cron` 时，请一并确认所需时区对应的 UTC 时间。

## 手动运行与本地使用

在 **Actions → Daily GitHub Streak Commit → Run workflow** 启动手动运行：

- `force=false`：当天已有自动条目时不重复写入。
- `force=true`：即使当天已有条目，也追加一条并提交。

项目使用 Python 标准库，无需安装第三方依赖。需要 Python 3.11 或更高版本；在仓库根目录运行：

```bash
# 更新当天的日志
python scripts/update_streak.py

# 即使当天已有条目，也再次追加
python scripts/update_streak.py --force

# 运行测试
python -m unittest discover -s tests
```

脚本也支持通过参数或环境变量自定义日志文件和时区：

```bash
python scripts/update_streak.py --file activity/streak.md --timezone Asia/Shanghai
```

对应环境变量为 `STREAK_FILE`、`STREAK_TIMEZONE` 和 `FORCE_UPDATE`。

## 项目结构

```text
.
├── .github/workflows/daily-commit.yml  # 定时任务与提交配置
├── activity/streak.md                  # 自动追加的每日日志
├── scripts/update_streak.py            # 日志更新脚本
├── tests/test_update_streak.py         # 单元测试
└── pyproject.toml                      # Python 项目配置
```

## 常见问题

**工作流成功运行，但没有新提交？** 先查看当天的自动条目是否已存在。默认行为会跳过重复日期；如确实要再写一条，手动运行时选择 `force=true`。

**推送时出现权限错误？** 检查 **Settings → Actions → General → Workflow permissions** 是否设为 **Read and write permissions**，以及默认分支的保护规则是否允许此工作流推送。

**提交没有显示在个人贡献图上？** 检查 `GIT_AUTHOR_EMAIL` 是否关联到 GitHub 账号，以及对应提交是否进入默认分支；私有仓库也需要检查个人资料中的私有贡献显示设置。

**改了 GitHub 用户名之后，贡献图上的连续记录断了？** GitHub 按提交邮箱解析作者归属，旧式 noreply 邮箱 `<旧用户名>@users.noreply.github.com` 会随用户名一并失效，历史提交因此不再计入贡献图。修复分两步：把 `GIT_AUTHOR_EMAIL` 改成带数字 ID 的形式（`<数字ID>+<新用户名>@users.noreply.github.com`），让后续提交继续归属；再改写已有历史的作者邮箱，把中断的日期补回贡献图。

改写历史可以用 `git filter-repo`，**保留每条提交原本的 author 日期**，GitHub 会按原日期重新计入：

```bash
git filter-repo --force --commit-callback '
commit.author_name = b"你的用户名"
commit.author_email = b"数字ID+你的用户名@users.noreply.github.com"
commit.committer_name = b"你的用户名"
commit.committer_email = b"数字ID+你的用户名@users.noreply.github.com"
'
git push --force-with-lease origin main
```

改写前先备份分支（如 `git branch backup/pre-identity-rewrite && git push origin backup/pre-identity-rewrite`），确认无误后再删除。

自动运行会真实地修改仓库文件并推送提交。请根据自己的使用场景设置运行频率与日志位置。
