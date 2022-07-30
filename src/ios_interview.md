# iOS-interview

[TOC]

## What was the latest version of iOS you worked with? What do you like about it and why?

This question is typically asked to check how in touch are you with latest iOS tech and developments in Swift and the iOS platform.

Expected answer:

The expectation is that you worked with at least the latest stable iOS version (iOS 10). But you get extra points if you"ve played with, or even
updated your apps to, the current iOS beta (iOS 11). Talk about exciting new features iOS 11 has and why you like them. It is always good to have
developers on your team who are passionate about the latest iOS technologies. Typically people who are curious enough to play with the new stuff come
up with interesting new feature ideas or think of creative solutions.

Red flag: It is usually a bad sign if a candidate hasn"t worked much with or isn"t interested in the current stable version of iOS. This means that
most likely this person is not up to date on the latest technologies, solutions, and features iOS provides, which most likely means this person would
either miss out on an opportunity to utilize the iOS system or, even worse, is not aware of some new pitfalls the latest system has.

## What is an iOS application and where does your code fit into it?

This is a big picture question that could be asked in one form or another to gauge your understanding of what an iOS application is and where the code
you write fits in it and in the iOS system overall.  Expected answer: We might think the apps we build are something special because they cover a
unique use case. But your typical iOS application is just a giant, glorified run loop. It waits for user input and gets interrupted by external
signals such as phone calls, push notifications, home button presses, and other app life cycle events. The only difference is that instead of being
just a simple mail loop function that gets launched every time the user taps on your app icon, it has a higher level of abstraction, UIApplication and
AppDelegate, that we developers work with.  The rest of the code you write to implement the business logic of your app is placed somewhere in the
"trigger points" delegated by that main loop to our app via AppDelegate. That"s pretty much it. Simple. The code you write for your app, though, can
be as simple as a method/function call or as complex as VIPER architecture. What you do with it is your choice.

Red flag: Typically developers think of iOS apps as the code they write and the complex intricate details of the implementation, which are all true.
But if you take a step back and look at the big picture, you can see what iOS apps really are - a run loop.

## What features of Swift do you like or dislike? Why?

