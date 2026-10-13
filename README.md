# make-knowledge-cards

一个遵循 [Agent Skills](https://agentskills.io) 开放标准的开源技能：把一篇文章提炼成 5~8 张结构化的 Markdown 知识卡片。

## 1. 解决什么问题

读完一篇长文章，常见的困境是：

- **摘要不够用**：文章摘要只是压缩了原文，知识点仍然挤在同一段里，复习时抓不住单个概念；
- **手写卡片太耗时**：手工做学习卡要先判断哪些内容值得记、再逐张整理，一篇文章往往要半小时以上；
- **AI 总结容易注水**：直接让模型"总结成几条"，经常会拆碎观点凑数量、把"多数研究发现"写成"研究证明"，甚至补进原文没有的信息。

make-knowledge-cards 针对的就是这三点：由 Agent 提取**真正重要的知识点**，一张卡只讲一个知识点，去重，不编造，信息不足时如实少出卡。

## 2. 主要功能

| 能力 | 说明 |
|---|---|
| 输入读取 | 对话中粘贴的文章正文，或本地 `.md` / `.txt` 文件 |
| 知识提取 | 只提取概念定义、可操作规则、有数据支撑的结论、易混淆区别、反直觉事实；剔除铺垫、作者自述、重复表述、纯态度观点 |
| 卡片生成 | 每张卡含四个字段：**标题**、**核心知识**（一句话结论）、**简明解释**（2~4 句）、**例子 / 自测问题**（二选一，不混排） |
| 数量控制 | 目标 5~8 张；超过 8 个知识点按重要度取前 8 并列出被舍弃主题；**不足 5 个按实际数量输出**，绝不拆碎凑数 |
| 忠实约束 | 每个事实性表述必须能回溯原文；引用数字与限定词保持原口径；补充构造的例子必须标注"示意例子（非原文提供）" |
| 语言跟随 | 输出语言跟随原文：中文文章出中文卡，英文文章自动改用 `Core idea / Explanation / Example / Self-test` 等英文标签 |
| 去重与自检 | 交付前逐项检查：一卡一知识点、无重复卡、无编造、例子与知识点严格对应 |

**暂不支持**：网页抓取、PDF 解析、Word 文档、图片输入、Anki 导出、图形界面/图片卡片渲染。遇到这些输入，技能会明确拒绝并提示用户改用其他工具，不会猜测内容。

## 3. 安装方法

技能就是一个文件夹，复制即用：

1. 把本仓库的 `skills/make-knowledge-cards/` 整个目录复制到你的 Agent 的技能目录，例如：

   ```bash
   # Claude Code（用户级）
   cp -r skills/make-knowledge-cards ~/.claude/skills/

   # 或 Claude Code（项目级，随仓库共享）
   cp -r skills/make-knowledge-cards .claude/skills/
   ```

   其他支持 Agent Skills 标准的产品（Codex、OpenClaw 等），放到各自文档指定的 skills 位置即可。

2. 重启 Agent 或刷新技能列表，确认列表中出现 `make-knowledge-cards`。

安装前可选跑一次结构校验（依赖 Python 和 PyYAML）：

```bash
pip install pyyaml
python quick_validate.py skills/make-knowledge-cards
# 期望输出：Skill is valid!
```

## 4. 使用方法

安装后直接用自然语言触发，无需固定命令：

```text
把这篇文章做成知识卡片：<粘贴文章正文>
```

指定本地文件并要求保存：

```text
用 make-knowledge-cards 处理 C:\docs\notes.md，把卡片保存成一个文件
```

要求英文版输出（不说明时按原文语言自动匹配）：

```text
把这篇文章做成知识卡片，卡片正文用英文
```

触发词示例：做知识卡片、整理成卡片、读书笔记卡片、学习卡、记忆卡，或直接粘贴长文说"帮我提炼成几张卡片"。

## 5. 输入输出示例

**输入**（文章片段）：

> 咖啡因并不能消除腺苷，它只是堵住了腺苷受体，让你暂时感觉不到困意。等咖啡因代谢掉，积压的腺苷会一次性涌上来。

**输出**（其中一张卡片，完整输出含首行"来源"标注）：

```markdown
### 卡片：咖啡因不消除困意

**核心知识**：咖啡因只是阻断腺苷受体，并不减少脑内腺苷。

**简明解释**：腺苷白天不断积累，浓度越高越困。咖啡因堵住受体后困意"感觉不到"而非消失；咖啡因代谢完，积压的腺苷一次性起作用，清醒是要还的。

**例子 / 自测问题**：为什么下午喝咖啡，晚上反而更难睡？
```

**英文版输出**（同一技能处理英文文章时的卡片样式）：

```markdown
### Card: Caffeine Doesn't Remove Sleep Pressure

**Core idea**: Caffeine only blocks adenosine receptors; it does not reduce adenosine in the brain.

**Explanation**: Adenosine accumulates through the day — the more of it, the sleepier you feel. Caffeine masks the signal instead of clearing it, so the built-up adenosine hits back once it wears off.

**Example / Self-test**: Why does an afternoon coffee often make the following night's sleep worse?
```

**信息不足时**：对只有 2 个独立知识点的短文，技能输出 2 张卡并注明"原文可提取的独立知识点有限"，而不是硬凑到 5 张。

## 项目结构

```text
.
├── README.md
├── quick_validate.py                        # 官方结构校验脚本（副本）
└── skills/make-knowledge-cards/
    ├── SKILL.md                             # 工作流程、卡片模板、硬性约束
    └── agents/
        └── openai.yaml                      # 界面元数据（展示名、简介、默认提示词）
```

## 开发背景

- 使用 openai/skills 官方仓库的 `init_skill.py` 初始化骨架，`agents/openai.yaml` 由官方 `generate_openai_yaml.py` 生成；
- 用三类文章实测迭代：信息量足的中文科普文（出 7 卡）、英文技术文（英文标签输出）、极短观点文（如实只出 2 卡），并据此修正了数量规则、英文标签、例子/自测混排等歧义；
- 最终通过官方 `quick_validate.py` 校验：`Skill is valid!`

## 许可证

MIT
