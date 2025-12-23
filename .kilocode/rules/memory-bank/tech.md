# 技术栈与环境 (tech.md)

`novel-plus` 是一个基于 Java 和 Spring Boot 的现代化 Web 应用程序，采用了一系列成熟的技术来支持其功能。

## 开发环境

*   **Java 版本**: `Java 21`
*   **项目构建**: `Apache Maven`
*   **核心框架**: `Spring Boot 3.4.0` (后台管理模块 `novel-admin` 使用独立的 `Spring Boot 2.7.18`)

## 核心技术

*   **后端**
    *   **Web 框架**: `Spring MVC`
    *   **数据持久化**:
        *   `MyBatis`: SQL 映射框架。
        *   `MyBatis Dynamic SQL`: 用于动态生成 SQL。
        *   `PageHelper`: MyBatis 的物理分页插件。
    *   **数据库**: `MySQL 8.0`
    *   **分库分表**: `Apache ShardingSphere`
    *   **缓存**: `Redis` (通过 `Redisson` 和 `spring-boot-starter-data-redis` 集成)
    *   **安全与认证**:
        *   `Apache Shiro`: 用于后台管理系统的权限控制。
        *   `JSON Web Tokens (JWT)`: 用于前台门户的用户认证。
    *   **AI 集成**: `Spring AI` (特别是与 `OpenAI` 的集成)
    *   **模板引擎**: `Thymeleaf`

*   **前端**
    *   **核心**: 原生 `JavaScript`，`jQuery`
    *   **UI 框架**: `Layui`

## 关键第三方库与服务

*   **云存储**: 阿里云 `OSS`
*   **支付**: 支付宝 `Alipay SDK`
*   **工具库**:
    *   `Apache Commons Lang3`: 通用工具类。
    *   `Lombok`: 简化 Java 代码。
    *   `Jackson`: JSON 处理。
    *   `ip2region`: IP 地址定位。

## 构建与部署

*   项目通过 `maven-antrun-plugin` 插件打包，生成包含可执行 JAR 文件、配置文件、启动脚本 (`.sh`, `.bat`) 和 `Dockerfile` 的部署包。
*   系统支持多主题切换，模板文件和静态资源在构建时会被自动复制和组织。
*   数据库结构通过 `doc/sql` 目录下的 SQL 脚本进行管理。