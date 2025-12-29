# 任务:构建 REST API

创建一个具有用户管理功能的 Flask REST API。

## 要求

- [ ] Flask 应用程序结构
- [ ] 用户模型,包含 id、name、email 字段
- [ ] CRUD 端点:
  - GET /users - 列出所有用户
  - GET /users/<id> - 获取特定用户
  - POST /users - 创建新用户
  - PUT /users/<id> - 更新用户
  - DELETE /users/<id> - 删除用户
- [ ] 输入验证
- [ ] 错误处理
- [ ] 单元测试
- [ ] 文档

## 技术规格

- 使用 Flask 2.0+
- SQLite 用于数据存储
- 返回 JSON 响应
- 正确的 HTTP 状态代码
- RESTful 设计原则

## 成功标准

- 所有端点功能正常
- 测试通过
- 处理边缘情况
- 清晰的错误消息

<!-- 编排器将继续迭代,直到满足所有要求 -->
