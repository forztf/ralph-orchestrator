# 更新日志

Ralph Orchestrator 的所有重要变更都将记录在此文件中。

格式基于 [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
并且本项目遵循 [语义化版本](https://semver.org/spec/v2.0.0.html)。

## [未发布]

### 新增

- **完成标记检测**: 任务现在可以通过提示词文件中的 `- [x] TASK_COMPLETE` 复选框标记来发出完成信号
  - 编排器在每次迭代之前检查标记
  - 发现标记时立即退出循环
  - 支持 `- [x] TASK_COMPLETE` 和 `[x] TASK_COMPLETE` 两种格式
- **循环检测**: 使用 rapidfuzz 自动检测重复的代理输出
  - 将当前输出与最近 5 次输出进行比较
  - 使用 90% 相似度阈值来检测循环
  - 防止代理失控导致无限循环
- 新依赖: `rapidfuzz>=3.0.0,<4.0.0` 用于快速模糊字符串匹配
- 使用 MkDocs 的文档静态站点
- 全面的 API 参考文档
- 额外的示例场景
- 性能监控工具

### 变更

- 改进代理执行中的错误处理
- 增强检查点创建逻辑
- `SafetyGuard.reset()` 现在也会清除循环检测历史

### 修复

- 状态文件更新中的竞态条件
- 长时间运行会话中的内存泄漏

## [1.2.0] - 2025-12

### 新增

- **ACP (代理客户端协议) 支持**: 与符合 ACP 标准的代理完全集成
  - JSON-RPC 2.0 消息协议实现
  - 权限处理,支持四种模式: `auto_approve`、`deny_all`、`allowlist`、`interactive`
  - 文件操作 (`fs/read_text_file`、`fs/write_text_file`) 带安全验证
  - 终端操作 (`terminal/create`、`terminal/output`、`terminal/wait_for_exit`、`terminal/kill`、`terminal/release`)
  - 会话管理和流式更新
  - 代理草稿板机制,用于跨迭代的上下文持久化
- 新的 CLI 选项: `--acp-agent`、`--acp-permission-mode`
- 在 `ralph.yml` 的 `adapters.acp` 下支持 ACP 配置
- 环境变量覆盖: `RALPH_ACP_AGENT`、`RALPH_ACP_PERMISSION_MODE`、`RALPH_ACP_TIMEOUT`
- 305+ 新的 ACP 专用测试

### 变更

- 扩展测试套件至 920+ 测试
- 更新 ACP 支持的文档

## [1.1.0] - 2025-12

### 新增

- 异步优先架构,用于非阻塞操作
- 线程安全的异步日志记录,支持轮转和安全掩码
- 带语法高亮的美观终端输出
- 内联提示词支持 (`-p "your task"`)
- Claude Agent SDK 集成,支持 MCP 服务器
- 异步 Git 检查点 (非阻塞)
- 安全验证系统,带路径遍历保护
- 日志中的敏感数据掩码 (API 密钥、令牌、密码)
- 带有 RLock 的线程安全配置
- VerboseLogger,带会话指标和可重入性保护
- 迭代统计跟踪,采用内存高效存储

### 变更

- 扩展测试套件至 620+ 测试
- 使用 ClaudeErrorFormatter 改进错误处理
- 使用进程优先清理增强信号处理

### 修复

- 倒计时进度条中的除以零错误
- QChatAdapter 中的进程引用泄漏
- 异步函数中的阻塞文件 I/O
- 错误处理程序中的异常链

## [1.0.3] - 2025-09-07

### 新增

- 生产部署指南
- Docker 支持,包含 Dockerfile 和 docker-compose.yml
- Kubernetes 部署清单
- 用于监控的健康检查端点

### 变更

- 改进资源限制处理
- 增强日志记录,支持结构化 JSON 输出
- 更新依赖项到最新版本

### 修复

- Windows 上的 Git 检查点创建
- 边缘情况下的代理超时处理

## [1.0.2] - 2025-09-07

### 新增

- Q Chat 集成改进
- 实时指标收集
- 交互式 CLI 模式
- Bash 和 ZSH 自动补全脚本

### 变更

- 重构代理管理器以提高可扩展性
- 改进上下文窗口管理
- 增强进度报告

### 修复

- 提示词文件中的 Unicode 处理
- 跨中断的状态持久化

## [1.0.1] - 2025-09-07

### 新增

- Gemini CLI 集成
- 高级上下文管理策略
- 成本跟踪和估算
- HTML 报告生成

### 变更

- 优化迭代性能
- 改进错误恢复机制
- 增强 Git 操作

### 修复

- macOS 上的代理检测
- 带特殊字符的提示词归档
- 检查点间隔计算

## [1.0.0] - 2025-09-07

### 新增

- 初始版本,包含核心功能
- Claude CLI 集成
- Q Chat 集成
- 基于 Git 的检查点
- 提示词归档
- 状态持久化
- 全面的测试套件
- CLI 包装脚本
- 配置管理
- 指标收集

### 特性

- 自动检测可用的 AI 代理
- 可配置的迭代和运行时限制
- 带指数退避的错误恢复
- 详细模式和干运行模式
- JSON 配置文件支持
- 环境变量配置

### 文档

- 带示例的完整 README
- 安装说明
- 使用指南
- API 文档
- 贡献指南

## [0.9.0] - 2025-09-06 (Beta)

### 新增

- 用于测试的 Beta 版本
- 基本编排循环
- Claude 集成
- 简单的检查点

### 已知问题

- 错误处理有限
- 无指标收集
- 仅支持单个代理

## [0.5.0] - 2025-09-05 (Alpha)

### 新增

- 初始 Alpha 版本
- 概念验证实现
- 基本 Ralph 循环
- 仅手动测试

---

## 版本历史摘要

### 主要版本

- **1.0.0** - 首个稳定版本,包含完整功能集
- **0.9.0** - 用于社区测试的 Beta 版本
- **0.5.0** - Alpha 概念验证

### 版本控制策略

我们使用语义化版本 (SemVer):

- **MAJOR** 版本: 不兼容的 API 变更
- **MINOR** 版本: 向后兼容的功能新增
- **PATCH** 版本: 向后兼容的错误修复

### 弃用策略

被标记为弃用的功能将:

1. 在更新日志中记录
2. 在 2 个次要版本中显示弃用警告
3. 在下一个主要版本中移除

### 支持策略

- **当前版本**: 全面支持,包含错误修复和功能
- **上一版本**: 仅错误修复
- **旧版本**: 仅社区支持

## 升级指南

### 从 0.x 升级到 1.0

1. **配置变更**
   - 旧: `max_iter` → 新: `max_iterations`
   - 旧: `agent_name` → 新: `agent`

2. **API 变更**
   - `RalphOrchestrator.execute()` → `RalphOrchestrator.run()`
   - 返回格式从元组更改为字典

3. **文件结构**
   - 状态文件从 `.ralph/` 移动到 `.agent/metrics/`
   - 检查点格式已更新

### 迁移脚本

```bash
#!/bin/bash
# 从 0.x 迁移到 1.0

# 备份旧数据
cp -r .ralph .ralph.backup

# 创建新结构
mkdir -p .agent/metrics .agent/prompts .agent/checkpoints

# 迁移状态文件
mv .ralph/*.json .agent/metrics/ 2>/dev/null

# 更新配置
if [ -f "ralph.conf" ]; then
    python -c "
import json
with open('ralph.conf') as f:
    old_config = json.load(f)
# 更新键
old_config['max_iterations'] = old_config.pop('max_iter', 100)
old_config['agent'] = old_config.pop('agent_name', 'auto')
# 保存新配置
with open('ralph.json', 'w') as f:
    json.dump(old_config, f, indent=2)
"
fi

echo "迁移完成!"
```

## 发布流程

### 1. 发布前检查清单

- [ ] 所有测试通过
- [ ] 文档已更新
- [ ] 更新日志已更新
- [ ] setup.py 中的版本已更新
- [ ] 已测试 README 示例

### 2. 发布步骤

```bash
# 1. 更新版本
vim setup.py  # 更新版本号

# 2. 提交更改
git add -A
git commit -m "Release version X.Y.Z"

# 3. 标记发布
git tag -a vX.Y.Z -m "Version X.Y.Z"

# 4. 推送到 GitHub
git push origin main --tags

# 5. 创建 GitHub 发布
gh release create vX.Y.Z --title "Version X.Y.Z" --notes-file RELEASE_NOTES.md

# 6. 发布到 PyPI (如果适用)
python setup.py sdist bdist_wheel
twine upload dist/*
```

### 3. 发布后

- [ ] 在社交媒体上宣布
- [ ] 更新文档站点
- [ ] 关闭相关问题
- [ ] 规划下一次发布

## 贡献者

感谢所有帮助改进 Ralph Orchestrator 的贡献者:

- Geoffrey Huntley (@ghuntley) - 原始 Ralph Wiggum 技术
- 通过 GitHub 贡献的社区贡献者

## 如何贡献

参见 [CONTRIBUTING.md](contributing.md) 了解以下详细信息:

- 报告错误
- 建议功能
- 提交拉取请求
- 开发设置

## 链接

- [GitHub 仓库](https://github.com/mikeyobrien/ralph-orchestrator)
- [问题跟踪器](https://github.com/mikeyobrien/ralph-orchestrator/issues)
- [讨论区](https://github.com/mikeyobrien/ralph-orchestrator/discussions)
- [文档](https://mikeyobrien.github.io/ralph-orchestrator/)
