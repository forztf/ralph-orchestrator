# 测试示例

## 概述

本指南演示如何使用 Ralph Orchestrator 生成全面的测试套件。

## 示例 1：单元测试生成

### 源代码
```python
class Calculator:
    def add(self, a, b):
        return a + b

    def divide(self, a, b):
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b

    def calculate_percentage(self, value, percentage):
        return (value * percentage) / 100
```

### 测试生成请求
```python
prompt = """
使用 pytest 为这个 Calculator 类生成全面的单元测试。
包括边缘情况、错误处理和参数化测试。
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 生成的测试
```python
import pytest
from calculator import Calculator

class TestCalculator:
    """Calculator 类的全面测试套件。"""

    @pytest.fixture
    def calc(self):
        """为测试提供 Calculator 实例。"""
        return Calculator()

    # 加法测试
    @pytest.mark.parametrize("a,b,expected", [
        (2, 3, 5),
        (-1, 1, 0),
        (0, 0, 0),
        (1.5, 2.5, 4.0),
        (-5, -3, -8),
        (1e10, 1e10, 2e10),  # 大数字
    ])
    def test_add(self, calc, a, b, expected):
        """测试各种输入的加法。"""
        assert calc.add(a, b) == expected

    # 除法测试
    @pytest.mark.parametrize("a,b,expected", [
        (10, 2, 5),
        (7, 2, 3.5),
        (-10, 2, -5),
        (0, 5, 0),
        (1, 3, 0.3333333333333333),
    ])
    def test_divide(self, calc, a, b, expected):
        """测试有效输入的除法。"""
        assert calc.divide(a, b) == pytest.approx(expected)

    def test_divide_by_zero(self, calc):
        """测试除以零会引发 ValueError。"""
        with pytest.raises(ValueError, match="Cannot divide by zero"):
            calc.divide(10, 0)

    # 百分比测试
    @pytest.mark.parametrize("value,percentage,expected", [
        (100, 10, 10),
        (50, 50, 25),
        (200, 25, 50),
        (0, 50, 0),
        (100, 0, 0),
        (100, 150, 150),  # 超过 100%
        (100, -10, -10),   # 负百分比
    ])
    def test_calculate_percentage(self, calc, value, percentage, expected):
        """测试各种输入的百分比计算。"""
        assert calc.calculate_percentage(value, percentage) == expected

    # 边缘情况
    def test_add_with_none(self, calc):
        """测试添加 None 会引发 TypeError。"""
        with pytest.raises(TypeError):
            calc.add(None, 5)

    def test_float_precision(self, calc):
        """测试浮点精度处理。"""
        result = calc.add(0.1, 0.2)
        assert result == pytest.approx(0.3)

    # 基于属性的测试（使用 hypothesis）
    @pytest.mark.skipif(not pytest.importorskip("hypothesis"),
                        reason="hypothesis 未安装")
    def test_add_commutative(self, calc):
        """测试加法满足交换律。"""
        from hypothesis import given, strategies as st

        @given(st.floats(allow_nan=False), st.floats(allow_nan=False))
        def check_commutative(a, b):
            assert calc.add(a, b) == calc.add(b, a)

        check_commutative()
```

## 示例 2：集成测试生成

### API 代码
```python
class UserService:
    def __init__(self, db, email_service):
        self.db = db
        self.email_service = email_service

    def create_user(self, email, name):
        if self.db.user_exists(email):
            raise ValueError("User already exists")

        user = self.db.create_user(email=email, name=name)
        self.email_service.send_welcome_email(email, name)
        return user
