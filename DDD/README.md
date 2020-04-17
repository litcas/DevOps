# DDD 领域驱动设计

> DDD（Domain-Driven Design 领域驱动设计）是由Eric Evans最先提出，目的是对软件所涉及到的领域进行建模，以应对系统规模过大时引起的软件复杂性的问题。整个过程大概是这样的，开发团队和领域专家一起通过 通用语言(Ubiquitous Language)去理解和消化领域知识，从领域知识中提取和划分为一个一个的子领域（核心子域，通用子域，支撑子域），并在子领域上建立模型，再重复以上步骤，这样周而复始，构建出一套符合当前领域的模型。

#### 开发目标

依靠领域驱动设计的设计思想，通过事件风暴建立领域模型，合理划分领域逻辑和物理边界，建立领域对象及服务矩阵和服务架构图，定义符合DDD分层架构思想的代码结构模型，保证业务模型与代码模型的一致性。通过上述设计思想、方法和过程，指导团队按照DDD设计思想完成微服务设计和开发。

* 1、拒绝泥球小单体、拒绝污染功能与服务、拒绝一加功能排期一个月
* 2、架构出高可用极易符合互联网高速迭代的应用服务
* 3、物料化、组装化、可编排的服务，提高人效

#### 服务架构

* 应用层{application}

  * 应用服务位于应用层。用来表述应用和用户行为，负责服务的组合、编排和转发，负责处理业务用例的执行顺序以及结果的拼装。
  * 应用层的服务包括应用服务和领域事件相关服务。
  * 应用服务可对微服务内的领域服务以及微服务外的应用服务进行组合和编排，或者对基础层如文件、缓存等数据直接操作形成应用服务，对外提供粗粒度的服务。
  * 领域事件服务包括两类：领域事件的发布和订阅。通过事件总线和消息队列实现异步数据传输，实现微服务之间的解耦。



* 领域层{domain}

  * 领域服务位于领域层，为完成领域中跨实体或值对象的操作转换而封装的服务，领域服务以与实体和值对象相同的方式参与实施过程。
  * 领域服务对同一个实体的一个或多个方法进行组合和封装，或对多个不同实体的操作进行组合或编排，对外暴露成领域服务。领域服务封装了核心的业务逻辑。实体自身的行为在实体类内部实现，向上封装成领域服务暴露。
  * 为隐藏领域层的业务逻辑实现，所有领域方法和服务等均须通过领域服务对外暴露。
  * 为实现微服务内聚合之间的解耦，原则上禁止跨聚合的领域服务调用和跨聚合的数据相互关联。



* 基础层{infrastructrue}

  * 基础服务位于基础层。为各层提供资源服务（如数据库、缓存等），实现各层的解耦，降低外部资源变化对业务逻辑的影响。
  * 基础服务主要为仓储服务，通过依赖反转的方式为各层提供基础资源服务，领域服务和应用服务调用仓储服务接口，利用仓储实现持久化数据对象或直接访问基础资源。



* 接口层{interfaces}

  * 接口服务位于用户接口层，用于处理用户发送的Restful请求和解析用户输入的配置文件等，并将信息传递给应用层


#### 代码示例

```java
itstack-demo-ddd-01
└── src
    ├── main
    │   ├── java
    │   │   └── org.itstack.demo
    │   │       ├── application
    │   │       │	├── event
    │   │       │	│   └── ApplicationRunner.java	
    │   │       │	└── service
    │   │       │	    └── UserService.java	
    │   │       ├── domain
    │   │       │	├── model
    │   │       │	│   ├── aggregates
    │   │       │	│   │   └── UserRichInfo.java	
    │   │       │	│   └── vo
    │   │       │	│       ├── UserInfo.java	
    │   │       │	│       └── UserSchool.java	
    │   │       │	├── repository
    │   │       │	│   └── IuserRepository.java	
    │   │       │	└── service
    │   │       │	    └── UserServiceImpl.java	
    │   │       ├── infrastructure
    │   │       │	├── dao
    │   │       │	│   ├── impl
    │   │       │	│   │   └── UserDaoImpl.java	
    │   │       │	│   └── UserDao.java	
    │   │       │	├── po
    │   │       │	│   └── UserEntity.java	
    │   │       │	├── repository
    │   │       │	│   ├── mysql
    │   │       │	│   │   └── UserMysqlRepository.java
    │   │       │	│   ├── redis
    │   │       │	│   │   └── UserRedisRepository.java		
    │   │       │	│   └── UserRepository.java	
    │   │       │	└── util
    │   │       │	    └── RdisUtil.java
    │   │       ├── interfaces
    │   │       │	├── dto
    │   │       │	│	└── UserInfoDto.java	
    │   │       │	└── facade
    │   │       │		└── DDDController.java
    │   │       └── DDDApplication.java
    │   ├── resources	
    │   │   └── application.yml
    │   └── webapp	
    │       └── WEB-INF
    │        	└── index.jsp	
    └── test
         └── java
             └── org.itstack.demo.test
                 └── ApiTest.java

```