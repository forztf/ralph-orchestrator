# 示例

通过实践示例学习如何使用 Ralph Orchestrator。

## 快速示例

### Hello World

最简单的 Ralph 任务:

```markdown
# PROMPT.md
编写一个打印 "Hello, World!" 的 Python 函数
将其保存到 hello.py。协调器将持续迭代直到完成。
```

运行方式:
```bash
python ralph_orchestrator.py --prompt PROMPT.md --max-iterations 5
```

### 基础数学函数

生成计算器模块:

```markdown
# PROMPT.md
创建一个 Python 计算器模块,包含:
- 加、减、乘、除函数
- 除零错误处理
- 所有函数的文档字符串
- 保存到 calculator.py

协调器将持续迭代直到完成。
```

## 完整示例

探索我们的详细示例指南:

### 📝 [简单任务](simple-task.md)
构建一个带文件持久化的命令行待办事项应用。

### 🌐 [Web API](web-api.md)
使用 Flask 创建 RESTful API,包括身份验证和数据库集成。

### 🛠️ [CLI 工具](cli-tool.md)
开发功能丰富的命令行工具,支持参数解析和配置。

### 📊 [数据分析](data-analysis.md)
处理 CSV 数据,生成统计信息和可视化。

## 示例分类

### 代码生成

**使用场景**: 自动生成样板代码、工具函数或整个模块。

```markdown
创建一个 Python 日志工具,包含:
- 彩色控制台输出
- 文件轮换
- JSON 格式选项
- 多个日志级别
```

### 测试

**使用场景**: 为现有代码生成全面的测试套件。

```markdown
为 user_auth.py 模块编写 pytest 测试:
- 测试所有公共函数
- 包含边界情况
- 模拟外部依赖
- 目标: 100% 覆盖率
```

### 文档

**使用场景**: 创建或更新项目文档。

```markdown
为这个项目生成全面的 API 文档:
- 记录所有公共类和函数
- 包含使用示例
- 创建入门指南
- 格式化为 Markdown
```

### 重构

**使用场景**: 改进代码质量和结构。

```markdown
重构 data_processor.py 文件:
- 拆分大函数(>50 行)
- 提取通用模式
- 添加类型提示
- 改进变量命名
- 保持功能不变
```

### 修复 Bug

**使用场景**: 识别和修复代码中的问题。

```markdown
调试并修复支付处理模块:
- calculate_tax() 函数返回错误的值
- 支付状态未正确更新
- 添加日志以跟踪问题
- 编写测试以防止回归
```

### 数据处理

**使用场景**: 转换和分析数据文件。

```markdown
处理 sales_data.csv:
- 清理缺失值
- 计算月度总计
- 找出前 10 个产品
- 生成汇总统计
- 将结果导出到 report.json
```

## 示例最佳实践

### 1. 明确目标

始终准确指定你想要的内容:

✅ **好的**:
```markdown
创建一个 REST API 端点,要求:
- 接受发送到 /api/users 的 POST 请求
- 验证电子邮件和密码
- 成功时返回 JWT 令牌
- 使用 SQLite 存储
```

❌ **不好的**:
```markdown
创建一个用户 API
```

### 2. 包含约束条件

指定限制和要求:

```markdown
构建一个网络爬虫,要求:
- 仅使用标准库(无需 pip 安装)
- 遵守 robots.txt
- 实现速率限制(1 请求/秒)
- 优雅地处理错误
```

### 3. 定义成功标准

使完成条件明确:

```markdown
任务在以下情况完成:
1. 所有测试通过(运行: pytest test_calculator.py)
2. 代码遵循 PEP 8(运行: flake8 calculator.py)
3. 文档完整
4. 满足所有完成标准
```

### 4. 提供上下文

包含相关信息:

```markdown
上下文: 我们正在构建一个订单处理的微服务。
现有文件: models.py, database.py

创建一个订单验证模块,要求:
- 与现有模型集成
- 根据业务规则验证
- 返回详细的错误消息
```

