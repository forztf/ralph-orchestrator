# 为 Ralph Orchestrator 做贡献

感谢您对为 Ralph Orchestrator 做贡献感兴趣!本指南将帮助您开始为项目做贡献。

## 行为准则

通过参与本项目,您同意遵守我们的[行为准则](https://github.com/mikeyobrien/ralph-orchestrator/blob/main/CODE_OF_CONDUCT.md)。请在贡献前阅读它。

## 贡献方式

### 1. 报告错误

发现了错误?帮助我们修复它:

1. **检查现有问题**以避免重复
2. **创建新问题**,包括:
   - 清晰的标题和描述
   - 重现步骤
   - 预期行为与实际行为
   - 系统信息
   - 错误消息/日志

**错误报告模板:**
```markdown
## 描述
错误的简要描述

## 重现步骤
1. 运行命令: `python ralph_orchestrator.py ...`
2. 看到错误

## 预期行为
应该发生什么

## 实际行为
实际发生了什么

## 环境
- 操作系统: [例如: Ubuntu 22.04]
- Python: [例如: 3.10.5]
- Ralph 版本: [例如: 1.0.0]
- AI 代理: [例如: claude]

## 日志
```
错误消息放在这里
```
```

### 2. 建议功能

有想法?我们很乐意听取:

1. **检查现有功能请求**
2. **开启讨论**以进行重大更改
3. **创建功能请求**,包括:
   - 用例描述
   - 建议的解决方案
   - 替代方案
   - 实现考虑事项

### 3. 改进文档

文档改进总是受欢迎的:

- 修复拼写和语法错误
- 澄清令人困惑的部分
- 添加缺失的信息
- 创建新示例
- 翻译文档

### 4. 贡献代码

准备编码?按照以下步骤:

#### 设置开发环境

```bash
# Fork 并克隆仓库
git clone https://github.com/YOUR_USERNAME/ralph-orchestrator.git
cd ralph-orchestrator

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Windows 上: venv\Scripts\activate

# 安装开发依赖
pip install -e .
pip install pytest pytest-cov black ruff

# 安装 pre-commit 钩子(可选)
pip install pre-commit
pre-commit install
```

#### 开发工作流

1. **创建分支**
   ```bash
   git checkout -b feature/your-feature-name
   # 或
   git checkout -b fix/issue-number
   ```

2. **进行更改**
   - 遵循现有代码风格
   - 添加/更新测试
   - 更新文档

3. **测试您的更改**
   ```bash
   # 运行所有测试
   pytest

   # 运行特定测试
   pytest test_orchestrator.py::test_function

   # 检查覆盖率
   pytest --cov=ralph_orchestrator --cov-report=html
   ```

4. **格式化代码**
   ```bash
   # 使用 black 格式化
   black ralph_orchestrator.py

   # 使用 ruff 检查
   ruff check ralph_orchestrator.py
   ```

5. **提交更改**
   ```bash
   git add .
   git commit -m "feat: add new feature"
   # 使用约定式提交: feat, fix, docs, test, refactor, style, chore
   ```

6. **推送并创建 PR**
   ```bash
   git push origin feature/your-feature-name
   ```

## 开发指南

### 代码风格

我们遵循 PEP 8,偏好如下:

- **行长度**: 88 个字符(Black 默认)
- **引号**: 字符串使用双引号
- **导入**: 使用 `isort` 排序
- **类型提示**: 在有益处的地方使用
- **文档字符串**: Google 风格

**示例:**
```python
def calculate_cost(
    input_tokens: int,
    output_tokens: int,
    agent_type: str = "claude"
) -> float:
    """
    计算 token 使用成本。

    Args:
        input_tokens: 输入 token 数量
        output_tokens: 输出 token 数量
        agent_type: AI 代理类型

    Returns:
        成本(美元)

    Raises:
        ValueError: 如果 agent_type 未知
    """
    if agent_type not in TOKEN_COSTS:
        raise ValueError(f"Unknown agent: {agent_type}")

    rates = TOKEN_COSTS[agent_type]
    cost = (input_tokens * rates["input"] +
            output_tokens * rates["output"]) / 1_000_000
    return round(cost, 4)
```

### 测试指南

所有新功能都需要测试:

1. **单元测试**用于单个函数
2. **集成测试**用于工作流
3. **边界情况**和错误条件
4. 测试目的的**文档说明**

**测试示例:**
```python
def test_calculate_cost():
    """测试不同代理的成本计算。"""
    # 测试 Claude 定价
    cost = calculate_cost(1000, 500, "claude")
    assert cost == 0.0105

    # 测试无效代理
    with pytest.raises(ValueError):
        calculate_cost(1000, 500, "invalid")

    # 测试边界情况: 零 token
    cost = calculate_cost(0, 0, "claude")
    assert cost == 0.0
```

### 提交消息约定

我们使用[约定式提交](https://www.conventionalcommits.org/):

- `feat:` 新功能
- `fix:` 错误修复
- `docs:` 文档更改
- `test:` 测试添加/更改
- `refactor:` 代码重构
- `style:` 代码风格更改
- `chore:` 维护任务
- `perf:` 性能改进

**示例:**
```bash
feat: add Gemini agent support
fix: resolve token overflow in long prompts
docs: update installation guide for Windows
test: add integration tests for checkpointing
refactor: extract prompt validation logic
```

### 拉取请求流程

1. **标题**: 使用约定式提交格式
2. **描述**: 解释内容和原因
3. **测试**: 描述执行的测试
4. **截图**: 如果有 UI 更改则包含
5. **清单**: 完成 PR 模板

**PR 模板:**
```markdown
## 描述
更改的简要描述

## 更改类型
- [ ] 错误修复
- [ ] 新功能
- [ ] 文档更新
- [ ] 性能改进

## 测试
- [ ] 所有测试通过
- [ ] 添加了新测试
- [ ] 执行了手动测试

## 清单
- [ ] 代码遵循风格指南
- [ ] 自我审查了代码
- [ ] 更新了文档
- [ ] 无破坏性更改
```

## 项目结构

```
ralph-orchestrator/
├── ralph_orchestrator.py   # 主编排器
├── ralph                   # CLI 包装器
├── tests/                  # 测试文件
│   ├── test_orchestrator.py
│   ├── test_integration.py
│   └── test_production.py
├── docs/                   # 文档
│   ├── index.md
│   ├── guide/
│   └── api/
├── examples/               # 示例提示词
├── .agent/                 # 运行时数据
└── .github/               # GitHub 配置
```

## 测试

### 运行测试

```bash
# 所有测试
pytest

# 带覆盖率
pytest --cov=ralph_orchestrator

# 特定测试文件
pytest test_orchestrator.py

# 详细输出
pytest -v

# 首次失败时停止
pytest -x
```

### 测试类别

1. **单元测试**: 测试单个函数
2. **集成测试**: 测试组件交互
3. **端到端测试**: 测试完整工作流
4. **性能测试**: 测试资源使用
5. **安全测试**: 测试输入验证

## 文档

### 本地构建文档

```bash
# 安装 MkDocs
pip install mkdocs mkdocs-material

# 本地服务
mkdocs serve

# 构建静态站点
mkdocs build
```

### 文档标准

- 清晰、简洁的语言
- 所有功能的代码示例
- 解释"为什么"而不仅仅是"如何"
- 保持示例最新
- 包含故障排除提示

## 发布流程

1. **版本更新**: 更新代码中的版本
2. **变更日志**: 更新 CHANGELOG.md
3. **测试**: 确保所有测试通过
4. **文档**: 如需要则更新
5. **标签**: 创建版本标签
6. **发布**: 创建 GitHub 发布

## 获取帮助

### 面向贡献者

- 💬 [Discord 服务器](https://discord.gg/ralph-orchestrator)
- 📧 [邮件维护者](mailto:maintainers@ralph-orchestrator.dev)
- 🗣️ [GitHub 讨论](https://github.com/mikeyobrien/ralph-orchestrator/discussions)

### 资源

- [开发设置视频](https://youtube.com/...)
- [架构概述](advanced/architecture.md)
- [API 文档](api/orchestrator.md)
- [测试指南](testing.md)

## 认可

贡献者在以下地方获得认可:

- [贡献者列表](https://github.com/mikeyobrien/ralph-orchestrator/blob/main/CONTRIBUTORS.md)
- 发布说明
- 文档致谢

## 许可证

通过贡献,您同意您的贡献将在 MIT 许可证下许可。

---

感谢为 Ralph Orchestrator 做贡献! 🎉
