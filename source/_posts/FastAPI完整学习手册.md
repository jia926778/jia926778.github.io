---
title: FastAPI学习手册
date: 2026-04-28 19:30:00
tags: [python, 技术分享, FastAPI]
categories: python
toc: true
---
# FastAPI完整学习手册

## 文档核心说明

本文档适用于 Python 后端开发者、FastAPI 入门到进阶学习者，系统梳理了 FastAPI 全栈开发的核心知识，按照从基础到进阶、从理论到实操的学习逻辑重构，解决了原始内容零散、重复的问题，可直接作为学习手册使用。

---

## 一、基础入门

### 1.1 FastAPI 简介

FastAPI 是一款现代、高性能的 Python Web 框架，专为构建 API 而生，核心基于**Starlette**（Web 层）和**Pydantic**（数据校验层），核心优势：

- **极致性能**：性能比肩 Node.js 和 Go，是 Python 性能天花板级别的 Web 框架

- **极速开发**：基于类型提示，开发效率提升约 200%，减少人为 bug

- **自动文档**：零配置生成 Swagger UI 和 ReDoc 两份交互式 API 文档

- **强类型校验**：自动完成请求 / 响应数据的类型转换与合法性校验

- **标准兼容**：完全兼容 OpenAPI 和 JSON Schema 规范

- **企业级能力**：内置依赖注入、认证授权、异步支持等生产级特性

### 1.2 环境搭建

#### 环境要求

- Python 3.8\+（推荐 3.10\+，支持更简洁的联合类型语法）

- 虚拟环境（推荐 venv/conda，避免全局依赖污染）

#### 安装依赖

官方推荐一键安装完整版本：

```bash
pip install "fastapi[standard]"
```

最小化安装：

```bash
pip install fastapi uvicorn
```

#### 开发工具推荐

- 编辑器：VS Code（安装 Python 和 Pylance 插件，完美支持类型提示）

- 接口调试：内置的`/docs`文档、Postman、Apifox

- 环境管理：venv、conda、poetry

### 1.3 快速入门：第一个 FastAPI 应用

#### 最小应用示例

创建`main.py`：

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello FastAPI!"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "query": q}
```

#### 启动服务

开发环境启动（自动热重载）：

```bash
fastapi dev main.py
```

传统 uvicorn 启动：

```bash
uvicorn main:app --reload
```

> **注意**：`--reload`仅用于开发环境，**生产环境禁止使用**
>
>

#### 访问接口与自动文档

服务启动后，可访问：

1. 接口根地址：`http://127.0.0.1:8000`

2. 交互式 API 文档（Swagger UI）：`http://127.0.0.1:8000/docs`

3. 备用文档（ReDoc）：`http://127.0.0.1:8000/redoc`

#### 核心代码解析

- **路径操作装饰器**：`@app.get("/ ")`，绑定 HTTP 方法与路径，支持`post`/`put`/`delete`等所有方法

- **路径操作函数**：支持`async def`异步函数（IO 密集型）和普通`def`同步函数（CPU 密集型）

- **返回值**：自动将字典、模型、ORM 对象序列化为 JSON

---

## 二、核心基础：请求与响应处理

### 2.1 路径参数

路径参数是 URL 中可变的资源标识，FastAPI 自动完成类型转换和校验。

```python
from enum import Enum
from fastapi import FastAPI

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    return {"model_name": model_name}
```

> 固定路径优先于动态路径，避免被动态路径覆盖。
>
>

### 2.2 查询参数

查询参数是 URL 中`?`后的键值对，用于分页、筛选，函数中非路径参数自动识别为查询参数。

```python
@app.get("/items/")
async def read_items(skip: int = 0, limit: int = 10, name: str | None = None):
    return {"skip": skip, "limit": limit, "name": name}
```

支持布尔类型自动转换，可识别`1/0`、`true/false`等格式。

### 2.3 请求体与 Pydantic 模型

