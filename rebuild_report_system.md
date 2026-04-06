# AI 技术汇报体系一键重建文档（v5）

> 最后更新：2026-04-06
> 版本：v7（动态跟踪列表机制 · 时机由周报/月报驱动 · 每次修改自动同步本文档）

---

## 🏗️ 架构原则

```
每个任务（topic）
  └─ 独立生成 <topic>_YYYY-MM-DD.md，写入 daily_reports/
  └─ 单独发一封邮件（正文+附件=自身.md）

每日邮件汇总（09:30）
  └─ glob *_TODAY.md 自动发现所有 topic
  └─ 生成 daily_summary_TODAY.md
  └─ 邮件正文 = 汇总内容，附件 = 所有子报告 + 汇总

各周期汇总任务（周/月/季/半年/年/2年/3年）
  └─ glob *_日期范围.md 自动发现所有 topic（按 topic 分类）
  └─ 生成对应汇总 .md
  └─ 邮件正文 = 汇总内容，附件 = 周期内所有子报告 + 汇总（去重）

大模型专题汇总（半年/年/3年）
  └─ 只读取 llm_progress_*.md 文件
  └─ 按模型/机构分类整理
  └─ 同上投递规范

新增任务时
  └─ 文件命名 <topic>_YYYY-MM-DD.md 写入 daily_reports/
  └─ 所有通用汇总任务自动感知，无需修改
```

---

## ⏰ 时间规范（固定，不可修改）

| 类型 | 触发时间 |
|------|---------|
| 所有日报任务 | **08:00** Asia/Shanghai |
| 每日邮件汇总 | **09:30** Asia/Shanghai |
| 周报 | 每周五 **08:00** |
| 月报 | 月末最后工作日 **08:00** |
| 季度报 | 季末最后工作日 **08:00** |
| 半年报 | 半年末最后工作日 **08:00** |
| 年度报 | 12月最后工作日 **08:00** |
| 两年度报 | 偶数年12月最后工作日 **08:00** |
| 三年度报 | 3的倍数年12月最后工作日 **08:00** |

---

## 📋 重建前准备

### 1. 替换变量

| 占位符 | 当前值 |
|--------|--------|
| `{GROUP_ID}` | `aibY2r-rXTKG9AJ1bKpoPbL69AcynSnoUvy` |
| `{EMAIL_1}` | `920325364@qq.com` |
| `{EMAIL_2}` | `danerli@tencent.com` |
| `{EMAIL_3}` | `lirunchh@gmail.com` |

### 2. 部署邮件脚本

保存为 `/root/.openclaw/workspace/send_report_email.py`：

```python
#!/usr/bin/env python3
"""用法: python3 send_report_email.py "主题" "正文" [附件路径1] [附件路径2] ..."""
import smtplib, ssl, sys, os, mimetypes
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.header import Header
from email import encoders

SMTP_HOST = 'smtp.qq.com'
SMTP_PORT = 465
FROM_ADDR = '920325364@qq.com'
FROM_PASS = ''   # ← QQ邮箱授权码
TO_ADDRS  = ['920325364@qq.com', 'danerli@tencent.com', 'lirunchh@gmail.com']

def send(subject, body, attachment_paths=None):
    msg = MIMEMultipart('mixed')
    msg['Subject'] = Header(subject, 'utf-8')
    msg['From'] = FROM_ADDR
    msg['To'] = ', '.join(TO_ADDRS)
    msg.attach(MIMEText(body, 'plain', 'utf-8'))
    for fpath in (attachment_paths or []):
        fpath = fpath.strip()
        if not os.path.isfile(fpath): continue
        filename = os.path.basename(fpath)
        with open(fpath, 'rb') as f: data = f.read()
        mime_type, _ = mimetypes.guess_type(fpath)
        if mime_type is None: mime_type = 'application/octet-stream'
        main_type, sub_type = mime_type.split('/', 1)
        if main_type == 'text':
            part = MIMEText(data.decode('utf-8', errors='replace'), sub_type, 'utf-8')
        else:
            part = MIMEBase(main_type, sub_type)
            part.set_payload(data)
            encoders.encode_base64(part)
        encoded_name = Header(filename, 'utf-8').encode()
        part['Content-Disposition'] = f'attachment; filename="{encoded_name}"'
        part['Content-Type'] += f'; name="{encoded_name}"'
        msg.attach(part)
    ctx = ssl.create_default_context()
    with smtplib.SMTP_SSL(SMTP_HOST, SMTP_PORT, context=ctx) as s:
        s.login(FROM_ADDR, FROM_PASS)
        s.sendmail(FROM_ADDR, TO_ADDRS, msg.as_string())
    print(f'[OK] 邮件已发送: {subject}')

if __name__ == '__main__':
    if len(sys.argv) < 3: sys.exit('用法: send_report_email.py "主题" "正文" [附件...]')
    send(sys.argv[1], sys.argv[2], sys.argv[3:])
```

