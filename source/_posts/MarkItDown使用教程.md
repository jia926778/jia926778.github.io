---
title: MarkItDown 使用指南
date: 2026-04-20 21:30:00
tags: [MarkItDown, RAG, 技术分享, 文档识别]
categories: MarkItDown
toc: true
---
# MarkItDown 使用指南

## 一、MarkItDown 基础使用教程

MarkItDown 是微软开源的 Python 库，核心能力是**将 PDF、Office、图片、音频、网页等异构文档，统一转换为标准化 Markdown 文本**，是 RAG 检索增强、文档批量处理的常用工具。

### 1.1 安装

#### 基础安装（核心功能）

```
pip install markitdown
```

#### 完整安装（支持所有格式，推荐）

```
# 支持PDF、Office、图片OCR、音频转写等全格式

pip install "markitdown[all]"
```

#### 按需安装（节省空间）

| 目标格式                      | 安装命令                               |
| ------------------------- | ---------------------------------- |
| PDF 文件                    | `pip install "markitdown[pdf]"`    |
| Office 文档（Word/PPT/Excel） | `pip install "markitdown[office]"` |
| 图片 OCR（提取图片内文字）           | `pip install "markitdown[ocr]"`    |
| 音频 / 视频（语音转文字）            | `pip install "markitdown[audio]"`  |
| 网页 / URL 链接               | 核心自带，无需额外安装                        |

### 1.2 核心支持格式

* 文档类：PDF、DOCX、PPTX、XLSX、CSV、TXT、MD、HTML

* 媒体类：PNG/JPG 等图片（OCR）、MP3/WAV 等音频（Whisper 转写）

* 网络类：网页 URL、YouTube 链接、GitHub 仓库链接

* 其他：ZIP 压缩包、JSON、XML 等

### 1.3 基础使用示例

#### 示例 1：本地文件转换

```
from markitdown import MarkItDown

import os

# 初始化实例
md = MarkItDown()

# 动态获取文件路径（避免路径报错）
current_dir = os.path.dirname(os.path.abspath(__file__))
file_path = os.path.join(current_dir, "data", "物流信息.pdf")

# 执行转换
result = md.convert(file_path)

# 核心输出
print("转换后的Markdown内容：")
print(result.text_content)  # 核心：转换后的完整Markdown文本

# 辅助属性
print("n文件格式：", result.file_format)  # 识别到的文件格式
print("转换耗时：", result.duration_ms, "ms")  # 转换耗时
print("页面数量：", result.page_count)  # 多页文档页数（PDF/PPT等）
```

#### 示例 2：网页 URL 转换

直接抓取网页内容并转为 Markdown：

```
from markitdown import MarkItDown

md = MarkItDown()

result = md.convert("https://example.com")

print(result.text_content)
```

#### 示例 3：图片 OCR 文字提取

需先安装 OCR 依赖 `pip install "markitdown[ocr]"`：

```
from markitdown import MarkItDown

md = MarkItDown(ocr_languages="chi_sim,eng")  # 开启中文+英文识别

result = md.convert("./data/发票截图.png")

print(result.text_content)
```

#### 示例 4：音频语音转文字

需先安装音频依赖 `pip install "markitdown[audio]"`，基于 OpenAI Whisper 实现：

```
from markitdown import MarkItDown

md = MarkItDown(whisper_model="base")  # 可调整模型大小

result = md.convert("./data/会议录音.mp3")

print(result.text_content)
```

### 1.4 进阶用法

#### 自定义初始化参数，优化转换效果

```
from markitdown import MarkItDown

md = MarkItDown(
    pdf_ocr=True,  # 开启PDF图片OCR，解决扫描件PDF内容缺失
    ocr_languages="chi_sim,eng",  # OCR识别语言（中文+英文）
    whisper_model="small",  # 音频转写模型，越大精度越高
    max_file_size_mb=200,  # 放宽文件大小限制，默认100MB
)

result = md.convert("./data/扫描件物流单.pdf")

print(result.text_content)
```

#### 批量转换文件夹内所有文件

批量处理文件，转换后自动保存为 Markdown 文件：

```python
from markitdown import MarkItDown

import os

# 初始化

md = MarkItDown()
source_dir = "./data"  # 源文件文件夹
output_dir = "./output_md"  # 转换后md保存文件夹

# 创建输出文件夹
os.makedirs(output_dir, exist_ok=True)

# 遍历所有文件
for filename in os.listdir(source_dir):
    file_path = os.path.join(source_dir, filename)
    if os.path.isdir(file_path):
        continue

    try:
        print(f"正在转换：{filename}")
        result = md.convert(file_path)
        # 保存为md文件
        md_filename = os.path.splitext(filename)[0] + ".md"
        with open(os.path.join(output_dir, md_filename), "w", encoding="utf-8") as f:
            f.write(result.text_content)
        print(f"转换完成：{md_filename}")
    except Exception as e:
        print(f"转换失败 {filename}：{str(e)}")
```

