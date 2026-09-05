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

skill/hook 变化执行对应已有测试；安装前至少核对版本一致性，hook 路由变化再运行对应测试：

```bash
node scripts/check-versions.js
node --test tests/hooks.test.js
```

当前 marketplace 使用 `.claude-plugin/marketplace.json`，安装器读取 Claude manifest 的版本并复用同版本缓存。
仅给 `.codex-plugin/plugin.json` 加版本后缀不能刷新该入口，且会破坏上游多平台版本一致性。
保留上游版本合同；同版本个人改动通过 Codex 官方 remove/add 命令刷新插件缓存，不手改 marketplace 或缓存文件。
验证并提交本任务文件后：

```bash
git push origin codex/personal
codex plugin remove ponytail@ponytail
codex plugin add ponytail@ponytail
```

remove/add 是更新整个插件的连续步骤，不能只安装其 skill 子目录。执行前备份已安装插件和独立配置；安装失败时按已确认的本地 source 重装，必要时恢复备份。
安装后核对 marketplace 仍指向本 fork、缓存中的 skills/hooks/manifest 与个人分支逐文件一致，再在新任务中验证注入效果。
不要通过原作者 marketplace 的自动升级覆盖本地维护来源。

内置 plugin-creator 静态校验器目前拒绝上游现有的 `hooks` 字段，原版与本 fork 均有相同结果；保留该字段，并用 Codex 实际安装结果与 hook 测试核验，不为通过此校验器移除插件能力。
默认模式在独立的 `~/.config/ponytail/config.json` 管理，不写入公共 fork。回滚优先 `git revert`，提交并重新安装。
