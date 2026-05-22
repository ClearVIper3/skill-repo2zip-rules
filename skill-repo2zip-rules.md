---
description: Skill 仓库打包成 zip 时的禁入文件规则。当用户要求"打包 skill / 生成 skill zip / 发布 skill / build skill / package skill"时 MUST 遵守。
alwaysApply: false
---

# Skill ZIP 打包规则

## 适用场景
当且仅当任务是「把一个 skill 仓库打包为可分发的 `*.zip`」时，遵循本规则。
其他场景（开发、调试、提交 git）**不**适用。

## R1 · 打包方式
- ✅ MUST 用「显式白名单」方式打包：仅传入 R3 列出的路径
  - PowerShell：`Compress-Archive -Path <R3 路径列表> -DestinationPath ..\<name>.zip`
  - bash：`zip -r ../<name>.zip <R3 路径列表> -x '**/__pycache__/*' '**/*.pyc' '**/.DS_Store'`
- ❌ NEVER 使用 `git archive HEAD`：会带入仓库中所有 git 跟踪的治理类文件，违反 R2.8
- ❌ NEVER 使用 `Compress-Archive -Path .\*` 或 `zip -r ... .`：会带入未跟踪文件
- ❌ NEVER 用资源管理器右键"压缩"

## R2 · 禁入文件清单（NEVER include）
打包结果中 NEVER 出现以下任意条目；命中即 MUST 中止打包并报错：

### R2.1 VCS 与 CI 元信息
- `.git/`、`.gitignore`、`.gitattributes`
- `.github/`、`.gitlab/`、`.gitlab-ci.yml`
- `.husky/`、`.pre-commit-config.yaml`

### R2.2 凭证与密钥（红线，泄漏需立即旋转）
- 任意 `.env*`（`.env.example` 也禁止，用 `env.template` 代替）
- `*.pem`、`*.key`、`*.p12`、`*.pfx`、`id_rsa*`、`id_ed25519*`
- `*credentials*`、`*secret*`、`*token*`、`*.netrc`
- 任何文件内含正则匹配项：
  - `(?i)(password|passwd|pwd)\s*[:=]\s*["']?[^\s"']+`
  - `-----BEGIN (RSA |EC |OPENSSH |DSA )?PRIVATE KEY-----`
  - `gh[ps]_[A-Za-z0-9]{36,}`（GitHub PAT）
  - `sk-[A-Za-z0-9]{20,}`（OpenAI / Anthropic key）
  - `xox[baprs]-[A-Za-z0-9-]{10,}`（Slack token）

### R2.3 真实业务数据
- `*.eml`、`*.mbox`、`*.msg`（真实邮件）
- `inbox.json`、`messages.json`、`contacts.*`
- 任何含真实用户邮箱、手机号、身份证号、姓名的文件
- skill 自测产生的输出目录：`out/`、`output/`、`downloads/`、`attachments/`

### R2.4 构建/运行产物
- `__pycache__/`、`*.pyc`、`*.pyo`、`.pytest_cache/`、`.mypy_cache/`、`.ruff_cache/`
- `node_modules/`、`dist/`、`build/`、`.next/`、`.turbo/`
- `venv/`、`.venv/`、`env/`、`.tox/`
- `*.log`、`*.tmp`、`*.swp`、`*.lock`（除 `requirements.txt` / `package-lock.json` 等显式声明的依赖锁）

### R2.5 IDE 与 OS
- `.vscode/`、`.idea/`、`*.iml`、`.cursor/`、`.zed/`
- `.DS_Store`、`Thumbs.db`、`desktop.ini`、`$RECYCLE.BIN/`

### R2.6 个人临时与历史文件
- `*_smoketest*`、`*_test_local*`、`scratch*`、`tmp_*`
- `*.bak`、`*.old`、`*.orig`、`*~`
- 旧版 zip 套娃：`*-v[0-9]*.zip`、`*-old.zip`

### R2.7 内部草稿文档
- `*-完善报告.md`、`*-review.md`、`*-todo.md`、`MEETING*.md`
- `NOTES.md`、`SCRATCH.md`、`personal-*.md`

