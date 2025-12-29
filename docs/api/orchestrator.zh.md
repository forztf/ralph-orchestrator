# Orchestrator API 参考文档

Ralph Orchestrator 核心模块的完整 API 文档。

## 模块：`ralph_orchestrator`

协调 AI 代理执行的主要编排模块。

## 类

### `RalphOrchestrator`

管理执行循环的主要编排器类。

```python
class RalphOrchestrator:
    def __init__(
        self,
        prompt_file_or_config = None,
        primary_tool: str = "claude",
        max_iterations: int = 100,
        max_runtime: int = 14400,
        track_costs: bool = False,
        max_cost: float = 10.0,
        checkpoint_interval: int = 5,
        archive_dir: str = "./prompts/archive",
        verbose: bool = False
    ):
        """使用配置或独立参数初始化编排器。"""
```

#### 方法

##### `run()`

```python
def run(self) -> None:
    """运行编排循环直到完成或达到限制。"""
```

##### `arun()`

```python
async def arun(self) -> None:
    """异步运行编排循环。"""
```

### `RalphConfig`

编排器的配置数据类。

```python
@dataclass
class RalphConfig:
    agent: AgentType = AgentType.AUTO
    prompt_file: str = "PROMPT.md"
    max_iterations: int = 100
    max_runtime: int = 14400
    checkpoint_interval: int = 5
    retry_delay: int = 2
    archive_prompts: bool = True
    git_checkpoint: bool = True
    verbose: bool = False
    dry_run: bool = False
    max_tokens: int = 1000000
    max_cost: float = 50.0
    context_window: int = 200000
    context_threshold: float = 0.8
    metrics_interval: int = 10
    enable_metrics: bool = True
    max_prompt_size: int = 10485760
    allow_unsafe_paths: bool = False
    agent_args: List[str] = field(default_factory=list)
    adapters: Dict[str, AdapterConfig] = field(default_factory=dict)
```

### `AgentType`

```python
class AgentType(Enum):
    CLAUDE = "claude"
    Q = "q"
    GEMINI = "gemini"
    AUTO = "auto"
```

## 函数

### `main()`

CLI 执行的入口点。

```python
def main() -> int:
    """CLI 执行的主入口点。"""
```

## 使用示例

```python
from ralph_orchestrator import RalphOrchestrator, RalphConfig

# 使用配置对象
config = RalphConfig(agent=AgentType.CLAUDE)
orchestrator = RalphOrchestrator(config)
orchestrator.run()

# 使用独立参数
orchestrator = RalphOrchestrator(
    prompt_file_or_config="PROMPT.md",
    primary_tool="claude",
    max_iterations=50
)
orchestrator.run()
```

实现 Ralph Wiggum 技术的主要编排模块。

### 类

#### `RalphOrchestrator`

管理迭代循环的主要编排器类。

```python
class RalphOrchestrator:
    """
    编排 AI 代理迭代以实现自主任务完成。

    属性：
        config (RalphConfig): 配置对象
        agent (Agent): 活跃的 AI 代理实例
        metrics (MetricsCollector): 指标跟踪
        state (OrchestratorState): 当前状态
    """
```

##### 构造函数

```python
def __init__(self, config: RalphConfig) -> None:
    """
    使用配置初始化编排器。

    参数：
        config: 包含设置的 RalphConfig 对象

    异常：
        ValueError: 如果配置无效
        RuntimeError: 如果没有可用的代理
    """
```

##### 方法

###### `run()`

```python
def run(self) -> int:
    """
    执行主编排循环。

    返回：
        int: 退出码（成功为 0，失败为非零）

    异常：
        SecurityError: 如果安全验证失败
        RuntimeError: 如果发生不可恢复的错误
    """
```

###### `iterate()`

```python
def iterate(self) -> bool:
    """
    执行单次迭代。

    返回：
        bool: 如果任务完成返回 True，否则返回 False

    异常：
        AgentError: 如果代理执行失败
        TokenLimitError: 如果超过 token 限制
        CostLimitError: 如果超过成本限制
    """
```

###### `checkpoint()`

```python
def checkpoint(self) -> None:
    """
    创建当前状态的 Git 检查点。

    异常：
        GitError: 如果 Git 操作失败
    """
```

