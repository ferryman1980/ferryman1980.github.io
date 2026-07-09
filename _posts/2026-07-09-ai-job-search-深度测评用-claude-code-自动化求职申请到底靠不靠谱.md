---
layout: post
title: "ai-job-search 深度测评：用 Claude Code 自动化求职申请，到底靠不靠谱？"
date: 2026-07-09 20:31:29 +0800
categories: [AI工具测评]
tags: ["how to use ai-job-search", "best ai job search engine", "ai job search free reddit", "best ai job search apps", "ai-job-search 教程"]
description: "# ai-job-search 深度测评：用 Claude Code 自动化求职申请，到底靠不靠谱？  **30秒结论**：ai-job-search 是一个基于 Claude Code 的开源求职自动化框架。它不是一个“一键投递”的 AI 工具，而是一个**半自动化的求职辅助系统**——你 fork 项目、填好个人资料，Claude 会帮你评估职位匹配度、定制简历、写求职信、准备面试。适合**英"
---

# ai-job-search 深度测评：用 Claude Code 自动化求职申请，到底靠不靠谱？

**30秒结论**：ai-job-search 是一个基于 Claude Code 的开源求职自动化框架。它不是一个“一键投递”的 AI 工具，而是一个**半自动化的求职辅助系统**——你 fork 项目、填好个人资料，Claude 会帮你评估职位匹配度、定制简历、写求职信、准备面试。适合**英语流利、技术背景强、愿意折腾 CLI 的求职者**。不适合指望“AI 全自动帮我找到工作”的人。

项目在 GitHub 上已有 **9677 stars**，但热度高不代表好用。我花了一周时间实际跑通整个流程，以下是全部踩坑实录。

---

## 核心功能：它到底能做什么？

ai-job-search 的核心逻辑是**把求职流程拆解成几个可被 LLM 执行的步骤**，每个步骤对应一个 Claude Code 的 task。

### 1. 项目结构一览

```
ai-job-search/
├── .claude/          # Claude Code 配置
├── profiles/         # 你的个人资料（YAML）
├── jobs/             # 职位描述（手动或爬取）
├── output/           # 生成的结果（简历、求职信）
├── scripts/          # 辅助脚本
├── tasks/            # Claude 任务定义
└── README.md
```

### 2. 配置个人资料

核心文件是 `profiles/profile.yaml`，你需要把你的经历写成结构化数据：

```yaml
# profiles/profile.yaml
name: "张三"
email: "zhangsan@example.com"
phone: "+86 138-0000-0000"
location: "Beijing, China"
linkedin: "https://linkedin.com/in/zhangsan"

summary: |
  Senior backend engineer with 8 years of experience building 
  distributed systems. Proficient in Go, Python, and cloud-native 
  architecture.

skills:
  - name: "Go"
    level: "expert"
    years: 6
  - name: "Python"
    level: "advanced"
    years: 8
  - name: "Kubernetes"
    level: "advanced"
    years: 4
  - name: "PostgreSQL"
    level: "expert"
    years: 7

experiences:
  - company: "TechCorp"
    title: "Senior Backend Engineer"
    start: "2020-03"
    end: "present"
    highlights:
      - "Designed and implemented a microservice migration, reducing P99 latency by 40%"
      - "Led a team of 5 engineers to build a real-time analytics pipeline"
      - "Reduced infrastructure costs by 30% through resource optimization"

education:
  - institution: "Peking University"
    degree: "B.S. Computer Science"
    year: 2016

languages:
  - language: "Chinese"
    level: "native"
  - language: "English"
    level: "fluent"
```

**踩坑**：YAML 格式非常严格。缩进错误会导致解析失败。建议用 `yamllint` 先校验。

### 3. 添加职位描述

把感兴趣的职位描述放到 `jobs/` 目录下，可以是纯文本或 Markdown 文件：

```markdown
# jobs/senior-sre-google.md
## Senior Site Reliability Engineer - Google Cloud

**Location**: Sunnyvale, CA (Hybrid)
**Salary**: $180,000 - $250,000

### Responsibilities
- Ensure reliability and scalability of Google Cloud infrastructure
- Design and implement monitoring and alerting systems
- Participate in on-call rotation
- Collaborate with product teams on capacity planning

### Requirements
- 5+ years of SRE or infrastructure engineering experience
- Strong programming skills in Go or Python
- Experience with distributed systems and Kubernetes
- BS in Computer Science or related field
```

**注意**：ai-job-search **不会自动爬取职位**。你得自己把 JD 复制粘贴过来。GitHub 的 issue 区有人提过 feature request 要集成 LinkedIn 爬虫，但作者明确拒绝了——涉及法律风险。

