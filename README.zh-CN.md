<p align="center">
  <a href="https://pi.dev">
    <img alt="pi logo" src="https://pi.dev/logo-auto.svg" width="128">
  </a>
</p>
<p align="center">
  <a href="https://discord.com/invite/3cU7Bz4UPx"><img alt="Discord" src="https://img.shields.io/badge/discord-community-5865F2?style=flat-square&logo=discord&logoColor=white" /></a>
  <a href="https://www.npmjs.com/package/@earendil-works/pi-coding-agent"><img alt="npm" src="https://img.shields.io/npm/v/@earendil-works/pi-coding-agent?style=flat-square" /></a>
</p>

> 新贡献者提交的 issue 和 PR 默认会被自动关闭。维护者会每日审阅被自动关闭的 issue。详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

[English README](README.md)

# Pi Agent Harness

这是 Pi agent harness 项目的主仓库，包含可自扩展的编程 agent。

* **[@earendil-works/pi-coding-agent](packages/coding-agent)**：交互式编程 agent CLI
* **[@earendil-works/pi-agent-core](packages/agent)**：支持工具调用与状态管理的 agent 运行时
* **[@earendil-works/pi-ai](packages/ai)**：统一的多厂商 LLM API（OpenAI、Anthropic、Google 等）

了解更多 Pi 相关内容：

* 访问 [pi.dev](https://pi.dev)，项目官网（含演示）
* 阅读 [文档](https://pi.dev/docs/latest)，也可以直接让 agent 解释自身

## 所有包

| 包 | 说明 |
|---------|-------------|
| **[@earendil-works/pi-ai](packages/ai)** | 统一的多厂商 LLM API（OpenAI、Anthropic、Google 等） |
| **[@earendil-works/pi-agent-core](packages/agent)** | 支持工具调用与状态管理的 agent 运行时 |
| **[@earendil-works/pi-coding-agent](packages/coding-agent)** | 交互式编程 agent CLI |
| **[@earendil-works/pi-tui](packages/tui)** | 支持差分渲染的终端 UI 库 |

Slack/聊天自动化与工作流见 [earendil-works/pi-chat](https://github.com/earendil-works/pi-chat)。

## 权限与容器化

Pi 没有内置权限系统来限制文件系统、进程、网络或凭据访问。默认情况下，它以启动它的用户和进程的权限运行。

若需要更强的隔离边界，请容器化或沙箱化 Pi。三种模式见 [packages/coding-agent/docs/containerization.md](packages/coding-agent/docs/containerization.md)：

- **Gondolin 扩展**：在宿主机保留 `pi` 与厂商认证，将内置工具和 `!` 命令路由到本地 Linux 微虚拟机。
- **纯 Docker**：在本地容器中运行整个 `pi` 进程，实现简单隔离。
- **OpenShell**：在策略控制的沙箱中运行整个 `pi` 进程。

## 贡献

贡献指南见 [CONTRIBUTING.md](CONTRIBUTING.md)，项目规则（面向人类与 agent）见 [AGENTS.md](AGENTS.md)。Pi 的长期规划见 [RFCs](https://rfc.earendil.com/keyword/pi/)。

## 开发

```bash
npm install --ignore-scripts  # 安装所有依赖，不运行生命周期脚本
npm run build         # 刷新模型数据，然后构建所有包
npm run build:offline # 使用现有模型数据离线构建，无需网络
npm run check         #  lint、格式化与类型检查
./test.sh            # 运行测试（无 API key 时会跳过依赖 LLM 的测试）
./pi-test.sh         # 从源码运行 pi（可在任意目录执行）
```

## 从发布源码构建独立二进制

GitHub 发布包含带版本号的源码归档，其 `SHA256SUMS` 文件覆盖该归档。解压后运行与官方独立二进制相同的构建脚本：

```bash
VERSION="<release-version>"
tar -xzf "pi-${VERSION}-source.tar.gz"
cd "pi-${VERSION}"
./scripts/build-binaries.sh --offline-model-data --platform linux-x64 --out "$PWD/out"
```

源码归档包含发布时生成的厂商模型数据。`--offline-model-data` 使用该快照构建，而非从在线厂商目录刷新。脚本仍会安装依赖、构建 monorepo、编译 Bun 可执行文件并准备运行时资源。单独提供依赖的包维护者可传入 `--skip-install --skip-deps`。

## 供应链加固

我们将 npm 依赖变更视为需审阅的代码变更。

- 直接外部依赖固定为精确版本；内部 workspace 包仍使用版本范围。
- `.npmrc` 设置 `save-exact=true` 和 `min-release-age=2`，避免 npm 解析时采用当日发布的依赖。
- `package-lock.json` 是依赖的权威来源。pre-commit 会阻止意外提交 lockfile，除非设置 `PI_ALLOW_LOCKFILE_CHANGE=1`。
- `npm run check` 会校验固定的直接依赖、原生 TypeScript 导入兼容性，以及生成的 coding-agent shrinkwrap。
- 发布的 CLI 包包含 `packages/coding-agent/npm-shrinkwrap.json`，由根 lockfile 生成，为 npm 用户固定传递依赖。
- 发布冒烟测试使用 `npm run release:local` 在仓库外构建、打包并创建隔离的 npm 与 Bun 安装，然后再打 tag 发布。
- 本地发布安装、文档中的 npm 安装，以及 `pi update --self` 在支持时使用 `--ignore-scripts`。
- CI 使用 `npm ci --ignore-scripts` 安装；定时 GitHub workflow 运行 `npm audit --omit=dev` 和 `npm audit signatures --omit=dev`。
- shrinkwrap 生成对依赖生命周期脚本有显式白名单；新的生命周期脚本依赖在未审阅前会导致检查失败。

## 分享你的开源编程 agent 会话

若你用 Pi 或其他编程 agent 做开源工作，请分享你的会话。

公开的开源会话数据有助于用真实任务、工具使用、失败与修复来改进编程 agent，而非玩具基准。

完整说明见 [X 上的这篇帖子](https://x.com/badlogicgames/status/2037811643774652911)。

发布会话请使用 [`badlogic/pi-share-hf`](https://github.com/badlogic/pi-share-hf)。阅读其 README.md 了解配置。你只需要 Hugging Face 账号、Hugging Face CLI 和 `pi-share-hf`。

也可观看 [这段视频](https://x.com/badlogicgames/status/2041151967695634619)，演示如何发布 `pi-mono` 会话。

我定期将 `pi-mono` 工作会话发布在：

- [badlogicgames/pi-mono on Hugging Face](https://huggingface.co/datasets/badlogicgames/pi-mono)

## 许可证

MIT

<p align="center">
  <a href="https://pi.dev">pi.dev</a> 域名由
  <br /><br />
  <a href="https://exe.dev"><img src="packages/coding-agent/docs/images/exy.png" alt="Exy mascot" width="48" /><br />exe.dev</a>
  慷慨捐赠
</p>
