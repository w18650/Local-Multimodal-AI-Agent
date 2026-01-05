# 本地 AI 智能文献与图像管理助手 (Local Multimodal AI Agent)

## 📖 1. 项目简介 (Project Introduction)
本项目是一个基于 Python 的本地化多模态 AI 智能助手，通过利用多模态技术解决本地大量 PDF 文献和图片管理困难的问题。

本项目采用 **RAG (检索增强生成)** 的核心思想，实现了以下功能：

* **🔍 深层语义搜索**: 不再局限于文件名匹配，系统能深入理解 PDF 内容，**精确返回相关的段落、文字片段及其对应的页码**。
* **📂 智能自动归档**: 能够理解论文的核心主题（如 "Computer Vision", "Natural Language Processing", "Reinforcement Learning"），自动将混乱的文件移动到分类文件夹中。
* **🖼️ 跨模态以文搜图**: 支持使用自然语言描述（如"a dog on the grass"）来搜索本地图片。

---

## 🚀 2. 核心功能 (Core Features)

### 2.1 智能文献管理
* **片段级语义搜索 (Fragment-level Search)**: 支持自然语言提问，返回最相关的论文片段、所在页码及文件路径，实现“所问即所得”。
* **批量一键整理**: 扫描混乱的文件夹，自动识别每一篇 PDF 的主题，将其移动归档到对应的子目录，并建立向量索引。
* **单文件自动处理**: 下载新论文后，通过命令行一键完成分类、移动和索引入库。

### 2.2 智能图像管理
* **以文搜图**: 利用 CLIP 多模态模型，支持通过自然语言描述检索本地图片库。

---

## 🛠️ 3. 技术选型 (Technical Stack)

本项目的核心组件如下：

| 模块 (Module) | 技术/模型 (Tech/Model) | 说明 (Description) |
| :--- | :--- | :--- |
| **文本嵌入** | **Sentence-Transformer** <br> (`all-MiniLM-L6-v2`) | 它负责将长篇论文切分为向量片段，支持高效的语义搜索和聚类。 |
| **图像/多模态** | **OpenAI CLIP** <br> (`clip-ViT-B-32`) | 它将图像和文本映射到同一个向量空间，从而实现通过自然语言描述来搜索本地图片。 |
| **自动分类** | **Embedding Similarity** <br> (基于向量相似度) | 利用 `all-MiniLM` 计算文档摘要与主题词（如"Computer Vision", "Natural Language Processing"）的余弦相似度，实现极速的零样本分类。 |
| **向量数据库** | **ChromaDB** | 它无需服务器配置（Serverless），直接以文件形式在本地持久化存储海量向量索引。 |
| **PDF 处理** | **PyPDF2** | 用于逐页提取论文文本，结合 **Sliding Window (滑动窗口)** 算法进行文本切片。 |
| **图像处理** | **Pillow (PIL)** | Python 图像处理标准库，用于加载、格式转换及预处理图片数据以适配 CLIP 模型输入。 |

---

## 💻 4. 环境配置与安装 (Environment Setup)

### 4.1 基础环境准备
建议使用 `Conda` 创建独立的虚拟环境，以避免依赖冲突。

```bash
# 1. 创建虚拟环境 (建议 Python 3.8 或 3.10)
conda create -n ai_agent python=3.10

# 2. 激活环境
conda activate ai_agent

```

### 4.2 安装依赖库

在项目根目录下执行以下命令安装所需库：

```bash
pip install chromadb sentence-transformers Pillow PyPDF2 tqdm

```

*(注意: 首次运行程序时，`sentence-transformers` 会自动下载模型权重文件，约需 300-500MB 空间，请保持网络通畅。)*

### 4.3 项目目录结构


```text
Multimodal-AI-Agent/
├── db/                 # ChromaDB 数据库存储目录 
├── papers/             # PDF论文 
├── images/             # 图片
├── main.py             # 主程序
├── agent.py            # 核心逻辑
└── README.md           # 说明文档

```

---

## 🖥️ 5. 详细使用说明与演示 (Usage & Demo)

所有功能均通过命令行工具 `main.py` 调用。

### ⚠️ 初始化必读

由于初次使用时数据库为空，请务必先运行 **5.1 批量整理** 或 **5.3 添加论文** 命令，否则搜索将无结果。

### 5.1 批量整理与初始化 (Batch Organize)

**功能**: 扫描指定文件夹下的所有 PDF，根据语义自动分类移动到子文件夹，并建立按页切片的向量索引。

**命令**:

```bash
python main.py organize_papers <文件夹路径> --topics "<主题1,主题2,主题3>"

```

**示例**:

```bash
python main.py organize_papers ./papers --topics "Computer Vision, Natural Language Processing, Reinforcement Learning"

```

**运行演示**:

![image](https://github.com/w18650/Local-Multimodal-AI-Agent/blob/main/assets/papers.png)

---

### 5.2 添加单篇论文 (Add Single Paper)

**功能**: 处理新下载的单篇 PDF，自动归类并追加索引到现有数据库中。

**命令**:

```bash
python main.py add_paper <PDF路径> --topics "<主题列表>"

```

**示例**:

```bash
python main.py add_paper ./papers/Attention_Is_All_You_Need.pdf --topics "Computer Vision, Natural Language Processing, Reinforcement Learning"

```

**运行演示**:

> *(此处展示：日志显示文件被分类移动，并建立了索引)*

---

### 5.3 语义搜索 - 查论文 (Semantic Search)

**功能**: 输入自然语言问题，返回最相关的论文片段、页码和路径。

**命令**:

```bash
python main.py search_paper "<英文问题或关键词>"

```

**示例**:

```bash
python main.py search_paper "mechanism of self-attention"

```

**运行演示**:

> 系统返回了具体的片段内容和页码，方便快速阅读原文。
> *(此处展示：Output 输出包含 [Page X] 和具体的 Fragment 文本)*

---

### 5.4 建立图片索引 (Index Images)

**功能**: 扫描图片文件夹，将图像特征存入向量数据库。

**命令**:

```bash
python main.py index_images <图片文件夹路径>

```

**示例**:

```bash
python main.py index_images ./images

```

**运行演示**:

> *(此处展示：图片扫描的进度条)*

---

### 5.5 以文搜图 (Search Image)

**功能**: 通过文本描述查找最匹配的图片。

**命令**:

```bash
python main.py search_image "<图片描述>"

```

**示例**:

```bash
python main.py search_image "a cute dog running on grass"

```

**运行演示**:

> *(此处展示：终端输出了最匹配的图片文件名和路径)*



```
