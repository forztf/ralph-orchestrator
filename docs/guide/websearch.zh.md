# WebSearch 集成指南

## 概述

Ralph Orchestrator 现在包含完整的 WebSearch 支持,适用于 Claude 适配器,使 Claude 能够搜索网络以获取当前信息、研究主题以及访问超出其知识截止日期的数据。

## 功能特性

WebSearch 允许 Claude:
- 搜索当前事件和最新新闻
- 研究技术文档和最佳实践
- 查找库和框架的最新信息
- 从多个来源收集数据
- 访问实时信息(天气、股票价格等)

## 配置

### 默认配置

使用 Claude 适配器时,WebSearch **默认启用**。无需额外配置。

### 显式配置

您可以通过多种方式显式控制 WebSearch:

#### 1. 通过 CLI(自动)

使用 Claude 运行 Ralph 时,WebSearch 会自动启用:

```bash
ralph -a claude  # WebSearch 默认启用
```

#### 2. 通过适配器配置

```python
from src.ralph_orchestrator.adapters.claude import ClaudeAdapter

# 创建适配器并启用 WebSearch(默认)
adapter = ClaudeAdapter()
adapter.configure(enable_web_search=True)  # 这是默认设置

# 或者在需要时禁用 WebSearch
adapter.configure(enable_web_search=False)
```

#### 3. 通过 Orchestrator

```python
from src.ralph_orchestrator.orchestrator import RalphOrchestrator

orchestrator = RalphOrchestrator(
    prompt_file="TASK.md",
    primary_tool="claude"
)

# Claude 适配器自动启用 WebSearch
orchestrator.run()
```

## 使用示例

### 示例 1: 研究当前主题

```python
adapter = ClaudeAdapter()
adapter.configure(enable_all_tools=True)

response = adapter.execute("""
    搜索网络上关于量子计算的最新发展,
    并总结 2024 年最重要的突破。
""")
```

### 示例 2: 技术文档研究

```python
response = adapter.execute("""
    使用 WebSearch 查找 Python 异步编程的最新最佳实践。
    比较不同的方法并提供建议。
""", enable_web_search=True)
```

### 示例 3: 实时信息

```python
response = adapter.execute("""
    搜索主要科技中心的当前天气状况:
    - 旧金山
    - 西雅图
    - 奥斯汀
    - 纽约

    同时查找主要科技公司的当前股票价格。
""", enable_all_tools=True)
```

### 示例 4: 框架研究

```python
response = adapter.execute("""
    研究 React 19 和 Next.js 15 的最新功能。
    使用 WebSearch 查找迁移指南和重大更改。
    创建一个新功能的对比表。
""")
```

## 与其他工具结合

WebSearch 与其他 Claude 工具无缝协作:

```python
response = adapter.execute("""
    1. 使用 WebSearch 查找最新的 Python Web 框架基准测试
    2. 在名为 benchmarks.md 的文件中创建对比表
    3. 在本地代码库中搜索当前的框架使用情况
    4. 根据发现提供建议
""", enable_all_tools=True)
```

## 测试 WebSearch

运行包含的测试脚本以验证 WebSearch 功能:

```bash
python test_websearch.py
```

这将测试:
- 基本 WebSearch 功能
- 使用特定工具列表的 WebSearch
- 异步 WebSearch 操作

## 最佳实践

1. **明确具体**: 提供清晰的搜索查询以获得更好的结果
2. **结合来源**: 将 WebSearch 与本地文件分析结合使用以进行全面研究
3. **验证信息**: 从多次搜索中交叉参考重要信息
4. **时效性数据**: 使用 WebSearch 获取当前事件、价格和最新发展
5. **文档**: 搜索官方文档和最新更新

## 安全考虑

启用 WebSearch 后,Claude 可以:
- 访问任何公开可用的网页内容
- 向外部站点发出 HTTP/HTTPS 请求
- 处理和分析网页内容

在生产环境中启用 WebSearch 时,请考虑您的安全要求。

## 故障排查

### WebSearch 不工作

1. 验证 Claude SDK 已安装:
   ```bash
   pip install claude-code-sdk
   ```

2. 检查 WebSearch 是否启用:
   ```python
   adapter = ClaudeAdapter(verbose=True)
   adapter.configure(enable_web_search=True, enable_all_tools=True)
   ```

3. 使用简单查询进行测试:
   ```python
   response = adapter.execute("今天的日期是什么?", enable_web_search=True)
   ```

### 速率限制

WebSearch 可能会受到速率限制。如果遇到问题:
- 在搜索之间添加延迟
- 将相关查询批量处理
- 在适当的时候使用缓存

## 高级配置

### 使用 WebSearch 的自定义工具列表

```python
# 仅启用特定工具,包括 WebSearch
adapter.configure(
    allowed_tools=['WebSearch', 'Read', 'Write', 'Edit'],
    enable_web_search=True
)
```

### 条件性 WebSearch

```python
# 仅对特定任务启用 WebSearch
if task_requires_research:
    adapter.configure(enable_web_search=True)
else:
    adapter.configure(enable_web_search=False)
```

## 与 Ralph 工作流集成

WebSearch 增强了 Ralph 在各种工作流中的能力:

1. **文档生成**: 研究和创建最新文档
2. **依赖更新**: 查找最新版本和迁移指南
3. **Bug 调查**: 搜索已知问题和解决方案
4. **最佳实践**: 研究当前行业标准
5. **API 集成**: 查找 API 文档和示例

## 性能提示

- 在适当的时候缓存搜索结果
- 将相关搜索批量处理
- 使用特定的搜索查询以更快获得结果
- 与本地工具结合进行全面分析

## 未来增强

WebSearch 集成的计划改进:
- 搜索结果缓存
- 自定义搜索提供商
- 高级过滤选项
- 搜索历史跟踪
- 离线回退选项
