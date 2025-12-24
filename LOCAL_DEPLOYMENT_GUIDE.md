# novel-plus 本地环境部署攻略

本文档将指导您如何在本地环境中成功部署 `novel-plus` 项目。

## 1. 环境准备

在开始之前，请确保您的开发环境中已经安装并正确配置了以下软件。所有软件建议使用推荐版本以避免不必要的兼容性问题。

* **JDK 21**:
    * 项目基于 Java 21 构建。请确保您的 `JAVA_HOME` 环境变量已正确设置，并且 `java -version` 命令可以输出 Java 21 的版本信息。
* **Apache Maven**:
    * 项目使用 Maven 进行构建和依赖管理。请确保 `mvn -version` 命令可以正常工作。
* **MySQL 8**:
    * 项目使用 MySQL 作为主数据库。建议使用 `8.0` 或更高版本。
* **Redis**:
    * 项目使用 Redis 进行数据缓存。请确保 Redis 服务已安装并正在运行。

## 2. 数据库配置

项目使用 MySQL 数据库，并依赖 `ShardingSphere-JDBC` 进行分表。

1. **创建数据库**:
   使用任何 MySQL 客户端（如 Navicat, DataGrip, or `mysql` 命令行）连接到您的数据库服务器，并执行以下 SQL 命令来创建一个名为 `novel_plus` 的数据库。

    ```sql
    CREATE
    DATABASE novel_plus DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    ```

2. **导入数据**:
   数据库创建成功后，您需要导入项目的初始表结构和数据。SQL 脚本位于项目根目录的 `doc/sql/` 文件夹下。

    * 找到并执行 `novel_plus.sql` 文件。这个全量脚本会创建所有必需的表，包括 `book_content` 的分表（`book_content0` 到 `book_content9`），并插入初始数据。

   **注意**: `doc/sql/` 目录下的其他 `yyyyMMdd.sql` 文件是用于版本升级的增量脚本，首次部署时无需执行。

## 3. 应用程序配置

为了让应用程序能正确连接到数据库和缓存，您需要修改以下配置文件。

### 3.1 数据库连接配置

数据库连接信息由 ShardingSphere-JDBC 管理，其配置文件位于项目根目录下的 `config/shardingsphere-jdbc.yml`。

打开 [`config/shardingsphere-jdbc.yml`](config/shardingsphere-jdbc.yml:1) 文件，找到 `dataSources` 部分，并根据您的本地 MySQL 环境修改 `ds_1` 和 `ds_2` 的配置：

```yaml
dataSources:
  ds_1:
    # ... 其他配置 ...
    jdbcUrl: jdbc:mysql://localhost:3306/novel_plus?... # 修改主机和端口
    username: root        # 修改为您的数据库用户名
    password: test123456  # 修改为您的数据库密码
  ds_2:
    # ... 其他配置 ...
    url: jdbc:mysql://localhost:3306/information_schema?... # 修改主机和端口
    username: root        # 修改为您的数据库用户名
    password: test123456  # 修改为您的数据库密码
```

**特别说明**: `ds_1` 是主业务数据库，`ds_2` 在这里用于访问 `information_schema`，通常只需要确保连接信息与 `ds_1` 一致即可。

### 3.2 Redis 连接配置

Redis 的连接信息位于 `novel-common` 模块中。

打开 [`novel-common/src/main/resources/application-common-dev.yml`](novel-common/src/main/resources/application-common-dev.yml:1) 文件，修改 `spring.data.redis` 部分的配置：

```yaml
      spring:
        data:
          redis:
            host: 127.0.0.1          # 修改为您的 Redis 主机地址
            port: 6379              # 修改为您的 Redis 端口
            password: test123456      # 修改为您的 Redis 密码（如果没有则留空）
```

## 4. 第三方服务与安全配置

为了保证所有功能正常运行，特别是支付、文件上传和 AI 相关功能，您需要配置相关的第三方服务密钥和安全凭证。

**强烈建议** : 为了安全起见，不要将生产环境的密钥直接硬编码在配置文件中。最佳实践是使用环境变量或专门的配置中心来管理这些敏感信息。

### 4.1 Spring AI (OpenAI)

AI 写作功能依赖于 OpenAI 或兼容的 API。

* **文件** : [ `novel-front/src/main/resources/application.yml` ](novel-front/src/main/resources/application.yml:1)
* **修改项** :
    * `spring.ai.openai.api-key`: 替换为您自己的 API 密钥。
    * `spring.ai.openai.base-url`: 如果您使用代理或第三方服务，请修改此 URL。

