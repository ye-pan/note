# 重构—改善既有代码的设计

## 记

又读过一遍了，但去尝试精读一些书籍，是从六月开始的，所以我决定重读一遍。只为将所有重构手法了然于胸，将重构做到如书中所描绘的， 就是日常 开发工作。为什么我要这么重视重构以及重构的基本手法，我见过了因为项目的烂代码而不得不重写的，也见到过 写过烂代码，但我希望以后的 我，就算是真的产生了烂代码，我也会去经常性的重构它，直到完善为止。书中所描绘的有讲究的，一次一小步的重构，以后的开发中我也需要做到。

## 原则

### 定义

重构的定义，根据上下文有两种不同的的定义：

名词：对软件内部结构的一种调整，目的是在不改变软件可观察行为的前提下，提高其可理解性，降低修改成本

动词：使用一系列重构手法，在不改变软件可观察行为的前提下，调整其结构

其实，重构就是改进代码，改进代码的结构，改进程序的设计。最重要的一点需要牢记在心—不改变软件的可观察行为，无论是功能还是接口，无论外部使用者是用户还是接口使用者，我们的重构都不能影响到他们对这块代码功能的使用。

### 为何重构

通过重构改进软件设计，开发进程中，因为时间紧急，或在完全理解一块代码之前就进行修改，可能会造成改变原来的设计结构，随着这种情况的累计，最终整块代码失去了它原本的设计，变得很难通过阅读代码来理解其设计，越难看出代码所代表的设计意图，就越难保护其中的设计，于是设计就腐败的越快。

重构使软件更容易理解，随着不断的重构，代码结构逐渐清晰，设计也随之浮现。同时，在不断的重构过程中，也会加深对程序系统的理解，甚至可能会产生新的设计！

### 何时重构

* 添加新功能
* 修改功能的bug
* 代码复审
* 三次法则

添加功能，或者修改bug的时候随之进行重构。这样，我修改的代码在随后的开发中也会用到，有问题的话就会即时发现。同时，在做完修改后，自己对修改的代码进行一次review，这也可能发现其中值得改进的地方。

基于成本来考量重构，重构不是取悦我们对美学的考究，它是实践的艺术。在需要它的地方使用它。

两顶帽子：使用重构技术开发软件时，你把自己的时间分配给两种截然不同的行为：添加新功能，以及重构。添加新功能时，你不应该修改既有代码，只管添加新功能。通过测试（并让测试正常运行），你可以衡量自己的工作进度。重构时你就不能再添加新功能，只管改进程序结构，此时，你不应该添加任何测试（除非发现先前遗漏的东西），只在绝对必要（用以处理接口变化）时，才修改测试。重构时，你总会发现某些代码并不正确，你绝对相信自己的判断，因此想马上把它们改正过来。别那么做！重构时你的目标之一就是保持代码的功能完全不表，既不多也不少，对于那些需要修改的东西，列个清
单把他们记录下来。重构完成之后再去做这些事情也不迟。

## 单元测试与重构

如果你想要进行重构，首要前提就是拥有一个可靠的测试环境。编写优良的测试程序，可以极大的提高我的编程速度。

* 确保所有测试都完全自动化，让他们检查自己的测试结果
* 一套测试就是一个强大的bug侦测器，能够大大的缩减查找bug的时间
* 需要添加特性的时候，先写相应的测试代码，编写测试还能使精力集中于该接口而非实现
* 频繁地运行测试，每次编译请把测试也考虑进去——每天至少执行每个测试一次
* 每当你收到bug报告，请先写一个单元测试来暴露这个bug

## 代码坏味道

### Duplicated Code 重复代码

在一个以上的地方看到相同的程序结构

重构手法：
* extract method，提炼重复代码到一个方法
* pull up method，如果重复代码再不同的子类，可以将提炼出来的方放推到超类中
* from template method，如果代码只是相似，并非完全相同，那要通过模板模式，提炼相同逻辑
* extract class，如果是完全不相关的类，则考虑新建一个类

### Long Method 过长的方法

重构手法：
* extract method，99%的场景都能起作用
* replace temp with query，用来消除函数内大量的参数和临时变量
* introduce parameter object & preserve whole object，将过长的参数列表变得简洁些
* replace method with method object，如果以上都不能有效，考虑将这个方法提炼为一个对象

### Large Class 过大的类

重构手法：
* extract class，用于处理一个类单纯的有大量的状态和逻辑
* extract subclass，用于处理一个类有大量的状态和逻辑，但这些状态并非所有时候都存在
* extract interface，如果有大量的代码，且提炼了不同的逻辑，还可以根据客户端使用的不同逻辑，进一步提炼不同的接口

### Long Parameter List 过长的参数列表

重构手法：
* replace parameter with method，用向该对象发消息，取代参数形式
* preserve whole object，将来自同一个对象的一堆数据收集起来，并以对象替代它们
* introduce parameter object，

### Divergent Change 发散式变化

*SRP*

我们希望软件能够更容易被修改——毕竟软件再怎么说本来就该是“软”的。一旦需要修改，我们希望能够跳到系统的某一点，只在该处做修改。如果不能做到这点，你就嗅出了两种紧密相关的刺鼻味道中的一种了。

