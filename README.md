# NovaOps

NovaOps 继承自 open-capacity-platform，并将整体技术栈升级到 JDK 21（未来目标 JDK 25）。项目致力于提供开源、可审计、可持续维护的企业级微服务能力开放平台，避免任何商业软件绑定或与开源精神冲突的内容。

## 项目目标
- 保持与社区主流开源生态兼容，持续更新至新的 LTS JDK 版本。
- 提供统一认证、网关管控、监控观测、任务调度等通用能力，降低企业微服务建设成本。
- 通过开源协作模式推进质量、安全与可维护性。

## 技术栈（当前规划）
- **JDK**: 21（计划演进至 25）
- **框架**: Spring Boot、Spring Cloud & Spring Cloud Alibaba
- **构建**: Maven 3.9+
- **数据库与中间件**: 以开源实现为优先（如 MySQL、PostgreSQL、Redis、Kafka 等），不绑定商业闭源产品。
- **部署**: 容器化（Docker/Kubernetes）、可选基础设施即代码

## 模块概览
- `register-center`：注册中心服务
- `new-api-gateway`：API 网关
- `oauth-center`：统一认证与授权
- `business-center`：业务示例与通用能力
- `monitor-center`：链路追踪与指标采集
- `job-center`：分布式调度与任务编排
- `web-portal`：前端门户
- `inner-intergration`、`tuning-center` 等：运维与内部集成能力

## 快速开始
1. 安装 JDK 21 与 Maven 3.9+。
2. 克隆仓库并拉取最新依赖：
   ```bash
   mvn -version
   git clone <repository-url>
   cd NovaOps
   mvn clean install -DskipTests
   ```
3. 根据需要启动核心服务（注册中心 → 网关 → 认证中心 → 业务服务 → 监控与调度）。详细的服务配置与环境变量可参考各模块的 `README` 或 `application` 配置文件。

## 开源与合规
- 本项目采用 Apache License 2.0，并遵循开源协作原则，严禁夹带广告、商业推广或限制性授权条款。
- 依赖选择优先使用开源许可组件，避免对商业闭源软件的强制绑定。若需可选的商业集成，请以插件化方式在独立仓库维护。

## 贡献指南
- 贡献流程、代码与架构规范见 [开发规范](doc/development-guidelines.md)。
- 质量保障流程见 [测试规范](doc/testing-guidelines.md)。
- 欢迎通过 Issue/PR 反馈问题与建议。

## 致谢
感谢 open-capacity-platform 的原始贡献者与社区，为 NovaOps 的延续与升级奠定了基础。

## 许可证
本项目遵循 [Apache License 2.0](LICENSE)。
