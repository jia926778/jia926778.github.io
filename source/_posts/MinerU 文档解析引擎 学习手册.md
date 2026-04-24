---
title: MinerU 文档解析引擎 学习手册
date: 2026-04-25 18:30:00
tags: [MinerU, RAG技术问答, AI大模型]
categories: MinerU
toc: true
---

# 《MinerU 文档解析引擎 学习手册》

## 文档核心说明

本文档适用于 AI 大模型 RAG 知识库搭建从业者、学术论文 / 企业文档数字化处理人员、Python 开发者、私有化部署运维人员，完整覆盖 MinerU 从入门到进阶的全链路知识，包括：

1. 产品定位与同类型工具对比选型

2. 全平台环境准备与三种安装部署方式

3. 4 种核心使用方式的实操教程与参数详解

4. 底层双引擎架构与端到端处理流程原理

5. VLM 引擎基于开源多模态大模型的微调全方案

6. 全场景常见问题排查与最佳实践

---

## 一、MinerU 基础认知

### 1\.1 产品核心定位

MinerU 是上海人工智能实验室 OpenDataLab 开源的**AI 原生端到端文档解析引擎**，核心能力是将 PDF、图片、DOCX 等文档高精度转换为结构化 Markdown/JSON 格式，专为大模型 RAG 知识库构建、学术论文解析、企业文档数字化设计，核心优势是复杂布局、数学公式、跨页表格的高保真还原，中文场景深度优化，支持全平台私有化部署。

### 1\.2 同类型工具对比与选型指南

#### 1\.2\.1 核心同赛道开源 AI 驱动竞品对比

|对比维度|MinerU|Unstructured|Marker|Nougat（Meta）|Docling|
|---|---|---|---|---|---|
|开源协议|AGPLv3|核心开源，企业版闭源收费|MIT|MIT 非商用授权|MIT|
|技术路线|VLM\+Pipeline 双引擎，布局 / 公式 / 表格 / OCR 专项模型拆分优化|规则 \+ 轻量 CV 模型，分块提取为主|端到端 VLM\+OCR 流水线，主打 PDF 转 Markdown|纯 Transformer 端到端模型，专为学术论文设计|IBM 研发，基于 Transformer 规则融合，轻量化解析|
|综合解析准确率（OmniDocBench）|90\.7%|\~68%|\~82%|\~75%（仅学术场景）|\~78%|
|公式识别（LaTeX）准确率|87\.4%|22%（免费版几乎不可用）|71%|82%（仅英文论文）|不支持|
|表格结构化准确率|85\.6%|61%|76%|不支持结构化输出|72%（简单表格）|
|多语言 OCR 支持|109 种语言，中文深度优化|\~30 种，中文支持一般|主流语言，中文适配一般|仅英文为主|主流语言，中文支持弱|
|复杂布局适配|多栏、跨页、图文混排、手写体全面支持|基础支持，复杂布局易丢失结构|较好支持，跨页表格能力弱|仅支持单 / 双栏学术论文|仅支持标准办公文档布局|
|处理速度（A100 GPU）|2\.1 页 / 秒（Pipeline 模式）|无 GPU 加速，CPU 0\.8 页 / 秒|1\.5 页 / 秒|0\.7 页 / 秒|无 GPU 加速，CPU 1\.2 页 / 秒|
|本地私有化部署|完全支持，推荐 GPU≥8GB|支持，Docker 一键部署|支持，pip 一键安装|支持，GPU≥10GB|支持，纯 CPU 即可运行|
|RAG/LLM 生态|官方 LangChain/LlamaIndex 集成，支持 MCP Server|生态最全，几乎所有 RAG 框架原生适配|基础集成，需二次开发|仅学术场景适配|轻量化集成，适合简单 RAG 场景|

#### 1\.2\.2 传统开源 PDF 解析工具对比

|对比维度|MinerU|PyMuPDF（fitz）|pdfplumber|
|---|---|---|---|
|技术路线|AI 驱动，VLM \+ 多模型融合|纯规则解析，无 AI 模型|规则 \+ 坐标匹配，无 AI 视觉能力|
|原生 PDF 文本提取准确率|93\.2%|91%|89%|
|扫描件 / 图片 PDF 处理|自动 OCR，精度 90%\+|不支持，需额外对接 Tesseract|不支持，需额外对接 OCR 工具|
|公式 / 表格处理|公式 LaTeX 输出，表格 HTML 结构化还原|公式完全不支持，简单表格基础提取|仅有线表格基础提取，公式完全不支持|
|复杂布局适配|多栏、跨页、图文混排智能排序|仅按坐标提取，易出现段落错乱|仅单栏布局适配良好，多栏易错位|
|处理速度|GPU 2\.1 页 / 秒，CPU 0\.3 页 / 秒|CPU 50 \+ 页 / 秒，速度极快|CPU 5\-10 页 / 秒，速度中等|
|部署与易用性|需 AI 环境配置，有一定上手门槛|pip 一键安装，几行代码即可运行|安装简单，API 友好，适合新手|

#### 1\.2\.3 商业 SaaS / 闭源文档解析服务对比