## 运行示例

### 基本执行

```bash
# 使用默认设置运行
python ralph_orchestrator.py --prompt examples/simple-task.md
```

### 设置成本限制

```bash
# 限制支出
python ralph_orchestrator.py \
  --prompt examples/web-api.md \
  --max-cost 5.0 \
  --max-tokens 100000
```

### 使用特定代理

```bash
# 使用 Claude 处理复杂任务
python ralph_orchestrator.py \
  --agent claude \
  --prompt examples/cli-tool.md

# 使用 Gemini 处理研究任务
python ralph_orchestrator.py \
  --agent gemini \
  --prompt examples/data-analysis.md
```

### 开发模式

```bash
# 详细输出,频繁检查点
python ralph_orchestrator.py \
  --prompt examples/simple-task.md \
  --verbose \
  --checkpoint-interval 1 \
  --max-iterations 10
```

## 示例提示词模板

### Web 应用

```markdown
# 任务: 创建 [应用名称]

## 要求
- 框架: [Flask/FastAPI/Django]
- 数据库: [SQLite/PostgreSQL/MongoDB]
- 身份验证: [JWT/Session/OAuth]

## 功能
1. [功能 1]
2. [功能 2]
3. [功能 3]

## 文件结构
```
project/
├── app.py
├── models.py
├── routes.py
└── tests/
```

## 完成标准
- 所有端点正常工作
- 测试通过
- 文档完整
- 满足所有标准
```

### 数据处理

```markdown
# 任务: 处理 [数据描述]

## 输入
- 文件: [filename.csv]
- 格式: [CSV/JSON/XML]
- 大小: [近似大小]

## 处理步骤
1. [步骤 1: 加载和验证]
2. [步骤 2: 清理和转换]
3. [步骤 3: 分析]
4. [步骤 4: 导出结果]

## 输出
- 格式: [JSON/CSV/Report]
- 包含: [指标、可视化等]

## 成功标准
- 处理期间无错误
- 输出根据架构验证
- 性能: < [X] 秒
- 满足所有标准
```

### CLI 工具

```markdown
# 任务: 构建 [工具名称] CLI

## 命令
- `tool command1` - [描述]
- `tool command2` - [描述]

## 选项
- `--option1` - [描述]
- `--option2` - [描述]

## 要求
- 使用 argparse 进行参数解析
- 支持配置文件
- 彩色输出
- 长时间操作的进度条

## 示例
```bash
tool process --input file.txt --output result.json
tool analyze --verbose
```

## 完成
- 所有命令正常工作
- 帮助文本完整
- 错误处理健壮
- 满足所有标准
```

## 从示例中学习

### 研究模式

1. **提示词结构**: 成功提示词的组织方式
2. **迭代次数**: 不同任务类型的典型迭代次数
3. **Token 使用**: 各种复杂度的成本
4. **完成时间**: 任务的预期运行时间

### 实验

1. 从提供的示例开始
2. 根据你的需求修改它们
3. 比较不同的方法
4. 分享成功的模式

## 贡献示例

有好的示例?分享它:

1. 创建一个新的示例文件
2. 记录用例
3. 包含预期结果
4. 提交拉取请求

## 示例故障排除

### 任务未完成

如果示例无限期运行:
- 检查完成标准的清晰度
- 验证代理可以修改文件
- 查看迭代日志
- 调整最大迭代次数

### 成本过高

如果示例成本很高:
- 使用更简单的提示词
- 设置 token 限制
- 选择合适的代理
- 启用上下文管理

### 结果不佳

如果输出质量低:
- 提供更多上下文
- 在提示词中包含示例
- 明确指定约束
- 使用更强大的代理

## 下一步

- 尝试[简单任务示例](simple-task.md)
- 探索 [Web API 示例](web-api.md)
- 构建 [CLI 工具](cli-tool.md)
- 分析[数据](data-analysis.md)

---

📚 继续阅读[简单任务示例](simple-task.md) →