###### `save_state()`

```python
def save_state(self) -> None:
    """
    将当前状态持久化到磁盘。

    状态包括：
    - 当前迭代编号
    - Token 使用情况
    - 成本累积
    - 时间戳
    - 代理信息
    """
```

###### `load_state()`

```python
def load_state(self) -> Optional[OrchestratorState]:
    """
    从磁盘加载先前的状态。

    返回：
        OrchestratorState，如果不存在状态则返回 None
    """
```

#### `RalphConfig`

编排器的配置数据类。

```python
@dataclass
class RalphConfig:
    """
    Ralph 编排器的配置。

    所有参数可以通过以下方式设置：
    - 命令行参数
    - 环境变量 (RALPH_*)
    - 配置文件 (.ralph.conf)
    - 默认值
    """

    # 代理配置
    agent: AgentType = AgentType.AUTO
    agent_args: List[str] = field(default_factory=list)

    # 文件路径
    prompt_file: str = "PROMPT.md"

    # 迭代限制
    max_iterations: int = 100
    max_runtime: int = 14400  # 4 小时

    # Token 和成本限制
    max_tokens: int = 1000000  # 100 万 token
    max_cost: float = 50.0  # 50 美元

    # 上下文管理
    context_window: int = 200000  # 200K token
    context_threshold: float = 0.8  # 80% 触发

    # 检查点
    checkpoint_interval: int = 5
    git_checkpoint: bool = True
    archive_prompts: bool = True

    # 重试配置
    retry_delay: int = 2
    max_retries: int = 3

    # 监控
    metrics_interval: int = 10
    enable_metrics: bool = True

    # 安全
    max_prompt_size: int = 10485760  # 10MB
    allow_unsafe_paths: bool = False

    # 输出
    verbose: bool = False
    dry_run: bool = False
```

#### `OrchestratorState`

编排器的状态跟踪。

```python
@dataclass
class OrchestratorState:
    """
    用于持久化和恢复的编排器状态。
    """

    # 迭代跟踪
    current_iteration: int = 0
    total_iterations: int = 0

    # 时间跟踪
    start_time: datetime = field(default_factory=datetime.now)
    last_iteration_time: Optional[datetime] = None
    total_runtime: float = 0.0

    # Token 跟踪
    total_input_tokens: int = 0
    total_output_tokens: int = 0

    # 成本跟踪
    total_cost: float = 0.0

    # 代理信息
    agent_type: str = ""
    agent_version: Optional[str] = None

    # 完成状态
    is_complete: bool = False
    completion_reason: Optional[str] = None
```

### 函数

#### `detect_agents()`

```python
def detect_agents() -> List[AgentType]:
    """
    检测系统上可用的 AI 代理。

    返回：
        可用的 AgentType 枚举列表

    示例：
        >>> detect_agents()
        [AgentType.CLAUDE, AgentType.GEMINI]
    """
```

#### `validate_prompt_file()`

```python
def validate_prompt_file(
    file_path: str,
    max_size: int = DEFAULT_MAX_PROMPT_SIZE
) -> None:
    """
    验证提示词文件的安全性和大小。

    参数：
        file_path: 提示词文件的路径
        max_size: 允许的最大文件大小（字节）

    异常：
        FileNotFoundError: 如果文件不存在
        SecurityError: 如果文件包含危险模式
        ValueError: 如果文件超过大小限制
    """
```

#### `sanitize_input()`

```python
def sanitize_input(text: str) -> str:
    """
    对输入文本进行安全清理。

    参数：
        text: 要清理的输入文本

    返回：
        经过安全处理的文本，可安全用于处理

    示例：
        >>> sanitize_input("rm -rf /; echo 'done'")
        "rm -rf _; echo 'done'"
    """
```

#### `calculate_cost()`

```python
def calculate_cost(
    input_tokens: int,
    output_tokens: int,
    agent_type: AgentType
) -> float:
    """
    基于 token 使用量计算成本。

    参数：
        input_tokens: 输入 token 数量
        output_tokens: 输出 token 数量
        agent_type: 使用的代理类型

    返回：
        成本（美元）

    示例：
        >>> calculate_cost(1000, 500, AgentType.CLAUDE)
        0.0105  # $0.0105
    """
```

