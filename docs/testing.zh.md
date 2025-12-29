# 测试指南

## 概述

本指南涵盖 Ralph Orchestrator 开发和部署的测试策略、工具和最佳实践。

## 测试套件结构

```
tests/
├── unit/                 # 单元测试
│   ├── test_orchestrator.py
│   ├── test_agents.py
│   ├── test_config.py
│   └── test_metrics.py
├── integration/          # 集成测试
│   ├── test_full_cycle.py
│   ├── test_git_operations.py
│   └── test_agent_execution.py
├── e2e/                  # 端到端测试
│   ├── test_cli.py
│   └── test_scenarios.py
├── fixtures/             # 测试夹具
│   ├── prompts/
│   └── configs/
└── conftest.py          # Pytest 配置
```

## 运行测试

### 所有测试
```bash
# 运行所有测试
pytest

# 生成覆盖率报告
pytest --cov=ralph_orchestrator --cov-report=html

# 详细输出
pytest -v

# 运行特定测试文件
pytest tests/unit/test_orchestrator.py
```

### 测试分类
```bash
# 仅运行单元测试
pytest tests/unit/

# 集成测试
pytest tests/integration/

# 端到端测试
pytest tests/e2e/

# 快速测试(排除慢速测试)
pytest -m "not slow"
```

## 单元测试

### 测试 Orchestrator

```python
import pytest
from unittest.mock import Mock, patch, MagicMock
from ralph_orchestrator import RalphOrchestrator

class TestRalphOrchestrator:
    """RalphOrchestrator 单元测试"""

    @pytest.fixture
    def orchestrator(self):
        """创建 orchestrator 实例"""
        config = {
            'agent': 'claude',
            'prompt_file': 'test.md',
            'max_iterations': 10,
            'dry_run': True
        }
        return RalphOrchestrator(config)

    def test_initialization(self, orchestrator):
        """测试 orchestrator 初始化"""
        assert orchestrator.config['agent'] == 'claude'
        assert orchestrator.config['max_iterations'] == 10
        assert orchestrator.iteration_count == 0

    @patch('subprocess.run')
    def test_execute_agent(self, mock_run, orchestrator):
        """测试 agent 执行"""
        mock_run.return_value = MagicMock(
            returncode=0,
            stdout='Task completed',
            stderr=''
        )

        success, output = orchestrator.execute_agent('claude', 'test.md')

        assert success is True
        assert output == 'Task completed'
        mock_run.assert_called_once()

    def test_check_task_complete(self, orchestrator, tmp_path):
        """测试任务完成检查"""
        # 创建不带标记的 prompt 文件
        prompt_file = tmp_path / "prompt.md"
        prompt_file.write_text("Do something")
        assert orchestrator.check_task_complete(str(prompt_file)) is False

        # 添加完成标记
        prompt_file.write_text("Do something\n<!-- Legacy marker removed -->")
        assert orchestrator.check_task_complete(str(prompt_file)) is True

    @patch('ralph_orchestrator.RalphOrchestrator.execute_agent')
    def test_iteration_limit(self, mock_execute, orchestrator):
        """测试最大迭代次数限制"""
        mock_execute.return_value = (True, "Output")
        orchestrator.config['max_iterations'] = 2

        result = orchestrator.run()

        assert result['iterations'] == 2
        assert result['success'] is False
        assert 'max_iterations' in result['stop_reason']
```

### 测试 Agents

```python
import pytest
from unittest.mock import patch, MagicMock
from agents import ClaudeAgent, GeminiAgent, AgentManager

class TestAgents:
    """AI agents 单元测试"""

    @patch('shutil.which')
    def test_claude_availability(self, mock_which):
        """测试 Claude agent 可用性检查"""
        mock_which.return_value = '/usr/bin/claude'
        agent = ClaudeAgent()
        assert agent.available is True

        mock_which.return_value = None
        agent = ClaudeAgent()
        assert agent.available is False

    @patch('subprocess.run')
    def test_agent_execution_timeout(self, mock_run):
        """测试 agent 超时处理"""
        mock_run.side_effect = subprocess.TimeoutExpired('cmd', 300)

        agent = ClaudeAgent()
        success, output = agent.execute('prompt.md')

        assert success is False
        assert 'timeout' in output.lower()

    def test_agent_manager_auto_select(self):
        """测试自动 agent 选择"""
        manager = AgentManager()

        with patch.object(manager.agents['claude'], 'available', True):
            with patch.object(manager.agents['gemini'], 'available', False):
                agent = manager.get_agent('auto')
                assert agent.name == 'claude'
```