通过 Pydantic BaseModel 定义请求体结构，自动完成校验、解析和文档生成。

```python
from pydantic import BaseModel, Field

class Item(BaseModel):
    name: str = Field(min_length=1, max_length=100, description="商品名称")
    price: float = Field(gt=0, description="商品价格")
    description: str | None = None
    is_offer: bool = False

@app.post("/items/")
async def create_item(item: Item):
    return item.model_dump()
```

### 2.4 参数校验：Query/Path

通过`Query`和`Path`实现精细化参数校验，所有配置同步到 Swagger 文档。

```python
from fastapi import FastAPI, Query, Path

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: str = Query(..., min_length=3, max_length=50, regex="^[a-zA-Z0-9]+$"),
    limit: int = Query(default=10, ge=1, le=100)
):
    return {"q": q, "limit": limit}

@app.get("/items/{item_id}")
async def read_item(item_id: int = Path(..., ge=1, le=1000)):
    return {"item_id": item_id}
```

### 2.5 Cookie 与 Header 参数

FastAPI 提供`Cookie`和`Header`类，自动处理类型转换和大小写兼容。

```python
from fastapi import FastAPI, Cookie, Header

app = FastAPI()

@app.get("/items/")
async def read_items(
    session_id: str | None = Cookie(default=None),
    user_agent: str | None = Header(default=None)
):
    return {"session_id": session_id, "user_agent": user_agent}
```

### 2.6 响应模型与响应配置

使用`response_model`定义响应结构，自动过滤敏感字段、校验响应格式。

```python
from pydantic import BaseModel, EmailStr

class UserCreate(BaseModel):
    username: str
    password: str
    email: EmailStr

class UserOut(BaseModel):
    username: str
    email: EmailStr

@app.post("/users/", response_model=UserOut, status_code=201)
async def create_user(user: UserCreate):
    return user
```

支持`response_model_exclude_none`、`response_model_include`等配置，精简响应数据。

### 2.7 表单数据处理

处理前端表单提交，需先安装依赖：

```bash
pip install python-multipart
```

```python
from fastapi import FastAPI, Form

app = FastAPI()

@app.post("/login/")
async def login(username: str = Form(...), password: str = Form(...)):
    return {"username": username, "login_status": "success"}
```

### 2.8 文件上传

通过`File`和`UploadFile`处理文件上传，支持单文件、多文件。

```python
from fastapi import FastAPI, File, UploadFile
from typing import List

app = FastAPI()

@app.post("/upload/file/")
async def upload_file(file: bytes = File(...)):
    return {"file_size": len(file)}

@app.post("/upload/uploadfile/")
async def upload_uploadfile(file: UploadFile = File(...)):
    content = await file.read()
    return {"filename": file.filename, "file_size": len(content)}

@app.post("/upload/multi/")
async def upload_multi_files(files: List[UploadFile] = File(...)):
    return {"filenames": [file.filename for file in files]}
```

### 2.9 错误处理

#### 基础 HTTP 异常

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id not in [1,2,3]:
        raise HTTPException(
            status_code=404,
            detail="商品不存在",
            headers={"X-Error": "Item Not Found"}
        )
    return {"item_id": item_id}
```

#### 自定义异常处理器

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

class BusinessException(Exception):
    def __init__(self, code: int, message: str):
        self.code = code
        self.message = message

app = FastAPI()

@app.exception_handler(BusinessException)
async def business_exception_handler(request: Request, exc: BusinessException):
    return JSONResponse(
        status_code=400,
        content={"code": exc.code, "message": exc.message}
    )
```

---

## 三、进阶核心：依赖注入系统

### 3.1 依赖注入核心概念

依赖注入是 FastAPI 的核心灵魂，用于解耦业务逻辑、复用公共代码，支持：

- 嵌套依赖

- 同步 / 异步兼容

- 自动集成到文档

- 资源自动清理

### 3.2 基础依赖示例

#### 函数依赖

