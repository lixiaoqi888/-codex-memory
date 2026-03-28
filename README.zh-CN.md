# codex-memory 中文说明

`codex-memory` 是一个面向 Codex Desktop 的本地 memory sidecar。它会读取
`~/.codex/state_*.sqlite` 和 `~/.codex/sessions/**/*.jsonl`，然后把可检索的
记忆索引写到本地。

英文说明见：
[README.md](./README.md)

## 这项目和 claude-mem 是什么关系

这项目是**明确参考了**
[thedotmack/claude-mem](https://github.com/thedotmack/claude-mem)
的设计思路，主要参考点包括：

- 跨会话持久记忆
- 围绕生命周期事件的 hook 化接入方式
- 会话开始时自动注入历史上下文
- 把对话压缩成 task / result / decision / observation 这类高价值记忆

但这里也要写清楚边界，避免误解：

- `codex-memory` **不是** `claude-mem` 的官方版本
- `codex-memory` **不是** Claude Code 插件
- `codex-memory` **不是** Codex 官方 hook 实现
- 这个仓库面向的是 **Codex Desktop 当前暴露出来的本地 transcript / state 文件**
- 这里出现的 `SessionStart / UserPromptSubmit / PostToolUse / Stop / SessionEnd`
  这些事件名，是为了做兼容风格的运行时表面，不代表和 Claude Code 的官方 hook
  机制一模一样

一句话讲明白：

`codex-memory` 是一个**受 claude-mem 启发、但针对 Codex Desktop 本地运行环境重新实现的 memory 方案**。

## 为什么要特别写这个

因为如果不写清楚，别人很容易误会成：

- 这是 `claude-mem` 官方移植版
- 这是 Claude Code / Codex 官方支持的原生 hook
- 这是和上游完全一致的实现

实际上都不是。

更准确的说法应该是：

- **参考了 `claude-mem` 的产品思路和交互方式**
- **在 Codex Desktop 上做了本地化、适配性的实现**
- **尽量对齐体验，但不宣称官方等价**

## 当前这版做到了什么

- 真 embedding：默认本地 `fastembed`
- 真向量库：本地 `Qdrant`
- SQLite + FTS5 + Qdrant 混合检索
- `hook` 命令暴露 Claude 风格事件名
- `watch` 自动监听最近线程与 rollout 事件
- macOS `launchd` 自启动
- 后台 runtime 会把 bundle 和依赖一起装进 `~/.codex`
- watcher 退出时可补发 `SessionEnd`

## 当前没法假装成什么

目前 **Codex 没有公开的官方 hook API**，所以这套方案本质上仍然是：

- 本地 shim
- watcher 推断事件
- 再把 memory 结果注入 / 落盘

所以它能做到“体验上尽量接近”，但不能诚实地宣称“和官方 Claude Code hook 完全一样”。

## 常用命令

```bash
./codex-memory status
./codex-memory search "claude-mem" --cwd /Users/alex/Desktop/dev
./codex-memory hook SessionStart --cwd /Users/alex/Desktop/dev
./codex-memory watch --cwd /Users/alex/Desktop/dev
./codex-memory autostart install --cwd /Users/alex/Desktop/dev --load
```

## 致谢

- 主要参考项目：
  [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem)
- 本仓库的目标不是替代上游，而是把类似的 memory 体验带到 Codex Desktop 当前可用的本地运行环境里
