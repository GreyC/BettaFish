# BettaFish 5个AI Agent 架构深度分析

## 目录
- [为什么需要5个AI Agent？](#为什么需要5个ai-agent)
- [5个AI Agent详细对比](#5个ai-agent详细对比)
  - [1. ForumHost (论坛主持人)](#1-forumhost-论坛主持人)
  - [2. Query Agent (精准信息搜索专家)](#2-query-agent-精准信息搜索专家)
  - [3. Media Agent (多模态内容分析专家)](#3-media-agent-多模态内容分析专家)
  - [4. Insight Agent (私有数据库挖掘专家)](#4-insight-agent-私有数据库挖掘专家)
  - [5. Report Agent (智能报告生成专家)](#5-report-agent-智能报告生成专家)
- [5个Agent对比一览表](#5个agent对比一览表)
- [为什么这种设计能突破信息茧房？](#为什么这种设计能突破信息茧房)

---

## 为什么需要5个AI Agent？

### 核心设计哲学
**分工明确、视角互补、智能协作**。单一Agent存在三大致命缺陷：

1. **视角局限** - 一种思维模式无法捕捉多面信息
2. **工具同质化** - 相同工具集导致分析路径单一
3. **信息茧房** - 单Agent会强化自己的偏见

通过**论坛辩论机制**（ForumEngine）让专业Agent互相挑战，像人类专家团队那样通过碰撞产生高质量集体智能。

---

## 5个AI Agent详细对比

### 1. ForumHost (论坛主持人)

**角色定位**：多Agent协作的"智能裁判"和"思维催化剂"

**核心职责**：
- 使用 **Qwen3-235B** 作为主持人模型
- 监控INSIGHT/MEDIA/QUERY三个Agent的讨论
- 每轮生成综合性发言，引导讨论深入
- 识别事实错误、逻辑矛盾、观点分歧
- 提出关键问题和后续研究方向

**独特工作机制**：
```python
# ForumEngine/llm_host.py:146-164
- 事件梳理：自动识别关键事件、人物、时间节点
- 观点整合：综合不同Agent视角，找出共识与分歧
- 趋势预测：基于已有信息分析舆情发展趋势
- 问题引导：提出2-3个值得深入探讨的关键问题
```

**与其他Agent的根本区别**：
- **不直接搜索数据**，而是分析和引导其他Agent的讨论
- **使用最大参数模型**（235B），确保思维深度
- **多轮交互**（论文式多轮辩论机制）
- **全局视角**，连接不同Agent的专业洞察

---

### 2. Query Agent (精准信息搜索专家)

**角色定位**：**广度优先**的国内外信息导航员

**专属工具集** (Tavily引擎 - 6种专业工具)：
```python
- basic_search_news: 快速通用新闻搜索
- deep_search_news: 深度新闻分析（返回AI摘要）
- search_news_last_24_hours: 追踪24小时最新动态
- search_news_last_week: 周度舆情回顾
- search_images_for_news: 多媒体配图搜索
- search_news_by_date: 指定历史日期范围搜索（YYYY-MM-DD格式）
```

**核心差异点**：
- **信息来源**：国内外通用网络搜索（Google/Bing等）
- **时效性最强**：覆盖过去1天到1年的历史新闻
- **广度覆盖**：不局限于社交平台，涵盖全网信息
- **专业特长**：追踪突发新闻、事件脉络、媒体报道

**使用场景示例**：
```
Query: "某某公司最新产品发布舆情"
→ search_news_last_24_hours: 获取最新媒体报道
→ search_images_for_news: 收集发布会现场图片
→ deep_search_news: 分析产品行业影响力
```

---

### 3. Media Agent (多模态内容分析专家)

**角色定位**：**模态维度突破**的视觉与结构化信息解析专家

**专属工具集** (Bocha多模态引擎 - 5种专业工具)：
```python
- comprehensive_search: 全面综合搜索（网页+图片+AI总结+模态卡）
- search_for_structured_data: 结构化数据查询（天气/股票/百科/医疗卡）
- web_search_only: 纯网页快速搜索（无AI总结）
- search_last_24_hours: 24小时内最新信息
- search_last_week: 本周主要报道
```

**核心差异化能力**：

1. **模态卡解析** (MediaEngine/tools/search.py:69-76)
```python
@dataclass
class ModalCardResult:
    card_type: str  # weather_china, stock, baike_pro, medical_common
    content: Dict[str, Any]  # 结构化可直接使用的数据
```
- 天气卡片：直接解析温度/湿度/风速
- 股票卡片：实时价格/涨跌幅/成交量
- 百科卡片：结构化知识点
- 医疗卡片：症状/诊断/治疗信息

2. **多模态融合**：同时处理网页、图片、视频、结构化数据

3. **AI追问建议**：返回`follow_ups`，主动引导Agent深入提问

**独特价值**：
- 唯一能解析**短视频内容**的Agent（抖音、快手）
- 唯一能提取**搜索引擎结构化卡片**的Agent
- 理解视觉内容的传播效果和媒体表现

**使用场景示例**：
```
Query: "某明星演唱会舆情"
→ comprehensive_search: 解析现场视频片段情感
→ search_images_for_news: 分析观众反应图片
→ ModalCardResult: 获取票房结构化数据
```

---

### 4. Insight Agent (私有数据库挖掘专家)

**角色定位**：**深度优先**的社交媒体舆情矿工

**专属工具集** (自研MediaCrawlerDB引擎 - 5种+情感分析)：
```python
# InsightEngine/tools/search.py:63-398
- search_hot_content: 查找热点内容（加权热度算法）
- search_topic_globally: 全局话题搜索（全库跨表查询）
- search_topic_by_date: 按日期范围搜索话题
- get_comments_for_topic: 专门获取公众评论
- search_topic_on_platform: 平台定向精确搜索（B站/微博/抖音等7个平台）
- analyze_sentiment: 多语言情感分析（支持22种语言）
```

**核心差异化能力**：

1. **私有数据库访问** (MediaCrawlerDB)
```python
# 直连由MindSpider爬取的社交媒体数据
覆盖平台: bilibili, weibo, douyin, kuaishou, xhs, zhihu, tieba
数据类型: 帖子内容、评论、互动数据（点赞/转发/评论）
```

2. **智能关键词优化中间件** (InsightEngine/tools/keyword_optimizer.py)
```python
# 使用Qwen3将Agent搜索词优化为"网民真实用语"
输入: "武汉大学舆情管理"
输出: ["武大", "武汉大学", "学校管理", "教育", "高校", "学生"]
# 避免:"公众态度分析"、"未来展望"等官方术语
```

3. **深度情感分析集成**
- 多语言情感分析（WeiboMultilingualSentiment）
- 自动对搜索结果进行情感打分
- 识别正负情感比例、情感强度

4. **热度算法** (InsightEngine/tools/search.py:97-159)
```python
# 加权热度计算
W_LIKE = 1.0      # 点赞权重
W_COMMENT = 5.0   # 评论权重
W_SHARE = 10.0    # 转发权重（最高）
W_VIEW = 0.1      # 浏览权重
```

**独特价值**：
- **唯一访问私有数据库**的Agent
- **唯一集成情感分析**的Agent
- **深度理解社交媒体生态**（知道网民怎么说）

**使用场景示例**：
```
Query: "某品牌用户满意度"
→ search_topic_globally: 全局搜索品牌提及
→ get_comments_for_topic: 提取真实用户评论
→ 情感分析: 自动计算正负情感比例（如：65%正面，25%负面，10%中性）
→ 关键词优化: 优化搜索词提升召回率
```

---

### 5. Report Agent (智能报告生成专家)

**角色定位**：**信息整合与可视化呈现**的编排师

**核心职责**：
```python
# ReportEngine/agent.py:180-295
1. 模板智能选择: 从markdown模板库选择最适合的模板
   - 社会公共热点事件分析模板
   - 商业品牌舆情监测模板
   - 竞品分析模板
   - 用户可自定义上传模板

2. 多轮HTML生成:
   - 整合Query/Media/Insight三份报告
   - 融合ForumEngine的论坛日志（多轮讨论记录）
   - 生成美观的HTML报告（含图表、时间线、数据可视化）

3. 文件基准管理:
   - 跟踪各Agent报告文件数量变化
   - 确保所有Agent完成分析后再生成最终报告
```

**核心差异化能力**：
- **唯一不直接搜索数据**的Agent
- **唯一专注报告呈现**的Agent
- **模板驱动**：支持丰富的markdown模板库
- **多模态输出**：生成HTML+CSS+JS的交互式报告

**工作流程**：
```
Step 1: 等待三个Agent报告就绪
Step 2: 智能选择模板（基于Query分析主题）
Step 3: 多轮LLM生成（整合所有输入）
Step 4: 输出HTML格式的最终研究报告
```

---

## 5个Agent对比一览表

| 维度 | ForumHost | Query Agent | Media Agent | Insight Agent | Report Agent |
|------|-----------|-------------|-------------|---------------|--------------|
| **核心模型** | Qwen3-235B | 可配置OpenAI兼容 | 可配置OpenAI兼容 | 可配置OpenAI兼容 | 可配置OpenAI兼容 |
| **数据范围** | 内部讨论日志 | 全网公开搜索 | 全网多媒体搜索 | 私有爬虫数据库 | 整合前3者结果 |
| **信息来源** | 3个Agent发言 | Tavily新闻索引 | Bocha多模态引擎 | MindSpider爬取的7平台 | 三者报告+论坛日志 |
| **专业特长** | 思维引导、辩论主持 | 广度搜索、时效性 | 多模态解析 | 深度情感分析、热度计算 | 信息整合、可视化 |
| **独特工具** | 对话分析、趋势预测 | 6种Tavily搜索 | 模态卡解析、AI追问建议 | 关键词优化、情感分析、热度算法 | 模板引擎、HTML生成 |
| **输出形式** | 主持人发言（引导性） | Markdown段落报告 | Markdown段落报告 | Markdown段落报告 | HTML交互报告 |
| **协作角色** | 裁判+催化剂 | 信息收集者 | 多媒体解析者 | 深度挖掘者 | 报告整合者 |
| **AI能力** | 最强推理（235B） | 通用LLM | 通用LLM | 通用LLM+专用小模型 | 通用LLM |

---

## 为什么这种设计能突破信息茧房？

### 1. **认知视角多样化**
- **Query Agent**: 像**记者**，广度覆盖、官方视角
- **Insight Agent**: 像**社会学家**，深入民间、情感洞察
- **Media Agent**: 像**媒体分析师**，视觉传播、结构化数据

### 2. **思维碰撞机制**

ForumEngine每轮都生产生：
- **事件梳理**（时间线）
- **观点对比**（共识 vs 分歧）
- **错误纠正**（事实核查）
- **问题深化**（提出新研究方向）

### 3. **数据护城河**
- **私有数据库**（Insight Agent）vs **公开搜索**（Query Agent）
- **即时数据**（Query Agent）vs **历史沉淀**（Insight Agent）
- **文本数据**（Query Agent）vs **多模态数据**（Media Agent）

### 4. **工具差异化**
不同工具集确保每个Agent走独立的分析路径：
- Query用Tavily（传统搜索引擎API）
- Media用Bocha（多模态AI搜索引擎）
- Insight直连私有数据库

---

## 总结

这种设计完美诠释了 **"三个臭皮匠，胜过诸葛亮"** 的多智能体哲学！

通过以下核心机制实现突破：
1. **专业分工** - 每个Agent有独特的工具集和思维模式
2. **智能辩论** - ForumHost引导多轮思维碰撞
3. **数据互补** - 公域+私域、文本+多模态、广度+深度
4. **质量把控** - 主持人模型识别错误、整合观点、预测趋势

**始于舆情，而不止于舆情**。这套架构可作为通用数据分析引擎，应用于金融分析、市场研究、竞品分析等多种场景。