### R2.8 仓库治理类文件（仅人类/平台可见，agent 不读）
原则：**zip 是 agent 的运行包，不是仓库镜像。** 凡是给「人」或「git 平台」看的文件，一律禁入。
- `README.md`、`README.*`、`CHANGELOG.md`、`HISTORY.md`、`RELEASES.md`
- `LICENSE`、`LICENSE.*`、`COPYING`、`NOTICE`、`AUTHORS`、`CONTRIBUTORS`
- `CONTRIBUTING.md`、`CODE_OF_CONDUCT.md`、`SECURITY.md`、`SUPPORT.md`、`MAINTAINERS*`
- `CITATION.cff`、`FUNDING.yml`、`.zenodo.json`
- 依赖锁与构建配置：`requirements.txt`、`requirements-*.txt`、`Pipfile*`、`poetry.lock`、`pyproject.toml`、`setup.py`、`setup.cfg`、`package.json`、`package-lock.json`、`yarn.lock`、`pnpm-lock.yaml`、`Makefile`、`Dockerfile`、`docker-compose*.yml`、`tsconfig*.json`、`.editorconfig`、`.eslintrc*`、`.prettierrc*`、`pytest.ini`、`tox.ini`
- 徽章/截图等仅 README 引用的资源：`badges/`、`screenshots/`、`docs/images/`（除非脚本运行时真实使用）

> 例外：若 skill 运行时 MUST 读取依赖清单，重命名为 `runtime-deps.txt` 之类非标准名并在 SKILL.md 中显式声明用途。skill 的依赖 MUST 由 SKILL.md 文字描述（"需要 Python 3.10+ 与 imapclient"），NEVER 通过 `requirements.txt` 让人去 `pip install`。

## R3 · 必入文件清单（MUST include）
zip 内 MUST 只包含 agent 运行 skill 所必需的文件：
- `SKILL.md`（skill 入口与全部使用说明）
- `scripts/`（可执行脚本，MUST 不含 `__pycache__` 等产物）
- `references/`（**运行时**被 SKILL.md / 脚本引用的参考资料）
- `assets/`（**运行时**被脚本读取的静态资源，如模板、prompt 片段）

未在上述白名单中的文件 NEVER 加入 zip。

判定标准：「**这个文件不在 zip 里，agent 还能正确执行 skill 吗？**」
- 能 → NEVER 入 zip（属于仓库治理，留在 git 仓库即可）
- 不能 → MUST 入 zip

## R4 · 结构约束
- 顶层目录名 = skill 名（如 `exmail-io/`），zip 解压后只产生这一个目录
- 路径分隔符使用 `/`，禁止 `\`
- 文本文件行结束符 LF，禁止 CRLF
- 不含空目录
- 单个 zip 体积 ≤ 2 MB；超出需先排查 R2.3 / R2.4

## R5 · 打包前置检查
按顺序执行，任一失败 MUST 中止：

1. `git status` MUST clean（无 untracked / modified）
2. 扫描 R2.2 凭证正则，命中数 MUST 为 0
3. 扫描 R2 全量模式，命中数 MUST 为 0
4. 按 R1 白名单方式生成 zip
5. 解压到临时目录，再次执行步骤 3，命中数 MUST 为 0

## R6 · 泄漏应急（命中 R2.2 后）
1. **立即旋转**所有泄漏的凭证（改密码、吊销 token）
2. 用 `git filter-repo` 或 BFG 从 git 历史中清除
3. `git push --force` 覆盖远端（仅此场景允许 force push）
4. 在本规则 R2.2 中补充对应正则，防止复发

## 反例对照

❌ NEVER：`Compress-Archive -Path .\<skill>\* -Destination <skill>.zip`
理由：会把 `__pycache__/` / `.env` / `README.md` / `LICENSE` / 本地 `_smoketest.py` 一并带入。

❌ NEVER：`git archive --format=zip --prefix=<skill>/ -o ..\<skill>.zip HEAD`
理由：会带入仓库根所有 git 跟踪的 `README.md` / `LICENSE` / `requirements.txt` / `.gitignore`，违反 R2.8。

✅ MUST：按 R1 显式白名单。命令模式见 R1（Compress-Archive 或 zip 仅传 R3 路径列表）。

❌ NEVER：`config/.env.example` 作为占位（仍匹配 `.env*`，触发 R2.2）。
✅ MUST：改名为 `config/env.template`，内容用占位符。

## 备注
- `.gitignore` 解决"git 不跟踪"，本规则解决"zip 不包含"，两者职责不重合，不能互相替代。
- 本规则与 skill 内容无关，适用于任何 skill 仓库。
