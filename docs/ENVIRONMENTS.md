# 实验环境清单（Chapter 1–10）

> 实测日期：2026-08-08 · 主机：macOS（Apple Silicon）· 安装后磁盘剩余约 17GiB。
> 本文件只负责“依赖环境能否跑通”；真实模型实验仍需用户配置 API Key。

## 一、共享环境（仓库根 `.venv`）

- Python：3.12.11（uv 管理的 CPython，`uv sync` 自动创建）
- 包数量：330（首次输出 `Installed 330 packages`，重复执行输出 `Audited 330 packages`）
- 覆盖范围：ch1–ch10 全部 extra + `dev`

一条命令装完（等价于逐章执行）：

```bash
uv sync --locked \
  --extra ch1 --extra ch2 --extra ch3 --extra ch4 --extra ch5 \
  --extra ch6 --extra ch7 --extra ch8 --extra ch9 --extra ch10 \
  --extra dev
```

重复执行应显示 `Audited`。

### 网络排障（国内网络）

PyPI 直连超时（`curl -sS -o /dev/null -w '%{http_code}\n' --max-time 15 https://pypi.org/simple/pytz/`
返回 `000`）时，先确认本地代理再重试：

```bash
lsof -nP -iTCP -sTCP:LISTEN | rg 7890   # ClashX 默认端口
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
export ALL_PROXY=http://127.0.0.1:7890
```

**不要**在 `--locked` 下使用镜像源（`--default-index` / `--index-url`）：锁文件固定指向
PyPI，换源会直接触发 `--locked` 校验失败。

### Playwright Chromium（ch4 / ch8 / ch9 / ch10）

```bash
.venv/bin/python -m playwright install chromium
```

实测已安装 `chromium-1228` 与 `chromium_headless_shell-1228`
（用户级缓存 `~/Library/Caches/ms-playwright`）。

## 二、验证结果

### 1. 共享环境逐章 import 检查

全部通过（0 失败）：

| 章 | 模块 |
| --- | --- |
| ch1 | pdfplumber、PyPDF2、reportlab |
| ch2 | torch、transformers、fastapi、anthropic、google.genai、litellm、ollama、tiktoken、playwright |
| ch3 | chromadb、faiss、sentence_transformers、mem0（包名 mem0ai）、memobase |
| ch4 | cv2、fitz、browser_use、mcp、arxiv、yfinance、github（PyGithub 包）、langchain_openai |
| ch5 | fitz、constraint（python-constraint）、fastapi、mcp |
| ch6 | sklearn、scipy、plotly、numba、fish_audio_sdk |
| ch7 | torch、transformers、datasets、accelerate、peft、trl、bitsandbytes、wandb、librosa、snac |
| ch8 | chromadb、langchain_core、slack_sdk、browser_use |
| ch9 | aiortc、whisper（openai-whisper）、torch、playwright |
| ch10 | tiktoken、playwright、httpx |

注意：PyGithub 的导入名是 `github`（仓库代码也是 `import github`），不是 `PyGithub`。

### 2. 共享环境项目入口命令

对 29 个 `main.py` 入口执行 `--help`（失败时退回 `--dry-run`，每个入口限时 20s）：

- 26 个直接通过；
- `chapter2/attention_visualization/main.py`：交互式 ReAct demo（需真实模型/服务），
  改用 README 推荐的 `attention_cli.py --help` 验证通过；
- `chapter4/perception-tools/src/main.py`：需要 MCP v2，已按 README 单独建 venv（见下）；
- `chapter6/elo-leaderboard/main.py`：无 `--help`/`--dry-run`，入口为
  `python main.py bradley-terry|online-elo`，需要下载真实 Arena 数据，未自动验证
  （数据依赖型入口，依赖本身已装好）。

## 三、独立 venv 的项目

README 明确要求独立环境的项目共 5 个（含提示词未列出的
`chapter4/perception-tools` 与 `chapter9/end-to-end-speech`）：

