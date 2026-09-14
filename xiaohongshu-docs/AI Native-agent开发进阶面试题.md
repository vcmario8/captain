# AI Native--agent 开发进阶面试题

最近有粉丝咨询，建议我可以出一些面试相关的题，但我实在不想写一些流水账一样的面试题，比如 rag 是什么，什么是幂等之类的单调问题，这些问题即便答出来面试官视角也没啥好加分的，而且这些单点问题未来会彻底被 AI 替换。
但面试中又绕不开 agent 相关的问题，因此比较系统的写一篇 agent 进阶面试题，当然和计算机基础息息相关，也是古法程序员的进阶指南。

Agent 本质上是围绕 LLM 组装的一套任务运行系统。它对内负责任务规划、上下文管理、知识检索、工具调用、状态恢复、结果验证和权限控制，对外则表现为一个能够理解目标并持续完成工作的伙伴。
可以把 Agent 类比成一台计算机：任务是进程，LLM 是 CPU，Context 是内存，知识库和过程产物是磁盘，MCP/Tool 是 I/O 与外设接口，Workflow 是进程调度与控制，Checkpoint 是进程快照，Trace 是系统日志，Agent Runtime 则相当于操作系统。
这个类比需要加一个限定：LLM 不是传统的确定性 CPU，而是概率型 CPU。相同输入不一定产生完全相同的结果，它还可能遗漏信息、产生错误判断或生成不存在的事实。因此，Agent 工程不是简单调用 LLM，而是使用传统软件工程方法，管理一个能力很强但输出不完全确定的计算单元。

Agent 开发涉及哪些计算机问题？
如何把复杂目标拆成可执行任务？
这是进程建模和任务调度问题。系统需要把复杂目标拆成若干节点，明确节点的输入、动作、产物、依赖关系、准入条件和准出条件，并决定哪些节点串行、哪些节点并行，以及当前应该执行哪个节点。对应的传统计算机问题包括进程模型、线程划分、优先级调度、依赖调度、并发执行和任务抢占；映射到 Agent 系统，就是 Planning、Workflow、Task Graph 和 Scheduler。

如何控制 Agent 的上下文规模？
这是内存和工作集管理问题。Context 容量有限，加载过多历史信息会挤占有效空间，并让模型难以判断哪些内容仍然有效。对应的传统计算机问题包括分页、虚拟内存、缓存、工作集和淘汰策略；映射到 Agent 系统，就是 Context Selection、Context Budget、摘要压缩、按需加载、信息去重和过期内容淘汰。

如何建设 Agent 的长期知识？
这是存储、索引和数据库问题。知识库需要解决资料如何组织、如何建立索引、如何更新、如何处理版本、如何控制权限，以及旧知识何时失效。对应的传统计算机问题包括文件系统、数据库、索引、缓存一致性、版本控制和访问权限；映射到 Agent 系统，就是 RAG、Knowledge Base、Memory、知识树、向量索引、实体关系索引和失效策略。

如何管理任务过程中的产物？
这是文件系统、构建系统和依赖管理问题。复杂任务会产生分析结果、决策记录、执行方案和验证报告，这些 Artifact 需要保存来源、版本和依赖关系。当上游输入发生变化时，系统要判断哪些下游产物需要失效、哪些可以继续复用。对应的传统计算机问题是依赖追踪、缓存失效和增量构建；映射到 Agent 系统，就是 Artifact Version、Dependency Graph、Content Hash 和 Invalidation。

如何让 Agent 支持断点执行？
这是进程快照和故障恢复问题。LLM 不会自动保存任务进度，因此需要把当前节点、输入版本、已有产物、验证证据、人工决策和下一步动作持久化到外部存储。对应的传统计算机问题包括进程上下文保存、Checkpoint、故障恢复、事务日志和状态机持久化；映射到 Agent 系统，就是 Task State、Node State、Checkpoint、Resume 和 Replay。

如何让 Agent 安全调用外部系统？
这是 I/O、设备驱动、网络和权限控制问题。LLM 只能生成调用意图，真正的外部操作需要通过 MCP 或 Tool 完成。对应的传统计算机问题包括接口协议、驱动适配、参数校验、超时重试、权限控制和故障隔离；映射到 Agent 系统，就是 Tool Schema、MCP Server、Tool Router、Permission Check 和 Human Confirmation。

如何判断 Agent 是否完成任务？
这是断言、退出码和状态迁移问题。Agent 自己声称“完成”不能作为准出依据，系统需要检查产物是否存在、数据是否完整、验证是否通过，以及必要的人工审批是否完成。对应的传统计算机问题包括断言、返回码、状态机和契约检查；映射到 Agent 系统，就是 Condition、Validator、Evidence 和 Acceptance Gate。

Agent 执行失败后如何处理？
这是异常处理、重试、回滚和补偿问题。系统需要区分临时故障、输入缺失、前置方案错误、工具异常和外部环境不可用，再决定重试当前节点、回流前置节点、执行补偿还是终止任务。对应的传统计算机问题包括错误分类、Retry Policy、Rollback、Compensation、Circuit Breaker 和 Stop Condition；映射到 Agent 系统，就是 Failure Taxonomy、Loop、Escalation 和 Human Takeover。