### 3. 创建目录

```bash
mkdir -p /root/.openclaw/workspace/daily_reports
```

### 4. 文件命名规范

| 文件 | 命名格式 |
|------|---------|
| 各 topic 日报 | `<topic>_YYYY-MM-DD.md` |
| 每日汇总 | `daily_summary_YYYY-MM-DD.md` |
| 周报 | `weekly_YYYY-WNN.md` |
| 月报 | `monthly_YYYY-MM.md` |
| 季度报 | `quarterly_YYYY-Qn.md` |
| 半年报（通用） | `semiannual_YYYY-Hnn.md` |
| 年度报（通用） | `annual_YYYY.md` |
| 两年度报 | `biannual_YYYY.md` |
| 三年度报（通用） | `triannual_YYYY.md` |
| 大模型日报 | `llm_progress_YYYY-MM-DD.md` |
| 大模型半年报 | `llm_semiannual_YYYY-Hnn.md` |
| 大模型年报 | `llm_annual_YYYY.md` |
| 大模型三年报 | `llm_triannual_YYYY.md` |

---

## 📋 任务创建指令（共 17 个）

向新 OpenClaw 逐条发送即可重建全部任务。

---

### ① Megatron 每日技术进度汇报
- **Cron**: `0 8 * * *` | Asia/Shanghai | isolated

```
请创建 cron 任务（0 8 * * * Asia/Shanghai isolated delivery to={GROUP_ID}）：
请抓取 Megatron 相关最新进展并生成今日汇报：
1. https://github.com/NVIDIA/Megatron-LM/commits/main（近24h）
2. https://github.com/NVIDIA/Megatron-LM/releases
3. https://github.com/NVIDIA/Megatron-LM/pulls?state=open&sort=updated
4. ArXiv: https://arxiv.org/search/?searchtype=all&query=megatron
5. 【动态发现机制】发现新兴相关项目列出「🆕 新发现」
6. 生成中文汇报：📦Release/🔧代码变更/🔀PR/📄论文/🆕新发现/📌总结

投递：
① 写入 daily_reports/megatron_$(date +%Y-%m-%d).md
② --channel openclaw-wecom-bot --to {GROUP_ID}
③ python3 .../send_report_email.py "【Megatron 日报】$(date +%Y-%m-%d)" \
     "$(cat .../megatron_$(date +%Y-%m-%d).md)" \
     .../megatron_$(date +%Y-%m-%d).md
```

---

### ② verl 每日技术进度汇报
- **Cron**: `0 8 * * *` | Asia/Shanghai | isolated

```
请创建 cron 任务（0 8 * * * Asia/Shanghai isolated delivery to={GROUP_ID}）：
请抓取 verl 项目相关最新进展并生成今日汇报：
1. https://github.com/volcengine/verl/commits/main（近24h）
2. https://github.com/volcengine/verl/releases
3. https://github.com/volcengine/verl/pulls?state=open&sort=updated
4. https://github.com/volcengine/verl/issues?state=open&sort=updated
5. ArXiv: verl+reinforcement+learning
6. 【动态发现机制】发现新兴相关项目列出「🆕 新发现」
7. 生成中文汇报：📦Release/🔧代码变更/🔀PR/🐛Issue/📄论文/🆕新发现/📌总结

投递（同 Megatron，topic=verl）
```

