# 指标 API 参考

## 概述

指标 API 提供了从 Ralph Orchestrator 运行中收集、存储和分析执行指标的功能。

## 指标收集

### MetricsCollector 类

```python
class MetricsCollector:
    """
    收集和管理执行指标。

    示例:
        collector = MetricsCollector()
        collector.start_iteration(1)
        # ... 执行 ...
        collector.end_iteration(1, success=True)
        collector.save()
    """

    def __init__(self, metrics_dir: str = '.agent/metrics'):
        """
        初始化指标收集器。

        参数:
            metrics_dir (str): 存储指标文件的目录
        """
        self.metrics_dir = metrics_dir
        self.current_metrics = {
            'start_time': None,
            'iterations': [],
            'errors': [],
            'checkpoints': [],
            'agent_executions': [],
            'resource_usage': []
        }
        self._ensure_metrics_dir()

    def _ensure_metrics_dir(self):
        """如果指标目录不存在则创建。"""
        os.makedirs(self.metrics_dir, exist_ok=True)

    def start_iteration(self, iteration_num: int):
        """
        标记迭代开始。

        参数:
            iteration_num (int): 迭代编号

        示例:
            collector.start_iteration(1)
        """
        self.current_iteration = {
            'number': iteration_num,
            'start_time': time.time(),
            'end_time': None,
            'duration': None,
            'success': False,
            'output_size': 0,
            'errors': []
        }

    def end_iteration(self, iteration_num: int,
                     success: bool = True,
                     output_size: int = 0):
        """
        标记迭代结束。

        参数:
            iteration_num (int): 迭代编号
            success (bool): 迭代是否成功
            output_size (int): 输出大小(字节)

        示例:
            collector.end_iteration(1, success=True, output_size=2048)
        """
        if hasattr(self, 'current_iteration'):
            self.current_iteration['end_time'] = time.time()
            self.current_iteration['duration'] = (
                self.current_iteration['end_time'] -
                self.current_iteration['start_time']
            )
            self.current_iteration['success'] = success
            self.current_iteration['output_size'] = output_size

            self.current_metrics['iterations'].append(self.current_iteration)
            del self.current_iteration

    def record_error(self, error: str, iteration: int = None):
        """
        记录错误。

        参数:
            error (str): 错误消息
            iteration (int): 发生错误的迭代编号

        示例:
            collector.record_error("Agent timeout", iteration=5)
        """
        error_record = {
            'timestamp': time.time(),
            'iteration': iteration,
            'message': error,
            'traceback': traceback.format_exc() if sys.exc_info()[0] else None
        }

        self.current_metrics['errors'].append(error_record)

        if hasattr(self, 'current_iteration'):
            self.current_iteration['errors'].append(error)

    def record_checkpoint(self, iteration: int, commit_hash: str = None):
        """
        记录检查点。

        参数:
            iteration (int): 迭代编号
            commit_hash (str): Git 提交哈希

        示例:
            collector.record_checkpoint(5, "abc123def")
        """
        checkpoint = {
            'iteration': iteration,
            'timestamp': time.time(),
            'commit_hash': commit_hash
        }
        self.current_metrics['checkpoints'].append(checkpoint)

    def record_agent_execution(self, agent: str,
                              duration: float,
                              success: bool):
        """
        记录代理执行详情。

        参数:
            agent (str): 代理名称
            duration (float): 执行时长
            success (bool): 执行是否成功

        示例:
            collector.record_agent_execution('claude', 45.3, True)
        """
        execution = {
            'agent': agent,
            'duration': duration,
            'success': success,
            'timestamp': time.time()
        }
        self.current_metrics['agent_executions'].append(execution)

    def record_resource_usage(self):
        """
        记录当前资源使用情况。

        示例:
            collector.record_resource_usage()
        """
        import psutil

        process = psutil.Process()
        usage = {
            'timestamp': time.time(),
            'cpu_percent': process.cpu_percent(),
            'memory_mb': process.memory_info().rss / 1024 / 1024,
            'disk_io': process.io_counters()._asdict() if hasattr(process, 'io_counters') else {},
            'open_files': len(process.open_files()) if hasattr(process, 'open_files') else 0
        }
        self.current_metrics['resource_usage'].append(usage)

    def save(self, filename: str = None):
        """
        将指标保存到文件。

        参数:
            filename (str): 自定义文件名或自动生成

        返回:
            str: 已保存的指标文件路径

        示例:
            path = collector.save()
            print(f"Metrics saved to {path}")
        """
        if not filename:
            timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
            filename = f"metrics_{timestamp}.json"

        filepath = os.path.join(self.metrics_dir, filename)

        with open(filepath, 'w') as f:
            json.dump(self.current_metrics, f, indent=2, default=str)

        return filepath
```

