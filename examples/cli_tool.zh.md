# CLI 工具示例

创建一个用于文件整理的 Python CLI 工具:

## 需求

1. 使用 argparse 的命令行界面
2. 命令:
   - `organize photos` - 按拍摄日期整理照片
   - `organize documents` - 按类型排序文档
   - `organize downloads` - 清理下载文件夹
   - `organize --custom <pattern>` - 自定义整理规则

3. 功能:
   - 预览更改的试运行模式
   - 撤销功能
   - 大型操作的进度条
   - 配置文件支持 (~/.file_organizer.yml)
   - 日志记录到文件

4. 整理规则:
   - 照片: 基于 EXIF 数据的 年/月 文件夹
   - 文档: 按扩展名分文件夹 (pdf/, docx/, txt/)
   - 下载: 归档旧文件,按类型分组
   - 自定义: 用户定义的模式

5. 安全性:
   - 永不删除文件
   - 移动前创建备份
   - 处理重复文件名
   - 保留文件权限

保存为 file_organizer.py 及支持模块:
- organizers/photo_organizer.py
- organizers/document_organizer.py
- utils/config.py
- utils/backup.py

包含依赖项的 requirements.txt

orchestrator 将继续迭代直到所有组件实现和测试完成
