---
title: AI Agent的设计模式
date: 2026-01-05
category: insight
tags: [AI Agent, LangChain, ReAct, Planning]
---

# AI Agent的设计模式

AI Agent(智能代理)正在从概念走向实用。本文将深入探讨构建有效AI Agent的核心设计模式和关键组件。

## 什么是AI Agent?

AI Agent是能够**自主感知环境、制定计划并采取行动**以实现目标的智能系统。与传统的问答式LLM不同,Agent具有:

- 🎯 **目标导向**:明确的任务目标
- 🔄 **迭代执行**:多步骤推理和行动
- 🛠️ **工具使用**:调用外部API和工具
- 💾 **记忆能力**:保持上下文和历史信息

## 核心架构:ReAct模式

**ReAct**(Reasoning + Acting)是目前最流行的Agent架构,交替进行推理和行动:

```
思考(Thought) → 行动(Action) → 观察(Observation) → 思考 → ...
```

### 示例:旅行规划Agent

```
User: 帮我规划一个三天的北京旅行

Thought: 我需要先了解北京的主要景点和天气情况
Action: search("北京旅游景点推荐")
Observation: 故宫、长城、颐和园、天坛...

Thought: 还需要查看未来三天的天气
Action: get_weather("北京", days=3)
Observation: 晴天,温度15-25°C

Thought: 现在可以制定行程了
Action: create_itinerary(...)
```

## 关键组件详解

### 1. 规划器(Planner)

将复杂任务分解为可执行的子任务。

**策略:**
- **分层规划**:先制定高层计划,再细化每一步
- **动态调整**:根据执行结果调整计划
- **并行执行**:识别可并行的子任务

**实现示例:**

```python
class TaskPlanner:
    def decompose(self, task):
        """将任务分解为子任务"""
        prompt = f"""
        任务: {task}
        请将此任务分解为具体的步骤:
        """
        steps = llm.generate(prompt)
        return self.parse_steps(steps)
    
    def replan(self, failed_step, error):
        """任务失败时重新规划"""
        prompt = f"""
        步骤 "{failed_step}" 失败,错误: {error}
        请提供替代方案:
        """
        return llm.generate(prompt)
```

### 2. 记忆系统(Memory)

Agent需要记住对话历史、中间结果和学到的经验。

**记忆类型:**

| 类型 | 用途 | 实现 |
|------|------|------|
| 短期记忆 | 当前对话上下文 | 对话历史缓冲 |
| 长期记忆 | 持久化知识 | 向量数据库 |
| 工作记忆 | 中间计算结果 | 键值存储 |
| 情景记忆 | 过往经验 | 检索增强 |

**向量检索示例:**

```python
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings

class AgentMemory:
    def __init__(self):
        self.vectorstore = Chroma(
            embedding_function=OpenAIEmbeddings()
        )
    
    def remember(self, experience):
        """存储经验"""
        self.vectorstore.add_texts([experience])
    
    def recall(self, query, k=3):
        """检索相关经验"""
        return self.vectorstore.similarity_search(query, k=k)
```

### 3. 工具库(Tools)

赋予Agent与外部世界交互的能力。

**常见工具类型:**
- 🔍 **搜索工具**:Google Search, Wikipedia
- 🧮 **计算工具**:Python REPL, 计算器
- 📊 **数据工具**:SQL查询, API调用
- 📝 **内容工具**:文件读写, 邮件发送

**工具定义示例:**

```python
from langchain.tools import Tool

def search_web(query: str) -> str:
    """搜索网络获取信息"""
    # 实际调用搜索API
    return search_api.query(query)

tools = [
    Tool(
        name="WebSearch",
        func=search_web,
        description="当需要获取最新信息时使用此工具"
    ),
    Tool(
        name="Calculator",
        func=calculator,
        description="执行数学计算"
    )
]
```

### 4. 执行引擎(Executor)

协调各组件,执行Agent的决策循环。

