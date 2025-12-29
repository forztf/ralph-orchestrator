# 术语表

## A

**Agent** (代理)
: 基于 CLI 的 AI 工具,根据提示执行任务。Ralph 支持 Claude、Gemini 和 Q 代理。

**Agent Manager** (代理管理器)
: 负责检测、选择和管理 AI 代理的组件,包括当首选代理不可用时的自动回退机制。

**Archive** (归档)
: 在 `.agent/prompts/` 目录中存储历史提示和迭代记录,用于调试和分析。

**Auto-detection** (自动检测)
: Ralph 自动发现系统上已安装并可用的 AI 代理的能力。

## C

**Checkpoint** (检查点)
: 定期创建的 Git 提交,用于保存进度并支持恢复。默认间隔为每 5 次迭代一次。

**Claude**
: Anthropic 的 AI 助手,可通过 Claude Code CLI 访问。以大上下文窗口(200K tokens)和代码生成能力著称。

**CLI** (命令行界面)
: Command Line Interface - 与 Ralph Orchestrator 交互的主要方式,通过终端命令。

**Config** (配置)
: 存储在 `ralph.json` 中的配置设置,或通过命令行参数和环境变量传递。

**Context Window** (上下文窗口)
: AI 代理在单个请求中可处理的最大文本/token 数量。因代理而异(Claude: 200K, Gemini: 32K, Q: 8K)。

**Convergence** (收敛)
: 迭代通过持续改进逐步接近任务完成的过程。

**Completion Marker** (完成标记)
: 提示文件中的特殊复选框模式(`- [x] TASK_COMPLETE`),表示任务已完成。当编排器检测到此标记时,会立即退出循环。

## D

**Dry Run** (试运行)
: 模拟执行但不实际运行 AI 代理的测试模式。用于测试配置和设置。

**Deterministic Failure** (确定性失败)
: 一种哲学理念,认为可预测的失败胜过不可预测的成功 - 这是 Ralph Wiggum 技术的核心。

## E

**Exponential Backoff** (指数退避)
: 重试策略,每次失败后等待时间加倍(2、4、8、16 秒),以处理临时错误。

**Execution Cycle** (执行周期)
: 读取提示、执行代理、检查完成状态和更新状态的完整迭代过程。

## G

**Gemini**
: Google 的 AI 模型,可通过 Gemini CLI 访问。具有平衡的上下文窗口(32K tokens)和能力。

**Git Integration** (Git 集成)
: Ralph 使用 Git 进行检查点、历史跟踪和从失败状态恢复。

## I

**Iteration** (迭代)
: Ralph 循环的一个完整周期 - 使用当前提示执行代理并处理结果。

**Iteration Limit** (迭代限制)
: Ralph 停止前的最大迭代次数。默认为 100,可通过 `max_iterations` 配置。

## L

**Loop** (循环)
: Ralph 的核心模式 - 持续运行 AI 代理,直到任务完成或达到限制。

**Loop Detection** (循环检测)
: 安全功能,检测代理何时产生重复输出。使用模糊字符串匹配(rapidfuzz),相似度阈值为 90%,以比较最近的输出并防止无限循环。

## M

**Metrics** (指标)
: 在 `.agent/metrics/` 中收集的性能和执行数据,包括计时、错误和资源使用情况。

**MkDocs**
: 用于 Ralph 文档的静态站点生成器,在 `mkdocs.yml` 中配置。

## O

**Orchestrator** (编排器)
: 管理执行循环、代理交互和状态管理的主要组件。

## P

**Prompt** (提示)
: 任务描述文件(通常是 `PROMPT.md`),告诉 AI 代理要完成什么。

**Prompt Archive** (提示归档)
: 在 `.agent/prompts/` 中存储所有提示迭代的历史记录,用于调试和分析。

**Plugin** (插件)
: 为 Ralph 添加自定义代理或命令的扩展机制。

## Q

**Q Chat**
: 可通过 Q CLI 访问的 AI 助手。上下文窗口较小(8K tokens),但执行速度快。

## R

**Ralph Wiggum Technique** (Ralph Wiggum 技术)
: 将 AI 代理置于循环中直到任务完成的软件工程模式,由 Geoffrey Huntley 创建。