如果某个类经常因为不同的原因在不同的方向上发生变化，*Divergent Change*就出现了。针对某一外界变化的所有相应修改，都只应该发生在单一类中，而这个新类内的所有内容都应该反应此变化。

重构手法：
* extract class，将因不同原因需要修改的部分，提炼到不同的类，这样每个类只会因为对应的原因而变化

### Shotgun Surgery 霰弹式修改

*ShotgunSurgery*类似*Divergent Change*，但恰恰相反，如果每遇到某种变化，你都必须在许多不同的类内做出许多小修改，你所面临的坏味道就是*Shotgun Surgery*。

*Divergent Change*是指“一个类受到了多种变化的影响”，*Shotgun Surgery*则是指“一种变化引发多个类相应修改”。这两种情况下你都会希望整理代码，使“外界变化”与“需要修改的类”趋于一一对应。

重构手法：
* move method & move field，多个类不同的状态或逻辑受到同一个原因而变化，考虑将这些类中分布的不同状态，或逻辑放到一起
* inline class，如果没有合适的类安置提炼的方法和字段，考虑创造一个

### Feature Envy 依恋情结

函数对某个类的兴趣高过对自己所处类的兴趣，这种孺慕之情最通常的焦点便是数据。

* extract method，将逻辑中这部分提炼出来
* move method，将提炼出来的方法放到合适的类中

### Data Clumps 数据泥团

数据项就像小孩子，喜欢成群结队的待在一起，你常常可以在很多地方看到相同的三四项数据：两个类中相同的字段，许多函数签名中相同的参数，这些总是绑在一起出现的数据真应该拥有属于它们自己的对象。

重构手法：
* extract class，将分不到不同类中的相同字段或逻辑，提炼到同一个类中
* introduce parameter object & preserve whole object，用到这些字段的函数签名做对应修改

### Primitive Obsession 基本类型偏执

重构手法：
* replace data value with object，提炼值对象
* replace type code with class，替换类型编码
* replace type code with subclass & replace type code with state/strategy，如果类型码涉及到条件表达式

### Switch Statements switch惊悚现身

重构手法：
* extract method 
* move method 
* replace type code with subclass & replace type code with state/strategy
* replace conditional with polymorphism
* replace parameter with explicit methods
* introduce null object 

### Parallel Inheritance Hierarchies 平行继承体系

*Parallel Inheritance Hierarchies*其实是*Shotgun Surgery*的特殊情况。在这种情况下，每当你为某个类增加一个子类，必须也为另一个类相应的增加一个子类，如果你发现某个继承体系的类名称前缀和另一个继承体系的类名称前缀完全相同，便是闻到了这种坏味道。

重构手法：
* move method & move field，将这种平行继承体系的关系消除 

### Lazy Class 冗赘类

重构手法：
* collapse hierarchy 消除继承体系
* inline class 消除不必要的类

### Speculative Generality 夸夸其谈未来性

重构手法：
* collapse hierarchy 消除不必要地继承体系
* inline class 消除不必要地类
* remove parameter 消除不必要地参数
* rename method 消除不必要的方法

### Temporary Field 令人迷惑的暂时字段

重构手法：
* extract class 将这些暂时性地字段提炼到一个类中，再使用这个类
* introduce null object 用 空对象取代 null地情况，避免条件表达式或者NPE

### Message Chains 过度耦合的消息链

重构手法：
* hide delegate，
* extract method，把使用消息链的方法提炼到一个方法中
* move method，将该方法推入消息链

### Middle Man 中间人

重构手法：
* remove middle man，消除中间人
* inline method，消除不必要的方法包装
* replace delegation with inheritance

### Inappropriate Intimacy 狎昵关系

重构手法：
* move method & move field，将两个类中关系过分紧密的字段或方法放在一起
* extract class，将两个类共同点提炼到新的类
* replace inheritance with delegation，如果子类过分依赖超类，可能说明可以不用继承而是用引用

### Alternative Classes with Different Interfaces 异曲同工的类

重构手法：
* rename method，
* move method，
* extract superclass，

### Incomplete Library Class 不完美的库类

重构手法：
* move method，
* introduce foreign method，
* introduce local extension，

### Data Class 纯稚的数据类

重构手法：
* encapsulate field，隐藏暴露的公共字段
* encapsulate collection，隐藏暴露的集合接口
* remove setting method，消除不必要的setter
* move method，尝试将调用点的部分涉及该类的逻辑移到该类中
* extract method，提炼调用点可以迁移的逻辑
* hide method，隐藏不必要的setter，getter

### Refused Bequest 被拒绝的遗赠

重构手法：
* push down method & push down field，将超类中的方法或字段放到实际使用它的子类中

### Comments 过多的注释

过多的注释意味着该函数有太多的意义，需要大段的注释来说明。

重构手法：
* extract method，提炼其中的逻辑
* rename method，取更有意义的名称
* introduce assertion，如果有些需求规格，可以考虑用断言来尝试说明

## 重构手法