## 状态管理

### StateManager 类

```python
class StateManager:
    """
    管理 Ralph 执行状态。

    示例:
        state = StateManager()
        state.update(iteration=5, status='running')
        state.save()
    """

    def __init__(self, state_dir: str = '.agent/metrics'):
        """
        初始化状态管理器。

        参数:
            state_dir (str): 状态文件目录
        """
        self.state_dir = state_dir
        self.state_file = os.path.join(state_dir, 'state_latest.json')
        self.state = self.load() or self.initialize_state()

    def initialize_state(self) -> dict:
        """
        初始化空状态。

        返回:
            dict: 初始状态结构
        """
        return {
            'status': 'idle',
            'iteration_count': 0,
            'start_time': None,
            'last_update': None,
            'runtime': 0,
            'agent': None,
            'prompt_file': None,
            'errors': [],
            'checkpoints': [],
            'task_complete': False
        }

    def load(self) -> dict:
        """
        从文件加载状态。

        返回:
            dict: 已加载的状态或 None

        示例:
            state = StateManager()
            current = state.load()
        """
        if os.path.exists(self.state_file):
            try:
                with open(self.state_file) as f:
                    return json.load(f)
            except json.JSONDecodeError:
                return None
        return None

    def save(self):
        """
        将当前状态保存到文件。

        示例:
            state.update(iteration=10)
            state.save()
        """
        os.makedirs(self.state_dir, exist_ok=True)

        # 更新 last_update 时间戳
        self.state['last_update'] = time.time()

        # 保存到最新文件
        with open(self.state_file, 'w') as f:
            json.dump(self.state, f, indent=2, default=str)

        # 同时保存带时间戳的版本
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        archive_file = os.path.join(
            self.state_dir,
            f"state_{timestamp}.json"
        )
        with open(archive_file, 'w') as f:
            json.dump(self.state, f, indent=2, default=str)

    def update(self, **kwargs):
        """
        更新状态值。

        参数:
            **kwargs: 要更新的状态值

        示例:
            state.update(
                iteration_count=5,
                status='running',
                agent='claude'
            )
        """
        self.state.update(kwargs)

        # 如果存在 start_time,计算运行时间
        if self.state.get('start_time'):
            self.state['runtime'] = time.time() - self.state['start_time']

    def get(self, key: str, default=None):
        """
        获取状态值。

        参数:
            key (str): 状态键
            default: 如果键未找到的默认值

        返回:
            状态中的值

        示例:
            iteration = state.get('iteration_count', 0)
        """
        return self.state.get(key, default)

    def reset(self):
        """
        将状态重置为初始值。

        示例:
            state.reset()
        """
        self.state = self.initialize_state()
        self.save()
```

## 指标分析

### MetricsAnalyzer 类

