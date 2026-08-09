# NyxNiri Fork 维护说明

本仓库是从原作者 `ech678/NyxNiri` Fork 而来。后续维护采用双分支结构：`main` 只跟随原仓库，`custom` 保存个人修改。

## 分支职责

| 分支 | 用途 | 维护原则 |
|---|---|---|
| `main` | 跟踪原作者仓库的 `main` | 不直接写个人改动，只用于同步上游 |
| `custom` | 保存个人配置和功能修改 | 日常使用和继续开发都在这里完成 |

`main` 是干净的上游镜像，`custom` 是自己的长期维护分支。这样做可以把上游更新和个人修改分开，后续变基、对比、回滚都更清楚。

## 远端配置

本地仓库应保留两个远端：

```bash
git remote -v
```

期望结构：

```text
origin    git@github.com:QinAoi/NyxNiri.git
upstream  https://gh-proxy.org/https://github.com/ech678/NyxNiri.git
```

其中：

- `origin` 是自己的 Fork，用于推送 `main` 和 `custom`。
- `upstream` 是原作者仓库，只用于拉取上游更新。

## 日常开发

所有个人修改都应发生在 `custom` 分支：

```bash
git switch custom
```

修改完成后正常提交并推送：

```bash
git status
git add <file>
git commit -m "feat: describe your change"
git push origin custom
```

不要把个人配置、脚本或桌面行为改动直接提交到 `main`。`main` 的价值在于保持和原仓库一致。

## 同步原仓库更新

当原作者仓库有新提交时，先更新本地远端引用：

```bash
git fetch upstream
git fetch origin
```

然后让自己的 `main` 快进到上游：

```bash
git switch main
git merge --ff-only upstream/main
git push origin main
```

这里使用 `--ff-only` 是为了保证 `main` 仍然是上游历史的直接延续。如果命令失败，说明本地 `main` 已经混入了额外提交，应先停止并检查，而不是强行合并。

## 将个人分支变基到最新上游

`main` 同步完成后，再把 `custom` 放到最新上游之上：

```bash
git switch custom
git rebase main
```

如果没有冲突，推送更新后的 `custom`：

```bash
git push --force-with-lease origin custom
```

`rebase` 会改写 `custom` 的提交基底，因此推送时需要非快进更新。这里必须使用 `--force-with-lease`，不要使用普通 `--force`。前者会在远端分支被别人或另一台机器更新时拒绝覆盖，能降低误删提交的风险。

## 冲突处理

如果变基时出现冲突，先查看冲突文件：

```bash
git status
```

手动解决文件内容后继续变基：

```bash
git add <resolved-file>
git rebase --continue
```

如果发现方向不对，可以放弃本次变基，回到变基前状态：

```bash
git rebase --abort
```

解决冲突时优先保留上游新增能力，再把个人修改重新叠加上去。这样可以避免为了保住旧改动而错过上游已经修复的问题。

## 变更记录维护

每次在 `custom` 分支加入个人改动后，都应在本文档记录一次。记录内容至少包括：

- 日期：修改完成或提交日期。
- 提交：对应的 Git commit 短哈希。
- 新增功能：这次具体加入了什么能力。
- 涉及文件：主要修改了哪些文件。
- 维护说明：后续同步上游时需要注意什么。

建议格式如下：

```markdown
### YYYY-MM-DD commit-short-hash

- 新增功能：描述这次加入的功能。
- 涉及文件：`path/to/file`、`path/to/another-file`
- 维护说明：记录上游同步、冲突处理或部署时需要注意的点。
```

这份记录不是流水账，只写会影响长期维护的信息。临时调试、一次性测试和没有进入提交历史的改动，不需要写入。

## 当前个人改动

### 2026-08-09 `254fd91`

- 新增功能：加入可搜索快捷键菜单，用 Kitty 启动 fzf，从 Niri 配置里的 `hotkey-overlay-title` 元数据提取快捷键说明。
- 新增功能：使用 `Super+Shift+/` 呼出自定义快捷键菜单，替代原生快捷键浮层的日常使用路径。
- 新增功能：菜单字体改为 `Noto Sans Mono CJK SC`，改善中英文混排时的基线和字重一致性。
- 新增功能：fzf 选中行指示条固定为粉色 `#f5c2e7`，并移除冗余提示文字，保持菜单界面纯净。
- 保留上游功能：保留上游新增的 `Super+G` Scratchpad 终端，并将其纳入快捷键菜单说明。
- 涉及文件：`README.md`、`v2/fish/config.fish`、`v2/niri/binds.kdl`、`v2/niri/rules.kdl`、`v2/niri/scripts/niri-binds`
- 维护说明：这些改动属于个人定制，应长期保留在 `custom`，不要提交回 `main`。

### 2026-08-09 文档维护说明

- 新增功能：新增本维护文档，明确原仓库同步、Fork 分支结构、个人改动维护方式和变更记录格式。
- 涉及文件：`docs/fork-maintenance.md`
- 维护说明：后续每次向 `custom` 加入功能性改动，都应同步更新本文档的“当前个人改动”部分。本文档自身的提交不记录自引用哈希，避免 amend 后哈希变化导致记录失真。

## 禁止操作

不要运行：

```bash
nyxniri update
```

该命令会执行类似 `git reset --hard origin/main` 的硬重置逻辑，可能直接覆盖个人修改。后续更新应使用本文档中的 `git fetch`、`merge --ff-only` 和 `rebase` 流程。

也不要在没有确认备份的情况下重新运行安装器。源码维护和部署到 `~/.config` 是两件事：先确认 Git 历史正确，再决定是否部署配置。

## 验证命令

维护完成后，可用以下命令确认结构正确：

```bash
git branch -vv
git log --oneline --decorate --graph --all -n 20
git status --short
```

期望状态：

- `main` 跟踪 `origin/main`，并与 `upstream/main` 保持一致。
- `custom` 跟踪 `origin/custom`，并包含个人改动。
- 日常工作目录干净，除非正在开发新的修改。
