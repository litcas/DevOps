# 性能测试

## 并发虚拟用户、RPS、TPS的解读
1\. 背景
======

在做性能测试的时候，传统方式都是用并发虚拟用户数来衡量系统的性能（站在客户端视角），一般适用于一些网页站点比如首页、H5的压测；而RPS（Requests per second）模式主要是为了方便直接衡量系统的吞吐能力-TPS（Transaction Per Second, 每秒事务数）而设计的（站在服务端视角），按照被压测端需要达到TPS等量设置相应的RPS，应用场景主要是一些动态的接口API，比如登陆、提交订单等等。

VU（虚拟用户）和TPS之间也有其逻辑关系。具体请看下方的说明。

2\. 术语定义
========

*   并发用户数：简称VU ,指的是现实系统中操作业务的用户，在性能测试工具中，一般称为虚拟用户数(Virutal User)，注意并发用户数跟注册用户数、在线用户数有很大差别的，并发用户数一定会对服务器产生压力的，而在线用户数只是 ”挂” 在系统上，对服务器不产生压力，注册用户数一般指的是数据库中存在的用户数。
*   处理能力:　简称TPS, 每秒事务数, 是衡量系统性能的一个非常重要的指标。
*   响应时间：简称RT,指的是业务从客户端发起到客户端接受的时间。

3\. VU和TPS换算
============

*   简单例子：在术语中解释了TPS是每秒事务数，但是事务时要靠虚拟用户做出来的，假如1个虚拟用户在1秒内完成1笔事务，那么TPS明显就是1；如果某笔业务响应时间是1ms,那么1个用户在1秒内能完成1000笔事务，TPS就是1000了；如果某笔业务响应时间是1s,那么1个用户在1秒内只能完成1笔事务，要想达到1000TPS，至少需要1000个用户；因此可以说1个用户可以产生1000TPS，1000个用户也可以产生1000TPS，无非是看响应时间快慢。
    
*   复杂公式：试想一下复杂场景，多个脚本，每个脚本里面定义了多个事务（例如一个脚本里面有100个请求，我们把这100个连续请求叫做Action,只有第10个请求，第20个请求分别定义了事务10和事务20）具体公式如下：符号代表意义：Vui表示的是第i个脚本使用的并发用户数Rtj表示的是第i个脚本第j个事务花费的时间，此时间会影响整个Action时间Rti表示的是第i个脚本一次完成所有操作的时间，即Action时间n 表示的是第n个脚本m 表示的是每个脚本中m个事务
    

那么第j个事务的TPS = Vui/Rti

总的TPS=[![](//docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/cn/pts/0.3.51/assets/pic/technology-system/vu-rt-tps/totaltps.png)](//docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/cn/pts/0.3.51/assets/pic/technology-system/vu-rt-tps/totaltps.png)

4\. 如何获取VU和TPS
==============

*   并发用户数(VU)获取新系统：没有历史数据作参考，只能通过业务部门进行评估。旧系统：对于已经上线的系统，可以选取高峰时刻，在一定时间内使用系统的人数，这些人数认为属于在线用户数，并发用户数取10%就可以了，例如在半个小时内，使用系统的用户数为10000，那么取10%作为并发用户数基本就够了。
    
*   TPS获取新系统：没有历史数据作参考，只能通过业务部门进行评估。旧系统：对于已经上线的系统，可以选取高峰时刻，在5分钟或10分钟内，获取系统每笔交易的业务量和总业务量，按照单位时间内完成的笔数计算出TPS，即业务笔数/单位时间（5_60或10_60）。
    

5\. 如何评价系统的性能
=============

**针对服务器端的性能，以TPS为主来衡量系统的性能**，并发用户数为辅来衡量系统的性能，如果必须要用并发用户数来衡量的话，需要一个前提，那就是交易在多长时间内完成，因为在系统负载不高的情况下，将思考时间(思考时间的值等于交易响应时间)加到串联链路中，并发用户数基本可以增加一倍，因此用并发用户数来衡量系统的性能没太大的意义。同样的，如果系统间的吞吐能力差别很大，那么同样的并发下TPS差距也会很大。

6\. 性能测试策略
==========

做性能测试需要一套标准化流程及测试策略。在做负载测试的时候，传统方式一般都是按照梯度施压的方式去加用户数，避免在没有预估的情况下，一次加几万个用户，导致交易失败率非常高，响应时间非常长，已经超过了使用者忍受范围内；较为适合互联网分布式架构的方式，也是阿里的最佳实践是用TPS模式（吞吐量模式）+设置起始和目标最大量级，然后根据系统表现灵活的手工实时调速，效率更高，服务端吞吐能力的衡量一步到位。

7\. 总结
======

*   系统的性能由TPS决定，跟并发用户数没有多大关系。
*   系统的最大TPS是一定的（在一个范围内），但并发用户数不一定，可以调整。
*   建议性能测试的时候，不要设置过长的思考时间，以最坏的情况下对服务器施压。
*   一般情况下，大型系统（业务量大、机器多）做压力测试，10000～50000个用户并发，中小型系统做压力测试，5000个用户并发比较常见。
