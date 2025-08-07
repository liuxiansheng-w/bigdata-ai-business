# Apache Superset 安装指南

## 一、环境要求
- Linux系统
- Python3环境

## 二、安装步骤

### 1. 安装virtualenv
用于建立虚拟环境，在这里安装包，不影响python自身的工作环境
```bash
pip install virtualenv
```

### 2. 创建并激活虚拟环境
```bash
python3 -m venv venv
. venv/bin/activate
```

### 3. Superset的安装和初始化
```bash
pip install apache-superset
```

创建管理员用户：
```bash
export FLASK_APP=superset
# 安装一系列依赖包，大概半小时左右
superset db upgrade
```

### 4. 配置文件设置（待测试，慎重使用）
配置文件位置：`/lib/python3.6/site-packages/superset/config.py`

```python
# 端口
SUPERSET_WEBSERVER_PORT = 9158

# 元数据库地址
SQLALCHEMY_DATABASE_URI = 'mysql://hdp:password@bd-prod-master01/superset?charset=utf8'

# 汉化
BABEL_DEFAULT_LOCALE = 'zh'
```

### 5. 创建超级管理员账号
```bash
# Create an admin user (you will be prompted to set a username, first and last name before setting a password)
export FLASK_APP=superset
superset fab create-admin
```

### 6. 初始化数据源
```bash
# Load some data to play with
superset load_examples
```

### 7. 初始化默认的角色与权限
```bash
# Create default roles and permissions
superset init
```

### 8. 启动superset服务
默认端口：8088
```bash
# To start a development web server on port 8088, use -p to bind to another port
superset run -p 8088 --with-threads --reload --debugger
```

## 注意事项
- 配置文件修改部分标记为"待测试，慎重使用"，请在测试环境中验证后再应用到生产环境
- 数据库初始化过程可能需要较长时间（约30分钟）
- 确保MySQL数据库连接信息正确配置