|对比维度|MinerU|LlamaParse|ABBYY FineReader|腾讯云 / 阿里云文档智能|
|---|---|---|---|---|
|开源 / 商业|开源免费，AGPLv3|闭源 SaaS，按页计费（免费额度 1000 页 / 月）|闭源买断制 / 订阅制|闭源 SaaS，按调用量计费|
|技术路线|本地双引擎 AI 解析|基于 GPT\-4o 的端到端语义解析|传统 OCR \+ 规则引擎，多年技术沉淀|国内大厂自研多模态大模型 \+ OCR 流水线|
|综合解析精度|开源赛道顶尖，复杂公式 / 表格超越多数商业工具|语义理解能力强，通用场景精度高|印刷体 OCR 精度顶尖，小语种支持全|中文场景深度优化，票据 / 证照专项能力强|
|数据隐私|本地部署，数据不出域，完全可控|云端处理，数据需上传至第三方服务器|本地版数据可控，云端版需上传|国内合规机房，符合等保要求，数据可控性优于海外服务|
|成本|完全免费，仅需承担服务器硬件成本|超过免费额度后，约 0\.003\-0\.01 美元 / 页，大规模使用成本高|商业授权千元起步，企业级万元级|按量计费，千次调用几十元至百元不等，大规模使用成本高|

#### 1\.2\.4 综合选型指南

1. **首选 MinerU 的场景**

    - 有**私有化部署、数据不出域**的强需求，处理敏感文档 / 内部资料；

    - 核心需求是**学术论文解析、数学公式 LaTeX 还原、复杂跨页表格结构化**；

    - 中文文档占比高，需要深度优化的中文 OCR 和布局理解能力；

    - 搭建 RAG 知识库，需要高保真结构化输出，提升下游检索和生成质量。

2. **优先选其他开源工具的场景**

    - 仅需**纯文本快速提取**，无公式 / 表格 / 复杂布局需求，选 PyMuPDF；

    - 企业级多格式文档 ETL 处理，需要全 RAG 生态原生适配，无复杂公式需求，选 Unstructured；

    - 个人用户轻量使用，需要开箱即用、极简安装，无复杂文档需求，选 Marker；

    - 仅处理英文学术论文，无表格结构化需求，选 Nougat。

3. **优先选商业工具的场景**

    - 无技术开发能力，无需私有化，需要开箱即用的云端服务，选 LlamaParse；

    - 企业批量纸质档案 / 印刷体文档数字化，需要极致的 OCR 精度和格式还原，选 ABBYY FineReader；

    - 国内企业处理票据、合同、证照等行业专项场景，有合规等保要求，选腾讯云 / 阿里云文档智能。

---

## 二、环境准备与安装部署

### 2\.1 硬件与系统要求

|解析后端|核心定位|系统支持|纯 CPU 运行|最低硬件要求|推荐硬件配置|
|---|---|---|---|---|---|
|pipeline|通用高精度解析（默认）|Linux/Windows/macOS|✅ 支持|CPU 4 核 \+、内存 16GB、存储 20GB SSD|CPU 8 核 \+、内存 32GB、NVIDIA GPU 6GB \+ 显存 / Apple Silicon|
|vlm 系列|端到端 VLM 高速解析|Linux/Windows|❌ 仅 GPU|NVIDIA GPU 8GB \+ 显存、内存 16GB|NVIDIA RTX 3090/4090 24GB 显存、内存 32GB|
|http\-client|远程服务调用|全平台|✅ 支持|内存 4GB、网络通畅|无额外硬件要求，依赖远程服务性能|

> 注意事项：
>
> 1. Windows 系统 Python 仅支持 3\.10\\3\.12（ray 依赖不兼容 3\.13）；Linux/macOS 支持 Python 3\.10\\3\.13
>
> 2. Linux 推荐 Ubuntu 20\.04/22\.04 LTS，macOS 需 14\.0 及以上版本
>
> 3. 离线部署需提前下载模型文件，预留至少 20GB 存储空间
>
>

### 2\.2 前置软件环境准备

#### 2\.2\.1 Python 环境配置（必选）

推荐使用 Anaconda/miniconda 创建隔离虚拟环境，避免依赖冲突：

```bash
# 1. 创建并激活虚拟环境（推荐Python 3.10，兼容性最佳）
conda create -n mineru python=3.10 -y
conda activate mineru

# 2. 升级pip并安装uv包管理器（大幅提升安装速度）
pip install --upgrade pip -i https://mirrors.aliyun.com/pypi/simple
pip install uv -i https://mirrors.aliyun.com/pypi/simple
```

#### 2\.2\.2 系统依赖补装（按需）

- **Linux/WSL2 系统**：解决图形库缺失、中文乱码报错

```bash
sudo apt-get update
sudo apt-get install libgl1-mesa-glx libglib2.0-0 -y
# 安装中文字体
sudo apt install fonts-noto-core fonts-noto-cjk -y
fc-cache -fv
```

- **macOS 系统**：安装 Homebrew 依赖

```bash
brew install libgl1 glib
```

#### 2\.2\.3 GPU 加速环境配置（可选，NVIDIA 显卡）

