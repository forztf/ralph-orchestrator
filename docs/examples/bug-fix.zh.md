# Bug 修复示例

## 概述

本指南演示如何使用 Ralph Orchestrator 进行代码调试和问题修复。

## 示例 1: 空指针异常

### 问题代码
```python
def calculate_average(numbers):
    total = sum(numbers)
    return total / len(numbers)

# 这会崩溃并报错: TypeError: unsupported operand type(s)
result = calculate_average(None)
```

### 修复请求
```python
prompt = """
修复这个函数在传入 None 时崩溃的问题:

def calculate_average(numbers):
    total = sum(numbers)
    return total / len(numbers)

函数应该优雅地处理 None、空列表和非数值的情况。
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 修复后的代码
```python
def calculate_average(numbers):
    """
    计算数字列表的平均值。

    Args:
        numbers: 数值列表或 None

    Returns:
        float: 平均值,如果输入无效则返回 None
    """
    # 处理 None 输入
    if numbers is None:
        return None

    # 处理空列表
    if not numbers:
        return 0

    # 过滤掉非数值的项
    numeric_values = []
    for num in numbers:
        try:
            numeric_values.append(float(num))
        except (TypeError, ValueError):
            continue

    # 如果没有有效数字则返回 None
    if not numeric_values:
        return None

    return sum(numeric_values) / len(numeric_values)
```

## 示例 2: 内存泄漏修复

### 问题代码
```javascript
class EventManager {
    constructor() {
        this.events = {};
    }

    on(event, callback) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(callback);
    }

    emit(event, data) {
        if (this.events[event]) {
            this.events[event].forEach(cb => cb(data));
        }
    }
}
```

### 修复请求
```python
prompt = """
修复这个 JavaScript EventManager 类中的内存泄漏。
该类从不移除事件监听器,导致内存泄漏。
添加一个 'off' 方法并确保正确清理。
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 修复后的代码
```javascript
class EventManager {
    constructor() {
        this.events = {};
        this.eventIds = new WeakMap();
        this.nextId = 0;
    }

    on(event, callback) {
        if (!this.events[event]) {
            this.events[event] = [];
        }

        // 存储回调函数并带上唯一 ID 以便移除
        const id = this.nextId++;
        const callbackWrapper = { id, callback };
        this.events[event].push(callbackWrapper);
        this.eventIds.set(callback, id);

        // 返回取消订阅函数
        return () => this.off(event, callback);
    }

    off(event, callback) {
        if (!this.events[event]) return;

        const id = this.eventIds.get(callback);
        if (id !== undefined) {
            this.events[event] = this.events[event].filter(
                wrapper => wrapper.id !== id
            );
            this.eventIds.delete(callback);

            // 清理空的事件数组
            if (this.events[event].length === 0) {
                delete this.events[event];
            }
        }
    }

    emit(event, data) {
        if (this.events[event]) {
            this.events[event].forEach(wrapper =>
                wrapper.callback(data)
            );
        }
    }

    // 清除某个事件的所有监听器
    removeAllListeners(event) {
        if (event) {
            delete this.events[event];
        } else {
            this.events = {};
        }
    }

    // 销毁实例的清理方法
    destroy() {
        this.events = {};
        this.eventIds = new WeakMap();
    }
}
```

## 示例 3: 竞态条件修复

### 问题代码
```python
import threading

class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        temp = self.count
        # 模拟一些处理
        temp = temp + 1
        self.count = temp
```

### 修复请求
```python
prompt = """
修复这个 Counter 类中的竞态条件问题,
当多个线程同时调用 increment() 时会出现该问题。
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 修复后的代码
```python
import threading
from contextlib import contextmanager

class Counter:
    def __init__(self):
        self.count = 0
        self._lock = threading.Lock()

    def increment(self):
        """线程安全的递增操作。"""
        with self._lock:
            temp = self.count
            # 模拟一些处理
            temp = temp + 1
            self.count = temp

    def decrement(self):
        """线程安全的递减操作。"""
        with self._lock:
            self.count -= 1

    def get_value(self):
        """线程安全的读取操作。"""
        with self._lock:
            return self.count

    @contextmanager
    def batch_operation(self):
        """用于批量操作的上下文管理器。"""
        self._lock.acquire()
        try:
            yield self
        finally:
            self._lock.release()

# 使用示例:
counter = Counter()

# 批量操作
with counter.batch_operation():
    for _ in range(100):
        counter.count += 1  # 在上下文中直接访问
```

