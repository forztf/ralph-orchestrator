# CLI API 参考

## 概述

CLI API 提供了 Ralph Orchestrator 的命令行接口，包括命令、参数和 shell 集成。

## 主 CLI 接口

### RalphCLI 类

```python
class RalphCLI:
    """
    Ralph Orchestrator 的主 CLI 接口。

    示例:
        cli = RalphCLI()
        cli.run(sys.argv[1:])
    """

    def __init__(self):
        """使用命令注册表初始化 CLI。"""
        self.commands = {
            'run': self.cmd_run,
            'init': self.cmd_init,
            'status': self.cmd_status,
            'clean': self.cmd_clean,
            'config': self.cmd_config,
            'agents': self.cmd_agents,
            'metrics': self.cmd_metrics,
            'checkpoint': self.cmd_checkpoint,
            'rollback': self.cmd_rollback,
            'help': self.cmd_help
        }
        self.parser = self.create_parser()

    def create_parser(self) -> argparse.ArgumentParser:
        """
        创建参数解析器。

        返回:
            ArgumentParser: 配置好的解析器
        """
        parser = argparse.ArgumentParser(
            prog='ralph',
            description='Ralph Orchestrator - AI 任务自动化',
            formatter_class=argparse.RawDescriptionHelpFormatter,
            epilog="""
示例:
  ralph run                    # 使用自动检测的 agent 运行
  ralph run -a claude          # 使用 Claude 运行
  ralph run -a acp             # 使用 ACP agent 运行
  ralph run -a acp --acp-agent gemini --acp-permission-mode auto_approve
  ralph status                 # 检查当前状态
  ralph clean                  # 清理工作区
  ralph init                   # 初始化新项目
            """
        )

        # 全局参数
        parser.add_argument(
            '--version',
            action='version',
            version='%(prog)s 1.0.0'
        )

        parser.add_argument(
            '--verbose', '-v',
            action='store_true',
            help='启用详细输出'
        )

        parser.add_argument(
            '--config', '-c',
            help='配置文件路径'
        )

        # 子命令
        subparsers = parser.add_subparsers(
            dest='command',
            help='可用命令'
        )

        # Run 命令
        run_parser = subparsers.add_parser(
            'run',
            help='运行 orchestrator'
        )
        run_parser.add_argument(
            '--agent', '-a',
            choices=['claude', 'q', 'gemini', 'acp', 'auto'],
            default='auto',
            help='要使用的 AI agent'
        )
        run_parser.add_argument(
            '--acp-agent',
            default='gemini',
            help='ACP agent 命令（用于 -a acp）'
        )
        run_parser.add_argument(
            '--acp-permission-mode',
            choices=['auto_approve', 'deny_all', 'allowlist', 'interactive'],
            default='auto_approve',
            help='ACP agent 的权限处理模式'
        )
        run_parser.add_argument(
            '--prompt', '-p',
            default='PROMPT.md',
            help='提示词文件路径'
        )
        run_parser.add_argument(
            '--max-iterations', '-i',
            type=int,
            default=100,
            help='最大迭代次数'
        )
        run_parser.add_argument(
            '--dry-run',
            action='store_true',
            help='不执行的测试模式'
        )

        # Init 命令
        subparsers.add_parser(
            'init',
            help='初始化新项目'
        )

        # Status 命令
        subparsers.add_parser(
            'status',
            help='显示当前状态'
        )

        # Clean 命令
        subparsers.add_parser(
            'clean',
            help='清理工作区'
        )

        # Config 命令
        config_parser = subparsers.add_parser(
            'config',
            help='管理配置'
        )
        config_parser.add_argument(
            'action',
            choices=['show', 'set', 'get'],
            help='配置操作'
        )
        config_parser.add_argument(
            'key',
            nargs='?',
            help='配置键'
        )
        config_parser.add_argument(
            'value',
            nargs='?',
            help='配置值'
        )

        # Agents 命令
        subparsers.add_parser(
            'agents',
            help='列出可用 agents'
        )

        # Metrics 命令
        metrics_parser = subparsers.add_parser(
            'metrics',
            help='查看指标'
        )
        metrics_parser.add_argument(
            '--format',
            choices=['text', 'json', 'csv'],
            default='text',
            help='输出格式'
        )

        # Checkpoint 命令
        checkpoint_parser = subparsers.add_parser(
            'checkpoint',
            help='创建检查点'
        )
        checkpoint_parser.add_argument(
            '--message', '-m',
            help='检查点消息'
        )

        # Rollback 命令
        rollback_parser = subparsers.add_parser(
            'rollback',
            help='回滚到检查点'
        )
        rollback_parser.add_argument(
            'checkpoint',
            nargs='?',
            help='检查点 ID 或 "last"'
        )

        return parser

    def run(self, args: List[str] = None):
        """
        使用参数运行 CLI。

        参数:
            args (list): 命令行参数

        返回:
            int: 退出代码

        示例:
            cli = RalphCLI()
            exit_code = cli.run(['run', '--agent', 'claude'])
        """
        args = self.parser.parse_args(args)

        # 设置日志
        if args.verbose:
            logging.basicConfig(level=logging.DEBUG)
        else:
            logging.basicConfig(level=logging.INFO)

        # 加载配置
        if args.config:
            config = load_config(args.config)
        else:
            config = load_config()

        # 执行命令
        if args.command:
            command = self.commands.get(args.command)
            if command:
                return command(args, config)
            else:
                print(f"未知命令: {args.command}")
                return 1
        else:
            self.parser.print_help()
            return 0
```