***

## 二、文档特殊元素处理：页眉页脚、水印

### 2.1 MarkItDown 原生处理现状

**MarkItDown 原生不支持自动识别、过滤或专门处理页眉页脚、水印及页码**。

由于 PDF 等格式本身**不会在文件结构中明确标记 “页眉 / 页脚 / 水印”**（这些元素在底层通常只是普通的文本或图形对象），MarkItDown 会将它们与正文内容一视同仁地提取并转换为 Markdown。

### 2.2 解决方案

#### 方案一：转换前预处理（推荐，效果最好）

在调用 `md.convert()` 之前，先用专门的库去除文档中的页眉页脚和水印。

##### 针对 PDF 文件（使用 `pdfplumber`）

`pdfplumber` 可以根据文本在页面上的**坐标位置**来判断并剔除页眉页脚。

```python
import pdfplumber
import fitz  # PyMuPDF
import io


def preprocess_pdf(pdf_path, margin=50):
    """
    PDF预处理：去除水印 + 裁剪掉页眉页脚区域
    """
    # 1. 去除水印
    doc = fitz.open(pdf_path)
    for page in doc:
        blocks = page.get_text("dict")["blocks"]
        for block in blocks:
            if "lines" not in block:
                continue
            for line in block["lines"]:
                for span in line["spans"]:
                    # 简单规则：如果是灰色文字或特定关键词，视为水印并移除
                    if (
                        span["color"] == 12632256
                        or "机密" in span["text"]
                        or "内部资料" in span["text"]
                    ):
                        page.add_redact_annot(span["bbox"], fill=(1, 1, 1))
        page.apply_redactions()
    temp_stream = io.BytesIO()
    doc.save(temp_stream)
    doc.close()

    # 2. 裁剪页面，去除页眉页脚
    temp_stream.seek(0)
    with pdfplumber.open(temp_stream) as pdf:
        cleaned_texts = []
        for page in pdf.pages:

            # 定义裁剪区域 (left, top, right, bottom)
            bbox = (50, 50, page.width - 50, page.height - 50)
            cropped_page = page.within_bbox(bbox)
            text = cropped_page.extract_text()
            if text:
                cleaned_texts.append(text)
    return "nn".join(cleaned_texts)
```

##### 针对 Word 文件（使用 `python-docx`）

Word 的页眉页脚存储在特定的节（Section）对象中，可以直接移除。

```PYTHON
from docx import Document

def preprocess_word(word_path, output_path):
    doc = Document(word_path)

    # 1. 删除所有节的页眉页脚
    for section in doc.sections:
        section.header.is_linked_to_previous = False
        section.footer.is_linked_to_previous = False
        for paragraph in section.header.paragraphs:
            paragraph.clear()
        for paragraph in section.footer.paragraphs:
            paragraph.clear()

    # 2. 删除水印
    for section in doc.sections:
        header = section.header
        for shape in header._element.xpath('.//w:drawing'):
            shape.getparent().remove(shape)
    doc.save(output_path)
```

#### 方案二：转换后后处理（基于规则清洗）

如果不想处理原文件，可以在拿到 MarkItDown 的输出后，利用**正则表达式**或**文本规则**清理重复出现的页眉页脚内容。

```python
import re

def clean_markdown_text(md_text):
    cleaned = md_text
    # 1. 移除常见的页码模式
    cleaned = re.sub(r'第s*d+s*页s*/s*共s*d+s*页', '', cleaned)
    cleaned = re.sub(r'Pages*d+s*ofs*d+', '', cleaned, flags=re.IGNORECASE)

    # 2. 移除特定的水印文字
    watermark_keywords = ["机密", "内部资料", "Company Confidential"]
    for keyword in watermark_keywords:
        cleaned = cleaned.replace(keyword, '')
    return cleaned
```

***

## 三、企业级文档清洗完整方案

针对企业文档中常见的乱码、重复内容、无意义符号、多余换行、广告、免责声明等问题，我们采用 **“MarkItDown 转换 + 自定义 Python 脚本后处理 / 预处理”** 的组合方案。

### 3.1 通用文本清洗器

