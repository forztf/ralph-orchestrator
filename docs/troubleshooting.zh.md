# 故障排除指南

## 常见问题与解决方案

### 安装问题

#### 未找到代理

**问题**: `ralph: command 'claude' not found`

**解决方案**:

1. 验证代理安装:

   ```bash
   which claude
   which gemini
   which q
   ```

2. 安装缺失的代理:

   ```bash
   # Claude
   npm install -g @anthropic-ai/claude-code

   # Gemini
   npm install -g @google/gemini-cli
   ```

3. 添加到 PATH:

   ```bash
   export PATH=$PATH:/usr/local/bin
   ```

#### 权限被拒绝

**问题**: `Permission denied: './ralph'`

**解决方案**:

```bash
chmod +x ralph ralph_orchestrator.py
```

### 执行问题

#### 任务运行时间过长

**问题**: Ralph 运行达到最大迭代次数仍未实现目标

**可能原因**:

1. 任务描述不清晰或过于复杂
2. 代理未向目标取得进展
3. 任务范围对于迭代限制来说太大

**解决方案**:

1. 检查迭代进度和日志:

   ```bash
   ralph status
   ```

2. 分解复杂任务:

   ```markdown
   # 不要这样:

   构建一个完整的 Web 应用程序

   # 而是这样:

   创建一个 Flask 应用,包含一个返回 "Hello World" 的端点
   ```

3. 增加迭代限制或尝试不同的代理:

   ```bash
   ralph run --max-iterations 200
   ralph run --agent gemini
   ```

#### 代理超时

**问题**: `Agent execution timed out`

**解决方案**:

1. 增加超时时间:

   ```python
   # 在 ralph.json 中
   {
     "timeout_per_iteration": 600
   }
   ```

2. 降低提示词复杂性:
   - 将大任务分解为小任务
   - 移除不必要的上下文

3. 检查系统资源:

   ```bash
   htop
   free -h
   ```

#### 重复错误

**问题**: 多次迭代中出现相同错误

**解决方案**:

1. 检查错误模式:

   ```bash
   cat .agent/metrics/state_*.json | jq '.errors'
   ```

2. 清除工作区并重试:

   ```bash
   ralph clean
   ralph run
   ```

3. 手动干预:
   - 修复具体问题
   - 向 PROMPT.md 添加说明
   - 恢复执行

#### 循环检测问题

**问题**: `Loop detected: XX% similarity to previous output`

当代理输出与最近 5 次输出中的任何一次相似度≥90%时,会触发 Ralph 的循环检测。

**可能原因**:

1. 代理卡在同一子任务上
2. 代理产生类似的"正在处理"消息
3. API 错误导致相同的重试消息
4. 任务需要重复执行相同的操作(误报)

**解决方案**:

1. **检查是否是真正的循环**:

   ```bash
   # 查看最近的输出
   ls -lt .agent/prompts/ | head -10
   diff .agent/prompts/prompt_N.md .agent/prompts/prompt_N-1.md
   ```

2. **改进提示词以鼓励多样性**:

   ```markdown
   # 添加明确的进度跟踪

   ## 当前状态

   记录你在哪个步骤,以及自上次迭代以来有什么变化
   ```

3. **分解任务**:
   - 如果代理一直做同样的事情,可能需要重构任务
   - 拆分为更小、更独特的子任务

4. **检查潜在问题**:
   - API 错误导致重试
   - 权限问题阻止进展
   - 缺少依赖项

#### 未检测到完成标记

**问题**: Ralph 继续运行尽管有 `TASK_COMPLETE` 标记

**可能原因**:

1. 标记格式不正确
2. 不可见字符或编码问题
3. 标记埋在代码块中

**解决方案**:

1. **使用精确格式**:

   ```markdown
   # 正确格式:

   - [x] TASK_COMPLETE
         [x] TASK_COMPLETE

   # 不正确(不会触发):

   - [ ] TASK_COMPLETE # 未勾选
         TASK_COMPLETE # 无复选框
   - [X] TASK_COMPLETE # 大写 X
   ```