## 集成测试

### 完整周期测试

```python
import pytest
import tempfile
import shutil
from pathlib import Path
from ralph_orchestrator import RalphOrchestrator

class TestIntegration:
    """完整 Ralph 周期集成测试"""

    @pytest.fixture
    def test_dir(self):
        """创建临时测试目录"""
        temp_dir = tempfile.mkdtemp()
        yield Path(temp_dir)
        shutil.rmtree(temp_dir)

    def test_full_execution_cycle(self, test_dir):
        """测试完整执行周期"""
        # 设置
        prompt_file = test_dir / "PROMPT.md"
        prompt_file.write_text("""
        Create a Python function that returns 'Hello'
        <!-- Legacy marker removed -->
        """)

        config = {
            'agent': 'auto',
            'prompt_file': str(prompt_file),
            'max_iterations': 5,
            'working_directory': str(test_dir),
            'dry_run': False
        }

        # 执行
        orchestrator = RalphOrchestrator(config)
        result = orchestrator.run()

        # 验证
        assert result['success'] is True
        assert result['iterations'] > 0
        assert (test_dir / '.agent').exists()

    @pytest.mark.slow
    def test_checkpoint_creation(self, test_dir):
        """测试 Git 检查点创建"""
        # 初始化 Git 仓库
        subprocess.run(['git', 'init'], cwd=test_dir)

        prompt_file = test_dir / "PROMPT.md"
        prompt_file.write_text("Test task")

        config = {
            'prompt_file': str(prompt_file),
            'checkpoint_interval': 1,
            'working_directory': str(test_dir)
        }

        orchestrator = RalphOrchestrator(config)
        orchestrator.create_checkpoint(1)

        # 检查 Git 日志
        result = subprocess.run(
            ['git', 'log', '--oneline'],
            cwd=test_dir,
            capture_output=True,
            text=True
        )

        assert 'Ralph checkpoint' in result.stdout
```

## 端到端测试

### CLI 测试

```python
import pytest
from click.testing import CliRunner
from ralph_cli import cli

class TestCLI:
    """端到端 CLI 测试"""

    @pytest.fixture
    def runner(self):
        """创建 CLI 测试运行器"""
        return CliRunner()

    def test_cli_help(self, runner):
        """测试 help 命令"""
        result = runner.invoke(cli, ['--help'])
        assert result.exit_code == 0
        assert 'Ralph Orchestrator' in result.output

    def test_cli_init(self, runner):
        """测试 init 命令"""
        with runner.isolated_filesystem():
            result = runner.invoke(cli, ['init'])
            assert result.exit_code == 0
            assert Path('PROMPT.md').exists()
            assert Path('.agent').exists()

    def test_cli_status(self, runner):
        """测试 status 命令"""
        with runner.isolated_filesystem():
            # 创建 prompt 文件
            Path('PROMPT.md').write_text('Test')

            result = runner.invoke(cli, ['status'])
            assert result.exit_code == 0
            assert 'PROMPT.md exists' in result.output

    def test_cli_run_dry(self, runner):
        """测试 dry run"""
        with runner.isolated_filesystem():
            Path('PROMPT.md').write_text('Test task')

            result = runner.invoke(cli, ['run', '--dry-run'])
            assert result.exit_code == 0
```

## 测试夹具

### Prompt 夹具

