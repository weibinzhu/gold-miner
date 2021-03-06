> * 原文地址：[The Art of System Performance for Engineers](https://levelup.gitconnected.com/the-art-of-system-performance-for-engineers-1c85a398d6f2)
> * 原文作者：[The AI LAB](https://medium.com/@ailab)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/TODO1/the-art-of-system-performance-for-engineers.md](https://github.com/xitu/gold-miner/blob/master/TODO1/the-art-of-system-performance-for-engineers.md)
> * 译者：[lhd951220](https://github.com/lhd951220)
> * 校对者：[z0gSh1u](https://github.com/z0gSh1u)  [plusmultiply0](https://github.com/plusmultiply0)  [kylinholmes](https://github.com/kylinholmes)

# 写给工程师的《系统性能兵法》

遇到系统性能问题是软件工程师的职业生涯中不可避免的一部分。一些常见的系统性能问题示例如下：

* 磁盘 I/O：比如，从磁盘中加载应用程序代码、加载资源
* 网络 I/O：从服务器中加载数据或者图片
* 进程间的调用：实现单点登录，或者与操作系统交互

在 Java 中，这些问题中的大部分都不会在写代码的时候立即暴露出来。 大多数磁盘IO方法的抽象化加之类的隐式加载，让它看起来像是在调用普通方法。实际上，大多数情况下， 所有这些问题（除了网络 IO）在非正式的本地测试中都运行得足够快， 从而不会注意到这些问题的发生。代码加载快的原因是在运行之前代码就已经在内存中，并且你调用的进程在之前的调用中仍然存活。而对于网络 IO 来说，如果因为网络太快而没有发现的问题也在会在生产环境中显现出来。

如果工程师可以“看见”他们的调用变慢，这样不是很好吗？

## 线程

一般情况下，应用需要保持 60fps 的帧率，或者每 16ms 一帧。为了实现这个目的，UI 线程必须在 0 ~ 16ms 的时间范围内更新 UI。网络 IO 通常会花费数倍于 16ms 的时间，而这就给了我们两个选择：在更小的时间片中拆分任务，或者让其他进程执行工作并返回结果。在 Java 中，第二种选择是使用线程来实现的。

有一个后台线程来完成耗时的任务，任务的结果需要在 UI 中展示。因此需要将数据从后台线程传送到 UI 线程。实践中，可能有几种模式：

* UI 线程订阅了数据的事件。
* UI 线程从共享对象中读取/轮询数据。

第一种模式更强大，但需要做更多的工作。第二种模式更加自然，但风险更大。举个例子，在整个应用程序中，开发者也许需要知道当前用户的信息。但是，当启动应用程序时，这些信息仍然需要从磁盘中加载。这导致了一个有趣的情况，开发者为了等待从后台线程生成的结果，而阻塞 UI 线程。不幸的是，这种使 UI 线程等待后台线程的模式，比预期的更为普遍。

如果工程师可以“看见”他们调用阻塞方法，这样不是很好吗？

## 权衡取舍

在 UI 线程上完成所有工作是最简单解决方法，而且很多时候都有效。但是，在真实世界中，一些任务存在严重的性能问题。在大多数时候，开发者会将大部分工作转移到后台线程来完成。

直接从后台线程准备好的结果中读取数据通常是更优的，除了在数据读取操作的过程中存在着并发写的操作的情况。所以，要通过同步操作来保证独占数据访问。

使用独占访问方式直接从结果中读取数据通常是很快的，除了有时会出现争抢对数据的独占访问的情况。因此，大多数的时候，开发者会将一些任务工作在请求-响应模式下，让 UI 线程订阅结果。

订阅数据的变化还要求预期计划，这会给用户在做正确的事情时添加更多的复杂度。

如果工程师可以提供一个更简单的模式来实现正确，高效的代码，这样不是更好吗？

尽管这些挑战听起来好像很难应对，这里有工程师克服这些挑战的一些建议：

1. 向工程师提供相关的运行时信息，以此帮助他们在写代码时做出更好的策略，例如：创建一个展示数据的插件。
2. 原型是一个性能优秀且可读性好的 reactive/redux 样式的 UI。

毫无疑问，不论工程师们的努力程度如何，这些系统挑战要求工程师除了具有“硬”背景之外，还需要一些”软”技能。

作为一名个人贡献者，将他们的问题和疑虑带到系统审查会议上是工程师自己的责任。可能需要创建一个与管理员共享的文档，提出一个你想要讨论的主题，并跟进提交。

不要在公司范围的正式调查表上或在全体会议上提出一个复杂的话题。请确保你正在与经理讨论困难的事情。这些都是相互之间的信任。

## 输出产品，并让每个人都知道它

在不同的公司中，以不同的方式来衡量每一个工程师的成功。但是，有一个事情是与每一个想要被认可和被奖励的人相关的。就是**输出产品**，或者为基础架构团队进行相关的系统开发工作。这应该出现在自我审视和讨论中：交付了什么东西。

如果你向你的经理解释你面对的挑战，以及如何克服它们的，这将会对他大有帮助。也许你的经理需要一些理由来帮助你提名晋升，帮助他们找到一些理由。

## 进入下一等级

不要低估职级评定环节的价值，并且不要忘记寻找你的组织中的最佳表现者。其他团队中可能已经存在高级工程师，他们可能正是因为该标准而被提名晋升。

如果你想要得到晋升 —— 开始以下一等级的标准来工作，承担责任并且扩大项目范围。不能等到有人来告诉你做什么事情。现在就开始前进，建立你的责任感，对你的认可将会随之而来。

## 在下一等级意味着什么？

![](https://cdn-images-1.medium.com/max/2000/0*tTqppxhq9ubffWf-)

大多数的科技公司都会对能力做等级划分（他们也许会称专业人士为“初级”，“中级”，“高级”，“专家”，或者有一些相应的编码，比如 IC3-IC6）。一些公司定义了职业期望：在每一个能力等级上你应该做什么。很少见的职业期望是“交付时间”维度。

比如，如果一个高级工程师可以在两周的时间内交付特性 X，而这对于中级工程师来说可能需要三周的时间。如果组织想要提名中级工程师晋升 —— 他们不仅应该交付高质量的工作，还需要在下一等级的时间范围内完成工作。

## 以可持续的方式发展

大多数的公司都有两种类型的奖励来表彰员工的出色业绩。第一个是奖金，第二个则是晋升。奖金是对人们过去的一段时间的表现的额外回报。晋升是对他们拥有下一级别能力和长久以来的表现的认可。这两种奖励通常不会同时到来。一个好的管理者应该及时发现是否有人 24×7 小时制的工作，并且应该考虑将其标记为不能在下一等级中工作。

没有人可以在 24×7 全天候的工作，所以你的表现将会在未来下降。在花费了合理的时间在工作上的时候，确保收获是超出预期的。

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。

---

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。
