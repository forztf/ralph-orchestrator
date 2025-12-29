# 常见问题解答

## 通用问题

### 什么是 Ralph Orchestrator?

Ralph Orchestrator 是 Ralph Wiggum 技术的实现 - 一种简单但有效的使用 AI 代理自主完成任务的模式。它持续针对提示文件运行 AI 代理,直到任务被标记为完成或达到限制为止。

### 为什么叫 "Ralph Wiggum"?

该技术以《辛普森一家》中的角色 Ralph Wiggum 命名,他的名言 "Me fail English? That's unpossible!" (我英语不及格?这不可能!) 体现了在不可预测的世界中确定性失败的哲学。系统会不断尝试直到成功,拥抱"不可能"。

### 谁创建了 Ralph Orchestrator?

Ralph Wiggum 技术由 [Geoffrey Huntley](https://ghuntley.com/ralph/) 创建。此实现基于他的概念,并添加了多代理支持、检查点和全面测试等额外功能。

### 它支持哪些 AI 代理?

Ralph Orchestrator 目前支持:

- **Claude** (Anthropic Claude Code CLI)
- **Gemini** (Google Gemini CLI)
- **Q Chat** (Q CLI 工具)

系统会自动检测可用的代理,并可以自动选择最佳的一个。

## 安装和设置

### 我需要安装所有三个 AI 代理吗?

不需要,您只需要至少安装一个 AI 代理。Ralph 会自动检测哪些代理可用并相应地使用它们。

### 如何安装 AI 代理?

```bash
# Claude
npm install -g @anthropic-ai/claude-code

# Gemini
npm install -g @google/gemini-cli

# Q Chat
# 按照 https://github.com/qchat/qchat 上的说明操作
```

### 系统要求是什么?

- **操作系统**: Linux、macOS 或 Windows (带 WSL)
- **Python**: 3.9 或更高版本
- **Git**: 2.25 或更高版本
- **内存**: 最少 4GB,推荐 8GB
- **存储**: 20GB 可用空间

### 我可以在 Docker 中运行 Ralph 吗?

可以!提供了 Dockerfile:

```bash
docker build -t ralph-orchestrator .
docker run -v $(pwd):/workspace ralph-orchestrator
```

## 使用问题

### 我如何知道 Ralph 何时完成?

Ralph 在以下情况下停止:

1. 达到最大迭代次数(默认: 100)
2. 超过最大运行时间(默认: 4 小时)
3. 达到成本限制(默认: $50)
4. 出现太多连续错误
5. 检测到完成标记
6. 触发循环检测(重复输出)

### 如何发出任务完成信号?

在您的 PROMPT.md 中添加复选框标记:

```markdown
- [x] TASK_COMPLETE
```

Ralph 将检测此标记并立即停止编排。这允许 AI 代理发出"我完成了"的信号,而不是仅仅依赖迭代限制。

**重要提示**: 标记必须是复选框格式(`- [x]` 或 `[x]`),而不是纯文本。

### 什么会触发循环检测?

当当前代理输出与最近 5 个输出中的任何一个相似度 ≥90% 时(使用模糊字符串匹配),会触发循环检测。这可以防止代理反复产生基本相同响应的无限循环。

常见触发原因:

- 代理卡在同一个任务上
- 在类似方法之间摆动
- 一致的 API 错误消息
- 占位符"仍在工作"响应

触发时,您会看到: `WARNING - Loop detected: 92.3% similarity to previous output`

### 我可以禁用循环检测吗?

循环检测无法直接禁用,但它只在高度相似的输出上触发(≥90% 阈值)。为避免误报:

1. 确保代理输出包含特定于迭代的详细信息
2. 添加每次迭代都会变化的进度指示器
3. 检查代理是否卡在同一个子任务上
4. 完善您的提示以鼓励不同的响应

有关详细文档,请参阅[循环检测](advanced/loop-detection.md)。

### 我应该在 PROMPT.md 中放入什么?

编写清晰、具体的要求和可衡量的成功标准。包括:

- 任务描述
- 要求列表
- 成功标准
- 示例输入/输出(如适用)
- 文件结构(针对复杂项目)

### 通常需要多少次迭代?

这因任务复杂性而异:

- 简单函数: 5-10 次迭代
- Web API: 20-30 次迭代
- 复杂应用程序: 50-100 次迭代

### 如果 Ralph 停止,我可以恢复吗?

可以!Ralph 会保存状态并可以从中断的地方恢复:

```bash
# Ralph 将自动从上次的状态恢复
ralph run
```

### 如何监控进度?

```bash
# 检查状态
ralph status

# 实时监视
watch -n 5 'ralph status'

# 查看日志
tail -f .agent/logs/ralph.log
```

## 配置

### 如何更改默认代理?

编辑 `ralph.json`:

```json
{
  "agent": "claude" // 或 "gemini", "q", "auto"
}
```

或使用命令行:

```bash
ralph run --agent claude
```

### 我可以设置自定义迭代限制吗?

可以,有多种方式:

```bash
# 命令行
ralph run --max-iterations 50

# 配置文件 (ralph.json)
{
  "max_iterations": 50
}

# 环境变量
export RALPH_MAX_ITERATIONS=50
```

### 什么是检查点间隔?

检查点间隔决定 Ralph 创建 Git 提交以保存进度的频率。默认为每 5 次迭代一次。

### 如何禁用 Git 操作?

```bash
ralph run --no-git
```

或在配置中:

```json
{
  "git_enabled": false
}
```

## 故障排除

### 为什么我的任务没有完成?

常见原因:

1. 任务描述不清楚
2. 要求对于单个提示来说太复杂
3. 代理不理解格式
4. 缺少资源或依赖项

### Ralph 一直遇到相同的错误

尝试:

1. 简化任务
2. 在 PROMPT.md 中添加澄清
3. 使用不同的代理
4. 手动修复特定问题

### 如何降低 API 成本?

1. 使用更高效的代理(Q 是免费的)
2. 减少最大迭代次数
3. 编写更清晰的提示以减少迭代
4. 使用检查点恢复而不是重新开始

### 我可以离线使用 Ralph 吗?

不可以,Ralph 需要互联网访问来与 AI 代理 API 通信。但是,如果您创建兼容的 CLI 包装器,可以使用本地 AI 模型。

## 高级使用

### 我可以用自定义代理扩展 Ralph 吗?

可以!实现 Agent 接口:

```python
class MyAgent(Agent):
    def __init__(self):
        super().__init__('myagent', 'myagent-cli')

    def execute(self, prompt_file):
        # 您的实现
        pass
```

### 我可以运行多个 Ralph 实例吗?

可以,但在不同的目录中以避免冲突:

```bash
# 终端 1
cd project1 && ralph run

# 终端 2
cd project2 && ralph run
```

### 如何将 Ralph 集成到 CI/CD 中?

```yaml
# GitHub Actions 示例
- name: Run Ralph
  run: |
    ralph run --max-iterations 50 --dry-run

- name: Check completion
  run: |
    ralph status
```

### Ralph 可以修改项目外的文件吗?

默认情况下,Ralph 在当前目录内工作。为了安全起见,它被设计为不修改系统文件或项目目录外的文件。

## 最佳实践

### 什么构成好的提示?

好的提示应该是:

- **具体的**: 清晰的要求和约束
- **可衡量的**: 定义的成功标准
- **结构化的**: 按部分组织
- **完整的**: 包含所有必要信息

### 我应该将 PROMPT.md 提交到 Git 吗?

应该!对提示进行版本控制以:

- 跟踪需求变更
- 与团队成员共享
- 重现结果
- 构建提示库

### 我应该多久检查一次 Ralph?

对于典型任务:

- 前 5 次迭代: 仔细监视
- 5-20 次迭代: 每 5 分钟检查一次
- 20+ 次迭代: 每 15 分钟检查一次

### 何时应该手动干预?

在以下情况下干预:

- 相同错误重复 3 次以上
- 进度停滞 10 次以上迭代
- 输出偏离要求
- 资源使用过多

## 成本和性能

### 运行 Ralph 需要多少费用?

每个任务的近似成本:

- 简单函数: $0.05-0.10
- Web API: $0.20-0.30
- 复杂应用程序: $0.50-1.00

(因代理和 API 定价而异)

### 哪个代理最快?

一般来说:

1. **Q**: 响应时间最快
2. **Gemini**: 平衡的速度和能力
3. **Claude**: 能力最强但较慢

### 如何加快执行速度?

1. 使用更简单的提示
2. 减少上下文大小
3. 选择更快的代理
4. 增加系统资源
5. 禁用不必要的功能

### Ralph 可以处理速率限制吗?

可以,Ralph 使用以下方式处理速率限制:

- 指数退避
- 重试逻辑
- 代理切换(如果有多个可用)

## 安全和隐私

### 我的代码会发送给 AI 提供商吗?

会,PROMPT.md 的内容和相关文件会发送到 AI 代理的 API。永远不要包含敏感数据,例如:

- API 密钥
- 密码
- 个人信息
- 专有代码

### 如何保护敏感信息?

1. 为机密使用环境变量
2. 将敏感文件添加到 .gitignore
3. 运行前检查提示
4. 使用本地开发凭据
5. 审查生成的代码

### Ralph 可以访问我的系统吗?

Ralph 在子进程中运行 AI 代理,具有:

- 超时保护
- 资源限制
- 工作目录限制

但是,代理可以执行代码,因此请始终检查输出。

### 在生产服务器上运行 Ralph 安全吗?

不推荐。Ralph 专为开发环境设计。对于生产环境,请在本地运行 Ralph 并部署经过测试的代码。

## 社区和支持

### 如何报告错误?

1. 在 GitHub 上检查现有问题
2. 创建详细的错误报告,包括:
   - Ralph 版本
   - 错误消息
   - 重现步骤
   - 系统信息

### 我可以为 Ralph 做贡献吗?

可以!我们欢迎贡献:

- 错误修复
- 新功能
- 文档改进
- 代理集成

请参阅 [CONTRIBUTING.md](contributing.md) 了解指南。

### 在哪里可以获得帮助?

- **GitHub Issues**: 错误报告和功能请求
- **GitHub Discussions**: 问题和社区帮助
- **Discord**: 与社区实时聊天

### 是否有商业支持?

目前,Ralph Orchestrator 是社区支持的开源软件。将来可能会提供商业支持。