---

### ③ Agentic RL + Agent-Lightning 每日技术进度汇报
- **Cron**: `0 8 * * *` | Asia/Shanghai | isolated
- **合并说明**：Agent-Lightning 已并入本任务（原⑤已禁用）

```
请创建 cron 任务（0 8 * * * Asia/Shanghai isolated delivery to={GROUP_ID}）：

【Agentic RL 部分】
核心仓库：OpenRLHF/OpenRLHF, huggingface/trl, volcengine/verl,
          AREAL-GT/AREAL, ROLL-LM/roll, SLIME-LM/slime,
          hiyouga/LLaMA-Factory, microsoft/DeepSpeedExamples
ArXiv：agentic RL / RLVR / LLM agent RL training
博客：https://huggingface.co/blog

【Agent-Lightning 部分】
仓库：microsoft/agent-lightning（commits/PRs/issues/releases）+ microsoft/DeepSpeed
ArXiv：agent lightning RL
博客：https://www.microsoft.com/en-us/research/blog/

【动态发现机制】发现新兴框架/仓库列出「🆕 新发现」
生成中文汇报：
  一、最新论文亮点（Agentic RL）
  二、主要框架代码进展（ROLL/AREAL/SLIME/OpenRLHF/verl/TRL）
  三、Agent-Lightning 动态（Release/代码变更/PR/Issue/研究博客）
  四、值得关注的 PR/新特性 / 五、业界动态 / 六、新发现 / 七、总结

投递（同 Megatron，topic=agentic_rl，文件名 agentic_rl_YYYY-MM-DD.md）
```

---

### ④ AI Infra 训练推理每日技术进度汇报
- **Cron**: `0 8 * * *` | Asia/Shanghai | isolated

```
请创建 cron 任务（0 8 * * * Asia/Shanghai isolated delivery to={GROUP_ID}）：
推理引擎：vllm/sglang/TensorRT-LLM/FastChat（近24h commits）
训练基础设施：flash-attention/triton/nccl/DeepSpeed/Megatron-LM
ArXiv：LLM inference system / training infrastructure / AI compiler GPU kernel
【动态发现机制】发现新兴项目列出「🆕 新发现」
生成中文汇报：🚀推理引擎/🔧训练基础设施/📦Release/🔀PR/📄论文/🆕新发现/📌总结

投递（同 Megatron，topic=ai_infra）
```

---

### ~~⑤ Agent-Lightning 每日技术进度汇报~~
> ⚠️ **已禁用**：Agent-Lightning 内容已合并入③，此任务不再单独运行。

---

### ⑥ 大模型算法进展每日技术汇报
- **Cron**: `0 8 * * *` | Asia/Shanghai | isolated