| 项目 | Python | 包数 | 关键版本 / 上游提交 | 验证结果 |
| --- | ---: | ---: | --- | --- |
| `chapter9/computer-use-open-model` | 3.11.15 | 112 | browser-use 0.9.5 @ `ec9277c…`；补装 playwright、pydantic-settings；Chromium | `main.py --help` ✅（`--dry-run` 需 endpoint 配置）；`pytest -q` **7 passed** ✅ |
| `chapter10/generative-agents` | 3.11.15 | 46 | openai==0.27.10 等旧栈；上游 `/tmp/generative_agents` @ `fe05a71d…` | `run_campaign.py --help` ✅；`pytest -q` **23 passed** ✅ |
| `chapter10/use-computer-while-calling` | 3.12.13 | 132 | TalkAct 上游 @ `7d70007…`；Chromium | `bench/run_bench.py --help` ✅；上游无测试；真实基准需 API Key |
| `chapter4/perception-tools` | 3.12.13 | 78 | mcp 2.0.0（共享环境为 1.x，不兼容） | `test_imports.py` ✅、`smoke_test_mcp_v2.py`（126 tools）✅；pytest 有环境 caveat（见下） |
| `chapter9/end-to-end-speech` | 3.10.18 | 68 | transformers==4.51.0、torch==2.8.0、minicpmo-utils 1.0.6 | `pytest -q tests` **7 passed** ✅；`demo.py --help` ✅；真实运行需 Linux + CUDA |

### 各项目复现命令

```bash
# chapter9/computer-use-open-model
cd chapter9/computer-use-open-model
python3.11 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/pip install playwright pydantic-settings   # 固定 browser-use 提交漏声明
.venv/bin/python -m playwright install chromium
cp env.example .env

# chapter10/generative-agents（上游源码固定到 /tmp/generative_agents）
cd chapter10/generative-agents
python3.11 -m venv .venv
.venv/bin/pip install -r requirements.txt
git clone https://github.com/joonspk-research/generative_agents.git /tmp/generative_agents
git -C /tmp/generative_agents checkout --detach fe05a71d3e4ed7d10bf68aa4eda6dd995ec070f4

# chapter10/use-computer-while-calling（TalkAct）
git clone https://github.com/19PINE-AI/TalkAct.git chapter10/use-computer-while-calling
git -C chapter10/use-computer-while-calling checkout --detach 7d70007f72d45ddfc1a14e8e229b6d444e4919a2
cd chapter10/use-computer-while-calling
python3.12 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/python -m playwright install chromium

# chapter4/perception-tools（mcp>=2,<3，与共享环境不兼容）
cd chapter4/perception-tools
python3.12 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/pip install pytest pytest-asyncio
cp env.example .env

# chapter9/end-to-end-speech（Python 3.10 + transformers 4.51 固定栈）
cd chapter9/end-to-end-speech
uv venv .venv --python 3.10
uv pip install --python .venv/bin/python -r requirements.txt
```

### perception-tools 的 pytest 说明

- `test_imports.py` 是独立脚本（模块级 `sys.exit(1)`），不是 pytest 用例；全量
  `pytest -q` 收集时因 pytest 导入模式与 yfinance→requests 的交互报 5 个
  collection error。单独执行 `python test_imports.py` 或
  `pytest test_imports.py` 均正常。
- `test_expanded_catalog.py::test_code_interpreter_nonzero_exit_fails_closed`
  期望 `ProcessExecutionError`，实测得到 `FileNotFoundError`（沙箱进程执行器在当前
  环境不可用），属环境相关失败，不代表 MCP v2 依赖未装好。

## 四、未搭建 / 未覆盖项

- `chapter7/sesame`：固定 `transformers==4.52.3` + Unsloth/Triton，面向 Linux+GPU；
  macOS 不搭建（与根目录 ch7 extra 的 `transformers>=4.55` 冲突）。
- `chapter7/orpheus`：README 要求独立 venv 且与本机 CUDA 匹配的 torch/torchaudio，
  本机未搭建。
- `chapter8/prompt-distillation`：README 要求 Linux/CUDA 独立环境（含 vLLM），
  本机未搭建。
- `chapter7/retool` 的 SandboxFusion：README 走 conda（Linux），未搭建。
- `chapter9` 外部复现轨（claude-quickstarts / browser-use / XLeRobot / RoboCrew /
  lerobot-sim2real）：需要 Docker/真机/GPU，按各 README 手动克隆，不自动处理。
- `chapter9/end-to-end-speech` 真实实验：需 Linux + NVIDIA CUDA GPU（约 21GB 显存）
  并下载 MiniCPM-o 4.5 固定 revision；本机只完成依赖安装与离线测试。
- `chapter6/elo-leaderboard`：入口需下载真实 Arena 数据，无 `--help`/`--dry-run`，
  未自动验证。
- 所有真实模型实验仍需用户配置 API Key（`.env` 或环境变量），本清单只负责依赖环境。
- 外部 checkout（`chapter10/use-computer-while-calling/`、`/tmp/generative_agents`）
  与 `.venv`、`.env` 均不提交；显示为未跟踪属预期。
