% !TEX TS-program = xelatex
% !TEX encoding = UTF-8 Unicode
% !Mode:: "TeX:UTF-8"

\documentclass{resume}
\usepackage{zh_CN-Adobefonts_external} % Simplified Chinese Support using external fonts (./fonts/zh_CN-Adobe/)
%\usepackage{zh_CN-Adobefonts_internal} % Simplified Chinese Support using system fonts
\usepackage{linespacing_fix} % disable extra space before next section
\usepackage{cite}

\begin{document}
\pagenumbering{gobble} % suppress displaying page number

\name{莫肇邦}

\basicInfo{
 \email{kevinleee9527@gmail.com} \textperiodcentered\ 
 \phone{(+86) 198-8099-4035} \textperiodcentered\ 
 }
 
\section{\faCogs\ IT 技能}
% increase linespacing [parsep=0.5ex]
\begin{itemize}[parsep=0.5ex]
 \item 熟练掌握 Java 基础知识，如集合、面向对象、反射、异常、 IO 流等，具有良好的编程习惯及代码规范；
 \item 熟悉 JVM 内存结构、JMM、垃圾回收算法、双亲委派机制、常见垃圾回收器等;
 \item 熟悉 JUC 并发编程，理解各种锁机制、CAS、AQS、线程池、ThreadLocal等实现原理;
 \item 熟练使用 SSM、SpringBoot等框架，深刻理解IoC、AOP、Bean 生命周期、循环依赖等;
 \item 熟练掌握 MySQL，深刻理解事务、索引、锁机制、MVCC、各种日志等，能够进行简单的SQL优化;
 \item 熟练掌握 Redis，深刻理解其数据结构、持久化策略、IO多路模型、哨兵机制、高性能原理等;
 \item 熟练掌握 RabbitMQ 消息中间件，实现高效的异步数据处理和消息传递;
 \item 熟悉常用设计模式，如单例、工厂、模板方法、责任链、策略模式等;
 \item 熟悉 Linux操作系统及常用命令、Docker常用命令;
 \item 了解 DDD 方法论的基本思想，以及常用的基本概念和分层架构;
 \item 熟悉 Claude Code 的各项命令操作，包括代码生成、代码解释、代码审查、重构建议、调试辅助、单元测试生成等核心功能，能够高效利用 AI 编程助手提升开发效率；
 \item 了解 CI/CD 流程编排与自动化任务执行的基本思路
\end{itemize}


\section{\faUsers\ 项目经历}
\datedsubsection{\textbf{拼得多系统}}{2026年2月 -- 至今}
\begin{onehalfspacing}
项目描述:
\begin{itemize}
 \item 通过调研发现，拼团模式是电商系统中提升用户粘性和订单转化率的重要手段。该项目基于此背景实现了一个拼团营销平台系统，采用领域驱动设计划分为活动、交易、标签三大领域，核心业务覆盖资格核验、优惠锁单、交易结算、逆向退单全流程，支持人群标签过滤与系统配置热更新，并通过多种设计模式提升可扩展性。
\end{itemize}

核心技术: SpringBoot + MyBatis + MySQL+ Redis + RocketMQ + Docker + Retrofit + SpringTask

技术亮点: 
\begin{itemize}
 \item 设计并提炼通用的责任链模板，实现锁单流程和结算流程的前置规则过滤以及拼团退单的流程串联。
 \item 设计运用规则树模板，通过配置多层树节点，实现参数校验、折扣计算与人群过滤的优惠试算流程的动态编中排，提高可维护性与可扩展性。
 \item 采用 Redis BitMap 存储人群标签数据，实现轻量化存储，便于对用户做精准定向互动投放和标签过滤。
 \item 为提高用户体验，将优惠试算所需商品、活动等数据由串行改为 FutureTask 异步数据加载，提高查询效率。
 \item 交易域锁单借鉴分段锁思想，通过无锁化设计抢占库存，降低数据库压力。
 \item 使用 HTTP + MQ 双触达机制，结合补偿任务与分布式锁确保结算结果最终一致性，提升系统适配性与鲁棒性。
 \item 采用策略模式实现不同退单类型的退单逻辑，恢复缓存库存量，并通过定时任务实现超时未支付退单。
 \item 基于 Redis 发布/订阅机制实现动态配置中心，覆盖降级、切量、白名单测试等场景，提升系统的灵活性与可维护性。
\end{itemize}
\end{onehalfspacing}

\begin{minipage}{\textwidth}
\datedsubsection{\textbf{AI Agent 智能助手平台}}{2026年1月 -- 2026年3月}
\begin{onehalfspacing}

项目描述:
\begin{itemize}
 \item 参与开发一套基于大语言模型的智能对话代理系统（AI Agent），集成多种 AI 能力组件（AI Advisor、Prompt 工程、MCP 协议），实现自动化任务调度与多轮对话管理。系统支持高并发场景下的稳定运行，具备完整的日志监控与检索能力，并引入自研 AI Auto Agent 模块，显著提升任务自动化程度。
\end{itemize}

核心技术: Spring AI + SpringBoot + MySQL + PGVector + MyBatis + Docker + ELK + SSE

技术亮点:
\begin{itemize}
 \item 负责整体 AI Agent 架构设计与核心模块开发，基于 Spring AI 框架搭建 Agent 运行环境，统一封装 Model、MCP 与 Prompt 调用链路，实现模块解耦与灵活扩展。
 \item 自主实现 MCP Client 多协议接入层，支持 stdio / SSE 两种传输模式，集成 ELK、Baidu Search、filesystem 等多类工具能力，提升 Agent 的信息获取与处理能力。
 \item 基于 PostgreSQL + PGVector 构建 RAG 检索增强生成管道，实现文档向量化存储、语义检索与上下文注入，通过自定义 RagAnswerAdvisor 动态注入 Agent Prompt，显著提升回答准确性。
 \item 设计并实现流式输出（SSE）机制，优化 Agent 响应体验，支持实时结果推送；对 Agent 执行链路进行性能调优，降低端到端延迟。
\end{itemize}
\end{onehalfspacing}
\end{minipage}


\section{\faGraduationCap\ 教育背景}
\datedsubsection{\textbf{东莞城市学院}, 东莞}{2023 -- 至今}
\textit{本科}\ 人工智能, 预计 2027 年 6 月毕业

\section{\faInfo\ 自我评价}
具备坚实的编程基础、团队合作能力、学习和问题解决能力，以及自我管理和自我驱动能力。保持良好的自学能力，对新技术敏感，能够快速提取可落地的技术方案运用到实际项目开发中。

\end{document}