```
请创建 cron 任务（0 8 * * * Asia/Shanghai isolated delivery to={GROUP_ID}）：
收集各主流大模型最新动态（近24-48h）：
🔴 OpenAI: openai.com/news + platform changelog + github openai-python
🟠 DeepSeek: github deepseek-ai（V3/R1/Prover-V2）+ huggingface
🟡 Qwen: github QwenLM（Qwen3/Qwen-Agent）+ huggingface + qwenlm.github.io/blog
🟢 Kimi(Moonshot): platform.moonshot.cn/docs/changelog + github MoonshotAI
🔵 MiniMax: github MiniMaxAI + huggingface MiniMaxAI
🟣 Claude(Anthropic): anthropic.com/news + docs release-notes + github anthropic-sdk
🔵 Gemini(Google): blog.google/technology/google-deepmind + ai.google.dev changelog
⚫ Llama(Meta): github meta-llama（llama-models/llama-stack）+ huggingface meta-llama
🟤 Mistral: mistral.ai/news + huggingface mistralai
ArXiv：LLM alignment / LLM reasoning scaling / foundation model
【动态发现机制】新兴模型列出「🆕 新发现模型/技术」

生成中文汇报：🤖大模型算法进展日报 [YYYY-MM-DD]
  一、各模型今日动态（按模型，数量不固定）
  二、今日重点论文（若有）
  三、新发现模型/技术（若有）
  四、今日技术亮点总结

投递：
① 写入 daily_reports/llm_progress_$(date +%Y-%m-%d).md
② --channel openclaw-wecom-bot --to {GROUP_ID}
③ python3 .../send_report_email.py "【大模型日报】$(date +%Y-%m-%d)" \
     "$(cat .../llm_progress_$(date +%Y-%m-%d).md)" \
     .../llm_progress_$(date +%Y-%m-%d).md
```

---

### ⑦ 每日技术汇报邮件汇总（09:30）
- **Cron**: `30 9 * * *` | Asia/Shanghai | isolated

```
请创建 cron 任务（30 9 * * * Asia/Shanghai isolated delivery to={GROUP_ID}）：
1. glob daily_reports/*_$(date +%Y-%m-%d).md（自动发现所有 topic，跳过 daily_summary_*）
2. 按 topic 拼接汇总，各 topic 用 ===== {topic} ===== 分隔，缺失标注「未生成」
3. 写入 daily_reports/daily_summary_$(date +%Y-%m-%d).md
4. --channel openclaw-wecom-bot --to {GROUP_ID} 发送摘要+共 N 个方向
5. 发邮件：
   SUMMARY_FILE=daily_reports/daily_summary_$(date +%Y-%m-%d).md
   ATTACHMENTS=（所有 *_TODAY.md 排除 SUMMARY_FILE）
   send_report_email.py "【每日技术汇报】DATE" "$(cat SUMMARY_FILE)" $ATTACHMENTS $SUMMARY_FILE
```

---

### ⑧ 每周技术进展汇总（周五 08:00）
- **Cron**: `0 8 * * 5` | Asia/Shanghai | isolated

```
请创建 cron 任务（0 8 * * 5 Asia/Shanghai isolated delivery to={GROUP_ID}）：
1. glob 过去7天 daily_reports/*_DATE.md（跳过 weekly_*），按 topic 分类整理
2. 缺失方向从网络补充过去7天数据
3. 生成周汇总：各方向进展/重点论文TOP5/新发现/趋势分析/下周预测
4. 写入 daily_reports/weekly_$(date +%Y-W%V).md
5. --channel openclaw-wecom-bot --to {GROUP_ID}
6. 发邮件正文=周汇总，附件=本周所有子报告+周汇总（去重）
```

---

### ⑨ 每月技术进展汇总（月末最后工作日 08:00）
- **Cron**: `0 8 L * 1-5` | Asia/Shanghai | isolated

```
请创建 cron 任务（0 8 L * 1-5 Asia/Shanghai isolated delivery to={GROUP_ID}）：
1. glob 本月 daily_reports/*_$(date +%Y-%m)-*.md（跳过 monthly_*），按 topic 分类
2. 缺失方向从网络补充本月数据
3. 生成月汇总：各方向进展/重点论文TOP10/新发现/趋势分析/下月展望
4. 写入 daily_reports/monthly_$(date +%Y-%m).md
5. --channel openclaw-wecom-bot --to {GROUP_ID}
6. 发邮件正文=月汇总，附件=本月所有子报告+月汇总（去重）
```

---

### ⑩ 季度技术进展汇总（每季度末最后工作日 08:00）
- **Cron**: `0 8 L 3,6,9,12 1-5` | Asia/Shanghai | isolated

