# 使用 Ralph 的数据分析脚本

此示例演示如何使用 Ralph Orchestrator 创建一个数据分析脚本,包含 pandas 数据处理、可视化和报告生成功能。

## 任务描述

创建一个 Python 数据分析脚本,实现以下功能:
- 加载和清洗 CSV 数据
- 执行统计分析
- 创建可视化图表
- 生成 HTML 报告

## PROMPT.md 文件

```markdown
# 任务:构建销售数据分析脚本

创建一个 Python 脚本来分析销售数据,具体要求如下:

## 数据处理

1. 从 CSV 文件加载销售数据
2. 清洗和验证数据:
   - 处理缺失值
   - 转换数据类型
   - 删除重复记录
   - 验证日期范围

## 分析要求

1. **销售指标**
   - 按月统计总收入
   - 平均订单价值
   - 按收入排名的前 10 个产品
   - 销售增长率

2. **客户分析**
   - 客户细分(RFM 分析)
   - 客户生命周期价值
   - 复购率
   - 地理分布

3. **产品分析**
   - 表现最好/最差的产品
   - 产品类别表现
   - 季节性趋势
   - 库存周转率

## 可视化

创建以下图表:
1. 月度收入趋势(折线图)
2. 产品类别分布(饼图)
3. 客户分布地图(地理)
4. 按天/小时的销售热力图
5. 热门产品柱状图

## 输出

生成 HTML 报告,包含:
- 执行摘要
- 关键指标仪表板
- 交互式图表(使用 plotly)
- 数据表格
- 洞察和建议

## 文件结构

```
sales-analysis/
├── analyze.py          # 主分析脚本
├── data_loader.py      # 数据加载和清洗
├── analysis.py         # 分析函数
├── visualizations.py   # 图表生成
├── report_generator.py # HTML 报告创建
├── requirements.txt    # 依赖项
├── config.yaml        # 配置文件
├── templates/         # HTML 模板
│   └── report.html
├── data/             # 数据目录
│   └── sales.csv     # 示例数据
└── output/           # 输出目录
    └── report.html   # 生成的报告
```

## 示例数据结构

CSV 列:
- order_id, customer_id, product_id, product_name, category
- quantity, unit_price, total_price, discount
- order_date, ship_date, region, payment_method

<!-- orchestrator 将继续迭代直到达到限制 -->
```

## 运行 Ralph

```bash
ralph init
cp data-analysis-prompt.md PROMPT.md
ralph run --agent claude --max-iterations 35
```

## 预期输出

### analyze.py (主脚本)

