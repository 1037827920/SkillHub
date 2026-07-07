# ponytail — 克制的资深工程师

> He says nothing. He writes one line. It works.

把一位"话不多、只写一行、但代码能跑"的资深工程师塞进你的 AI agent。核心是在动手写代码前，先沿着一条"少即是多"的阶梯做取舍，只保留任务真正需要的部分，从根上避免过度工程。

## 它解决什么问题

你让 agent 加一个日期选择器，它可能给你装上 flatpickr、写一个包装组件、再加一份样式表，顺便开始讨论时区。

用了 ponytail：

```html
<!-- ponytail: browser has one -->
<input type="date">
```

在真实 agent 场景下（headless Claude Code 编辑真实的 FastAPI + React 仓库，12 个功能任务）测得的效果：

| 对比无 skill 基线 | 代码量 | tokens | 成本 | 耗时 | 安全 |
| --- | --: | --: | --: | --: | --: |
| **ponytail** | **-54%** | **-22%** | **-20%** | **-27%** | **100%** |

代码量平均减少约 54%（最高 94%），是唯一一个"每项指标都降、且安全性仍保持 100%"的方案。收益最大的地方往往是本可以不写的过度实现（日期选择器从 400 多行降到 23 行，颜色选择器从 287 行降到 23 行，都是因为直接用了原生 `<input>`）；对本就精简的代码则几乎没有影响。

## 它怎么工作

在真正理解问题、读过改动所涉及的代码之后，停在第一条成立的阶梯上——

```
1. 这东西需要存在吗？   → 不需要：直接跳过（YAGNI）
2. 代码库里已经有了？   → 复用，别重写
3. 标准库能做？         → 用标准库
4. 平台原生能力能做？   → 用原生
5. 已装的依赖能做？     → 用依赖
6. 一行能搞定？         → 就写一行
7. 到这一步才：写能跑的最小实现
```

阶梯是在理解问题**之后**才走，而不是替代理解：它会先读相关代码、追清真实调用流程，再挑阶梯。对解决方案偷懒，对读代码从不偷懒。

关键前提是"懒但不失职"：信任边界的校验、数据丢失处理、安全和可访问性永远不在被砍的范围内。代码变小是因为它本就没必要那么大，而不是为了压行数。

## 安装

ponytail 是一个通过插件市场安装的插件，而不是放进目录即用的 skill。

> 前置：Claude Code 和 Codex 的插件会跑两个很小的 Node.js 生命周期 hook，所以 `node` 需要在 PATH 上（Nix/nvm 用户注意：要在非交互式 shell 的 PATH 里）。即使没有 `node`，skill 本身仍能用，只是"常驻激活"会安静地不生效而已。

### Claude Code

分两次发送以下两条命令（合在一条里发不生效）：

```
/plugin marketplace add DietrichGebert/ponytail
```

```
/plugin install ponytail@ponytail
```

桌面版没有 `/plugin` 命令，改从界面安装：Customize → personal plugins 旁边的 + → Create plugin and add marketplace → Add from repository，然后填入仓库地址。

### Codex

命令行：

```bash
codex plugin marketplace add DietrichGebert/ponytail
codex
```

然后打开 `/plugins`，选中 Ponytail 市场并安装；再打开 `/hooks`，检查并信任它的两个生命周期 hook，最后开一个新会话。

这套安装同样覆盖 Codex 桌面版：安装后重启 app 即可自动识别插件。

在 Codex 里，`/ponytail` 系列是以 skill 形式提供的，用 `@` 调用（例如 `@ponytail-review`）。

## 命令

| 命令 | 作用 |
| --- | --- |
| `/ponytail [lite \| full \| ultra \| off]` | 设置强度或关闭；不带参数则显示当前级别 |
| `/ponytail-review` | 审查当前 diff 的过度工程，给出一份"可删清单" |
| `/ponytail-audit` | 审查整个仓库的过度工程，而不只是 diff |
| `/ponytail-debt` | 把你用 `ponytail:` 标记推迟的取舍汇总成账本，避免"以后"变成"永远不做" |
| `/ponytail-gain` | 展示基准测试里量化的收益（更少代码、更低成本、更快速度） |
| `/ponytail-help` | 上述命令的快速参考 |

命令需要支持 skill 的宿主（Claude Code、Codex、Devin CLI、OpenCode、Gemini、pi、Swival、Hermes 等）。在 Codex 里它们是 skill，用 `@` 调用。`/ponytail ultra` 适用于"这套代码库让你个人蒙受了损失"的时刻。

## 配置

安装后每次会话常驻，默认强度为 `full`。可用以下任一方式改默认级别（`lite`/`full`/`ultra`/`off`）：

- 环境变量 `PONYTAIL_DEFAULT_MODE`
- `~/.config/ponytail/config.json` 里的 `defaultMode` 字段（Windows 为 `%APPDATA%\ponytail\config.json`）

都不设则用 `full`。配置文件不是必需的。

## 卸载

| 宿主 | 命令 |
| --- | --- |
| Claude Code | `/plugin remove ponytail` |
| Codex | `codex plugin remove ponytail` |

这些只删插件自身的文件，会留下少量 ponytail 写在插件目录之外的状态（mode 标记、`~/.config/ponytail/config.json`，以及你若接受过设置提示则还有 `~/.claude/settings.json` 里的一条 statusLine）。要连这些一起清掉，运行 `node scripts/uninstall.js`——注意要在执行上面的宿主卸载命令**之前**跑，因为脚本本身就是插件文件，先删插件会把它一起删掉。

## 更多

官方仓库（含更多接入方式、基准测试细节与复现方法）：https://github.com/DietrichGebert/ponytail