```

### 测试生成请求
```python
prompt = """
为 UserService 生成集成测试，测试数据库和电子邮件服务之间的交互。
在适当的地方使用 mock。
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 生成的集成测试
```python
import pytest
from unittest.mock import Mock, MagicMock, call
from user_service import UserService

class TestUserServiceIntegration:
    """UserService 的集成测试。"""

    @pytest.fixture
    def mock_db(self):
        """创建模拟数据库。"""
        db = Mock()
        db.user_exists.return_value = False
        db.create_user.return_value = {
            "id": 1,
            "email": "test@example.com",
            "name": "Test User"
        }
        return db

    @pytest.fixture
    def mock_email_service(self):
        """创建模拟电子邮件服务。"""
        return Mock()

    @pytest.fixture
    def user_service(self, mock_db, mock_email_service):
        """创建带有模拟依赖项的 UserService。"""
        return UserService(mock_db, mock_email_service)

    def test_create_user_success(self, user_service, mock_db, mock_email_service):
        """测试成功的用户创建流程。"""
        # Arrange
        email = "newuser@example.com"
        name = "New User"

        # Act
        user = user_service.create_user(email, name)

        # Assert
        mock_db.user_exists.assert_called_once_with(email)
        mock_db.create_user.assert_called_once_with(email=email, name=name)
        mock_email_service.send_welcome_email.assert_called_once_with(email, name)
        assert user["email"] == "test@example.com"

    def test_create_user_already_exists(self, user_service, mock_db):
        """测试用户已存在时的用户创建。"""
        # Arrange
        mock_db.user_exists.return_value = True

        # Act & Assert
        with pytest.raises(ValueError, match="User already exists"):
            user_service.create_user("existing@example.com", "Existing User")

        # 验证未发送电子邮件
        mock_db.create_user.assert_not_called()

    def test_create_user_email_failure(self, user_service, mock_email_service):
        """测试电子邮件服务失败的处理。"""
        # Arrange
        mock_email_service.send_welcome_email.side_effect = Exception("Email failed")

        # Act & Assert
        with pytest.raises(Exception, match="Email failed"):
            user_service.create_user("user@example.com", "User")

    def test_create_multiple_users(self, user_service, mock_db, mock_email_service):
        """测试按顺序创建多个用户。"""
        users = [
            ("user1@example.com", "User One"),
            ("user2@example.com", "User Two"),
            ("user3@example.com", "User Three"),
        ]

        for email, name in users:
            user_service.create_user(email, name)

        # 验证所有调用都已进行
        assert mock_db.user_exists.call_count == 3
        assert mock_db.create_user.call_count == 3
        assert mock_email_service.send_welcome_email.call_count == 3

        # 验证调用顺序
        email_calls = [call(email, name) for email, name in users]
        mock_email_service.send_welcome_email.assert_has_calls(email_calls)
```

## 示例 3：端到端测试生成

### Web 应用程序路由
```python
@app.route('/api/login', methods=['POST'])
def login():
    data = request.json
    username = data.get('username')
    password = data.get('password')

    if not username or not password:
        return jsonify({"error": "Missing credentials"}), 400

    user = authenticate_user(username, password)
    if not user:
        return jsonify({"error": "Invalid credentials"}), 401

    token = generate_token(user)
    return jsonify({"token": token}), 200
```

### 测试生成请求
```python
prompt = """
使用 pytest 和 requests 为这个登录端点生成端到端测试。
测试成功登录、缺少字段和无效凭据。
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 生成的 E2E 测试
```python
import pytest
import requests
import json
from unittest.mock import patch

class TestLoginEndToEnd:
    """登录端点的端到端测试。"""

    @pytest.fixture
    def base_url(self):
        """API 的基础 URL。"""
        return "http://localhost:5000"

    @pytest.fixture
    def valid_credentials(self):
        """有效的用户凭据。"""
        return {
            "username": "testuser",
            "password": "testpass123"
        }

    def test_successful_login(self, base_url, valid_credentials):
        """测试成功登录返回令牌。"""
        response = requests.post(
            f"{base_url}/api/login",
            json=valid_credentials,
            headers={"Content-Type": "application/json"}
        )

        assert response.status_code == 200
        data = response.json()
        assert "token" in data
        assert len(data["token"]) > 0

    def test_missing_username(self, base_url):
        """测试缺少用户名的登录。"""
        response = requests.post(
            f"{base_url}/api/login",
            json={"password": "testpass123"},
            headers={"Content-Type": "application/json"}
        )

        assert response.status_code == 400
        data = response.json()
        assert data["error"] == "Missing credentials"

    def test_missing_password(self, base_url):
        """测试缺少密码的登录。"""
        response = requests.post(
            f"{base_url}/api/login",
            json={"username": "testuser"},
            headers={"Content-Type": "application/json"}
        )

        assert response.status_code == 400
        data = response.json()
        assert data["error"] == "Missing credentials"

    def test_empty_request_body(self, base_url):
        """测试空请求主体的登录。"""
        response = requests.post(
            f"{base_url}/api/login",
            json={},
            headers={"Content-Type": "application/json"}
        )

        assert response.status_code == 400
        data = response.json()
        assert data["error"] == "Missing credentials"

    def test_invalid_credentials(self, base_url):
        """测试使用无效凭据登录。"""
        response = requests.post(
            f"{base_url}/api/login",
            json={
                "username": "wronguser",
                "password": "wrongpass"
            },
            headers={"Content-Type": "application/json"}
        )

        assert response.status_code == 401
        data = response.json()
        assert data["error"] == "Invalid credentials"

    @pytest.mark.parametrize("content_type", [
        "text/plain",
        "application/xml",
        None
    ])
    def test_invalid_content_type(self, base_url, valid_credentials, content_type):
        """测试无效内容类型的登录。"""
        headers = {}
        if content_type:
            headers["Content-Type"] = content_type

        response = requests.post(
            f"{base_url}/api/login",
            data=json.dumps(valid_credentials),
            headers=headers
        )

        assert response.status_code in [400, 415]

    def test_sql_injection_attempt(self, base_url):
        """测试安全处理 SQL 注入尝试。"""
        response = requests.post(
            f"{base_url}/api/login",
            json={
                "username": "admin' OR '1'='1",
                "password": "' OR '1'='1"
            },
            headers={"Content-Type": "application/json"}
        )

        assert response.status_code == 401
        data = response.json()
        assert data["error"] == "Invalid credentials"

    def test_rate_limiting(self, base_url):
        """测试强制执行速率限制。"""
        # 发起 10 次快速请求
        for _ in range(10):
            requests.post(
                f"{base_url}/api/login",
                json={"username": "test", "password": "wrong"},
                headers={"Content-Type": "application/json"}
            )

        # 第 11 次请求应被速率限制
        response = requests.post(
            f"{base_url}/api/login",
            json={"username": "test", "password": "wrong"},
            headers={"Content-Type": "application/json"}
        )

        assert response.status_code == 429  # Too Many Requests
```

