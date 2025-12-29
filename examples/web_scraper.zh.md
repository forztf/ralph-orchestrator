# 网页爬虫示例

构建一个 Python 网页爬虫,要求:

1. 从 Hacker News (https://news.ycombinator.com) 抓取热门故事
2. 提取以下信息:
   - 标题
   - URL
   - 积分
   - 评论数
   - 提交时间

3. 将数据保存到以下两种格式:
   - JSON 文件 (hn_stories.json)
   - CSV 文件 (hn_stories.csv)

4. 实现速率限制(每秒 1 次请求)
5. 包括网络问题的错误处理
6. 添加用于调试的日志记录

要求:
- 使用 requests 和 BeautifulSoup
- 遵守 robots.txt 准则
- 包含 main() 函数
- 添加命令行参数以指定要获取的故事数量

保存为 hn_scraper.py

编排器将继续迭代,直到爬虫完全实现