```python
class MetricsAnalyzer:
    """
    分析收集的指标。

    示例:
        analyzer = MetricsAnalyzer('.agent/metrics')
        report = analyzer.generate_report()
        print(report)
    """

    def __init__(self, metrics_dir: str = '.agent/metrics'):
        """
        初始化指标分析器。

        参数:
            metrics_dir (str): 包含指标文件的目录
        """
        self.metrics_dir = metrics_dir
        self.metrics_files = self.find_metrics_files()

    def find_metrics_files(self) -> List[str]:
        """
        查找所有指标文件。

        返回:
            list: 指标文件路径列表
        """
        pattern = os.path.join(self.metrics_dir, 'metrics_*.json')
        return sorted(glob.glob(pattern))

    def load_metrics(self, filepath: str) -> dict:
        """
        从文件加载指标。

        参数:
            filepath (str): 指标文件路径

        返回:
            dict: 已加载的指标
        """
        with open(filepath) as f:
            return json.load(f)

    def analyze_iterations(self, metrics: dict) -> dict:
        """
        分析迭代性能。

        参数:
            metrics (dict): 指标数据

        返回:
            dict: 迭代分析

        示例:
            metrics = analyzer.load_metrics('metrics.json')
            analysis = analyzer.analyze_iterations(metrics)
        """
        iterations = metrics.get('iterations', [])

        if not iterations:
            return {}

        durations = [i['duration'] for i in iterations if i.get('duration')]
        success_count = sum(1 for i in iterations if i.get('success'))

        return {
            'total_iterations': len(iterations),
            'successful_iterations': success_count,
            'success_rate': success_count / len(iterations) if iterations else 0,
            'avg_duration': sum(durations) / len(durations) if durations else 0,
            'min_duration': min(durations) if durations else 0,
            'max_duration': max(durations) if durations else 0,
            'total_duration': sum(durations)
        }

    def analyze_errors(self, metrics: dict) -> dict:
        """
        分析错误。

        参数:
            metrics (dict): 指标数据

        返回:
            dict: 错误分析
        """
        errors = metrics.get('errors', [])

        if not errors:
            return {'total_errors': 0}

        # 按迭代分组错误
        errors_by_iteration = {}
        for error in errors:
            iteration = error.get('iteration', 'unknown')
            if iteration not in errors_by_iteration:
                errors_by_iteration[iteration] = []
            errors_by_iteration[iteration].append(error['message'])

        return {
            'total_errors': len(errors),
            'errors_by_iteration': errors_by_iteration,
            'unique_errors': len(set(e['message'] for e in errors))
        }

    def analyze_resource_usage(self, metrics: dict) -> dict:
        """
        分析资源使用情况。

        参数:
            metrics (dict): 指标数据

        返回:
            dict: 资源使用分析
        """
        usage = metrics.get('resource_usage', [])

        if not usage:
            return {}

        cpu_values = [u['cpu_percent'] for u in usage if 'cpu_percent' in u]
        memory_values = [u['memory_mb'] for u in usage if 'memory_mb' in u]

        return {
            'avg_cpu_percent': sum(cpu_values) / len(cpu_values) if cpu_values else 0,
            'max_cpu_percent': max(cpu_values) if cpu_values else 0,
            'avg_memory_mb': sum(memory_values) / len(memory_values) if memory_values else 0,
            'max_memory_mb': max(memory_values) if memory_values else 0
        }

    def generate_report(self) -> str:
        """
        生成综合指标报告。

        返回:
            str: 格式化的报告

        示例:
            report = analyzer.generate_report()
            print(report)
        """
        if not self.metrics_files:
            return "No metrics files found"

        # 加载最新指标
        latest_file = self.metrics_files[-1]
        metrics = self.load_metrics(latest_file)

        # 分析不同方面
        iteration_analysis = self.analyze_iterations(metrics)
        error_analysis = self.analyze_errors(metrics)
        resource_analysis = self.analyze_resource_usage(metrics)

        # 格式化报告
        report = []
        report.append("=" * 50)
        report.append("RALPH ORCHESTRATOR METRICS REPORT")
        report.append("=" * 50)
        report.append(f"Metrics File: {os.path.basename(latest_file)}")
        report.append("")

        # 迭代统计
        report.append("ITERATION STATISTICS:")
        report.append(f"  Total Iterations: {iteration_analysis.get('total_iterations', 0)}")
        report.append(f"  Success Rate: {iteration_analysis.get('success_rate', 0):.1%}")
        report.append(f"  Avg Duration: {iteration_analysis.get('avg_duration', 0):.2f}s")
        report.append(f"  Total Runtime: {iteration_analysis.get('total_duration', 0):.2f}s")
        report.append("")

        # 错误统计
        report.append("ERROR STATISTICS:")
        report.append(f"  Total Errors: {error_analysis.get('total_errors', 0)}")
        report.append(f"  Unique Errors: {error_analysis.get('unique_errors', 0)}")
        report.append("")

        # 资源使用
        if resource_analysis:
            report.append("RESOURCE USAGE:")
            report.append(f"  Avg CPU: {resource_analysis.get('avg_cpu_percent', 0):.1f}%")
            report.append(f"  Max CPU: {resource_analysis.get('max_cpu_percent', 0):.1f}%")
            report.append(f"  Avg Memory: {resource_analysis.get('avg_memory_mb', 0):.1f} MB")
            report.append(f"  Max Memory: {resource_analysis.get('max_memory_mb', 0):.1f} MB")

        return "\n".join(report)
```

