# Workflow


## Activiti

### Activiti 5

#### Database table names explained

The database names of Activiti all start with `ACT_`. The second part is a two-character identification of the use case of the table. This use case will also roughly match the service API.

* `ACT_RE_*`: RE stands for `repository`. Tables with this prefix contain static information such as process definitions and process resources (images, rules, etc.).

* `ACT_RU_*`: RU stands for `runtime`. These are the runtime tables that contain the runtime data of process instances, user tasks, variables, jobs, etc. Activiti only stores the runtime data during process instance execution, and removes the records when a process instance ends. This keeps the runtime tables small and fast.

* `ACT_ID_*`: ID stands for `identity`. These tables contain identity information, such as users, groups, etc.

* `ACT_HI_*`: HI stands for `history`. These are the tables that contain historic data, such as past process instances, variables, tasks, etc.

* `ACT_GE_*`: `general` data, which is used in various use cases.

#### Activiti 5 文档

* [官方文档](https://www.activiti.org/5.x/userguide/)
  * [3.17. Mapped Diagnostic Contexts](https://www.activiti.org/5.x/userguide/#MDC)
  * [4.6. Expressions](https://www.activiti.org/5.x/userguide/#apiExpressions)
  * [5.5. Unit testing](https://www.activiti.org/5.x/userguide/#springUnitTest)
  * [5.7. Spring Boot](https://www.activiti.org/5.x/userguide/#springSpringBoot)
  * [6.3. Versioning of process definitions](https://www.activiti.org/5.x/userguide/#versioningOfProcessDefinitions)
  * [6.6. Category](https://www.activiti.org/5.x/userguide/#deploymentCategory)
  * [8.6.3. Transaction subprocess](https://www.activiti.org/5.x/userguide/#bpmnTransactionSubprocess)
  * [8.7. Transactions and Concurrency](https://www.activiti.org/5.x/userguide/#bpmnConcurrencyAndTransactions)
    * [8.7.1. Asynchronous Continuations](https://www.activiti.org/5.x/userguide/#asyncContinuations) 
    * [8.7.2. Fail Retry](https://www.activiti.org/5.x/userguide/#failRetry)
    * [8.7.3. Exclusive Jobs](https://www.activiti.org/5.x/userguide/#exclusiveJobs)
  * [18.1. Async and job executor](https://www.activiti.org/5.x/userguide/#advanced_parseHandlers)
  * [18.2. Hooking into process parsing](https://www.activiti.org/5.x/userguide/#_hooking_into_process_parsing)
  * [18.6. Advanced Process Engine configuration with a ProcessEngineConfigurator](https://www.activiti.org/5.x/userguide/#advanced.process.engine.configurators)
  * [18.8. Custom identity management by overriding standard SessionFactory](https://www.activiti.org/5.x/userguide/#advanced.custom.session.manager)
  * [18.12. Secure Scripting](https://www.activiti.org/5.x/userguide/#advancedSecureScripting)

* [Activiti 快速入门教程：SpringBoot 集成 Activiti6 + Activiti Modeler 流程配置可视化](https://blog.csdn.net/qq_37143673/article/details/102667824)

* [Activiti5 表结构说明](https://lucaslz.gitbooks.io/activiti-5-22/content/)


#### Activiti 5 知识点

* 上传文件执行流程部署 - `DeploymentCollectionResource#uploadDeployment()` - `此类在activiti-rest.jar包中`
* `BpmnModel`和`XML`互转 - `BpmnXMLConverter`
* `BpmnModel`和`JSON`互转 - `BpmnJsonConverter`：JSON为`ACT_RE_MODEL`表`EDITOR_SOURCE_VALUE_ID_`字段对应的`ACT_GE_BYTEARRAY`数据
* `org.activiti.engine.impl.identity.Authentication` - `设置/获取` 当前操作人 
  > 也可使用`IdentityService#setAuthenticatedUserId()`设置，内部调用同一代码


## BPMN

> [BPMN 2.0 详细文档](https://www.omg.org/spec/BPMN/2.0/PDF)

### bpmn-js

BPMN 2.0 for the web, View and edit BPMN 2.0 diagrams in the browser.

* [bpmn-js 官网](https://bpmn.io/)


## 参考链接

* [awesome-workflow-engines](https://github.com/meirwah/awesome-workflow-engines)
* [开源流程引擎哪个好，如何选型？](https://zhuanlan.zhihu.com/p/369761832)



