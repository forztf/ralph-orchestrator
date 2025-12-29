# Agents API 参考

## 概述

Agents API 提供了与不同 AI 代理（Claude、Gemini、Q）交互的接口以及管理代理执行的功能。

## Agent 接口

### 基础 Agent 类

```python
class Agent:
    """
    AI 代理的抽象基类。

    所有代理实现必须继承此类
    并实现必需的方法。
    """

    def __init__(self, name: str, command: str):
        """
        初始化代理。

        Args:
            name (str): 代理标识符
            command (str): 执行代理的命令
        """
        self.name = name
        self.command = command
        self.available = self.check_availability()

    def check_availability(self) -> bool:
        """
        检查代理在系统上是否可用。

        Returns:
            bool: 如果代理可用则返回 True

        Example:
            agent = ClaudeAgent()
            if agent.available:
                agent.execute(prompt)
        """
        return shutil.which(self.command) is not None

    def execute(self, prompt_file: str) -> Tuple[bool, str]:
        """
        使用提示文件执行代理。

        Args:
            prompt_file (str): 提示文件路径

        Returns:
            tuple: (success, output)

        Raises:
            AgentExecutionError: 如果执行失败
        """
        raise NotImplementedError

    def validate_prompt(self, prompt_file: str) -> bool:
        """
        在执行前验证提示文件。

        Args:
            prompt_file (str): 提示文件路径

        Returns:
            bool: 如果提示有效则返回 True
        """
        if not os.path.exists(prompt_file):
            return False

        with open(prompt_file) as f:
            content = f.read()

        return len(content) > 0 and len(content) < self.max_context
```

## Agent 实现

### Claude Agent

```python
class ClaudeAgent(Agent):
    """
    Claude AI 代理实现。

    Attributes:
        max_context (int): 最大上下文窗口（200K tokens）
        timeout (int): 执行超时时间（秒）
    """

    def __init__(self):
        super().__init__('claude', 'claude')
        self.max_context = 800000  # ~200K tokens
        self.timeout = 300

    def execute(self, prompt_file: str) -> Tuple[bool, str]:
        """
        使用提示执行 Claude。

        Args:
            prompt_file (str): 提示文件路径

        Returns:
            tuple: (success, output)

        Example:
            claude = ClaudeAgent()
            success, output = claude.execute('PROMPT.md')
        """
        if not self.available:
            return False, "Claude not available"

        try:
            result = subprocess.run(
                [self.command, prompt_file],
                capture_output=True,
                text=True,
                timeout=self.timeout,
                env=self.get_environment()
            )

            return result.returncode == 0, result.stdout

        except subprocess.TimeoutExpired:
            return False, "Claude execution timed out"
        except Exception as e:
            return False, f"Claude execution error: {str(e)}"

    def get_environment(self) -> dict:
        """获取 Claude 的环境变量。"""
        env = os.environ.copy()
        # 添加 Claude 特定的环境变量
        return env
```

### Gemini Agent

```python
class GeminiAgent(Agent):
    """
    Gemini AI 代理实现。

    Attributes:
        max_context (int): 最大上下文窗口（32K tokens）
        timeout (int): 执行超时时间（秒）
    """

    def __init__(self):
        super().__init__('gemini', 'gemini')
        self.max_context = 130000  # ~32K tokens
        self.timeout = 300

    def execute(self, prompt_file: str) -> Tuple[bool, str]:
        """
        使用提示执行 Gemini。

        Args:
            prompt_file (str): 提示文件路径

        Returns:
            tuple: (success, output)

        Example:
            gemini = GeminiAgent()
            success, output = gemini.execute('PROMPT.md')
        """
        if not self.available:
            return False, "Gemini not available"

        try:
            # Gemini 可能需要额外的参数
            cmd = [self.command, '--no-web', prompt_file]

            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=self.timeout
            )

            return result.returncode == 0, result.stdout

        except subprocess.TimeoutExpired:
            return False, "Gemini execution timed out"
        except Exception as e:
            return False, f"Gemini execution error: {str(e)}"
```

### Q Agent