如需启用 GPU 加速，需提前安装对应版本的 CUDA 与 cuDNN，推荐 CUDA 12\.4/12\.8 \+ cuDNN 8\.9\+，兼容 PyTorch 2\.2\~2\.9，安装完成后通过 `nvidia\-smi` 命令验证。

### 2\.3 三种安装方式

#### 2\.3\.1 一键 pip/uv 安装（推荐，绝大多数用户）

```bash
# 全功能安装（包含pipeline、WebUI、API所有核心功能，兼容全平台）
uv pip install -U "mineru[all]" -i https://mirrors.aliyun.com/pypi/simple

# 安装完成后验证版本
mineru --version
```

> 国内用户必看：无法访问 HuggingFace 时，执行以下命令切换为国内 ModelScope 镜像源（全局生效）
>
>

```bash
# Linux/macOS 临时生效
export MINERU_MODEL_SOURCE=modelscope

# Windows CMD 临时生效
set MINERU_MODEL_SOURCE=modelscope

# Windows PowerShell 临时生效
$env:MINERU_MODEL_SOURCE = "modelscope"
```

#### 2\.3\.2 源码安装（开发者 / 二次开发）

```bash
# 1. 克隆源码仓库
git clone https://github.com/opendatalab/MinerU.git
cd MinerU

# 2. 可编辑模式安装（修改代码实时生效）
uv pip install -e .[all] -i https://mirrors.aliyun.com/pypi/simple
```

#### 2\.3\.3 Docker 容器化部署（生产环境 / 快速体验）

##### 单容器快速启动

```bash
# 1. 拉取官方镜像
docker pull opendatalab/mineru:latest

# 2. 启动容器（无GPU环境删除--gpus all参数）
docker run -d \
  --name mineru \
  --gpus all \
  --shm-size 32g \
  --ipc=host \
  -p 8000:8000 \
  -p 7860:7860 \
  -v ./mineru_data:/app/data \
  -e MINERU_MODEL_SOURCE=modelscope \
  opendatalab/mineru:latest
```

##### Docker Compose 一键部署（推荐生产环境）

```bash
# 1. 下载官方compose配置文件
wget https://gcore.jsdelivr.net/gh/opendatalab/MinerU@master/docker/compose.yaml

# 2. 按需启动服务
# 启动API服务
docker compose --profile api up -d
# 启动Gradio WebUI可视化界面
docker compose --profile gradio up -d
# 启动多卡负载均衡router服务
docker compose --profile router up -d

# 3. 验证服务状态
docker compose ps
curl http://localhost:8000/health
```

> 端口说明：8000 为 FastAPI 服务端口，7860 为 Gradio WebUI 端口，30000 为 OpenAI 兼容服务端口。
>
>

### 2\.4 离线部署配置

完全断网环境部署，需提前在有网络的环境下载模型文件，再拷贝到离线服务器：

```bash
# 1. 有网络环境下载全量模型
mineru-models-download --source modelscope --model_type all

# 2. 查看模型下载路径，将整个模型目录打包拷贝到离线服务器
# 3. 离线服务器编辑用户目录下的 mineru.json 配置文件，指定模型路径
{
  "models-dir": {
    "pipeline": "/opt/mineru/models/pipeline",
    "vlm": "/opt/mineru/models/vlm"
  },
  "config_version": "3.0.0"
}

# 4. 设置环境变量，禁用自动下载
export MINERU_MODEL_SOURCE=local
```

---

## 三、全场景核心使用教程

### 3\.1 命令行 CLI 使用（最常用）

#### 3\.1\.1 核心基础命令

```bash
# 1. GPU环境基础解析（自动下载模型，输出Markdown）
mineru -p 输入文件/目录路径 -o 输出目录路径

# 2. 纯CPU环境解析（指定pipeline后端）
mineru -p 输入文件/目录路径 -o 输出目录路径 -b pipeline

# 3. DOCX原生解析（3.0+版本支持，无需转PDF）
mineru -p test.docx -o ./output -b pipeline

# 4. 图片文档解析（支持jpg/png等格式）
mineru -p scan_image.jpg -o ./output -b pipeline
```

#### 3\.1\.2 核心参数详解

|参数简写|参数全称|功能说明|适用场景|
|---|---|---|---|
|\-p|\-\-path|输入文件 / 目录路径，支持 PDF / 图片 / DOCX|必选，指定解析源|
|\-o|\-\-output|输出目录路径，自动为每个文件创建子目录|必选，指定结果保存位置|
|\-b|\-\-backend|解析后端，可选 pipeline/vlm\-transformers/vlm\-sglang\-engine/hybrid\-http\-client|按硬件环境选择，CPU 强制用 pipeline|
|\-m|\-\-method|解析模式，可选 auto/txt/ocr|auto 自动识别文本 / 扫描件，ocr 强制全页 OCR|
|\-l|\-\-lang|OCR 语言，默认 ch（中英日繁混合），可选 en/japan/korean 等|非中文文档指定对应语言提升精度|
|\-\-start|\-\-start\-page\-id|解析起始页码（0 开始计数）|长文档指定解析范围|
|\-\-end|\-\-end\-page\-id|解析结束页码（0 开始计数）|长文档指定解析范围|
|\-\-formula|\-\-formula\-enable|是否启用公式识别，默认 true|无公式文档关闭可大幅提速|
|\-\-table|\-\-table\-enable|是否启用表格识别，默认 true|无表格文档关闭可提速|
|\-\-simple|\-\-simple\-output|仅输出 Markdown 和内容列表 JSON，关闭可视化文件|批量处理时精简输出|

