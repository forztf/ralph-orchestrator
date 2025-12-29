# 安装指南

Ralph Orchestrator 的综合安装说明。

## 系统要求

### 最低要求

- **Python**：3.8 或更高版本
- **内存**：512 MB RAM
- **磁盘**：100 MB 可用空间
- **操作系统**：Linux、macOS 或 Windows

### 推荐要求

- **Python**：3.10 或更高版本
- **内存**：2 GB RAM
- **磁盘**：1 GB 可用空间
- **Git**：用于检查点功能
- **网络**：稳定的互联网连接

## 安装方法

### 方法 1：Git 克隆（推荐）

```bash
# 克隆仓库
git clone https://github.com/mikeyobrien/ralph-orchestrator.git
cd ralph-orchestrator

# 使编排器可执行
chmod +x ralph_orchestrator.py
chmod +x ralph

# 安装可选依赖项
pip install psutil  # 用于系统指标
```

### 方法 2：直接下载

```bash
# 下载最新版本
wget https://github.com/mikeyobrien/ralph-orchestrator/archive/refs/tags/v1.0.0.tar.gz

# 解压存档
tar -xzf v1.0.0.tar.gz
cd ralph-orchestrator-1.0.0

# 使其可执行
chmod +x ralph_orchestrator.py
```

### 方法 3：pip 安装（即将推出）

```bash
# 未来通过 pip 安装
pip install ralph-orchestrator
```

## AI 代理安装

Ralph 需要至少一个 AI 代理才能运行。选择一个或多个安装：

### Claude (Anthropic)

Claude 是大多数用例的推荐代理。

```bash
# 通过 npm 安装
npm install -g @anthropic-ai/claude-code

# 或从下载
# https://claude.ai/code

# 验证安装
claude --version
```

**配置：**
```bash
# 设置您的 API 密钥（如果需要）
export ANTHROPIC_API_KEY="your-api-key-here"
```

### Q Chat

Q Chat 是一个轻量级的替代代理。

```bash
# 通过 pip 安装
pip install q-cli

# 或从仓库克隆
git clone https://github.com/qchat/qchat.git
cd qchat
python setup.py install

# 验证安装
q --version
```

**配置：**
```bash
# 配置 Q Chat
q config --set api_key="your-api-key"
```

### Gemini (Google)

Gemini 提供对 Google AI 模型的访问。

```bash
# 通过 npm 安装
npm install -g @google/gemini-cli

# 验证安装
gemini --version
```

**配置：**
```bash
# 设置您的 API 密钥
export GEMINI_API_KEY="your-api-key-here"

# 或使用配置文件
gemini config set api_key "your-api-key"
```

## 依赖项安装

### 必需的 Python 包

Ralph Orchestrator 的依赖项很少，但某些功能需要额外的包：

```bash
# 核心功能（不需要额外的包）
# Ralph 仅使用 Python 标准库实现核心功能

# 可选：系统指标监控
pip install psutil

# 可选：增强的 JSON 处理
pip install orjson  # 更快的 JSON 处理

# 可选：开发依赖项
pip install pytest pytest-cov black ruff
```

### 使用 requirements.txt

如果您想安装所有可选依赖项：

```bash
# 创建 requirements.txt
cat > requirements.txt << EOF
psutil>=5.9.0
orjson>=3.9.0
pytest>=7.0.0
pytest-cov>=4.0.0
black>=23.0.0
ruff>=0.1.0
EOF

# 安装所有依赖项
pip install -r requirements.txt
```

### 使用 uv（推荐用于开发）

```bash
# 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 使用 uv 安装依赖项
uv pip install psutil orjson

# 或使用 pyproject.toml
uv sync
```

## 验证

### 验证安装

运行这些命令以验证您的安装：

```bash
# 检查 Python 版本
python --version  # 应该是 3.8+

# 检查 Ralph Orchestrator
python ralph_orchestrator.py --version

# 检查可用代理
python ralph_orchestrator.py --list-agents

# 运行测试
echo "说你好（编排器将迭代直到完成）" > test.md
python ralph_orchestrator.py --prompt test.md --dry-run
```