```python
# tests/fixtures/prompts.py

SIMPLE_TASK = """
Create a Python function that adds two numbers.
"""

COMPLEX_TASK = """
Build a REST API with:
- User authentication
- CRUD operations
- Database integration
- Unit tests
"""

COMPLETED_TASK = """
Create a hello world function.
<!-- Legacy marker removed -->
"""
```

### Mock Agent

```python
# tests/fixtures/mock_agent.py

class MockAgent:
    """用于测试的 mock agent"""

    def __init__(self, responses=None):
        self.responses = responses or ['Default response']
        self.call_count = 0

    def execute(self, prompt_file):
        """返回预定义的响应"""
        if self.call_count < len(self.responses):
            response = self.responses[self.call_count]
        else:
            response = self.responses[-1]

        self.call_count += 1
        return True, response
```

## 性能测试

```python
import pytest
import time
from ralph_orchestrator import RalphOrchestrator

@pytest.mark.performance
class TestPerformance:
    """性能和负载测试"""

    def test_iteration_performance(self):
        """测试迭代执行时间"""
        config = {
            'agent': 'auto',
            'max_iterations': 10,
            'dry_run': True
        }

        orchestrator = RalphOrchestrator(config)

        start_time = time.time()
        orchestrator.run()
        execution_time = time.time() - start_time

        # dry run 应该很快完成
        assert execution_time < 5.0

    def test_memory_usage(self):
        """测试内存消耗"""
        import psutil
        import os

        process = psutil.Process(os.getpid())
        initial_memory = process.memory_info().rss / 1024 / 1024  # MB

        # 运行多次迭代
        config = {'max_iterations': 100, 'dry_run': True}
        orchestrator = RalphOrchestrator(config)
        orchestrator.run()

        final_memory = process.memory_info().rss / 1024 / 1024  # MB
        memory_increase = final_memory - initial_memory

        # 内存增长应该合理
        assert memory_increase < 100  # 小于 100MB
```

## 测试覆盖率

### 覆盖率配置

```ini
# .coveragerc
[run]
source = ralph_orchestrator
omit =
    */tests/*
    */test_*.py
    */__pycache__/*

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise AssertionError
    raise NotImplementedError
    if __name__ == .__main__.:
    if TYPE_CHECKING:
```

### 覆盖率报告

```bash
# 生成 HTML 覆盖率报告
pytest --cov=ralph_orchestrator --cov-report=html

# 查看报告
open htmlcov/index.html

# 终端报告显示缺失行
pytest --cov=ralph_orchestrator --cov-report=term-missing
```

## 持续集成

### GitHub Actions

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.8, 3.9, 3.10, 3.11]

    steps:
    - uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        pip install -e .
        pip install pytest pytest-cov

    - name: Run tests
      run: |
        pytest --cov=ralph_orchestrator --cov-report=xml

    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
```

## 测试最佳实践

### 1. 测试隔离
- 每个测试应该是独立的
- 使用 fixtures 进行设置/清理
- 测试后清理资源

### 2. Mock 外部依赖
- Mock subprocess 调用
- 尽可能 mock 文件系统操作
- Mock 网络请求

### 3. 测试边界情况
- 空输入
- 无效配置
- 网络故障
- 超时场景

### 4. 使用描述性名称
```python
# 好的命名
def test_orchestrator_stops_at_max_iterations():
    pass

# 不好的命名
def test_1():
    pass
```

### 5. Arrange-Act-Assert 模式
```python
def test_example():
    # Arrange (准备)
    orchestrator = RalphOrchestrator(config)

    # Act (执行)
    result = orchestrator.run()

    # Assert (断言)
    assert result['success'] is True
```

## 调试测试

### Pytest 选项
```bash
# 显示 print 语句
pytest -s

# 第一次失败时停止
pytest -x

# 运行特定测试
pytest tests/test_file.py::TestClass::test_method

# 运行匹配模式的测试
pytest -k "test_orchestrator"

# 失败时显示局部变量
pytest -l
```

### 使用 pdb
```python
def test_debugging():
    import pdb; pdb.set_trace()
    # 调试器将在此处停止
    result = some_function()
    assert result == expected
```
