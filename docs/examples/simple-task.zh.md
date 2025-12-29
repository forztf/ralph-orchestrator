# 简单任务示例:待办事项列表 CLI

本示例演示如何使用 Ralph Orchestrator 构建一个简单的命令行待办事项应用程序。

## 概述

我们将创建一个 Python CLI 应用程序,它具有以下功能:
- 管理待办事项(添加、列出、完成、删除)
- 将数据持久化到 JSON 文件
- 包含彩色输出
- 具有完善的错误处理

## 提示词

创建文件 `todo-prompt.md`:

```markdown
# 构建待办事项列表 CLI 应用程序

## 目标
创建一个具有文件持久化功能的命令行待办事项管理器。

## 需求

### 核心功能
1. 添加带有描述的新待办事项
2. 列出所有待办事项及其状态
3. 将待办事项标记为完成
4. 删除待办事项
5. 清除所有待办事项
6. 将待办事项保存到 JSON 文件

### 技术规格
- 语言:Python 3.8+
- 文件存储:todos.json
- 使用 argparse 进行 CLI 开发
- 添加彩色输出(使用 colorama 或 ANSI 代码)
- 包含适当的错误处理

### 命令
- `todo add <description>` - 添加新待办事项
- `todo list` - 显示所有待办事项
- `todo done <id>` - 标记为完成
- `todo remove <id>` - 删除待办事项
- `todo clear` - 删除所有待办事项

### 文件结构
```
todo-app/
├── todo.py          # 主 CLI 应用程序
├── todos.json       # 数据存储
├── test_todo.py     # 单元测试
└── README.md        # 文档
```

## 使用示例

```bash
$ python todo.py add "Buy groceries"
✅ Added: Buy groceries (ID: 1)

$ python todo.py add "Write documentation"
✅ Added: Write documentation (ID: 2)

$ python todo.py list
Todo List:
[ ] 1. Buy groceries
[ ] 2. Write documentation

$ python todo.py done 1
✅ Completed: Buy groceries

$ python todo.py list
Todo List:
[✓] 1. Buy groceries
[ ] 2. Write documentation

$ python todo.py remove 1
✅ Removed: Buy groceries
```

## 数据格式

todos.json:
```json
{
  "todos": [
    {
      "id": 1,
      "description": "Buy groceries",
      "completed": false,
      "created_at": "2024-01-10T10:00:00",
      "completed_at": null
    }
  ],
  "next_id": 2
}
```

## 成功标准
- [ ] 所有命令按指定方式工作
- [ ] 数据在运行之间持久化
- [ ] 彩色输出以改善用户体验
- [ ] 边缘情况的错误处理
- [ ] 测试覆盖主要功能
- [ ] 包含使用说明的 README

编排器将继续迭代,直到满足所有标准或达到限制。
```

## 运行示例

### 基本执行

```bash
python ralph_orchestrator.py --prompt todo-prompt.md
```

### 使用特定设置

```bash
# 预算控制方法
python ralph_orchestrator.py \
  --agent q \
  --prompt todo-prompt.md \
  --max-cost 2.0 \
  --max-iterations 20

# 质量优先方法
python ralph_orchestrator.py \
  --agent claude \
  --prompt todo-prompt.md \
  --max-cost 10.0 \
  --checkpoint-interval 3
```

## 预期结果

### 迭代次数

典型完成时间:5-15 次迭代

### 成本估算

- **Q Chat**:$0.50 - $1.50
- **Gemini**:$0.75 - $2.00
- **Claude**:$2.00 - $5.00

### 创建的文件

成功完成后:

```
todo-app/
├── todo.py          # 约 200 行
├── todos.json       # 初始空结构
├── test_todo.py     # 约 100 行
└── README.md        # 约 50 行
```

## 示例输出

这是生成的 `todo.py` 可能的样子:

