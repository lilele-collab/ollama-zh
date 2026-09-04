# Fork 接手指南

本仓库已在 2026-08-30 停止维护。这份文档说明如何用 fork 接管"自动跟随上游构建汉化补丁"的流水线。

## 一次性设置

1. 在 GitHub 上 fork 本仓库到自己的账号。
2. 打开 fork 仓库的 **Actions** 标签页,如果出现"启用 workflows"的提示,点击启用。
   (fork 出来的仓库 Actions 默认关闭,必须手动开一次。)
3. 在 Actions 列表里分别找到 **Build Ollama Hanhua** 和 **Keepalive** 两个 workflow,
   各自点进去,在右上角 "..." 菜单里确认定时调度已启用。
4. 手动触发一次验证: 进入 **Build Ollama Hanhua** → **Run workflow** → version 留空 → 运行。
   成功的标志是 fork 仓库出现 `zh-vX.Y.Z` Release 和 win64 zip 产物。

## 之后的日常运行(全自动)

- `build-zh.yml` 每天 UTC 04:00(北京 12:00)检测上游 [ollama/ollama](https://github.com/ollama/ollama) 的最新 Release:
  有新版本 → 自动应用词典补丁 → 构建前端 + 桌面端 → 发布 `zh-vX.Y.Z` Release。
- `keepalive.yml` 每周做一次空提交,防止 GitHub 因 60 天无活动自动关闭定时构建。
- 构建失败时会自动创建/更新 `build-fail` Issue,带失败运行的日志链接,不需要盯着 Actions 页面。

## 上游更新后构建失败的修复流程

1. 打开 `build-fail` Issue,点进失败运行的日志,确认是哪个步骤失败:
   - "应用汉化补丁"失败 → 上游改了被 patch 的文件,需要修 hunk;
   - "构建前端/编译"失败 → 上游改了代码结构或依赖,可能需要同步改 `tools/translate.mjs` 的 specialRules;
   - 偶发网络失败 → 直接 Re-run 一次。
2. 在本地:
   ```bash
   git clone --depth 1 --branch vX.Y.Z https://github.com/ollama/ollama.git upstream
   git -C upstream apply translations/zh-CN.patch
   ```
   按冲突位置更新 `translations/zh-CN.json`(新词条)或 `tools/translate.mjs`(结构性改动)。
3. 重新生成补丁并提交:
   ```bash
   node tools/translate.mjs upstream translated
   git -C translated diff --no-index --git ../upstream ../translated > translations/zh-CN.patch
   ```
4. 推送到 fork 后手动触发 workflow,或等当天定时任务自动重试。

## 本地安装汉化包

下载 fork 仓库 Release 里的 `ollama-zh-vX.Y.Z-win64.zip`,解压后双击 `apply-patch.cmd`。
官方自动更新覆盖 exe 后,重跑一次 `apply-patch.ps1` 即可恢复中文。