```python
from fastapi import FastAPI, Depends

app = FastAPI()

async def common_pagination(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

@app.get("/items/")
async def read_items(pagination: dict = Depends(common_pagination)):
    return pagination
```

#### 类依赖

```python
class CommonQueryParams:
    def __init__(self, skip: int = 0, limit: int = 10, q: str | None = None):
        self.skip = skip
        self.limit = limit
        self.q = q

@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends(CommonQueryParams)):
    return {"skip": commons.skip, "limit": commons.limit}
```

### 3.3 嵌套依赖

依赖可调用其他依赖，形成依赖链，FastAPI 自动解析执行顺序。

```python
async def get_query_token(token: str):
    if token != "my-secret-token":
        raise HTTPException(status_code=401, detail="无效的Token")
    return token

async def get_current_user(token: str = Depends(get_query_token)):
    return {"user_id": 1, "username": "test"}

@app.get("/users/me")
async def read_users_me(current_user: dict = Depends(get_current_user)):
    return current_user
```

### 3.4 路径装饰器依赖

当依赖仅需执行无需返回值时，可在装饰器中声明。

```python
async def verify_admin(token: str):
    if token != "admin-token":
        raise HTTPException(status_code=403, detail="需要管理员权限")

@app.get("/admin/items/", dependencies=[Depends(verify_admin)])
async def read_admin_items():
    return {"message": "管理员专属数据"}
```

### 3.5 全局依赖

为整个应用的所有接口添加依赖，适用于全局日志、认证。

```python
async def verify_global_token(x_token: str = Header(...)):
    if x_token != "global-secret-token":
        raise HTTPException(status_code=401, detail="全局Token无效")

app = FastAPI(dependencies=[Depends(verify_global_token)])
```

### 3.6 带 yield 的依赖：资源管理

使用`yield`实现资源的自动创建和清理，无论接口是否报错都会执行清理逻辑。

```python
async def get_db_session():
    print("创建数据库会话")
    db_session = "模拟的数据库会话"
    try:
        yield db_session
    finally:
        print("关闭数据库会话")

@app.get("/db/items/")
async def read_db_items(db = Depends(get_db_session)):
    return {"db_session": db}
```

---

## 四、路由拆分与项目结构

### 4.1 APIRouter 基础用法

通过`APIRouter`实现路由拆分，模块化管理接口。

#### 步骤 1：创建路由模块

新建`routers/items.py`：

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/items",
    tags=["商品管理"],
    responses={404: {"description": "资源不存在"}},
)

@router.get("/")
async def read_items():
    return [{"item_id": 1, "name": "商品1"}]
```

#### 步骤 2：注册路由

在`main.py`中注册：

```python
from fastapi import FastAPI
from routers import items

app = FastAPI()
app.include_router(items.router)
```

### 4.2 大型项目标准目录结构

FastAPI 官方推荐的生产级目录结构：

```Plain Text
my_fastapi_project/
├── app/
│   ├── __init__.py
│   ├── main.py                   # 应用入口
│   ├── dependencies.py           # 全局依赖
│   ├── config.py                 # 配置文件
│   ├── routers/                  # 路由模块
│   │   ├── __init__.py
│   │   ├── items.py
│   │   └── users.py
│   ├── schemas/                  # Pydantic模型
│   ├── models/                   # ORM模型
│   ├── crud/                     # 数据库CRUD操作
│   ├── core/                     # 核心工具
│   └── tests/                    # 单元测试
├── .env
├── requirements.txt
├── Dockerfile
└── README.md
```

---

## 五、认证与安全

### 5.1 CORS 跨域配置

前后端分离项目中，通过`CORSMiddleware`配置跨域。

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

origins = [
    "http://localhost:3000",
    "https://your-frontend.com",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### 5.2 JWT 认证完整实现

#### 安装依赖

```bash
pip install "python-jose[cryptography]" passlib[bcrypt] python-multipart
```

#### 完整实现

```python
from datetime import datetime, timedelta, timezone
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI(swagger_ui_parameters={"persistAuthorization": True})

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "hashed_password": pwd_context.hash("secret123"),
        "disabled": False,
    }
}

