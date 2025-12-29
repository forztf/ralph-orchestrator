# 上下文管理

## 概述

有效管理上下文窗口对 Ralph Orchestrator 的成功至关重要。AI 代理的上下文窗口有限，超出限制会导致失败或性能下降。

## 上下文窗口限制

### 当前代理限制

| 代理 | 上下文窗口 | Token 限制 | 大约字符数 |
|------|-----------|-----------|-----------|
| Claude | 200K tokens | 200,000 | ~800,000 字符 |
| Gemini | 32K tokens | 32,768 | ~130,000 字符 |
| Q Chat | 8K tokens | 8,192 | ~32,000 字符 |

## 上下文组件

### 消耗上下文的内容

1. **PROMPT.md 文件** - 任务描述
2. **之前的输出** - 代理响应
3. **文件内容** - 正在修改的代码
4. **系统消息** - 给代理的指令
5. **错误消息** - 调试信息

### 上下文计算

```python
def estimate_context_usage(prompt_file, workspace_files):
    """估算总上下文使用量"""
    total_chars = 0

    # Prompt 文件
    with open(prompt_file) as f:
        total_chars += len(f.read())

    # 工作区文件
    for file in workspace_files:
        if os.path.exists(file):
            with open(file) as f:
                total_chars += len(f.read())

    # 估算 token（粗略：4 字符 = 1 token）
    estimated_tokens = total_chars / 4

    return {
        'characters': total_chars,
        'estimated_tokens': estimated_tokens,
        'percentage_used': {
            'claude': (estimated_tokens / 200000) * 100,
            'gemini': (estimated_tokens / 32768) * 100,
            'q': (estimated_tokens / 8192) * 100
        }
    }
```

## 上下文优化策略

### 1. Prompt 优化

#### 保持 Prompt 简洁
```markdown
# 糟糕 - 过于冗长
创建一个综合性的 Python 应用程序，实现一个计算器，
具有广泛的错误处理、日志记录功能、用户友好的界面，
以及支持基本算术运算，包括加法、减法、乘法和除法...

# 良好 - 简洁明了
创建一个 Python 计算器，具有：
- 基本运算：+、-、*、/
- 除零错误处理
- 简单的 CLI 界面
```

#### 使用结构化格式
```markdown
# 任务：计算器模块

## 要求：
- [ ] 基本运算（加、减、乘、除）
- [ ] 输入验证
- [ ] 单元测试

## 约束：
- Python 3.11+
- 无外部依赖
- 100% 测试覆盖率
```

### 2. 文件管理

#### 拆分大文件
```python
# 不要使用一个大文件
# calculator.py（5000 行）

# 使用模块化结构
# calculator/
#   ├── __init__.py
#   ├── operations.py（500 行）
#   ├── validators.py（300 行）
#   ├── interface.py（400 行）
#   └── utils.py（200 行）
```

#### 排除不必要的文件
```python
# .agent/config.json
{
  "exclude_patterns": [
    "*.pyc",
    "__pycache__",
    "*.log",
    "test_*.py",  # 实现期间排除
    "docs/",      # 排除文档
    ".git/"       # 永不包含 git 目录
  ]
}
```

### 3. 增量处理

#### 任务分解
```markdown
# 不要使用一个大任务
"构建一个完整的 Web 应用程序"

# 分解为阶段
阶段 1：创建项目结构
阶段 2：实现数据模型
阶段 3：添加 API 端点
阶段 4：构建前端
阶段 5：添加测试
```

#### 检查点策略
```python
def create_context_aware_checkpoint(iteration, context_usage):
    """当上下文快满时创建检查点"""
    if context_usage['percentage_used']['current_agent'] > 70:
        # 通过创建检查点重置上下文
        create_checkpoint(iteration)
        # 清除工作内存
        clear_agent_memory()
        # 总结进度
        create_progress_summary()
```

### 4. 上下文窗口滑动

#### 维护滚动上下文
```python
class ContextManager:
    def __init__(self, max_history=5):
        self.history = []
        self.max_history = max_history

    def add_iteration(self, prompt, response):
        """使用滑动窗口将迭代添加到历史记录"""
        self.history.append({
            'prompt': prompt,
            'response': response,
            'timestamp': time.time()
        })

        # 仅保留最近的历史
        if len(self.history) > self.max_history:
            self.history.pop(0)

    def get_context(self):
        """获取代理的当前上下文"""
        # 仅包含最近的迭代
        return '\n'.join([
            f"Iteration {i+1}:\n{h['response'][:500]}..."
            for i, h in enumerate(self.history[-3:])
        ])
```

## 高级技巧

### 1. 上下文压缩

```python
def compress_context(text, max_length=1000):
    """在保留关键信息的同时压缩文本"""
    if len(text) <= max_length:
        return text

    # 提取关键部分
    lines = text.split('\n')
    important_lines = []

    for line in lines:
        # 保留标题、错误和关键代码
        if any(marker in line for marker in
               ['#', 'def ', 'class ', 'ERROR', 'TODO']):
            important_lines.append(line)

    compressed = '\n'.join(important_lines)

    # 如果仍然太长，则截断并添加摘要
    if len(compressed) > max_length:
        return compressed[:max_length-20] + "\n... (truncated)"

    return compressed
```

### 2. 语义分块

