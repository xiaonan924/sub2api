# Fork 定制与上游同步

## 仓库与分支

- `origin`：https://github.com/xiaonan924/sub2api.git（自己的 Fork）。
- `upstream`：https://github.com/Wei-Shaw/sub2api.git（官方仓库）。
- `main`：只跟随官方，不放定制代码。
- `custom`：长期维护定制功能；定制部署应使用此分支通过验证后的提交。
- `feat/<功能名>`：从 `custom` 创建，完成后合并回 `custom`。

初次配置于 2026-10-14：Fork 原 `main` 为 `efe9aab1e`，没有独有提交，落后官方 655 个提交；本地已快进到 `9c2cca9e6`，再创建 `custom`。本次只配置代码维护流程，不代表应用已通过部署验证。远程分支状态以实际推送结果为准。

## 新电脑首次配置

```bash
git clone https://github.com/xiaonan924/sub2api.git
cd sub2api
git remote add upstream https://github.com/Wei-Shaw/sub2api.git
git fetch upstream
git switch custom
git config remote.pushDefault origin
git config pull.ff only
git config push.default simple
```

这些配置仅作用于当前仓库，重新克隆后需要再配置。HTTPS 推送需要本机有有效的 GitHub 凭据；不要把令牌放进远程地址或提交到仓库。

## 开发定制功能

确认工作区干净后，从最新的 `custom` 创建功能分支：

```bash
git switch custom
git pull --ff-only origin custom
git switch -c feat/my-feature
```

完成修改并提交、验证后，推送功能分支，向自己 Fork 的 `custom` 分支发起 PR。不要误选官方仓库或自己的 `main` 为目标。

## 同步官方更新

建议按稳定版本或每周评估一次；关键修复及时处理。以下命令逐段执行，任一步失败就停下处理。开始前用 `git status` 确认没有未提交改动或未完成的合并。

先更新原版分支：

```bash
git fetch origin
git fetch upstream
git switch main
git merge --ff-only origin/main
git merge --ff-only upstream/main
git push origin main
```

如果快进失败，不要使用强制推送或硬重置，先检查是否有人在 `main` 做了定制提交。

然后从定制分支创建本次同步分支。将示例中的日期替换成本次日期，必要时加序号，确保分支名未被使用：

```bash
git switch custom
git merge --ff-only origin/custom
git switch -c sync/upstream-20261014
git merge --no-edit upstream/main
```

有冲突时，按业务含义保留定制逻辑并适配新接口，解决后只暂存确认过的文件，再执行 `git commit`。要取消本次尚未完成的合并，可执行 `git merge --abort`。

检查发布说明、配置变化与数据库迁移，并完成相关测试。当前仓库已有 CI；本地常用验证入口为：

```bash
pnpm --dir frontend install --frozen-lockfile
make test-frontend
make -C backend test-unit
make -C backend test-integration
make build
```

运行前安装与当前项目要求匹配的 Go、Node.js、pnpm，并满足集成测试环境要求。具体版本和完整检查以 `.github/workflows/backend-ci.yml`、`backend/go.mod` 及构建配置为准。

```bash
git push -u origin sync/upstream-20261014
```

向自己的 `custom` 发起 PR，确认 CI 和定制功能验证通过再合并。同步 PR 使用 **Create a merge commit**，保留上游提交关系；不要对同步 PR 使用 squash 或 rebase 合并。若 Fork 的 Actions 未启用，先在 GitHub 的 Actions 页面启用，不能把“未运行”当作通过。

部署使用验证后的确定提交或自己的发布标签。数据库迁移前先备份；代码回退不等于数据库可以直接回退。

## 减少后续冲突

- 尽量通过配置或独立模块实现定制，减少核心文件改动。
- 每个提交聚焦一个功能，避免无关格式化与批量重命名。
- 数据库变更使用新的迁移文件，不改已执行的官方迁移。
- 长期 `custom` 分支通过 merge 接收上游更新，不强制改写历史。
- 本次未配置定时任务；后续同步由手动操作或明确配置的自动化触发。
