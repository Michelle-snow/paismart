
# 派聪明项目开发规范指南

为保证代码质量、可维护性、安全性与可扩展性，请在开发过程中严格遵循以下规范。

## 一、项目基本信息

### 用户工作目录
- **路径**：`D:\tools\IDEA2024_3.3\code\PaiSmart-main`
- **操作系统**：Windows 11
- **开发时间**：2026-03-02 15:13:20
- **项目作者**：xue

### 技术栈要求

- **主框架**：Spring Boot 3.4.2
- **语言版本**：Java 17
- **核心依赖**：
  - `spring-boot-starter-web`
  - `spring-boot-starter-data-jpa`
  - `spring-boot-starter-security`
  - `spring-boot-starter-data-redis`
  - `spring-boot-starter-websocket`
  - `lombok`
  - `jjwt-api` (JWT支持)
  - `elasticsearch-java` (搜索引擎支持)
  - `minio` (对象存储)
  - `spring-kafka` (消息队列)
  - `hanlp` (中文分词)

### 构建工具
- **Maven**：3.x
- **Spring Boot Maven Plugin**：用于构建和打包

## 二、项目目录结构

```
PaiSmart-main/
├── .qoder/                 # Qoder配置
├── docs/                   # 文档目录
│   └── databases/          # 数据库文档
├── frontend/               # 前端项目
│   ├── packages/          # 共享包
│   │   ├── alova/        # Alova请求库
│   │   ├── axios/        # Axios请求库
│   │   ├── color/        # 颜色工具
│   │   ├── hooks/        # 共享Hook
│   │   ├── materials/    # UI组件库
│   │   ├── ofetch/       # Ofetch请求库
│   │   ├── scripts/      # 共享脚本
│   │   ├── uno-preset/   # UnoCSS预设
│   │   └── utils/       # 共享工具
│   ├── public/           # 静态资源
│   └── src/             # 源代码
│       ├── assets/      # 资源文件
│       ├── components/  # 组件
│       ├── layouts/     # 布局
│       ├── router/      # 路由
│       ├── service/     # 服务
│       ├── store/       # 状态管理
│       ├── styles/      # 样式
│       ├── theme/       # 主题
│       ├── typings/     # 类型定义
│       ├── utils/       # 工具函数
│       └── views/       # 页面视图
├── homepage/             # 首页项目
└── src/                 # 后端项目
    ├── main/
    │   ├── java/com/yizhaoqi/smartpai/  # Java源代码
    │   │   ├── client/          # 客户端配置
    │   │   ├── config/          # 配置类
    │   │   ├── consumer/        # Kafka消费者
    │   │   ├── controller/      # 控制器
    │   │   ├── entity/          # 实体类
    │   │   ├── exception/       # 异常处理
    │   │   ├── handler/         # 处理器
    │   │   ├── model/           # 数据模型
    │   │   ├── repository/      # 数据仓库
    │   │   ├── service/         # 服务层
    │   │   ├── test/           # 测试类
    │   │   └── utils/         # 工具类
    │   └── resources/         # 资源文件
    │       ├── es-mappings/   # Elasticsearch映射
    │       └── static/        # 静态资源
    └── test/                 # 测试代码
        └── java/com/yizhaoqi/smartpai/
            ├── service/       # 服务测试
            └── utils/         # 工具测试
```

## 三、分层架构规范

| 层级        | 职责说明                         | 开发约束与注意事项                                               |
|-------------|----------------------------------|----------------------------------------------------------------|
| **Controller** | 处理 HTTP 请求与响应，定义 API 接口 | 不得直接访问数据库，必须通过 Service 层调用                  |
| **Service**    | 实现业务逻辑、事务管理与数据校验   | 必须通过 Repository 层访问数据库；返回 DTO 而非 Entity（除非必要） |
| **Repository** | 数据库访问与持久化操作             | 继承 `JpaRepository`；使用 `@EntityGraph` 避免 N+1 查询问题     |
| **Entity**     | 映射数据库表结构                   | 不得直接返回给前端（需转换为 DTO）；包名统一为 `entity`         |

