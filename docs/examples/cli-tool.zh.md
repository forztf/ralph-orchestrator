# 使用 Ralph 构建命令行工具

本示例展示如何使用 Ralph Orchestrator 创建一个包含 argparse、子命令和适当打包的命令行工具。

## 任务描述

创建一个用于文件管理的 Python CLI 工具,包含以下功能:
- 多个子命令
- 进度条
- 配置文件支持
- 错误处理
- 安装脚本

## PROMPT.md 文件

```markdown
# 任务:构建文件管理器 CLI 工具

创建一个名为 'fman' 的 Python CLI 工具,具有以下功能:

## 命令

1. **list** - 列出目录中的文件
   - 选项: --all, --size, --date
   - 显示文件大小和修改日期

2. **search** - 搜索文件
   - 选项: --name, --extension, --content
   - 支持通配符和正则表达式

3. **copy** - 复制文件/目录
   - 为大文件显示进度条
   - 选项: --recursive, --overwrite

4. **move** - 移动文件/目录
   - 覆盖前确认
   - 选项: --force

5. **delete** - 删除文件/目录
   - 需要确认
   - 选项: --force, --recursive

## 要求

- 使用 argparse 进行 CLI 解析
- 使用 click 或 rich 提供更好的用户体验
- 使用 tqdm 显示进度条
- 彩色输出
- 配置文件支持 (~/.fmanrc)
- 全面的错误处理
- 单元测试
- Setup.py 用于安装

## 项目结构

```
file-manager-cli/
├── fman/
│   ├── __init__.py
│   ├── __main__.py      # 入口点
│   ├── cli.py           # CLI 接口
│   ├── commands/        # 命令实现
│   │   ├── __init__.py
│   │   ├── list_cmd.py
│   │   ├── search_cmd.py
│   │   ├── copy_cmd.py
│   │   ├── move_cmd.py
│   │   └── delete_cmd.py
│   ├── utils.py         # 工具函数
│   └── config.py        # 配置处理
├── tests/
│   └── test_commands.py
├── setup.py
├── requirements.txt
└── README.md
```

<!-- orchestrator 将继续迭代直到满足所有要求 -->
```

## 运行 Ralph

```bash
# 初始化并运行
ralph init
cp cli-tool-prompt.md PROMPT.md
ralph run --agent claude --max-iterations 40
```

## 期望输出

### fman/cli.py

