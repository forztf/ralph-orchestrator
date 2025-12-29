# 使用 Ralph 构建 Web API

本示例演示如何使用 Ralph Orchestrator 构建完整的 REST API 并集成数据库。

## 任务描述

创建一个待办事项列表的 Flask REST API,要求:
- SQLite 数据库
- CRUD 操作
- 输入验证
- 错误处理
- 单元测试

## PROMPT.md 文件

```markdown
# 任务:构建待办事项列表 REST API

创建一个符合以下要求的 Flask REST API:

## API 端点

1. GET /todos - 列出所有待办事项
2. GET /todos/<id> - 获取单个待办事项
3. POST /todos - 创建新待办事项
4. PUT /todos/<id> - 更新待办事项
5. DELETE /todos/<id> - 删除待办事项

## 数据模型

Todo:
- id (整数,主键)
- title (字符串,必填,最多 200 字符)
- description (文本,可选)
- completed (布尔值,默认 false)
- created_at (日期时间)
- updated_at (日期时间)

## 要求

- 使用 Flask 和 SQLAlchemy
- SQLite 数据库
- 输入验证
- 正确的 HTTP 状态码
- JSON 响应的错误处理
- 使用 pytest 进行单元测试
- README 包含 API 文档

## 文件结构

```
todo-api/
├── app.py           # 主应用程序
├── models.py        # 数据库模型
├── schemas.py       # 验证模式
├── requirements.txt # 依赖项
├── test_api.py      # 单元测试
└── README.md        # 文档
```

<!-- orchestrator 将持续迭代直到满足所有要求 -->
```

## 运行 Ralph

```bash
# 初始化项目
ralph init

# 将提示词复制到 PROMPT.md
cp web-api-prompt.md PROMPT.md

# 使用 Claude 运行(推荐用于复杂任务)
ralph run --agent claude --max-iterations 50
```

## 预期输出

Ralph 将创建以下文件:

### app.py

```python
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime
from marshmallow import ValidationError
from schemas import TodoSchema
import os

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///todos.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

# 在 db 初始化后导入模型
from models import Todo

# 创建表
with app.app_context():
    db.create_all()

# 模式实例
todo_schema = TodoSchema()
todos_schema = TodoSchema(many=True)

@app.route('/todos', methods=['GET'])
def get_todos():
    """获取所有待办事项"""
    todos = Todo.query.all()
    return jsonify(todos_schema.dump(todos)), 200

@app.route('/todos/<int:id>', methods=['GET'])
def get_todo(id):
    """获取单个待办事项"""
    todo = Todo.query.get_or_404(id)
    return jsonify(todo_schema.dump(todo)), 200

@app.route('/todos', methods=['POST'])
def create_todo():
    """创建新待办事项"""
    try:
        data = todo_schema.load(request.json)
    except ValidationError as err:
        return jsonify({'errors': err.messages}), 400

    todo = Todo(**data)
    db.session.add(todo)
    db.session.commit()

    return jsonify(todo_schema.dump(todo)), 201

@app.route('/todos/<int:id>', methods=['PUT'])
def update_todo(id):
    """更新待办事项"""
    todo = Todo.query.get_or_404(id)

    try:
        data = todo_schema.load(request.json, partial=True)
    except ValidationError as err:
        return jsonify({'errors': err.messages}), 400

    for key, value in data.items():
        setattr(todo, key, value)

    todo.updated_at = datetime.utcnow()
    db.session.commit()

    return jsonify(todo_schema.dump(todo)), 200

@app.route('/todos/<int:id>', methods=['DELETE'])
def delete_todo(id):
    """删除待办事项"""
    todo = Todo.query.get_or_404(id)
    db.session.delete(todo)
    db.session.commit()

    return '', 204

@app.errorhandler(404)
def not_found(error):
    return jsonify({'error': 'Resource not found'}), 404

@app.errorhandler(500)
def internal_error(error):
    db.session.rollback()
    return jsonify({'error': 'Internal server error'}), 500

if __name__ == '__main__':
    app.run(debug=True)
```

### models.py

```python
from datetime import datetime
from app import db

class Todo(db.Model):
    __tablename__ = 'todos'

    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    description = db.Column(db.Text)
    completed = db.Column(db.Boolean, default=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow)

    def __repr__(self):
        return f'<Todo {self.id}: {self.title}>'
```

