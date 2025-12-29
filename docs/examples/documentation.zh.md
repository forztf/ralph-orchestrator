# 文档编写示例

## 概述

本指南提供了使用 Ralph Orchestrator 进行文档任务的示例。

## 示例 1: API 文档生成

### 提示词
```python
prompt = """
为以下 Python 类生成全面的 API 文档:

class DataProcessor:
    def process_batch(self, data: List[dict], batch_size: int = 100) -> List[dict]:
        '''Process data in batches'''
        pass

    def validate_data(self, data: dict) -> bool:
        '''Validate data structure'''
        pass
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 生成的文档
```markdown
## DataProcessor API 参考

### 类: DataProcessor

用于批量处理和验证数据的实用类。

#### 方法

##### `process_batch(data: List[dict], batch_size: int = 100) -> List[dict]`

以可配置的批量大小处理字典列表。

**参数:**
- `data` (List[dict]): 要处理的输入数据
- `batch_size` (int, 可选): 每批处理的项目数。默认为 100。

**返回:**
- List[dict]: 保持相同结构的处理后数据

**示例:**
```python
processor = DataProcessor()
result = processor.process_batch([{"id": 1}, {"id": 2}], batch_size=50)
```

##### `validate_data(data: dict) -> bool`

验证数据字典的结构和内容。

**参数:**
- `data` (dict): 要验证的数据字典

**返回:**
- bool: 如果数据有效则返回 True,否则返回 False
```

## 示例 2: README 生成

### 提示词
```python
prompt = """
为名为 'quicksort-plus' 的 Python 包创建一个 README.md,该包实现了
优化的快速排序算法,具有以下功能:
- 小数组使用插入排序的混合方法
- 重复元素的三路分区
- 并行处理支持
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 生成的 README
```markdown
# QuickSort Plus

具有现代增强功能的快速排序算法优化实现。

## 功能特性

- **混合排序**: 小数组(< 10 个元素)自动切换到插入排序
- **三路分区**: 高效处理包含大量重复元素的数组
- **并行处理**: 利用多个内核处理大型数据集
- **类型支持**: 适用于任何可比较的 Python 对象

## 安装

```bash
pip install quicksort-plus
```

## 快速开始

```python
from quicksort_plus import quicksort

# 基本用法
data = [3, 1, 4, 1, 5, 9, 2, 6]
sorted_data = quicksort(data)

# 使用并行处理
large_data = list(range(1000000, 0, -1))
sorted_data = quicksort(large_data, parallel=True)
```

## 性能

| 数组大小 | 标准快速排序 | QuickSort Plus | 性能提升 |
|------------|-------------------|----------------|-------------|
| 100        | 0.012ms          | 0.008ms        | 33%         |
| 10,000     | 1.8ms            | 1.2ms          | 33%         |
| 1,000,000  | 248ms            | 142ms          | 43%         |

## 许可证

MIT 许可证 - 详见 LICENSE 文件
```

## 示例 3: 代码注释生成

### 提示词
```python
prompt = """
为此函数添加全面的文档字符串和内联注释:

def merge_sorted_arrays(arr1, arr2):
    result = []
    i = j = 0

    while i < len(arr1) and j < len(arr2):
        if arr1[i] <= arr2[j]:
            result.append(arr1[i])
            i += 1
        else:
            result.append(arr2[j])
            j += 1

    result.extend(arr1[i:])
    result.extend(arr2[j:])
    return result
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 带文档的结果
```python
def merge_sorted_arrays(arr1: list, arr2: list) -> list:
    """
    将两个已排序的数组合并为一个已排序的数组。

    此函数实现归并排序的合并步骤,将两个预排序的数组
    在 O(n+m) 时间内组合成一个已排序的数组。

    参数:
        arr1 (list): 第一个已排序的数组
        arr2 (list): 第二个已排序的数组

    返回:
        list: 包含来自两个输入的所有元素的新已排序数组

    示例:
        >>> merge_sorted_arrays([1, 3, 5], [2, 4, 6])
        [1, 2, 3, 4, 5, 6]

        >>> merge_sorted_arrays([1, 2], [3, 4, 5])
        [1, 2, 3, 4, 5]

    时间复杂度: O(n + m),其中 n 和 m 是数组的长度
    空间复杂度: O(n + m),用于结果数组
    """
    result = []
    i = j = 0  # 初始化两个数组的指针

    # 比较两个数组的元素并添加较小的元素
    while i < len(arr1) and j < len(arr2):
        if arr1[i] <= arr2[j]:
            result.append(arr1[i])
            i += 1  # 移动第一个数组的指针
        else:
            result.append(arr2[j])
            j += 1  # 移动第二个数组的指针

    # 添加 arr1 中的剩余元素(如果有)
    result.extend(arr1[i:])

    # 添加 arr2 中的剩余元素(如果有)
    result.extend(arr2[j:])

    return result
```

## 示例 4: 技术规范

### 提示词
```python
prompt = """
为速率限制系统编写技术规范文档,要求:
- 支持多种算法(令牌桶、滑动窗口)
- 使用 Redis 后端
- 提供 Flask/FastAPI 中间件
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 生成的规范
```markdown
# 速率限制系统 - 技术规范

## 1. 概述

一个灵活的速率限制系统,支持多种算法,使用 Redis 后端
和框架中间件集成。

## 2. 系统架构

### 2.1 核心组件

- **速率限制器引擎**: 算法实现层
- **存储后端**: 基于 Redis 的持久化
- **中间件层**: 框架集成
- **配置管理器**: 动态规则管理

### 2.2 支持的算法

#### 令牌桶
- 可配置的容量和补充速率
- 突发流量支持
- 每用户/IP 桶

#### 滑动窗口日志
- 精确的速率限制
- 可配置的窗口大小
- 内存高效实现

## 3. API 规范

### 3.1 核心接口

```python
class RateLimiter:
    def __init__(self, backend: RedisBackend, algorithm: Algorithm):
        pass

    def is_allowed(self, key: str, cost: int = 1) -> bool:
        pass

    def get_remaining(self, key: str) -> int:
        pass
```

### 3.2 中间件集成

```python
# Flask
@app.before_request
@rate_limit(requests=100, window=3600)
def check_rate_limit():
    pass

# FastAPI
@app.middleware("http")
async def rate_limit_middleware(request: Request, call_next):
    pass
```

## 4. Redis 模式

```
rate_limit:{algorithm}:{key} -> {
    "tokens": 100,
    "last_refill": 1234567890,
    "requests": []
}
```

## 5. 性能要求

- 延迟: 每次检查 < 5ms
- 吞吐量: 10,000 次检查/秒
- 内存: O(n),其中 n = 唯一键数

## 6. 配置

```yaml
rate_limiting:
  default_algorithm: token_bucket
  redis:
    host: localhost
    port: 6379
  rules:
    - path: /api/*
      limit: 100
      window: 3600
```
```

## 文档生成的最佳实践

### 1. 提供上下文
在提示词中包含相关的代码结构、项目类型和目标受众。

### 2. 指定格式
请求特定的文档格式(Markdown、reStructuredText 等)。

### 3. 包含示例
要求在文档中包含使用示例和代码片段。

### 4. 请求章节
指定所需的章节,如安装、配置、API 参考等。

## 相关内容

- [提示词工程指南](../guide/prompts.md)
- [Web API 示例](./web-api.md)
- [测试示例](./testing.md)
