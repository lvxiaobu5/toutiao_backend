# 掘金头条后端

掘金头条是一个基于 FastAPI 和 MySQL 的新闻资讯类后端项目，为新闻客户端提供新闻、注册、登录和用户信息等 API 接口。项目采用异步数据库访问和模块化分层结构。

## 技术栈

- Python 3.12+
- FastAPI：构建 Web API
- Uvicorn：运行 ASGI 服务
- SQLAlchemy 2.x：ORM 与异步数据库访问
- aiomysql / PyMySQL：连接 MySQL
- Pydantic / Starlette：数据校验与 Web 基础能力
- python-dotenv：加载本地环境变量

## 当前项目结构

```text
toutiao_backend/
├── config/       # 数据库连接与配置
├── crud/         # 数据库操作封装
├── models/       # SQLAlchemy 数据模型
├── routers/      # API 路由
├── schemas/      # 接口数据结构（预留扩展）
├── utils/        # 通用工具
├── main.py       # 应用入口
├── requirements.txt
├── .env.example  # 环境变量示例
└── test_main.http
```

## 已完成工作

### 基础工程

- 搭建 FastAPI 应用入口
- 配置 CORS，支持前端跨域访问
- 按 `config / models / crud / routers` 分层组织代码
- 接入 SQLAlchemy 异步引擎和异步 Session
- 使用环境变量管理数据库连接配置

### 新闻模块

- 新闻分类列表
- 按分类分页获取新闻列表
- 新闻总数与是否有更多数据判断
- 新闻详情查询
- 新闻详情访问量自动增加
- 同分类相关新闻推荐
- 新闻分类、新闻内容、发布时间、作者、封面和浏览量等数据模型
- 为分类查询和发布时间查询建立数据库索引

### 用户模块

- 用户注册
- 用户登录
- Token 认证
- 获取当前用户信息
- 修改用户信息
- 修改密码
- 密码 bcrypt 加密存储
- 用户名和密码参数校验

## API 接口

基础地址：`http://127.0.0.1:8000`

当前已实现接口：

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/` | 服务健康检查 |
| GET | `/api/news/categories` | 获取新闻分类列表 |
| GET | `/api/news/list?categoryId=1&page=1&pageSize=10` | 分页获取分类新闻 |
| GET | `/api/news/detail?id=1` | 获取新闻详情并增加浏览量 |
| POST | `/api/user/register` | 用户注册 |
| POST | `/api/user/login` | 用户登录 |
| GET | `/api/user/info` | 获取当前用户信息 |
| PUT | `/api/user/update` | 修改用户信息 |
| PUT | `/api/user/password` | 修改密码 |

FastAPI 启动后可访问：

- Swagger UI：`/docs`
- ReDoc：`/redoc`

## 配置与运行

1. 创建并激活虚拟环境：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

2. 安装依赖：

```powershell
pip install -r requirements.txt
```

3. 创建 `.env` 文件并配置 MySQL：

```env
ASYNC_DATABASE_URL=mysql+aiomysql://root:你的密码@localhost:3306/news_app?charset=utf8mb4
```

4. 启动服务：

```powershell
uvicorn main:app --reload
```

`.env`、虚拟环境、IDE 配置和 Python 缓存均已加入 `.gitignore`，不会提交到仓库。

## 数据库说明

当前代码使用以下核心表：

- `news_category`：新闻分类
- `news`：新闻内容及浏览量

请先创建数据库 `news_app`，并确保 MySQL 服务已启动。表结构可根据 `models/` 中的 SQLAlchemy 模型建立。

## 开发说明

本项目已完成新闻模块和用户模块的核心后端能力，包括用户注册、登录、Token 认证、用户信息管理和密码修改。
