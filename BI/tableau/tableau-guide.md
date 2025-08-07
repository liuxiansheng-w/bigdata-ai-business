# Tableau 操作指南与 TabPy 集成教程

## 一、Tableau 常用操作
### 1. 全屏设置
在 URL 中添加参数：`:embed=yes&:tabs=no&:toolbar=no`（添加前需移除末尾的`:iid=[#]`临时视图计数器）。最后通过浏览器全屏快捷键（Mac: `control+command+f`）进入全屏模式。

### 2. Tableau Server 工作簿自动刷新
1. 使用 QQ 浏览器或 360 浏览器打开 Tableau Server；
2. 在工作表网页 URL 后添加 `&:refresh=yes`；
3. 点击浏览器右侧菜单栏「工具」-「自动刷新」，选择刷新频次。

### 3. 工具提示中显示条形图
具体步骤参考：[举个栗子！Tableau技巧（37）](https://mp.weixin.qq.com/s?__biz=MzA5MTU3NDI2NQ==&mid=2649467479&idx=1&sn=13169ad83cc090f9d4008455622114ed&chksm=886550c1bf12d9d7e2b5aad5f4850e3280bf4a54e43e180da95f44c3190fb3dbe0c773f8bb8a&scene=21#wechat_redirect)，核心步骤包括：
- 创建年份销售额计算字段；
- 将字段添加到「详细信息」；
- 优化提示内容并设置颜色。

### 4. 同一视图切换不同图表
参考链接：[微信公众平台文章](https://mp.weixin.qq.com/s?__biz=MzA5MTU3NDI2NQ==&mid=2649467017&idx=1&sn=6b7f2ed9e0aeb5e5303e66cabee09cb6&chksm=8865521fbf12db09ad127c5570e4bb8020d3d7c9c3e0426d3791e190ffeb5dbb845ec433c88f&scene=21#wechat_redirect)（内容已删除，建议通过 Tableau 智能显示或图表类型切换功能实现）。

### 5. Tap 页实现页面跳转
参考：[IT610 教程](https://www.it610.com/article/1282814369477967872.htm)，通常通过超链接或操作（Actions）配置实现。

### 6. 用集操作实现树状图数据下钻
步骤参考：[举个栗子！Tableau 技巧（129）](https://mp.weixin.qq.com/s?__biz=MzA5MTU3NDI2NQ==&mid=2649478072&idx=1&sn=b0d62b8610e757fdfe862510ac2f7b90&chksm=8865292ebf12a0387d2dc199e801b6d9c7a71f03e3326a5caf24ee7df806223590e3dfb781fa&scene=21#wechat_redirect)，核心步骤：
- 创建「类别集」「子类集」等计算字段；
- 生成树状图并配置集操作（更改集值）。

---

## 二、TabPy 集成教程
### 1. TabPy 下载安装
- 源码下载：[GitHub 仓库](https://github.com/tableau/TabPy)，Windows 运行 `setup.bat`，Linux 运行 `setup.sh`；
- Anaconda 安装：在环境中搜索并安装 `tabpy_client` 和 `tabpy_server`；
- Python 环境安装：`pip install tabpy_server` 和 `pip install tabpy_client`。

### 2. TabPy Server 部署
- 找到安装目录（如 Anaconda 路径：`E:\Program Files\anaconda3\Lib\site-packages\tabpy_server`）；
- Windows 运行 `startup.bat`，Linux/Mac 运行 `startup.sh` 或 `python tabpy.py`；
- 启动成功后监听 9004 端口。

### 3. Tableau 连接配置
- 打开 Tableau，进入「帮助」-「设置和性能」-「管理外部服务连接」；
- 服务器地址设为 `localhost` 或 `127.0.0.1`，端口 `9004`；
- 测试连接成功后完成配置。

### 4. Tableau 发送 Python 脚本
- 新建计算字段（分析-创建计算字段），使用 `SCRIPTINT/SCRIPTREAL/SCRIPTSTR/SCRIPT_BOOL` 函数；
- 示例：`SCRIPT_STR("return _arg1 + _arg2", [字段1], [字段2])`，参数通过 `_arg#` 传递。

### 5. 部署 Python 脚本到 TabPy Server
- 编写函数（如 `weight_rate`）并通过 `tabpy_client.Client` 部署：
  ```python
  import tabpy_client
  import numpy as np
  def weight_rate(a, b):
      none_num = 0
      a = list(a)
      b = list(b)
      new_a, new_b = [], []
      for idx, num in enumerate(a):
          if num is None and idx == none_num:
              none_num += 1
              continue
          new_a.append(num if num is not None else 0)
          new_b.append(b[idx])
      try:
          return sum(new_a) / sum(new_b)
      except: return 0
  client = tabpy_client.Client('http://localhost:9004')
  client.deploy('weight_rate', weight_rate, override=True)
  ```

### 6. Tableau 调用已部署脚本
- 在 Tableau 中使用 `SCRIPT_*` 函数调用部署的脚本，如 `SCRIPT_REAL('weight_rate', [字段1], [字段2])`。

### 7. 常见错误
- 外部服务连接失败：检查 TabPy Server 是否启动，端口是否正确；
- 脚本执行错误：确保参数类型与函数要求一致，处理空值等异常情况。

参考链接：
- [TabPy GitHub 仓库](https://github.com/tableau/TabPy)
- [TabPy 配置文档](https://github.com/tableau/TabPy/blob/master/TableauConfiguration.md)