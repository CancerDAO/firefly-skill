# 安装指南 · Installation

萤火是 **agent-agnostic skill 家族**，可装到 Claude Code / Codex / OpenCode / Cursor 等 [vercel-labs/skills](https://github.com/vercel-labs/skills) 支持的 45+ 个 AI agent。

## 一行安装 · One-line install

### Claude Code（最常见）

```bash
npx skills add CancerDAO/firefly-skill -g -a claude-code -y
```

### Codex

```bash
npx skills add CancerDAO/firefly-skill -g -a codex -y
```

### OpenCode

```bash
npx skills add CancerDAO/firefly-skill -g -a opencode -y
```

### Cursor

```bash
npx skills add CancerDAO/firefly-skill -g -a cursor -y
```

### 多 agent 同时安装

```bash
npx skills add CancerDAO/firefly-skill -g -a claude-code -a codex -a opencode -a cursor -y
```

### 全部支持的 agent（45+）

```bash
npx skills add CancerDAO/firefly-skill -g --all -y
```

装完**重启对应 agent**即可生效。

## 选项 · Options

```bash
# 项目级（不全局，装到 ./<agent>/skills/，可随项目 commit）
npx skills add CancerDAO/firefly-skill -a claude-code -y

# 只装某几个 companion
npx skills add CancerDAO/firefly-skill -g -a claude-code \
  --skill firefly --skill firefly-organize --skill firefly-mind -y

# 列出本仓所有 skill
npx skills add CancerDAO/firefly-skill --list

# 物理拷贝（不用 symlink）
npx skills add CancerDAO/firefly-skill -g -a claude-code --copy -y
```

完整 CLI 文档：https://github.com/vercel-labs/skills

## 验证 · Verify

按你装的 agent，到对应 skills 目录验证：

```bash
# Claude Code (global)
ls -la ~/.claude/skills/ | grep firefly

# Codex (global)
ls -la ~/.codex/skills/ | grep firefly

# OpenCode (global)
ls -la ~/.opencode/skills/ | grep firefly

# Cursor (global, project: ./.cursor/skills/)
ls -la ~/.cursor/skills/ | grep firefly
```

应见 11 条 symlink 指向 clone 出的源代码目录。

快速测试：在你的 agent 里输入 "我家孩子确诊不了，想做诊断导航"，应触发 `firefly` 路由到 `firefly-organize`。

## 更新 · Update

```bash
npx skills update -a claude-code         # Claude Code
npx skills update -a codex               # Codex
# ...
```

或全 agent：

```bash
npx skills update
```

## 卸载 · Uninstall

```bash
npx skills remove firefly firefly-organize firefly-genetic-counseling \
  firefly-education firefly-caregiver firefly-mind firefly-diet \
  firefly-second-opinion firefly-vault firefly-disclosure firefly-patient-org \
  -g -a claude-code
```

（替换 `claude-code` 为你装到的 agent）

## 手动安装（不依赖 CLI）· Manual install

如果你不想用 `npx`，也可以直接 clone + symlink。下面以 Claude Code 为例，其他 agent 类似（替换 `~/.claude/skills/` 为对应 agent 的 skills 目录）：

```bash
git clone https://github.com/CancerDAO/firefly-skill.git ~/firefly-skill

mkdir -p ~/.claude/skills

for skill in firefly firefly-organize firefly-genetic-counseling firefly-education \
             firefly-caregiver firefly-mind firefly-diet firefly-second-opinion \
             firefly-vault firefly-disclosure firefly-patient-org; do
  ln -sf ~/firefly-skill/skills/$skill ~/.claude/skills/$skill
done
```

各 agent 的 skills 目录约定见 [vercel-labs/skills](https://github.com/vercel-labs/skills) 的 "Available Agents" 表。

## 疑难排查 · Troubleshooting

**`npx skills` 找不到命令**
- 确认 Node.js ≥ 18：`node --version`
- 第一次运行会下载 CLI，等几秒
- 网络问题用：`pnpm dlx skills add ...` 或 `bunx skills add ...`

**Agent 不识别 skills**
- 完全退出 agent 再启动（不只是关闭窗口）
- 确认 symlink 有效：`readlink ~/.claude/skills/firefly`（或对应 agent 路径）

**更新后行为没变**
- SKILL.md frontmatter 改动需要重启 agent

**Windows**
- 用 WSL2 或 Git Bash；原生 Windows 的 symlink 需要管理员权限或开启开发者模式