```
请创建 cron 任务（0 8 L 3,6,9,12 1-5 Asia/Shanghai isolated delivery to={GROUP_ID}）：
1. glob 过去90天 daily_reports/*_DATE.md（跳过 quarterly_*），按 topic 分类
2. 缺失方向从网络补充季度数据
3. 生成季度汇总：各方向进展/重点论文TOP15/新兴项目盘点/趋势分析/下季度展望
4. 写入 daily_reports/quarterly_$(date +%Y-Q%q).md
5. --channel openclaw-wecom-bot --to {GROUP_ID}
6. 发邮件正文=季度汇总，附件=季度所有子报告+季度汇总（去重）
```

---

### ⑪ 半年技术进展汇总（每半年末最后工作日 08:00）
- **Cron**: `0 8 L 6,12 1-5` | Asia/Shanghai | isolated

```
同⑩，时间范围改为180天，文件名 semiannual_$(date +%Y-H%m).md，论文TOP20，含下半年展望
```

---

### ⑫ 年度技术进展汇总（12月最后工作日 08:00）
- **Cron**: `0 8 L 12 1-5` | Asia/Shanghai | isolated

```
同⑩，时间范围改为365天，文件名 annual_$(date +%Y).md，论文TOP25，含年表+明年展望
```

---

### ⑬ 两年度技术进展汇总（偶数年12月最后工作日 08:00）
- **Cron**: `0 8 L 12 1-5/2` | Asia/Shanghai | isolated

```
同⑩，时间范围改为730天，文件名 biannual_$(date +%Y).md，论文TOP30，含两年年表+未来两年展望
```

---

### ⑭ 三年度技术进展汇总（3的倍数年12月最后工作日 08:00）
- **Cron**: `0 8 L 12 1-5/3` | Asia/Shanghai | isolated

```
同⑩，时间范围改为1095天，文件名 triannual_$(date +%Y).md，论文TOP40，含完整年表+未来三年展望
```

---

### ⑮ 大模型算法进展半年度汇总（每半年末最后工作日 08:00）
- **Cron**: `0 8 L 6,12 1-5` | Asia/Shanghai | isolated

```
请创建 cron 任务（0 8 L 6,12 1-5 Asia/Shanghai isolated delivery to={GROUP_ID}）：
1. 读取过去180天所有 daily_reports/llm_progress_*.md 文件
2. 按模型/机构分类整理，缺失从网络补充
3. 生成大模型半年度报：
   各大模型半年重大进展 / 标志性模型发布 / 重点论文TOP20
   新兴模型盘点 / 技术趋势深度分析（规模/推理/多模态/对齐/开源 vs 闭源/商业化）/ 下半年展望
4. 写入 daily_reports/llm_semiannual_$(date +%Y-H%m).md
5. --channel openclaw-wecom-bot --to {GROUP_ID}
6. 发邮件正文=汇总，附件=所有 llm_progress_*.md + 汇总.md（去重）
```

---

### ⑯ 大模型算法进展年度汇总（12月最后工作日 08:00）
- **Cron**: `0 8 L 12 1-5` | Asia/Shanghai | isolated

```
同⑮，时间范围改为365天，文件名 llm_annual_$(date +%Y).md，论文TOP25，含年表+明年展望
```

---

### ⑰ 大模型算法进展三年度汇总（3的倍数年12月最后工作日 08:00）
- **Cron**: `0 8 L 12 1-5/3` | Asia/Shanghai | isolated

```
同⑮，时间范围改为1095天，文件名 llm_triannual_$(date +%Y).md，论文TOP40，含完整年表+未来三年展望+技术周期规律分析
```

---

## 📊 全部任务一览（18 个）

