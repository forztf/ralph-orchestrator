# Prompt 工程指南

有效的 prompt 工程对于 Ralph Orchestrator 任务的成功至关重要。本指南涵盖了编写能获得结果的 prompt 的最佳实践、模式和技术。

## Prompt 文件基础

### 文件格式

Ralph Orchestrator 使用 Markdown 文件作为 prompt:

```markdown
# 任务标题

## 目标
清晰描述需要完成的内容。

## 要求
- 具体要求 1
- 具体要求 2

## 成功标准
任务完成时:
- 标准 1 已满足
- 标准 2 已满足

orchestrator 将运行直到达到迭代/时间/成本限制。
```

### 文件位置

默认 prompt 文件:`PROMPT.md`

自定义位置:
```bash
python ralph_orchestrator.py --prompt path/to/task.md
```

## Prompt 结构

### 核心组件

每个 prompt 应该包含:

1. **清晰的目标**
2. **具体的要求**
3. **成功标准**
4. **完成标记**

### 模板

```markdown
# [任务名称]

## 目标
[一两句话描述目标]

## 背景
[代理需要的背景信息]

## 要求
1. [具体要求]
2. [具体要求]
3. [具体要求]

## 约束
- [限制或边界]
- [技术约束]
- [资源约束]

## 成功标准
任务完成时:
- [ ] [可衡量的结果]
- [ ] [可验证的结果]
- [ ] [具体的交付物]

## 注意事项
[额外指导或提示]

---
orchestrator 将继续迭代直到达到限制。
```

## Prompt 模式

### 1. 软件开发模式

```markdown
# 构建 Web API

## 目标
创建一个用于用户管理的 RESTful API,包含身份验证。

## 要求
1. 实现用户 CRUD 操作
2. 添加 JWT 身份验证
3. 包含输入验证
4. 编写全面的测试
5. 创建 API 文档

## 技术规格
- 框架:FastAPI
- 数据库:PostgreSQL
- 身份验证:JWT 令牌
- 测试:pytest

## 端点
- POST /auth/register
- POST /auth/login
- GET /users
- GET /users/{id}
- PUT /users/{id}
- DELETE /users/{id}

## 成功标准
- [ ] 所有端点正常工作
- [ ] 测试通过,覆盖率 >80%
- [ ] API 文档已生成
- [ ] 身份验证正常工作

orchestrator 将运行直到满足完成标准或达到限制。
```

### 2. 文档模式

```markdown
# 创建用户文档

## 目标
为应用程序编写全面的用户文档。

## 要求
1. 安装指南
2. 配置参考
3. 使用示例
4. 故障排除部分
5. FAQ

## 结构
```
docs/
├── getting-started.md
├── installation.md
├── configuration.md
├── usage/
│   ├── basic.md
│   └── advanced.md
├── troubleshooting.md
└── faq.md
```

## 风格指南
- 使用清晰简洁的语言
- 包含代码示例
- 在有帮助的地方添加截图
- 遵循 Markdown 最佳实践

## 成功标准
- [ ] 所有部分完成
- [ ] 示例已测试并正常工作
- [ ] 已审查清晰度
- [ ] 没有损坏的链接

orchestrator 将继续迭代直到达到限制。
```

### 3. 数据分析模式

```markdown
# 分析销售数据

## 目标
分析 Q4 销售数据并生成洞察报告。

## 数据源
- sales_data.csv
- customer_demographics.json
- product_catalog.xlsx

## 分析要求
1. 按月的收入趋势
2. 表现最佳的产品
3. 客户细分
4. 区域表现
5. 同比比较

## 交付物
1. Python 分析脚本
2. 带有可视化的 Jupyter notebook
3. 执行摘要(PDF)
4. 原始数据导出

## 成功标准
- [ ] 所有分析完成
- [ ] 可视化已创建
- [ ] 洞察已记录
- [ ] 代码可重现

orchestrator 将运行直到达到限制。
```

### 4. 调试模式

```markdown
# 调试应用程序问题

## 问题描述
用户报告上传大文件时应用程序崩溃。

## 症状
- 文件 >100MB 时发生崩溃
- 错误:"内存分配失败"
- 影响 30% 的用户

## 调查步骤
1. 重现问题
2. 分析内存使用
3. 审查上传处理代码
4. 检查服务器资源
5. 检查错误日志

## 所需修复
- 识别根本原因
- 实施解决方案
- 添加错误处理
- 编写回归测试
- 更新文档

## 成功标准
- [ ] 问题已重现
- [ ] 根本原因已识别
- [ ] 修复已实施
- [ ] 测试通过
- [ ] 无回归

orchestrator 将继续验证迭代直到达到限制。
```

## 最佳实践

### 1. 具体明确

❌ **不好:**
```markdown
构建一个网站
```

✅ **好:**
```markdown
使用 React 和 Node.js 构建一个响应式电子商务网站,包含:
- 带搜索的产品目录
- 购物车功能
- Stripe 支付集成
- 用户身份验证
- 订单跟踪
```

### 2. 提供背景

❌ **不好:**
```markdown
修复 bug
```

✅ **好:**
```markdown
修复图像处理模块中的内存泄漏,该问题发生在:
- 处理大于 10MB 的图像时
- 同时处理多个图像时
- ImageProcessor.process() 中的清理函数可能未释放缓冲区
```

### 3. 明确定义成功