class Token(BaseModel):
    access_token: str
    token_type: str

def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="无效的认证凭证",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    user = fake_users_db.get(username)
    if user is None:
        raise credentials_exception
    return user

@app.post("/token", response_model=Token)
async def login_for_access_token(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    user = fake_users_db.get(form_data.username)
    if not user or not pwd_context.verify(form_data.password, user["hashed_password"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="用户名或密码错误",
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user["username"]}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me/")
async def read_users_me(current_user: Annotated[dict, Depends(get_current_user)]):
    return current_user
```

---

## 六、高级实时通信

### 6.1 核心选型对比

|特性|SSE（Server-Sent Events）|WebSocket|
|---|---|---|
|通信方向|**单向**：仅服务端→客户端推送|**全双工双向**：客户端和服务端可随时互发数据|
|协议基础|纯 HTTP/HTTPS 协议，无需升级握手|基于 TCP，需通过 HTTP 完成协议升级握手|
|数据格式|仅支持 UTF-8 文本，内置标准化事件流格式|支持文本、二进制数据，格式完全自定义|
|重连机制|浏览器**原生内置自动重连**，支持断线续传|需手动实现心跳检测、断线重连、消息补发|
|开发成本|极低，复用现有 HTTP 配置|中等，需单独处理连接管理、心跳|
|适用场景|AI 流式输出、实时日志、消息通知|在线聊天、协同编辑、多人游戏|

**选型原则**：仅需单向推送优先选 SSE，双向交互选 WebSocket。

### 6.2 SSE 完整使用教程

#### 6.2.1 什么是 SSE

SSE 是轻量级实时通信协议，本质是长期保持的 HTTP 长连接，服务端通过`text/event-stream`格式推送数据，浏览器通过原生`EventSource`接收。

#### 6.2.2 环境准备

- FastAPI ≥ 0.112.0：内置 SSE 支持，无需额外依赖

- 低版本：安装`sse-starlette`兼容库

#### 6.2.3 快速入门

服务端代码：

```python
from collections.abc import AsyncIterable
from fastapi import FastAPI, Request
from fastapi.sse import EventSourceResponse
import asyncio

app = FastAPI()

async def event_generator(request: Request) -> AsyncIterable[dict]:
    count = 0
    while True:
        if await request.is_disconnected():
            break
        count += 1
        yield {"data": f"第 {count} 条推送消息", "id": str(count)}
        await asyncio.sleep(1)

@app.get("/sse/stream")
async def sse_stream(request: Request):
    return EventSourceResponse(event_generator(request))
```

前端测试：

```html
<script>
    const eventSource = new EventSource("http://127.0.0.1:8000/sse/stream");
    eventSource.onmessage = (event) => {
        console.log("收到消息", event.data);
    };
</script>
```

#### 6.2.4 核心进阶用法

- **客户端断开检测**：通过`request.is_disconnected()`检测客户端断开，及时终止生成器，避免资源泄漏

- **鉴权认证**：Query 参数携带 Token（兼容 EventSource），或同域场景使用 Cookie

- **断线续传**：通过`Last-Event-ID`请求头，实现断线后补发遗漏消息

#### 6.2.5 生产级实战案例

- **AI 流式输出**：实现 ChatGPT 的打字机效果，逐 Token 推送大模型生成内容

- **实时日志监控**：将服务端日志实时推送到前端，实现在线日志查看

#### 6.2.6 SSE 避坑指南

- **Nginx 配置**：必须关闭`proxy_buffering`，延长`proxy_read_timeout`，避免连接超时

- **连接数限制**：HTTP/1.1 有 6 个并发连接限制，生产环境使用 HTTP/2 突破限制

- **格式规范**：使用`EventSourceResponse`自动处理格式，不要手动拼接

### 6.3 WebSocket 完整使用教程

#### 6.3.1 什么是 WebSocket

WebSocket 是全双工通信协议，通过一次 HTTP 握手升级为 WebSocket 协议，之后可双向实时传输数据，延迟极低。

#### 6.3.2 环境准备

无需额外依赖，FastAPI 原生支持。

#### 6.3.3 快速入门

服务端代码：

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

@app.websocket("/ws/chat")
async def websocket_chat(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            text_data = await websocket.receive_text()
            await websocket.send_text(f"服务端已收到：{text_data}")
    except WebSocketDisconnect:
        print("客户端断开")
```

前端测试：

```html
<script>
    const ws = new WebSocket("ws://127.0.0.1:8000/ws/chat");
    ws.onmessage = (event) => {
        console.log("收到消息", event.data);
    };
</script>
```

#### 6.3.4 核心进阶用法

- **连接管理与广播**：通过连接管理器维护活跃连接，实现群发、广播

- **鉴权认证**：握手阶段通过 Query 参数 / Header 校验 Token，非法连接直接拒绝

- **心跳保活**：30 秒发送一次心跳，避免负载均衡器断开空闲连接

- **房间 / 群组功能**：实现直播间、游戏房间的分组广播

#### 6.3.5 生产级实战：多人在线聊天室

实现完整的多人聊天室，支持登录、群聊、私聊、在线用户列表。

#### 6.3.6 WebSocket 避坑指南

- **Nginx 配置**：开启`Upgrade`和`Connection`头，支持协议升级

- **多实例部署**：使用 Redis Pub/Sub 实现跨实例消息同步，解决广播不同步问题

- **内存泄漏**：客户端断开后及时清理连接，定期巡检无效连接

---

## 七、BaseModel 全参数详解

### 7.1 基础定义

BaseModel 是 Pydantic 的核心类，实现请求 / 响应的自动校验、类型转换、文档生成。

```python
from pydantic import BaseModel, Field

class UserCreate(BaseModel):
    username: str
    age: int = 18
    email: str | None = None
```

### 7.2 字段类型注解

支持所有 Python 原生类型和复合类型：

|类型分类|支持的类型||
|---|---|---|
|基础类型|`str`/`int`/`float`/`bool`||
|可选类型|\`T|None\`|
|容器类型|`list[T]`/`dict[str, T]`/`set[T]`||
|特殊类型|`datetime`/`UUID`/`EmailStr`/`UrlStr`||
|嵌套模型|其他 BaseModel 子类||

### 7.3 Field 字段级核心参数

`Field`用于给字段添加校验规则和元数据，所有配置同步到 Swagger 文档。

#### 7.3.1 必填与默认值

|参数|作用|
|---|---|
|`default`|静态默认值|
|`default_factory`|动态默认值生成函数|
|`...`|占位符，标记字段为必填|

#### 7.3.2 数值校验

|参数|作用|
|---|---|
|`gt`/`ge`|大于 / 大于等于|
|`lt`/`le`|小于 / 小于等于|
|`multiple_of`|必须是该值的倍数|

#### 7.3.3 字符串校验

|参数|作用|
|---|---|
|`min_length`/`max_length`|字符串长度限制|
|`pattern`|正则匹配|
|`strip_whitespace`|自动去除首尾空格|

#### 7.3.4 容器校验

|参数|作用|
|---|---|
|`min_length`/`max_length`|数组元素个数限制|
|`unique_items`|数组元素必须唯一|

#### 7.3.5 文档元数据

|参数|作用|
|---|---|
|`title`/`description`|字段标题和描述|
|`examples`|字段示例值|
|`deprecated`|标记字段废弃|

#### 7.3.6 序列化与别名

|参数|作用|
|---|---|
|`alias`|字段别名，序列化 / 反序列化都生效|
|`validation_alias`|仅反序列化生效的别名|
|`serialization_alias`|仅序列化生效的别名|
|`exclude`|序列化时排除该字段|

### 7.4 模型级配置 model_config

通过`model_config`设置全局模型配置，替代 v1 的`class Config`。

|配置项|作用|
|---|---|
|`extra`|处理额外字段：`ignore`/`forbid`/`allow`|
|`from_attributes`|支持从 ORM 对象转换，替代 v1 的`orm_mode`|
|`populate_by_name`|允许同时使用字段名和别名传参|
|`str_strip_whitespace`|自动去除所有字符串首尾空格|
|`validate_assignment`|赋值时自动校验|

**推荐基础模板**：

```python
class BaseSchema(BaseModel):
    model_config = ConfigDict(
        extra="forbid",
        populate_by_name=True,
        str_strip_whitespace=True,
        validate_assignment=True,
        from_attributes=True
    )
```

### 7.5 核心内置方法

|方法|作用|
|---|---|
|`model_dump()`|转换为字典，替代 v1 的`dict()`|
|`model_dump_json()`|序列化为 JSON 字符串|
|`model_validate()`|校验字典，转换为模型实例|
|`model_copy()`|复制模型实例|

### 7.6 自定义校验器

通过`@field_validator`和`@model_validator`实现复杂自定义校验：

```python
class UserRegister(BaseModel):
    password: str
    confirm_password: str

    @field_validator("confirm_password")
    def check_password_match(cls, value, info: ValidationInfo):
        if value != info.data.get("password"):
            raise ValueError("两次密码不一致")
        return value
```

---

## 八、Swagger UI 完整配置与使用

### 8.1 核心原理

FastAPI 自动扫描接口和模型，生成 OpenAPI 规范的 JSON 文件，Swagger UI 读取该文件渲染交互式文档。

### 8.2 全局配置

创建 FastAPI 实例时配置全局文档信息：

```python
tags_metadata = [
    {"name": "用户管理", "description": "用户相关接口"},
    {"name": "商品管理", "description": "商品相关接口"},
]

app = FastAPI(
    title="电商后台API",
    description="电商后台接口文档，支持Markdown",
    version="1.0.0",
    docs_url="/api-docs",
    redoc_url="/redoc-docs",
    openapi_tags=tags_metadata,
    servers=[
        {"url": "http://127.0.0.1:8000", "description": "开发环境"},
        {"url": "https://api.example.com", "description": "生产环境"},
    ],
    swagger_ui_parameters={
        "persistAuthorization": True,
        "filter": True,
        "displayRequestDuration": True,
    }
)
```

### 8.3 接口级配置

```python
@app.post(
    "/items/{item_id}",
    tags=["商品管理"],
    summary="更新商品信息",
    description="支持Markdown的详细说明",
    response_description="更新成功的商品信息",
    deprecated=False,
    responses={
        200: {"description": "更新成功"},
        404: {"description": "商品不存在"},
    }
)
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.model_dump()}
```

### 8.4 集成鉴权认证

#### Bearer Token 认证

```python
security = HTTPBearer(scheme_name="JWT认证")
```

配置后 Swagger 右上角会出现**Authorize**按钮，输入 Token 后所有接口自动携带认证头。

#### OAuth2 密码流认证

支持在 Swagger 中直接输入用户名密码获取令牌，自动完成认证。

### 8.5 生产环境最佳实践

1. **环境隔离**：生产环境关闭文档，避免对外暴露

    ```python
    is_prod = os.getenv("ENV") == "prod"
    app = FastAPI(
        docs_url=None if is_prod else "/api-docs",
        redoc_url=None if is_prod else "/redoc-docs",
        openapi_url=None if is_prod else "/openapi.json",
    )
    ```

2. **文档访问权限**：给文档添加基础认证，避免未授权访问

3. **文档规范**：所有接口必须添加`tags`、`summary`，字段必须添加说明和示例

---

## 九、数据库集成与接口测试

### 9.1 数据库集成（SQLAlchemy 2.0 异步版）

#### 安装依赖

```bash
pip install sqlalchemy aiosqlite
```

#### 完整实现

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "sqlite+aiosqlite:///./test.db"
engine = create_async_engine(DATABASE_URL, echo=True)
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session
```

实现完整的 CRUD 接口，支持异步数据库操作。

### 9.2 接口测试

使用`TestClient`编写单元测试：

```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_read_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello FastAPI!"}
```

执行测试：

```bash
pytest test_main.py -v
```

---

## 十、部署上线

### 10.1 生产环境部署

#### 官方启动命令

```bash
fastapi run main.py --host 0.0.0.0 --port 8000 --workers 4
```

> Worker 数建议设置为`CPU核心数 * 2 + 1`
>
>

#### Gunicorn \+ UvicornWorker

```bash
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker -b 0.0.0.0:8000
```

### 10.2 Docker 容器化部署

#### Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY ./app /app/app
EXPOSE 8000
CMD ["fastapi", "run", "app/main.py", "--host", "0.0.0.0", "--port", "8000"]
```

#### 构建运行

```bash
docker build -t fastapi-app .
docker run -d --name fastapi-app -p 8000:8000 fastapi-app
```

### 10.3 生产最佳实践

1. 使用 Nginx 作为反向代理，处理静态资源、SSL 证书、负载均衡

2. 生产环境必须使用 HTTPS，敏感配置使用环境变量

3. 使用 Systemd 管理服务，实现开机自启、异常重启

4. 配置结构化日志和健康检查

---

## 十一、最佳实践与常见问题

### 11.1 项目最佳实践

- 严格使用类型提示，充分发挥 FastAPI 的优势

- 按业务拆分模块，避免单文件代码过大

- 所有数据使用 Pydantic 模型统一管理

- 公共逻辑使用依赖注入实现，避免代码重复

- 异步函数必须使用异步库，禁止在 async 中使用阻塞 IO

### 11.2 性能优化

- 使用异步数据库驱动，充分发挥异步性能

- 合理设置 Worker 数，避免过多进程导致上下文切换

- 高频数据使用 Redis 缓存，减少数据库压力

- 大文件使用流式处理，避免占用大量内存

### 11.3 常见问题排查

- **422 校验错误**：检查请求参数是否符合模型定义，查看 detail 字段

- **异步接口慢**：检查是否在 async 函数中使用了阻塞 IO

- **跨域问题**：检查 CORS 配置，生产环境禁止使用`*`通配符

- **Pydantic 版本兼容**：v2 中`orm_mode`改为`from_attributes`，`dict()`改为`model_dump()`

---

## 核心知识点速览

- FastAPI 基于 Starlette 和 Pydantic，支持自动类型校验、自动生成 Swagger 文档，性能比肩 Node.js/Go。

- 依赖注入是 FastAPI 的核心特性，支持嵌套、全局、资源自动清理，用于解耦和复用代码。

- 仅需服务端单向推送数据优先选 SSE，双向交互选 WebSocket，SSE 内置自动重连，开发成本更低。

- BaseModel 是 Pydantic 核心，支持字段校验、别名、自定义校验，所有配置自动同步到 Swagger 文档。

- 生产环境可通过配置关闭 Swagger 文档，或添加基础认证，避免文档对外暴露。

- 长连接（SSE/WebSocket）必须实现心跳保活，同时配置 Nginx 反向代理的超时和协议升级。

- 异步接口必须使用异步库，禁止在 async 函数中使用阻塞 IO，否则会阻塞整个事件循环。

- 大型项目按业务拆分路由，使用官方推荐的目录结构，实现模块化、可维护的工程化开发。

- JWT 是 FastAPI 最常用的认证方式，可无缝集成到 Swagger UI，实现一键认证调试。

- 生产环境推荐使用 Docker 容器化部署，配合 Nginx 反向代理，实现负载均衡、HTTPS 和静态资源托管。
