# 技术栈和选型

## 1.0 DDD
> DDD（领域驱动设计）是一个软件开发方法论，强调在复杂软件项目中以领域模型为中心进行设计和开发。

你可能会问，那么 ML-GPT 这种文本生成项目何为领域中心？<br/>

我的回答是：**任务** <br/>

对于简单的文本生成而言，我们只需要一个输入和一个输出就可以了，但是对于复杂的文本生成任务而言，比如来自消息队列的一条生成式消息 或者 来自用户请求
的一条综合概要式消息，需要你结合业务的上下文，提交多个输入、大量的文本内容进行生成，并且非常耗时耗资源的请求，你完全可以将其视为一个任务来处理。<br/>

所以我们将整个系统的设计围绕着任务来进行设计，我们需要考虑到：
- **任务的创建**:比如任务的状态，任务的输入参数持久化
- **任务的执行过程**:比如状态的管理以及重试条件和机制
- **任务的结果存储和通知方式**：比如将结果预存储，然后通过消息队列通知出去给下游

> 接下来我会重点介绍 DDD 的设计思路和实现方式

## 1.1 语言
- **Python** <br/>
考虑整个团队过往的技术经验（ML-Scribe 项目就是整个团队用 Python 来写的）和项目交接和维护的便利性，这次我也才用了 Python 作为主要的开发语言

## 1.2 框架集成

- **FAST-API** <br/>
同样，FastAPI 也是我们在过往的项目中常用的 WebMVC 架构，不同的是这次我们仅用他的 uvicorn应用服务器和 Restful 风格的endpoint 注解，
至于其依赖注入，我们暂时没有用到

- **Pydantic** <br/>
我们用 Pydantic 来做数据模型的定义和数据校验，主要是为了方便后续的代码维护和可读性

- **Dependency Injector** <br/>
一款轻量级的依赖注入框架，主要是为了方便后续的代码维护和可读性，引入的目的是依赖解耦和自动注入，同时方便后续的单元测试和集成测试，让 Mock 变得
更加简单。

## 1.3 数据存储
- **MongoDB** <br/>
考虑服务底层之间的数据流转和存储，我们也选择了 MongoDB 作为底层的数据库，因为我们可能会涉及复用 ML-Scribe 项目的部分表，而 ML-Scribe 本身
底层就是用 MongoDB 来做数据存储的。

- **Redis** <br/>
Redis 在此处的作用主要是：
  - 缓存和存储一些临时的数据，比如路由规则，没有必要每次请求都去查询数据库
  - 锁机制，比如为了保持幂等性，我们需要在请求的过程中加锁，避免重复请求

## 1.4 中间件
- **AWS SQS** <br/>
简单消息队列，我们用来做异步任务的处理，当有批量任务或者异步任务进来时候，我们会考虑异步执行，然后将结果通过消息队列的形式通知给调用方，通过快速
存储+异步处理的方式来提高系统的性能和响应速度
- **RabbitMQ** <br/>
当然，它只是一个 SQS 的备选方案，假设我们有一天需要迁移到其他的云服务商上，RabbitMQ 可能会是一个不错的选择

## 1.5 其他
- **Docker** <br/>
em...,快速部署，将所需的中间件依赖打包，方便 DevOps 能够快速部署到云端。

- **AWS System Manager** <br/>
只是一个地方来存储一些敏感的配置，比如数据库的密码，API 的密钥等，在系统启动的时候，读取这些配置更新到环境变量中。当然，我提供了公共的接口实现，
你可以自定义实现，当迁移到其他平台上的时候，你可能需要写个 AzureVariableLoader 从某个地方将变量加载进来。这一切都很 Easy。

