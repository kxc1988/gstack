---
name: gstack-upgrade
version: 1.1.0
description: |
  将 gstack 升级到最新版本。检测全局 vs  vendored 安装，
  运行升级，并显示新增内容。当被要求"升级 gstack"、
  "更新 gstack"或"获取最新版本"时使用。
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /gstack-upgrade

将 gstack 升级到最新版本并显示新增内容。

## 内联升级流程

当所有技能序言检测到 `UPGRADE_AVAILABLE` 时，会引用此部分。

### 步骤 1：询问用户（或自动升级）

首先，检查是否启用了自动升级：
```bash
_AUTO=""
[ "${GSTACK_AUTO_UPGRADE:-}" = "1" ] && _AUTO="true"
[ -z "$_AUTO" ] && _AUTO=$(~/.claude/skills/gstack/bin/gstack-config get auto_upgrade 2>/dev/null || true)
echo "AUTO_UPGRADE=$_AUTO"
```

**如果 `AUTO_UPGRADE=true` 或 `AUTO_UPGRADE=1`：** 跳过 AskUserQuestion。记录 "Auto-upgrading gstack v{old} → v{new}..." 并直接进入步骤 2。如果自动升级期间 `./setup` 失败，从备份（`.bak` 目录）恢复并警告用户："Auto-upgrade failed — restored previous version. Run `/gstack-upgrade` manually to retry."

**否则**，使用 AskUserQuestion：
- 问题："gstack **v{new}** 可用（你当前在 v{old}）。现在升级？"
- 选项：["Yes, upgrade now", "Always keep me up to date", "Not now", "Never ask again"]

**如果选择 "Yes, upgrade now"：** 进入步骤 2。

**如果选择 "Always keep me up to date"：**
```bash
~/.claude/skills/gstack/bin/gstack-config set auto_upgrade true
```
告诉用户："Auto-upgrade enabled. Future updates will install automatically." 然后进入步骤 2。

**如果选择 "Not now"：** 写入带递增退避的 snooze 状态（第一次 snooze = 24 小时，第二次 = 48 小时，第三次及以上 = 1 周），然后继续当前技能。不再提及升级。
```bash
_SNOOZE_FILE=~/.gstack/update-snoozed
_REMOTE_VER="{new}"
_CUR_LEVEL=0
if [ -f "$_SNOOZE_FILE" ]; then
  _SNOOZED_VER=$(awk '{print $1}' "$_SNOOZE_FILE")
  if [ "$_SNOOZED_VER" = "$_REMOTE_VER" ]; then
    _CUR_LEVEL=$(awk '{print $2}' "$_SNOOZE_FILE")
    case "$_CUR_LEVEL" in *[!0-9]*) _CUR_LEVEL=0 ;; esac
  fi
fi
_NEW_LEVEL=$((_CUR_LEVEL + 1))
[ "$_NEW_LEVEL" -gt 3 ] && _NEW_LEVEL=3
echo "$_REMOTE_VER $_NEW_LEVEL $(date +%s)" > "$_SNOOZE_FILE"
```
注意：`{new}` 是来自 `UPGRADE_AVAILABLE` 输出的远程版本 — 从更新检查结果中替换它。

告诉用户 snooze 持续时间："Next reminder in 24h"（或 48h 或 1 周，取决于级别）。提示："Set `auto_upgrade: true` in `~/.gstack/config.yaml` for automatic upgrades."

**如果选择 "Never ask again"：**
```bash
~/.claude/skills/gstack/bin/gstack-config set update_check false
```
告诉用户："Update checks disabled. Run `~/.claude/skills/gstack/bin/gstack-config set update_check true` to re-enable."
继续当前技能。

### 步骤 2：检测安装类型

```bash
if [ -d "$HOME/.claude/skills/gstack/.git" ]; then
  INSTALL_TYPE="global-git"
  INSTALL_DIR="$HOME/.claude/skills/gstack"
elif [ -d ".claude/skills/gstack/.git" ]; then
  INSTALL_TYPE="local-git"
  INSTALL_DIR=".claude/skills/gstack"
elif [ -d ".claude/skills/gstack" ]; then
  INSTALL_TYPE="vendored"
  INSTALL_DIR=".claude/skills/gstack"
elif [ -d "$HOME/.claude/skills/gstack" ]; then
  INSTALL_TYPE="vendored-global"
  INSTALL_DIR="$HOME/.claude/skills/gstack"
else
  echo "ERROR: gstack not found"
  exit 1
fi
echo "Install type: $INSTALL_TYPE at $INSTALL_DIR"
```

上面打印的安装类型和目录路径将在所有后续步骤中使用。

### 步骤 3：保存旧版本

使用步骤 2 输出中的安装目录：

```bash
OLD_VERSION=$(cat "$INSTALL_DIR/VERSION" 2>/dev/null || echo "unknown")
```

### 步骤 4：升级

使用步骤 2 中检测到的安装类型和目录：

**对于 git 安装**（global-git, local-git）：
```bash
cd "$INSTALL_DIR"
STASH_OUTPUT=$(git stash 2>&1)
git fetch origin
git reset --hard origin/main
./setup
```
如果 `$STASH_OUTPUT` 包含 "Saved working directory"，警告用户："Note: local changes were stashed. Run `git stash pop` in the skill directory to restore them."