```python
#!/usr/bin/env python3
"""
Todo List CLI Application
A simple command-line todo manager with JSON persistence.
"""

import argparse
import json
import os
from datetime import datetime
from pathlib import Path

# ANSI color codes
GREEN = '\033[92m'
YELLOW = '\033[93m'
RED = '\033[91m'
RESET = '\033[0m'
BOLD = '\033[1m'

class TodoManager:
    def __init__(self, filename='todos.json'):
        self.filename = filename
        self.todos = self.load_todos()

    def load_todos(self):
        """Load todos from JSON file."""
        if not os.path.exists(self.filename):
            return {'todos': [], 'next_id': 1}

        try:
            with open(self.filename, 'r') as f:
                return json.load(f)
        except json.JSONDecodeError:
            return {'todos': [], 'next_id': 1}

    def save_todos(self):
        """Save todos to JSON file."""
        with open(self.filename, 'w') as f:
            json.dump(self.todos, f, indent=2, default=str)

    def add_todo(self, description):
        """Add a new todo item."""
        todo = {
            'id': self.todos['next_id'],
            'description': description,
            'completed': False,
            'created_at': datetime.now().isoformat(),
            'completed_at': None
        }

        self.todos['todos'].append(todo)
        self.todos['next_id'] += 1
        self.save_todos()

        print(f"{GREEN}✅ Added: {description} (ID: {todo['id']}){RESET}")
        return todo['id']

    def list_todos(self):
        """List all todos."""
        if not self.todos['todos']:
            print(f"{YELLOW}No todos found.{RESET}")
            return

        print(f"{BOLD}Todo List:{RESET}")
        for todo in self.todos['todos']:
            status = f"{GREEN}[✓]{RESET}" if todo['completed'] else "[ ]"
            print(f"{status} {todo['id']}. {todo['description']}")

    def complete_todo(self, todo_id):
        """Mark a todo as complete."""
        for todo in self.todos['todos']:
            if todo['id'] == todo_id:
                todo['completed'] = True
                todo['completed_at'] = datetime.now().isoformat()
                self.save_todos()
                print(f"{GREEN}✅ Completed: {todo['description']}{RESET}")
                return True

        print(f"{RED}❌ Todo with ID {todo_id} not found.{RESET}")
        return False

    def remove_todo(self, todo_id):
        """Remove a todo."""
        for i, todo in enumerate(self.todos['todos']):
            if todo['id'] == todo_id:
                removed = self.todos['todos'].pop(i)
                self.save_todos()
                print(f"{GREEN}✅ Removed: {removed['description']}{RESET}")
                return True

        print(f"{RED}❌ Todo with ID {todo_id} not found.{RESET}")
        return False

    def clear_todos(self):
        """Clear all todos."""
        count = len(self.todos['todos'])
        self.todos = {'todos': [], 'next_id': 1}
        self.save_todos()
        print(f"{GREEN}✅ Cleared {count} todos.{RESET}")

def main():
    parser = argparse.ArgumentParser(description='Todo List CLI')
    subparsers = parser.add_subparsers(dest='command', help='Commands')

    # Add command
    add_parser = subparsers.add_parser('add', help='Add a new todo')
    add_parser.add_argument('description', nargs='+', help='Todo description')

    # List command
    subparsers.add_parser('list', help='List all todos')

    # Done command
    done_parser = subparsers.add_parser('done', help='Mark todo as complete')
    done_parser.add_argument('id', type=int, help='Todo ID')

    # Remove command
    remove_parser = subparsers.add_parser('remove', help='Remove a todo')
    remove_parser.add_argument('id', type=int, help='Todo ID')

    # Clear command
    subparsers.add_parser('clear', help='Clear all todos')

    args = parser.parse_args()

    if not args.command:
        parser.print_help()
        return

    manager = TodoManager()

    if args.command == 'add':
        description = ' '.join(args.description)
        manager.add_todo(description)
    elif args.command == 'list':
        manager.list_todos()
    elif args.command == 'done':
        manager.complete_todo(args.id)
    elif args.command == 'remove':
        manager.remove_todo(args.id)
    elif args.command == 'clear':
        manager.clear_todos()

if __name__ == '__main__':
    main()
```

## 变体

### 1. 增强版本

在提示词中添加这些功能:

```markdown
## 附加功能
- 优先级(高、中、低)
- 带提醒的到期日期
- 类别/标签
- 搜索功能
- 导出为 CSV/Markdown
```

### 2. Web 界面

转换为 Web 应用程序:

```markdown
## Web 版本
不是 CLI,而是创建一个 Flask Web 应用程序,具有:
- HTML 界面
- REST API 端点
- SQLite 数据库
- 基本身份验证
```

### 3. 协作版本

添加多用户支持:

```markdown
## 多用户功能
- 用户账户
- 共享待办事项列表
- 权限(查看/编辑)
- 活动日志
```

## 故障排除

### 问题:文件未创建

**解决方案**:确保代理具有写权限:

```bash
# 检查权限
ls -la

# 使用显式路径运行
python ralph_orchestrator.py --prompt ./todo-prompt.md
```

### 问题:测试失败

**解决方案**:指定测试框架:

```markdown
## 测试需求
使用 pytest 进行测试:
- 安装:pip install pytest
- 运行:pytest test_todo.py
- 覆盖率:pytest --cov=todo
```

### 问题:颜色不工作

**解决方案**:为 Windows 添加回退方案:

```markdown
## 彩色输出
- 首先尝试 colorama(跨平台)
- 回退到 ANSI 代码
- 检测终端支持
- 添加 --no-color 选项
```

## 学习要点

### 本示例教授的内容

1. **CLI 开发**:有效使用 argparse
2. **数据持久化**:JSON 文件处理
3. **错误处理**:优雅的失败模式
4. **用户体验**:彩色输出和清晰的反馈
5. **测试**:为 CLI 应用程序编写单元测试

### 关键模式

- CLI 操作的命令模式
- 数据存储的存储库模式
- 清晰的关注点分离
- 全面的错误消息

## 后续步骤

完成此示例后:

1. **扩展功能**:添加上述变体
2. **改进测试**:添加集成测试
3. **打包**:创建 setup.py 以进行分发
4. **添加 CI/CD**:GitHub Actions 工作流

## 相关示例

- [Web API 示例](web-api.md) - 构建 REST API 版本
- [CLI 工具示例](cli-tool.md) - 更高级的 CLI 模式
- [数据分析示例](data-analysis.md) - 处理待办事项统计信息

---

📚 继续阅读 [Web API 示例](web-api.md) →