## 命令实现

### Run 命令

```python
def cmd_run(self, args, config):
    """
    执行 run 命令。

    参数:
        args: 解析后的参数
        config: 配置字典

    返回:
        int: 退出代码

    示例:
        cli.cmd_run(args, config)
    """
    # 使用 CLI 参数更新配置
    if args.agent:
        config['agent'] = args.agent
    if args.prompt:
        config['prompt_file'] = args.prompt
    if args.max_iterations:
        config['max_iterations'] = args.max_iterations
    if args.dry_run:
        config['dry_run'] = True

    # 创建并运行 orchestrator
    orchestrator = RalphOrchestrator(config)

    try:
        result = orchestrator.run()

        if result['success']:
            print(f"✓ 任务在 {result['iterations']} 次迭代内完成")
            return 0
        else:
            print(f"✗ 任务失败: {result.get('error', '未知错误')}")
            return 1

    except KeyboardInterrupt:
        print("\n⚠ 用户中断")
        return 130
    except Exception as e:
        print(f"✗ 错误: {str(e)}")
        return 1
```

### Init 命令

```python
def cmd_init(self, args, config):
    """
    初始化新的 Ralph 项目。

    参数:
        args: 解析后的参数
        config: 配置字典

    返回:
        int: 退出代码

    示例:
        cli.cmd_init(args, config)
    """
    print("正在初始化 Ralph Orchestrator 项目...")

    # 创建目录
    directories = ['.agent', '.agent/metrics', '.agent/prompts',
                  '.agent/checkpoints', '.agent/plans']
    for directory in directories:
        os.makedirs(directory, exist_ok=True)
        print(f"  ✓ 已创建 {directory}")

    # 创建默认 PROMPT.md
    if not os.path.exists('PROMPT.md'):
        with open('PROMPT.md', 'w') as f:
            f.write("""# 任务描述

在此处描述您的任务...

## 需求
- [ ] 需求 1
- [ ] 需求 2

## 成功标准
- 当以下条件满足时任务完成...

<!-- Ralph 将继续迭代直到达到限制 -->
""")
        print("  ✓ 已创建 PROMPT.md 模板")

    # 创建默认配置
    if not os.path.exists('ralph.json'):
        with open('ralph.json', 'w') as f:
            json.dump({
                'agent': 'auto',
                'max_iterations': 100,
                'checkpoint_interval': 5
            }, f, indent=2)
        print("  ✓ 已创建 ralph.json 配置")

    # 如果不存在则初始化 Git
    if not os.path.exists('.git'):
        subprocess.run(['git', 'init'], capture_output=True)
        print("  ✓ 已初始化 Git 仓库")

    print("\n✓ 项目初始化成功!")
    print("\n后续步骤:")
    print("  1. 编辑 PROMPT.md 填写您的任务")
    print("  2. 运行: ralph run")

    return 0
```