### 接口与实现分离

- 所有接口实现类需放在接口所在包下的 `impl` 子包中。

## 四、安全与性能规范

### 输入校验

- 使用 `@Valid` 与 JSR-303 校验注解（如 `@NotBlank`, `@Size` 等）
  - 注意：Spring Boot 3.x 中校验注解位于 `jakarta.validation.constraints.*`

- 禁止手动拼接 SQL 字符串，防止 SQL 注入攻击。

### 事务管理

- `@Transactional` 注解仅用于 **Service 层**方法。
- 避免在循环中频繁提交事务，影响性能。

### 安全配置

- 使用 Spring Security 进行身份认证和授权
- JWT 令牌用于 API 认证
- 密码加密存储（BCrypt）

### 缓存策略

- 使用 Redis 作为缓存中间件
- 合理设置缓存过期时间
- 避免缓存雪崩和穿透

## 五、代码风格规范

### 命名规范

| 类型       | 命名方式             | 示例                  |
|------------|----------------------|-----------------------|
| 类名       | UpperCamelCase       | `UserServiceImpl`     |
| 方法/变量  | lowerCamelCase       | `saveUser()`          |
| 常量       | UPPER_SNAKE_CASE     | `MAX_LOGIN_ATTEMPTS`  |

### 注释规范

- 所有类、方法、字段需添加 **Javadoc** 注释，使用中文注释。
- 关键业务逻辑需添加行内注释说明。

### 类型命名规范（阿里巴巴风格）

| 后缀 | 用途说明                     | 示例         |
|------|------------------------------|--------------|
| DTO  | 数据传输对象                 | `UserDTO`    |
| DO   | 数据库实体对象               | `UserDO`     |
| BO   | 业务逻辑封装对象             | `UserBO`     |
| VO   | 视图展示对象                 | `UserVO`     |
| Query| 查询参数封装对象             | `UserQuery`  |

### 实体类简化工具

- 使用 Lombok 注解替代手动编写 getter/setter/构造方法：
  - `@Data`
  - `@NoArgsConstructor`
  - `@AllArgsConstructor`

## 六、扩展性与日志规范

### 接口优先原则

- 所有业务逻辑通过接口定义（如 `UserService`），具体实现放在 `impl` 包中（如 `UserServiceImpl`）。

### 日志记录

- 使用 `@Slf4j` 注解代替 `System.out.println`
- 日志级别使用规范：
  - `DEBUG`：调试信息
  - `INFO`：重要业务信息
  - `WARN`：警告信息
  - `ERROR`：错误信息

## 七、编码原则总结

| 原则       | 说明                                       |
|------------|--------------------------------------------|
| **SOLID**  | 高内聚、低耦合，增强可维护性与可扩展性     |
| **DRY**    | 避免重复代码，提高复用性                   |
| **KISS**   | 保持代码简洁易懂                           |
| **YAGNI**  | 不实现当前不需要的功能                     |
| **OWASP**  | 防范常见安全漏洞，如 SQL 注入、XSS 等      |

## 八、特殊配置说明

### 数据库配置

- **MySQL**：主数据库，使用 `com.mysql.cj.jdbc.Driver`
- **JPA**：`ddl-auto: update`，开发环境自动更新表结构

### 缓存配置

- **Redis**：本地缓存，端口 6379
- **文件上传**：最大 50MB，请求最大 100MB

### 消息队列

- **Kafka**：本地运行，端口 9092
- **事务支持**：启用幂等性，重试 3 次

### 对象存储

- **MinIO**：本地运行，端口 9000
- **存储桶**：`uploads`

### AI集成

- **DeepSeek API**：用于智能对话
- **阿里云DashScope**：用于文本嵌入
- **HanLP**：用于中文文本分词

### 搜索引擎

- **Elasticsearch**：本地运行，端口 9200
- **HTTPS**：启用安全连接