### 异常

#### `OrchestratorError`

编排器错误的基本异常。

```python
class OrchestratorError(Exception):
    """编排器错误的基本异常。"""
    pass
```

#### `SecurityError`

```python
class SecurityError(OrchestratorError):
    """当安全验证失败时抛出。"""
    pass
```

#### `TokenLimitError`

```python
class TokenLimitError(OrchestratorError):
    """当超过 token 限制时抛出。"""
    pass
```

#### `CostLimitError`

```python
class CostLimitError(OrchestratorError):
    """当超过成本限制时抛出。"""
    pass
```

#### `AgentError`

```python
class AgentError(OrchestratorError):
    """当代理执行失败时抛出。"""
    pass
```

### 常量

```python
# 版本
VERSION = "1.0.0"

# 默认值
DEFAULT_MAX_ITERATIONS = 100
DEFAULT_MAX_RUNTIME = 14400  # 4 小时
DEFAULT_PROMPT_FILE = "PROMPT.md"
DEFAULT_CHECKPOINT_INTERVAL = 5
DEFAULT_RETRY_DELAY = 2
DEFAULT_MAX_TOKENS = 1000000  # 100 万 token
DEFAULT_MAX_COST = 50.0  # 50 美元
DEFAULT_CONTEXT_WINDOW = 200000  # 200K token
DEFAULT_CONTEXT_THRESHOLD = 0.8  # 80%
DEFAULT_METRICS_INTERVAL = 10
DEFAULT_MAX_PROMPT_SIZE = 10485760  # 10MB

# 每百万 token 的成本
TOKEN_COSTS = {
    "claude": {"input": 3.0, "output": 15.0},
    "q": {"input": 0.5, "output": 1.5},
    "gemini": {"input": 0.5, "output": 1.5}
}

# 传统完成标记（已弃用 - 编排器现在使用迭代/成本/时间限制）
# COMPLETION_MARKERS = ["TASK_COMPLETE", "TASK_DONE", "COMPLETE"]

# 安全模式
DANGEROUS_PATTERNS = [
    r"rm\s+-rf\s+/",
    r":(){ :|:& };:",
    r"dd\s+if=/dev/zero",
    r"mkfs\.",
    r"format\s+[cC]:",
]
```

## 使用示例

### 基本用法

```python
from ralph_orchestrator import RalphOrchestrator, RalphConfig

# 创建配置
config = RalphConfig(
    agent=AgentType.CLAUDE,
    prompt_file="task.md",
    max_iterations=50,
    max_cost=25.0
)

# 初始化编排器
orchestrator = RalphOrchestrator(config)

# 运行编排
exit_code = orchestrator.run()
```

### 自定义配置

```python
# 从环境变量加载并添加覆盖
config = RalphConfig()
config.max_iterations = 100
config.checkpoint_interval = 10
config.verbose = True

# 使用自定义配置初始化
orchestrator = RalphOrchestrator(config)
```

### 状态管理

```python
# 手动保存状态
orchestrator.save_state()

# 加载先前的状态
state = orchestrator.load_state()
if state:
    print(f"从迭代 {state.current_iteration} 恢复")
```

### 错误处理

```python
try:
    orchestrator = RalphOrchestrator(config)
    exit_code = orchestrator.run()
except SecurityError as e:
    print(f"安全违规：{e}")
except TokenLimitError as e:
    print(f"超过 token 限制：{e}")
except CostLimitError as e:
    print(f"超过成本限制：{e}")
except Exception as e:
    print(f"意外错误：{e}")
```

## 线程安全

编排器**不是线程安全的**。如果需要并发执行：

1. 创建独立的编排器实例
2. 使用不同的工作目录
3. 实现外部同步

## 性能考虑

- **内存使用**：约 50MB 基础内存 + 代理开销
- **磁盘 I/O**：检查点会创建 Git 提交
- **网络**：代理 API 调用可能有延迟
- **CPU**：最小开销（迭代间 <1%）

## 参见

- [配置 API](config.md)
- [代理 API](agents.md)
- [指标 API](metrics.md)
- [CLI 参考](cli.md)

---

📚 继续阅读 [配置 API](config.md) →