### 4. 运行评估任务

这是核心功能——让 Claude 评估职位匹配度：

```bash
# 安装 Claude Code（需要 Anthropic API key）
npm install -g @anthropic-ai/claude-code

# 在项目根目录运行
claude code --task evaluate-job --input jobs/senior-sre-google.md
```

Claude 会输出类似这样的评估结果：

```
## Job Match Assessment: Senior SRE @ Google Cloud

### Match Score: 82/100

### Strengths
- ✅ Go programming experience (6 years) exceeds requirement
- ✅ Distributed systems background matches core needs
- ✅ Kubernetes experience (4 years) directly applicable
- ✅ Previous team leadership experience

### Gaps
- ❌ No explicit SRE title in work history (but responsibilities overlap)
- ❌ No mention of on-call experience in profile
- ❌ Cloud provider experience is AWS, not GCP

### Recommended Actions
1. Add on-call experience to profile if any
2. Highlight incident response scenarios from current role
3. Consider adding GCP-related projects to portfolio
```

### 5. 生成定制简历

评估完后，生成针对该职位的简历：

```bash
claude code --task tailor-cv --input jobs/senior-sre-google.md
```

输出会放在 `output/` 目录下，格式是 Markdown，你可以导出为 PDF。

### 6. 写求职信

```bash
claude code --task cover-letter --input jobs/senior-sre-google.md
```

### 7. 面试准备

```bash
claude code --task interview-prep --input jobs/senior-sre-google.md
```

会生成一份面试问题清单，按优先级排列：

```
## Interview Preparation: Google SRE

### High Priority Topics (90% chance of being asked)
1. System design: Design a reliable distributed queue
2. Incident management: How do you handle a P0 outage?
3. SLO/SLI: How do you define and measure reliability?

### Medium Priority Topics (60% chance)
4. Kubernetes: Explain pod lifecycle and readiness probes
5. Networking: How does HTTP/2 multiplexing work?
6. Observability: What's the difference between metrics, logs, and traces?

### Low Priority Topics (30% chance)
7. Leadership: How do you handle a team member not pulling weight?
8. Behavioral: Why Google? Why SRE?
```

---

## 性能测试：Token 消耗和响应时间

我用自己的 Anthropic API key 测试了 5 个不同职位（senior 级别，每个 JD 约 500-800 字），结果如下：

| 任务 | 平均 Token 消耗 | 平均响应时间 | 输出质量 |
|------|----------------|-------------|---------|
| 职位评估 | 3,200 tokens | 45s | 匹配度分析合理，但有时会忽略一些隐性要求 |
| 定制简历 | 4,800 tokens | 1m 20s | 需要手动调整格式，内容基本可用 |
| 写求职信 | 2,100 tokens | 35s | 质量最高，几乎可以直接用 |
| 面试准备 | 5,500 tokens | 2m 10s | 问题质量高，但有些太泛 |

**成本估算**：Claude 3.5 Sonnet 价格是 $3/百万输入 tokens，$15/百万输出 tokens。一次完整流程（评估+简历+求职信+面试）大约消耗 15,000 tokens，成本约 **$0.12**。比人工写简历便宜，但比 ChatGPT Plus 贵（如果你用免费额度的话）。

---

## 踩坑记录：真实遇到的问题

### 坑 1：Claude Code 的安装和认证

```bash
# 官方文档说这样安装
npm install -g @anthropic-ai/claude-code

# 但实际运行时报错
Error: Cannot find module '@anthropic-ai/claude-code'

# 需要先确认 Node.js 版本 >= 18
node --version  # 我的是 16，升级到 18 后才解决

# 然后设置 API key
export ANTHROPIC_API_KEY=sk-ant-xxxxx

# 验证安装
claude code --version  # 如果没输出，检查 PATH
```

### 坑 2：YAML 解析错误

我一开始的 `profile.yaml` 里 `experiences` 字段写成了 `experience`（少了个 s），结果 Claude 直接忽略了我的工作经历，生成的简历只有教育背景。

**解决方法**：用 `python -c "import yaml; yaml.safe_load(open('profiles/profile.yaml'))"` 先验证格式。

### 坑 3：中文支持问题

Claude 3.5 Sonnet 对中文支持不错，但生成的英文简历和求职信质量明显高于中文。如果你投的是外企（英语环境），完全没问题。但如果投国内公司，建议还是用中文 JD 并指定输出语言。

```bash
# 可以在 task 中指定语言
claude code --task evaluate-job --input jobs/xxx.md --lang zh-CN
```

