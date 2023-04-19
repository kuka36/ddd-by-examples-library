[![CircleCI](https://circleci.com/gh/ddd-by-examples/library.svg?style=svg)](https://circleci.com/gh/ddd-by-examples/library)
[![Code Coverage](https://codecov.io/gh/ddd-by-examples/library/branch/master/graph/badge.svg)](https://codecov.io/gh/ddd-by-examples/library)

# Table of contents 目录

1. [About](#about)
2. [Domain description 领域描述](#domain-description)
3. [General assumptions 一般假设](#general-assumptions)  
    3.1 [Process discovery 流程发现](#process-discovery)  
    3.2 [Project structure and architecture 项目结构和架构](#project-structure-and-architecture)    
    3.3 [Aggregates 聚合](#aggregates)  
    3.4 [Events 事件](#events)  
    3.4.1 [Events in Repositories 仓库中的事件](#events-in-repositories)   
    3.5 [ArchUnit](#archunit)  
    3.6 [Functional thinking 功能思维](#functional-thinking)  
    3.7 [No ORM](#no-orm)  
    3.8 [Architecture-code gap 架构-代码差距](#architecture-code-gap)  
    3.9 [Model-code gap 模型-代码差距](#model-code-gap)   
    3.10 [Spring](#spring)  
    3.11 [Tests 测试](#tests)  
4. [How to contribute](#how-to-contribute)
5. [References](#references)

## About

This is a project of a library, driven by real [business requirements](#domain-description).  
这是一个由实际业务需求驱动的图书馆项目。  
We use techniques strongly connected with Domain Driven Design, Behavior-Driven Development,
Event Storming, User Story Mapping.  
我们使用与领域驱动设计、行为驱动开发、事件风暴和用户故事映射密切相关的技术。

## Domain description 领域描述

A public library allows patrons to place books on hold at its various library branches.  
公共图书馆允许读者在各个分馆预留图书。  
Available books can be placed on hold only by one patron at any given point in time.  
在任何给定时间点，只有一个读者可以预留一本可用的书。  
Books are either circulating or restricted, and can have retrieval or usage fees.  
书籍要么流通，要么受限，并且可以收取检索费或使用费。   
A restricted book can only be held by a researcher patron.   
限制类图书只能由研究员读者预留。  
A regular patron is limited to five holds at any given moment,   
普通读者在任何时刻最多可以预留五本书，  
while a researcher patron is allowed an unlimited number of holds.   
而研究员读者可以预留无限数量的书。  

An open-ended book hold is active until the patron checks out the book, at which time it
is completed.  
一个无限期的图书预留在读者借阅图书时生效，此时预留完成。  
A closed-ended book hold that is not completed within a fixed number of 
days after it was requested will expire.  
一个有限期的图书预留如果在要求的固定天数内未完成，将会过期。  
This check is done at the beginning of a day by 
taking a look at daily sheet with expiring holds.  
每天开始时，通过查看每日过期预留表来进行检查。  
Only a researcher patron can request an open-ended hold duration.  
只有研究员读者可以要求无限期的预留。  
Any patron with more than two overdue checkouts at a library
branch will get a rejection if trying a hold at that same library branch.  
在同一分馆有超过两本逾期未还图书的读者，如果尝试在该分馆预留图书，将会被拒绝。  
A book can be checked out for up to 60 days.  
图书最多可借阅60天。  
Check for overdue checkouts is done by taking a look at
daily sheet with overdue checkouts.  
通过查看每日逾期未还图书表来检查逾期未还图书。  

Patron interacts with his/her current holds, checkouts, etc.
by taking a look at patron profile.  
读者通过查看读者档案来查看当前预留、借阅等情况。  
Patron profile looks like a daily sheet, but the
information there is limited to one patron and is not necessarily daily.  
读者档案看起来像每日表格，但其中的信息仅限于一个读者，不一定是每天的。  
Currently a patron can see current holds (not canceled nor expired) and current checkouts (including overdue).  
目前，读者可以查看当前预留（未取消且未过期）和当前借阅（包括逾期）。  
Also, he/she is able to hold a book and cancel a hold.  
此外，他/她还可以预留图书和取消预留。  

How actually a patron knows which books are there to lend?   
那么，读者如何知道哪些图书可供借阅呢？   
Library has its catalogue of books where books are added together with their specific instances.  
图书馆有自己的图书目录，图书和它们的具体实例一起被添加进去。  
A specific book instance of a book can be added only if there is book with matching ISBN already in
the catalogue.  
只有当目录中已有与之匹配的ISBN的书时，才能添加特定书的实例。  
Book must have non-empty title and price.  
图书必须有非空标题和价格。  
At the time of adding an instance we decide whether it will be Circulating or Restricted.  
在添加实例时，我们决定它将是流通还是限制。  
This enables us to have book with same ISBN as circulated and restricted at the same time (for instance,
there is a book signed by the author that we want to keep as Restricted)  
这使我们能够同时拥有相同ISBN的流通和限制图书（例如，我们想将作者签名的书作为限制图书）。

## General assumptions 一般假设

### Process discovery 流程发现

The first thing we started with was domain exploration with the help of Big Picture EventStorming.  
我们首先从大局观的事件风暴（Big Picture EventStorming）开始探索领域。

The description you found in the previous chapter, landed on our virtual wall:    
在上一章中，您找到的描述出现在我们的虚拟墙上：
![Event Storming Domain description 事件风暴领域描述](docs/images/eventstorming-domain-desc.png)   

The EventStorming session led us to numerous discoveries, modeled with the sticky notes:  
事件风暴会议让我们发现了许多模型，用便签纸表示：
![Event Storming Big Picture 事件风暴大局观](docs/images/eventstorming-big-picture.jpg)   

During the session we discovered following definitions:  
在会议期间，我们发现了以下定义：
![Event Storming Definitions 事件风暴定义](docs/images/eventstorming-definitions.png)    

This made us think of real life scenarios that might happen.   
这让我们想到了现实生活中可能发生的场景。  
We discovered them described with the help of the **Example mapping**:  
我们在**示例映射**的帮助下发现了它们的描述：
![Example mapping 示例映射](docs/images/example-mapping.png)  

This in turn became the base for our *Design Level* sessions, where we analyzed each example:  
反过来，这成为了我们设计层次会议的基础，在这些会议中，我们分析了每个示例：
![Example mapping 示例映射](docs/images/eventstorming-design-level.jpg)  

Please follow the links below to get more details on each of the mentioned steps:  
请点击下面的链接以获取有关上述每个步骤的更多详细信息：
- [Big Picture EventStorming 大局观事件风暴](./docs/big-picture.md)
- [Example Mapping 示例映射](docs/example-mapping.md)
- [Design Level EventStorming 设计层次事件风暴](docs/design-level.md)

### Project structure and architecture 项目结构与架构
At the very beginning, not to overcomplicate the project, we decided to assign each bounded context
to a separate package, which means that the system is a modular monolith.  
在最开始的时候，为了不使项目过于复杂，我们决定将每个有界上下文分配到单独的包中，这意味着系统是一个模块化的单体架构。  
There are no obstacles, though, to put contexts into maven modules or finally into microservices.  
然而，我们并没有障碍将上下文放入Maven模块或最终放入微服务中。  

Bounded contexts should (amongst others) introduce autonomy in the sense of architecture.   
有界上下文应该（除其他因素外）在架构上引入自治。  
Thus, each module encapsulating the context has its own local architecture aligned to problem complexity.  
因此，封装上下文的每个模块都有自己的本地架构，以解决问题的复杂性。  

In the case of a context, where we identified true business logic (**lending**)   
在我们确定了真正的业务逻辑（借阅）的上下文中，  
we introduced a domain model that is a simplified (for the purpose of the project) abstraction of the reality   
我们引入了一个领域模型，它是现实的简化（为了项目的目的），  
and utilized hexagonal architecture.   
并利用了六边形架构。  
In the case of a context, that during Event Storming turned out to lack any complex
domain logic,   
在事件风暴期间发现缺乏任何复杂领域逻辑的（目录）上下文中，  
we applied CRUD-like local architecture.    
我们应用了类似于CRUD的本地架构。  

![Architecture 架构](docs/images/architecture-big-picture.png) 

If we are talking about hexagonal architecture,   
如果我们谈论的是六边形架构，  
it lets us separate domain and application logic from frameworks (and infrastructure).  
它可以让我们将领域和应用程序逻辑与框架（和基础设施）分开。  
What do we gain with this approach?   
我们通过这种方法获得了什么？  
Firstly, we can unit test most important part of the application - **business logic** - usually without the need to stub any dependency.  
首先，我们可以对应用程序的最重要的部分——业务逻辑——进行单元测试，通常不需要存根任何依赖项。  
Secondly, we create ourselves an opportunity to adjust infrastructure layer without the worry of
breaking the core functionality.  
其次，我们为自己创建了一个机会，可以调整基础架构层，而不必担心破坏核心功能。  
In the infrastructure layer we intensively use Spring Framework
as probably the most mature and powerful application framework with an incredible test support.  
在基础架构层，我们密集地使用Spring框架，作为可能是最成熟和功能强大的应用程序框架，具有令人难以置信的测试支持。  
More information about how we use Spring you will find [here](#spring).  
有关我们如何使用Spring的更多信息，请参见[这里](#spring)。   
As we already mentioned, the architecture was driven by Event Storming sessions.   
正如我们已经提到的，架构是由事件风暴会议推动的。  
Apart from identifying contexts and their complexity, we could also make a decision that we separate read and write models (CQRS).  
除了确定上下文及其复杂性之外，我们还可以决定将读取和写入模型（CQRS）分开。  
As an example you can have a look at **Patron Profiles** and *Daily Sheets*.  
例如，您可以查看Patron Profiles和Daily Sheets。

### Aggregates 聚合
Aggregates discovered during Event Storming sessions communicate with each other with events.  
在事件风暴会议期间发现的聚合使用事件相互通信。  
There is a contention, though, should they be consistent immediately or eventually?  
然而，他们应该立即一致还是最终一致存在争议？  
As aggregates in general determine business boundaries, eventual consistency sounds like a better choice,   
由于聚合通常确定业务边界，因此最终一致性听起来更好，  
but choices in software are never costless.  
但在软件中做出选择从来都不是没有成本的。  
Providing eventual consistency requires some infrastructural tools,   
提供最终一致性需要一些基础设施工具，  
like message broker or event store.  
例如消息代理或事件存储。  
That's why we could (and did) start with immediate consistency.   
这就是为什么我们可以（并且确实）从立即一致性开始。  

> Good architecture is the one which postpones all important decisions  
  好的架构是可以推迟所有重要决策的架构

... that's why we made it easy to change the consistency model,   
...这就是为什么我们使更改一致性模型变得容易，  
providing tests for each option, including basic implementations based on **DomainEvents** interface,   
为每个选项提供测试，包括基于DomainEvents接口的基本实现，  
which can be adjusted to our needs and toolset in future.  
这些实现可以根据我们的需求和工具集进行调整。  
Let's have a look at following examples:  
让我们看看以下示例：  

* Immediate consistency 立即一致性
    ```groovy
    def 'should synchronize Patron, Book and DailySheet with events'() {
        given:
            bookRepository.save(book)
        and:
            patronRepo.publish(patronCreated())
        when:
            patronRepo.publish(placedOnHold(book))
        then:
            patronShouldBeFoundInDatabaseWithOneBookOnHold(patronId)
        and:
            bookReactedToPlacedOnHoldEvent()
        and:
            dailySheetIsUpdated()
    }
    
    boolean bookReactedToPlacedOnHoldEvent() {
        return bookRepository.findBy(book.bookId).get() instanceof BookOnHold
    }
    
    boolean dailySheetIsUpdated() {
        return new JdbcTemplate(datasource).query("select count(*) from holds_sheet s where s.hold_by_patron_id = ?",
                [patronId.patronId] as Object[],
                new ColumnMapRowMapper()).get(0)
                .get("COUNT(*)") == 1
    }
    ```
   _Please note that here we are just reading from database right after events are being published  
    请注意，这里我们只是在事件发布后立即从数据库中读取。_

   Simple implementation of the event bus is based on Spring application events:  
   事件总线的简单实现基于Spring应用程序事件：  
    ```java
    @AllArgsConstructor
    public class JustForwardDomainEventPublisher implements DomainEvents {
    
        private final ApplicationEventPublisher applicationEventPublisher;
    
        @Override
        public void publish(DomainEvent event) {
            applicationEventPublisher.publishEvent(event);
        }
    }
    ```

* Eventual consistency
    ```groovy
    def 'should synchronize Patron, Book and DailySheet with events'() {
        given:
            bookRepository.save(book)
        and:
            patronRepo.publish(patronCreated())
        when:
            patronRepo.publish(placedOnHold(book))
        then:
            patronShouldBeFoundInDatabaseWithOneBookOnHold(patronId)
        and:
            bookReactedToPlacedOnHoldEvent()
        and:
            dailySheetIsUpdated()
    }
    
    void bookReactedToPlacedOnHoldEvent() {
        pollingConditions.eventually {
            assert bookRepository.findBy(book.bookId).get() instanceof BookOnHold
        }
    }
    
    void dailySheetIsUpdated() {
        pollingConditions.eventually {
            assert countOfHoldsInDailySheet() == 1
        }
    }
    ```
    _Please note that the test looks exactly the same as previous one,   
    请注意，测试看起来与之前的测试完全相同，  
    but now we utilized Groovy's **PollingConditions** to perform asynchronous functionality tests  
    但现在我们利用了Groovy的PollingConditions来执行异步功能测试。_

    Sample implementation of event bus is following:  
    事件总线的示例实现如下：   
    ```java
    @AllArgsConstructor
    public class StoreAndForwardDomainEventPublisher implements DomainEvents {
    
        private final JustForwardDomainEventPublisher justForwardDomainEventPublisher;
        private final EventsStorage eventsStorage;
    
        @Override
        public void publish(DomainEvent event) {
            eventsStorage.save(event);
        }
    
        @Scheduled(fixedRate = 3000L)
        @Transactional
        public void publishAllPeriodically() {
            List<DomainEvent> domainEvents = eventsStorage.toPublish();
            domainEvents.forEach(justForwardDomainEventPublisher::publish);
            eventsStorage.published(domainEvents);
        }
    }
    ```

To clarify, we should always aim for aggregates that can handle a business operation atomically
(transactionally if you like),   
为了澄清，我们应该始终以原子方式处理业务操作的聚合（如果您喜欢，可以进行事务处理），  
so each aggregate should be as independent and decoupled from other
aggregates as possible.   
因此每个聚合应尽可能独立且与其他聚合解耦。  
Thus, eventual consistency is promoted. As we already mentioned, it comes with some tradeoffs,   
因此，推广最终一致性。正如我们已经提到的，它有一些权衡，  
so from the pragmatic point of view immediate consistency is also a choice.   
因此从实用的角度来看，立即一致性也是一种选择。  
You might ask yourself a question now: _What if I don't have any events yet?_.   
现在您可能会问自己一个问题：_如果我还没有任何事件怎么办？_   
Well, a pragmatic approach would be to encapsulate the communication between aggregates in a _Service-like_ class,  
好吧，实用的方法是在一个类似于“服务”的类中封装聚合之间的通信，  
where you could call proper aggregates line by line explicitly.  
在其中您可以显式地逐行调用适当的聚合。 

### Events 事件
Talking about inter-aggregate communication, we must remember that events reduce coupling, but don't remove
it completely.   
在谈到聚合之间的通信时，我们必须记住事件可以减少耦合，但不能完全消除它。  
Thus, it is very vital to share(publish) only those events, that are necessary for other
aggregates to exist and function.   
因此，非常重要的是只共享（发布）那些对其他聚合存在和运行必要的事件。  
Otherwise there is a threat that the level of coupling will increase
introducing **feature envy**,   
否则，耦合的级别将增加，引入功能嫉妒，  
because other aggregates might start using those events to perform actions
they are not supposed to perform.   
因为其他聚合可能会开始使用这些事件执行它们不应执行的操作。  
A solution to this problem could be the distinction of domain events
and integration events, which will be described here soon.  
解决这个问题的方法可能是区分领域事件和集成事件，这将很快在这里描述。

### Events in Repositories 存储库中的事件
Repositories are one of the most popular design pattern.   
存储库是最流行的设计模式之一。  
They abstract our domain model from data layer.   
它们将我们的领域模型与数据层抽象分离。  
In other words, they deal with state.   
换句话说，它们处理状态。  
That said, a common use-case is when we pass a new state to our repository,  
也就是说，常见的用例是我们将新状态传递给存储库，以便将其持久化。  
so that it gets persisted. It may look like so:  
它可能看起来像这样：

```java
public class BusinessService {
   
    private final PatronRepository patronRepository;
    
    void businessMethod(PatronId patronId) {
        Patron patron = patronRepository.findById(patronId);
        //do sth
        patronRepository.save(patron);
    }
}
```

Conceptually, between 1st and 3rd line of that business method   
从概念上讲，在该业务方法的第1行和第3行之间，  
we change state of our Patron from A to B.
This change might be calculated by dirty checking   
我们将Patron的状态从A更改为B。这种更改可能是通过脏检查计算的，  
or we might just override entire Patron state in the database.   
或者我们可能只是在数据库中覆盖整个Patron状态。  
Third option is _Let's make implicit explicit_   
第三个选择是“让隐含的显式化”，  
and actually call this state change A->B an **event**.   
实际上将此状态更改A->B称为事件。  
After all, event-driven architecture is all about promoting state changes as domain events.
毕竟，事件驱动的架构是关于将状态更改作为领域事件进行广播的。

Thanks to this our domain model may become immutable and just return events as results of invoking a command like so:
由此，我们的领域模型可以变得不可变，并且只返回作为调用命令的结果的事件，如下所示：

```java
public BookPlacedOnHold placeOnHold(AvailableBook book) {
      ...
}
```

And our repository might operate directly on events like so:  
我们的存储库可能直接在事件上运行，如下所示：  

```java
public interface PatronRepository {
     void save(PatronEvent event) {
}
```

### ArchUnit

One of the main components of a successful project is technical leadership that lets the team go in the right direction.   
成功项目的主要组成部分之一是技术领导力，它可以让团队朝着正确的方向前进。  
Nevertheless, there are tools that can support teams in keeping the code clean and protect the
architecture,   
然而，有些工具可以支持团队保持代码的清洁并保护架构，  
so that the project won't become a Big Ball of Mud, and thus will be pleasant to develop and
to maintain.   
以便项目不会变成一个大泥球，从而开发和维护起来更加愉快。  
The first option, the one we proposed, is [ArchUnit](https://www.archunit.org/) - a Java architecture
test tool.   
我们提出的第一个选择是ArchUnit——一个Java架构测试工具。  
ArchUnit lets you write unit tests of your architecture, so that it is always consistent with initial vision.   
ArchUnit可以让您编写架构单元测试，以便它始终与初始设计一致。  
Maven modules could be an alternative as well, but let's focus on the former.  
Maven模块也可以作为替代方案，但让我们专注于前者。

In terms of hexagonal architecture, it is essential to ensure, that we do not mix different levels of
abstraction (hexagon levels):  
在六边形架构方面，重要的是确保我们不混合不同的抽象级别（六边形级别）：

```java 
@ArchTest
public static final ArchRule model_should_not_depend_on_infrastructure =
    noClasses()
        .that()
        .resideInAPackage("..model..")
        .should()
        .dependOnClassesThat()
        .resideInAPackage("..infrastructure..");
```      
and that frameworks do not affect the domain model   
并且框架不会影响领域模型：

```java
@ArchTest
public static final ArchRule model_should_not_depend_on_spring =
    noClasses()
        .that()
        .resideInAPackage("..io.pillopl.library.lending..model..")
        .should()
        .dependOnClassesThat()
        .resideInAPackage("org.springframework..");
```    

### Functional thinking 函数式思维
When you look at the code you might find a scent of functional programming. Although we do not follow
a _clean_ FP, we try to think of business processes as pipelines or workflows, utilizing functional style through
following concepts.  
当你看代码时，可能会发现一些函数式编程的痕迹。虽然我们并不遵循“干净”的 FP，但我们尝试将业务流程视为管道或工作流，并通过以下概念利用函数式风格。

_Please note that this is not a reference project for FP.  
请注意，这不是 FP 的参考项目。_

#### Immutable objects 不可变对象
Each class that represents a business concept is immutable, thanks to which we:  
代表业务概念的每个类都是不可变的，这使我们能够：  
* provide full encapsulation and objects' states protection,  
  提供完全的封装和对象状态保护，  
* secure objects for multithreaded access,  
  为多线程访问安全的对象提供保障，  
* control all side effects much clearer.   
  更清晰地控制所有副作用。  

#### Pure functions 纯函数
We model domain operations, discovered in Design Level Event Storming,   
我们将在设计层事件风暴中发现的领域操作建模为纯函数，  
as pure functions, and declare them in both domain and application layers in the form of Java's functional interfaces.   
并以 Java 的函数接口形式在领域和应用层中声明它们。  
Their implementations are placed in infrastructure layer as ordinary methods with side effects.   
它们的实现作为带有副作用的普通方法放置在基础设施层中。  
Thanks to this approach we can follow the abstraction of ubiquitous language explicitly,   
由于这种方法，我们可以明确遵循普遍语言的抽象，  
and keep this abstraction implementation-agnostic.   
并保持这种抽象的实现无关性。  
As an example, you could have a look at `FindAvailableBook` interface and its implementation:
例如，您可以查看 FindAvailableBook 接口及其实现：

```java
@FunctionalInterface
public interface FindAvailableBook {

    Option<AvailableBook> findAvailableBookBy(BookId bookId);
}
```

```java
@AllArgsConstructor
class BookDatabaseRepository implements FindAvailableBook {

    private final JdbcTemplate jdbcTemplate;

    @Override
    public Option<AvailableBook> findAvailableBookBy(BookId bookId) {
        return Match(findBy(bookId)).of(
                Case($Some($(instanceOf(AvailableBook.class))), Option::of),
                Case($(), Option::none)
        );
    }  

    Option<Book> findBy(BookId bookId) {
        return findBookById(bookId)
                .map(BookDatabaseEntity::toDomainModel);
    }

    private Option<BookDatabaseEntity> findBookById(BookId bookId) {
        return Try
                .ofSupplier(() -> of(jdbcTemplate.queryForObject("SELECT b.* FROM book_database_entity b WHERE b.book_id = ?",
                                      new BeanPropertyRowMapper<>(BookDatabaseEntity.class), bookId.getBookId())))
                .getOrElse(none());
    }  
} 
```
    
#### Type system 类型系统
_Type system - like_ modelling - we modelled each domain object's state discovered during EventStorming as separate
classes: `AvailableBook`, `BookOnHold`, `CheckedOutBook`.   
类型系统 - 类似于建模 - 我们将在 EventStorming 中发现的每个领域对象状态建模为单独的类：AvailableBook、BookOnHold、CheckedOutBook。  
With this approach we provide much clearer abstraction than
having a single `Book` class with an enum-based state management.   
采用这种方法，我们提供了比使用基于枚举的状态管理的单个 Book 类更清晰的抽象。  
Moving the logic to these specific classes brings
Single Responsibility Principle to a different level.   
将逻辑移到这些特定的类中将单一责任原则提升到了不同的层次。  
Moreover, instead of checking invariants in every business method
we leave the role to the compiler.   
此外，我们将不变量的检查留给编译器，而不是在每个业务方法中进行检查。  
As an example, please consider following scenario: _you can place on hold only a book
that is currently available_. We could have done it in a following way:  
例如，请考虑以下场景：只有当前可用的书籍才能被预订。我们可以这样做：

```java
public Either<BookHoldFailed, BookPlacedOnHoldEvents> placeOnHold(Book book) {
  if (book.status == AVAILABLE) {  
      ...
  }
}
```
but we use the _type system_ and declare method of following signature   
但我们使用类型系统，并声明以下签名的方法：  
```java
public Either<BookHoldFailed, BookPlacedOnHoldEvents> placeOnHold(AvailableBook book) {
      ...
}
```  
The more errors we discover at compile time the better.  
我们在编译时发现的错误越多，就越好。  
Yet another advantage of applying such type system is that we can represent business flows and state transitions
with functions much easier. As an example, following functions:  
应用这种类型系统的另一个优点是，我们可以更轻松地使用函数表示业务流程和状态转换。例如，以下函数：
```
placeOnHold: AvailableBook -> BookHoldFailed | BookPlacedOnHold
cancelHold: BookOnHold -> BookHoldCancelingFailed | BookHoldCanceled
``` 
are much more concise and descriptive than these:  
比这些更加简洁和描述性:  
```
placeOnHold: Book -> BookHoldFailed | BookPlacedOnHold
cancelHold: Book -> BookHoldCancelingFailed | BookHoldCanceled
```
as here we have a lot of constraints hidden within function implementations.  
因为在这里我们有很多约束隐藏在函数实现中。

Moreover if you think of your domain as a set of operations (functions) that are being executed on business objects
(aggregates)   
此外，如果您将领域视为在业务对象（聚合）上执行的一组操作（函数），  
you don't think of any execution model (like async processing).   
It is fine, because you don't have to.  
则不用考虑任何执行模型（例如异步处理）。这没问题，因为您不必考虑。  
Domain functions are free from I/O operations, async, and other side-effects-prone things, which are put into the
infrastructure layer.   
领域函数不受 I/O 操作、异步和其他易受副作用影响的事物的限制，这些事物被放入基础设施层。  
Thanks to this, we can easily test them without mocking mentioned parts.  
由于这一点，我们可以轻松地测试它们，而不需要模拟上述部分。

#### Monads 单子
Business methods might have different results.   
业务方法可能具有不同的结果。  
One might return a value or a `null`, throw an exception when something
unexpected happens or just return different objects under different circumstances.   
一个方法可能返回一个值或 null，当发生意外情况时抛出异常，或在不同情况下返回不同的对象。  
All those situations are typical
to object-oriented languages like Java, but do not fit into functional style.   
所有这些情况都是 Java 等面向对象语言的典型问题，但不适合函数式风格。  
We are dealing with this issues
with monads (monadic containers provided by [Vavr](https://www.vavr.io)):  
我们使用单子（由 Vavr 提供的单子容器）解决这些问题：

* When a method returns optional value, we use the `Option` monad:  
  当方法返回可选值时，我们使用 Option 单子：

    ```java
    Option<Book> findBy(BookId bookId) {
        ...
    }
    ```

* When a method might return one of two possible values, we use the `Either` monad:  
  当方法可能返回两个可能值之一时，我们使用 Either 单子：

    ```java
    Either<BookHoldFailed, BookPlacedOnHoldEvents> placeOnHold(AvailableBook book) {
        ...
    }
    ```

* When an exception might occur, we use `Try` monad:  
  当可能发生异常时，我们使用 Try 单子：

    ```java
    Try<Result> placeOnHold(@NonNull PlaceOnHoldCommand command) {
        ...
    }
    ```

Thanks to this, we can follow the functional programming style, but we also enrich our domain language and
make our code much more readable for the clients.  
由此，我们可以遵循函数式编程风格，但也丰富了我们的领域语言，并使我们的代码对客户更加可读。

#### Pattern Matching 模式匹配
Depending on a type of a given book object we often need to perform different actions.   
根据给定书籍对象的类型，我们经常需要执行不同的操作。  
Series of if/else or switch/case statements could be a choice,   
一系列 if/else 或 switch/case 语句可能是一种选择，   
but it is the pattern matching that provides the most conciseness and flexibility.   
但模式匹配提供了最简洁和灵活的方式。  
With the code like below we can check numerous patterns against objects and access their constituents,   
使用以下代码，我们可以针对对象检查多种模式并访问其组成部分，  
so our code has a minimal dose of language-construct noise:  
因此我们的代码具有最小的语言构造噪声：

```java
private Book handleBookPlacedOnHold(Book book, BookPlacedOnHold bookPlacedOnHold) {
    return API.Match(book).of(
        Case($(instanceOf(AvailableBook.class)), availableBook -> availableBook.handle(bookPlacedOnHold)),
        Case($(instanceOf(BookOnHold.class)), bookOnHold -> raiseDuplicateHoldFoundEvent(bookOnHold, bookPlacedOnHold)),
        Case($(), () -> book)
    );
}
```

### (No) ORM
If you run `mvn dependency:tree` you won't find any JPA implementation.   
如果您运行 mvn dependency:tree，您将找不到任何 JPA 实现。  
Although we think that ORM solutions (like Hibernate)
are very powerful and useful, we decided not to use them,   
虽然我们认为 ORM 解决方案（如 Hibernate）非常强大和有用，但我们决定不使用它们，  
as we wouldn't utilize their features.   
因为我们不会利用它们的功能。  
What features are talking about? Lazy loading, caching, dirty checking.   
我们在谈论什么功能？懒加载、缓存、脏数据检查。  
Why don't we need them? We want to have more control over SQL queries  
为什么我们不需要它们？我们希望更多地控制 SQL 查询，  
and minimize the object-relational impedance mismatch ourselves.   
并自己最小化对象关系不匹配。  
Moreover, thanks to relatively
small aggregates, containing as little data as it is required to protect the invariants,   
此外，由于相对较小的聚合只包含所需的最少数据以保护不变量，  
we don't need the  lazy loading mechanism either.  
因此我们也不需要懒加载机制。

With Hexagonal Architecture we have the ability to separate domain and persistence models and test them
independently.   
使用六边形架构，我们可以将域模型和持久化模型分开并独立测试它们。  
Moreover, we can also introduce different persistence strategies for different aggregates.   
此外，我们还可以为不同的聚合引入不同的持久化策略。

In this project, we utilize both plain SQL queries and `JdbcTemplate`   
在这个项目中，我们利用了纯 SQL 查询和 JdbcTemplate，  
and use new and very promising project called Spring Data JDBC,   
并使用了一个非常有前途的新项目，称为 Spring Data JDBC，  
that is free from the JPA-related overhead mentioned before.  
它不受前面提到的 JPA 相关开销的影响。
Please find below an example of a repository:  
请参见下面的存储库示例：

```java
interface PatronEntityRepository extends CrudRepository<PatronDatabaseEntity, Long> {

    @Query("SELECT p.* FROM patron_database_entity p where p.patron_id = :patronId")
    PatronDatabaseEntity findByPatronId(@Param("patronId") UUID patronId);

}
```

At the same time we propose other way of persisting aggregates, with plain SQL queries and `JdbcTemplate`:  
与此同时，我们提出了使用纯 SQL 查询和 JdbcTemplate 持久化聚合的其他方式。

```java
@AllArgsConstructor
class BookDatabaseRepository implements BookRepository, FindAvailableBook, FindBookOnHold {

    private final JdbcTemplate jdbcTemplate;

    @Override
    public Option<Book> findBy(BookId bookId) {
        return findBookById(bookId)
                .map(BookDatabaseEntity::toDomainModel);
    }

    private Option<BookDatabaseEntity> findBookById(BookId bookId) {
        return Try
                .ofSupplier(() -> of(jdbcTemplate.queryForObject("SELECT b.* FROM book_database_entity b WHERE b.book_id = ?",
                                     new BeanPropertyRowMapper<>(BookDatabaseEntity.class), bookId.getBookId())))
                .getOrElse(none());
    }
    
    ...
}
```
_Please note that despite having the ability to choose different persistence implementations for aggregates
it is recommended to stick to one option within the app/team  
请注意，尽管有能力为聚合选择不同的持久化实现，但建议在应用程序/团队内坚持一种选项。_ 
    
### Architecture-code gap 架构-代码差距
We put a lot of attention to keep the consistency between the overall architecture (including diagrams) and the code structure.   
我们非常注重保持总体架构（包括图表）和代码结构之间的一致性。  
Having identified bounded contexts we could organize them in modules (packages, to be more specific).   
通过确定有界上下文，我们可以将它们组织成模块（包，更具体地说）。  
Thanks to this we gain the famous microservices' autonomy, while having a monolithic application.   
由此，我们获得了著名的微服务自治性，同时拥有了单体应用程序。  
Each package has well defined public API, encapsulating all implementation details by using package-protected or private scopes.  
每个包都有明确定义的公共API，通过使用包保护或私有范围封装所有实现细节。  
Just by looking at the package structure:  
仅通过查看包结构

```
└── library
    ├── catalogue
    ├── commons
    │   ├── aggregates
    │   ├── commands
    │   └── events
    │       └── publisher
    └── lending
        ├── book
        │   ├── application
        │   ├── infrastructure
        │   └── model
        ├── dailysheet
        │   ├── infrastructure
        │   └── model
        ├── librarybranch
        │   └── model
        ├── patron
        │   ├── application
        │   ├── infrastructure
        │   └── model
        └── patronprofile
            ├── infrastructure
            ├── model
            └── web
```
you can see that the architecture is screaming that it has two bounded contexts: **catalogue**
and **lending**.   
你可以看到，架构表达了它有两个有界上下文：目录和借阅。  
Moreover, the **lending context** is built around five business objects: **book**,
**dailysheet**, **librarybranch**, **patron**, and **patronprofile**,   
此外，借阅上下文围绕五个业务对象构建：书籍、日报表、图书馆分支、读者和读者档案，  
while **catalogue** has no subpackages,
而目录没有子包，  
which suggests that it might be a CRUD with no complex logic inside. Please find the architecture diagram
below.  
这表明它可能是一个没有复杂逻辑的CRUD。请查看下面的组件架构图。

![Component diagram](docs/c4/component-diagram.png)

Yet another advantage of this approach comparing to packaging by layer for example is that   
与按层打包等方法相比，这种方法的另一个优点是，  
in order to deliver a functionality you would usually need to do it in one package only, which is the aforementioned
autonomy.   
为了提供功能，通常只需要在一个包中执行，这就是前面提到的自治性。  
This autonomy, then, could be transferred to the level of application as soon as we split our
_context-packages_ into separate microservices.   
然后，这种自治性可以转移到应用程序级别，只要我们将上下文包拆分为独立的微服务。  
Following this considerations, autonomy can be given away
to a product team that can take care of the whole business area end-to-end.  
遵循这些考虑，自治性可以转移到负责整个业务领域的产品团队。

### Model-code gap 模型代码差距
In our project we do our best to reduce _model-code gap_ to bare minimum.   
在我们的项目中，我们尽力将模型代码差距减少到最低限度。  
It means we try to put equal attention
to both the model and the code and keep them consistent. Below you will find some examples.  
这意味着我们尝试给予模型和代码同等的关注，并保持它们一致。下面是一些示例。

#### Placing on hold 预留 
![Placing on hold](docs/images/placing_on_hold.jpg)

Starting with the easiest part, below you will find the model classes corresponding to depicted command and events:  
从最简单的部分开始，下面是对应于所示命令和事件的模型类：

```java
@Value
class PlaceOnHoldCommand {
    ...
}
```
```java
@Value
class BookPlacedOnHold implements PatronEvent {
    ...
}
```
```java
@Value
class MaximumNumberOfHoldsReached implements PatronEvent {
    ...    
}
```
```java
@Value
class BookHoldFailed implements PatronEvent {
    ...
}
```

We know it might not look impressive now,  
我们知道现在可能看起来并不令人印象深刻，  
but if you have a look at the implementation of an aggregate,
you will see that the code reflects not only the aggregate name,   
但是如果您查看聚合的实现，您将看到代码不仅反映聚合名称，  
but also the whole scenario of `PlaceOnHold` command handling.   
还反映了“PlaceOnHold”命令处理的整个场景。  
Let us uncover the details:
让我们揭开细节：

```java
public class Patron {

    public Either<BookHoldFailed, BookPlacedOnHoldEvents> placeOnHold(AvailableBook book) {
        return placeOnHold(book, HoldDuration.openEnded());
    }
    
    ...
}    
```

The signature of `placeOnHold` method screams, that it is possible to place a book on hold only when it
is available (more information about protecting invariants by compiler you will find in [Type system section](#type-system)).  
placeOnHold方法的签名表明，只有在书籍可用时才能将书籍暂停（有关编译器保护不变量的更多信息，请参见类型系统部分）。  
Moreover, if you try to place available book on hold it can **either** fail (`BookHoldFailed`) or produce some events -
what events?  
此外，如果您尝试将可用的书籍暂停，它可能会失败（BookHoldFailed）或产生一些事件-哪些事件？

```java
@Value
class BookPlacedOnHoldEvents implements PatronEvent {
    @NonNull UUID eventId = UUID.randomUUID();
    @NonNull UUID patronId;
    @NonNull BookPlacedOnHold bookPlacedOnHold;
    @NonNull Option<MaximumNumberOfHoldsReached> maximumNumberOfHoldsReached;

    @Override
    public Instant getWhen() {
        return bookPlacedOnHold.when;
    }

    public static BookPlacedOnHoldEvents events(BookPlacedOnHold bookPlacedOnHold) {
        return new BookPlacedOnHoldEvents(bookPlacedOnHold.getPatronId(), bookPlacedOnHold, Option.none());
    }

    public static BookPlacedOnHoldEvents events(BookPlacedOnHold bookPlacedOnHold, MaximumNumberOfHoldsReached maximumNumberOfHoldsReached) {
        return new BookPlacedOnHoldEvents(bookPlacedOnHold.patronId, bookPlacedOnHold, Option.of(maximumNumberOfHoldsReached));
    }

    public List<DomainEvent> normalize() {
        return List.<DomainEvent>of(bookPlacedOnHold).appendAll(maximumNumberOfHoldsReached.toList());
    }
}
```

`BookPlacedOnHoldEvents` is a container for `BookPlacedOnHold` event,   
BookPlacedOnHoldEvents是包含BookPlacedOnHold事件的容器，  
and - if patron has 5 book placed on hold already -
`MaximumNumberOfHoldsReached` (please mind the `Option` monad).   
如果读者已经将5本书预留，则会显示MaximumNumberOfHoldsReached（请注意Option单子）。  
You can see now how perfectly the code reflects the model.  
现在，您可以看到代码如何完美地反映模型。

It is not everything, though.   
然而，这还不是全部。  
In the picture above you can also see a big rectangular yellow card with rules (policies)  
在上面的图片中，您还可以看到一个带有规则（策略）的大型矩形黄卡，  
that define the conditions that need to be fulfilled in order to get the given result.   
All those rules are implemented
这些规则定义了需要满足的条件，以获得给定的结果。  
as functions **either** allowing or rejecting the hold:  
所有这些规则都实现为允许或拒绝暂停的函数：

![Restricted book policy](docs/images/placing-on-hold-policy-restricted.png)
```java
PlacingOnHoldPolicy onlyResearcherPatronsCanHoldRestrictedBooksPolicy = (AvailableBook toHold, Patron patron, HoldDuration holdDuration) -> {
    if (toHold.isRestricted() && patron.isRegular()) {
        return left(Rejection.withReason("Regular patrons cannot hold restricted books"));
    }
    return right(new Allowance());
};
```

![Overdue checkouts policy](docs/images/placing-on-hold-policy-overdue.png)

```java
PlacingOnHoldPolicy overdueCheckoutsRejectionPolicy = (AvailableBook toHold, Patron patron, HoldDuration holdDuration) -> {
    if (patron.overdueCheckoutsAt(toHold.getLibraryBranch()) >= OverdueCheckouts.MAX_COUNT_OF_OVERDUE_RESOURCES) {
        return left(Rejection.withReason("cannot place on hold when there are overdue checkouts"));
    }
    return right(new Allowance());
};
```

![Max number of holds policy](docs/images/placing-on-hold-policy-max.png)

```java
PlacingOnHoldPolicy regularPatronMaximumNumberOfHoldsPolicy = (AvailableBook toHold, Patron patron, HoldDuration holdDuration) -> {
    if (patron.isRegular() && patron.numberOfHolds() >= PatronHolds.MAX_NUMBER_OF_HOLDS) {
        return left(Rejection.withReason("patron cannot hold more books"));
    }
    return right(new Allowance());
};
```

![Open ended hold policy](docs/images/placing-on-hold-policy-open-ended.png)

```java
PlacingOnHoldPolicy onlyResearcherPatronsCanPlaceOpenEndedHolds = (AvailableBook toHold, Patron patron, HoldDuration holdDuration) -> {
    if (patron.isRegular() && holdDuration.isOpenEnded()) {
        return left(Rejection.withReason("regular patron cannot place open ended holds"));
    }
    return right(new Allowance());
};
```

#### Spring
Spring Framework seems to be the most popular Java framework ever used.   
Spring框架似乎是最广泛使用的Java框架。  
Unfortunately it is also quite common to overuse its features in the business code.   
不幸的是，在业务代码中过度使用其功能也很普遍。  
What you find in this project is that the domain packages
are fully focused on modelling business problems, and are free from any DI,   
在这个项目中，您会发现领域包完全专注于建模业务问题，并且不包含任何DI，
which makes it easy to unit-test it which is invaluable in terms of code reliability and maintainability.   
这使得单元测试变得容易，这在代码可靠性和可维护性方面是无价的。  
It does not mean, though, that we do not use Spring Framework - we do.   
这并不意味着我们不使用Spring框架-我们使用。  
Below you will find some details:  
下面是一些细节:  

- Each bounded context has its own independent application context. It means that we removed the runtime
coupling, which is a step towards extracting modules (and microservices). How did we do that? Let's have
a look:  
  每个有界上下文都有自己独立的应用程序上下文。这意味着我们删除了运行时耦合，这是提取模块（和微服务）的一步。我们是如何做到的？让我们看一下：  
    ```java
    @SpringBootConfiguration
    @EnableAutoConfiguration
    public class LibraryApplication {
    
        public static void main(String[] args) {
            new SpringApplicationBuilder()
                    .parent(LibraryApplication.class)
                    .child(LendingConfig.class).web(WebApplicationType.SERVLET)
                    .sibling(CatalogueConfiguration.class).web(WebApplicationType.NONE)
                    .run(args);
        }
    }
    ```
- As you could see above, we also try not to use component scan wherever possible. Instead we utilize
`@Configuration` classes where we define module specific beans in the infrastructure layer. Those
configuration classes are explicitly declared in the main application class.  
  正如您在上面看到的，我们也尽可能不使用组件扫描。相反，我们在基础架构层中利用@Configuration类定义模块特定的bean。这些配置类在主应用程序类中明确声明。

### Tests
Tests are written in a BDD manner, expressing stories defined with Example Mapping.  
测试以BDD方式编写，表达用Example Mapping定义的故事。  
It means we utilize both TDD and Domain Language discovered with Event Storming.  
这意味着我们利用了TDD和通过Event Storming发现的域语言。  

We also made an effort to show how to create a DSL, that enables to write
tests as if they were sentences taken from the domain descriptions.    
我们还努力展示如何创建DSL，使其能够编写测试，就好像它们是从域描述中取出的句子一样。  
Please find an example below:  
请在下面找到一个例子：

```groovy
def 'should make book available when hold canceled'() {
    given:
        BookDSL bookOnHold = aCirculatingBook() with anyBookId() locatedIn anyBranch() placedOnHoldBy anyPatron()
    and:
        PatronEvent.BookHoldCanceled bookHoldCanceledEvent = the bookOnHold isCancelledBy anyPatron()

    when:
        AvailableBook availableBook = the bookOnHold reactsTo bookHoldCanceledEvent
    then:
        availableBook.bookId == bookOnHold.bookId
        availableBook.libraryBranch == bookOnHold.libraryBranchId
        availableBook.version == bookOnHold.version
}
``` 
_Please also note the **when** block, where we manifest the fact that books react to 
cancellation event   
请注意when块，其表明书籍会对取消事件做出反应。_

## How to contribute

The project is still under construction, so if you like it enough to collaborate, just let us
know or simply create a Pull Request.


## How to Build

### Requirements

* Java 11
* Maven

### Quickstart

You can run the library app by simply typing the following:

```console
$ mvn spring-boot:run
...
...
2019-04-03 15:55:39.162  INFO 18957 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 2 endpoint(s) beneath base path '/actuator'
2019-04-03 15:55:39.425  INFO 18957 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-04-03 15:55:39.428  INFO 18957 --- [           main] io.pillopl.library.LibraryApplication    : Started LibraryApplication in 5.999 seconds (JVM running for 23.018)

```

### Build a Jar package

You can build a jar with maven like so:

```console
$ mvn clean package
...
...
[INFO] Building jar: /home/pczarkowski/development/spring/library/target/library-0.0.1-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

### Build with Docker

If you've already built the jar file you can run:

```console
docker build -t spring/library .
```

Otherwise you can build the jar file using the multistage dockerfile:

```console
docker build -t spring/library -f Dockerfile.build .
```

Either way once built you can run it like so:

```console
$ docker run -ti --rm --name spring-library -p 8080:8080 spring/library
```

### Production ready metrics and visualization
To run the application as well as Prometheus and Grafana dashboard for visualizing metrics you can run all services:

```console
$ docker-compose up
```

If everything goes well, you can access the following services at given location:
* http://localhost:8080/actuator/prometheus - published Micrometer metrics
* http://localhost:9090 - Prometheus dashboard
* http://localhost:3000 - Grafana dashboard

In order to see some metrics, you must create a dashboard. Go to `Create` -> `Import` and select attached `jvm-micrometer_rev8.json`. File has been pulled from 
`https://grafana.com/grafana/dashboards/4701`.

Please note application will be run with `local` Spring profile to setup some initial data.

## References

1. [Introducing EventStorming](https://leanpub.com/introducing_eventstorming) by Alberto Brandolini
2. [Domain Modelling Made Functional](https://pragprog.com/book/swdddf/domain-modeling-made-functional) by Scott Wlaschin
3. [Software Architecture for Developers](https://softwarearchitecturefordevelopers.com) by Simon Brown
4. [Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164) by Robert C. Martin
5. [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) by Eric Evans
