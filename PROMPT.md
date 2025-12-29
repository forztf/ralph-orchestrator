将所有的markdown文件翻译为中文。
- 保留原文件
- 不能批量翻译
- 不能使用脚本翻译

## 翻译进度

### 已完成的文件
- ✅ README.md → README.zh.md
- ✅ docs/index.md → docs/index.zh.md
- ✅ docs/quick-start.md → docs/quick-start.zh.md
- ✅ docs/installation.md → docs/installation.zh.md
- ✅ docs/guide/overview.md → docs/guide/overview.zh.md
- ✅ docs/guide/claude-adapter.zh.md
- ✅ docs/guide/agents.md → docs/guide/agents.zh.md
- ✅ docs/guide/checkpointing.md → docs/guide/checkpointing.zh.md
- ✅ docs/guide/configuration.md → docs/guide/configuration.zh.md
- ✅ docs/guide/cost-management.md → docs/guide/cost-management.zh.md
- ✅ docs/guide/prompts.md → docs/guide/prompts.zh.md
- ✅ docs/guide/web-monitoring-complete.md → docs/guide/web-monitoring-complete.zh.md
- ✅ docs/guide/web-monitoring.md → docs/guide/web-monitoring.zh.md
- ✅ docs/guide/web-quickstart.md → docs/guide/web-quickstart.zh.md
- ✅ docs/guide/websearch.md → docs/guide/websearch.zh.md
- ✅ docs/examples/bug-fix.md → docs/examples/bug-fix.zh.md
- ✅ docs/examples/cli-tool.md → docs/examples/cli-tool.zh.md
- ✅ docs/examples/data-analysis.md → docs/examples/data-analysis.zh.md
- ✅ docs/examples/documentation.md → docs/examples/documentation.zh.md
- ✅ docs/examples/index.md → docs/examples/index.zh.md
- ✅ docs/examples/simple-task.md → docs/examples/simple-task.zh.md
- ✅ docs/examples/testing.md → docs/examples/testing.zh.md
- ✅ docs/examples/web-api.md → docs/examples/web-api.zh.md
- ✅ docs/api/agents.md → docs/api/agents.zh.md
- ✅ docs/api/cli.md → docs/api/cli.zh.md
- ✅ docs/api/config.md → docs/api/config.zh.md
- ✅ docs/api/metrics.md → docs/api/metrics.zh.md
- ✅ docs/api/orchestrator.md → docs/api/orchestrator.zh.md
- ✅ docs/03-best-practices/best-practices.md → docs/03-best-practices/best-practices.zh.md
- ✅ docs/06-analysis/comparison-matrix.md → docs/06-analysis/comparison-matrix.zh.md

### 待翻译的文件 (按优先级排序)

#### 示例文档 (高优先级)
(所有示例文档已完成)

#### API 文档 (中等优先级)
(所有 API 文档已完成)

#### 高级主题 (中等优先级)
- docs/advanced/architecture.md → docs/advanced/architecture.zh.md
- docs/advanced/context-management.md → docs/advanced/context-management.zh.md
- docs/advanced/loop-detection.md → docs/advanced/loop-detection.zh.md
- docs/advanced/monitoring.md → docs/advanced/monitoring.zh.md
- docs/advanced/production-deployment.md → docs/advanced/production-deployment.zh.md
- docs/advanced/security.md → docs/advanced/security.zh.md

#### 部署文档 (中等优先级)
- docs/deployment/ci-cd.md → docs/deployment/ci-cd.zh.md
- docs/deployment/docker.md → docs/deployment/docker.zh.md
- docs/deployment/kubernetes.md → docs/deployment/kubernetes.zh.md
- docs/deployment/production.md → docs/deployment/production.zh.md
- docs/deployment/qchat-production.md → docs/deployment/qchat-production.zh.md

#### 其他文档 (低优先级)
- docs/changelog.md → docs/changelog.zh.md
- docs/contributing.md → docs/contributing.zh.md
- docs/faq.md → docs/faq.zh.md
- docs/glossary.md → docs/glossary.zh.md
- docs/license.md → docs/license.zh.md
- docs/research.md → docs/research.zh.md
- docs/testing.md → docs/testing.zh.md
- docs/troubleshooting.md → docs/troubleshooting.zh.md

#### 示例目录
- examples/cli_tool.md → examples/cli_tool.zh.md
- examples/simple_function.md → examples/simple_function.zh.md
- examples/simple-task.md → examples/simple-task.zh.md
- examples/web_scraper.md → examples/web_scraper.zh.md
- examples/web-api.md → examples/web-api.zh.md

#### 提示词目录
- prompts/WEB_PROMPT.md → prompts/WEB_PROMPT.zh.md

## 工作说明
每次迭代翻译一个文件,遵循以下步骤:
1. 选择下一个待翻译的文件(按优先级顺序)
2. 读取原文件内容
3. 翻译为中文(保留代码块、命令、技术术语)
4. 保存为 .zh.md 文件
5. 更新本进度文件
6. 提交 Git 更改