**Recovery** (恢复)
: 在失败或中断后,从保存的状态或 Git 检查点恢复执行的过程。

**Retry Logic** (重试逻辑)
: 使用指数退避处理临时故障的自动重试机制。

**Runtime Limit** (运行时限制)
: Ralph 停止前的最大执行时间(秒)。默认为 14400(4 小时)。

## S

**State** (状态)
: 当前执行状态,包括迭代计数、错误和指标,保存在 `.agent/metrics/state_latest.json` 中。

**State Manager** (状态管理器)
: 负责在迭代之间保存、加载和更新执行状态的组件。

**Success Criteria** (成功标准)
: 在 PROMPT.md 中定义的可测量目标,指导编排器完成。

## T

**Task** (任务)
: 要完成的工作,在 PROMPT.md 中描述,包含要求和成功标准。

**Task Complete** (任务完成)
: 编排器达到最大迭代次数、运行时间或成本限制,并朝着定义目标努力时的状态。

**Timeout** (超时)
: 单次代理执行允许的最长时间。默认为每次迭代 300 秒(5 分钟)。

**Token**
: AI 模型处理的文本单位。大约 4 个字符 = 1 个 token。

## U

**Unpossible**
: 引用 Ralph Wiggum 的话,体现了通过坚持实现看似不可能的事情的哲学。

## V

**Verbose Mode** (详细模式)
: 使用 `--verbose` 标志启用的详细日志模式,用于调试和监控。

## W

**Working Directory** (工作目录)
: Ralph 执行的目录,包含 PROMPT.md 和项目文件。默认为当前目录。

**Workspace** (工作空间)
: `.agent/` 目录,包含 Ralph 的运营数据,包括指标、检查点和归档。

## Technical Terms (技术术语)

**API** (应用程序接口)
: Application Programming Interface - Ralph 与 AI 服务通信的接口。

**JSON**
: JavaScript Object Notation - 用于配置文件和状态存储的格式。

**Subprocess** (子进程)
: 为执行 AI 代理而生成的独立进程,提供隔离和超时控制。

**YAML**
: YAML Ain't Markup Language - 人类可读的数据格式,用于某些配置文件。

## File Formats (文件格式)

**`.md`**
: 用于提示和文档的 Markdown 文件。

**`.json`**
: 用于配置和状态存储的 JSON 文件。

**`.yaml`/`.yml`**
: 用于配置的 YAML 文件(例如 `mkdocs.yml`)。

**`.log`**
: 包含执行历史和调试信息的日志文件。

## Directory Structure (目录结构)

**`.agent/`**
: Ralph 的工作空间目录,包含所有运营数据。

**`.agent/metrics/`**
: 执行指标和状态文件的存储。

**`.agent/prompts/`**
: 历史提示迭代的归档。

**`.agent/checkpoints/`**
: 执行期间创建的 Git 检查点标记。

**`.agent/logs/`**
: 用于调试和分析的执行日志。

**`.agent/plans/`**
: 代理规划文档和长期策略。

## Command Reference (命令参考)

**`ralph run`**
: 使用当前配置执行编排器。

**`ralph init`**
: 使用默认结构初始化新的 Ralph 项目。

**`ralph status`**
: 检查当前执行状态和指标。

**`ralph clean`**
: 清理工作空间并重置状态。

**`ralph agents`**
: 列出系统上可用的 AI 代理。

## Environment Variables (环境变量)

**`RALPH_AGENT`**
: 覆盖默认代理选择。

**`RALPH_MAX_ITERATIONS`**
: 设置最大迭代限制。

**`RALPH_MAX_RUNTIME`**
: 设置最大运行时间(秒)。

**`RALPH_VERBOSE`**
: 启用详细日志(true/false)。

**`RALPH_DRY_RUN`**
: 启用试运行模式(true/false)。

## Exit Codes (退出代码)

**0**: 成功 - 任务成功完成

**1**: 一般失败 - 检查日志获取详细信息

**130**: 中断 - 用户按下了 Ctrl+C

**137**: 终止 - 进程被终止(通常是内存问题)

**124**: 超时 - 执行时间超出
