# 快速开始指南

在 5 分钟内启动并运行 Ralph Orchestrator！

## 前置要求

开始之前，请确保您有：

- Python 3.8 或更高版本
- Git（用于检查点功能）
- 至少安装了一个 AI CLI 工具

## 步骤 1：安装 AI 代理

Ralph 适用于多个 AI 代理。至少安装一个：

=== "Claude（推荐）"

    ```bash
    npm install -g @anthropic-ai/claude-code
    # 或访问 https://claude.ai/code 获取设置说明
    ```

=== "Q Chat"

    ```bash
    pip install q-cli
    # 或按照 https://github.com/qchat/qchat 中的说明操作
    ```

=== "Gemini"

    ```bash
    npm install -g @google/gemini-cli
    # 使用您的 API 密钥配置
    ```

=== "ACP 代理"

    ```bash
    # 可以使用任何 ACP 兼容代理
    # 示例：ACP 模式的 Gemini CLI
    npm install -g @google/gemini-cli
    # 运行：ralph run -a acp --acp-agent gemini
    ```

## 步骤 2：克隆 Ralph Orchestrator

```bash
# 克隆仓库
git clone https://github.com/mikeyobrien/ralph-orchestrator.git
cd ralph-orchestrator

# 安装监控的可选依赖项
pip install psutil  # 推荐用于系统指标
```

## 步骤 3：创建您的第一个任务

创建一个包含您任务的 `PROMPT.md` 文件：

```markdown
# 任务：创建待办事项 CLI

构建一个 Python 命令行待办事项应用程序，具有：

- 添加任务
- 列出任务
- 将任务标记为完成
- 将任务保存到 JSON 文件

包含适当的错误处理和帮助命令。

编排器将继续迭代，直到满足所有要求或达到限制。
```

## 步骤 4：运行 Ralph

```bash
# 基本执行（自动检测可用代理）
python ralph_orchestrator.py --prompt PROMPT.md

# 或明确指定代理
python ralph_orchestrator.py --agent claude --prompt PROMPT.md

# 或使用 ACP 兼容代理
python ralph_orchestrator.py --agent acp --acp-agent gemini --prompt PROMPT.md
```

## 步骤 5：监控进度

Ralph 现在将：

1. 读取您的提示文件
2. 执行 AI 代理
3. 检查完成情况
4. 迭代直到完成或达到限制

您将看到如下输出：

```
2025-09-08 10:30:45 - INFO - 启动 Ralph Orchestrator v1.0.0
2025-09-08 10:30:45 - INFO - 使用代理：claude
2025-09-08 10:30:45 - INFO - 开始迭代 1/100
2025-09-08 10:30:52 - INFO - 迭代 1 完成
2025-09-08 10:30:52 - INFO - 任务未完成，继续...
```

## 接下来会发生什么？

Ralph 将继续迭代，直到满足以下条件之一：

- 🎯 所有要求似乎都已满足
- ⏱️ 达到最大迭代次数（默认：100）
- ⏰ 超过最大运行时间（默认：4 小时）
- 💰 达到令牌或成本限制
- ❌ 发生不可恢复的错误
- ✅ 在提示文件中检测到完成标记
- 🔄 循环检测触发（重复输出）

## 标记完成

当任务完成时，向您的 PROMPT.md 添加完成标记：

```markdown
## 状态

- [x] 创建带有 CLI 接口的 todo.py
- [x] 实现添加、列出、完成命令
- [x] 添加 JSON 持久化
- [x] 编写单元测试
- [x] TASK_COMPLETE
```

Ralph 将检测 `- [x] TASK_COMPLETE` 标记并立即停止编排。这允许 AI 代理发出"我完成了"的信号，而不是仅依赖于迭代限制。

## 基本配置

使用命令行选项控制 Ralph 的行为：

```bash
# 限制迭代
python ralph_orchestrator.py --prompt PROMPT.md --max-iterations 50

# 设置成本限制
python ralph_orchestrator.py --prompt PROMPT.md --max-cost 10.0

# 启用详细日志记录
python ralph_orchestrator.py --prompt PROMPT.md --verbose

# 试运行（不执行测试）
python ralph_orchestrator.py --prompt PROMPT.md --dry-run
```

## 任务示例

### 简单函数

```markdown
编写一个使用正则表达式验证电子邮件地址的 Python 函数。
包含综合单元测试。
```

### 网络爬虫

```markdown
创建一个网络爬虫：

1. 获取 HackerNews 首页
2. 提取前 10 个故事
3. 将它们保存到 JSON 文件
   使用 requests 和 BeautifulSoup。
```

### CLI 工具

```markdown
构建一个 markdown 到 HTML 转换器 CLI 工具：

- 接受输入/输出文件参数
- 支持基本 markdown 语法
- 添加 --watch 模式用于自动转换
```

## 后续步骤

现在您已经运行了第一个 Ralph 任务：

- 📖 阅读[用户指南](guide/overview.md)了解详细配置
- 🔒 了解[安全功能](advanced/security.md)
- 💰 了解[成本管理](guide/cost-management.md)
- 📊 设置[监控](advanced/monitoring.md)
- 🚀 部署到[生产环境](advanced/production-deployment.md)

## 故障排除

### 找不到代理

如果 Ralph 找不到 AI 代理：

```bash
错误：未检测到 AI 代理。请安装 claude、q、gemini 或 ACP 兼容代理。
```

**解决方案**：安装支持的代理之一（参见步骤 1）

### 权限被拒绝

如果您遇到权限错误：

```bash
chmod +x ralph_orchestrator.py
```

### 任务未完成

如果您的任务无限期运行：

- 检查您的提示是否包含明确的完成标准
- 确保代理可以修改文件并朝着完成努力
- 查看 `.agent/metrics/` 中的迭代日志

## 获取帮助

- 查看[常见问题](faq.md)
- 阅读[故障排除指南](troubleshooting.md)
- 在 [GitHub](https://github.com/mikeyobrien/ralph-orchestrator/issues) 上打开问题
- 加入[讨论](https://github.com/mikeyobrien/ralph-orchestrator/discussions)

---

🎉 **恭喜！** 您已成功运行了第一次 Ralph 编排！