## 示例 4：性能测试生成

### 测试生成请求
```python
prompt = """
为处理大型数据集的函数生成性能测试。
包括执行时间、内存使用和可扩展性的测试。
"""

response = orchestrator.execute(prompt, agent="claude")
```

### 生成的性能测试
```python
import pytest
import time
import tracemalloc
from memory_profiler import profile
import numpy as np

class TestPerformance:
    """数据处理函数的性能测试套件。"""

    @pytest.fixture
    def large_dataset(self):
        """生成用于测试的大型数据集。"""
        return np.random.rand(1000000)

    def test_execution_time(self, large_dataset):
        """测试处理在时间限制内完成。"""
        start_time = time.perf_counter()

        # 处理数据
        result = process_data(large_dataset)

        end_time = time.perf_counter()
        execution_time = end_time - start_time

        # 断言执行时间在 1 秒以下
        assert execution_time < 1.0, f"执行耗时 {execution_time:.2f}s"

    def test_memory_usage(self, large_dataset):
        """测试内存使用保持在限制范围内。"""
        tracemalloc.start()

        # 处理数据
        result = process_data(large_dataset)

        current, peak = tracemalloc.get_traced_memory()
        tracemalloc.stop()

        # 转换为 MB
        peak_mb = peak / 1024 / 1024

        # 断言内存使用在 100MB 以下
        assert peak_mb < 100, f"峰值内存使用: {peak_mb:.2f}MB"

    @pytest.mark.parametrize("size", [100, 1000, 10000, 100000])
    def test_scalability(self, size):
        """测试性能与数据大小呈线性扩展。"""
        data = np.random.rand(size)

        start_time = time.perf_counter()
        result = process_data(data)
        execution_time = time.perf_counter() - start_time

        # 计算每个元素的时间
        time_per_element = execution_time / size

        # 断言每个元素的时间大致恒定（容差 20%）
        expected_time_per_element = 1e-6  # 1 微秒
        assert time_per_element < expected_time_per_element * 1.2

    def test_concurrent_processing(self):
        """测试并发负载下的性能。"""
        import concurrent.futures

        def process_batch():
            data = np.random.rand(10000)
            return process_data(data)

        start_time = time.perf_counter()

        with concurrent.futures.ThreadPoolExecutor(max_workers=4) as executor:
            futures = [executor.submit(process_batch) for _ in range(10)]
            results = [f.result() for f in futures]

        execution_time = time.perf_counter() - start_time

        # 应该在 2 秒内通过并行处理完成 10 个批次
        assert execution_time < 2.0

    @pytest.mark.benchmark
    def test_benchmark(self, benchmark):
        """使用 pytest-benchmark 对函数进行基准测试。"""
        data = np.random.rand(10000)
        result = benchmark(process_data, data)

        # 基准统计数据的断言
        assert benchmark.stats["mean"] < 0.01  # 平均时间低于 10ms
        assert benchmark.stats["stddev"] < 0.002  # 低方差
```

## 测试生成最佳实践

### 1. 覆盖率目标
- 力争 >80% 的代码覆盖率
- 测试所有公共方法
- 包括边缘情况和错误路径

### 2. 测试组织
- 在类中分组相关测试
- 使用描述性的测试名称
- 遵循 AAA 模式（Arrange、Act、Assert）

### 3. Fixtures 和 Mocking
- 使用 fixtures 进行通用设置
- 模拟外部依赖项
- 保持测试隔离和独立

### 4. 参数化测试
- 使用参数化处理类似的测试用例
- 测试边界值
- 包括负面测试用例

### 5. 性能测试
- 设定现实的性能目标
- 使用具有代表性的数据大小进行测试
- 监控资源使用

## 另请参阅

- [测试最佳实践](../testing.md)
- [CI/CD 集成](../deployment/ci-cd.md)
- [检查点指南](../guide/checkpointing.md)