#### 3\.1\.3 常用场景命令示例

```bash
# 示例1：批量解析整个目录的PDF，仅输出Markdown，关闭公式识别提速
mineru -p ./pdf_docs -o ./output -b pipeline --formula false --simple true

# 示例2：解析扫描件PDF，强制OCR模式，指定中文优化模型
mineru -p scanned.pdf -o ./output -b pipeline -m ocr -l ch_server

# 示例3：解析长文档指定页码（第10页到第50页，0开始计数）
mineru -p long_book.pdf -o ./output --start 9 --end 49

# 示例4：使用远程VLM服务解析，无需本地GPU
mineru-openai-server --port 30000  # 先启动OpenAI兼容服务
mineru -p test.pdf -o ./output -b hybrid-http-client -u http://127.0.0.1:30000
```

### 3\.2 Gradio WebUI 可视化使用（新手友好）

#### 3\.2\.1 启动命令

```bash
# 基础启动（本地访问，默认端口7860）
mineru-gradio

# 全网络可访问，指定端口，启用sglang加速
mineru-gradio --server-name 0.0.0.0 --server-port 7860 --enable-sglang-engine true
```

#### 3\.2\.2 使用步骤

1. 启动成功后，浏览器访问 `http://127\.0\.0\.1:7860`

2. 上传需要解析的文档（PDF / 图片 / DOCX）

3. 选择解析后端、语言、是否启用公式 / 表格识别等参数

4. 点击「开始解析」，等待处理完成

5. 在线预览 Markdown 结果，一键下载解析后的压缩包

### 3\.3 FastAPI 服务部署（开发者 / 生产集成）

#### 3\.3\.1 启动 API 服务

```bash
# 基础启动（默认端口8000）
mineru-api

# 全网络访问，指定端口，配置CORS
mineru-api --host 0.0.0.0 --port 8000 --cors-allow-origins "*"
```

#### 3\.3\.2 核心接口调用示例

启动成功后，浏览器访问 `http://127\.0\.0\.1:8000/docs` 查看完整 OpenAPI 接口文档。

##### 同步解析接口（适合小文件）

```python
import requests

url = "http://127.0.0.1:8000/file_parse"
params = {
    "backend": "pipeline",
    "lang": "ch",
    "method": "auto",
    "formula_enable": True,
    "table_enable": True
}
files = {"file": open("test.pdf", "rb")}

response = requests.post(url, params=params, files=files)
result = response.json()
print(result["markdown"])  # 解析后的Markdown内容
print(result["content_list"])  # 结构化内容列表
```

##### 异步任务接口（适合大文件 / 批量处理）

```python
import requests
import time

# 1. 提交任务
submit_url = "http://127.0.0.1:8000/tasks"
files = {"file": open("long_book.pdf", "rb")}
data = {"backend": "pipeline", "priority": 10}
response = requests.post(submit_url, files=files, data=data)
task_id = response.json()["task_id"]

# 2. 轮询任务状态
while True:
    status_url = f"http://127.0.0.1:8000/tasks/{task_id}"
    status = requests.get(status_url).json()
    if status["status"] == "completed":
        break
    print(f"任务状态：{status['status']}，进度：{status['progress']}%")
    time.sleep(2)

# 3. 获取解析结果
result_url = f"http://127.0.0.1:8000/tasks/{task_id}/data"
result = requests.get(result_url).json()
print(result["markdown"])
```

### 3\.4 Python SDK 二次开发

#### 3\.4\.1 基础同步调用示例

```python
import os
from pathlib import Path
import mineru

# 国内用户设置模型源
os.environ["MINERU_MODEL_SOURCE"] = "modelscope"

def parse_document(file_path: str, output_dir: str = "./output"):
    """
    解析单文档，返回Markdown内容
    """
    # 读取文件
    file_bytes = Path(file_path).read_bytes()
    file_name = Path(file_path).stem

    # 调用解析API
    result = mineru.do_parse(
        output_dir=output_dir,
        pdf_file_names=[file_name],
        pdf_bytes_list=[file_bytes],
        p_lang_list=["ch"],
        backend="pipeline",
        parse_method="auto",
        formula_enable=True,
        table_enable=True
    )

    # 读取生成的Markdown文件
    md_path = Path(output_dir) / f"{file_name}.md"
    if md_path.exists():
        return md_path.read_text(encoding="utf-8")
    return None

# 使用示例
if __name__ == "__main__":
    md_content = parse_document("test.pdf")
    print(md_content)
```

#### 3\.4\.2 异步批量处理示例

