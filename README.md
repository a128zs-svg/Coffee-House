# Coffee House 智能自提点餐客户端 (Android)

## 📌 项目概述

本项目为移动应用开发课程设计电子作品。

`Coffee House` 是一款专为校园及周边社区打造的智能自提点餐系统。项目采用前后端分离的**双端闭环架构**：
- **移动客户端 (Android)**：基于原生 Android SDK (Java + XML) 进行构建，采用现代化且符合行业标准的 **MVVM (Model-View-ViewModel)** 设计模式 [3]；
- **商户管理端与服务端 (Web & RESTful API)**：基于 **Spring Boot** 与 **MyBatis** 框架提供轻量级高并发支持，采用 **MySQL** 进行数据持久化，并配有传统的 Web 管理端网页供商家进行订单流转与出餐处理。

项目打通了商品陈列、自适应购物车、独立的自提结算表单、订单自提流水号哈希生成，以及微信 Open SDK 好物分享的全业务闭环。

---

## 🛠️ 技术选型与技术亮点

### 1. 技术栈

| 层次 | 选型 | 说明 |
| :--- | :--- | :--- |
| **客户端前端** | Android SDK (Java, JDK 1.8) | 原生布局架构，采用 XML 绘制极简留白风格商业界面 |
| **网络层框架** | Retrofit 2.9 + OkHttp 3 / Gson | 异步非阻塞网络库，自动管理及解析 JSON 实体数据 |
| **架构设计模式** | **MVVM (LiveData & ViewModel)** [3] | 视图与业务逻辑完全解耦，利用观察者模式实时驱动数据渲染 |
| **图片加载库** | Glide 4.16 | 高效的图片缓存与异步异步网络加载框架 |
| **第三方 SDK** | 微信 Open SDK 6.8 | 深度集成微信开放平台，支持在订单详情页直连分享至微信好友 |
| **服务端后端** | Spring Boot + WebMvcConfigurer | 提供 RESTful API 并通过资源处理器实现物理图片磁盘映射 |
| **持久层框架** | MyBatis (XML 映射器) | 动态 SQL 编写，实现 SQL 层面的高并发安全库存原子扣减 |
| **关系型数据库**| MySQL 8.x | 关系型关系设计，配置多表联合查询映射（Association/Collection） |

### 2. 核心技术亮点

- **跨端 Session 会话保持拦截器**：客户端通过重写 OkHttpClient 的拦截器（Interceptor），自动提取响应头中的 `Set-Cookie`（`JSESSIONID`）并作本地 SharedPreference 存储，在后续请求中自动添加 `Cookie` 头部，模拟浏览器级的登录保持。
- **多端状态实时校验与优雅降级**：客户端对后端网络请求的 `401` 状态码（Session会话过期）进行拦截。一旦拦截成功，自动清空本地过期的登录缓存，并提示“登录已失效”强制跳转回登录页，保证极高的运行健壮性。
- **Android 11+ 包可见性声明与 SDK 防崩溃防御**：针对高版本 Android 系统，在 `AndroidManifest.xml` 中配置 `<queries>` 标签声明对微信包名的查询可见性，成功突破系统级限制。同时，对微信分享 SDK 的注册及调起增加 `try-catch` 异常安全防线，在未装微信或签名不匹配时优雅退回精美模拟预览，保障 App 永不闪退。
- **本地订单备注反显闭环**：在订单表缺少备注字段的情况下，利用 SharedPreferences 以生成的 `orderId` 为唯一标识符将结算备注存储在本地。当跳转进入订单详情页时，自动读取并反显，形成完美的业务闭环。
- **自提号哈希生成算法**：本地在订单支付成功后，通过对后端返回的 `orderId`（UUID）执行哈希转换，取末尾2位数字拼接成取餐号（如 `A-140`），既防止泄露订单真实编号，又保证同一笔订单每次打开时展示的自提号完全一致。

---

## 📦 项目目录结构 (Android端)

```text
cn.jxufe.iet.code/
│
├── adapter/
│   ├── ProductAdapter.java       # 首页商品 RecyclerView 适配器
│   ├── CartAdapter.java          # 购物车加减/点击数字修改适配器
│   └── OrderAdapter.java         # 历史订单卡片 RecyclerView 适配器
│
├── api/
│   └── ApiService.java           # Retrofit 网络请求端点声明（Login/Register/Cart/Order）
│
├── entity/
│   ├── User.java                 # 用户实体
│   ├── Product.java              # 咖啡商品实体
│   ├── Cart.java                 # 购物车商品（包含 Product 嵌套）实体
│   ├── CartData.java             # 购物车总价与列表响应实体
│   ├── Order.java                # 订单主表实体
│   └── OrderItem.java            # 订单明细（包含 Product 嵌套）实体
│
├── fragment/
│   ├── HomeFragment.java         # 首页 Tab 选项卡与商品渲染控制器
│   ├── CartFragment.java         # 购物车数量实时修改与空状态控制器
│   └── ProfileFragment.java      # 个人中心（登录检测、我的订单、设置去向）控制器
│
├── viewmodel/
│   ├── HomeViewModel.java        # 首页数据驱动视图模型
│   └── CartViewModel.java        # 购物车更新/删除状态观察模型
│
├── utils/
│   ├── ApiClient.java            # Retrofit 全局实例化、Cookie 拦截器、IP 地址管理
│   └── Result.java               # 全局服务端通用 JSON 返回包装类
│
├── MainActivity.java             # APP 主页（包含 BottomNavigationView 碎片管理）
├── LoginActivity.java            # 登录逻辑主页面
├── RegisterActivity.java         # 注册逻辑主页面
├── SettingsActivity.java         # 个人信息设置编辑主页面（修改昵称、绑定手机号、登出）
├── ChangePasswordActivity.java   # 原密码验证与新密码修改页面
├── OrderSettlementActivity.java  # 自提结算、模拟收银台、订单提交页面
├── MyOrdersActivity.java         # 我的历史订单列表展示页面
└── OrderDetailActivity.java      # 订单详情、动态项填充、自提流水号、微信直连分享页面