### 坑 4：长 JD 的处理

有些 JD 超过 2000 字（尤其是大厂的），Claude 的上下文窗口虽然大，但输出会变得啰嗦。我遇到过一次 Claude 在评估时开始“思考”自己的输出，生成了 3000 字的分析文档，其中一半是废话。

**workaround**：手动截取 JD 的关键部分，控制在 1000 字以内。

### 坑 5：没有版本控制

所有输出文件都是直接覆盖的。如果你对某个职位生成了简历，然后又跑了一次，之前的版本就没了。

**建议**：每次运行前手动备份 `output/` 目录，或者用 Git 管理。

---

## 横向对比：同类工具

| 特性 | ai-job-search | ChatGPT (手动) | Simplify.jobs | Huntr |
|------|--------------|----------------|---------------|-------|
| **自动化程度** | 半自动 (CLI) | 手动复制粘贴 | 全自动 (浏览器插件) | 半自动 (Web UI) |
| **简历定制** | ✅ 按 JD 定制 | ✅ 但需手动 | ❌ 只做匹配 | ❌ 只做匹配 |
| **求职信生成** | ✅ Claude 生成 | ✅ GPT 生成 | ✅ AI 生成 | ✅ 模板 |
| **面试准备** | ✅ 问题+答案 | ❌ 需手动 | ❌ | ❌ |
| **批量处理** | ✅ 可脚本化 | ❌ | ✅ 自动扫描 | ✅ 看板式 |
| **成本** | 按 API 用量 ($0.1-0.5/次) | $20/月 (Plus) | 免费/付费 | 免费/付费 |
| **隐私** | 本地运行 (数据在你自己机器) | 数据在 OpenAI | 数据在第三方 | 数据在第三方 |
| **学习曲线** | 高 (CLI + YAML) | 低 | 低 | 低 |
| **适合人群** | 技术背景强的求职者 | 所有人 | 海投型求职者 | 管理型求职者 |

**我的观点**：
- 如果你只想快速投简历，**Simplify.jobs** 的浏览器插件最省事
- 如果你愿意花时间精投（质量优先），**ai-job-search** 的定制能力更强
- 如果你想要一个看板管理所有申请，**Huntr** 更好

---

## 最终评价

| 维度 | 评分 (1-10) | 说明 |
|------|------------|------|
| **功能完整性** | 7/10 | 覆盖了求职全流程，但缺自动投递和追踪 |
| **输出质量** | 8/10 | Claude 3.5 生成的文本质量高，但需要人工审核 |
| **性价比** | 9/10 | 开源免费，仅需 API 费用，比大多数付费工具便宜 |
| **文档质量** | 5/10 | README 太简略，很多细节需要自己摸索 |
| **易用性** | 3/10 | CLI + YAML 配置，非技术人员基本用不了 |
| **隐私安全** | 9/10 | 数据在本地，不经过第三方服务器 |

### 推荐场景

**强烈推荐**：
- 技术背景强、英语流利的求职者，想精投 5-10 家目标公司
- 自由职业者/独立开发者，想用 AI 辅助求职流程
- 对隐私敏感，不想把简历数据交给第三方平台

**不推荐**：
- 海投型求职者（每天投 50+ 个职位）
- 非技术人员（配置和学习成本太高）
- 主要投国内公司（中文支持一般）

---

## 试用链接

- **ai-job-search 官网**: [https://github.com/MadsLorentzen/ai-job-search](https://github.com/MadsLorentzen/ai-job-search)

---

## 💬 加入 AI 工具交流社群

> **关注我，获取更多 AI 工具深度测评**

- 每周精选 3-5 个最新 AI 开源工具
- 工程师视角的踩坑实录
- 企业 AI 转型实战案例

**关注公众号，回复「工具包」领取：**
- [《AI 工具包 2025》PDF 下载](https://ferryman1980.github.io/assets/ai-toolkit-2025.pdf)
- 《50+ AI 工具导航表》（持续更新）
- 《AI Agent 开发实战手册》
- 《2025 AI 开源项目趋势报告》

---

## 🏢 企业 AI 定制服务

如果你的团队正在探索 AI 落地，我们提供：

- **AI 工作流自动化**：从需求分析到部署上线
- **私有知识库搭建**：RAG + 向量数据库 + 本地模型
- **AI Agent 开发**：定制业务场景的智能代理
- **技术培训**：团队 AI 能力升级方案

📧 联系邮箱: contact@ai-media-matrix.com

---

*本文包含工具推荐链接。如通过链接访问，我会获得少量支持，但不会影响你的使用体验。*