```python
import re
from collections import Counter


class EnterpriseTextCleaner:
    def __init__(self):
        # 定义常见的无意义符号正则
        self.special_chars_pattern = re.compile(
            r'[■□▲△●○◆◇★☆▪▫→←↑↓│┃┆┇┊┋┌┐└┘├┤┬┴┼╭╮╰╯═║╔╗╚╝╠╣╦╩╬]'
        )

        # 定义常见的广告/免责声明关键词
        self.ad_keywords = [
            "扫码关注",
            "限时优惠",
            "点击领取",
            "广告",
            "免责声明",
            "风险提示",
            "本公司不承担",
        ]

        # 定义乱码常见模式
        self.gibberish_pattern = re.compile(r'[uFFFDu0080-u009F]')

    def clean(self, text):
        # 1. 去除乱码
        text = self._remove_gibberish(text)
        # 2. 统一换行符，去除多余空白
        text = self._normalize_whitespace(text)
        # 3. 去除无意义符号
        text = self._remove_special_chars(text)
        # 4. 去除重复段落/句子
        text = self._remove_duplicates(text)
        # 5. 去除广告和免责声明
        text = self._remove_ads_and_disclaimers(text)
        return text

    def _remove_gibberish(self, text):
        """去除乱码和不可见控制字符"""
        text = self.gibberish_pattern.sub('', text)
        text = re.sub(r'[^u4e00-u9fa5a-zA-Z0-9，。！？,.!?s-()（）]{3,}', '', text)
        return text

    def _normalize_whitespace(self, text):
        """统一换行，去除多余空行和空格"""
        text = text.replace('rn', 'n').replace('r', 'n')
        lines = [line.rstrip() for line in text.split('n')]
        cleaned_lines = []
        prev_empty = False
        for line in lines:
            is_empty = len(line.strip()) == 0
            if is_empty and prev_empty:
                continue
            cleaned_lines.append(line)
            prev_empty = is_empty
        return 'n'.join(cleaned_lines).strip()

    def _remove_special_chars(self, text):
        """去除装饰性符号"""
        return self.special_chars_pattern.sub('', text)

    def _remove_duplicates(self, text):
        """去除重复的段落（页眉页脚通常表现为每页重复的段落）"""
        paragraphs = text.split('n')
        counter = Counter([p.strip() for p in paragraphs if len(p.strip()) > 10])
        duplicate_paragraphs = {p for p, cnt in counter.items() if cnt > 2}
        cleaned_paragraphs = []
        for p in paragraphs:
            if p.strip() in duplicate_paragraphs:
                continue
            cleaned_paragraphs.append(p)
        return 'n'.join(cleaned_paragraphs)

    def _remove_ads_and_disclaimers(self, text):
        """基于关键词移除广告和免责声明段落"""
        paragraphs = text.split('n')
        cleaned_paragraphs = []
        for p in paragraphs:
            has_keyword = any(kw in p for kw in self.ad_keywords)
            if not has_keyword:
                cleaned_paragraphs.append(p)
        return 'n'.join(cleaned_paragraphs)
```

### 3.2 完整流水线调用示例

``` python
from markitdown import MarkItDown
import os

# 1. 定义路径
input_file = "./data/企业文档.pdf"
temp_clean_file = "./data/temp_clean.pdf"

# 2. 初始化工具
md = MarkItDown()
cleaner = EnterpriseTextCleaner()

# 3. 流程：先预处理 -> 再转换 -> 最后后处理
print("正在转换文档...")
result = md.convert(input_file)
raw_markdown = result.text_content
print("正在清洗文本...")
final_clean_text = cleaner.clean(raw_markdown)

# 3.3 保存结果
output_path = "./data/企业文档_最终清洗版.md"
with open(output_path, "w", encoding="utf-8") as f:
    f.write(final_clean_text)
print(f"处理完成！结果已保存至：{output_path}")
```

***

## 四、企业痛点处理总结

| 企业痛点          | 处理方式         | 推荐工具                                   |
| ------------- | ------------ | -------------------------------------- |
| **页眉页脚 / 页码** | 预处理（基于坐标裁剪）  | `pdfplumber`, `PyMuPDF`, `python-docx` |
| **水印**        | 预处理（覆盖或移除对象） | `PyMuPDF`, `python-docx`               |
| **乱码 / 控制字符** | 后处理（正则替换）    | Python `re`                            |
| **多余换行 / 空白** | 后处理（文本规范化）   | Python 字符串操作                           |
| **无意义符号**     | 后处理（正则过滤）    | Python `re`                            |
| **重复内容**      | 后处理（频率统计）    | `collections.Counter`                  |
| **广告 / 免责声明** | 后处理（关键词匹配）   | 自定义规则列表                                |
