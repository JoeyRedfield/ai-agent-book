# 任务：为《深入理解 AI Agent》第 1～10 章搭建可复现的实验环境

## 目标
在仓库根目录建立覆盖 ch1～ch10 的共享 Python 环境，并为文档要求独立 venv 的项目
单独建环境；全部验证到“能跑通入口命令”为止。产出环境清单与复现提示词。

## 前置检查（先做，不要直接开始安装）
1. 确认 `uv --version`、`python3.11`、`python3.12` 可用；缺失时用
   `uv python install 3.11` / `3.12` 补齐。
2. 检查网络可达性：
   `curl -sS -o /dev/null -w '%{http_code}\n' --max-time 15 https://pypi.org/simple/pytz/`
   若超时，检查本机代理 `lsof -nP -iTCP -sTCP:LISTEN | rg 7890`（ClashX 默认端口），
   然后 `export HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 ALL_PROXY=http://127.0.0.1:7890` 再执行后续命令。
3. 检查磁盘剩余空间（建议至少 15GB）。
4. 先读 `pyproject.toml` 的 `[project.optional-dependencies]` 与根 README 安装段，
   确认各章 extra 的语义。注意：`--locked` 锁文件固定指向 PyPI，**不要**用镜像源
   （`--default-index`/`--index-url`）跑 `--locked`，会直接校验失败。

## 步骤
1. 共享环境（一次安装全部章节，等价于逐章执行）：
   ```bash
   uv sync --locked \
     --extra ch1 --extra ch2 --extra ch3 --extra ch4 --extra ch5 \
     --extra ch6 --extra ch7 --extra ch8 --extra ch9 --extra ch10 \
     --extra dev
   ```
   重复执行应显示 `Audited`。
2. Playwright 浏览器（ch4/ch8/ch9/ch10 需要）：
   ```bash
   .venv/bin/python -m playwright install chromium
   ```
3. 独立 venv 的项目：
   - `chapter9/computer-use-open-model`：`python3.11 -m venv .venv`，
     `.venv/bin/pip install -r requirements.txt`；
     该固定 browser-use 提交漏声明 `playwright` / `pydantic-settings`，需补装：
     `.venv/bin/pip install playwright pydantic-settings`；
     再 `.venv/bin/python -m playwright install chromium`，`cp env.example .env`。
   - `chapter10/generative-agents`：`python3.11 -m venv .venv`，
     `.venv/bin/pip install -r requirements.txt`；上游克隆到
     `/tmp/generative_agents` 并 `git checkout --detach fe05a71d3e4ed7d10bf68aa4eda6dd995ec070f4`。
   - `chapter10/use-computer-while-calling`：`git clone https://github.com/19PINE-AI/TalkAct.git`
     后 `git checkout --detach 7d70007f72d45ddfc1a14e8e229b6d444e4919a2`，
     `python3.12 -m venv .venv`，`.venv/bin/pip install -r requirements.txt`，
     `.venv/bin/python -m playwright install chromium`。

## 验证标准（不满足不算完成）
1. 共享环境逐章 import 检查通过：ch1（pdfplumber/PyPDF2/reportlab）、
   ch2（torch/transformers/fastapi/anthropic/google.genai/litellm/ollama/tiktoken/playwright）、
   ch3（chromadb/faiss/sentence_transformers/mem0/memobase）、
   ch4（cv2/fitz/browser_use/mcp/arxiv/yfinance/PyGithub/langchain_openai）、
   ch5（fitz/constraint/fastapi/mcp）、ch6（sklearn/scipy/plotly/numba/fish_audio_sdk）、
   ch7（torch/transformers/datasets/accelerate/peft/trl/bitsandbytes/wandb/librosa/snac）、
   ch8（chromadb/langchain_core/slack_sdk/browser_use）、
   ch9（aiortc/whisper/torch/playwright）、ch10（tiktoken/playwright/httpx）。
   注意模块名：`mem0ai` 包导入名为 `mem0`，`python-constraint` 为 `constraint`，
   `openai-whisper` 为 `whisper`。
2. 每个项目跑一个不依赖 API Key 的入口命令：
   `python main.py --help` 或 `--dry-run`；有测试的项目 `pytest -q`。
3. 记录 Python 版本、包总数、克隆 commit，写入 `docs/ENVIRONMENTS.md`。

## 交付物
- `docs/ENVIRONMENTS.md`：共享环境命令、网络排障、独立 venv、未覆盖项清单。
- 本提示词（存为 `docs/ENVIRONMENT_SETUP_PROMPT.md`）。
- 不提交 `.venv`、`.env`、外部 checkout（它们可能显示为未跟踪，属预期）。

## 已知坑与边界
- PyPI 直连超时 → 走本地代理；禁止镜像源 + `--locked` 组合。
- `chapter7/sesame` 依赖 Unsloth/Triton，面向 Linux+GPU，macOS 不搭建。
- ch9 外部复现轨（claude-quickstarts/browser-use/XLeRobot/RoboCrew/lerobot-sim2real）
  需要 Docker/真机/GPU，按各 README 手动克隆，不自动处理。
- 真实模型实验仍需用户配置 API Key，环境搭建只负责依赖。
- 报告完成度时区分“已搭建/未搭建”，不要笼统说“全部完成”。