```python
def chunk_by_semantics(code_file):
    """将代码分割为语义块"""
    chunks = []
    current_chunk = []

    with open(code_file) as f:
        lines = f.readlines()

    for line in lines:
        current_chunk.append(line)

        # 在逻辑边界处结束块
        if line.strip().startswith('def ') or \
           line.strip().startswith('class '):
            if len(current_chunk) > 1:
                chunks.append(''.join(current_chunk[:-1]))
                current_chunk = [line]

    # 添加剩余部分
    if current_chunk:
        chunks.append(''.join(current_chunk))

    return chunks
```

### 3. 渐进式披露

```python
class ProgressiveContext:
    """根据需要逐渐揭示上下文"""

    def __init__(self):
        self.levels = {
            'summary': 100,      # 简要摘要
            'outline': 500,      # 仅结构
            'essential': 2000,   # 关键组件
            'detailed': 10000,   # 完整细节
        }

    def get_context_at_level(self, content, level='essential'):
        """获取指定详细级别的上下文"""
        max_chars = self.levels[level]

        if level == 'summary':
            return self.create_summary(content, max_chars)
        elif level == 'outline':
            return self.extract_outline(content, max_chars)
        elif level == 'essential':
            return self.extract_essential(content, max_chars)
        else:
            return content[:max_chars]
```

## 上下文监控

### 跟踪使用情况

```python
def monitor_context_usage():
    """监控并记录上下文使用情况"""
    usage = estimate_context_usage('PROMPT.md', glob.glob('*.py'))

    # 接近限制时记录警告
    for agent, percentage in usage['percentage_used'].items():
        if percentage > 80:
            logging.warning(
                f"{agent} 的上下文使用率：{percentage:.1f}% - "
                f"考虑优化"
            )

    # 保存指标
    with open('.agent/metrics/context_usage.json', 'w') as f:
        json.dump(usage, f, indent=2)

    return usage
```

### 可视化

```python
import matplotlib.pyplot as plt

def visualize_context_usage(iterations_data):
    """绘制迭代过程中的上下文使用情况"""
    iterations = [d['iteration'] for d in iterations_data]
    usage = [d['context_percentage'] for d in iterations_data]

    plt.figure(figsize=(10, 6))
    plt.plot(iterations, usage, marker='o')
    plt.axhline(y=80, color='orange', linestyle='--', label='警告')
    plt.axhline(y=100, color='red', linestyle='--', label='限制')
    plt.xlabel('迭代')
    plt.ylabel('上下文使用率 (%)')
    plt.title('上下文窗口使用率随时间变化')
    plt.legend()
    plt.savefig('.agent/context_usage.png')
```

## 最佳实践

### 1. 从小开始
- 从最小的上下文开始
- 仅在需要时添加细节
- 删除已完成的章节

### 2. 使用引用
```markdown
# 不要包含完整代码
参见 `calculator.py` 了解实现细节

# 引用特定章节
参考 `utils.py` 的第 45-67 行了解错误处理
```

### 3. 定期总结
```python
def create_iteration_summary(iteration_num):
    """每 N 次迭代创建摘要"""
    if iteration_num % 10 == 0:
        summary = {
            'completed': [],
            'in_progress': [],
            'pending': [],
            'issues': []
        }
        # ... 填充摘要

        with open(f'.agent/summaries/summary_{iteration_num}.md', 'w') as f:
            f.write(format_summary(summary))
```

### 4. 清理工作目录
```bash
# 删除不必要的文件
rm -f *.pyc
rm -rf __pycache__
rm -f *.log

# 归档旧的迭代
tar -czf .agent/archive/iteration_1-50.tar.gz .agent/prompts/prompt_*.md
rm .agent/prompts/prompt_*.md
```

## 故障排除

### 上下文溢出症状

| 症状 | 可能原因 | 解决方案 |
|------|---------|---------|
| 代理忘记之前的指令 | 上下文窗口已满 | 创建检查点并重置 |
| 响应不完整 | 达到 token 限制 | 减小 prompt 大小 |
| 重复工作 | 上下文丢失 | 使用摘要 |
| 关于缺失信息的错误 | 上下文被截断 | 分解为更小的任务 |

### 恢复策略

```python
def recover_from_context_overflow():
    """上下文限制超出时恢复"""

    # 1. 保存当前状态
    save_state()

    # 2. 创建已完成工作的摘要
    summary = create_work_summary()

    # 3. 使用最小上下文重置
    new_prompt = f"""
    从检查点继续。之前的工作摘要：
    {summary}

    当前任务：{get_current_task()}
    """

    # 4. 使用新上下文恢复
    return new_prompt
```

## 代理特定提示

### Claude（200K 上下文）
- 可以处理大型代码库
- 包含更多上下文以获得更好的结果
- 用于复杂的多文件任务

### Gemini（32K 上下文）
- 在上下文和细节之间取得平衡
- 适用于中等规模的项目
- 优化文件包含

### Q Chat（8K 上下文）
- 积极最小化上下文
- 专注于单个文件/函数
- 用于针对性任务

## 配置

```json
{
  "context_management": {
    "max_prompt_size": 5000,
    "max_file_size": 10000,
    "max_files_included": 10,
    "compression_enabled": true,
    "sliding_window_size": 5,
    "checkpoint_on_high_usage": true,
    "usage_warning_threshold": 80,
    "usage_critical_threshold": 95
  }
}
```
