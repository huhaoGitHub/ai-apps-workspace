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
现在这个博客项目有 backend、admin、user 三个服务，README 里本地启动和 docker-compose 启动都给了，但整体工程链路还是偏散。请基于现有 Spring Boot、Vue3、Vite、Docker、Nginx 结构，补一套更稳的工程化要求，重点看环境配置分层、三端构建发布一致性、服务启动顺序和部署前后校验，不要脱离当前目录结构重来。

---

## Q: label-02429

### 项目路径
D:\charles\program\ai\apps\session-4\label-02429

### Prompt
这个秒杀项目现在是 Spring Boot、JPA、MySQL 和 docker-compose 的单后端结构，README 里已经有 dev/prod profile 说明和并发测试脚本。别重写业务，按现有 backend、tests 和容器编排补一套工程化方案，重点把环境分层、构建缓存、测试接入、日志规范和 CI 门禁梳理清楚。

---

## Q: label-02436

### 项目路径
D:\charles\program\ai\apps\session-4\label-02436

### Prompt
这个任务管理系统后端已经是 Spring Boot、MyBatis-Plus、MySQL，README 里把单测、集成验证和 docker-compose 都写出来了。别推翻重来，基于现有 backend、db 初始化脚本和 src/test 结构，补一套更像正式交付项目的工程化方案，重点看配置分层、测试门禁、镜像构建、日志级别和 CI 流程。

---

## Q: label-02465

### 项目路径
D:\charles\program\ai\apps\session-4\label-02465

### Prompt
这个账户余额系统已经有 Spring Boot 3、Sa-Token、Vue3 管理端、Nginx 和 docker-compose，后端本地默认走 H2，容器里走 MySQL。不要重写功能，按现有 backend、frontend-admin、schema.sql 和 nginx 配置，整理一套工程化补齐方案，把环境切换、前后端构建发布、一致性校验、日志和部署回滚想清楚。

---

## Q: label-02470

### 项目路径
D:\charles\program\ai\apps\session-4\label-02470

### Prompt
这个学生管理系统有 Spring Boot 3、JPA、MySQL、JWT 后端和 Vue3 管理端，接口覆盖学生、班级、课程、成绩，登录后走 Bearer Token。请按现有前后端目录设计一套测试任务，重点放在登录鉴权、核心 CRUD、筛选查询、成绩录入编辑和 401 场景，顺手补少量可自动化的接口测试思路