```python
#!/usr/bin/env python3
import argparse
import sys
from pathlib import Path
from rich.console import Console
from rich.table import Table

from fman.commands import (
    list_cmd, search_cmd, copy_cmd,
    move_cmd, delete_cmd
)
from fman.config import load_config

console = Console()

def create_parser():
    """创建参数解析器"""
    parser = argparse.ArgumentParser(
        prog='fman',
        description='文件管理器 CLI 工具',
        formatter_class=argparse.RawDescriptionHelpFormatter
    )

    parser.add_argument(
        '--version',
        action='version',
        version='%(prog)s 1.0.0'
    )

    parser.add_argument(
        '--config',
        help='配置文件路径',
        default='~/.fmanrc'
    )

    subparsers = parser.add_subparsers(
        dest='command',
        help='可用命令'
    )

    # List 命令
    list_parser = subparsers.add_parser(
        'list',
        help='列出目录中的文件'
    )
    list_parser.add_argument(
        'path',
        nargs='?',
        default='.',
        help='目录路径'
    )
    list_parser.add_argument(
        '--all', '-a',
        action='store_true',
        help='显示隐藏文件'
    )
    list_parser.add_argument(
        '--size', '-s',
        action='store_true',
        help='按大小排序'
    )
    list_parser.add_argument(
        '--date', '-d',
        action='store_true',
        help='按日期排序'
    )

    # Search 命令
    search_parser = subparsers.add_parser(
        'search',
        help='搜索文件'
    )
    search_parser.add_argument(
        'pattern',
        help='搜索模式'
    )
    search_parser.add_argument(
        '--path', '-p',
        default='.',
        help='搜索路径'
    )
    search_parser.add_argument(
        '--name', '-n',
        action='store_true',
        help='在文件名中搜索'
    )
    search_parser.add_argument(
        '--content', '-c',
        action='store_true',
        help='在文件内容中搜索'
    )
    search_parser.add_argument(
        '--extension', '-e',
        help='按扩展名过滤'
    )

    # Copy 命令
    copy_parser = subparsers.add_parser(
        'copy',
        help='复制文件或目录'
    )
    copy_parser.add_argument('source', help='源路径')
    copy_parser.add_argument('dest', help='目标路径')
    copy_parser.add_argument(
        '--recursive', '-r',
        action='store_true',
        help='递归复制'
    )
    copy_parser.add_argument(
        '--overwrite', '-o',
        action='store_true',
        help='覆盖现有文件'
    )

    # Move 命令
    move_parser = subparsers.add_parser(
        'move',
        help='移动文件或目录'
    )
    move_parser.add_argument('source', help='源路径')
    move_parser.add_argument('dest', help='目标路径')
    move_parser.add_argument(
        '--force', '-f',
        action='store_true',
        help='强制移动无需确认'
    )

    # Delete 命令
    delete_parser = subparsers.add_parser(
        'delete',
        help='删除文件或目录'
    )
    delete_parser.add_argument(
        'path',
        nargs='+',
        help='要删除的路径'
    )
    delete_parser.add_argument(
        '--force', '-f',
        action='store_true',
        help='强制删除无需确认'
    )
    delete_parser.add_argument(
        '--recursive', '-r',
        action='store_true',
        help='递归删除目录'
    )

    return parser

def main():
    """主入口点"""
    parser = create_parser()
    args = parser.parse_args()

    # 加载配置
    config = load_config(args.config)

    # 执行命令
    commands = {
        'list': list_cmd.execute,
        'search': search_cmd.execute,
        'copy': copy_cmd.execute,
        'move': move_cmd.execute,
        'delete': delete_cmd.execute
    }

    if args.command:
        try:
            result = commands[args.command](args, config)
            sys.exit(0 if result else 1)
        except KeyboardInterrupt:
            console.print("\n[yellow]用户中断[/yellow]")
            sys.exit(130)
        except Exception as e:
            console.print(f"[red]错误: {e}[/red]")
            sys.exit(1)
    else:
        parser.print_help()
        sys.exit(0)

if __name__ == '__main__':
    main()
```

### fman/commands/list_cmd.py

```python
import os
from pathlib import Path
from datetime import datetime
from rich.console import Console
from rich.table import Table
from rich.filesize import decimal

console = Console()

def execute(args, config):
    """执行 list 命令"""
    path = Path(args.path).expanduser().resolve()

    if not path.exists():
        console.print(f"[red]错误: 路径 '{path}' 不存在[/red]")
        return False

    if not path.is_dir():
        console.print(f"[red]错误: '{path}' 不是目录[/red]")
        return False

    # 获取文件
    if args.all:
        files = list(path.iterdir())
    else:
        files = [f for f in path.iterdir() if not f.name.startswith('.')]

    # 排序文件
    if args.size:
        files.sort(key=lambda f: f.stat().st_size if f.is_file() else 0,
                  reverse=True)
    elif args.date:
        files.sort(key=lambda f: f.stat().st_mtime, reverse=True)
    else:
        files.sort(key=lambda f: f.name.lower())

    # 创建表格
    table = Table(title=f"{path} 中的文件")
    table.add_column("名称", style="cyan")
    table.add_column("类型", style="magenta")
    table.add_column("大小", justify="right", style="green")
    table.add_column("修改时间", style="yellow")

    for file in files:
        stat = file.stat()

        # 确定类型
        if file.is_dir():
            file_type = "DIR"
            size = "-"
        elif file.is_symlink():
            file_type = "LINK"
            size = "-"
        else:
            file_type = "FILE"
            size = decimal(stat.st_size)

        # 格式化修改时间
        mtime = datetime.fromtimestamp(stat.st_mtime)
        mtime_str = mtime.strftime("%Y-%m-%d %H:%M")

        table.add_row(
            file.name,
            file_type,
            size,
            mtime_str
        )

    console.print(table)
    return True
```

### fman/commands/copy_cmd.py

