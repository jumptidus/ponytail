# Ponytail 个人 fork 维护

- fork：`jumptidus/ponytail`
- upstream：`DietrichGebert/ponytail`
- 个人分支：`codex/personal`
- 初始基线：`974d940`，包含迁移前安装的 v4.9.0 skill 与 hook 兼容修复。

此插件同时包含 skills 与 hooks。修改真正的 `skills/`、`hooks/` 源文件，保留两者合同；不编辑 Codex 版本缓存。
本机 marketplace 应指向该 fork 的本地 checkout，名称保持 `ponytail`，避免重复加载两个同名插件。

## 合并上游更新

在 `codex/personal` 维护个人提交，`origin` 指向个人 fork，`upstream` 指向原作者仓库。
升级前确认工作区干净；存在未提交工作时先处理本任务改动，不自动 stash、reset 或覆盖。

```bash
git switch codex/personal
git status --short
git fetch upstream --tags
git log --oneline HEAD..upstream/main
```

默认选已审查的上游发布 tag；只有明确需要未发布修复时才选 `upstream/main`。
将下面的 `UPSTREAM_REF` 设置为选定的实际 tag 或 commit，再执行合并：

```bash
git merge --no-ff --no-commit "$UPSTREAM_REF"
```

处理冲突时同时核对个人改动目的与上游行为，不直接整份选择 ours/theirs。
验证完成后提交 merge 并 push 到 `origin codex/personal`，再刷新安装；若显示 Already up to date，则无需创建空 merge 提交。
不得用 rebase、reset 或重新复制上游文件替代合并，也不在安装目录维护源码。

## 验证与安装

skill/hook 变化执行对应已有测试；hook 路由变化至少运行：

```bash
node --test tests/hooks.test.js
```

通过后用 Codex 的 plugin-creator helper 更新本地安装版本标记，再提交本任务文件并 push：

```bash
python3 "$HOME/.codex/skills/.system/plugin-creator/scripts/update_plugin_cachebuster.py" .
git diff --check
git push origin codex/personal
codex plugin add ponytail@ponytail
```

上面 push 前必须先提交版本标记与已验证的个人改动。安装后核对 marketplace 仍指向本 fork、本次缓存版本与源码一致，再在新任务中验证注入效果。
不要通过原作者 marketplace 的自动升级覆盖本地维护来源。
默认模式在独立的 `~/.config/ponytail/config.json` 管理，不写入公共 fork。回滚优先 `git revert`，重新生成版本标记、提交并安装。
