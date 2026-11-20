# 掘金头条后端

掘金头条是一套面向新闻资讯客户端的异步 REST API 服务，提供新闻内容分发、用户身份认证、个人资料管理、收藏管理与浏览历史管理等核心能力。项目以 FastAPI 为应用层，以 SQLAlchemy 2.x 异步 ORM 为数据访问层，采用路由、数据访问、模型和数据校验分层，适合作为资讯类应用的后端服务基础。

## 项目能力

- 新闻分类浏览与分页查询
- 新闻详情、阅读量统计与同类内容推荐
- 用户注册、登录与 Token 身份认证
- 用户资料维护与密码修改
- 新闻收藏状态查询、添加、删除、分页查看和批量清空
- 浏览历史记录写入、分页查看、单条删除和批量清空
- 统一响应结构、参数校验和异常处理
- MySQL 异步连接池与事务级数据库会话
- CORS 跨域支持与自动生成的 Swagger UI / ReDoc 文档

## 技术架构

| 层级 | 目录 | 职责 |
| --- | --- | --- |
| 应用入口 | `main.py` | 创建 FastAPI 应用、注册中间件与路由 |
| 配置层 | `config/` | 数据库引擎、异步 Session 与运行配置 |
| 路由层 | `routers/` | 定义 HTTP 接口、参数接收与依赖注入 |
| 数据访问层 | `crud/` | 封装新闻、用户、收藏和历史记录的数据操作 |
| 数据模型层 | `models/` | 定义 ORM 模型、外键、索引与约束 |
| 数据契约层 | `schemas/` | 定义请求体、响应体和分页数据结构 |
| 通用能力 | `utils/` | 认证、密码安全、统一响应与异常处理 |

## 技术栈

- Python 3.12+
- FastAPI 0.125+、Uvicorn
- SQLAlchemy 2.x Async ORM、aiomysql / MySQL
- Pydantic 2.x
- Passlib、bcrypt、cryptography
- python-dotenv

## 目录结构

```text
toutiao_backend/
├── config/                 # 数据库连接与运行配置
├── crud/                   # 数据访问与业务查询封装
│   ├── favorite.py         # 收藏数据操作
│   ├── history.py          # 浏览历史数据操作
│   ├── news.py             # 新闻与分类数据操作
│   └── users.py            # 用户与 Token 数据操作
├── models/                 # SQLAlchemy ORM 模型
├── routers/                # FastAPI 路由模块
├── schemas/                # Pydantic 请求/响应模型
├── utils/                  # 认证、安全、响应和异常处理
├── main.py                 # 应用入口
├── requirements.txt        # Python 依赖
├── .env.example            # 环境配置示例
└── test_main.http          # HTTP 接口调试示例
```

## API 设计

基础地址：`http://127.0.0.1:8000`

### 系统与新闻

| 方法 | 路径 | 说明 | 认证 |
| --- | --- | --- | --- |
| GET | `/` | 服务健康检查 | 否 |
| GET | `/api/news/categories` | 获取新闻分类 | 否 |
| GET | `/api/news/list?categoryId=1&page=1&pageSize=10` | 分页获取分类新闻 | 否 |
| GET | `/api/news/detail?id=1` | 获取详情、增加阅读量并返回相关推荐 | 否 |

### 用户与认证

| 方法 | 路径 | 说明 | 认证 |
| --- | --- | --- | --- |
| POST | `/api/user/register` | 注册用户并返回访问 Token | 否 |
| POST | `/api/user/login` | 登录并返回访问 Token | 否 |
| GET | `/api/user/info` | 获取当前用户信息 | 是 |
| PUT | `/api/user/update` | 修改用户资料 | 是 |
| PUT | `/api/user/password` | 修改登录密码 | 是 |

### 收藏

| 方法 | 路径 | 说明 | 认证 |
| --- | --- | --- | --- |
| GET | `/api/favorite/check?newsId=1` | 查询当前用户是否收藏新闻 | 是 |
| POST | `/api/favorite/add` | 添加新闻收藏 | 是 |
| DELETE | `/api/favorite/remove?newsId=1` | 移除新闻收藏 | 是 |
| GET | `/api/favorite/list?page=1&pageSize=10` | 分页获取收藏列表 | 是 |
| DELETE | `/api/favorite/clear` | 清空当前用户收藏 | 是 |

### 浏览历史

| 方法 | 路径 | 说明 | 认证 |
| --- | --- | --- | --- |
| POST | `/api/history/add` | 写入一条新闻浏览记录 | 是 |
| GET | `/api/history/list?page=1&pageSize=10` | 分页获取浏览历史 | 是 |
| DELETE | `/api/history/delete/{history_id}` | 删除指定历史记录 | 是 |
| DELETE | `/api/history/clear` | 清空当前用户浏览历史 | 是 |

认证接口通过请求头传递 Token：

```http
Authorization: Bearer <token>
```

分页接口统一返回列表、总数和 `hasMore` 字段；业务成功响应包含 `code`、`message` 和 `data`，便于客户端统一处理。

## 数据模型

- `news_category`：新闻分类及分类信息
- `news`：新闻标题、正文、封面、作者、发布时间、分类和阅读量
- `users`：用户账号、密码摘要、头像与个人资料
- `tokens`：用户登录凭证及认证关联信息
- `favorite`：用户与新闻的收藏关系，包含唯一约束，避免重复收藏
- `history`：用户与新闻的浏览记录，按浏览时间建立索引以支持历史分页查询

收藏关系对 `(user_id, news_id)` 设置唯一约束；收藏和历史表分别为用户、新闻建立索引，保证用户维度的列表查询与内容维度的关联查询具备稳定的数据访问路径。

## 安装与运行

### 1. 创建虚拟环境

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 2. 安装依赖

```powershell
pip install -r requirements.txt
```

### 3. 准备 MySQL

```sql
CREATE DATABASE news_app CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

复制 `.env.example` 为 `.env`，填写异步数据库连接信息，并根据部署环境调整 `config/db_conf.py` 中的数据库配置。

```env
ASYNC_DATABASE_URL=mysql+aiomysql://root:password@localhost:3306/news_app?charset=utf8mb4
```

### 4. 启动服务

```powershell
uvicorn main:app --reload
```

服务启动后可访问：

- Swagger UI：`http://127.0.0.1:8000/docs`
- ReDoc：`http://127.0.0.1:8000/redoc`

## 运行特性

- 所有数据库操作使用异步 Session，事务在请求生命周期内统一提交或回滚。
- 用户密码采用 bcrypt 进行摘要存储，不保存明文密码。
- 通过 FastAPI 依赖注入统一获取数据库会话和当前登录用户。
- 新闻详情访问会更新阅读量，并返回同分类相关推荐。
- 收藏与浏览历史均以当前用户为数据隔离边界，避免跨用户读取或修改数据。
- 请求参数由 Pydantic 和 FastAPI Query 约束校验，分页大小限制为 100。

## 接口调试

项目提供 `test_main.http` 作为 HTTP Client 调试入口，也可以直接使用 Swagger UI 生成请求。推荐先注册或登录获取 Token，再在 Swagger 的 `Authorize` 中填写 `Bearer <token>`，然后调试用户、收藏和浏览历史接口。

## 配置与安全建议

- 不要将真实数据库密码、Token 或其他密钥提交到版本库；`.env` 已被忽略，提交前请检查 Git 状态。
- 生产环境应将 CORS 的允许来源从通配符收紧为实际前端域名。
- 生产环境应关闭 SQL 调试日志，并为数据库账号配置最小权限。
- 生产环境应在反向代理层启用 HTTPS，并结合日志、限流和监控完善服务治理。
