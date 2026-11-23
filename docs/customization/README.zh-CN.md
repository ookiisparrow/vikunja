# Fork 定制起点

本文只记录 `ookiisparrow/vikunja` 的开发基线、验证结果和需求候选。它不改变产品行为。

仓库其余部分的许可证与版权不变：主体仍是 [AGPL-3.0-or-later](../../LICENSE)，`desktop/` 仍是 GPL-3.0-or-later。本文不替换 `LICENSE`、`README.md` 或源文件版权头。

核查日期：2026-11-23。数字来自当时的远程引用和本机只读比较，不是对生产环境的探测。

## 仓库与许可

| 项 | 事实 |
| --- | --- |
| 本 fork | <https://github.com/ookiisparrow/vikunja> |
| 上游 | <https://github.com/go-vikunja/vikunja>（GitHub `parent` / `source`） |
| 默认分支 | `main` |
| 许可证 | 主体 GNU AGPL-3.0-or-later；`desktop/` 为 GPL-3.0-or-later |
| 版权头 | Go 源文件保留 `Copyright 2018-present Vikunja and contributors` |
| 创建时间 | 2026-11-23T03:36:50Z |
| 远程分支 | 只有 `refs/heads/main` |
| 远程 tag | 无。`git ls-remote --tags origin` 为空 |

Fork 关联还在。当天 `main` 与上游 `main` 同为 `84751cab7f3f7b746dc2f57192e5f255e87b4793`（2026-11-22，`fix(auth): stop email confirm link looping for signed-in users (#4463)`）。相对上游 `main` 的独有提交数是 0。近期提交都是上游提交，不是本 fork 的定制。

本工作副本的 `origin` 指向本 fork。没有配置 `upstream` 远程，也没有自动同步。之后若无人从上游快进，`main` 会停在这次 fork 的时刻。

目录分工沿用仓库自带说明，不另起一套：

- Go API：仓库根模块 `code.vikunja.io/api`（`main.go`、`pkg/`、`magefile.go`）。`go.mod` 写 `go 1.27.0`。`mise.toml` 另写 Go `1.27.3`。
- 前端：`frontend/`，Vue 3 + TypeScript + Vite。包管理器是 `pnpm@12.16.1`，`engines.node` 为 `>=24`。`frontend/.nvmrc` 与 `mise.toml` 都是 Node `24.21.0`。
- `veans/`：独立 Go 模块 `code.vikunja.io/veans`，命令见 `veans/AGENTS.md`。不要从这里 import API 模块。
- `desktop/`：单独的 GPL 许可。

构建和测试以这些文件为准：

- 根目录 `AGENTS.md`、`CONTRIBUTING.md`
- `mage -l`（别名在 `magefile.go` 的 `Aliases`）
- `frontend/package.json` 的 scripts

后端：

- 列出目标：`mage -l`
- 编译二进制：`mage build:build`（别名 `mage build`）
- 测试：`mage test:feature`、`mage test:web`、`mage test:filter <go-test-filter>`。不要用裸的 `go test` 代替这套入口。`test:filter` 对除 `pkg/webtests` 以外的包加 `-short`，再对 `pkg/webtests` 不加 `-short` 跑第二遍，避免 web 测试在 `-short` 下被整包跳过。
- 后端 lint：`mage lint:fix`（别名到 `check:golangciFix`）

前端：

- 开发服务器：在 `frontend/` 执行 `pnpm dev`（端口 4173）
- 生产构建：`pnpm build`
- 单元测试：`pnpm vitest run <file>`
- lint：`pnpm lint:fix`；样式改动再加 `pnpm lint:styles:fix`
- 端到端测试走 `mage test:e2e`，不要直接 `pnpm test:e2e`

`mage` 版本与 `go.mod` 一致：`github.com/magefile/mage v1.17.2`。

## 版本差异

生产背景（不连接该环境）：正在运行的是官方 ghcr 镜像 Vikunja v2.8.0 + SQLite。仓库 `main` 高于这个 tag。

上游 tag `v2.8.0`：

- 附注 tag 对象：`5dff8b9e09426ce5d4b955d7dd66eb81dff29d5d`
- 指向的提交：`6e788bf87cebe50e2b8d034b213fcf94ae5449ee`
- 时间：2026-11-05 16:08:13 +0100
- 说明：`chore: v2.8.0 release preparations`
- 该提交是当前 `main` 的祖先

本 fork 的远程没有这个 tag。比较时从上游取了 tag，只放在本机，没有推回 fork。

`v2.8.0..main`：

