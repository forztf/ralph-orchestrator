# Ralph Orchestrator

[![文档](https://img.shields.io/badge/docs-mkdocs-blue)](https://mikeyobrien.github.io/ralph-orchestrator/)
[![版本](https://img.shields.io/badge/version-1.2.0-green)](https://github.com/mikeyobrien/ralph-orchestrator/releases)
[![许可证](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![测试](https://img.shields.io/badge/tests-passing-brightgreen)](tests/)

Ralph Wiggum 软件工程技术的一个还算实用的实现——将 AI 代理放入循环中直到任务完成。

> "我英语挂科？那是不可能的！" - Ralph Wiggum

### 注意：这是一个玩具项目，本身就是用 ralph wiggum 技术构建的。在我清理代码的过程中，预计会有错误、缺失功能和破坏性更改。主要使用 Claude Agent SDK 路径进行测试

## 📚 文档

**[查看完整文档](https://mikeyobrien.github.io/ralph-orchestrator/)** | [快速开始](https://mikeyobrien.github.io/ralph-orchestrator/quick-start/) | [API 参考](https://mikeyobrien.github.io/ralph-orchestrator/api/) | [示例](https://mikeyobrien.github.io/ralph-orchestrator/examples/)

## 概述

Ralph Orchestrator 实现了一个简单而有效的模式，使用 AI 代理完成自主任务。它会持续运行 AI 代理对抗提示文件，直到任务被标记为完成或达到限制。

基于 [Geoffrey Huntley](https://ghuntley.com/ralph/) 的 Ralph Wiggum 技术，这个实现为 AI 驱动的开发提供了一个健壮、经过测试且功能完整的编排系统。

## ✅ 生产就绪 - v1.2.0

- **Claude 集成**：✅ 完成（使用 Agent SDK）
- **Q Chat 集成**：✅ 完成
- **Gemini 集成**：✅ 完成
- **ACP 协议支持**：✅ 完成（代理客户端协议）
- **核心编排**：✅ 运行中
- **测试套件**：✅ 920+ 测试通过
- **文档**：✅ [完成](https://mikeyobrien.github.io/ralph-orchestrator/)
- **生产部署**：✅ [就绪](https://mikeyobrien.github.io/ralph-orchestrator/advanced/production-deployment/)

## 功能特性

- 🤖 **多 AI 代理支持**：适用于 Claude、Q Chat、Gemini CLI 和 ACP 兼容代理
- 🔍 **自动检测**：自动检测哪些 AI 代理可用
- 🌐 **网络搜索支持**：Claude 可以搜索网络获取当前信息
- 💾 **检查点**：基于 Git 的异步检查点，用于恢复和历史记录
- 📚 **提示归档**：跟踪提示在迭代过程中的演变
- 🔄 **错误恢复**：使用指数退避自动重试（非阻塞）
- 📊 **状态持久化**：保存指标和状态以供分析
- ⏱️ **可配置限制**：设置最大迭代次数和运行时间限制
- 🧪 **全面测试**：620+ 测试，包含单元、集成和异步覆盖
- 🎨 **丰富的终端输出**：带有语法高亮的漂亮格式化输出
- 🔒 **安全功能**：自动屏蔽日志中的 API 密钥等敏感数据
- ⚡ **异步优先设计**：整个过程中都是非阻塞 I/O（日志记录、git 操作）
- 📝 **内联提示**：使用 `-p "你的任务"` 运行，无需文件
- 🧠 **代理草稿本**：ACP 代理通过 `.agent/scratchpad.md` 在迭代过程中保持上下文

## 安装

```bash
# 克隆仓库
git clone https://github.com/mikeyobrien/ralph-orchestrator.git
cd ralph-orchestrator

# 使用 uv 安装（推荐）
uv sync

# 或者使用 pip 安装（需要在虚拟环境中）
python -m pip install -e .
```

## 前置要求

必须安装至少一个 AI CLI 工具：

- **[Claude SDK](https://pypi.org/project/claude-code-sdk/)**
  ```bash
  # 通过依赖项自动安装
  # 需要 ANTHROPIC_API_KEY 环境变量和适当的权限：
  # - 对话的读/写访问权限
  # - 模型访问权限（Claude 3.5 Sonnet 或类似模型）
  # - 连续操作的充足速率限制
  
  export ANTHROPIC_API_KEY="sk-ant-..."
  ```

- **[Q Chat](https://github.com/qchat/qchat)**
  ```bash
  # 按照仓库中的安装说明操作
  ```

- **[Gemini CLI](https://github.com/google-gemini/gemini-cli)**
  ```bash
  npm install -g @google/gemini-cli
  ```

- **ACP 兼容代理**（代理客户端协议）
  ```bash
  # 任何 ACP 兼容代理都可以通过 ACP 适配器使用
  # 示例：ACP 模式的 Gemini CLI
  ralph run -a acp --acp-agent gemini
  ```

## 快速开始

### 1. 初始化项目
```bash
ralph init
```

这会创建：
- `PROMPT.md` - 任务描述模板
- `ralph.yml` - 配置文件
- `.agent/` - 工作区目录，用于提示、检查点、指标、计划和内存

### 2. 配置 Ralph（可选）
编辑 `ralph.yml` 自定义设置：
```yaml
# Ralph Orchestrator 配置
agent: auto                    # 使用哪个代理：claude, q, gemini, acp, auto
prompt_file: PROMPT.md         # 提示文件路径
max_iterations: 100            # 停止前的最大迭代次数
max_runtime: 14400             # 最大运行时间（秒）（4 小时）
verbose: false                 # 启用详细输出

# 适配器配置
adapters:
  claude:
    enabled: true
    timeout: 300              # 超时时间（秒）
  q:
    enabled: true
    timeout: 300
  gemini:
    enabled: true
    timeout: 300
  acp:                        # 代理客户端协议适配器
    enabled: true
    timeout: 300
    tool_permissions:
      agent_command: gemini   # 运行 ACP 代理的命令
      agent_args: []          # 附加参数
      permission_mode: auto_approve  # auto_approve, deny_all, allowlist, interactive
      permission_allowlist: []  # 允许列表模式的模式
```

### 3. 用您的任务编辑 PROMPT.md
```markdown
# 任务：构建 Python 计算器

创建带有以下功能的计算器模块：
- 基本运算（加、减、乘、除）
- 除零错误处理
- 所有函数的单元测试

<!-- Ralph 将继续迭代直到达到限制 -->
```

### 4. 运行 Ralph
```bash
ralph run
# 或者使用配置文件
ralph -c ralph.yml
```

## 使用方法

### 基本命令

```bash
# 使用自动检测的代理运行
ralph

# 使用配置文件
ralph -c ralph.yml

# 使用特定代理
ralph run -a claude
ralph run -a q
ralph run -a gemini
ralph run -a acp               # ACP 兼容代理

# 检查状态
ralph status

# 清理工作区
ralph clean

# 试运行（不执行测试）
ralph run --dry-run
```

### 高级选项

```bash
ralph [选项] [命令]

命令：
  init                            初始化新的 Ralph 项目
  status                          显示当前 Ralph 状态  
  clean                           清理代理工作区
  prompt                          从粗略想法生成结构化提示
  run                             运行编排器（默认）

核心选项：
  -c, --config CONFIG             配置文件（YAML 格式）
  -a, --agent {claude,q,gemini,acp,auto}  要使用的 AI 代理（默认：auto）
  -P, --prompt-file FILE          提示文件路径（默认：PROMPT.md）
  -p, --prompt-text TEXT          内联提示文本（覆盖文件）
  -i, --max-iterations N          最大迭代次数（默认：100）
  -t, --max-runtime SECONDS      最大运行时间（默认：14400）
  -v, --verbose                   启用详细输出
  -d, --dry-run                   不执行代理的测试模式

ACP 选项：
  --acp-agent COMMAND             ACP 代理命令（默认：gemini）
  --acp-permission-mode MODE      权限处理：auto_approve, deny_all, allowlist, interactive

高级选项：
  --max-tokens MAX_TOKENS         最大总令牌数（默认：1000000）
  --max-cost MAX_COST             最大成本（美元）（默认：50.0）
  --checkpoint-interval N         Git 检查点间隔（默认：5）
  --retry-delay SECONDS           错误重试延迟（默认：2）
  --no-git                        禁用 git 检查点
  --no-archive                    禁用提示归档
  --no-metrics                    禁用指标收集
```

## ACP（代理客户端协议）集成

Ralph 通过其 ACP 适配器支持任何 ACP 兼容代理。这支持与实现[代理客户端协议](https://github.com/anthropics/agent-client-protocol)的代理（如 Gemini CLI）的集成。

### ACP 快速开始

```bash
# 与 Gemini CLI 的基本用法
ralph run -a acp --acp-agent gemini

# 使用权限模式
ralph run -a acp --acp-agent gemini --acp-permission-mode auto_approve
```

### 权限模式

ACP 适配器支持四种权限模式来处理代理工具请求：

| 模式 | 描述 | 用例 |
|------|-------------|----------|
| `auto_approve` | 自动批准所有请求 | 可信环境、CI/CD |
| `deny_all` | 拒绝所有权限请求 | 测试、沙箱执行 |
| `allowlist` | 仅批准匹配的模式 | 使用特定工具的生产环境 |
| `interactive` | 对每个请求提示用户 | 开发、手动监督 |

### 配置

在 `ralph.yml` 中配置 ACP：

```yaml
adapters:
  acp:
    enabled: true
    timeout: 300
    tool_permissions:
      agent_command: gemini      # 代理 CLI 命令
      agent_args: []             # 附加 CLI 参数
      permission_mode: auto_approve
      permission_allowlist:      # 用于允许列表模式
        - "fs/read_text_file:*.py"
        - "fs/write_text_file:src/*"
        - "terminal/create:pytest*"
```

### 代理草稿本

ACP 代理通过 `.agent/scratchpad.md` 在迭代过程中保持上下文。此文件持久保存：
- 之前迭代的进度
- 决策和上下文
- 当前的阻塞或问题
- 剩余工作项

草稿本使代理能够从上次中断的地方继续，而不是在每次迭代时重新开始。

### 支持的操作

ACP 适配器处理这些代理请求：

**文件操作：**
- `fs/read_text_file` - 读取文件内容（带路径安全验证）
- `fs/write_text_file` - 写入文件内容（带路径安全验证）

**终端操作：**
- `terminal/create` - 使用命令创建子进程
- `terminal/output` - 读取进程输出
- `terminal/wait_for_exit` - 等待进程完成
- `terminal/kill` - 终止进程
- `terminal/release` - 释放终端资源

## 工作原理

### Ralph 循环

```
┌─────────────────┐
│  读取 PROMPT.md │
└────────┬────────┘
         │
         v
┌─────────────────┐
│ 执行 AI 代理│<──────┐
└────────┬────────┘       │
         │                │
         v                │
┌─────────────────┐       │
│ 检查完成？ │───No──┘
└────────┬────────┘
         │Yes
         v
┌─────────────────┐
│      完成！      │
└─────────────────┘
```

### 执行流程

1. **初始化**：创建 `.agent/` 目录并验证提示文件
2. **代理检测**：自动检测可用的 AI 代理（claude、q、gemini）
3. **迭代循环**： 
   - 使用当前提示执行 AI 代理
   - 监控任务完成标记
   - 在间隔处创建检查点
   - 使用重试逻辑处理错误
4. **完成**：在以下情况下停止：
   - 达到最大迭代次数
   - 超过最大运行时间
   - 达到成本限制
   - 连续错误过多

## 项目结构

```
ralph-orchestrator/
├── src/
│   └── ralph_orchestrator/
│       ├── __main__.py      # CLI 入口点
│       ├── main.py          # 配置和类型
│       ├── orchestrator.py  # 核心编排逻辑（异步）
│       ├── adapters/        # AI 代理适配器
│       │   ├── base.py      # 基础适配器接口
│       │   ├── claude.py    # Claude Agent SDK 适配器
│       │   ├── gemini.py    # Gemini CLI 适配器
│       │   ├── qchat.py     # Q Chat 适配器
│       │   ├── acp.py       # ACP（代理客户端协议）适配器
│       │   ├── acp_protocol.py  # JSON-RPC 2.0 协议处理
│       │   ├── acp_client.py    # 子进程管理器
│       │   ├── acp_models.py    # 数据模型
│       │   └── acp_handlers.py  # 权限/文件/终端处理程序
│       ├── output/          # 输出格式化（新增）
│       │   ├── base.py      # 基础格式化器接口
│       │   ├── console.py   # 富控制台输出
│       │   ├── rich_formatter.py  # 富文本格式化
│       │   └── plain.py     # 纯文本回退
│       ├── async_logger.py  # 线程安全的异步日志记录
│       ├── context.py       # 上下文管理
│       ├── logging_config.py # 集中式日志设置
│       ├── metrics.py       # 指标跟踪
│       ├── security.py      # 安全验证和屏蔽
│       └── safety.py        # 安全检查
├── tests/                   # 测试套件（620+ 测试）
│   ├── test_orchestrator.py
│   ├── test_adapters.py
│   ├── test_async_logger.py
│   ├── test_output_formatters.py
│   ├── test_config.py
│   ├── test_integration.py
│   └── test_acp_*.py        # ACP 适配器测试（305+ 测试）
├── docs/                    # 文档
├── PROMPT.md               # 任务描述（用户创建）
├── ralph.yml               # 配置文件（由 init 创建）
├── pyproject.toml          # 项目配置
├── .agent/                 # CLI 工作区（由 init 创建）
│   ├── prompts/            # 提示工作区
│   ├── checkpoints/        # 检查点标记
│   ├── metrics/            # 指标数据
│   ├── plans/              # 规划文档
│   └── memory/             # 代理内存
├── .ralph/                 # 运行时指标目录
└── prompts/                # 提示归档目录
    └── archive/            # 归档的提示历史
```

## 测试

### 运行测试套件

```bash
# 所有测试
uv run pytest -v

# 带覆盖率
uv run pytest --cov=ralph_orchestrator

# 特定测试文件
uv run pytest tests/test_orchestrator.py -v

# 仅集成测试
uv run pytest tests/test_integration.py -v
```

### 测试覆盖率

- ✅ 所有核心函数的单元测试
- ✅ 使用模拟代理的集成测试
- ✅ CLI 接口测试
- ✅ 错误处理和恢复测试
- ✅ 状态持久化测试

## 示例

### 内联提示（快速任务）

```bash
# 直接使用内联提示运行 - 无需文件
ralph run -p "编写一个 Python 函数来检查数字是否为质数" -a claude --max-iterations 5
```

### 简单函数（基于文件）

```bash
echo "编写一个 Python 函数来检查数字是否为质数" > PROMPT.md
ralph run -a claude --max-iterations 5
```

### Web 应用程序

```bash
cat > PROMPT.md << 'EOF'
构建一个带有以下功能的 Flask Web 应用：
- 用户注册和登录
- SQLite 数据库
- 基本 CRUD 操作
- Bootstrap UI
EOF

ralph run --max-iterations 50
```

### 测试驱动开发

```bash
cat > PROMPT.md << 'EOF'
使用 TDD 在 Python 中实现链表：
1. 先写测试
2. 实现方法通过测试
3. 添加插入、删除、搜索操作
4. 确保 100% 测试覆盖率
EOF

ralph run -a q --verbose
```

## 监控

### 检查状态
```bash
# 一次性状态检查
ralph status

# 示例输出：
Ralph Orchestrator 状态
=========================
提示：PROMPT.md 存在
状态：进行中
最新指标：.ralph/metrics_20250907_154435.json
{
  "iteration_count": 15,
  "runtime": 234.5,
  "errors": 0
}
```

### 查看日志
```bash
# 如果使用详细模式
ralph run --verbose 2>&1 | tee ralph.log

# 检查 git 历史
git log --oneline | grep "Ralph checkpoint"
```

## 错误恢复

Ralph 优雅地处理错误：

- **重试逻辑**：失败的迭代在可配置的延迟后重试
- **错误限制**：连续 5 次错误后停止
- **超时保护**：每次迭代 5 分钟超时
- **状态持久化**：可以从保存的状态分析失败
- **Git 恢复**：可以重置到最后一个工作检查点

### 手动恢复

```bash
# 检查最后一个错误
cat .ralph/metrics_*.json | jq '.errors[-1]'

# 重置到最后一个检查点
git reset --hard HEAD

# 清理并重新启动
ralph clean
ralph run
```

## 最佳实践

1. **清晰的任务定义**：编写具体、可衡量的需求
2. **增量目标**：将复杂任务分解为更小的步骤
3. **成功标记**：定义明确的完成标准
4. **定期检查点**：使用默认的 5 次迭代检查点
5. **监控进度**：使用 `ralph status` 跟踪迭代
6. **版本控制**：开始前提交 PROMPT.md

## 故障排除

### 找不到代理
```bash
# 对于 Claude，确保设置了具有适当权限的 API 密钥
export ANTHROPIC_API_KEY="sk-ant-..."

# 验证 Claude API 密钥权限：
# - 应该具有 Claude 3.5 Sonnet 或类似模型的访问权限
# - 需要充足的速率限制（每分钟至少 40,000 个令牌）
# - 需要 API 的读/写访问权限

# 对于 Q 和 Gemini，检查 CLI 工具是否已安装
which q
which gemini

# 根据需要安装缺失的 CLI 工具
```

### 任务未完成
```bash
# 检查迭代次数和进度
ralph status

# 查看代理错误
cat .agent/metrics/state_*.json | jq '.errors'

# 尝试不同的代理
ralph run -a gemini
```

### 性能问题
```bash
# 减少迭代超时
ralph run --max-runtime 1800

# 增加检查点频率
ralph run --checkpoint-interval 3
```

## 研究与理论

Ralph Wiggum 技术基于以下几个关键原则：

1. **简单胜过复杂**：保持编排最小化（约 400 行）
2. **确定性失败**：在不可预测的世界中可预测地失败
3. **上下文恢复**：使用 git 和状态文件进行持久化
4. **人在回路中**：在需要时允许手动干预

有关详细研究和理论基础，请参见 [研究目录](../README.md)。

## 贡献

欢迎贡献！请：

1. Fork 仓库
2. 创建功能分支（`git checkout -b feature/amazing-feature`）
3. 为新功能编写测试
4. 确保所有测试通过（`uv run pytest`）
5. 提交更改（`git commit -m 'Add amazing feature'`）
6. 推送到分支（`git push origin feature/amazing-feature`）
7. 打开拉取请求

## 许可证

MIT 许可证 - 详见 LICENSE 文件

## 致谢

- **[Geoffrey Huntley](https://ghuntley.com/ralph/)** - Ralph Wiggum 技术的创建者
- **[Harper Reed](https://harper.blog/)** - 规范驱动的开发方法论
- **Anthropic、Google、Q** - 提供优秀的 AI CLI 工具

## 支持

- **文档**：[完整文档](https://mikeyobrien.github.io/ralph-orchestrator/)
- **部署指南**：[生产部署](https://mikeyobrien.github.io/ralph-orchestrator/advanced/production-deployment/)
- **问题**：[GitHub Issues](https://github.com/mikeyobrien/ralph-orchestrator/issues)
- **讨论**：[GitHub Discussions](https://github.com/mikeyobrien/ralph-orchestrator/discussions)
- **研究**：[Ralph Wiggum 研究](../)

## 版本历史

- **v1.2.0** (2025-12)
  - **ACP（代理客户端协议）支持**：与 ACP 兼容代理的完整集成
    - JSON-RPC 2.0 消息协议
    - 权限处理（auto_approve、deny_all、allowlist、interactive）
    - 文件操作（带安全性的读/写）
    - 终端操作（创建、输出、等待、终止、释放）
    - 会话管理和流式更新
    - 用于在迭代过程中保持上下文的代理草稿本机制
  - 新的 CLI 选项：`--acp-agent`、`--acp-permission-mode`
  - ralph.yml 中的配置支持
  - 305+ 个新的 ACP 特定测试
  - 扩展的测试套件（920+ 测试）

- **v1.1.0** (2025-12)
  - 异步优先架构，用于非阻塞操作
  - 带有轮换和安全屏蔽的线程安全异步日志记录
  - 带有语法高亮的丰富终端输出
  - 内联提示支持（`-p "你的任务"`）
  - 带有 MCP 服务器支持的 Claude Agent SDK 集成
  - 异步 git 检查点（非阻塞）
  - 扩展的测试套件（620+ 测试）
  - 带有调试日志记录的改进错误处理

- **v1.0.0** (2025-09-07)
  - 初始版本，支持 Claude、Q 和 Gemini
  - 全面的测试套件（17 个测试）
  - 生产就绪的错误处理
  - 完整文档
  - 基于 Git 的检查点
  - 状态持久化和指标

---

*"我在学习！" - Ralph Wiggum*

---

## 翻译说明

本文档是 Ralph Orchestrator 项目的中文翻译版本。原文档为英文，位于 [README.md](README.md)。

翻译遵循以下原则：
- 保持技术术语准确性
- 保留代码块和命令不变
- 保持文档结构一致性
- 使用标准中文技术表达

如发现翻译问题，欢迎提交 Issue 或 Pull Request 进行修正。