```python
class QAgent(Agent):
    """
    Q Chat AI 代理实现。

    Attributes:
        max_context (int): 最大上下文窗口（8K tokens）
        timeout (int): 执行超时时间（秒）
    """

    def __init__(self):
        super().__init__('q', 'q')
        self.max_context = 32000  # ~8K tokens
        self.timeout = 300

    def execute(self, prompt_file: str) -> Tuple[bool, str]:
        """
        使用提示执行 Q。

        Args:
            prompt_file (str): 提示文件路径

        Returns:
            tuple: (success, output)

        Example:
            q = QAgent()
            success, output = q.execute('PROMPT.md')
        """
        if not self.available:
            return False, "Q not available"

        try:
            result = subprocess.run(
                [self.command, prompt_file],
                capture_output=True,
                text=True,
                timeout=self.timeout
            )

            return result.returncode == 0, result.stdout

        except subprocess.TimeoutExpired:
            return False, "Q execution timed out"
        except Exception as e:
            return False, f"Q execution error: {str(e)}"
```

## Agent 管理器

```python
class AgentManager:
    """
    管理多个 AI 代理并处理代理选择。

    Example:
        manager = AgentManager()
        agent = manager.get_agent('auto')
        success, output = agent.execute('PROMPT.md')
    """

    def __init__(self):
        """使用所有可用代理初始化代理管理器。"""
        self.agents = {
            'claude': ClaudeAgent(),
            'gemini': GeminiAgent(),
            'q': QAgent()
        }
        self.available_agents = self.detect_available_agents()

    def detect_available_agents(self) -> List[str]:
        """
        检测系统上哪些代理可用。

        Returns:
            list: 可用代理的名称

        Example:
            manager = AgentManager()
            available = manager.detect_available_agents()
            print(f"Available agents: {available}")
        """
        available = []
        for name, agent in self.agents.items():
            if agent.available:
                available.append(name)
        return available

    def get_agent(self, name: str = 'auto') -> Agent:
        """
        获取特定代理或自动选择最佳可用代理。

        Args:
            name (str): 代理名称或 'auto' 用于自动选择

        Returns:
            Agent: 选定的代理实例

        Raises:
            ValueError: 如果请求的代理不可用

        Example:
            manager = AgentManager()

            # 获取特定代理
            claude = manager.get_agent('claude')

            # 自动选择最佳可用代理
            agent = manager.get_agent('auto')
        """
        if name == 'auto':
            return self.auto_select_agent()

        if name not in self.agents:
            raise ValueError(f"Unknown agent: {name}")

        agent = self.agents[name]
        if not agent.available:
            raise ValueError(f"Agent not available: {name}")

        return agent

    def auto_select_agent(self) -> Agent:
        """
        自动选择最佳可用代理。

        优先级: claude > gemini > q

        Returns:
            Agent: 最佳可用代理

        Raises:
            RuntimeError: 如果没有可用代理
        """
        priority = ['claude', 'gemini', 'q']

        for agent_name in priority:
            if agent_name in self.available_agents:
                return self.agents[agent_name]

        raise RuntimeError("No AI agents available")

    def execute_with_fallback(self, prompt_file: str,
                             preferred_agent: str = 'auto') -> Tuple[bool, str, str]:
        """
        执行代理，如果首选代理失败则回退到其他代理。

        Args:
            prompt_file (str): 提示文件路径
            preferred_agent (str): 首选代理名称

        Returns:
            tuple: (success, output, agent_used)

        Example:
            manager = AgentManager()
            success, output, agent = manager.execute_with_fallback('PROMPT.md')
            print(f"Executed with {agent}")
        """
        # 首先尝试首选代理
        try:
            agent = self.get_agent(preferred_agent)
            success, output = agent.execute(prompt_file)
            if success:
                return True, output, agent.name
        except (ValueError, RuntimeError):
            pass

        # 尝试其他可用代理
        for agent_name in self.available_agents:
            if agent_name != preferred_agent:
                agent = self.agents[agent_name]
                success, output = agent.execute(prompt_file)
                if success:
                    return True, output, agent.name

        return False, "All agents failed", None
```

## Agent 执行工具

### 重试逻辑

```python
def execute_with_retry(agent: Agent, prompt_file: str,
                       max_retries: int = 3,
                       delay: int = 2) -> Tuple[bool, str]:
    """
    使用重试逻辑执行代理。

    Args:
        agent (Agent): 要执行的代理
        prompt_file (str): 提示文件路径
        max_retries (int): 最大重试次数
        delay (int): 重试之间的延迟（秒）

    Returns:
        tuple: (success, output)

    Example:
        agent = ClaudeAgent()
        success, output = execute_with_retry(agent, 'PROMPT.md')
    """
    for attempt in range(max_retries):
        success, output = agent.execute(prompt_file)

        if success:
            return True, output

        if attempt < max_retries - 1:
            time.sleep(delay * (2 ** attempt))  # 指数退避

    return False, f"Failed after {max_retries} attempts"
```

