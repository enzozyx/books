# processEngine

[TOC]



## processEngine 创建：

### structure



![api.services](./images/api.services.png) 

### init and destory

- getDefaultProcessEngine()  依据配置文件创建processEngine实例



### configuration

- flowable.cfg.xml 
- flowable-context.xml 

### services

- RepositoryService(static information)

  > 提供对流程发布与定义的管理、操作、维护(This service offers operations for managing and manipulating `deployments` and `process definitions` )

  - 查询流程

  - 禁用与激活流程

  - 解析流程变量

  - 解析获取流程定义的Java POJO描述模型

- RuntimeService

  > 处理流程实例

  - 可用来解析流程实例变量

- TaskService

  - 查询指定用户或group的task
  - 创建独立的task，这个task与流程定义无关
  - 分发(Claim)与完成(Complete)任务

- IdentityService

  > 提供对用户及group的增删改查

  - processEngine 不会verify 用户信息

- Form Service

  > 可选

  - 提供开始流程、以及任务信息提交的表单服务

- HistoryService

  > 查询流程实例的历史信息

  - 任务完成时间
  - 流程实例流转路径
  - 等

- ManagementService

  > 

  - 提供检索数据以及表的元信息
  - 可以查询jobs
    - 1. 定时器
      2. asynchronous continuations 
      3. delayed suspension/activation 

- **DynamicBpmnService**  

  >在不重新发布流程情况下，改变流程的部分定义，如改变 用户任务的 委托人， 或者事改变service 任务的class name

  - 

  

