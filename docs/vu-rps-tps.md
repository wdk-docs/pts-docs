# 并发用户、RPS、TPS 的解读

本文介绍并发用户、RPS、TPS 的基本概念以及三者之间的关系。

## 术语定义

- 并发用户：在性能测试工具中，一般称为虚拟用户（Virtual User，简称 VU），指的是现实系统中操作业务的用户。
  说明 并发用户与注册用户、在线用户不同。
  注册用户一般指的是数据库中存在的用户。
  在线用户只是“挂”在系统上，对服务器不产生压力。
  但并发用户一定会对服务器产生压力。
- TPS：Transaction Per Second，每秒事务数，是衡量系统性能的一个非常重要的指标。
  说明 系统每秒处理事务数越多证明您的机器性能越好。
- RPS：Request Per Second，每秒请求数。RPS 模式适合用于容量规划和作为限流管控的参考依据。
- RT：Response Time，响应时间，指的是业务从客户端发起到客户端接收的时间。

在性能测试中，通常有两种施压模式：`并发模式` 和 `RPS 模式`。
传统方式是使用并发用户数来衡量系统的性能（站在客户端视角）。此方法一般适用于一些网页站点的压测（例如 H5 页面）；
而 RPS（Requests per second）模式主要是为了方便直接衡量系统的吞吐能力 TPS（Transaction Per Second，每秒事务数）而设计的（站在服务端视角），按照被压测端需要达到 TPS 等量设置相应的 RPS，应用场景主要是一些动态的接口 API，例如登录、提交订单等等。

## VU 和 TPS 换算

公式描述：`TPS = VU ÷ RT`，（RT 单位：秒）。

举例说明：
假如 1 个虚拟用户在 1 秒内完成 1 笔事务，那么 TPS 明显就是 1。
如果某笔业务响应时间是 1 ms，那么 1 个虚拟用户在 1s 内能完成 1000 笔事务，TPS 就是 1000 了；
如果某笔业务响应时间是 1s，那么 1 个虚拟用户在 1s 内只能完成 1 笔事务，要想达到 1000 TPS，就需要 1000 个虚拟用户。
因此可以说 1 个虚拟用户可以产生 1000 TPS，1000 个虚拟用户也可以产生 1000 TPS，无非是看响应时间快慢。

## 如何获取 VU 和 TPS

VU 获取方式：

- 已有系统：可选取高峰时刻，在一定时间内使用系统的人数，这些人数可认为是在线用户数，并发用户数可以取 10%，例如在半个小时内，使用系统的用户数为 10 万，那么取 10%（即 1 万）作为并发用户数基本就够了。
- 新系统：没有历史数据作参考，建议通过业务部门进行评估。

## TPS 获取方式：

- 已有系统：可选取高峰时刻，在一定时间内（如 3 分钟~10 分钟），获取系统总业务量，计算单位时间（秒）内完成的笔数，乘以 2~5 倍作为峰值的 TPS，例如峰值 3 分钟内处理订单 18 万笔，平均 TPS 是 1000，峰值 TPS 可以是 2000~5000。
- 新系统：没有历史数据作参考，建议通过业务部门进行评估。

## 如何评价系统的性能

针对服务器端的性能，以 TPS 为主来衡量系统的性能，并发用户数为辅来衡量系统的性能，如果必须要用并发用户数来衡量的话，需要一个前提，那就是交易在多长时间内完成，因为在系统负载不高的情况下，将思考时间（思考时间的值等于交易响应时间）加到串联链路中，并发用户数基本可以增加一倍，因此用并发用户数来衡量系统的性能没太大的意义。
同样的，如果系统间的吞吐能力差别很大，那么同样的并发下 TPS 差距也会很大。

## 性能测试策略

做性能测试需要一套标准化流程及测试策略。在做负载测试的时候，传统方式一般都是按照梯度施压的方式去加用户数，避免在没有预估的情况下，一次加几万个用户，导致交易失败率非常高，响应时间非常长，已经超过了使用者忍受范围内；较为适合互联网分布式架构的方式，也是阿里巴巴的最佳实践是用 TPS 模式（吞吐量模式）+设置起始和目标最大量级，然后根据系统表现灵活的手工实时调速，效率更高，服务端吞吐能力的衡量一步到位。更多信息，请参见如何设置目标并发或目标 RPS？。

## 总结

综上所述，可以得出以下结论：

- 系统的性能由 TPS 决定，跟并发用户数没有多大关系。
- 系统的最大 TPS 是一定的（在一个范围内），但并发用户数不一定，可以调整。
- 建议性能测试的时候，不要设置过长的思考时间，以最坏的情况下对服务器施压。
- 一般情况下，大型系统（业务量大、机器多）做压力测试，10000~50000 个用户并发，中小型系统做压力测试，5000 个用户并发比较常见。