```python
import asyncio
import os
from pathlib import Path
import mineru

os.environ["MINERU_MODEL_SOURCE"] = "modelscope"

async def batch_parse_docs(file_dir: str, output_dir: str = "./batch_output"):
    """批量解析目录下所有PDF文件"""
    file_paths = list(Path(file_dir).glob("*.pdf"))
    tasks = []

    for file_path in file_paths:
        file_bytes = file_path.read_bytes()
        file_name = file_path.stem
        # 创建异步任务
        task = mineru.aio_do_parse(
            output_dir=str(Path(output_dir) / file_name),
            pdf_file_names=[file_name],
            pdf_bytes_list=[file_bytes],
            p_lang_list=["ch"]
        )
        tasks.append(task)

    # 并发执行所有任务
    await asyncio.gather(*tasks)
    print(f"批量解析完成，共处理{len(file_paths)}个文件")

# 运行
if __name__ == "__main__":
    asyncio.run(batch_parse_docs("./pdf_docs"))
```

### 3\.5 输出结果说明

#### 3\.5\.1 输出目录结构

```Plain Text
output/
 ├── 文件名/
 │ ├── images/                # 文档中提取的所有图片/图表
 │ ├── 文件名.md              # 核心输出：结构化Markdown文件
 │ ├── 文件名_content_list.json  # 段落级结构化内容JSON
 │ ├── 文件名_middle.json     # 完整中间结果JSON（全量信息）
 │ ├── 文件名_layout.pdf      # 版面检测可视化结果（simple=false时输出）
 │ ├── 文件名_spans.pdf       # 文本块可视化结果（simple=false时输出）
 │ └── 文件名_origin.pdf      # 原始文件备份（simple=false时输出）
```

#### 3\.5\.2 核心输出格式说明

1. **Markdown 文件**：高保真还原文档结构，标题分级、段落、列表、表格（HTML 格式）、公式（LaTeX 格式）、图片（本地路径引用）完整保留，可直接用于 RAG 知识库、Obsidian 等场景。

2. **content\_list\.json**：按阅读顺序排序的段落级结构化数据，包含文本类型、坐标、内容、图片路径等，适合二次开发与内容提取。

3. **middle\.json**：全量中间结果，包含版面分析、OCR、公式识别、表格识别的所有原始数据，适合深度定制化开发。

### 3\.6 RAG 场景最佳实践

1. **参数优化**：

    - 学术论文 / 公式密集文档：开启 `--formula true`，使用 `--lang ch_server` 提升 OCR 精度

    - 合同 / 办公文档：关闭公式识别 `--formula false`，大幅提升处理速度

    - 扫描件 / 图片文档：强制 OCR 模式 `-m ocr`，指定对应语言模型

2. **输出优化**：使用 `--simple true` 仅输出 Markdown，减少不必要的文件生成

3. **集成适配**：官方原生支持 LangChain/LlamaIndex 集成，也可直接对接 RAGFlow、Dify 等主流 RAG 平台。

---

## 四、底层原理与端到端处理流程

### 4\.1 核心双引擎协同架构

MinerU 的底层核心是**Pipeline 精细化流水线引擎**与**VLM 端到端语义引擎**的双引擎架构，两者可独立运行，也可混合调度，适配不同硬件环境与业务场景，所有模块通过统一的 `PageData` 数据结构实现信息传递与解耦，支持模块级的替换、优化与二次开发。

#### 4\.1\.1 两大核心引擎说明

- **Pipeline 引擎（默认核心）**：采用「解析→理解→生成」三层解耦的串行 \+ 并行混合流水线架构，将复杂的文档解析任务拆解为多个独立的专项子任务，每个子任务由专门微调的 AI 模型负责，实现「专模专用」的精度最优解，是唯一支持纯 CPU 运行的引擎。

- **VLM 引擎**：基于开源多模态大模型针对文档解析场景专项微调，直接输入文档页面渲染图像，端到端输出结构化 Markdown 内容，无需拆分多个子任务，对非常规排版、复杂嵌套布局的泛化性更强，GPU 环境下解析速度较 Pipeline 引擎提升 3 倍以上。

#### 4\.1\.2 双引擎核心能力对比

|核心维度|Pipeline 引擎|VLM 引擎|
|---|---|---|
|技术路线|多专项模型拆分优化，流水线式处理|端到端多模态大模型，统一语义理解|
|硬件支持|CPU/GPU/NPU 全适配|仅支持 GPU，需 SGLang 加速|
|最低显存要求|6GB|8GB（推荐 24GB）|
|核心优势|公式 / 表格 / 中文精度顶尖，可控性强，成本低|复杂布局泛化性好，速度快，支持指令定制|
|适用场景|通用文档解析、学术论文、私有化部署、CPU 环境|海量文档批量处理、非常规布局解析、GPU 高并发场景|

### 4\.2 Pipeline 引擎完整端到端处理流程

Pipeline 引擎的完整处理流程分为 6 个核心阶段，从文档输入到结构化输出全链路本地完成，无需依赖任何外部服务。

#### 阶段 1：文档预处理与智能分类

核心目标是完成文档的合法性校验、类型判断与标准化预处理，为后续解析提供高质量输入。

1. 文档读取与元数据解析，校验加密状态，修复损坏文档，原生解析非 PDF 格式避免格式丢失；

2. 智能判断文档类型（原生文本 PDF / 扫描件 PDF / 混合类 PDF），自动匹配对应解析链路；

3. 页面渲染与图像预处理，完成倾斜校正、去水印、去噪点、对比度增强，自动检测文档主语言。

#### 阶段 2：版面分析（Layout Detection）