```python
#!/usr/bin/env python3
"""
销售数据分析脚本
分析销售数据并生成综合 HTML 报告
"""

import pandas as pd
import numpy as np
from datetime import datetime
import yaml
import logging
from pathlib import Path

from data_loader import DataLoader
from analysis import SalesAnalyzer, CustomerAnalyzer, ProductAnalyzer
from visualizations import ChartGenerator
from report_generator import ReportGenerator

# 设置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def load_config(config_path='config.yaml'):
    """从 YAML 文件加载配置"""
    with open(config_path, 'r') as f:
        return yaml.safe_load(f)

def main():
    """主分析流程"""
    logger.info("开始销售数据分析...")

    # 加载配置
    config = load_config()

    # 步骤 1: 加载和清洗数据
    logger.info("加载数据...")
    loader = DataLoader(config['data']['input_file'])
    df = loader.load_and_clean()
    logger.info(f"已加载 {len(df)} 条记录")

    # 步骤 2: 执行分析
    logger.info("执行分析...")

    # 销售分析
    sales_analyzer = SalesAnalyzer(df)
    sales_metrics = {
        'total_revenue': sales_analyzer.calculate_total_revenue(),
        'monthly_revenue': sales_analyzer.get_monthly_revenue(),
        'avg_order_value': sales_analyzer.calculate_avg_order_value(),
        'growth_rate': sales_analyzer.calculate_growth_rate(),
        'top_products': sales_analyzer.get_top_products(10)
    }

    # 客户分析
    customer_analyzer = CustomerAnalyzer(df)
    customer_metrics = {
        'total_customers': customer_analyzer.count_unique_customers(),
        'repeat_rate': customer_analyzer.calculate_repeat_rate(),
        'rfm_segments': customer_analyzer.perform_rfm_analysis(),
        'lifetime_value': customer_analyzer.calculate_clv(),
        'geographic_dist': customer_analyzer.get_geographic_distribution()
    }

    # 产品分析
    product_analyzer = ProductAnalyzer(df)
    product_metrics = {
        'category_performance': product_analyzer.analyze_categories(),
        'seasonal_trends': product_analyzer.find_seasonal_trends(),
        'inventory_turnover': product_analyzer.calculate_turnover(),
        'product_ranking': product_analyzer.rank_products()
    }

    # 步骤 3: 生成可视化
    logger.info("创建可视化...")
    chart_gen = ChartGenerator(df)

    charts = {
        'revenue_trend': chart_gen.create_revenue_trend(
            sales_metrics['monthly_revenue']
        ),
        'category_pie': chart_gen.create_category_pie(
            product_metrics['category_performance']
        ),
        'customer_map': chart_gen.create_customer_map(
            customer_metrics['geographic_dist']
        ),
        'sales_heatmap': chart_gen.create_sales_heatmap(),
        'top_products_bar': chart_gen.create_top_products_bar(
            sales_metrics['top_products']
        )
    }

    # 步骤 4: 生成报告
    logger.info("生成 HTML 报告...")
    report_gen = ReportGenerator()

    report_data = {
        'generated_at': datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
        'record_count': len(df),
        'date_range': f"{df['order_date'].min()} 到 {df['order_date'].max()}",
        'sales_metrics': sales_metrics,
        'customer_metrics': customer_metrics,
        'product_metrics': product_metrics,
        'charts': charts,
        'insights': generate_insights(sales_metrics, customer_metrics, product_metrics)
    }

    output_path = Path(config['output']['report_path'])
    output_path.parent.mkdir(parents=True, exist_ok=True)

    report_gen.generate(report_data, output_path)

    logger.info(f"报告已生成: {output_path}")
    logger.info("分析完成!")

    return output_path

def generate_insights(sales, customers, products):
    """从分析结果生成洞察"""
    insights = []

    # 收入洞察
    if sales['growth_rate'] > 0:
        insights.append(f"收入以每月 {sales['growth_rate']:.1f}% 的速度增长")
    else:
        insights.append(f"收入每月下降 {abs(sales['growth_rate']):.1f}%")

    # 客户洞察
    repeat_rate = customers['repeat_rate']
    if repeat_rate > 0.3:
        insights.append(f"客户忠诚度强,复购率为 {repeat_rate:.1%}")
    else:
        insights.append(f"有机会提高留存率(当前: {repeat_rate:.1%})")

    # 产品洞察
    top_category = products['category_performance'].iloc[0]
    insights.append(f"{top_category['category']} 是表现最佳的类别")

    return insights

if __name__ == '__main__':
    try:
        report_path = main()
        print(f"\n✅ 分析完成! 报告已保存到: {report_path}")
    except Exception as e:
        logger.error(f"分析失败: {e}")
        raise
```

### data_loader.py

