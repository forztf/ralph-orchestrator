# Ralph Orchestrator

<div align="center">

## 生产就绪的 AI 编排

*让您的 AI 代理循环工作直到任务完成*

[![版本](https://img.shields.io/badge/version-1.2.0-blue)](https://github.com/mikeyobrien/ralph-orchestrator/releases)
[![许可证](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![测试](https://img.shields.io/badge/tests-920%2B%20passing-brightgreen)](tests/)
[![Python](https://img.shields.io/badge/python-3.8%2B-blue)](https://www.python.org/)

> "我英语挂科？那是不可能的！" - Ralph Wiggum

</div>

## Ralph Orchestrator 是什么？

Ralph Orchestrator 是 **Ralph Wiggum 编排技术**的生产就绪实现——一种简单而强大的自主 AI 任务完成模式。正如 [Geoffrey Huntley](https://ghuntley.com/ralph/) 最初定义的那样：**"Ralph 是一个 Bash 循环"**，它持续运行 AI 代理对抗提示文件，直到任务被标记为完成或达到限制。

基于 Huntley 的技术，这个实现提供了企业级的安全性、监控和成本控制，适用于生产环境。对于 Claude Code 用户，另请参见官方的 [ralph-wiggum 插件](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum)。

## 主要功能

<div class="grid cards" markdown>

- **🤖 多代理支持**
  与 Claude、Q Chat、Gemini CLI 和 ACP 兼容代理无缝协作，支持自动检测

- **💰 成本管理**  
  实时令牌跟踪、成本计算和可配置的支出限制

- **🔒 企业安全**  
  输入清理、命令注入预防和路径遍历保护

- **📊 生产监控**  
  系统指标、性能跟踪和详细的 JSON 导出

- **🔄 弹性执行**  
  自动重试、熔断器和状态持久化

- **💾 Git 检查点**
  版本控制集成，用于状态恢复和历史跟踪

- **🔌 ACP 协议支持**
  完整的代理客户端协议集成，具有权限处理、文件/终端操作和会话管理功能

</div>

## 快速示例

```bash
# 1. 创建任务提示
cat > PROMPT.md << EOF
创建一个计算斐波那契数列的 Python 函数。
包含适当的文档和单元测试。
编排器将迭代直到函数完成。
EOF

# 2. 运行 Ralph
python ralph_orchestrator.py --prompt PROMPT.md

# 3. Ralph 迭代直到任务完成！
```

## 为什么选择 Ralph Orchestrator？

### 问题
现代 AI 代理功能强大，但需要监督。它们可能失去上下文、犯错误，或需要多次迭代才能完成复杂任务。手动监督耗时且容易出错。

### 解决方案
Ralph Orchestrator 自动化迭代循环，同时保持安全性和控制：

- **自主操作**：设置好就不用管了 - Ralph 处理迭代
- **安全第一**：内置限制防止成本失控和无限循环
- **生产就绪**：经过实战测试，具有全面的错误处理
- **可观察**：详细的指标和日志记录，用于调试和优化
- **可恢复**：检查点系统允许从任何点恢复

## 使用场景

Ralph Orchestrator 擅长：

- **代码生成**：构建功能、修复错误、编写测试
- **文档**：创建综合文档、API 参考、教程
- **数据处理**：ETL 管道、数据分析、报告生成
- **自动化**：CI/CD 设置、部署脚本、基础设施即代码
- **研究**：信息收集、总结、分析

## 入门

准备好让 Ralph 工作了吗？查看我们的[快速开始指南](quick-start.md)，在几分钟内启动并运行。

## 生产功能

Ralph Orchestrator 专为生产使用而设计，具有：

- **令牌和成本限制**：防止预算超支
- **上下文管理**：智能处理大提示
- **安全控制**：防止恶意输入
- **监控和指标**：跟踪性能和使用情况
- **错误恢复**：优雅处理失败
- **状态持久化**：恢复中断的任务

在我们的[生产部署指南](advanced/production-deployment.md)中了解更多信息。

## 社区与支持

- 📖 [文档](https://mikeyobrien.github.io/ralph-orchestrator/)
- 🐛 [问题跟踪器](https://github.com/mikeyobrien/ralph-orchestrator/issues)
- 💬 [讨论](https://github.com/mikeyobrien/ralph-orchestrator/discussions)
- 🤝 [贡献指南](contributing.md)

## 许可证

Ralph Orchestrator 是开源软件，[采用 MIT 许可证](license.md)。

---

<div align="center">
<i>由 Ralph Orchestrator 社区用 ❤️ 构建</i>
</div>