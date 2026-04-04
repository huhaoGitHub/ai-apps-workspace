# Session 4

该文件用于保存 session-4 通过 PromptArchitect 生成的提示词。

追加规则：

- 新项目追加到文件末尾
- 支持一次处理多个项目，按项目逐条追加
- 若存在语义相近的旧提示词，先询问用户是否保留
- 默认不覆盖已有记录

---

## Q: label-02404

### 项目路径
D:\charles\program\ai\apps\session-4\label-02404

### Prompt
现在项目有 README、API 文档、部署说明和启动脚本，但工程链路还是偏手工。基于现有 Java 17、Maven、Docker、JUnit 5 结构，提出一套更像正式项目的工程化方案，重点看配置分层、测试接入、打包发布、日志，做完之后需要检查对应的改动,确保项目能成功运行

---

## Q: label-02532

### 项目路径
D:\charles\program\ai\apps\session-4\label-02532

### Prompt
这个博客有 backend、admin、user 三端，Maven 和 Vite 各自独立构建，没有统一版本号约定，前端环境变量也是各自硬写在 vite.config.js 里。请按现有 pom.xml 和两个前端目录，梳理一套多端构建工程化方案，重点在版本管理、构建参数统一和构建产物命名规范，不要改业务逻辑。

---

## Q: label-02429

### 项目路径
D:\charles\program\ai\apps\session-4\label-02429

### Prompt
秒杀系统已经有并发压测脚本，但边界相关核心场景基本没有覆盖。按现有 backend/src/test 和 tests/api_concurrency_test.py，帮我想一套测试工程化思路，说清楚单测、集成测试和并发测试各自负责哪些路径，以及怎么验证防超卖逻辑是真的有效而不是靠事务运气。

---

## Q: label-02436

### 项目路径
D:\charles\program\ai\apps\session-4\label-02436

### Prompt
任务管理系统 common/ 里已经有 AOP 日志拦截，但格式不统一，出问题了追调用链很费劲。按现有 backend/src 和 logback-spring.xml，帮我把日志工程化梳理一下，重点放在格式标准化、分级输出和异常堆栈追踪，还有本地开发和生产怎么分开配置，不要动业务代码。

---

## Q: label-02465

### 项目路径
D:\charles\program\ai\apps\session-4\label-02465

### Prompt
账户余额系统后端本地走 H2，容器里切 MySQL，但 dev 和 prod 配置基本堆在一个 application.yml 里，前端 API 地址也是写死的。按 backend/src/main/resources 和 frontend-admin 现有结构，帮我把多环境配置分层梳理一下，让本地开发、集成自测和生产能各拿各的配置，互不干扰。

---

## Q: label-02470

### 项目路径
D:\charles\program\ai\apps\session-4\label-02470

### Prompt
这个学生管理系统有 Spring Boot 3、JPA、MySQL、JWT 后端和 Vue3 管理端，接口覆盖学生、班级、课程、成绩，登录后走 Bearer Token。请按现有前后端目录设计一套测试任务，重点放在登录鉴权、核心 CRUD、筛选查询、成绩录入编辑和 401 场景，顺手补少量可自动化的接口测试思路