With the latest 3.2/4 Swift update it is proving again and again that this language is the future of iOS development. These days the expectation,
especially from seasoned developers, is that you are well versed in Swift and the features it offers.  Expected answer: You could talk about strong
typing and functional features of the language and why you like or dislike them. There"s no right or wrong answer here per se, but the expectation is
that you are familiar with the language and the features it provides (generics, protocols, optionals, etc.). Also you should be able to explain and
argue if you like or dislike something about the language (I"m not a fan of optional chaining because it breaks the Law of Demeter, for example).

Red flag: Swift is becoming the major stable language for the iOS platform, so ignoring it these days doesn"t make sense.

## How is memory management handled in iOS?

Memory management is very important in any application, especially in iOS apps that have memory and other hardware and system constraints. Hence, this
is one of the questions that is asked in one form or another. It refers to ARC, MRC, reference types, and value types.  Expected answer: Swift uses
Automatic Reference Counting (ARC). This is conceptually the same thing in Swift as it is in Objective-C. ARC keeps track of strong references to
instances of classes and increases or decreases their reference count accordingly when you assign or unassign instances of classes (reference types)
to constants, properties, and variables. It deallocates memory used by objects whose reference count dropped to zero. ARC does not increase or
decrease the reference count of value types because, when assigned, these are copied. By default, if you don"t specify otherwise, all the references
will be strong references.

Red flag: This is a must know for every iOS developer! Memory leaks and app crashes are all too common due to poorly managed memory in iOS apps.

## What do you know about singletons? Where would you use one and where would you not?

Singleton is a common design pattern used in many OOP languages, and Cocoa considers it one of the "Cocoa Core Competencies." This question comes up
from time to time on interviews to either gauge your experience with singletons or to find out if you have a background in something other than just
iOS.  Expected answer: Singleton is a class that returns only one and the same instance no matter how many times you request it.  Singletons are
sometimes considered to be an anti-pattern. There are multiple disadvantages to using singletons. The two main ones are global state/statefulness and
object life cycle and dependency injection. When you have only one instance of something, it is very tempting to reference and use it everywhere
directly instead of injecting it into your objects. That leads to unnecessary coupling of concrete implementation in your code instead of interface
abstraction. Another malicious side effect of "convenient" singletons is global state. Quite often singletons enable global state sharing and play the
role of a "public bag" that every object uses to store some state. That leads to unpredictable results and bugs and crashes when this uncontrolled
state gets overridden or removed by someone.

Red flag: Even though in some languages/platforms singletons are considered to be good, they are in fact an anti-pattern that should be avoided at all
costs.

## Could you explain what the difference is between Delegate and KVO?

With this question your interviewer is assessing your knowledge of different types of messaging patterns used in iOS.  Expected answer: Both are ways
to have relationships between objects. Delegation is a one-to-one relationship where one object implements a delegate protocol and another uses it and
sends messages to it, assuming that those methods are implemented since the receiver promised to comply to the protocol. KVO is a many-to-many
relationship where one object could broadcast a message and one or multiple other objects can listen to it and react. KVO does not rely on protocols.
KVO is the first step and the fundamental block of reactive programming (RxSwift, ReactiveCocoa, etc.)

Red flag: A seasoned developer should know what the difference is between the two and where one should be used over another.

## What design patterns are commonly used in iOS apps?

This question is a common one on interviews for positions of all levels, maybe with the exception of junior positions. Essentially the idea is that in
working with the iOS platform, you as a developer should be familiar with commonly used techniques, architecture, and design patterns used on iOS.
Expected answer: Typical commonly used patterns when building iOS applications are those that Apple advocates for in their Cocoa, Cocoa Touch,
Objective-C, and Swift documentation. These are the patterns that every iOS developer learns. They include MVC, Singleton, Delegate, and Observer.

Red flag: When an interviewer asks this question (in one form or another) the interviewer is looking for something besides MVC. Because MVC is the
go-to design pattern, the expectation is that every iOS developer knows what it is. What they want to hear from you, though, is what else is commonly
used and available out of the box.

## What are the design patterns besides common Cocoa patterns that you know of?

This is an advanced question that an interviewer will ask when you interview for a senior or architect position. The expectation is that you know more
practical design patterns used in iOS apps besides the basic ones covered in the previous question. Be ready to recall a bunch of Gang of Four
patterns and other similar patterns.  Unfortunately design patterns are a huge topic on their own (they are also better covered in my book), so here
I'll give only an overview of some of them that I see occur commonly in iOS codebases.  Expected answer: Besides commonly used MVC, Singleton,
Delegate, and Observer patterns, there are many others that are perfectly applicable in iOS applications: Factory Method, Adapter, Decorator, Command,
Template, and many other.  Factory Method is used to replace class constructors, to abstract and hide objects initialization so that the type can be
determined at runtime, and to hide and contain switch/if statements that determine the type of object to be instantiated.  Adapter is a design pattern
that helps you, as the name suggests, adapt the interface of one object to the interface of another. This pattern is often used when you try to adapt
third-party code that you can"t change to your code, or when you need to use something that has an inconvenient or incompatible API.  Decorator is a
wrapper around another class that enhances its capabilities. It wraps around something that you want to decorate, implements its interface, and
delegates messages sent to it to the underlying object or enhances them or provides its own implementation.  Command is a design pattern where you"d
implement an object that represents an operation that you would like to execute. That operation can have its own state and logic to perform the task
it does. The main advantages of this design pattern are that you can hide internal implementation of the operation from the users, you can add
undo/redo capabilities to it, and you can execute operations at a later point in time (or not at all) instead of right away where the operation was
created.  Template is a design pattern where the main concept is to have a base class that outlines the algorithm of what needs to be done. The base
class has several abstract methods that are required to be implemented by its concrete subclasses. These methods are called hook methods. Users of the
Template Method classes only interact using the base class that implements the algorithm steps; concrete implementations of those steps are supplied
by subclasses.

Red flag: Sticking only to MVC, Singleton, Delegate, and Observer patterns is fine when you"re just starting with the iOS platform, but for advanced
things you need to reach deeper into more abstract and high-level stuff like Gang of Four OOP Design Patterns. They are very useful and make your
codebase more flexible and maintainable.

## Could you explain and show examples of SOLID principles?

SOLID principles are relatively old but incredibly useful concepts to apply to any OOP codebase in any language. Watch a few of Uncle Bob"s talks on
the topic to fully appreciate the history behind them.  On YouTube: Bob Martin SOLID Principles of Object Oriented and Agile Design.  Unfortunately
SOLID principles are a huge topic on their own (they are also better covered in my book), so here I'll give only an overview of them.  Expected
answer: SOLID stands for Single Responsibility Principle, Open/Closed Principle, Liskov Substitution Principle, Interface Segregation Principle, and
Dependency Inversion Principle. These principles feed into and support each other and are one of the best general design approaches you could take for
your code. Let"s go through each of them.  The Single Responsibility Principle (SRP) is the most important principle of the group. It states that
every module should have only one responsibility and reason to change. SRP starts with small concrete and specific cases such as a class and/or an
object having only one purpose and being used only for only one thing.  The Open/Closed Principle (OCP) states that your modules should be open for
extension but closed for modification. It"s one of those things that sounds easy enough but is kind of hard to wrap your head around when you start to
think about what it means. Effectively it means that when writing your code you should be able to extend the behavior of your objects through
inheritance, polymorphism, and composition by implementing them using interfaces, abstractions, and dependency injection.  The Liskov Substitution
Principle (LSP) states that objects in a program should be replaceable with instances of their subtypes without altering the correctness of that
program. What that means is that when you inherit from a class or an abstract class or implement an interface (protocol), your objects should be
replaceable and injectable wherever that interface or class that you subclassed from was used. This principle is often referred to as design by
contract or, as of late in the Swift community, referred to as protocol-oriented programming. The main message of this principle is that you should
not violate the contract that your interfaces you subclass from promise to fulfill, and that by subclassing, those subclasses could be used anywhere
that the superclass was previously used.  The Interface Segregation Principle (ISP) says many client-specific interfaces are better than one
general-purpose interface. It also states that no client should be forced to depend on and implemented methods it does not use. What that means is
that when you create interfaces (protocols) that your classes implement, you should strive for and depend on abstraction over specificity, but not
until it becomes a waste where you have to implement a bunch of methods your new class doesn"t even use.  The Dependency Inversion Principle (DIP)
states, "depend on abstractions, not concretions." The best example that showcases this principle is the Dependency Injection (DI) technique. With the
Dependency Injection technique, when you create an object, you supply and inject all of its dependencies upon its initialization or configuration
rather than let the object create or fetch/find its dependencies for itself.

> SOLID principles are the bedrock of good OOP design. Applying these principles will help you build better, more maintainable software. It is highly
> advised to be well versed in them if you are applying to a senior iOS position.

## What options do you have for implementing storage and persistence on iOS?

Interviewers ask this question to grasp your understanding of what tools and ways you have available to store and persist data on iOS.  Expected
answer: Generally there are the following ways to store data in order from simple to complex: In-memory arrays, dictionaries, sets, and other data
structures NSUserDefaults/Keychain File/Disk storage Core Data, Realm SQLite In-memory arrays, dictionaries, sets, and other data structures are
perfectly fine for storing data intermediately or if it doesn't have to be persisted.  NSUserDefaults/Keychain are simple key-value stores. One is
insecure and the other is secure, respectively.  File/Disk storage is actually a way of writing pieces of data (serialized or not) to/from a disk
using NSFileManager.  Core Data and Realm are frameworks that simplify work with databases.  SQLite is a relational database and is good when you need
to implement complex querying mechanics and Core Data or Realm won't cut it.

Red flag: You should be aware of different ways you could store data on iOS and their advantages or disadvantages. Don"t limit yourself to only one
solution that you"re used to (like Core Data, for example). Know when one is preferable over the other.

## What options do you have for implementing networking and HTTP on iOS?

Virtually every application these days is using some kind of networking to get data from APIs and other external resources. A lot of the apps are
useless when they are not connected to the internet. Every iOS developer should know what's available to them to build the service/networking layer of
their application.  Expected answer: In iOS there are several options to implement HTTP networking. You can go with good old NSURLSession, but unless
you abstract it out well enough, it can be daunting to work with. Another option would be to use a wrapper library around it. The most popular
solution on iOS is Alamofire/AFNetworking.  Senior developers should keep in mind that building the networking layer in iOS applications does not only
mean dealing with HTTP requests. It also means implementing the whole set of tasks your code does related to that: HTTP networking, data
serialization, and data mapping.

Red flag: These days, AFNetworking and Alamofire are the de facto standard for doing HTTP networking on iOS, and every developer should know how to
use them. At the same time, they are all based on Apple frameworks, and knowing the inner details of NSURLSession is also beneficial.

## How and when would you need to serialize and map data on iOS?

Data serialization is a common task that you need to perform when building iOS applications. Interviewers ask this question to see if you recognize
the situations where it is suitable and are aware of the tasks you need to perform working with data, whether it is networking or storage data.
Expected answer: There are two most common scenarios where you'd need to serialize and map data in iOS applications: receiving or sending data in the
networking layer (such as JSON or XML or something else), and persisting or retrieving models in the storage layer (NSData, NSManagedObject, etc.).
Every time you receive JSON or XML or any other kind of response from a backend API, you most likely get it in a JSON or binary or other
"inconvenient" format. The first thing you need to do to be able to work with the data you"ve received is to serialize it in something your app
understands. At the most simplest and basic level that would be a dictionary or array of objects containing other dictionaries, arrays, and primitives
from that response. NSJSONSerializationtakes care of that (and soon Codable protocol). The next step is to map that data into domain models of your
application. Those would be the model objects the rest of your application works with. You can either do it manually or use a library such as Mantle
or SwiftyJSON. The flow of data and serialization/mapping is as follows: binary data -> json-> NSDictionary/NSArray -> your domain model objects.
Similarly in the storage layer, you will need to serialize and map your data to and from your custom domain model objects to the format your storage
understands. The "mapping" chain for reading data looks like this: db -> raw data format -> custom domain models, and for writing like this: custom
domain models -> raw data format -> db. You'd use the NSManagedObject or NSCoding protocol here to achieve that.

Red flag: The main red flag here is not being aware that these data manipulations need to happen when working with the networking and storage layers
of your iOS applications. Things do not happen "automagically" nor is working with raw NSDictionaries appropriate and maintainable.

## What are the options for laying out UI on iOS?

Knowing your options for laying out things on the screen is crucial when you need to solve different UI challenges on iOS. This question helps gauge
your knowledge about how you put and align views on the screen. When answering this question you should at least mention CGRect Frames and AutoLayout,
but it would be great to mention other options such as ComponentKit and other Flexbox and React implementations on iOS.  Expected answer: Go-to
options for laying out views on the screen are good old CGRect Frames and AutoLayout. Frames, along with auto-resizing masks, were used in the past
before iOS 6 and are not a preferred option today. Frames are too error-prone and difficult to use because it"s hard to calculate precise coordinates
and view sizes for various devices.  Since iOS 6 we have AutoLayout, which is the go-to solution these days and which Apple prefers. AutoLayout is a
technology that helps you define relationships between views, called constraints, in a declarative way, letting the framework calculate precise frames
and positions of UI elements instead.  There are other options for laying out views, such as ASDK (Texture), ComponentKit, and LayoutKit, that are
more or less inspired by React. These alternatives are good in certain scenarios when, for example, you need to build highly dynamic and fast table
views and collection views. AutoLayout is not always perfect for that and knowing there are other options is always good.

Red flag: Not mentioning at least AutoLayout and the fact that Frames are notoriously hard to get right is definitely going to be a red flag for your
interviewer. These days no sane person would do CGRect frame calculations unless it is absolutely necessary (for example, when you do some crazy
drawings).

Knowing your options for laying out things on the screen is crucial when your need to solve different UI challenges on iOS. This question helps guage
your knowledge about how you put and align views on the screen. When answering this question you should at least mention CGRect Frames and Autolayout,

## How would you optimize scrolling performance of dynamically sized table or collection views?

One of the important questions that is sometimes asked on interviews along with UITableView questions is a question about table view scrolling
performance.  Expected answer: Scrolling performance is a big issue with UITableViews and quite often can be very hard to get right. The main
difficulty is cell height calculation. When the user scrolls, every next cell needs to calculate its content and then height before it can be
displayed. If you do manual Frame view layouts then it is more performant but the challenge is to get the height and size calculations just right. If
you use AutoLayout then the challenge is to set all the constraints right. But even AutoLayout itself could take some time to compute cell heights,
and your scrolling performance will suffer.  Potential solutions for scrolling performance issues could be calculate cell height yourself keep a
prototype cell that you fill with content and use to calculate cell height Alternatively, you could take a completely radical approach, which is to
use different technology like ASDK (Texture). ASDK (Texture) is made specifically for list views with dynamic content size and is optimized to
calculate cell heights in a background thread, which makes it super performant.

## How would you execute asynchronous tasks on iOS?

Multithreading is a vital part of any client-side, user-facing application these days. This question could be asked in the context of networking or as
a standalone question about GCD or async development.  Expected answer: These days on iOS your go-to solutions for async tasks are NSOperations and
GCD blocks. Grand Central Dispatch is a technology that was made to work with multiple background queues that in turn figure out which background
thread handles the work. The main thing is that this is abstracted out from you so that you don't have to worry about it. NSOperation is an OOP
abstraction on top of GCD that allows you to do more sophisticated async operations, but everything you could achieve with NSOperations you could do
with GCD. Many Cocoa frameworks use GCD and/or NSOperations under the hood (NSURLSession for example).  There are alternative ways of handling async
work using third-party libraries" help. The most notable are Promises (PromiseKit), RxSwift, and ReactiveCocoa. RxSwift and ReactiveCocoa are
especially good at modeling the asynchronous nature of time and work that needs to be done in the background and coordinated among threads.

Red flag: The basics that every iOS developer should know in regards of async work are GCD and NSOperations. RxSwift and Promises are advanced
concepts but senior developers should be aware of them as well.

## How do you manage dependencies?

Dependencies management is an important task on every iOS project. This question is asked to gauge your understanding of the problem and how it can be
solved.  Expected answer: A few years back we didn't have any dependency managers on iOS and had to copy-paste and drag and drop third-party code into
our projects or to use git submodules. All of those approaches quickly proved to be unmanageable as our codebase and dependencies grew.  These days we
have other dependency managers to choose from: CocoaPods, Carthage, and Swift Package Manager (SPM). So far the most dominant and robust one is
CocoaPods. It was built in the spirit of the Ruby Bundler gem and is a Ruby gem itself. The way it works is you install the gem, create Podfile in the
root directory of your project, declare the pods (libraries) you want to use, and run pod install. That's it.

Red flag: Every iOS developer should understand why copy-pasting third-party libraries into your codebase would lead to a maintenance nightmare when
several libraries could depend on two different versions of another library, causing mismatches and compile and runtime issues, and so on.

## How do you debug and profile things on iOS?

No one writes perfect code and occasionally developers need to debug their code and profile apps for things like performance and memory leaks.
Expected answer: There's always the good old NSLogging and printing we can do in iOS apps. Also there are breakpoints you can set using Xcode. For
performance of individual pieces of code you could use XCTest's measureBlock.  You can do more advanced debugging and profiling using Instruments.
Instruments is a profiling tool that helps you profile your app and find memory leaks and performance issues at runtime.

## Do you have TDD experience? How do you unit and UI test on iOS?

Even though, historically, the iOS community wasn't big on TDD, it is now becoming more popular thanks to improvements in tooling and influence from
other communities, such as Ruby, that embraced TDD a long time ago.  Expected answer: TDD is a technique and a discipline where you write failing
tests first before you write production code that makes them pass. The tests drive implementation and design of your production code, helping you
write only the code necessary to pass the tests' implementation, no more, no less. The discipline could be daunting at first and you don't see payoff
of that approach immediately, but if you stick to it, it helps you move faster in the long run. It is especially effective at helping you with
refactoring and code changes because at any given time you have the safety net of your tests to tell you if something broke or if everything is still
working fine as you change things.  Recently Apple made improvements to XCTest frameworks to make testing easier for us. They also made a lot of
improvements with UI testing in Xcode, so now we have a nice programmatic interface to interact with our apps and query things we see on the screen.
Alternatively you could go with frameworks like KIF.  In regards to unit testing, there are several options as well, but the two most popular ones are
XCTest and Quick and Nimble.  XCTest is an xUnit like testing framework built by Apple. This is what they recommend to use, and it has the best
integration with Xcode.  Quick is an RSpec-like BDD framework that helps you describe your specs/tests in terms of behavior rather than "tests." Fans
of RSpec like it a lot.  Nimble is a matcher library that can be used with XCTest or Quick to assert expectations in your tests/specs.

Red flag: More and more teams and companies embrace TDD, and it has become a vital part of the iOS development process. If you don't want to be left
behind, get on board with it and learn how to test-drive your code.

Red flag: More and more teams and companies embrace TDD, and it has become a vital part of the iOS development process. I you don't want to be left
behind, get on board with it and learn how to test-drive your code

## Do you code review and/or pair program?

Even though there are a lot of applications out there that were built by solo developers, the complexity of what apps do keeps increasing, demanding
that a team of developers work on it. Working in a team poses different challenges in terms of code maintenance, collaboration, and knowledge sharing.
Expected answer: Pair programming is a practice where two developers work on the same task together on the same machine (hopefully not sharing the
same screen and keyboard and having two sets of their own). The goal is to facilitate collaboration, discussion, code review, and QA right where the
code gets produced. This process makes knowledge transfer and architectural discussions a common day-to-day thing, preventing people from siloing and
becoming "an expert" in a certain part of the code (what happens when that person leaves or gets sick?). It also improves code quality because two
sets of eyes are looking at the code as it's written. This process happens for two developers at the same time and is sometimes called synchronous.
Pair programming is not for everyone and could be an exhausting process if people's personalities do not match. But nevertheless it is one of the most
efficient collaboration techniques in software development.  Code review is a similar process of collaboration and knowledge transfer, but unlike pair
programming, it doesn't happen at the same time, and therefore it is asynchronous. With code review, after a developer writes a piece of code or a
feature, someone else on the team has a look at it. The reviewer checks if the code makes sense and suggests changes and refactorings to be done to
improve it. That opens up an online or offline discussion about the code, which is great. That transfers knowledge about that piece of code to other
teammates and helps catch bugs and design smells early on.  Code reviews are a less-involved type of collaboration that achieves much of the same
results as pair programming does. It also is an exercise in empathy where you're giving feedback to others on their work.

1. Difference between shallow copy and deep copy?

答案: 浅层复制: 只复制指向对象的指针, 而不复制引用对象本身.  深层复制: 复制引用对象本身.  意思就是说我有个 A 对象, 复制一份后得到 A_copy 对象后,
对于浅复制来说, A 和 A_copy 指向的是同一个内存资源, 复制的只不过是是一个指针, 对象本身资源 还是只有一份, 那如果我们对 A_copy 执行了修改操作, 那么发现
A 引用的对象同样被修改, 这其实违背了我们复制拷贝的一个思想. 深复制就好理解了, 内存中存在了 两份独立对象本身.  用网上一哥们通俗的话将就是:
 浅复制好比你和你的影子, 你完蛋, 你的影子也完蛋 深复制好比你和你的克隆人, 你完蛋, 你的克隆人还活着.

3. Difference between categories and extensions?

答案: category 和 extensions 的不同在于 后者可以添加属性. 另外后者添加的方法是必须要实现的.  extensions 可以认为是一个私有的 Category.

5. What are KVO and KVC?

答案: kvc: 键 - 值编码是一种间接访问对象的属性使用字符串来标识属性, 而不是通过调用存取方法, 直接或通过实例变量访问的机制.
 很多情况下可以简化程序代码. apple 文档其实给了一个很好的例子.  kvo: 键值观察机制, 他提供了观察某一属性变化的方法, 极大的简化了代码.
 具体用看到嗯哼用到过的一个地方是对于按钮点击变化状态的的监控.  比如我自定义的一个 button [cpp]  [self addObserver:self forKeyPath:@"highlighted"
options:0 context:nil];      #pragma mark KVO    - (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change
context:(void *)context  {   if ([keyPath isEqualToString:@"highlighted"]) {   [self setNeedsDisplay];   }  } 对于系统是根据 keypath
去取的到相应的值发生改变, 理论上来说是和 kvc 机制的道理是一样的.  对于 kvc 机制如何通过 key 寻找到 value:  "当通过 KVC 调用对象时, 比如: [self
valueForKey:@"someKey"] 时, 程序会自动试图通过几种不同的方式解析这个调用. 首先查找对象是否带有 someKey 这个方法, 如果没找到, 会继续查找对象是否带有
someKey 这个实例变量 (iVar), 如果还没有找到, 程序会继续试图调用 -(id) valueForUndefinedKey: 这个方法. 如果这个方法还是没有被实现的话, 程序会抛出一个
NSUndefinedKeyException 异常错误.    (cocoachina.com 注: Key-Value Coding 查找方法的时候, 不仅仅会查找 someKey 这个方法, 还会查找 getsomeKey 这个方法,
前面加一个 get, 或者_someKey 以及_getsomeKey 这几种形式. 同时, 查找实例变量的时候也会不仅仅查找 someKey 这个变量, 也会查找_someKey 这个变量是否存在.) 
 设计 valueForUndefinedKey: 方法的主要目的是当你使用 -(id)valueForKey 方法从对象中请求值时, 对象能够在错误发生前, 有最后的机会响应这个请求.
