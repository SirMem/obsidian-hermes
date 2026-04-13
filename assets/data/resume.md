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
\end{itemize}


\section{\faUsers\ 项目经历}
\datedsubsection{\textbf{拼得多系统}}{2025年9月 -- 2025年10月}
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
  \item 交易域锁单借鉴 ConcurrentHashMap JDK1.7分段锁思想，通过无锁化设计抢占库存，降低数据库压力。
  \item 使用 HTTP + MQ 双触达机制，结合补偿任务与分布式锁确保结算结果最终一致性，提升系统适配性与鲁棒性:。
  \item 采用策略模式实现不同退单类型的退单逻辑，恢复缓存库存量，并通过定时任务实现超时未支付退单。
  \item 基于 Redis 发布/订阅机制实现动态配置中心，覆盖降级、切量、白名单测试等场景，提升系统的灵活性与可维护性。
\end{itemize}
\end{onehalfspacing}

\begin{minipage}{\textwidth}
\datedsubsection{\textbf{基于 Open AI的自动化代码评审服务}}{2025年9月 -- 至今}
\begin{onehalfspacing}

项目描述:
\begin{itemize}
  \item 该项目是一个基于AI技术的自动化代码评审工具，通过GitHub Actions自动化触发 → Git解析增量代码 → ChatGLM 智能评审 → 微信模板消息推送的完整链路，解决人工代码评审效率低、风险遗漏问题。
\end{itemize}

核心技术: SpringBoot + GitHub Actions + JGit + Chat GLM 对接 + 微信公众号对接

技术亮点:
\begin{itemize}
  \item 设计 GitHub Actions 多阶段工作流，支持指定分支提交自动触发，通过自动化流程优化持续集成效率，减少人工干预。
  \item 使用 Java 实现轻量级 SDK，集成到 CI/CD 流程中，支持 GitHub Actions、GitLab CI等平台，便于在不同项目中快速集成和使用。
  \item 集成日志仓库写入与微信公众号推送功能，实现评审结果的实时反馈与跟踪管理
  \item 项目使用DDD架构拆分重构，抽象接口，支持多种 AI 模型接入，提高项目可维护性与扩展性。
  \item 应用于“拼得多系统”代码审查，累计分析提交 40+ 次，发现潜在缺陷 17处，提供评审建议 38 条，显著提升代码质量与交付效率。
\end{itemize}
\end{onehalfspacing}
\end{minipage}


% Reference Test
%\datedsubsection{\textbf{Paper Title\cite{zaharia2012resilient}}}{May. 2015}
%An xxx optimized for xxx\cite{verma2015large}
%\begin{itemize}
%  \item main contribution
%\end{itemize}





\section{\faGraduationCap\  教育背景}
\datedsubsection{\textbf{东莞城市学院}, 东莞}{2023 -- 至今}
\textit{本科}\ 人工智能, 预计 2027 年 6 月毕业

\section{\faInfo\ 自我评价}
具备坚实的编程基础、团队合作能力、学习和问题解决能力，以及自我管理和自我驱动能力。我相信在实际工作中，我能够不断学习和成长，为团队和组织做出积极的贡献。
%% Reference
%\newpage
%\bibliographystyle{IEEETran}
%\bibliography{mycite}
\end{document}
