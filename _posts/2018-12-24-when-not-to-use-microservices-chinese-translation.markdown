---
title:  "（翻译）不适合微服务的场景"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: micro-services
---

*一句话总结：软件开发没有银弹，任何技术都有 trade-off，微服务也不例外*

*原文：[Charles Féval - When not to use microservices](https://www.feval.fr/posts/microservices/)*

---

*（跳过微服务的定义，直奔主题～～）*

## 微服务的难题

1. 微服务**很难**正确**设计**。有 关于他们陷阱的[全书](https://www.safaribooksonline.com/library/view/microservices-antipatterns-and/9781492042716/)。需要（很多）迭代才能获得令人满意的域设计和边界。几个基本和结构化问题没有直接的答案，需要调整和迭代，例如：如何管理[横切关注点](https://auth0.com/blog/introduction-to-microservices-part-4-dependencies/)（共享，耦合；不共享，冗余），如何共享数据和什么级别要冗余，如何管理报错，是否在服务中包含UI组件等。

2. 技术会做多了很多“无用功”，要额外考虑以下技术细节：*（例如下单 API，变成先请求用户服务 API 查用户、仓库服务 API 查库存，再请求订单服务 API 减库存）*

   - API 数量增加
   - 服务间延迟
   - 原子操作可能涉及多个服务
   - 涉及多个服务的调试

   而不是带来价值的业务功能

1. 对每个人要求非常高。他们需要了解分布式系统，devops，用代码控制基础服务，不同类型的数据库，组合和组件化前端，对所有内容进行单元测试，在没有人工干预的情况下发布到生产，迭代和失败，规划非常小的版本，在产品中进行测试，管理多个版本，等等

## 何时不该使用微服务

1. 该**应用程序太小**，无法证明微服务的合理性[7](https://www.feval.fr/posts/microservices/#fn:1)。当然，它可能会在未来发展，并将整个产品添加到其中; 在这种情况下，当它们实际接近RoI的阈值时切换到微服务。类似的推理可以应用于小型团队。

   ![img](https://www.feval.fr/img/2018/microservices/costs.png)

2. **业务没有清晰划分或具有不确定性**，例如最初的 CRM 实际上也可以管理订单、物流*（业务部门用得方便）*

3. 团队**缺乏对微服务**概念，DDD（[Domain-Driven Design](https://airbrake.io/blog/software-design/domain-driven-design)）或概念设计的**经验和理解**。*重构弄不好会非常伤*

4. 团队**不成熟**[12](https://www.feval.fr/posts/microservices/#fn:14)，技术**技能不适应**[13](https://www.feval.fr/posts/microservices/#fn:9)，或者**营业额高**[14](https://www.feval.fr/posts/microservices/#fn:3)。由于熵会导致代码清洁度和体系结构[随着时间的推移](http://www.rntz.net/post/against-software-development.html)而逐渐[衰减](http://www.rntz.net/post/against-software-development.html)，并且由于更复杂的体系结构难以维护，这可能会加剧上一点的后果。

   不理解和压力往往让人们[回归他们所知道的](https://en.wikipedia.org/wiki/Status_quo_bias)。他们会偷工减料，或者通过走最短路径来逃避复杂性。很快，架构可能会被污染到技术组件，“核心”库，对其他服务包的引用，协调器，“CSV导入服务”，服务将开始进入彼此的数据库。接下来你知道有人会开始问“我们如何协调部署以管理服务之间的依赖关系？”。这是地狱的软件版本，也称为**分布式整体**：

   ![img](https://www.feval.fr/img/2018/microservices/dependency-hell.png)

   操作和调试的复杂性可能阻碍提高和总体效率。调查错误和分析服务日志的复杂性可能会将这些任务重定向到那些技能更高的团队成员，他们的重点应该是长期而不是消防。简而言之 - 这种情况可能会让团队为没有优势的微服务带来所有后果。

## 除非

如果您正在构建的应用程序具有相当明确的域，将会经历相当大的变化规模，需要从一开始就构建一个庞大的团队，您对团队的技能有信心并且拥有一些分布式设计中经验或者至少入门，并且管理层对失败和学习的支持，微服务是一个很好的选择。

但要注意微服务布局可能会适得其反。如果前文说中了，那么从更简单的东西开始可能更明智，例如整体式或分层式架构（其本身可能包含一些专门的服务）。微服务架构带来的大多数功能也可以通过其他解决方案实现。

## 要点

1. Martin Fowler已经确定了[微服务的先决条件](https://martinfowler.com/bliki/MicroservicePrerequisites.html)清单。ThoughtWorks将[微服务](https://www.thoughtworks.com/insights/blog/microservices-adopt)保持[在他们](https://www.thoughtworks.com/insights/blog/microservices-adopt) 雷达[的“试验”](https://www.thoughtworks.com/insights/blog/microservices-adopt)类别中（这基本上意味着：它不是默认值，在你采用它之前尝试它。）并且提到它可能永远不会移动。我还遇到了一个有趣的读物，关于[停止让微服务成为](https://medium.com/@rbarcia/its-time-to-stop-making-microservices-the-goal-of-modernization-71758b400287) IBM杰出工程师[现代化的目标](https://medium.com/@rbarcia/its-time-to-stop-making-microservices-the-goal-of-modernization-71758b400287)，而[David Kerr也是如此](https://dwmkerr.com/the-death-of-microservice-madness-in-2018/)。 [↩](https://www.feval.fr/posts/microservices/#fnref:11)
2. 一些使用这里：[山姆·纽曼](https://samnewman.io/books/building_microservices/)，[刘易斯和福勒](https://martinfowler.com/articles/microservices.html)，[维基百科](https://en.wikipedia.org/wiki/Microservices)，[马克·理查兹](https://www.safaribooksonline.com/library/view/microservices-antipatterns-and/9781492042716/)，[鲍勃叔叔](https://blog.cleancoder.com/uncle-bob/2014/10/01/CleanMicroserviceArchitecture.html) [↩](https://www.feval.fr/posts/microservices/#fnref:10)
3. 每项服务都需要保持独立，同时保证其消费者的稳定性。每次服务修改其API时，其消费者也需要更新。但这些可能并不为人所知，客户的升级不应该是提供新功能的关键途径。直接后果是每个单独的微服务的API需要单独进行版本化，旧版本需要维护一段时间。这会产生变化的摩擦，具有讽刺意味的是可能会降低你的灵活性。跨服务交互是基于API的事实也会从您喜欢的IDE中删除所有那些好的重构功能，当您想要跨服务重构时  [↩](https://www.feval.fr/posts/microservices/#fnref:12)
4. 由于通信是通过网络而不是在进程中进行的，因此每次呼叫的延迟计算时间为10秒或100毫秒，而不是几分之一纳秒。假设每个服务调用100ms（在负载下这不是不现实的），如果一个呼叫组成了3个服务，那么*仅在网络中*已经有300毫秒。 [↩](https://www.feval.fr/posts/microservices/#fnref:13)
5. 这通常通过编排的复杂机制同步解决，或者与补偿机制或最终一致性异步解决 - 所有这些都很难。 [↩](https://www.feval.fr/posts/microservices/#fnref:15)
6. 一致性问题还意味着服务间锁定，引入死锁，竞争条件和难以检测的重入问题（A呼叫B呼叫C呼叫A呼叫B ......直到某些事情超时，然后查明发生的事情）。 如此不可能的**竞争条件**你可能会认为它们在进行中是不可能的，不仅可能成为可能，而且实际上发生在微服务中。而你现在最终会遇到每周3次陷入僵局的事情，没有任何相关的事情和几周的调试（真实的故事）。 [↩](https://www.feval.fr/posts/microservices/#fnref:16)
7. 维护微服务执行架构的成本高于它产生的好处。 [↩](https://www.feval.fr/posts/microservices/#fnref:1)
8. 请参阅：E. Evans的[域驱动设计](https://www.goodreads.com/book/show/179133.Domain_Driven_Design)，以及V. Vernon和[Microsoft架构中心](https://docs.microsoft.com/en-us/azure/architecture/microservices/)[实施的DDD](https://www.goodreads.com/book/show/15756865-implementing-domain-driven-design)。 [↩](https://www.feval.fr/posts/microservices/#fnref:2)
9. 如果你的“邮件报错系统”开始查询微服务的数据库，那已经太晚了。 [↩](https://www.feval.fr/posts/microservices/#fnref:7)
10. 如果您开始研究如何按特定顺序编排服务部署并防止“端到端”测试在部署所有服务之前运行，那已经太晚了。 [↩](https://www.feval.fr/posts/microservices/#fnref:6)
11. 当我在一个项目中解释几周或几个月我们意识到我们的架构选择错误时，我不止一次看到工程副总裁或类似职位感到恼火和沮丧，我们需要进行重大的重构。“为什么你不能从一开始就把它[弄好](https://www.feval.fr/posts/microservices/#fnref:17)？”  [↩](https://www.feval.fr/posts/microservices/#fnref:17)
12. 根据定义，一半的人和团队低于平均成熟度水平。 [↩](https://www.feval.fr/posts/microservices/#fnref:14)
13. 团队缺乏测试驱动开发，代码审查流程，高测试覆盖率目标（> 90％），devops，基础架构代码，持续集成和持续部署，减少其在制品工作，频繁部署（在此背景下经常出现）的经验每天多次），仪表监控和记录。 [↩](https://www.feval.fr/posts/microservices/#fnref:9)
14. 例如，如果团队有很多外部协作、新手或外包团队。 [↩](https://www.feval.fr/posts/microservices/#fnref:3)
15. 从[本文](https://blog.cleancoder.com/uncle-bob/2014/10/01/CleanMicroserviceArchitecture.html)开始，Bob叔叔在[Clean Architecture中](https://www.goodreads.com/book/show/18043011-clean-architecture)有关于该主题的完整章节。 [↩](https://www.feval.fr/posts/microservices/#fnref:4)