```python
import pandas as pd
import numpy as np
from datetime import datetime
import logging

logger = logging.getLogger(__name__)

class DataLoader:
    """处理数据加载和清洗"""

    def __init__(self, filepath):
        self.filepath = filepath

    def load_and_clean(self):
        """加载 CSV 并执行清洗"""
        # 加载数据
        df = pd.read_csv(self.filepath)
        logger.info(f"已加载 {len(df)} 条原始记录")

        # 清洗数据
        df = self.remove_duplicates(df)
        df = self.handle_missing_values(df)
        df = self.convert_data_types(df)
        df = self.validate_data(df)

        logger.info(f"清洗后数据: {len(df)} 条记录")
        return df

    def remove_duplicates(self, df):
        """删除重复记录"""
        before = len(df)
        df = df.drop_duplicates(subset=['order_id'])
        after = len(df)

        if before > after:
            logger.info(f"已删除 {before - after} 条重复记录")

        return df

    def handle_missing_values(self, df):
        """适当处理缺失值"""
        # 用 0 填充数值列
        numeric_cols = df.select_dtypes(include=[np.number]).columns
        df[numeric_cols] = df[numeric_cols].fillna(0)

        # 用 'Unknown' 填充分类列
        categorical_cols = df.select_dtypes(include=['object']).columns
        df[categorical_cols] = df[categorical_cols].fillna('Unknown')

        return df

    def convert_data_types(self, df):
        """将列转换为适当的数据类型"""
        # 转换日期
        date_columns = ['order_date', 'ship_date']
        for col in date_columns:
            if col in df.columns:
                df[col] = pd.to_datetime(df[col], errors='coerce')

        # 转换数值列
        numeric_columns = ['quantity', 'unit_price', 'total_price', 'discount']
        for col in numeric_columns:
            if col in df.columns:
                df[col] = pd.to_numeric(df[col], errors='coerce')

        # 将 ID 转换为字符串
        id_columns = ['order_id', 'customer_id', 'product_id']
        for col in id_columns:
            if col in df.columns:
                df[col] = df[col].astype(str)

        return df

    def validate_data(self, df):
        """验证数据完整性"""
        # 删除日期无效的行
        if 'order_date' in df.columns:
            df = df[df['order_date'].notna()]

        # 删除价格为负的行
        if 'total_price' in df.columns:
            df = df[df['total_price'] >= 0]

        # 删除数量无效的行
        if 'quantity' in df.columns:
            df = df[df['quantity'] > 0]

        return df

    def generate_sample_data(self, num_records=1000):
        """生成示例销售数据用于测试"""
        np.random.seed(42)

        # 生成日期
        dates = pd.date_range(
            start='2023-01-01',
            end='2023-12-31',
            periods=num_records
        )

        # 类别和产品
        categories = ['Electronics', 'Clothing', 'Books', 'Home', 'Sports']
        products = {
            'Electronics': ['Laptop', 'Phone', 'Tablet', 'Headphones'],
            'Clothing': ['Shirt', 'Pants', 'Jacket', 'Shoes'],
            'Books': ['Fiction', 'Non-fiction', 'TextBook', 'Magazine'],
            'Home': ['Furniture', 'Decor', 'Kitchen', 'Bedding'],
            'Sports': ['Equipment', 'Apparel', 'Footwear', 'Accessories']
        }

        # 生成记录
        records = []
        for i in range(num_records):
            category = np.random.choice(categories)
            product = np.random.choice(products[category])
            quantity = np.random.randint(1, 10)
            unit_price = np.random.uniform(10, 500)
            discount = np.random.uniform(0, 0.3)

            records.append({
                'order_id': f'ORD{i:05d}',
                'customer_id': f'CUST{np.random.randint(1, 200):04d}',
                'product_id': f'PROD{np.random.randint(1, 50):03d}',
                'product_name': product,
                'category': category,
                'quantity': quantity,
                'unit_price': unit_price,
                'total_price': quantity * unit_price * (1 - discount),
                'discount': discount,
                'order_date': dates[i],
                'ship_date': dates[i] + pd.Timedelta(days=np.random.randint(1, 7)),
                'region': np.random.choice(['North', 'South', 'East', 'West']),
                'payment_method': np.random.choice(['Credit Card', 'PayPal', 'Cash'])
            })

        return pd.DataFrame(records)
```

### visualizations.py

```python
import plotly.graph_objects as go
import plotly.express as px
from plotly.subplots import make_subplots
import pandas as pd

class ChartGenerator:
    """使用 Plotly 生成交互式图表"""

    def __init__(self, df):
        self.df = df

    def create_revenue_trend(self, monthly_revenue):
        """创建月度收入趋势折线图"""
        fig = go.Figure()

        fig.add_trace(go.Scatter(
            x=monthly_revenue.index,
            y=monthly_revenue.values,
            mode='lines+markers',
            name='Revenue',
            line=dict(color='#1f77b4', width=3),
            marker=dict(size=8)
        ))

        fig.update_layout(
            title='Monthly Revenue Trend',
            xaxis_title='Month',
            yaxis_title='Revenue ($)',
            hovermode='x unified',
            template='plotly_white'
        )

        return fig.to_html(include_plotlyjs='cdn')

    def create_category_pie(self, category_data):
        """创建类别分布饼图"""
        fig = px.pie(
            category_data,
            values='revenue',
            names='category',
            title='Revenue by Category',
            color_discrete_sequence=px.colors.qualitative.Set3
        )

        fig.update_traces(
            textposition='inside',
            textinfo='percent+label'
        )

        return fig.to_html(include_plotlyjs='cdn')

    def create_sales_heatmap(self):
        """创建按天和小时的销售热力图"""
        # 提取天和小时
        self.df['day_of_week'] = self.df['order_date'].dt.day_name()
        self.df['hour'] = self.df['order_date'].dt.hour

        # 汇总销售数据
        heatmap_data = self.df.groupby(['day_of_week', 'hour'])[
            'total_price'
        ].sum().reset_index()

        # 透视表用于热力图
        pivot_table = heatmap_data.pivot(
            index='day_of_week',
            columns='hour',
            values='total_price'
        )

        # 重新排序日期
        days_order = ['Monday', 'Tuesday', 'Wednesday', 'Thursday',
                     'Friday', 'Saturday', 'Sunday']
        pivot_table = pivot_table.reindex(days_order)

        fig = go.Figure(data=go.Heatmap(
            z=pivot_table.values,
            x=pivot_table.columns,
            y=pivot_table.index,
            colorscale='Viridis',
            text=pivot_table.values.round(0),
            texttemplate='%{text}',
            textfont={"size": 10}
        ))

        fig.update_layout(
            title='Sales Heatmap by Day and Hour',
            xaxis_title='Hour of Day',
            yaxis_title='Day of Week',
            template='plotly_white'
        )

        return fig.to_html(include_plotlyjs='cdn')

    def create_top_products_bar(self, top_products):
        """创建热门产品的水平柱状图"""
        fig = go.Figure(go.Bar(
            x=top_products['revenue'],
            y=top_products['product_name'],
            orientation='h',
            marker_color='lightblue',
            text=top_products['revenue'].round(0),
            textposition='outside'
        ))

        fig.update_layout(
            title='Top 10 Products by Revenue',
            xaxis_title='Revenue ($)',
            yaxis_title='Product',
            template='plotly_white',
            height=400
        )

        return fig.to_html(include_plotlyjs='cdn')

    def create_customer_map(self, geographic_data):
        """创建地理分布地图"""
        # 为简单起见,创建按区域的柱状图
        fig = px.bar(
            geographic_data,
            x='region',
            y='customer_count',
            title='Customer Distribution by Region',
            color='customer_count',
            color_continuous_scale='Blues'
        )

        fig.update_layout(
            xaxis_title='Region',
            yaxis_title='Number of Customers',
            template='plotly_white',
            showlegend=False
        )

        return fig.to_html(include_plotlyjs='cdn')
```