### Status 命令

```python
def cmd_status(self, args, config):
    """
    显示当前 Ralph 状态。

    参数:
        args: 解析后的参数
        config: 配置字典

    返回:
        int: 退出代码

    示例:
        cli.cmd_status(args, config)
    """
    print("Ralph Orchestrator 状态")
    print("=" * 40)

    # 检查提示词文件
    if os.path.exists('PROMPT.md'):
        print(f"✓ 提示词: PROMPT.md 存在")

        # 检查任务是否完成
        with open('PROMPT.md') as f:
            content = f.read()
        # 遗留的完成检查 - 不再使用
        # if 'TASK_COMPLETE' in content:
            print("✓ 状态: 已完成")
        else:
            print("⚠ 状态: 进行中")
    else:
        print("✗ 提示词: 未找到 PROMPT.md")

    # 检查状态
    state_file = '.agent/metrics/state_latest.json'
    if os.path.exists(state_file):
        with open(state_file) as f:
            state = json.load(f)

        print(f"\n最新状态:")
        print(f"  迭代次数: {state.get('iteration_count', 0)}")
        print(f"  运行时间: {state.get('runtime', 0):.1f}s")
        print(f"  Agent: {state.get('agent', 'none')}")
        print(f"  错误: {len(state.get('errors', []))}")

    # 检查可用 agents
    manager = AgentManager()
    available = manager.detect_available_agents()
    print(f"\n可用 Agents: {', '.join(available) if available else '无'}")

    # 检查 Git 状态
    result = subprocess.run(
        ['git', 'status', '--porcelain'],
        capture_output=True,
        text=True
    )
    if result.stdout:
        print(f"\n⚠ 存在未提交的更改")
    else:
        print(f"\n✓ Git: 工作目录干净")

    return 0
```

### Clean 命令

```python
def cmd_clean(self, args, config):
    """
    清理 Ralph 工作区。

    参数:
        args: 解析后的参数
        config: 配置字典

    返回:
        int: 退出代码

    示例:
        cli.cmd_clean(args, config)
    """
    print("正在清理 Ralph 工作区...")

    # 清理前确认
    response = input("这将删除所有 Ralph 数据。继续? [y/N]: ")
    if response.lower() != 'y':
        print("已取消")
        return 0

    # 清理目录
    directories = [
        '.agent/metrics',
        '.agent/prompts',
        '.agent/checkpoints',
        '.agent/logs'
    ]

    for directory in directories:
        if os.path.exists(directory):
            shutil.rmtree(directory)
            os.makedirs(directory)
            print(f"  ✓ 已清理 {directory}")

    # 重置状态
    state = StateManager()
    state.reset()
    print("  ✓ 已重置状态")

    print("\n✓ 工作区清理成功!")

    return 0
```

## Shell 集成

### Bash 补全

```bash
# ralph-completion.bash
_ralph_completion() {
    local cur prev opts
    COMPREPLY=()
    cur="${COMP_WORDS[COMP_CWORD]}"
    prev="${COMP_WORDS[COMP_CWORD-1]}"

    # 主命令
    opts="run init status clean config agents metrics checkpoint rollback help"

    case "${prev}" in
        ralph)
            COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
            return 0
            ;;
        --agent|-a)
            COMPREPLY=( $(compgen -W "claude q gemini acp auto" -- ${cur}) )
            return 0
            ;;
        --acp-agent)
            COMPREPLY=( $(compgen -c -- ${cur}) )
            return 0
            ;;
        --acp-permission-mode)
            COMPREPLY=( $(compgen -W "auto_approve deny_all allowlist interactive" -- ${cur}) )
            return 0
            ;;
        --format)
            COMPREPLY=( $(compgen -W "text json csv" -- ${cur}) )
            return 0
            ;;
        config)
            COMPREPLY=( $(compgen -W "show set get" -- ${cur}) )
            return 0
            ;;
    esac

    # 提示词文件的文件补全
    if [[ ${cur} == *.md ]]; then
        COMPREPLY=( $(compgen -f -X '!*.md' -- ${cur}) )
        return 0
    fi

    COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
}

complete -F _ralph_completion ralph
```

