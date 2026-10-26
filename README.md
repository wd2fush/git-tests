# 联通用户管理系统（ems）

基于 **Django 4.2 + MySQL** 的入门级后台管理系统，服务端渲染（Django 模板 + Bootstrap 3），未做前后端分离。包含**部门管理、用户管理、靓号管理**三大模块。

## 技术栈

| 分类 | 说明 |
|---|---|
| 语言 / 框架 | Python 3.12、Django 4.2 |
| 数据库 | MySQL（库名 `ems`），驱动 mysqlclient |
| 前端 | Bootstrap 3.4.1、jQuery、bootstrap-datepicker（中文日期插件） |

## 功能模块

### 部门管理 `/depart/`
- 列表、新增（两种模板写法）、编辑、删除

### 用户管理 `/user/`
- 列表（`select_related` 预加载部门，避免 N+1 查询）
- 新增两种写法：方式一手写表单手动取值；方式二 ModelForm 自动校验并保存
- 编辑（ModelForm 自动回显）、删除
- 「入职时间」绑定 bootstrap-datepicker 中文日历，格式 `yyyy-mm-dd`

### 靓号管理 `/pretty/`
- 列表：号码模糊搜索（`mobile__contains`）+ 分页
- **分页标准**：每页 10 条，前闭后开——第 1 页取第 1~10 条（切片 `[0:10)`），第 2 页取 `[10:20)`，以此类推
- 分页条由通用组件 `utils/pagination.py` 生成：首页 / 上一页 / 数字页码（当前页高亮）/ 下一页 / 尾页 / 页码跳转输入框，翻页时自动保留搜索词
- 新建、编辑（ModelForm）、删除（POST + CSRF + `confirm` 二次确认）

## 项目结构

```
ems/
├── manage.py                  # Django 管理入口
├── create_test_data.py        # 靓号测试数据脚本（可重复执行，按号码去重）
├── ems/                       # 项目配置目录
│   ├── settings.py            # 数据库、已注册应用等配置
│   └── urls.py                # 总路由（depart / user / pretty 三组）
└── app01/                     # 业务应用
    ├── models.py              # Department / UserInfo / PrettyNum 三张表
    ├── utils/                 # 通用工具箱（与具体业务无关）
    │   ├── bootstrap.py       # BootStrapModelForm：统一表单 Bootstrap 样式
    │   └── pagination.py      # Pagination：通用分页组件（切片 + 分页条 HTML）
    ├── views/                 # 视图按功能拆分为子模块
    │   ├── depart.py          # 部门管理视图
    │   ├── user.py            # 用户管理视图（含 UserModelForm）
    │   └── pretty.py          # 靓号管理视图（含 PrettyNumModelForm）
    ├── templates/             # 模板（layout.html 为公共布局，其余页面继承）
    └── static/                # 静态资源（bootstrap、datepicker、jquery）
```

## 数据模型

| 模型 | 字段 |
|---|---|
| `Department` 部门 | title |
| `UserInfo` 用户 | name、password、age、account、create_time、gender(1男/2女)、depart(外键，级联删除) |
| `PrettyNum` 靓号 | mobile、price、level(1~4级)、status(1已占用/2未占用) |

## 快速开始

```bash
# 1. 安装依赖
pip install django==4.2 mysqlclient

# 2. 在 MySQL 中建库
CREATE DATABASE ems DEFAULT CHARSET utf8mb4;

# 3. 修改数据库账号密码（ems/settings.py 的 DATABASES）

# 4. 执行迁移建表
python manage.py migrate

# 5.（可选）录入 11 条靓号测试数据
python create_test_data.py

# 6. 启动开发服务器
python manage.py runserver
```

启动后访问：

- 部门管理：<http://127.0.0.1:8000/depart/list/>
- 用户管理：<http://127.0.0.1:8000/user/list/>
- 靓号管理：<http://127.0.0.1:8000/pretty/list/>

## 已知问题（学习项目，仅限本地练习）

- `settings.py` 中 `DEBUG=True`，`SECRET_KEY` 与数据库口令为明文，不可用于生产
- `PrettyNum.mobile` 未加唯一约束，同一号码可重复录入
- 部门、用户的删除仍是 GET 链接，有误删风险（靓号删除已改为 POST）
- 未实现登录认证，导航栏「登录」为占位链接

