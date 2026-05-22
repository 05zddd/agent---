# 智扫通机器人智能客服

基于 **ReAct Agent + RAG（检索增强生成）** 架构的智能客服系统，专注于扫地/扫拖一体机器人领域的知识问答与个性化使用报告生成。

## 项目简介

本项目构建了一个具备自主推理与工具调用能力的 AI Agent，能够：

- **知识问答**：回答扫地机器人的选购、使用、故障排除、维护保养等专业问题
- **报告生成**：根据用户使用数据，自动生成个性化的月度使用报告与保养建议
- **环境适配**：结合天气、地理位置等信息，给出针对性的使用建议

## 技术架构

```
用户输入 (Streamlit UI)
    │
    ▼
ReactAgent (LangChain/LangGraph)
    │
    ├── 系统提示词 (动态切换)
    │   ├── main_prompt.txt    ← 客服答疑模式
    │   └── report_prompt.txt  ← 报告生成模式
    │
    ├── 中间件
    │   ├── monitor_tool         → 工具调用监控 + 上下文注入
    │   ├── log_before_model     → LLM 调用日志
    │   └── report_prompt_switch → 动态提示词切换
    │
    └── 工具集
        ├── rag_summarize        → RAG 知识库检索
        ├── get_weather          → 天气查询（模拟）
        ├── get_user_location    → 用户城市（模拟）
        ├── get_user_id          → 用户 ID（模拟）
        ├── get_current_month    → 当前月份（模拟）
        ├── fetch_external_data  → 外部数据读取
        └── fill_context_for_report → 报告模式触发
```

## 技术栈

| 组件 | 技术选型 |
|------|----------|
| **大语言模型** | 通义千问 Qwen3-Max（阿里云） |
| **嵌入模型** | Text Embedding v4（阿里云 DashScope） |
| **Agent 框架** | LangChain + LangGraph |
| **向量数据库** | ChromaDB |
| **前端界面** | Streamlit |
| **文本分割** | RecursiveCharacterTextSplitter |

## 项目结构

```
AI大模型RAG与智能体开发_Agent项目/
├── app.py                       # Streamlit 应用入口
├── md5.text                     # 知识库文件 MD5 去重记录
│
├── agent/                       # Agent 核心
│   ├── react_agent.py           # ReAct Agent 定义与流式执行
│   └── tools/
│       ├── agent_tools.py       # 7 个工具函数（RAG检索、天气、用户信息等）
│       └── middleware.py        # 3 个中间件（监控、日志、动态提示词）
│
├── rag/                         # RAG 检索增强生成
│   ├── rag_service.py           # 检索 + LLM 总结服务
│   └── vector_store.py          # ChromaDB 向量存储与文档加载
│
├── model/
│   └── factory.py               # 模型工厂（通义千问 + 嵌入模型）
│
├── config/                      # 配置文件
│   ├── agent.yml                # Agent 外部数据路径
│   ├── chroma.yml               # ChromaDB 参数配置
│   ├── rag.yml                  # 模型名称配置
│   └── prompts.yml              # 提示词文件路径
│
├── prompts/                     # 提示词模板
│   ├── main_prompt.txt          # 客服模式系统提示词
│   ├── report_prompt.txt        # 报告模式系统提示词
│   └── rag_summarize.txt        # RAG 总结提示词
│
├── data/                        # 知识库
│   ├── 选购指南.txt             # 53 条选购建议
│   ├── 故障排除.txt             # 200 条故障解决方案
│   ├── 维护保养.txt             # 200 条维护技巧
│   ├── 扫地机器人100问.txt      # 100 条常见问答
│   ├── 扫地机器人100问2.txt     # 补充问答内容
│   ├── 扫拖一体机器人100问.txt  # 100 条扫拖一体问答
│   └── external/
│       └── records.csv          # 模拟用户使用记录（10 用户 × 12 月）
│
├── utils/                       # 工具模块
│   ├── config_handler.py        # YAML 配置加载
│   ├── file_handler.py          # 文件处理（MD5、PDF/TXT 加载）
│   ├── logger_handler.py        # 日志系统
│   ├── path_tool.py             # 绝对路径解析
│   └── prompt_loader.py         # 提示词文件加载
│
└── logs/                        # 运行日志
```

## 快速开始

### 环境要求

- Python 3.10+
- 阿里云 DashScope API Key

### 安装步骤

1. **克隆项目**

```bash
git clone <your-repo-url>
cd AI大模型RAG与智能体开发_Agent项目
```

2. **安装依赖**

```bash
pip install streamlit langchain langchain-community langchain-chroma langchain-text-splitters langgraph chromadb pypdf
```

3. **配置 API Key**

设置阿里云通义千问 API Key 环境变量：

```bash
# Linux / macOS
export DASHSCOPE_API_KEY="your-api-key"

# Windows (CMD)
set DASHSCOPE_API_KEY=your-api-key

# Windows (PowerShell)
$env:DASHSCOPE_API_KEY="your-api-key"
```

4. **加载知识库**（首次运行）

```bash
python rag/vector_store.py
```

5. **启动应用**

```bash
streamlit run app.py
```

启动后在浏览器访问 `http://localhost:8501` 即可使用。

## 使用示例

### 知识问答

```
用户：小户型适合买什么样的扫地机器人？
客服：根据选购指南，小户型建议选择...
```

### 报告生成

```
用户：帮我生成我的使用报告
客服：[自动调用工具链：获取用户ID → 月份 → 数据]
      # 黑马程序员扫地机器人使用情况报告与保养建议
      ## 一、本月使用概况
      ...
```

## 核心特性

### ReAct 推理循环

Agent 遵循「思考 → 行动 → 观察 → 再思考」的推理流程，自主判断何时需要调用工具、调用哪个工具，最大支持 5 次工具调用迭代。

### 动态提示词切换

通过 LangGraph 中间件的 `@dynamic_prompt` 机制，Agent 在检测到报告生成场景时自动切换系统提示词，从客服角色切换为报告撰写角色。

### RAG 检索增强

知识库包含 650+ 条结构化问答数据，通过 ChromaDB 向量检索（Top-3）结合通义千问进行总结生成，确保回答的专业性和准确性。

### MD5 去重

知识库文件通过 MD5 哈希去重，避免重复加载相同内容到向量数据库。

## 配置说明

配置文件位于 `config/` 目录，均为 YAML 格式：

- **rag.yml**：指定聊天模型（默认 `qwen3-max`）和嵌入模型（默认 `text-embedding-v4`）
- **chroma.yml**：向量库集合名、持久化目录、检索数量 `k`、分块大小/重叠、支持文件类型
- **agent.yml**：外部数据文件路径配置
- **prompts.yml**：提示词文件路径映射

## 许可证

本项目仅供学习和研究使用。