- `main` 比 tag 多 188 个提交，tag 没有 `main` 之外的提交
- `git diff --shortstat`：293 个文件，+14077 / -3244
- 提交说明前缀大致是：61 个 `fix`、43 个 `feat`、25 个 `chore`、23 个 `test`、20 个 `refactor`，其余为 docs / ci 等
- 版本字符串还没为这些提交单独发版：`CHANGELOG.md` 最新发布节仍是 2.8.0（2026-11-05），`frontend/package.json` 的 `version` 仍是 `2.8.0`，`publiccode.yml` 的 `softwareVersion` 仍是 `v2.8.0`，`pkg/version.Version` 默认是 `dev`

数据库相关差异：`pkg/migration` 有 4 个文件相对 tag 有改动，其中新增 `pkg/migration/20261112082349.go`（说明为把逾期提醒时间补成 `HH:MM`）。把 `main` 部署到现有 SQLite 会跑迁移。那是升级，不是这次的范围。

## 隔离开发与验证

不接触正在运行的 Vikunja、SQLite、任务数据、token、密码或密钥。不在迷你主机上编译，不替换镜像，不改端口或网络。

1. 在本 fork 上从选定起点拉新分支。文档改动可以基于当前 `main`。产品改动等用户选定起点后再开分支：若选 v2.8.0，从该 tag 拉；若选 `main`，从当时的 `upstream/main` 拉。分支放自定义提交，不要把自定义提交直接堆在 `main` 上。
2. 远程只使用本 fork。需要上游提交时，另外 `git fetch` <https://github.com/go-vikunja/vikunja.git>。不要向上游开 PR。
3. 推送用普通 `git push`。不强推，不删除分支或历史，不改默认分支，不合并未获批准的 PR。
4. 测试使用进程内的内存 SQLite（`pkg/db/test.go` 在未设置 `VIKUNJA_TESTS_USE_CONFIG=1` 时如此）。不要把生产库路径或生产配置指给测试。
5. `frontend/embed.go` 使用 `//go:embed all:dist`。没有前端构建产物时，`mage` 的 `ensureFrontendDistExists` 会放一个空的 `frontend/dist/index.html`，该路径已被 `frontend/.gitignore` 忽略。这不是可用的前端包。

建议的最小命令（在隔离环境，且当前目录是仓库根）：

```bash
go install github.com/magefile/mage@v1.17.2
mage -l
mage test:filter TestSha256
```

前端需要 Node 24.21.0 和 pnpm 12.16.1，不能用 Node 22：

```bash
cd frontend
pnpm install --frozen-lockfile
pnpm vitest run src/helpers/ganttRelationArrows.spec.ts
```

`mage test:filter` 仍会编译几乎整个 API 模块，并在很多包里执行 `TestMain`。过滤名只减少实际断言，不把编译变成小工程。不要在 4GB 的生产主机上跑它。

## 本次真实验证

环境：云端隔离 VM，约 15GiB 内存，开始时可用约 5GiB，4 CPU。默认 `node` 是 v22.23.4（`/exec-daemon/node`），不满足 `frontend` 的 `>=24`。系统 `pnpm` 是 10.33.4，不是仓库锁定的 12.16.1。`mage` 起初未安装。Go 为 1.27.0。这些工具都装在 VM 里，没有装到迷你主机。

已通过：

1. `go install github.com/magefile/mage@v1.17.2`，随后 `mage -version` 为 Mage Build Tool v1.17.2。`mage -l` 退出码 0，能列出 `test:filter`、`test:web`、`build:build` 等目标。
2. 在仓库根执行 `GOMAXPROCS=2 GOMEMLIMIT=4GiB mage test:filter TestSha256`，输出写入 `/tmp/mage-test-sha256.log`。墙钟约 171 秒，退出码 0。日志中没有 `FAIL`。`code.vikunja.io/api/pkg/utils` 为 `ok`（0.004s，且不是 `[no tests to run]`），`TestSha256` 实际跑过。第二遍 `./pkg/webtests` 为 `ok` 且 `[no tests to run]`。两个 `go test` 命令都带 `-p 1 -timeout 45m -run TestSha256`；除 webtests 外带 `-short`。内存上限是这次 VM 可用内存不多时加的，不是仓库要求的参数。
3. `nvm install 24.21.0` 得到 Node v24.21.0。`corepack prepare pnpm@12.16.1 --activate` 后，`corepack pnpm -v` 为 12.16.1。在 `frontend/` 执行 `corepack pnpm install --frozen-lockfile`，退出码 0，约 5.4 秒，1372 个包，lockfile 通过 supply-chain 检查。
4. `corepack pnpm vitest run src/helpers/ganttRelationArrows.spec.ts`，输出写入 `/tmp/vitest-gantt-arrows.log`。Vitest v5.0.5：1 个文件通过，6 个测试通过，退出码 0。