## 指标导出

### 导出函数

```python
def export_to_csv(metrics: dict, output_file: str):
    """
    将指标导出为 CSV。

    参数:
        metrics (dict): 指标数据
        output_file (str): 输出 CSV 文件路径

    示例:
        metrics = load_metrics('metrics.json')
        export_to_csv(metrics, 'metrics.csv')
    """
    import csv

    iterations = metrics.get('iterations', [])

    with open(output_file, 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=[
            'number', 'start_time', 'end_time',
            'duration', 'success', 'output_size'
        ])
        writer.writeheader()
        writer.writerows(iterations)

def export_to_prometheus(metrics: dict, output_file: str):
    """
    以 Prometheus 格式导出指标。

    参数:
        metrics (dict): 指标数据
        output_file (str): 输出文件路径

    示例:
        export_to_prometheus(metrics, 'metrics.prom')
    """
    lines = []

    # 迭代指标
    iterations = metrics.get('iterations', [])
    if iterations:
        total = len(iterations)
        successful = sum(1 for i in iterations if i.get('success'))

        lines.append(f'ralph_iterations_total {total}')
        lines.append(f'ralph_iterations_successful {successful}')
        lines.append(f'ralph_success_rate {successful/total if total else 0}')

    # 错误指标
    errors = metrics.get('errors', [])
    lines.append(f'ralph_errors_total {len(errors)}')

    # 写入文件
    with open(output_file, 'w') as f:
        f.write('\n'.join(lines))

def export_to_json_lines(metrics: dict, output_file: str):
    """
    将指标导出为 JSON Lines 用于流式处理。

    参数:
        metrics (dict): 指标数据
        output_file (str): 输出文件路径

    示例:
        export_to_json_lines(metrics, 'metrics.jsonl')
    """
    with open(output_file, 'w') as f:
        for iteration in metrics.get('iterations', []):
            f.write(json.dumps(iteration) + '\n')
```

## 实时指标

```python
class RealtimeMetrics:
    """
    提供实时指标访问。

    示例:
        realtime = RealtimeMetrics()
        realtime.start_monitoring()
        # ... 执行 ...
        stats = realtime.get_current_stats()
    """

    def __init__(self):
        self.current_stats = {}
        self.monitoring = False

    def start_monitoring(self):
        """启动实时监控。"""
        self.monitoring = True
        self.monitor_thread = threading.Thread(target=self._monitor_loop)
        self.monitor_thread.daemon = True
        self.monitor_thread.start()

    def _monitor_loop(self):
        """后台监控循环。"""
        while self.monitoring:
            self.current_stats = {
                'timestamp': time.time(),
                'cpu_percent': psutil.cpu_percent(interval=1),
                'memory_percent': psutil.virtual_memory().percent,
                'disk_usage': psutil.disk_usage('.').percent
            }
            time.sleep(5)

    def get_current_stats(self) -> dict:
        """获取当前统计信息。"""
        return self.current_stats.copy()

    def stop_monitoring(self):
        """停止监控。"""
        self.monitoring = False
        if hasattr(self, 'monitor_thread'):
            self.monitor_thread.join(timeout=1)
```