### 输出处理

```python
def process_agent_output(output: str) -> dict:
    """
    处理和解析代理输出。

    Args:
        output (str): 原始代理输出

    Returns:
        dict: 带有元数据的处理后输出

    Example:
        success, raw_output = agent.execute('PROMPT.md')
        processed = process_agent_output(raw_output)
    """
    processed = {
        'raw': output,
        'lines': output.splitlines(),
        'size': len(output),
        'has_error': 'error' in output.lower(),
        'has_completion': False,  # 旧版完成标记 - 不再使用
        'files_modified': extract_modified_files(output),
        'commands_run': extract_commands(output)
    }

    return processed

def extract_modified_files(output: str) -> List[str]:
    """从输出中提取修改过的文件列表。"""
    files = []
    patterns = [
        r"Created file: (.+)",
        r"Modified file: (.+)",
        r"Writing to (.+)"
    ]

    for pattern in patterns:
        matches = re.findall(pattern, output)
        files.extend(matches)

    return list(set(files))

def extract_commands(output: str) -> List[str]:
    """从输出中提取执行的命令。"""
    commands = []
    patterns = [
        r"Running: (.+)",
        r"Executing: (.+)",
        r"\$ (.+)"
    ]

    for pattern in patterns:
        matches = re.findall(pattern, output)
        commands.extend(matches)

    return commands
```

## Agent 指标

```python
class AgentMetrics:
    """
    跟踪代理性能指标。

    Example:
        metrics = AgentMetrics()
        metrics.record_execution('claude', 45.3, True)
        stats = metrics.get_stats('claude')
    """

    def __init__(self):
        self.executions = []

    def record_execution(self, agent_name: str,
                        duration: float,
                        success: bool,
                        output_size: int = 0):
        """
        记录代理执行指标。

        Args:
            agent_name (str): 代理名称
            duration (float): 执行持续时间（秒）
            success (bool): 执行是否成功
            output_size (int): 输出大小（字节）
        """
        self.executions.append({
            'agent': agent_name,
            'duration': duration,
            'success': success,
            'output_size': output_size,
            'timestamp': time.time()
        })

    def get_stats(self, agent_name: str = None) -> dict:
        """
        获取代理的统计信息。

        Args:
            agent_name (str): 特定代理或 None 表示全部

        Returns:
            dict: 代理统计信息
        """
        if agent_name:
            data = [e for e in self.executions if e['agent'] == agent_name]
        else:
            data = self.executions

        if not data:
            return {}

        durations = [e['duration'] for e in data]
        success_rate = sum(1 for e in data if e['success']) / len(data)

        return {
            'total_executions': len(data),
            'success_rate': success_rate,
            'avg_duration': sum(durations) / len(durations),
            'min_duration': min(durations),
            'max_duration': max(durations),
            'total_duration': sum(durations)
        }
```

## 自定义 Agent 实现

```python
class CustomAgent(Agent):
    """
    实现自定义 AI 代理的模板。

    Example:
        class MyAgent(CustomAgent):
            def __init__(self):
                super().__init__('myagent', 'myagent-cli')

            def execute(self, prompt_file):
                # 自定义执行逻辑
                pass
    """

    def __init__(self, name: str, command: str):
        super().__init__(name, command)
        self.configure()

    def configure(self):
        """覆盖以配置自定义代理。"""
        pass

    def pre_execute(self, prompt_file: str):
        """执行前调用的钩子。"""
        pass

    def post_execute(self, output: str):
        """执行后调用的钩子。"""
        pass

    def execute(self, prompt_file: str) -> Tuple[bool, str]:
        """使用钩子执行自定义代理。"""
        self.pre_execute(prompt_file)

        # 实现自定义执行逻辑
        success, output = self._execute_command(prompt_file)

        self.post_execute(output)

        return success, output

    def _execute_command(self, prompt_file: str) -> Tuple[bool, str]:
        """覆盖以实现自定义命令执行。"""
        raise NotImplementedError
```
