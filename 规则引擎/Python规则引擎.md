
## Durable Rules

durable_rules 是一个多语言微框架，用于实时、一致且可扩展的事件协调。通过 durable_rules，您可以追踪和分析事件信息，通过整合来自多个来源的数据来推断更复杂的情况。

它采用完整的前向链实现（Rete）来实时评估事实和事件。简单的元语言抽象让您能够定义简单和复杂的规则集，以及控制流结构，如流程图、状态图、嵌套状态图和时间驱动流。

durable_rules 核心引擎采用 C 语言实现，既能实现快速规则评估，也支持多语言环境。

durable_rules 可通过将状态卸载到进程外数据存储（如 Redis）来实现横向扩展。状态卸载机制具备可扩展性，因此您可以根据需要集成任意数据存储方案。

> [项目地址](https://github.com/jruizgit/rules)


## Business Rules

- Venmo 开源
- 偏向“配置化规则”（JSON驱动）
- 非技术人员也能写规则
- 可与前端规则配置UI结合
- 简单易用

> [项目地址](https://github.com/venmo/business-rules)

## Rule Engine

一种轻量级、可选类型的表达式语言，采用自定义语法，用于匹配任意 Python 对象。

> [项目地址](https://github.com/zeroSteiner/rule-engine)
