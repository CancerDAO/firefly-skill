# 安装指南 · Installation

## 依赖 · Requirements

- [Claude Code](https://claude.ai/code) CLI（最新版）
- git

## 三步安装 · 3-Step Install

### 1. Clone 仓库

```bash
cd ~
git clone https://github.com/zwbao/firefly-skill.git
```

### 2. 建立 symlink

```bash
# 备份旧 firefly（如果有）
mkdir -p ~/.claude/skills/_deprecated
mv ~/.claude/skills/firefly ~/.claude/skills/_deprecated/firefly-pre-family-$(date +%Y%m%d) 2>/dev/null || true

# 建 11 条 symlink
for skill in firefly firefly-organize firefly-genetic-counseling firefly-education \
             firefly-caregiver firefly-mind firefly-diet firefly-second-opinion \
             firefly-vault firefly-disclosure firefly-patient-org; do
  ln -sf ~/firefly-skill/$skill ~/.claude/skills/$skill
done
```

### 3. 重启 Claude Code

关闭当前会话，重新启动。输入 `/help` 应能看到 `firefly` + 10 个 `firefly-*` companion。

## 验证 · Verify

```bash
ls -la ~/.claude/skills/ | grep firefly
```

应看到 11 条 symlink 指向 `~/firefly-skill/`。

快速测试：在 Claude Code 里输入 "我家孩子确诊不了，想做一下诊断导航"，应触发 `firefly` 路由到 `firefly-organize`。

## 更新 · Update

```bash
cd ~/firefly-skill && git pull
```

symlink 自动跟随。不需要重启 Claude Code（除非 skill frontmatter 变了）。

## 卸载 · Uninstall

```bash
for skill in firefly firefly-organize firefly-genetic-counseling firefly-education \
             firefly-caregiver firefly-mind firefly-diet firefly-second-opinion \
             firefly-vault firefly-disclosure firefly-patient-org; do
  rm ~/.claude/skills/$skill
done

# 可选：删除源代码
rm -rf ~/firefly-skill
```

## 疑难排查 · Troubleshooting

**Claude Code 不识别 skills**
- 确认 `~/.claude/skills/` 存在且 symlink 目标有效：`readlink ~/.claude/skills/firefly`
- 完全退出 Claude Code 再启动（不只是关闭窗口）

**Symlink 创建失败**
- 检查 `~/.claude/skills/` 是否存在：`mkdir -p ~/.claude/skills/`
- macOS 权限问题：System Preferences → Privacy → Full Disk Access 给 Terminal

**更新后 skill 行为没变**
- 某些 SKILL.md frontmatter 改动需要重启 Claude Code
- 清理潜在缓存：`rm -rf ~/.claude/cache/skills/` （如存在）

**想在其他路径使用**
- 修改 symlink 目标到你 clone 的位置：`ln -sf /path/to/your/firefly-skill/<skill> ~/.claude/skills/<skill>`