| # | 任务名称 | Cron | 输出文件 | 邮件附件 |
|---|---------|------|---------|---------|
| ① | Megatron 日报 | `0 8 * * *` | `megatron_YYYY-MM-DD.md` | 自身 |
| ② | verl 日报 | `0 8 * * *` | `verl_YYYY-MM-DD.md` | 自身 |
| ③ | Agentic RL + Agent-Lightning 日报 | `0 8 * * *` | `agentic_rl_YYYY-MM-DD.md` | 自身 |
| ④ | AI Infra 日报 | `0 8 * * *` | `ai_infra_YYYY-MM-DD.md` | 自身 |
| ~~⑤~~ | ~~Agent-Lightning 日报~~ | ~~已禁用~~ | ~~并入③~~ | — |
| ⑥ | 大模型算法进展日报 | `0 8 * * *` | `llm_progress_YYYY-MM-DD.md` | 自身 |
| ⑦ | 每日邮件汇总 | `30 9 * * *` | `daily_summary_YYYY-MM-DD.md` | 今日所有子报告+汇总 |
| ⑧ | 周报 | `0 8 * * 5` | `weekly_YYYY-WNN.md` | 本周子报告+汇总 |
| ⑨ | 月报 | `0 8 L * 1-5` | `monthly_YYYY-MM.md` | 本月子报告+汇总 |
| ⑩ | 季度报 | `0 8 L 3,6,9,12 1-5` | `quarterly_YYYY-Qn.md` | 季度子报告+汇总 |
| ⑪ | 半年报 | `0 8 L 6,12 1-5` | `semiannual_YYYY-Hnn.md` | 半年子报告+汇总 |
| ⑫ | 年度报 | `0 8 L 12 1-5` | `annual_YYYY.md` | 全年子报告+汇总 |
| ⑬ | 两年度报 | `0 8 L 12 1-5/2` | `biannual_YYYY.md` | 两年子报告+汇总 |
| ⑭ | 三年度报 | `0 8 L 12 1-5/3` | `triannual_YYYY.md` | 三年子报告+汇总 |
| ⑮ | 大模型半年报 | `0 8 L 6,12 1-5` | `llm_semiannual_YYYY-Hnn.md` | llm_progress_*.md+汇总 |
| ⑯ | 大模型年报 | `0 8 L 12 1-5` | `llm_annual_YYYY.md` | llm_progress_*.md+汇总 |
| ⑰ | 大模型三年报 | `0 8 L 12 1-5/3` | `llm_triannual_YYYY.md` | llm_progress_*.md+汇总 |
| ⑱ | GitHub 热门项目日报 | `0 8 * * *` | `github_trending_YYYY-MM-DD.md` | 自身 |

**活跃任务：17 个**（⑤已禁用）

---

## ➕ 新增任务说明

1. 创建 cron 任务，在 `daily_reports/<新topic>_$(date +%Y-%m-%d).md` 写入报告
2. 投递步骤与现有日报任务完全一致
3. **无需修改任何汇总任务**——通用汇总（⑦~⑭）通过 `glob *_日期.md` 自动感知

---

## 🔬 当前覆盖研究方向

| topic 文件前缀 | 研究方向 | 核心来源 |
|--------------|---------|---------|
| `megatron_` | Megatron-LM | NVIDIA/Megatron-LM |
| `verl_` | verl RL 框架 | volcengine/verl |
| `agentic_rl_` | Agentic RL 训练 | ROLL/AREAL/SLIME/OpenRLHF/TRL |
| `ai_infra_` | AI Infra 训练推理 | vLLM/SGLang/TRT-LLM/FlashAttention/Triton |
| `agent_lightning_` | Agent-Lightning | microsoft/agent-lightning |
| `llm_progress_` | 大模型算法进展 | OpenAI/DeepSeek/Qwen/Kimi/MiniMax/Claude/Gemini/Llama/Mistral |

---

## 📧 收件邮箱

- `920325364@qq.com`
- `danerli@tencent.com`
- `lirunchh@gmail.com`

---

## 📁 关键路径

| 路径 | 说明 |
|------|------|
| `/root/.openclaw/workspace/daily_reports/` | 所有报告文件 |
| `/root/.openclaw/workspace/send_report_email.py` | 邮件脚本 |
| `/root/.openclaw/workspace/rebuild_report_system.md` | 本文档 |