本阶段是文档解析的核心骨架，核心目标是将文档页面从「无意义的画布像素」转换为「有语义的元素块」，解决 PDF 仅靠坐标定位、无逻辑结构的本质痛点。

1. 基于**DocLayout\-YOLO 文档布局检测模型**（中文场景专项微调），结合 LayoutLMv3 语义辅助，实现像素级元素定位与分类；

2. 精准识别元素坐标边界框，分为标题、正文、列表、公式、表格、图片、页眉页脚等 10 \+ 类语义标签；

3. 自动识别多栏布局，区分主副栏，标记干扰元素，为后续阅读顺序还原提供基础。

#### 阶段 3：并行专项内容解析

本阶段是 MinerU 精度的核心保障，基于版面分析的结果，对不同类型的元素块，调用对应的专项 AI 模型**并行执行内容解析**，大幅提升处理效率。

1. **文本 OCR 与原生文本提取**：采用「原生文本提取优先，OCR 兜底」的双链路设计，基于 PP\-OCRv5 多语言模型，支持 109 种语言与手写体识别；

2. **公式识别与 LaTeX 还原**：基于 UniMERNet 数学公式识别模型，支持复杂公式端到端转换为标准 LaTeX 格式，还原准确率达 87\.4%；

3. **表格结构化解析**：基于 RapidTable 模型，完成单元格分割、行列跨度识别、表头层级还原，支持跨页表格自动合并，输出 HTML 结构化标签；

4. **图片 / 图表提取与图文关联**：高清裁剪提取图片，基于空间位置与语义上下文匹配对应的标题、注释与正文引用，建立正确的图文关联。

#### 阶段 4：跨页内容聚合与逻辑还原

核心目标是将单页的元素块，还原为符合人类阅读习惯的全文档语义流，解决单页解析带来的内容割裂、顺序错乱问题。

1. 基于**结构语义图网络（SSGN）**，建模全文档元素的空间与逻辑关系，自动推理阅读骨架，解决多栏布局顺序错乱问题；

2. 完成跨页段落拼接、跨页表格合并、跨页公式修复，保证语义连贯性；

3. 基于全文档上下文，最终过滤页眉、页脚、水印、冗余内容。

#### 阶段 5：后处理与质量校验

核心目标是修正解析错误、规范格式、提升内容质量。

1. 基于上下文语义修正 OCR 错字、乱码、标点错误、公式符号缺失；

2. 统一标题层级、列表格式、公式标识符、图片引用路径，保证 Markdown 格式标准；

3. 自动完成解析结果完整性校验，标记低置信度识别结果，生成质量报告。

#### 阶段 6：结构化输出

完成最终的格式转换，输出多维度的结构化结果，适配不同下游场景，包括核心 Markdown 文件、段落级 content\_list\.json、全量中间结果 middle\.json 与辅助可视化文件。

### 4\.3 核心技术创新与差异化优势

1. **双引擎协同的架构设计**：区别于行业内单一的流水线或纯 VLM 方案，兼顾了精度、可控性、速度与硬件适配，覆盖全场景需求；

2. **语义引导的渐进式解析**：构建跨模块反馈闭环，粗粒度布局结果优化细粒度识别，识别结果反向校正布局分割，减少 30% 以上冗余计算的同时提升精度；

3. **中文场景深度优化**：所有核心模型均针对中文文档专项微调，中文解析精度远超海外开源工具；

4. **复杂元素高保真还原**：数学公式、跨页表格、复杂嵌套布局的还原能力，不仅领先开源竞品，甚至超越部分商业 SaaS 服务；

5. **全链路私有化部署**：所有模型与处理流程均支持本地离线运行，数据完全不出域，满足敏感文档处理的合规要求。

---

## 五、VLM 引擎微调全方案

MinerU 的 VLM 引擎以通义千问开源的 Qwen2\-VL 多模态大模型为核心基座，辅以自研架构改造，针对文档解析场景完成全链路微调优化，实现了 1\.2B 轻量化规模超越百亿级通用模型的文档解析性能。

### 5\.1 微调前置：基座选型与架构定制化改造

#### 5\.1\.1 核心基座选型

官方最终选定**Qwen2\-VL**作为底层基座，核心选型逻辑：

- 开源协议友好（Apache 2\.0），支持商用与二次修改，无知识产权风险；

- 原生深度优化中文能力，契合 MinerU 的核心中文文档场景；

- 原生支持高分辨率图像输入，适配文档小字体、细节密集的特性；

- 推理效率高、生态完善，兼容主流训练与推理框架，便于定制化改造与部署。

#### 5\.1\.2 面向文档解析的架构核心改造

1. **自研 NativeRes\-ViT 视觉编码器**：支持动态分辨率输入（最高 4096×4096 像素），解决传统固定分辨率 ViT 小字体丢失、细节模糊的痛点，支持分块处理，平衡计算量与细节保留。

2. **M\-RoPE 多维旋转位置编码**：替换原生 1D RoPE，让解码器适配不同分辨率、不同空间排布的文档元素，强化表格、公式的空间结构理解能力。

3. **Patch Merger 补丁合并器**：通过像素重排合并相邻 2×2 图像补丁，在保留空间细节的前提下，减少 40% 以上的视觉 token 数量，降低显存与计算开销。

