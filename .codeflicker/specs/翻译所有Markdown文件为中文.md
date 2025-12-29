# Markdown文件翻译计划

## 项目结构分析
通过递归列出目录，发现了以下需要翻译的markdown文件：

### 根目录文件
- README.md → README.zh.md
- PROMPT.md → PROMPT.zh.md

### docs/ 目录文件
- docs/index.md → docs/index.zh.md
- docs/installation.md → docs/installation.zh.md
- docs/quick-start.md → docs/quick-start.zh.md
- docs/changelog.md → docs/changelog.zh.md
- docs/contributing.md → docs/contributing.zh.md
- docs/faq.md → docs/faq.zh.md
- docs/glossary.md → docs/glossary.zh.md
- docs/license.md → docs/license.zh.md
- docs/research.md → docs/research.zh.md
- docs/testing.md → docs/testing.md
- docs/troubleshooting.md → docs/troubleshooting.zh.md

### docs子目录文件
- docs/03-best-practices/best-practices.md → docs/03-best-practices/best-practices.zh.md
- docs/06-analysis/comparison-matrix.md → docs/06-analysis/comparison-matrix.zh.md

### docs/advanced/ 目录文件
- docs/advanced/architecture.md → docs/advanced/architecture.zh.md
- docs/advanced/context-management.md → docs/advanced/context-management.zh.md
- docs/advanced/loop-detection.md → docs/advanced/loop-detection.md
- docs/advanced/monitoring.md → docs/advanced/monitoring.zh.md
- docs/advanced/production-deployment.md → docs/advanced/production-deployment.zh.md
- docs/advanced/security.md → docs/advanced/security.zh.md

### docs/api/ 目录文件
- docs/api/agents.md → docs/api/agents.zh.md
- docs/api/cli.md → docs/api/cli.zh.md
- docs/api/config.md → docs/api/config.zh.md
- docs/api/metrics.md → docs/api/metrics.zh.md
- docs/api/orchestrator.md → docs/api/orchestrator.zh.md

### docs/deployment/ 目录文件
- docs/deployment/ci-cd.md → docs/deployment/ci-cd.zh.md
- docs/deployment/docker.md → docs/deployment/docker.zh.md
- docs/deployment/kubernetes.md → docs/deployment/kubernetes.zh.md
- docs/deployment/production.md → docs/deployment/production.zh.md
- docs/deployment/qchat-production.md → docs/deployment/qchat-production.zh.md

### docs/examples/ 目录文件
- docs/examples/bug-fix.md → docs/examples/bug-fix.zh.md
- docs/examples/cli-tool.md → docs/examples/cli-tool.zh.md
- docs/examples/data-analysis.md → docs/examples/data-analysis.zh.md
- docs/examples/documentation.md → docs/examples/documentation.zh.md
- docs/examples/index.md → docs/examples/index.zh.md
- docs/examples/simple-task.md → docs/examples/simple-task.zh.md
- docs/examples/testing.md → docs/examples/testing.zh.md
- docs/examples/web-api.md → docs/examples/web-api.zh.md

### docs/guide/ 目录文件
- docs/guide/agents.md → docs/guide/agents.zh.md
- docs/guide/checkpointing.md → docs/guide/checkpointing.zh.md
- docs/guide/configuration.md → docs/guide/configuration.zh.md
- docs/guide/cost-management.md → docs/guide/cost-management.zh.md
- docs/guide/overview.md → docs/guide/overview.zh.md
- docs/guide/prompts.md → docs/guide/prompts.zh.md
- docs/guide/web-monitoring-complete.md → docs/guide/web-monitoring-complete.zh.md
- docs/guide/web-monitoring.md → docs/guide/web-monitoring.zh.md
- docs/guide/web-quickstart.md → docs/guide/web-quickstart.zh.md
- docs/guide/websearch.md → docs/guide/websearch.zh.md

### examples/ 目录文件
- examples/cli_tool.md → examples/cli_tool.zh.md
- examples/simple_function.md → examples/simple_function.zh.md
- examples/simple-task.md → examples/simple-task.zh.md
- examples/web_scraper.md → examples/web_scraper.zh.md
- examples/web-api.md → examples/web-api.zh.md

### prompts/ 目录文件
- prompts/WEB_PROMPT.md → prompts/WEB_PROMPT.zh.md

## 翻译执行顺序
按照从重要到次要的顺序进行翻译：

1. **核心文档**（最高优先级）
   - README.md - 项目主文档
   - docs/index.md - 文档首页
   - docs/quick-start.md - 快速开始指南
   - docs/installation.md - 安装指南

2. **用户指南**（高优先级）
   - docs/guide/ 目录下的所有文件
   - docs/examples/ 目录下的所有文件

3. **API文档**（中等优先级）
   - docs/api/ 目录下的所有文件

4. **高级主题**（中等优先级）
   - docs/advanced/ 目录下的所有文件
   - docs/deployment/ 目录下的所有文件

5. **其他文档**（低优先级）
   - 剩余的杂项文档

## 翻译质量标准
- 保持技术术语的准确性
- 保留代码块和命令不变
- 保持文档结构和格式一致
- 使用标准的中文技术表达
- 适当保留英文术语当没有合适中文翻译时

## 完成标准
- 所有列出的markdown文件都有对应的中文版本
- 中文文件使用.zh.md后缀
- 保留所有原始英文文件不变
- 翻译内容准确、通顺、符合技术文档规范