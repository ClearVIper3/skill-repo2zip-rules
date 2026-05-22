# skill-repo2zip-rules

> Hard rules for packaging an Anthropic-style **skill repo** into a distributable **`.zip`**.
> 把一个 skill 仓库打包成可分发 zip 时必须遵守的硬性规则。

[![rules](https://img.shields.io/badge/type-hard%20rules-red)](#)
[![scope](https://img.shields.io/badge/scope-skill%20packaging-blue)](#)
[![violation](https://img.shields.io/badge/any%20violation-MUST%20abort-critical)](#)

---

## 这是什么

一份**面向 agent / AI 自动打包流程**的硬约束规则集。
核心理念：

> **zip 是 agent 的运行包，不是仓库镜像。**
> 凡是给「人」或「git 平台」看的文件，一律不进 zip。

适用场景：当任务是 *"把一个 skill 仓库打包为可分发的 `*.zip`"* 时。
其他场景（开发、调试、提交 git）**不**适用。

## 规则速览

| 编号 | 主题 | 一句话概括 |
|---|---|---|
| **R1** | 打包方式 | MUST 用显式白名单；NEVER 用 `git archive` / `Compress-Archive .\*` |
| **R2** | 禁入文件清单 | VCS / 密钥 / 业务数据 / 构建产物 / IDE / 临时文件 / 仓库治理文件 一律禁入 |
| **R3** | 必入文件清单 | 只包含 `SKILL.md` + `scripts/` + `references/` + `assets/` |
| **R4** | 结构约束 | 顶层目录 = skill 名；LF；无空目录；≤ 2 MB |
| **R5** | 前置检查 | git clean → 扫凭证 → 扫禁入项 → 打包 → 解压复扫 |
| **R6** | 泄漏应急 | 立即旋转凭证 → filter-repo 清史 → force push → 补正则 |

## 如何使用

1. 把 [`skill-repo2zip-rules.md`](./skill-repo2zip-rules.md) 作为 cursor / claude-code / codebuddy 的 rule 文件加载。
2. 在你给 agent 的打包指令里引用这份规则：

   > 打包前 MUST 加载并遵守 `skill-repo2zip-rules`。任一规则违反 → 中止打包并报错。

3. 推荐配合**前置检查脚本**使用（按 R5 顺序执行五步）。

## 判定原则

> **「这个文件不在 zip 里，agent 还能正确执行 skill 吗？」**
>
> - 能 → NEVER 入 zip（仓库治理，留在 git 仓库即可）
> - 不能 → MUST 入 zip

## 与 `.gitignore` 的关系

| | 解决问题 | 作用范围 |
|---|---|---|
| `.gitignore` | git 不跟踪 | 仓库 |
| 本规则 | zip 不包含 | 分发包 |

**两者职责不重合，不能互相替代。**

## 反例（最常见错误）

```powershell
# ❌ 把整个目录都压进去，会带入 __pycache__ / .env / README / LICENSE
Compress-Archive -Path .\<skill>\* -Destination <skill>.zip

# ❌ 看似干净，但会带入所有 git 跟踪的治理文件
git archive --format=zip --prefix=<skill>/ -o ..\<skill>.zip HEAD
```

正确做法见规则文件 R1。

## 贡献

新增禁入模式时，请同时：

1. 在 R2.x 对应小节追加条目
2. 给出一个真实泄漏 / 误打包的 case（脱敏）
3. 如属于凭证类 → 同时更新 R2.2 正则

## License

规则文本以 CC0 形式公开，欢迎在你的 skill 仓库里直接引用或裁剪。