### 4.2 阿里云对象存储 (OSS)

当图片存储方式配置为 OSS 时，需要配置以下信息。

* **文件** : [ `novel-front/src/main/resources/application-oss.yml` ](novel-front/src/main/resources/application-oss.yml:1)
* **修改项** :
    * `novel.file.endpoint`: 您的 OSS 服务地址。
    * `novel.file.key-id`: 您的 AccessKey ID。
    * `novel.file.key-secret`: 您的 AccessKey Secret。
    * `novel.file.bucket-name`: 您创建的 Bucket 名称。
    * `novel.file.web-url`: 您的 Bucket 的公开访问域名。

### 4.3 支付宝支付

支付功能需要配置您的支付宝开发者账户信息。

* **文件** : [ `novel-front/src/main/resources/application-alipay.yml` ](novel-front/src/main/resources/application-alipay.yml:1)
* **修改项** :
    * `alipay.app-id`: 您的应用 AppId。
    * `alipay.merchant-private-key`: 您的应用私钥。
    * `alipay.public-key`: 您的支付宝公钥。
    * `alipay.notify-url`: 用于接收异步通知的公网可访问地址。本地调试时，可以使用内网穿透工具（如 go-http、ngrok）生成一个临时公网地址。
    * `alipay.return-url`: 支付成功后跳转的前台页面地址。

### 4.4 网站信息

您可以自定义网站的基本信息。

* **文件** : [ `novel-front/src/main/resources/application-website.yml` ](novel-front/src/main/resources/application-website.yml:1)
* **修改项** :
    * `website.name`: 网站名称。
    * `website.domain`: 网站域名。
    * `website.keyword`: SEO 关键词。
    * `website.description`: 网站描述。

### 4.5 JWT 安全密钥

为了保障用户认证安全，请务必修改默认的 JWT 密钥。

* **文件** : [ `novel-front/src/main/resources/application.yml` ](novel-front/src/main/resources/application.yml:1)
* **修改项** :
    * `jwt.secret`: 替换为一个足够复杂和随机的字符串作为您的签名密钥。

## 5. 编译与启动

完成所有配置后，您可以通过 Maven 来编译和启动项目。

### 5.1 编译打包

        首先，在项目根目录下执行 Maven 打包命令。这将编译所有模块并生成可执行的 JAR 文件。

```bash
mvn clean package
```

### 5.2 启动应用程序

`novel-plus` 系统主要由以下几个可独立启动的模块构成：

* **`novel-front` (前台门户)**: 核心的用户访问界面。
* **`novel-admin` (后台管理)**: 用于管理整个平台。
* **`novel-crawl` (爬虫服务)**: 用于抓取小说内容。

您可以根据需要选择启动一个或多个模块。

#### 启动 `novel-front` (前台门户)

```bash
java -jar novel-front/target/novel-front.jar
```

服务启动后，默认监听在 `8083` 端口。

#### 启动 `novel-admin` (后台管理)

`novel-admin` 是一个独立的 Spring Boot 项目，拥有自己的依赖和配置。

```bash
java -jar novel-admin/target/novel-admin.jar
```

该模块启动后，默认监听在 `8085` 端口。

#### 启动 `novel-crawl` (爬虫服务)

```bash
java -jar novel-crawl/target/novel-crawl.jar
```

该模块启动后，会根据配置执行爬虫任务，通常没有对外的 Web 端口。

## 6. 访问与验证

当相关服务启动后，您可以通过以下方式访问系统并验证部署是否成功。

### 6.1 访问前台门户

* **地址**: `http://localhost:8083`
* **验证**: 打开浏览器访问该地址，如果能看到小说网站的首页，则表明 `novel-front` 模块已成功启动。您可以尝试注册、登录、浏览小说等操作。

### 6.2 访问后台管理系统

* **地址**: `http://localhost:8085`
* **验证**:
    * 打开浏览器访问该地址，您将看到后台登录页面。
    * 根据 `doc/sql/novel_plus.sql` 脚本中的数据，默认的管理员账户信息如下：
        * **用户名**: `admin`
        * **密码**: `123456`
    * 使用该账户登录，如果能成功进入后台管理仪表盘，则表明 `novel-admin` 模块已成功部署。

### 6.3 检查日志

在启动各个模块时，请密切关注控制台输出的日志。如果出现错误，特别是数据库连接错误、Redis 连接错误或配置相关的异常，请根据错误信息回顾之前的配置步骤，确保所有信息都已正确填写。

恭喜！到此为止，您已成功在本地部署了 `novel-plus` 项目。