## 示例 4: SQL 注入漏洞

### 问题代码
```python
def get_user(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"
    return db.execute(query)
```

### 修复请求
```python
prompt = """
修复这个函数中的 SQL 注入漏洞。
使用参数化查询并添加输入验证。
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 修复后的代码
```python
def get_user(user_id):
    """
    安全地通过 ID 获取用户,防止 SQL 注入。

    Args:
        user_id: 用户标识符(int 或 string)

    Returns:
        dict: 用户数据,如果未找到则返回 None

    Raises:
        ValueError: 如果 user_id 无效
    """
    # 输入验证
    if user_id is None:
        raise ValueError("User ID cannot be None")

    try:
        # 转换为整数以确保类型安全
        user_id = int(user_id)
    except (ValueError, TypeError):
        raise ValueError(f"Invalid user ID: {user_id}")

    # 额外验证
    if user_id <= 0:
        raise ValueError("User ID must be positive")

    # 使用参数化查询防止 SQL 注入
    query = "SELECT * FROM users WHERE id = %s"

    try:
        result = db.execute(query, (user_id,))
        return result.fetchone()
    except DatabaseError as e:
        logger.error(f"Database error fetching user {user_id}: {e}")
        return None

# 使用 ORM 的额外安全版本
def get_user_orm(user_id):
    """基于 ORM 的用户检索(推荐)。"""
    try:
        user_id = int(user_id)
        return User.query.filter_by(id=user_id).first()
    except (ValueError, TypeError):
        return None
```

## 示例 5: 无限循环修复

### 问题代码
```python
def find_target(arr, target):
    left = 0
    right = len(arr) - 1

    while left < right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid  # Bug: 应该是 mid + 1
        else:
            right = mid

    return -1
```

### 修复请求
```python
prompt = """
修复这个二分查找实现中的无限循环 bug。
当目标值不在数组中时,函数会卡住。
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 修复后的代码
```python
def find_target(arr, target):
    """
    修复了无限循环问题的二分查找实现。

    Args:
        arr: 要搜索的已排序数组
        target: 要查找的值

    Returns:
        int: 目标的索引,如果未找到则返回 -1
    """
    if not arr:
        return -1

    left = 0
    right = len(arr) - 1

    while left <= right:  # 修改条件以包含相等
        mid = (left + right) // 2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1  # 修复: 递增以避免无限循环
        else:
            right = mid - 1  # 修复: 递减以保持一致性

    return -1

# 带有额外功能的增强版本
def find_target_enhanced(arr, target, return_insertion_point=False):
    """
    带有插入点选项的增强二分查找。

    Args:
        arr: 要搜索的已排序数组
        target: 要查找的值
        return_insertion_point: 如果为 True,返回目标应该插入的位置

    Returns:
        int: 目标的索引,如果未找到且被请求则返回插入点
    """
    if not arr:
        return 0 if return_insertion_point else -1

    left = 0
    right = len(arr) - 1

    while left <= right:
        mid = (left + right) // 2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return left if return_insertion_point else -1
```

## 常见 Bug 模式和修复方法

### 1. 差一错误 (Off-by-One Errors)
- 检查数组边界
- 验证循环条件
- 测试边界情况(空、单个元素)

### 2. 空/未定义处理
- 在函数入口添加空值检查
- 在 JavaScript 中使用可选链
- 提供合理的默认值

### 3. 资源泄漏
- 实现正确的清理(关闭文件、连接)
- 在 Python 中使用上下文管理器
- 添加 finally 块用于清理

### 4. 并发问题
- 对共享资源使用锁
- 实现原子操作
- 考虑使用线程安全的数据结构

### 5. 类型错误
- 添加类型检查/验证
- 使用 TypeScript/类型提示
- 显式处理类型转换

## 调试技巧

1. **先重现问题**: 在修复之前总是先重现 bug
2. **添加日志**: 插入战略性日志以了解流程
3. **单元测试**: 编写暴露 bug 的测试
4. **边界情况**: 使用空、null 和边界值进行测试
5. **代码审查**: 让其他人审查修复方案

## 另请参阅

- [测试示例](./testing.zh.md)
- [Agent 指南](../guide/agents.zh.md)
- [错误处理最佳实践](../03-best-practices/best-practices.zh.md)
