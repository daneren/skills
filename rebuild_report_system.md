# AI 技术汇报体系重建文档
> 最后更新：2026-08-05

---

## 一、体系概览

每日 06:00 起自动触发 8 个方向日报（每5分钟错峰），07:30 汇总推送，每周/月/季/年自动生成长周期汇总。
所有任务均为 `sessionTarget=isolated`，投递到企业微信群 + 邮件（3个收件人）。

---

## 二、每日日报任务（Asia/Shanghai，错峰触发）

### 重试机制（所有日报任务统一）
- 抓取失败（网络错误/超时/内容为空）→ 按 30s / 60s / 120s 递增等待后重试，最多3次
- 3次均失败 → 跳过该来源，汇报中标注「⚠️ 数据获取失败，已跳过」，继续完成其余步骤
- timeoutSeconds: 300s

---

### 1. Megatron 每日技术进度汇报
- **Job ID**: 8ed3fb1d-c108-4b23-b582-eed219a2d475
- **触发时间**: 06:00
- **输出文件**: `daily_reports/megatron_YYYY-MM-DD.md`
- **数据来源**:
  - github.com/NVIDIA/Megatron-LM (commits/releases/PRs)
  - ArXiv: megatron
- **汇报章节**: Release动态 / 代码变更 / PR进展 / 论文 / 新发现 / 总结

### 2. AI Infra 推理引擎每日技术进度汇报
- **Job ID**: 6d11bfd9-2c1a-4b1a-ae23-1b7569ded8d6
- **触发时间**: 06:05
- **输出文件**: `daily_reports/ai_infra_inference_YYYY-MM-DD.md`
- **数据来源**:
  - github.com/vllm-project/vllm
  - github.com/sgl-project/sglang
  - github.com/NVIDIA/TensorRT-LLM
  - ArXiv: LLM inference system
- **汇报章节**: 推理引擎动态 / Release / PR / 论文 / 新发现 / 总结

### 3. AI Infra 训练基础设施每日技术进度汇报
- **Job ID**: 54744f4f-c60a-40bd-9fdc-f8533b7b28a0
- **触发时间**: 06:10
- **输出文件**: `daily_reports/ai_infra_training_YYYY-MM-DD.md`
- **数据来源**:
  - github.com/Dao-AILab/flash-attention
  - github.com/triton-lang/triton
  - github.com/NVIDIA/nccl
  - github.com/microsoft/DeepSpeed
  - ArXiv: LLM training infrastructure / AI compiler GPU kernel
- **汇报章节**: 训练基础设施动态 / Release / PR / 论文 / 新发现 / 总结

### 4. GitHub 热门项目每日汇报
- **Job ID**: e3eb9a21-5b8b-4bab-b7e5-c8fcd14a0918
- **触发时间**: 06:15
- **输出文件**: `daily_reports/github_trending_YYYY-MM-DD.md`
- **数据来源**:
  - github.com/trending (今日/本周/本月)
  - EdgeBrowser 搜索: github trending AI tools
- **汇报章节**: 今日TOP10 / 本周持续热门 / AI相关热门 / 开发者工具 / 新发现 / 总结

### 5. 大模型国内每日技术汇报（DeepSeek/Qwen/Kimi/MiniMax）
- **Job ID**: 83a60066-... （llm_cn）
- **触发时间**: 06:20
- **输出文件**: `daily_reports/llm_cn_YYYY-MM-DD.md`
- **数据来源**:
  - github.com/deepseek-ai / QwenLM / Kimi / MiniMax 相关仓库
  - ArXiv: DeepSeek Qwen 国内大模型
- **汇报章节**: 各模型动态 / 重点论文 / 新发现 / 总结

### 6. 大模型国际每日技术汇报（OpenAI/Claude/Gemini/Llama/Mistral）
- **Job ID**: 72e8a3ff-... （llm_intl）
- **触发时间**: 06:25
- **输出文件**: `daily_reports/llm_intl_YYYY-MM-DD.md`
- **数据来源**:
  - openai.com/news / anthropic.com/news / Google DeepMind blog
  - github.com/meta-llama / mistralai 相关仓库
  - ArXiv: LLM reasoning scaling
- **汇报章节**: 各模型动态 / 重点论文 / 新发现 / 总结

### 7. verl 每日技术进度汇报
- **Job ID**: 62a1d2b7-88d4-40c4-a5c5-6c8c6b85e8d9
- **触发时间**: 06:30
- **输出文件**: `daily_reports/verl_YYYY-MM-DD.md`
- **数据来源**:
  - github.com/volcengine/verl (commits/releases/PRs/issues)
  - ArXiv: verl reinforcement learning
- **汇报章节**: Release / 代码变更 / PR / Issue / 论文 / 新发现 / 总结
- **fallback 模型**: gongfeng/deepseek-v3-2

### 8. Agentic RL + Agent-Lightning 每日技术进度汇报
- **Job ID**: 3260f75a-2551-4e77-aff7-bf7c450da5da
- **触发时间**: 06:35
- **输出文件**: `daily_reports/agentic_rl_YYYY-MM-DD.md`
- **数据来源**:
  - ArXiv: agentic RL / RLVR / LLM agent RL training
  - 核心仓库: OpenRLHF / TRL / verl / LLaMA-Factory / AREAL / ROLL / SLIME / DeepSpeedExamples
  - Agent-Lightning: github.com/microsoft/agent-lightning
  - HuggingFace blog / Microsoft Research blog
- **汇报章节**: 论文 / 框架代码 / Agent-Lightning动态 / PR / 业界动态 / 新发现 / 总结
- **fallback 模型**: gongfeng/deepseek-v3-2

---

## 三、每日汇总任务（07:30 Asia/Shanghai）