这样做有很多好处, 下面的两个例子说明了这样做的好处. " 来至 cocoa, 这个说法应该挺有道理.   因为我们知道 button 却是存在一个 highlighted 实例变量.
因此为何上面我们只是 add 一个相关的 keypath 就行了,  可以按照 kvc 查找的逻辑理解, 就说的过去了.

6. What is purpose of delegates?

答案: 代理的目的是改变或传递控制链. 允许一个类在某些特定时刻通知到其他类, 而不需要获取到那些类的指针.  可以减少框架复杂度.  另外一点, 代理可以理解为
java 中的回调监听机制的一种类似.

7. What are mutable and immutable types in Objective C?

答案: 可修改不可修改的集合类. 这个我个人简单理解就是可动态添加修改和不可动态添加修改一样.  比如 NSArray 和 NSMutableArray.
前者在初始化后的内存控件就是固定不可变的, 后者可以添加等, 可以动态申请新的内存空间.

8. When we call objective c is runtime language what does it mean?

主要是将数据类型的确定由编译时, 推迟到了运行时.  这个问题其实浅涉及到两个概念, 运行时和多态.  简单来说,
运行时机制使我们直到运行时才去决定一个对象的类别, 以及调用该类别对象指定方法.  多态: 不同对象以自己的方式响应相同的消息的能力叫做多态.
意思就是假设生物类 (life) 都用有一个相同的方法 -eat; 那人类属于生物, 猪也属于生物, 都继承了 life 后, 实现各自的 eat, 但是调用是我们只需调用各自的 eat
方法.  也就是不同的对象以自己的方式响应了相同的消息 (响应了 eat 这个选择器).  因此也可以说, 运行时机制是多态的基础? ~~~