```python
class AgentExecutor:
    def __init__(self, llm, tools, memory, max_iterations=10):
        self.llm = llm
        self.tools = {t.name: t for t in tools}
        self.memory = memory
        self.max_iterations = max_iterations
    
    def run(self, task):
        for i in range(self.max_iterations):
            # 1. 思考下一步
            thought = self.llm.think(task, self.memory.get_context())
            
            # 2. 决定行动
            action = self.parse_action(thought)
            
            if action.type == "FINISH":
                return action.result
            
            # 3. 执行工具
            tool = self.tools[action.tool_name]
            observation = tool.run(action.input)
            
            # 4. 更新记忆
            self.memory.add(thought, action, observation)
        
        return "达到最大迭代次数"
```

## 高级设计模式

### 1. 多Agent协作

复杂任务可以由多个专门化的Agent协作完成。

**角色分工:**
- **研究员Agent**:收集和分析信息
- **规划员Agent**:制定执行计划
- **执行员Agent**:实施具体操作
- **评审员Agent**:检查和验证结果

### 2. 自我反思(Self-Reflection)

Agent评估自己的输出,发现错误并改进。

```python
def reflect(self, output):
    prompt = f"""
    我的输出: {output}
    
    请评估:
    1. 是否完整回答了问题?
    2. 是否有逻辑错误?
    3. 如何改进?
    """
    critique = self.llm.generate(prompt)
    
    if "需要改进" in critique:
        return self.improve(output, critique)
    return output
```

### 3. 人机协作(Human-in-the-Loop)

在关键决策点请求人类确认。

```python
def execute_action(self, action):
    if action.risk_level == "HIGH":
        approval = input(f"确认执行: {action}? (y/n)")
        if approval != 'y':
            return "用户取消操作"
    
    return action.execute()
```

## 实战案例:代码审查Agent

### 需求

自动审查GitHub PR,检查代码质量、安全问题和最佳实践。

### 设计

```python
class CodeReviewAgent:
    def __init__(self):
        self.tools = [
            GitHubTool(),      # 获取PR内容
            LinterTool(),      # 静态分析
            SecurityTool(),    # 安全扫描
            TestTool()         # 运行测试
        ]
    
    def review(self, pr_url):
        # 1. 获取PR信息
        pr_data = self.tools['GitHub'].get_pr(pr_url)
        
        # 2. 并行执行检查
        results = asyncio.gather(
            self.check_style(pr_data),
            self.check_security(pr_data),
            self.check_tests(pr_data)
        )
        
        # 3. 生成审查报告
        report = self.generate_report(results)
        
        # 4. 发布评论
        self.tools['GitHub'].post_comment(pr_url, report)
```

### 效果

- ⚡ **速度**:5分钟内完成审查(人工需30分钟)
- 🎯 **准确性**:发现87%的常见问题
- 📈 **一致性**:审查标准统一

## 挑战与解决方案

### 1. 幻觉问题

**问题:**LLM可能生成不存在的工具或错误的参数

**解决:**
- 严格的工具schema验证
- 少样本示例(Few-shot examples)
- 输出格式约束(如JSON mode)

### 2. 成本控制

**问题:**多次LLM调用成本高昂

**解决:**
- 使用小模型处理简单任务
- 缓存常见查询结果
- 设置最大迭代次数

### 3. 可靠性

**问题:**Agent可能陷入循环或产生错误行动

**解决:**
- 实现超时和重试机制
- 添加行动验证器
- 记录和监控Agent行为

## 未来趋势

1. **多模态Agent**:处理文本、图像、视频的统一Agent
2. **持续学习**:从交互中学习,不断改进
3. **可解释性**:清晰展示推理过程
4. **安全对齐**:确保Agent行为符合人类价值观

## 总结

构建有效的AI Agent需要:

- ✅ 清晰的架构设计(推荐ReAct)
- ✅ 完善的记忆系统
- ✅ 丰富的工具库
- ✅ 鲁棒的执行引擎
- ✅ 持续的监控和优化

AI Agent正在快速发展,掌握这些设计模式将帮助你构建更强大、更可靠的智能系统。

---

**推荐资源:**
- LangChain文档: https://python.langchain.com/docs/modules/agents/
- ReAct论文: "ReAct: Synergizing Reasoning and Acting in Language Models"
- AutoGPT: https://github.com/Significant-Gravitas/Auto-GPT
