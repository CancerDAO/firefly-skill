# 安装指南 · Installation

## 一行安装 · One-line install

```bash
npx skills add CancerDAO/firefly-skill -g -a claude-code -y
```

这会用 [vercel-labs/skills](https://github.com/vercel-labs/skills) CLI 把 11 个 skill 一次性 symlink 到 `~/.claude/skills/`（全局 + Claude Code）。装完重启 Claude Code 即可。

## 选项 · Options

```bash
# 仅装到当前项目（不全局）
npx skills add CancerDAO/firefly-skill -a claude-code -y

# 装到多个 agent（OpenCode / Cursor / Codex 等也支持）
npx skills add CancerDAO/firefly-skill -g -a claude-code -a cursor -y

# 只装某几个 companion
npx skills add CancerDAO/firefly-skill -g -a claude-code \
  --skill firefly --skill firefly-organize --skill firefly-mind -y

# 列出本仓所有 skill
npx skills add CancerDAO/firefly-skill --list

# 物理拷贝（不用 symlink）
npx skills add CancerDAO/firefly-skill -g -a claude-code --copy -y
```

## 验证 · Verify

```bash
ls -la ~/.claude/skills/ | grep firefly
```

应见 11 条 symlink 指向 clone 出的源代码目录。

快速测试：在 Claude Code 输入 "我家孩子确诊不了，想做诊断导航"，应触发 `firefly` 路由到 `firefly-organize`。

## 更新 · Update

```bash
npx skills update -a claude-code
```

或单独更新 firefly：

```bash
npx skills update firefly -a claude-code
```

## 卸载 · Uninstall

```bash
npx skills remove firefly firefly-organize firefly-genetic-counseling \
  firefly-education firefly-caregiver firefly-mind firefly-diet \
  firefly-second-opinion firefly-vault firefly-disclosure firefly-patient-org \
  -g -a claude-code
```

## 手动安装（不依赖 CLI）· Manual install

如果你不想用 `npx`，也可以手动 symlink：

```bash
git clone https://github.com/CancerDAO/firefly-skill.git ~/firefly-skill

mkdir -p ~/.claude/skills

for skill in firefly firefly-organize firefly-genetic-counseling firefly-education \
             firefly-caregiver firefly-mind firefly-diet firefly-second-opinion \
             firefly-vault firefly-disclosure firefly-patient-org; do
  ln -sf ~/firefly-skill/skills/$skill ~/.claude/skills/$skill
done
```

## 疑难排查 · Troubleshooting

**`npx skills` 找不到命令**
- 确认 Node.js ≥ 18：`node --version`
- 第一次运行会下载 CLI，等几秒

**Claude Code 不识别 skills**
- 完全退出 Claude Code 再启动（不只是关闭窗口）
- 确认 symlink 有效：`readlink ~/.claude/skills/firefly`

**更新后行为没变**
- SKILL.md frontmatter 改动需要重启 Claude Code

**想在 macOS 之外的系统**
- Linux 同理；Windows 用 WSL2 或 Git Bash