### 预期输出

```
Ralph Orchestrator v1.0.0
Python 3.10.12
可用代理：claude, q, gemini
试运行成功完成
```

## 平台特定说明

### Linux

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3 python3-pip git

# Fedora/RHEL
sudo dnf install python3 python3-pip git

# Arch Linux
sudo pacman -S python python-pip git
```

### macOS

```bash
# 使用 Homebrew
brew install python git

# 使用 MacPorts
sudo port install python310 git

# 验证 Python 安装
python3 --version
```

### Windows

```powershell
# 使用 PowerShell 作为管理员

# 从 Microsoft Store 安装 Python
winget install Python.Python.3.11

# 或从 python.org 下载
# https://www.python.org/downloads/windows/

# 安装 Git
winget install Git.Git

# 克隆 Ralph
git clone https://github.com/mikeyobrien/ralph-orchestrator.git
cd ralph-orchestrator

# 运行 Ralph
python ralph_orchestrator.py --prompt PROMPT.md
```

### Docker（替代方案）

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY . /app

RUN pip install psutil

# 安装您首选的 AI 代理
RUN npm install -g @anthropic-ai/claude-code

CMD ["python", "ralph_orchestrator.py"]
```

```bash
# 构建并运行
docker build -t ralph-orchestrator .
docker run -v $(pwd):/app ralph-orchestrator --prompt PROMPT.md
```

## 配置文件

### 基本配置

创建默认设置的配置文件：

```bash
# 创建 .ralph.conf
cat > .ralph.conf << EOF
# Ralph 默认配置
agent=claude
max_iterations=100
max_runtime=14400
checkpoint_interval=5
verbose=false
EOF
```

### 环境变量

设置常用设置的环境变量：

```bash
# 添加到您的 ~/.bashrc 或 ~/.zshrc
export RALPH_AGENT="claude"
export RALPH_MAX_ITERATIONS="100"
export RALPH_MAX_COST="50.0"
export RALPH_VERBOSE="false"
```

## 安装故障排除

### 常见问题

#### Python 版本太旧

```bash
错误：需要 Python 3.8+，找到 3.7.3
```

**解决方案**：升级 Python
```bash
# Ubuntu/Debian
sudo apt install python3.10

# macOS
brew upgrade python

# Windows
winget upgrade Python.Python.3.11
```

#### 找不到代理

```bash
错误：未检测到 AI 代理
```

**解决方案**：安装至少一个代理
```bash
npm install -g @anthropic-ai/claude-code
# 或
pip install q-cli
```

#### 权限被拒绝

```bash
权限被拒绝：'./ralph_orchestrator.py'
```

**解决方案**：使其可执行
```bash
chmod +x ralph_orchestrator.py
chmod +x ralph
```

#### 找不到模块

```bash
ModuleNotFoundError: 找不到名为 'psutil' 的模块
```

**解决方案**：安装可选依赖项
```bash
pip install psutil
```

## 卸载

要删除 Ralph Orchestrator：

```bash
# 删除目录
rm -rf ralph-orchestrator

# 卸载可选依赖项
pip uninstall psutil orjson

# 删除配置文件
rm ~/.ralph.conf
```

## 后续步骤

安装后：

1. 阅读[快速开始指南](quick-start.md)
2. 配置您的 [AI 代理](guide/agents.md)
3. 了解[配置选项](guide/configuration.md)
4. 尝试[示例](examples/index.md)

## 获取帮助

如果遇到问题：

- 查看[常见问题](faq.md)
- 阅读[故障排除](troubleshooting.md)
- 在 [GitHub](https://github.com/mikeyobrien/ralph-orchestrator/issues) 上打开问题
- 加入[讨论](https://github.com/mikeyobrien/ralph-orchestrator/discussions)

---

📚 继续到[用户指南](guide/overview.md) →