## 报告模板

### templates/report.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sales Analysis Report</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 20px;
            background: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #333;
            border-bottom: 3px solid #4CAF50;
            padding-bottom: 10px;
        }
        .metrics-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin: 30px 0;
        }
        .metric-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
        }
        .metric-value {
            font-size: 2em;
            font-weight: bold;
            margin: 10px 0;
        }
        .metric-label {
            font-size: 0.9em;
            opacity: 0.9;
        }
        .chart-container {
            margin: 30px 0;
        }
        .insights {
            background: #e8f5e9;
            padding: 20px;
            border-radius: 10px;
            margin: 30px 0;
        }
        .insight-item {
            margin: 10px 0;
            padding-left: 20px;
            position: relative;
        }
        .insight-item:before {
            content: "→";
            position: absolute;
            left: 0;
            color: #4CAF50;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📊 Sales Analysis Report</h1>

        <div class="report-meta">
            <p><strong>Generated:</strong> {{ generated_at }}</p>
            <p><strong>Data Range:</strong> {{ date_range }}</p>
            <p><strong>Total Records:</strong> {{ record_count }}</p>
        </div>

        <h2>Key Metrics</h2>
        <div class="metrics-grid">
            <div class="metric-card">
                <div class="metric-label">Total Revenue</div>
                <div class="metric-value">${{ total_revenue|round(0) }}</div>
            </div>
            <div class="metric-card">
                <div class="metric-label">Avg Order Value</div>
                <div class="metric-value">${{ avg_order_value|round(2) }}</div>
            </div>
            <div class="metric-card">
                <div class="metric-label">Total Customers</div>
                <div class="metric-value">{{ total_customers }}</div>
            </div>
            <div class="metric-card">
                <div class="metric-label">Repeat Rate</div>
                <div class="metric-value">{{ repeat_rate|round(1) }}%</div>
            </div>
        </div>

        <h2>Insights</h2>
        <div class="insights">
            {% for insight in insights %}
            <div class="insight-item">{{ insight }}</div>
            {% endfor %}
        </div>

        <h2>Revenue Trend</h2>
        <div class="chart-container">
            {{ revenue_trend_chart|safe }}
        </div>

        <h2>Category Performance</h2>
        <div class="chart-container">
            {{ category_pie_chart|safe }}
        </div>

        <h2>Top Products</h2>
        <div class="chart-container">
            {{ top_products_chart|safe }}
        </div>

        <h2>Sales Patterns</h2>
        <div class="chart-container">
            {{ sales_heatmap|safe }}
        </div>
    </div>
</body>
</html>
```

## 数据分析任务的建议

1. **明确数据结构**: 清晰定义输入数据格式
2. **列出所需分析**: 明确说明需要的计算
3. **请求可视化**: 指定图表类型和库
4. **输出格式**: 定义报告结构和格式
5. **错误处理**: 请求验证和错误处理

## 成本估算

- **迭代次数**: 完整实现约需 25-35 次
- **时间**: 约 12-18 分钟
- **代理**: 对于复杂分析推荐使用 Claude
- **API 调用**: 约 $0.25-0.35