### 每日技术汇报邮件汇总
- **Job ID**: ca22acec-c8f8-41e8-9232-9ad3f556729a
- **触发时间**: 07:30
- **输出文件**: `daily_reports/daily_summary_YYYY-MM-DD.md`
- **功能**: 自动收集当日所有 `*_YYYY-MM-DD.md` 文件，拼接为全方向汇总，发送企业微信群 + 邮件（含所有子报告附件）
- **timeoutSeconds**: 300s

---

## 四、长周期汇总任务

| 周期 | 任务名 | Job ID | 触发时间 | timeout |
|------|--------|--------|----------|---------|
| 周报 | 每周技术进展汇总 | 353c9169 | 每周五 06:00 | 900s |
| 月报 | 每月技术进展汇总 | d4007f78 | 每月1日 06:00 | 900s |
| 季报 | 季度技术进展汇总 | 0fd020ec | 每季1日 06:10 | 1200s |
| 半年报 | 半年技术进展汇总 | 1c59672f | 1月1日、7月1日 06:20 | 1800s |
| 年报 | 年度技术进展汇总 | c36fd94a | 每年1月1日 06:30 | 2400s |
| 两年报 | 两年度技术进展汇总 | d24bbd6d | 每年1月1日 06:40 | 3000s |
| 三年报 | 三年度技术进展汇总 | 750dcaa9 | 每年1月1日 06:50 | 3600s |

> 注：长周期汇总自动聚合所有 topic 日报，不再单独设置大模型长周期任务。

### 周报逻辑（2026-04-18 更新）
- **有日报文件**：直接读取文件内容，提炼进周报，不重新抓取数据源
- **无日报文件**：实时抓取该方向数据源，直接提炼进周报，**不单独生成日报文件**
- 每方向标注来源：📄 来自日报 / 🌐 实时抓取

---

## 五、动态跟踪列表

- 文件路径: `/root/.openclaw/workspace/tracking_list.json`
- **周报触发**: 每周评估，新发现 ≥2次 → watchlist提升为active；连续4周无进展 → 降级watchlist
- **月报触发**: 每月深度评估，watchlist连续2月无动态 → 移入removed；新发现 ≥3次 → 加入watchlist

---

## 六、邮件配置

- **脚本路径**: `/root/.openclaw/workspace/send_report_email.py`
- **发件人**: 920325364@qq.com（QQ邮箱 SMTP smtp.qq.com:465 SSL）
- **收件人**:
  - 920325364@qq.com
  - danerli@tencent.com
  - lirunchh@gmail.com

---

## 七、文件目录结构

```
/root/.openclaw/workspace/
├── daily_reports/
│   ├── megatron_YYYY-MM-DD.md
│   ├── ai_infra_inference_YYYY-MM-DD.md
│   ├── ai_infra_training_YYYY-MM-DD.md
│   ├── github_trending_YYYY-MM-DD.md
│   ├── llm_cn_YYYY-MM-DD.md
│   ├── llm_intl_YYYY-MM-DD.md
│   ├── verl_YYYY-MM-DD.md
│   ├── agentic_rl_YYYY-MM-DD.md
│   ├── daily_summary_YYYY-MM-DD.md
│   ├── weekly_YYYY-WXX.md
│   ├── monthly_YYYY-MM.md
│   ├── quarterly_YYYY-QX.md
│   ├── semiannual_YYYY-HX.md
│   ├── annual_YYYY.md
│   ├── biannual_YYYY-YYYY.md
│   └── triannual_YYYY-YYYY.md
├── tracking_list.json
├── send_report_email.py
├── weekly_report_prompt.md   ← 周报 prompt 备份
├── rebuild_report_system.md  ← 本文件
└── skills/
    └── tech-report-builder/
        └── SKILL.md
```

---

## 八、已禁用/废弃任务

| Job ID | 原任务名 | 废弃原因 |
|--------|----------|----------|
| 005cd45e | AI Infra 训练推理（合并版） | token超限，已拆分为推理+训练两个独立任务 |
| 27d36768 | 大模型算法进展（合并版） | token超限，已拆分为国内+国际两个独立任务 |
| 8e3b40f7 | Agent-Lightning 单独任务 | 已并入 Agentic RL 任务 |

---

## 九、变更记录

| 日期 | 变更内容 |
|------|----------|
| 2026-03-05 | 初始搭建：Megatron / verl / Agentic RL / AI Infra / Agent-Lightning 5个日报 |
| 2026-04-06 | 新增大模型日报、GitHub热门日报；新增周/月/季/半年/年/两年/三年汇总任务（共18个） |
| 2026-04-06 | 创建 tech-report-builder Skill；发送重建文档至邮件组 |
| 2026-04-07 | 删除大模型半年/年度/三年度独立任务（已被通用长周期汇总覆盖，任务数14个） |
| 2026-04-07 | 全部日报任务新增重试机制（30s/60s/120s递增，最多3次） |
| 2026-04-09 | AI Infra合并任务超时（194k tokens），拆分为推理引擎（6d11bfd9）+ 训练基础设施（54744f4f） |
| 2026-04-09 | 大模型合并任务超时（216k tokens），拆分为国内（83a60066）+ 国际（72e8a3ff） |
| 2026-04-13 | verl/Agentic RL 08:00并发过载，调整为08:20/08:25并新增 deepseek-v3-2 fallback |
| 2026-04-13 | 全部日报任务时间调整为从06:00起每5分钟错峰；汇总邮件从09:30改为07:30；长周期任务统一改为06:00 |
| 2026-04-18 | 周报逻辑优化：有日报时直接读取提炼，无日报时实时抓取但只写入周报不生成独立日报文件 |