如何保证重复执行不会产生错误副作用？
这是分布式系统中的幂等和事务问题。任务恢复、网络超时或状态写入失败都可能导致同一节点重复执行。如果节点涉及修改数据、发送信息或改变流程状态，就需要保证重复执行不会产生重复副作用。对应的传统计算机问题包括幂等键、事务、Compare-And-Set、去重、状态查询和补偿机制；映射到 Agent 系统，就是 Execution ID、Idempotency Key、Side-effect State 和 Compensation Action。

人应该在什么位置介入？
这是权限提升、特权操作和人工中断问题。高风险、不可逆或责任敏感的决策不能完全由概率模型完成，需要在关键节点暂停自动执行，并由人执行 approve、reject、override 或 stop。对应的传统计算机问题包括权限控制、管理员操作、审批、签名和审计；映射到 Agent 系统，就是 Human Gate、Approval、Version Binding 和 Decision Trace。

如何排查 Agent 为什么失败？
这是日志、调用链和可观测性问题。系统需要记录任务经过哪些节点、装载了什么 Context、调用了哪些工具、使用了哪些产物版本、Condition 为什么通过或失败，以及人做了什么决策。对应的传统计算机问题包括日志、Trace、Metrics、事件溯源和系统回放；映射到 Agent 系统，就是 Task ID、Node ID、Trace ID、Token Usage、Tool Call Trace 和 State Transition Log。

什么时候应该使用多 Agent？
这是多进程和分布式系统问题。多个 Agent 需要有明确的职责边界、独立工作集和可验证产物，并处理进程间通信、共享状态、并发冲突、故障隔离和结果合并。对应的传统计算机问题包括进程划分、消息传递、共享内存、锁、并发控制和分布式一致性；映射到 Agent 系统，就是 Agent Role、Message Bus、Shared Artifact、Conflict Detection 和 Result Merge。

如何评测 Agent 系统？
这是系统 Benchmark、性能测试和稳定性测试问题。评测不能只看最终任务是否完成，还要分析模型推理、知识检索、上下文装载、工具调用、节点准出和 Workflow 执行等层次。对应的传统计算机问题包括吞吐量、延迟、可用性、错误率、资源消耗、压力测试和回归测试；映射到 Agent 系统，就是任务完成率、一次通过率、平均回流次数、人工介入时间、上下文 token 成本、工具调用成功率和恢复耗时。

如何控制 Agent 的安全边界？
这是系统安全和进程隔离问题。Agent 可能读取不可信内容、调用高权限工具或处理敏感数据，因此需要防止外部文本覆盖系统指令，并限制不同任务和用户的数据访问范围。对应的传统计算机问题包括进程隔离、沙箱、最小权限、输入校验、数据脱敏和审计；映射到 Agent 系统，就是 Prompt Injection Defense、Tool Permission、Tenant Isolation、Sandbox 和 Data Policy。

如何管理 Agent 的成本和资源？
这是资源调度和容量管理问题。不同模型、工具和任务节点的成本不同，系统需要根据任务难度、风险和延迟要求选择模型，并限制单个任务能够消耗的 token、时间和工具调用次数。对应的传统计算机问题包括 CPU 调度、资源配额、限流、超时和容量规划；映射到 Agent 系统，就是 Model Routing、Token Budget、Tool Budget、Timeout、Rate Limit 和 Cost Guardrail。

传统软件工程师如何快速转型？
传统软件工程师最大的优势，不是比其他人更会写 Prompt，而是已经掌握了 Agent 系统需要的大部分工程基础。转型的重点不是推翻原有知识，而是重新建立概念映射。
操作系统经验可以迁移到 Workflow、Task State、Checkpoint 和 Scheduler；分布式系统经验可以迁移到幂等、重试、补偿、超时和一致性；数据库经验可以迁移到知识库、Memory、版本和索引；网络与接口经验可以迁移到 MCP、Tool Schema 和外部系统集成；安全经验可以迁移到权限、沙箱、Prompt Injection 和 Human Gate；可观测性经验可以迁移到 Trace、Metrics 和任务回放；测试经验则可以迁移到 Agent Eval、Condition 和 Evidence。
真正需要补充的新知识主要有四部分：理解 LLM 的概率特性和能力边界，理解 Context 如何影响模型推理，理解知识检索与 Tool Calling 如何提供外部能力，以及理解开放式任务如何建立可重复的评测体系。
正确的转型路径不是从“学习 Prompt 技巧”开始，而是从设计一个最小 Agent Runtime 开始：选择一个真实任务，将它拆成几个节点，为每个节点定义输入、产物和准出条件；把任务状态与产物保存到外存；接入一个外部工具；增加一次失败回流和一个 Human Gate；最后通过 Trace 分析任务为什么成功或失败。