2. **检查隐藏字符**:

   ```bash
   cat -A PROMPT.md | grep TASK_COMPLETE
   ```

3. **确保标记独占一行**:

   ````markdown
   # 好 - 独占一行

   - [x] TASK_COMPLETE

   # 坏 - 在代码块内

   ```markdown
   - [x] TASK_COMPLETE # 在代码块内 - 不会工作
   ```
   ````

   ```

   ```

4. **验证编码**:

   ```bash
   file PROMPT.md
   # 应显示: UTF-8 Unicode text
   ```

### Git 问题

#### 检查点失败

**问题**: `Failed to create checkpoint`

**解决方案**:

1. 初始化 Git 仓库:

   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   ```

2. 检查 Git 状态:

   ```bash
   git status
   ```

3. 修复 Git 配置:

   ```bash
   git config user.email "you@example.com"
   git config user.name "Your Name"
   ```

#### 未提交的更改警告

**问题**: `Uncommitted changes detected`

**解决方案**:

1. 提交更改:

   ```bash
   git add .
   git commit -m "Save work"
   ```

2. 暂存更改:

   ```bash
   git stash
   ralph run
   git stash pop
   ```

3. 禁用 Git 操作:

   ```bash
   ralph run --no-git
   ```

### 上下文问题

#### 上下文窗口超出

**问题**: `Context window limit exceeded`

**症状**:

- 代理忘记之前的指令
- 响应不完整
- 关于缺少信息的错误

**解决方案**:

1. 减小文件大小:

   ```bash
   # 拆分大文件
   split -l 500 large_file.py part_
   ```

2. 使用更简洁的提示词:

   ```markdown
   # 移除不必要的细节

   # 专注于当前任务
   ```

3. 切换到高上下文代理:

   ```bash
   # Claude 有 200K 上下文
   ralph run --agent claude
   ```

4. 清除迭代历史:

   ```bash
   rm .agent/prompts/prompt_*.md
   ```

### 性能问题

#### 执行缓慢

**问题**: 迭代耗时太长

**解决方案**:

1. 检查系统资源:

   ```bash
   top
   df -h
   iostat
   ```

2. 减少并行操作:
   - 关闭其他应用程序
   - 限制后台进程

3. 使用更快的代理:

   ```bash
   # Q 通常更快
   ralph run --agent q
   ```

#### 高内存使用

**问题**: Ralph 消耗过多内存

**解决方案**:

1. 设置资源限制:

   ```python
   # 在 ralph.json 中
   {
     "resource_limits": {
       "memory_mb": 2048
     }
   }
   ```

2. 清理旧状态文件:

   ```bash
   find .agent -name "*.json" -mtime +7 -delete
   ```

3. 重启 Ralph:

   ```bash
   pkill -f ralph_orchestrator
   ralph run
   ```

### 状态和指标问题

#### 状态文件损坏

**问题**: `Invalid state file`

**解决方案**:

1. 删除损坏的文件:

   ```bash
   rm .agent/metrics/state_latest.json
   ```

2. 从备份恢复:

   ```bash
   cp .agent/metrics/state_*.json .agent/metrics/state_latest.json
   ```

3. 重置状态:

   ```bash
   ralph clean
   ```

#### 缺少指标

**问题**: 未收集指标

**解决方案**:

1. 检查指标目录:

   ```bash
   ls -la .agent/metrics/
   ```

2. 如果缺失则创建目录:

   ```bash
   mkdir -p .agent/metrics
   ```

3. 检查权限:

   ```bash
   chmod 755 .agent/metrics
   ```

## 错误消息

### 常见错误代码

| 错误            | 含义              | 解决方案              |
| --------------- | ----------------- | --------------------- |
| `Exit code 1`   | 一般失败          | 检查日志了解详情      |
| `Exit code 130` | 被中断 (Ctrl+C)   | 正常中断              |
| `Exit code 137` | 被终止 (内存不足) | 增加内存限制          |
| `Exit code 124` | 超时              | 增加超时值            |

### 代理特定错误

#### Claude 错误

```
"Rate limit exceeded"
```

**解决方案**: 在迭代之间添加延迟或升级 API 计划

```
"Invalid API key"
```

**解决方案**: 检查 Claude CLI 配置

#### Gemini 错误

```
"Quota exceeded"
```

**解决方案**: 等待配额重置或升级计划

```
"Model not available"
```

**解决方案**: 检查 Gemini CLI 版本并更新

#### Q Chat 错误

```
"Connection refused"
```

**解决方案**: 确保 Q 服务正在运行

## 调试模式

### 启用详细日志

```bash
# 最大详细度
ralph run --verbose