未跑，也不把它们算作通过：

- `mage build:build`、`pnpm build`、完整的 `mage test:feature` / `mage test:web`、`mage test:e2e`
- `veans` 的构建和测试
- 任何针对生产服务、生产 SQLite 或真实账号的请求

Go 测试能编译，是因为本地有一个被 gitignore 的占位 `frontend/dist/index.html`。那不是前端生产构建。

## 需求候选

以下五项未经用户批准，不是实现范围。本文只记下，不设计界面，不改 API，不改数据库。

- 平板横屏大字与触控
- 生活与工作聚合视图
- 手机 agent 最小权限官方 API
- 录音待办去重
- 待核实状态

现有生活、工作两个项目及其待办 / 进行中 / 已完成，以及已录入的历史任务，都不在这次改动里。

## 建议与待决策

最多四条，都还不是实施任务。

1. 产品定制的稳定起点用上游 tag `v2.8.0`（`6e788bf87`），不要用当前 `main`。正在运行的官方镜像是 v2.8.0。`main` 多了 188 个未发版提交，并带有迁移差异。在 `main` 上做功能，等于同时带上一次未经安排的升级。`main` 继续只跟上游，不放自定义提交。本文件所在 PR 基于当前 `main`，只因为文档不改变运行中的程序；它没有代替这个选择。若选定 v2.8.0，以后的产品分支从该 tag 拉出，需要的话再把本文件 cherry-pick 过去。
2. 同步上游只做快进，不改写历史。给本机加上只用于 fetch 的 `upstream`（<https://github.com/go-vikunja/vikunja.git>）。确认 fork 的 `main` 没有独有提交后，快进到 `upstream/main` 并用普通 push 更新 fork。自定义永远在别的分支。要升级运行中的实例时，再把选定的上游 tag 合并进定制分支，并单独决定是否更换镜像和是否跑迁移。不改变 GitHub 默认分支，不强推，不删除历史，不向上游提 PR。fork 上目前没有 tag；若希望 `v2.8.0` 在 GitHub 上可见，可以只新增这个已存在的 tag 引用，仍然不要改 `main`。这次没有推 tag。
3. 不要在这台迷你主机上编译。机器是 i3-4330T，大约 4GB 内存，上面已经跑着官方镜像和 SQLite。本仓库要的是 Go 1.27、Node 24 和 pnpm 12；前端 lockfile 有 1372 个包；API 测试会编译带 CGO SQLite 的整个模块。本次云端 VM 有约 15GiB、可用约 5GiB，仅 `mage test:filter TestSha256` 就用了约 171 秒。在 4GB 且没有余量的机器上做同样的事，编译器和 Node 可能把内存吃满，系统为了回收内存可能停掉正在跑的 Vikunja，SQLite 写入中断的风险就落在生产库上。继续使用官方 ghcr 镜像。编译和测试放在更大的隔离环境，并且只用内存库或空库。
4. 如果下一轮只批准一件与存储无关的事，优先考虑平板横屏的大字和触控热区。它不需要新的任务状态，也不需要迁移。聚合视图、待核实状态、录音去重、手机 agent 权限都继续停在候选，直到单独批准。

需要用户决定：

- 最优先：后续产品定制从 `v2.8.0` 拉分支，还是从当前 `main` 拉分支？
- fork 的 `main` 是否定期快进上游，且仍然不部署、不升级数据库？
- 是否把上游 tag `v2.8.0` 以新增 tag 的方式推到本 fork，方便以后从 GitHub 上的 tag 开分支？
- 五项候选里，批准开发时先做哪一项？手机 agent 将来若使用官方 API，令牌是只读，还是允许创建任务？

## 非目标

- 不实现上面的需求候选，也不实现任何尚未点名批准的产品功能
- 不向上游 `go-vikunja/vikunja` 提 PR，不另建替代仓库
- 不改写 `main`，不强推，不删除分支或历史，不自动合并
- 不访问或修改正在运行的服务、SQLite、用户任务、token、密码或密钥
- 不部署，不替换镜像，不升级数据库，不改网络、端口或休眠，不产生费用
- 不接 Todos 双向同步，不实现短信采集，不上传私有录音或联系人
- 不修改上游 README、许可证和版权头