4. **两阶段解耦推理架构**：改造为「布局分析→内容识别」两阶段架构，先降采样完成全页布局检测，再基于布局结果裁剪高分辨率区域做精细化识别，兼顾效率与精度。

### 5\.2 核心微调方案：三阶段渐进式训练 \+ 强化学习对齐

官方采用渐进式三阶段训练 \+ 最终 GRPO 强化学习对齐的微调方案，从通用模态对齐到文档专项能力，再到复杂场景优化，逐步收敛，避免灾难性遗忘。

#### 阶段 0：模态对齐预训练

- **核心目标**：打通视觉编码器与语言解码器的表征空间，建立「文档图像像素」与「文本语义」的基础关联；

- **数据集**：LLaVA\-Pretrain（300K 图文对）\+ LLaVA\-Instruct（600K 样本）；

- **训练策略**：先冻结主干权重，仅训练 2 层 MLP 层完成基础映射，再解冻全参数做轻量微调；

- **输出结果**：完成模态对齐的基座模型，具备基础的文档图像理解和指令跟随能力。

#### 阶段 1：文档解析大规模预训练

- **核心目标**：让模型习得布局分析与内容识别两大核心能力，建立对文档结构的通用认知；

- **数据集**：6550 万页大规模文档自动标注数据集，覆盖全品类文档，包含布局、文本、公式、表格全量标注信息；

- **训练任务**：拆分布局检测任务与单元素专项识别任务，分别优化全局结构理解与单元素识别精度；

- **训练策略**：全参数微调，开启文档场景专属数据增强，序列长度设置为 8192；

- **输出结果**：具备通用文档解析能力的预训练模型，可处理绝大多数常规文档。

#### 阶段 2：文档解析精细化微调

- **核心目标**：针对复杂场景难样本做定向优化，解决旋转表格、无线表格、复杂公式、多栏布局等边缘场景痛点，优化输出格式规范性；

- **数据集**：400K 页高质量难样本数据集，通过 IMIC 难样本挖掘技术筛选，其中 19\.2 万页为人工精标样本；

- **训练任务**：端到端文档解析任务，输入全页图像，目标输出标准 Markdown 格式内容；

- **训练策略**：全参数微调，加入格式约束损失，序列长度提升至 32768；

- **输出结果**：MinerU VLM 正式发布权重，通用场景 \+ 复杂场景均达到开源 SOTA 性能。

#### 阶段 3：GRPO 强化学习对齐（2\.5\-Pro 版本新增）

- **核心目标**：通过强化学习优化模型输出的格式一致性、内容完整性、逻辑连贯性；

- **奖励函数设计**：包含布局还原度、内容准确率、格式规范性、阅读顺序一致性、无冗余内容 5 个维度的密集奖励信号；

- **训练设置**：基于阶段 2 的权重，使用 GRPO 算法微调，仅微调顶层注意力层和输出层，避免破坏基础能力。

### 5\.3 微调效果保障：闭环自动化数据引擎

1. **数据筛选与多样性保障**：基于布局复杂度、文档类型、语言分布做分层抽样，保证训练数据覆盖全场景，避免类型偏置；

2. **自动化标注流水线**：先用 MinerU Pipeline 引擎做初始标注，再用专家模型做交叉校验与自动修正，最终通过规则校验过滤低质量数据，保证标注准确率 \&gt; 98%；

3. **IMIC 难样本挖掘机制**：通过多次推理结果的一致性识别模型薄弱点，优先人工精标后回流到微调数据集，形成闭环迭代；

4. **文档专属数据增强**：设计随机倾斜、缩放、噪声叠加、水印叠加等专属增强策略，模拟真实扫描件场景，提升模型泛化性。

### 5\.4 微调技术实现细节

1. **训练框架与生态兼容**：基于 Hugging Face Transformers、PyTorch Lightning 搭建，完全兼容 Hugging Face 生态，推理端适配 SGLang、vLLM、LMDeploy 等主流框架；

2. **微调方式选型**：官方训练采用全参数微调，针对用户二次微调原生支持 LoRA 参数高效微调，仅需 16GB 显存即可完成；

3. **损失函数设计**：主损失为交叉熵损失，辅以布局坐标回归损失、格式约束损失、模态对齐损失；

4. **训练优化策略**：采用 AdamW 优化器、余弦退火学习率调度、FSDP 全分片数据并行、梯度裁剪，保证训练稳定性。

### 5\.5 用户自定义领域微调实现方案

官方权重完全兼容 Hugging Face 生态，可基于标准工具链完成二次领域微调，适配法律、医疗、金融等专业文档场景。

#### 5\.5\.1 环境与硬件准备

- 硬件最低要求：NVIDIA GPU 16GB 显存（推荐 24GB\+），32GB 内存，100GB\+ SSD；

- 软件环境：Python 3\.10\+，PyTorch 2\.4\+，安装 Transformers、PEFT、Accelerate、Datasets、TRL 等依赖库。

#### 5\.5\.2 数据集准备

- 核心格式：成对的「文档页面 / 区域图像」\+「目标标注文本」，标注格式参考 MinerU 输出的`middle\.json`和标准 Markdown 文件；