❌ **不好:**
```markdown
让它工作得更好
```

✅ **好:**
```markdown
## 成功标准
- 95% 的请求响应时间 < 200ms
- 内存使用保持在 512MB 以下
- 所有单元测试通过
- 24 小时压力测试无错误
```

### 4. 包含示例

```markdown
## 示例输入/输出

输入:
```json
{
  "user_id": 123,
  "action": "purchase",
  "items": ["SKU-001", "SKU-002"]
}
```

预期输出:
```json
{
  "order_id": "ORD-789",
  "status": "confirmed",
  "total": 99.99,
  "estimated_delivery": "2024-01-15"
}
```
```

### 5. 指定约束

```markdown
## 约束
- 必须兼容 Python 3.8+
- 不能使用外部 API
- 必须在 5 秒内完成
- 内存使用 < 1GB
- 必须遵循 PEP 8 风格指南
```

## 迭代式 Prompt

Ralph Orchestrator 在执行过程中修改 prompt 文件。设计支持迭代的 prompt:

### 自文档化进度

```markdown
## 进度日志
<!-- 代理将更新此部分 -->
- [ ] 步骤 1:设置环境
- [ ] 步骤 2:实现核心逻辑
- [ ] 步骤 3:添加测试
- [ ] 步骤 4:文档

## 当前状态
<!-- 代理更新此项 -->
正在进行:[当前任务]
已完成:[已完成项目列表]
下一步:[计划的下一步]
```

### 检查点标记

```markdown
## 检查点
- [ ] CHECKPOINT_1:基本结构完成
- [ ] CHECKPOINT_2:核心功能正常工作
- [ ] CHECKPOINT_3:测试通过
- [ ] CHECKPOINT_4:文档完成
- [ ] 所有标准已验证
```

## 高级技术

### 1. 多阶段 Prompt

```markdown
# 阶段 1:研究
研究现有解决方案并记录发现。

<!-- 阶段 1 完成后,更新 prompt 以进行阶段 2 -->

# 阶段 2:实施
基于研究,实施解决方案。

# 阶段 3:测试
全面测试和验证。
```

### 2. 条件指令

```markdown
## 实施

如果使用 Python:
- 使用类型提示
- 遵循 PEP 8
- 使用 pytest 进行测试

如果使用 JavaScript:
- 使用 TypeScript
- 遵循 Airbnb 风格指南
- 使用 Jest 进行测试
```

### 3. 学习式 Prompt

```markdown
## 方法
1. 首先,尝试简单的解决方案
2. 如果不起作用,研究替代方案
3. 记录学到的内容
4. 实施最佳解决方案

## 记录学习
<!-- 代理在执行期间填写此项 -->
- 尝试:[方法]
- 结果:[结果]
- 学习:[洞察]
```

### 4. 错误恢复

```markdown
## 错误处理
如果遇到错误:
1. 在此文件中记录错误
2. 研究解决方案
3. 尝试替代方法
4. 用发现更新此 prompt

## 错误日志
<!-- 代理更新此项 -->
```

## Prompt 安全

### 清理

Ralph Orchestrator 自动清理以下内容的 prompt:
- 命令注入尝试
- 路径遍历攻击
- 恶意模式

### 安全模式

```markdown
## 文件操作
仅在工作在 ./workspace 目录
不要修改系统文件
更改前创建备份
```

### 大小限制

默认最大 prompt 大小:10MB

根据需要调整:
```bash
python ralph_orchestrator.py --max-prompt-size 20971520  # 20MB
```

## 测试 Prompt

### 空运行

在不执行的情况下测试 prompt:

```bash
python ralph_orchestrator.py --dry-run --prompt test.md
```

### 限制迭代

使用少量迭代进行测试:

```bash
python ralph_orchestrator.py --max-iterations 3 --prompt test.md
```

### 详细模式

调试 prompt 处理:

```bash
python ralph_orchestrator.py --verbose --prompt test.md
```

## 常见陷阱

### 1. 模糊的指令

❌ **避免:**
- "让它好一点"
- "优化所有东西"
- "修复所有问题"

✅ **改为:**
- "达到 95% 的测试覆盖率"
- "将响应时间减少到 <100ms"
- "修复 process_image() 中的内存泄漏"

### 2. 缺少完成标准

❌ **避免:**
忘记指定任务何时完成

✅ **改为:**
始终包含 orchestrator 可以努力实现的清晰完成标准

### 3. 过于复杂的 Prompt

❌ **避免:**
单个 prompt 包含 50+ 个要求

✅ **改为:**
分解为阶段或单独的任务

### 4. 没有示例

❌ **避免:**
在没有示例的情况下描述所需行为

✅ **改为:**
包含输入/输出示例和边缘情况

## Prompt 库

### 入门模板

1. [Web API 开发](../examples/web-api.md)
2. [CLI 工具创建](../examples/cli-tool.md)
3. [数据分析](../examples/data-analysis.md)
4. [文档编写](../examples/documentation.md)
5. [Bug 修复](../examples/bug-fix.md)
6. [测试套件](../examples/testing.md)

## 下一步

- 探索[成本管理](cost-management.md)以实现高效的 prompt
- 了解[检查点](checkpointing.md)以处理长任务
- 查看[代理选择](agents.md)以获得最佳结果
- 查看[示例](../examples/index.md)以获取真实世界的 prompt