**对于 vendored 安装**（vendored, vendored-global）：
```bash
PARENT=$(dirname "$INSTALL_DIR")
TMP_DIR=$(mktemp -d)
git clone --depth 1 https://github.com/garrytan/gstack.git "$TMP_DIR/gstack"
mv "$INSTALL_DIR" "$INSTALL_DIR.bak"
mv "$TMP_DIR/gstack" "$INSTALL_DIR"
cd "$INSTALL_DIR" && ./setup
rm -rf "$INSTALL_DIR.bak" "$TMP_DIR"
```

### 步骤 4.5：同步本地 vendored 副本

使用步骤 2 中的安装目录。检查是否还有需要更新的本地 vendored 副本：

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
LOCAL_GSTACK=""
if [ -n "$_ROOT" ] && [ -d "$_ROOT/.claude/skills/gstack" ]; then
  _RESOLVED_LOCAL=$(cd "$_ROOT/.claude/skills/gstack" && pwd -P)
  _RESOLVED_PRIMARY=$(cd "$INSTALL_DIR" && pwd -P)
  if [ "$_RESOLVED_LOCAL" != "$_RESOLVED_PRIMARY" ]; then
    LOCAL_GSTACK="$_ROOT/.claude/skills/gstack"
  fi
fi
echo "LOCAL_GSTACK=$LOCAL_GSTACK"
```

如果 `LOCAL_GSTACK` 非空，通过从新升级的主安装复制来更新它（与 README vendored 安装相同的方法）：
```bash
mv "$LOCAL_GSTACK" "$LOCAL_GSTACK.bak"
cp -Rf "$INSTALL_DIR" "$LOCAL_GSTACK"
rm -rf "$LOCAL_GSTACK/.git"
cd "$LOCAL_GSTACK" && ./setup
rm -rf "$LOCAL_GSTACK.bak"
```
告诉用户："Also updated vendored copy at `$LOCAL_GSTACK` — commit `.claude/skills/gstack/` when you're ready."

如果 `./setup` 失败，从备份恢复并警告用户：
```bash
rm -rf "$LOCAL_GSTACK"
mv "$LOCAL_GSTACK.bak" "$LOCAL_GSTACK"
```
告诉用户："Sync failed — restored previous version at `$LOCAL_GSTACK`. Run `/gstack-upgrade` manually to retry."

### 步骤 5：写入标记 + 清除缓存

```bash
mkdir -p ~/.gstack
echo "$OLD_VERSION" > ~/.gstack/just-upgraded-from
rm -f ~/.gstack/last-update-check
rm -f ~/.gstack/update-snoozed
```

### 步骤 6：显示新增内容

读取 `$INSTALL_DIR/CHANGELOG.md`。找到旧版本和新版本之间的所有版本条目。按主题分组总结为 5-7 个项目符号。不要过度 — 专注于面向用户的更改。除非它们很重要，否则跳过内部重构。

格式：
```
gstack v{new} — upgraded from v{old}!

What's new:
- [bullet 1]
- [bullet 2]
- ...

Happy shipping!
```

### 步骤 7：继续

显示新增内容后，继续用户最初调用的任何技能。升级完成 — 不需要进一步操作。

---

## 独立使用

当直接以 `/gstack-upgrade` 调用时（不是从序言）：

1. 强制进行新的更新检查（绕过缓存）：
```bash
~/.claude/skills/gstack/bin/gstack-update-check --force 2>/dev/null || 
.claude/skills/gstack/bin/gstack-update-check --force 2>/dev/null || true
```
使用输出来确定是否有可用的升级。

2. 如果 `UPGRADE_AVAILABLE <old> <new>`：遵循上面的步骤 2-6。

3. 如果无输出（主安装已最新）：检查是否有过时的本地 vendored 副本。

运行上面的步骤 2 bash 块来检测主安装类型和目录（`INSTALL_TYPE` 和 `INSTALL_DIR`）。然后运行上面的步骤 4.5 检测 bash 块来检查本地 vendored 副本（`LOCAL_GSTACK`）。

**如果 `LOCAL_GSTACK` 为空**（无本地 vendored 副本）：告诉用户 "You're already on the latest version (v{version})."

**如果 `LOCAL_GSTACK` 非空**，比较版本：
```bash
PRIMARY_VER=$(cat "$INSTALL_DIR/VERSION" 2>/dev/null || echo "unknown")
LOCAL_VER=$(cat "$LOCAL_GSTACK/VERSION" 2>/dev/null || echo "unknown")
echo "PRIMARY=$PRIMARY_VER LOCAL=$LOCAL_VER"
```

**如果版本不同：** 遵循上面的步骤 4.5 同步 bash 块，从主安装更新本地副本。告诉用户："Global v{PRIMARY_VER} is up to date. Updated local vendored copy from v{LOCAL_VER} → v{PRIMARY_VER}. Commit `.claude/skills/gstack/` when you're ready."

**如果版本匹配：** 告诉用户 "You're on the latest version (v{PRIMARY_VER}). Global and local vendored copy are both up to date."