# 配置 API 参考

## 概述

配置 API 提供了管理 Ralph Orchestrator 设置的方法，包括代理选择、运行时限制和行为自定义。

## 配置结构

### 默认配置

```python
DEFAULT_CONFIG = {
    'agent': 'auto',                    # 使用的 AI 代理
    'prompt_file': 'PROMPT.md',         # 任务描述文件
    'max_iterations': 100,               # 最大循环迭代次数
    'max_runtime': 14400,                # 最大运行时间（秒）（4 小时）
    'checkpoint_interval': 5,            # 每迭代 N 次创建检查点
    'retry_delay': 2,                    # 重试之间的延迟（秒）
    'retry_max': 5,                      # 最大连续错误次数
    'timeout_per_iteration': 300,        # 每次迭代的超时时间（秒）
    'verbose': False,                    # 启用详细日志
    'dry_run': False,                    # 不执行操作的测试模式
    'git_enabled': True,                 # 启用 Git 检查点
    'archive_enabled': True,             # 启用提示词存档
    'working_directory': '.',            # 工作目录路径
    'agent_directory': '.agent'          # 代理工作空间目录
}
```

## 配置加载

### 从文件加载

```python
def load_config(config_file='config.json'):
    """
    从 JSON 文件加载配置。

    Args:
        config_file (str): 配置文件路径

    Returns:
        dict: 与默认值合并的配置

    Example:
        config = load_config('production.json')
    """
    config = DEFAULT_CONFIG.copy()

    if os.path.exists(config_file):
        with open(config_file) as f:
            user_config = json.load(f)
        config.update(user_config)

    return config
```

### 从环境变量加载

```python
def load_env_config():
    """
    从环境变量加载配置。

    环境变量:
        RALPH_AGENT: 使用的代理 (claude, q, gemini, auto)
        RALPH_MAX_ITERATIONS: 最大迭代次数
        RALPH_MAX_RUNTIME: 最大运行时间（秒）
        RALPH_CHECKPOINT_INTERVAL: 检查点间隔
        RALPH_VERBOSE: 启用详细模式 (true/false)
        RALPH_DRY_RUN: 启用试运行模式 (true/false)

    Returns:
        dict: 来自环境的配置

    Example:
        os.environ['RALPH_AGENT'] = 'claude'
        config = load_env_config()
    """
    config = {}

    # 字符串值
    for key in ['AGENT', 'PROMPT_FILE', 'WORKING_DIRECTORY']:
        env_key = f'RALPH_{key}'
        if env_key in os.environ:
            config[key.lower()] = os.environ[env_key]

    # 整数值
    for key in ['MAX_ITERATIONS', 'MAX_RUNTIME', 'CHECKPOINT_INTERVAL']:
        env_key = f'RALPH_{key}'
        if env_key in os.environ:
            config[key.lower()] = int(os.environ[env_key])

    # 布尔值
    for key in ['VERBOSE', 'DRY_RUN', 'GIT_ENABLED']:
        env_key = f'RALPH_{key}'
        if env_key in os.environ:
            config[key.lower()] = os.environ[env_key].lower() == 'true'

    return config
```

### 从命令行参数加载

```python
def parse_args():
    """
    解析用于配置的命令行参数。

    Returns:
        argparse.Namespace: 解析后的参数

    Example:
        args = parse_args()
        config = vars(args)
    """
    parser = argparse.ArgumentParser(
        description='Ralph Orchestrator - AI 任务自动化'
    )

    parser.add_argument(
        '--agent',
        choices=['claude', 'q', 'gemini', 'auto'],
        default='auto',
        help='使用的 AI 代理'
    )

    parser.add_argument(
        '--prompt',
        default='PROMPT.md',
        help='提示词文件路径'
    )

    parser.add_argument(
        '--max-iterations',
        type=int,
        default=100,
        help='最大迭代次数'
    )

    parser.add_argument(
        '--max-runtime',
        type=int,
        default=14400,
        help='最大运行时间（秒）'
    )

    parser.add_argument(
        '--checkpoint-interval',
        type=int,
        default=5,
        help='每 N 次迭代创建检查点'
    )

    parser.add_argument(
        '--verbose',
        action='store_true',
        help='启用详细输出'
    )

    parser.add_argument(
        '--dry-run',
        action='store_true',
        help='不执行操作的测试模式'
    )

    return parser.parse_args()
```

## 配置验证

```python
def validate_config(config):
    """
    验证配置值。

    Args:
        config (dict): 要验证的配置

    Raises:
        ValueError: 如果配置无效

    Example:
        config = load_config()
        validate_config(config)
    """
    # 检查必需字段
    required_fields = ['agent', 'prompt_file', 'max_iterations']
    for field in required_fields:
        if field not in config:
            raise ValueError(f"Missing required field: {field}")

    # 验证代理
    valid_agents = ['claude', 'q', 'gemini', 'auto']
    if config['agent'] not in valid_agents:
        raise ValueError(f"Invalid agent: {config['agent']}")

    # 验证数值限制
    if config['max_iterations'] < 1:
        raise ValueError("max_iterations must be at least 1")

    if config['max_runtime'] < 60:
        raise ValueError("max_runtime must be at least 60 seconds")

    if config['checkpoint_interval'] < 1:
        raise ValueError("checkpoint_interval must be at least 1")

    # 验证文件路径
    if not os.path.exists(config['prompt_file']):
        raise ValueError(f"Prompt file not found: {config['prompt_file']}")

    return True
```

## 配置合并