9. what is difference between NSNotification and protocol?

答案: 协议有控制链 (has-a) 的关系, 通知没有.   首先我一开始也不太明白, 什么叫控制链 (专业术语了~). 但是简单分析下通知和代理的行为模式,
我们大致可以有自己的理解 简单来说, 通知的话, 它可以一对多, 一条消息可以发送给多个消息接受者.  代理按我们的理解, 到不是直接说不能一对多,
比如我们知道的明星经济代理人, 很多时候一个经济人负责好几个明星的事务.   只是对于不同明星间, 代理的事物对象都是不一样的, 一一对应, 不可能说明天要处理 A
明星要一个发布会, 代理人发出处理发布会的消息后, 别称 B 的 发布会了.  但是通知就不一样, 他只关心发出通知, 而不关心多少接收到感兴趣要处理.  因此控制链
(has-a 从英语单词大致可以看出, 单一拥有和可控制的对应关系.

10. What is push notification?
11. Polymorphism?

答案: 多态, 子类指针可以赋值给父类.  这个题目其实可以出到一切面向对象语言中,  因此关于多态, 继承和封装基本最好都有个自我意识的理解,
也并非一定要把书上资料上写的能背出来.  最重要的是转化成自我理解.

12. Singleton?

答案: 11, 12 题目其实出的有点泛泛的感觉了, 可能说是编程语言需要或是必备的基础.  基本能用熟悉的语言写出一个单例,
以及可以运用到的场景或是你编程中碰到过运用的此种模式的框架类等.  进一步点, 考虑下如何在多线程访问单例时的安全性.

13. What is responder chain?  答案:  事件响应链. 包括点击事件, 画面刷新事件等. 在视图栈内从上至下, 或者从下之上传播.  可以说点事件的分发,
    传递以及处理. 具体可以去看下 touch 事件这块. 因为问的太抽象化了 严重怀疑题目出到越后面就越笼统.

14. Difference between frame and bounds?

答案:frame 指的是: 该 view 在父 view 坐标系统中的位置和大小. (参照点是父亲的坐标系统) bounds 指的是: 该 view 在本身坐标系统中 的位置和大小.
(参照点是本身坐标系统)

15. Difference between method and selector?

答案: selector 是一个方法的名字, method 是一个组合体, 包含了名字和实现. 详情可以看 apple 文档.

16. Is there any garbage collection mechanism in Objective C.?

答案:  OBC2.0 有 Garbage collection, 但是 iOS 平台不提供.  一般我们了解的 objective-c 对于内存管理都是手动操作的, 但是也有自动释放池.
 但是差了大部分资料, 貌似不要和 arc 机制搞混就好了.  求更多~~

17. NSOperation queue?

答案: 存放 NSOperation 的集合类.  操作和操作队列, 基本可以看成 java 中的线程和线程池的概念. 用于处理 ios 多线程开发的问题.  网上部分资料提到一点是,
虽然是 queue, 但是却并不是带有队列的概念, 放入的操作并非是按照严格的先进现出.  这边又有个疑点是, 对于队列来说, 先进先出的概念是 Afunc 添加进队列,
Bfunc 紧跟着也进入队列, Afunc 先执行这个是必然的,  但是 Bfunc 是等 Afunc 完全操作完以后, B 才开始启动并且执行,
因此队列的概念离乱上有点违背了多线程处理这个概念.  但是转念一想其实可以参考银行的取票和叫号系统.  因此对于 A 比 B 先排队取票但是 B 率先执行完操作,
我们亦然可以感性认为这还是一个队列.  但是后来看到一票关于这操作队列话题的文章, 其中有一句提到 "因为两个操作提交的时间间隔很近, 线程池中的线程,
谁先启动是不定的. " 瞬间觉得这个 queue 名字有点忽悠人了, 还不如 pool~ 综合一点, 我们知道他可以比较大的用处在于可以帮组多线程编程就好了.

18. What is lazy loading?

答案: 懒汉模式, 只在用到的时候才去初始化.  也可以理解成延时加载.  我觉得最好也最简单的一个列子就是 tableView 中图片的加载显示了.  一个延时载,
避免内存过高, 一个异步加载, 避免线程堵塞.

19. Can we use two tableview controllers on one viewcontroller?

答案: 一个视图控制只提供了一个 View 视图, 理论上一个 tableViewController 也不能放吧,  只能说可以嵌入一个 tableview 视图. 当然, 题目本身也有歧义,
如果不是我们定性思维认为的 UIViewController,  而是宏观的表示视图控制者, 那我们倒是可以把其看成一个视图控制者, 它可以控制多个视图控制器, 比如
TabbarController 那样的感觉.

20. Can we use one tableview with two different datasources? How you will achieve this?

答案: 首先我们从代码来看, 数据源如何关联上的, 其实是在数据源关联的代理方法里实现的.  因此我们并不关心如何去关联他, 他怎么关联上,
方法只是让我返回根据自己的需要去设置如相关的数据源.  因此, 我觉得可以设置多个数据源啊, 但是有个问题是, 你这是想干嘛呢? 想让列表如何显示,
不同的数据源分区块显示?

21. What is advantage of using RESTful webservices?
22. When to use NSMutableArray and when to use NSArray?
23. What is the difference between REST and SOAP?
24. Give us example of what are delegate methods and what are data source methods of uitableview.
25. How many autorelease you can create in your application? Is there any limit?
26. If we don't create any autorelease pool in our application then is there any autorelease pool already provided to us?
27. When you will create an autorelease pool in your application?
28. When retain count increase?
29. Difference between copy and assign in objective c?
30. What are commonly used NSObject class methods?
31. What is convenience constructor?
32. How to design universal application in Xcode?
33. What is keyword atomic in Objective C?
21. What are UIView animations
22. How can you store data in iPhone application

84. 什么是 Notification? 什么时候用 delegate, 什么时候用 Notification?

观察者模式, controller 向 defaultNotificationCenter 添加自己的 notification, 其他类注册这个 notification 就可以收到通知,
这些类可以在收到通知时做自己的操作 (多观察者默认随机顺序发通知给观察者们, 而且每个观察者都要等当前的某个观察者的操作做完才能轮到他来操作, 可以用
NotificationQueue 的方式安排观察者的反应顺序, 也可以在添加观察者中设定反映时间, 取消观察需要在 viewDidUnload 跟 dealloc 中都要注销).  delegate 针对
one-to-one 关系, 并且 reciever 可以返回值给 sender, notification 可以针对 one-to-one/many/none,reciever 无法返回值给 sender. 所以, delegate 用于
sender 希望接受到 reciever 的某个功能反馈值, notification 用于通知多个 object 某个事件.