### ZSH 补全

```zsh
# ralph-completion.zsh
#compdef ralph

_ralph() {
    local -a commands
    commands=(
        'run:运行 orchestrator'
        'init:初始化项目'
        'status:显示状态'
        'clean:清理工作区'
        'config:管理配置'
        'agents:列出 agents'
        'metrics:查看指标'
        'checkpoint:创建检查点'
        'rollback:回滚到检查点'
        'help:显示帮助'
    )

    _arguments \
        '--version[显示版本]' \
        '--verbose[启用详细输出]' \
        '--config[配置文件]:file:_files' \
        '1:command:->command' \
        '*::arg:->args'

    case $state in
        command)
            _describe 'command' commands
            ;;
        args)
            case $words[1] in
                run)
                    _arguments \
                        '--agent[AI agent]:agent:(claude q gemini acp auto)' \
                        '--prompt[提示词文件]:file:_files -g "*.md"' \
                        '--max-iterations[最大迭代次数]:number' \
                        '--acp-agent[ACP agent 命令]:command' \
                        '--acp-permission-mode[权限模式]:mode:(auto_approve deny_all allowlist interactive)' \
                        '--dry-run[测试模式]'
                    ;;
                config)
                    _arguments \
                        '1:action:(show set get)' \
                        '2:key' \
                        '3:value'
                    ;;
            esac
            ;;
    esac
}
```

## 交互模式

```python
class InteractiveCLI:
    """
    Ralph 的交互式 CLI 模式。

    示例:
        interactive = InteractiveCLI()
        interactive.run()
    """

    def __init__(self):
        self.running = True
        self.orchestrator = None
        self.config = load_config()

    def run(self):
        """运行交互模式。"""
        print("Ralph Orchestrator 交互模式")
        print("输入 'help' 查看命令，'exit' 退出")
        print()

        while self.running:
            try:
                command = input("ralph> ").strip()
                if command:
                    self.execute_command(command)
            except KeyboardInterrupt:
                print("\n使用 'exit' 退出")
            except EOFError:
                self.running = False

    def execute_command(self, command: str):
        """执行交互式命令。"""
        parts = command.split()
        cmd = parts[0]
        args = parts[1:] if len(parts) > 1 else []

        commands = {
            'help': self.cmd_help,
            'run': self.cmd_run,
            'status': self.cmd_status,
            'stop': self.cmd_stop,
            'config': self.cmd_config,
            'agents': self.cmd_agents,
            'exit': self.cmd_exit,
            'quit': self.cmd_exit
        }

        if cmd in commands:
            commands[cmd](args)
        else:
            print(f"未知命令: {cmd}")

    def cmd_help(self, args):
        """显示帮助。"""
        print("""
可用命令:
  run [agent]    - 启动 orchestrator
  status         - 显示当前状态
  stop           - 停止 orchestrator
  config [key]   - 显示/设置配置
  agents         - 列出可用 agents
  help           - 显示此帮助
  exit           - 退出交互模式
        """)

    def cmd_exit(self, args):
        """退出交互模式。"""
        if self.orchestrator:
            print("正在停止 orchestrator...")
            # 停止 orchestrator
        print("再见!")
        self.running = False
```

## 插件系统

```python
class CLIPlugin:
    """
    CLI 插件的基类。

    示例:
        class MyPlugin(CLIPlugin):
            def register_commands(self, cli):
                cli.add_command('mycommand', self.my_command)
    """

    def __init__(self, name: str):
        self.name = name

    def register_commands(self, cli: RalphCLI):
        """向 CLI 注册插件命令。"""
        raise NotImplementedError

    def register_arguments(self, parser: argparse.ArgumentParser):
        """注册插件参数。"""
        pass

class PluginManager:
    """管理 CLI 插件。"""

    def __init__(self):
        self.plugins = []

    def load_plugin(self, plugin: CLIPlugin):
        """加载插件。"""
        self.plugins.append(plugin)

    def register_all(self, cli: RalphCLI):
        """向 CLI 注册所有插件。"""
        for plugin in self.plugins:
            plugin.register_commands(cli)
```
