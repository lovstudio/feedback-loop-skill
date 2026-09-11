# 反馈飞轮 · Feedback Flywheel

![version](https://img.shields.io/badge/version-0.2.0-blue)
![distribution](https://img.shields.io/badge/distribution-encrypted%20paid-blueviolet)

把用户对 Agent 的情绪与判断变成可统计的反馈事件，再按最小作用域迭代系统提示词、
reference 与 Skill。被动情绪（“还是不行”“太失望了”）与主动 judge（`👍`、`👎`、
评分）走同一条闭环。

本仓库是**加密付费 Skill 的私有真源**：`src/` 是真实内容，`public/` 是激活前的占位与密文包，
目录布局与 `skill-forge` 一致。

## 结构

```text
feedback-loop-skill/
├── src/                         # 加密交付的真实内容（打包源）
│   ├── SKILL.md
│   ├── skill.yaml
│   ├── references/protocol.md   # 判定、作用域、状态机
│   ├── references/store-schema.md
│   ├── references/host-adapters.md
│   ├── references/system-prompt.md
│   ├── assets/system-prompt-block.md
│   └── scripts/
│       ├── feedback_store.py    # 账本：append / update / list / stats / verify
│       ├── scan_session.py      # 会话回扫
│       ├── report.py            # 满意度报告
│       ├── install_feedback_loop.py
│       └── profile_store.py
├── public/                      # 占位 SKILL.md + 密文包（发布产物）
├── dist/                        # 打包产物（gitignored）
├── cases/cases.json             # 真实用例
├── references/skill-composition.md
├── skill-card.yaml / skill-card.md
└── pricing-card.yaml            # 定价依据（公开价格在 catalog）
```

发布时 `src/` 整体加密为 `dist/`，再镜像到 `public/` 与
`lovstudio/skills` 的 `skills/feedback-loop/`。

## 用户安装

```bash
npx lovstudio skills add feedback-loop
```

付费 Skill 需要登录并确认 Credits 兑换；加密包安装后由 `lovstudio-skill-helper` 按需解密：

```bash
uvx lovstudio-skill-helper decrypt feedback-loop                  # 读取真实 SKILL.md
uvx lovstudio-skill-helper decrypt feedback-loop references/protocol.md
uvx lovstudio-skill-helper exec feedback-loop scripts/feedback_store.py stats --since 30d
```

## 维护命令

```bash
# 1) 改 src/ 内容，升 src/SKILL.md 顶层 version
# 2) 打包并自检
skill-forge pack . --key <64位十六进制>
skill-forge verify . --key <同一个 key>

# 3) 注册到授权服务并更新 catalog（私有仓库 + 加密包）
#    发布脚本见 skill-forge publish，或按 PUBLISHER 流程手动执行
```

## 质量门禁

- 发布前：`python3 ../skill-publisher-skill/scripts/validate_skill.py . --target source`
  （在扁平布局下运行）与 `skill-forge verify` 全量回读通过。
- 发布后：catalog 条目、官网详情页与兑换/安装链路逐项回读。

## 依赖

- Python 3.8+，仅标准库；`scripts/validate_skill.py` 需要 PyYAML。
- 运行时账本默认在 `~/.feedback-loop/`，只写本地磁盘。