### test_api.py

```python
import pytest
import json
from app import app, db, Todo

@pytest.fixture
def client():
    app.config['TESTING'] = True
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'

    with app.test_client() as client:
        with app.app_context():
            db.create_all()
        yield client

def test_create_todo(client):
    response = client.post('/todos',
        json={'title': 'Test Todo', 'description': 'Test description'})
    assert response.status_code == 201
    data = json.loads(response.data)
    assert data['title'] == 'Test Todo'

def test_get_todos(client):
    # 创建测试待办事项
    client.post('/todos', json={'title': 'Test'})

    response = client.get('/todos')
    assert response.status_code == 200
    data = json.loads(response.data)
    assert len(data) == 1

def test_update_todo(client):
    # 创建待办事项
    create_response = client.post('/todos', json={'title': 'Original'})
    todo_id = json.loads(create_response.data)['id']

    # 更新待办事项
    response = client.put(f'/todos/{todo_id}',
        json={'title': 'Updated'})
    assert response.status_code == 200
    data = json.loads(response.data)
    assert data['title'] == 'Updated'

def test_delete_todo(client):
    # 创建待办事项
    create_response = client.post('/todos', json={'title': 'Delete Me'})
    todo_id = json.loads(create_response.data)['id']

    # 删除待办事项
    response = client.delete(f'/todos/{todo_id}')
    assert response.status_code == 204

    # 验证删除
    get_response = client.get(f'/todos/{todo_id}')
    assert get_response.status_code == 404
```

## 监控进度

```bash
# 查看 Ralph 的进度
ralph status

# 实时监控
watch -n 5 'ralph status'

# 查看日志
tail -f .agent/logs/ralph.log
```

## 迭代示例

### 迭代 1: 项目设置
- 创建项目结构
- 初始化 Flask 应用程序
- 设置 SQLAlchemy 配置

### 迭代 2-5: 模型实现
- 创建 Todo 模型
- 实现数据库模式
- 设置迁移

### 迭代 6-10: API 端点
- 实现 CRUD 操作
- 添加路由
- 处理 HTTP 方法

### 迭代 11-15: 验证
- 添加输入验证
- 实现错误处理
- 创建响应模式

### 迭代 16-20: 测试
- 编写单元测试
- 确保覆盖率
- 修复问题

### 最终迭代
- 创建 README
- 添加 requirements.txt
- 满足所有要求

## 成功技巧

1. **明确要求**: 具体说明 API 端点和数据模型
2. **包含示例**: 如需要,提供示例请求/响应
3. **测试要求**: 指定测试框架和覆盖率期望
4. **错误处理**: 明确要求适当的错误处理
5. **文档**: 要求在 README 中提供 API 文档

## 常见问题和解决方案

### 问题:数据库连接错误
```markdown
# 添加到提示词:
确保数据库在首次请求前正确初始化。
使用 app.app_context() 进行数据库操作。
```

### 问题:导入循环依赖
```markdown
# 添加到提示词:
通过在 db 初始化后导入模型来避免循环导入。
如需要,使用应用程序工厂模式。
```

### 问题:测试失败
```markdown
# 添加到提示词:
测试使用内存 SQLite 数据库。
确保使用 fixture 实现适当的测试隔离。
```

## 扩展示例

### 添加身份验证
```markdown
## 额外要求
- JWT 身份验证
- 用户注册和登录
- 受保护的端点
- 基于角色的访问控制
```

### 添加分页
```markdown
## 额外要求
- 对 GET /todos 端点进行分页
- 支持 page 和 per_page 参数
- 返回分页元数据
```

### 添加过滤
```markdown
## 额外要求
- 按完成状态过滤待办事项
- 按标题搜索待办事项
- 按 created_at 或 updated_at 排序
```

## 成本估算

- **迭代次数**: 约 20-30 次完整实现
- **时间**: 约 10-15 分钟
- **代理**: Claude 推荐用于复杂逻辑
- **API 调用**: 约 $0.20-0.30 (Claude 定价)

## 验证

Ralph 完成后:

```bash
# 安装依赖
pip install -r requirements.txt

# 运行测试
pytest test_api.py -v

# 启动服务器
python app.py

# 测试端点
curl http://localhost:5000/todos
curl -X POST http://localhost:5000/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "Test Todo"}'
```