# 使用调试环境
DEBUG=1 ralph run

# 保存日志
ralph run --verbose 2>&1 | tee debug.log
```

### 检查执行

```python
# 在 PROMPT.md 中添加调试点
print("DEBUG: 已到达检查点 1")
```

### 跟踪执行

```bash
# 跟踪系统调用
strace -o trace.log ralph run

# 分析 Python 执行
python -m cProfile ralph_orchestrator.py
```

## 恢复程序

### 从失败状态恢复

1. **保存当前状态**:

   ```bash
   cp -r .agent .agent.backup
   ```

2. **分析失败**:

   ```bash
   tail -n 100 .agent/logs/ralph.log
   ```

3. **修复问题**:
   - 更新 PROMPT.md
   - 修复代码错误
   - 清除有问题的文件

4. **恢复或重启**:

   ```bash
   # 从检查点恢复
   ralph run

   # 或从头开始
   ralph clean && ralph run
   ```

### 从 Git 检查点恢复

```bash
# 列出检查点
git log --oneline | grep checkpoint

# 重置到检查点
git reset --hard <commit-hash>

# 恢复执行
ralph run
```

## 获取帮助

### 自我诊断

运行诊断脚本:

```bash
cat > diagnose.sh << 'EOF'
#!/bin/bash
echo "Ralph Orchestrator 诊断"
echo "============================"
echo "可用的代理:"
which claude && echo "  ✓ Claude" || echo "  ✗ Claude"
which gemini && echo "  ✓ Gemini" || echo "  ✗ Gemini"
which q && echo "  ✓ Q" || echo "  ✗ Q"
echo ""
echo "Git 状态:"
git status --short
echo ""
echo "Ralph 状态:"
./ralph status
echo ""
echo "最近的错误:"
grep ERROR .agent/logs/*.log 2>/dev/null | tail -5
EOF
chmod +x diagnose.sh
./diagnose.sh
```

### 社区支持

1. **GitHub Issues**: [报告 bug](https://github.com/mikeyobrien/ralph-orchestrator/issues)
2. **Discussions**: [提问](https://github.com/mikeyobrien/ralph-orchestrator/discussions)
3. **Discord**: 加入社区聊天

### 报告 Bug

在 bug 报告中包含:

1. Ralph 版本: `ralph --version`
2. 代理版本
3. 错误消息
4. PROMPT.md 内容
5. 诊断输出
6. 重现步骤

## 预防提示

### 最佳实践

1. **从简单开始**: 先用基本任务测试
2. **定期检查点**: 使用默认的 5 次迭代间隔
3. **监控进度**: 经常检查状态
4. **版本控制**: 在运行 Ralph 之前提交
5. **资源限制**: 设置适当的限制
6. **明确要求**: 编写具体的、可测试的标准

### 预检查清单

在运行 Ralph 之前:

- [ ] PROMPT.md 清晰且具体
- [ ] Git 仓库干净
- [ ] 代理已安装且正常工作
- [ ] 有足够的磁盘空间
- [ ] 提示词中无敏感数据
- [ ] 备份重要文件