```python
import shutil
from pathlib import Path
from tqdm import tqdm
from rich.console import Console

console = Console()

def copy_with_progress(src, dst):
    """带进度条复制文件"""
    src_size = src.stat().st_size

    with tqdm(total=src_size, unit='B', unit_scale=True,
              desc=f"正在复制 {src.name}") as pbar:
        def callback(copied, total):
            pbar.update(copied - pbar.n)

        # 对于大文件,使用回调
        if src_size > 1024 * 1024:  # 1MB
            with open(src, 'rb') as fsrc:
                with open(dst, 'wb') as fdst:
                    copied = 0
                    while True:
                        buf = fsrc.read(1024 * 64)  # 64KB 块
                        if not buf:
                            break
                        fdst.write(buf)
                        copied += len(buf)
                        callback(copied, src_size)
        else:
            shutil.copy2(src, dst)
            callback(src_size, src_size)

def execute(args, config):
    """执行 copy 命令"""
    source = Path(args.source).expanduser().resolve()
    dest = Path(args.dest).expanduser().resolve()

    if not source.exists():
        console.print(f"[red]错误: 源路径 '{source}' 不存在[/red]")
        return False

    # 处理目录复制
    if source.is_dir():
        if not args.recursive:
            console.print("[red]错误: 使用 -r 复制目录[/red]")
            return False

        if dest.exists() and not args.overwrite:
            response = console.input(
                f"[yellow]'{dest}' 已存在。是否覆盖? [y/N]:[/yellow] "
            )
            if response.lower() != 'y':
                console.print("[yellow]复制已取消[/yellow]")
                return False

        console.print(f"正在复制目录 '{source}' 到 '{dest}'...")
        shutil.copytree(source, dest, dirs_exist_ok=args.overwrite)
        console.print("[green]✓ 目录复制成功[/green]")

    # 处理文件复制
    else:
        if dest.is_dir():
            dest = dest / source.name

        if dest.exists() and not args.overwrite:
            response = console.input(
                f"[yellow]'{dest}' 已存在。是否覆盖? [y/N]:[/yellow] "
            )
            if response.lower() != 'y':
                console.print("[yellow]复制已取消[/yellow]")
                return False

        copy_with_progress(source, dest)
        console.print("[green]✓ 文件复制成功[/green]")

    return True
```

### setup.py

```python
from setuptools import setup, find_packages

with open("README.md", "r", encoding="utf-8") as fh:
    long_description = fh.read()

setup(
    name="fman",
    version="1.0.0",
    author="Your Name",
    author_email="your.email@example.com",
    description="一个强大的文件管理器 CLI 工具",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/yourusername/fman",
    packages=find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
        "Environment :: Console",
        "Topic :: System :: Filesystems",
    ],
    python_requires=">=3.7",
    install_requires=[
        "rich>=10.0.0",
        "tqdm>=4.60.0",
        "click>=8.0.0",
    ],
    entry_points={
        "console_scripts": [
            "fman=fman.cli:main",
        ],
    },
    include_package_data=True,
)
```

## 测试 CLI

```bash
# 以开发模式安装
pip install -e .

# 测试命令
fman list --all
fman search "*.py" --path /home/user/projects
fman copy file.txt backup.txt
fman move old.txt new.txt
fman delete temp.txt --force

# 运行测试
pytest tests/ -v
```

## CLI 开发技巧

1. **清晰的命令结构**: 提前定义所有命令和选项
2. **用户体验**: 请求彩色输出和进度条
3. **错误处理**: 指定错误应如何显示
4. **配置**: 从一开始就包含配置文件支持
5. **测试**: 为每个命令请求单元测试

## 扩展工具

### 添加压缩支持
```markdown
## 附加命令
6. **compress** - 压缩文件/目录
   - 支持 zip, tar.gz, tar.bz2
   - 选项: --format, --level
   - 显示压缩比
```

### 添加远程操作
```markdown
## 附加功能
- 通过 SSH 支持远程文件操作
- 命令: remote-list, remote-copy, remote-delete
- 使用 paramiko 进行 SSH 连接
```

## 常见模式

### 确认提示
```python
def confirm_action(message):
    """获取用户确认"""
    response = console.input(f"[yellow]{message} [y/N]:[/yellow] ")
    return response.lower() == 'y'
```

### 错误处理
```python
def safe_operation(func):
    """安全操作的装饰器"""
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except PermissionError:
            console.print("[red]权限被拒绝[/red]")
        except FileNotFoundError:
            console.print("[red]文件未找到[/red]")
        except Exception as e:
            console.print(f"[red]错误: {e}[/red]")
        return False
    return wrapper
```

## 成本估算

- **迭代次数**: 完整实现约 30-40 次
- **时间**: 约 15-20 分钟
- **代理**: Claude 或 Gemini
- **API 调用**: 约 $0.30-0.40