- 数据规模：领域特定文档建议至少 1000 页标注数据，其中人工精标数据不少于 500 页，标注准确率 \&gt; 95%；

- 数据划分：训练集 80%、验证集 10%、测试集 10%；

- 格式转换：转换为 Hugging Face Dataset 格式，包含`image`、`text`字段。

#### 5\.5\.3 LoRA 微调核心配置

```python
from peft import LoraConfig
from trl import SFTTrainer
from transformers import TrainingArguments

# LoRA配置（适配单卡消费级GPU）
lora_config = LoraConfig(
    r=32,
    lora_alpha=64,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

# 训练参数配置
training_args = TrainingArguments(
    output_dir="./mineru-lora-finetuned",
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    learning_rate=2e-4,
    num_train_epochs=5,
    fp16=True,
    logging_steps=10,
    save_strategy="epoch",
    evaluation_strategy="epoch",
    optim="adamw_torch",
    lr_scheduler_type="cosine",
    warmup_ratio=0.05
)

# 初始化SFTTrainer，加载官方MinerU VLM权重与数据集
trainer = SFTTrainer(
    model="opendatalab/MinerU-VLM-2.5-Pro",
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=eval_dataset,
    peft_config=lora_config,
    max_seq_length=32768
)

# 启动训练
trainer.train()
```

#### 5\.5\.4 模型合并与部署

训练完成后，将 LoRA 适配器权重与原始模型权重合并，生成完整的微调后模型，可直接适配 SGLang/vLLM 推理框架，作为 MinerU 的 VLM 后端调用。

---

## 六、常见问题与报错排查

### 6\.1 安装相关问题

1. **报错：ImportError: \[libGL\.so\]\(libGL\.so\)\.1: cannot open shared object file**
解决方案：Linux/WSL2 系统安装缺失的图形库

    ```bash
    sudo apt-get update && sudo apt-get install libgl1-mesa-glx libglib2.0-0 -y
    ```

2. **安装后 mineru 命令找不到**
解决方案：确认已激活 conda 虚拟环境，重新执行安装命令，或使用 `python -m mineru` 替代直接调用。

3. **依赖冲突安装失败**
解决方案：使用 conda 新建干净的虚拟环境，严格使用 Python 3\.10 版本，通过 uv 安装避免依赖冲突。

### 6\.2 运行相关问题

1. **模型下载失败 / 速度慢**
解决方案：切换国内 ModelScope 镜像源

    ```bash
    export MINERU_MODEL_SOURCE=modelscope
    ```

2. **GPU 显存不足 / OOM 报错**
解决方案：切换到 pipeline 后端 CPU 模式运行；降低批处理大小 `export MINERU_BATCH_SIZE=1`；关闭公式 / 表格识别；长文档拆分页码分段解析。

3. **解析结果中文乱码 / 文字缺失**
解决方案：Linux 系统安装中文字体

    ```bash
    sudo apt install fonts-noto-cjk -y
    fc-cache -fv
    ```

4. **扫描件 PDF 解析内容为空**
解决方案：强制指定 OCR 模式解析

    ```bash
    mineru -p scanned.pdf -o ./output -b pipeline -m ocr
    ```

5. **公式渲染异常 / 识别错误**
解决方案：更新模型到最新版本，使用 VLM 后端提升公式识别精度，手动修改配置文件自定义 LaTeX 公式标识符。

---

## 核心知识点速览

- MinerU 是开源 AI 原生文档解析引擎，核心采用**Pipeline\+VLM 双引擎架构**，兼顾精度、可控性与硬件适配，中文场景优化领先。

- Pipeline 引擎是默认核心，支持纯 CPU 运行，采用多专项模型拆分优化，公式 LaTeX 还原准确率达 87\.4%，表格结构化准确率 85\.6%。

- VLM 引擎基于 Qwen2\-VL 开源基座定制化改造，采用三阶段渐进式微调方案，端到端解析速度较 Pipeline 引擎提升 3 倍以上，仅支持 GPU 运行。

- 安装推荐使用 uv pip 一键安装，国内用户需切换**ModelScope 镜像源**解决模型下载慢的问题，生产环境优先选择 Docker 容器化部署。

- 核心使用方式分为 CLI 命令行、Gradio WebUI、FastAPI 服务、Python SDK 四种，覆盖个人使用到企业级高并发集成全场景。

- Pipeline 引擎端到端处理分为 6 个核心阶段，核心是**版面分析搭建结构骨架 \+ 并行专项解析保障精度 \+ 跨页聚合还原语义逻辑**。

- 微调采用渐进式训练策略，从模态对齐到专项预训练，再到难样本精细化微调，最终通过 GRPO 强化学习优化输出质量。

- 所有核心模型与处理流程均支持**全链路私有化部署**，数据完全不出域，满足敏感文档处理的合规要求。

- 无公式的办公文档解析，关闭公式识别可大幅提升处理速度；扫描件文档需强制指定 OCR 模式保障解析效果。

- 同类型工具选型中，纯文本快速提取选 PyMuPDF，企业级多格式 ETL 选 Unstructured，无私有化需求的轻量场景选 LlamaParse。