```python
def merge_configs(*configs):
    """
    按优先级合并多个配置源。

    优先级（从高到低）：
    1. 命令行参数
    2. 环境变量
    3. 配置文件
    4. 默认值

    Args:
        *configs: 要合并的配置字典

    Returns:
        dict: 合并后的配置

    Example:
        final_config = merge_configs(
            DEFAULT_CONFIG,
            file_config,
            env_config,
            cli_config
        )
    """
    merged = {}

    for config in configs:
        if config:
            merged.update({k: v for k, v in config.items()
                          if v is not None})

    return merged
```

## 配置访问

```python
class Config:
    """
    支持点符号的配置访问器。

    Example:
        config = Config(load_config())
        print(config.agent)
        print(config.max_iterations)
    """

    def __init__(self, config_dict):
        self._config = config_dict

    def __getattr__(self, name):
        if name in self._config:
            return self._config[name]
        raise AttributeError(f"Config has no attribute '{name}'")

    def __setattr__(self, name, value):
        if name == '_config':
            super().__setattr__(name, value)
        else:
            self._config[name] = value

    def get(self, key, default=None):
        """使用默认值获取配置项。"""
        return self._config.get(key, default)

    def update(self, updates):
        """更新配置值。"""
        self._config.update(updates)

    def to_dict(self):
        """转换为字典。"""
        return self._config.copy()

    def save(self, filename):
        """将配置保存到文件。"""
        with open(filename, 'w') as f:
            json.dump(self._config, f, indent=2)
```

## 代理特定配置

### Claude 配置

```python
CLAUDE_CONFIG = {
    'command': 'claude',
    'args': [],
    'env': {},
    'timeout': 300,
    'context_limit': 200000,
    'features': {
        'code_execution': True,
        'web_search': False,
        'file_operations': True
    }
}
```

### Gemini 配置

```python
GEMINI_CONFIG = {
    'command': 'gemini',
    'args': ['--no-web'],
    'env': {},
    'timeout': 300,
    'context_limit': 32768,
    'features': {
        'code_execution': True,
        'web_search': True,
        'file_operations': True
    }
}
```

### Q 配置

```python
Q_CONFIG = {
    'command': 'q',
    'args': [],
    'env': {},
    'timeout': 300,
    'context_limit': 8192,
    'features': {
        'code_execution': True,
        'web_search': False,
        'file_operations': True
    }
}
```

## 运行时配置

### 动态更新

```python
class RuntimeConfig:
    """
    可以在执行期间更新的配置。

    Example:
        runtime_config = RuntimeConfig(initial_config)
        runtime_config.update_agent('gemini')
        runtime_config.adjust_limits(iterations=50)
    """

    def __init__(self, initial_config):
        self.config = initial_config.copy()
        self.history = [initial_config.copy()]

    def update_agent(self, agent):
        """切换到不同的代理。"""
        if agent in ['claude', 'q', 'gemini']:
            self.config['agent'] = agent
            self.history.append(self.config.copy())

    def adjust_limits(self, iterations=None, runtime=None):
        """调整运行时限制。"""
        if iterations:
            self.config['max_iterations'] = iterations
        if runtime:
            self.config['max_runtime'] = runtime
        self.history.append(self.config.copy())

    def rollback(self):
        """回滚到之前的配置。"""
        if len(self.history) > 1:
            self.history.pop()
            self.config = self.history[-1].copy()
```

## 配置模板

### 开发模板

```json
{
  "agent": "auto",
  "max_iterations": 50,
  "max_runtime": 3600,
  "checkpoint_interval": 10,
  "verbose": true,
  "dry_run": false,
  "git_enabled": true,
  "archive_enabled": true
}
```

### 生产模板

```json
{
  "agent": "claude",
  "max_iterations": 100,
  "max_runtime": 14400,
  "checkpoint_interval": 5,
  "retry_delay": 5,
  "retry_max": 3,
  "verbose": false,
  "dry_run": false,
  "git_enabled": true,
  "archive_enabled": true,
  "monitoring": {
    "enabled": true,
    "metrics_interval": 60,
    "alert_on_error": true
  }
}
```

### 测试模板

```json
{
  "agent": "auto",
  "max_iterations": 10,
  "max_runtime": 600,
  "checkpoint_interval": 1,
  "verbose": true,
  "dry_run": true,
  "git_enabled": false,
  "archive_enabled": false
}
```

## 配置示例

### 基本用法

```python
# 加载默认配置
config = Config(DEFAULT_CONFIG)

# 更新特定值
config.agent = 'claude'
config.max_iterations = 50

# 访问值
print(f"Using agent: {config.agent}")
print(f"Max iterations: {config.max_iterations}")
```

### 高级用法

```python
# 从多个来源加载
file_config = load_config('custom.json')
env_config = load_env_config()
cli_config = vars(parse_args())

# 按优先级合并
final_config = merge_configs(
    DEFAULT_CONFIG,
    file_config,
    env_config,
    cli_config
)

# 验证
validate_config(final_config)

# 创建访问器
config = Config(final_config)

# 保存以供复现
config.save('execution_config.json')
```

### 编程式配置

```python
# 以编程方式创建配置
config = Config({
    'agent': detect_best_agent(),
    'prompt_file': 'task.md',
    'max_iterations': calculate_iterations(task_complexity),
    'max_runtime': estimate_runtime(task_size),
    'checkpoint_interval': 5 if production else 10,
    'verbose': debug_mode,
    'dry_run': test_mode
})

# 使用配置
orchestrator = RalphOrchestrator(config.to_dict())
result = orchestrator.run()
```
