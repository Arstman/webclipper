# Client <-> Runtime
Client <-> Runtime
==================

原创 George [该帐号已冻结](javascript:void(0);)

**该帐号已冻结** 

微信号 gh_955f33a1fa04

功能介绍

_2022-06-03 02:35_ _发表于四川_

收录于合集

对于每一个接触Substrate的新手来说，尽管是有多年编码经验的老司机，都会感觉到框架的庞大和复杂，总感觉抓不到这个框架设计的灵魂（主要是感觉自己不能和Web2一样灵活地CURD）， 所以本文就通过区块链原理开始，一步一步帮各位新老司机扫平障碍。

  

  

为了搞清楚Substrate做了怎么样的抽象导致各位司机都觉得这个框架庞大且上手难度极高，第一件事儿就是要搞清楚区块链到底是个什么样的系统。对于区块链是什么，相信基本大家都有个概念，就是一个账本，想记账请给钱，不过不同的区块链拥有了不同的记账逻辑，比如Ethereum就是可以通过智能合约描述记账逻辑，Substrate则使用Pallet这种抽象来完成记账逻辑的定义。

  

**那么账本为什么是安全的？**这个问题看着还挺哲学的，不过我们把我们的记忆推回到上小学第一次选班长，按照我们现在的视角来看，这个班长并没有证书，也没有特殊的标志（难道是神奇的二道杠? ），但是我们还是每次有事的时候就会找到这个同学，插上一句：你还记得你的小学班长吗？，在我们最原始的记忆中，这个事情好像是我们和班主任一起达成了一个神奇的共识，当我们所有人都认为一个同学是班长的时候（达成共识），我们就认为这个同学是班长应该担当班长的责任。通过这个场景，我们可以得出一个结论：**当所有同学都认为一个同学是班长的时候，不会有另外一个不是班长的同学能履行班长的责任，稍微学术一点，这个同学当选班长就是安全的（safety）**。同样的，账本安全的定义则是：当没有人或者需要付出非常大代价修改账本的人可以修改账本的时候，那么这个账本就是安全的。更进一步，我们会发现我们让班长当选这件事而安全的选举以及共识的思路同样可以让账本安全，WoW，我们好像推导出了什么很了不起的结论，是的，我们发现了区块链需要共识算法来保证账本安全。

  

**那么账本是怎么工作的？** 感觉使用现金也是很久之前的事情了，不妨我们回忆一下我们使用现金的过程，当我们钱包空的时候，我们会去ATM取现金，然后钱包就鼓了，当我们有Money的时候，我们花掉一部分钱，钱包就剩下了一部分现金，那么循环往复，我们花掉的现金和我们钱包内的现金之和总是一样多的。并且我们记账的时候，总是喜欢习惯性地记住我们花掉一笔钱之后的剩余现金。那么就好像计算机内的状态模式一样，我们总是在记录从上一个状态到下一个状态之间的改变量。其实区块链也是一样的，只不过这个账本负责追踪所有账户的Token变动状况而已。

  

**那么什么会引发账本的变动**？很显然花掉现金和取出现金会让我们的钱包状态变化，那么在区块链里面，最简单的就是转账和收入会让区块链的地址内部的Token变化。更进一步如果我们让一笔转账变得复杂起来，我们就好像在钱包内拥有了各种各样的货币一样，智能合约也就诞生了（ink合约的实现依旧是一个系列文章，毕竟在WASM内跑WASM还是有点复杂的-_-）。  

  

* * *

  

根据上面三个一步一步的问题，我们就可以认为任何的一个区块链都符合下面的架构。  

  

![](https://i.imgur.com/V5EwP76.png)

  

  

在极度抽象的情况下，任何的区块链其实就三个部分，其中Computing Runtime部分就是账本的记账逻辑，State Storage负责状态的存储和查询，Consensus/Network负责共识以及接收外部的交易和查询。对于web2的同学来说，仍然是符合 MVC的分层逻辑的。  

  

那么在Substrate内部来说，账本具体怎么工作的，也就是上图的 Computing Runtime 部分是由一个一个的 Pallet 协同组合完成的，而State Storage 则是paritydb 或者rocksdb 负责的，Consensus/Network 则是Client的部分。但是Runtime 和 Client 都需要读取和写入状态，并且他们之间还需要互相通信，那么他们之间究竟是怎么被组织到一起从而完成了一条链的业务逻辑？  

  

  

一张图就可以搞清楚他们之间的协作关系：  

  

![](https://i.imgur.com/ClitvZG.png)

  

首先我们从离业务开发最近的Client 调用 Runtime 这条路径说起，Client 需要Runtime的API为了做什么呢？在Substrate中Client调用Runtime需要完成几个事情：  

  

1.  完成交易的合法性验证 （交易池）
    
2.  **暴露RPC调用方法，供外部调用（Runtime开发的日常）**
    
3.  完成共识的相关逻辑（验证人切换，验证人是否应该出块的逻辑）
    

  

不过对于实现链上业务逻辑来说，了解暴露RPC这一个用处就足够了，比如链上博客系统，就需要实现自定义的RPC，来给前端提供feed流，那么为什么不能简单粗暴的直接查询存储呢？是因为博客系统这个的业务逻辑决定的，在Substrate的存储中，所有的KV并不会按照进入Storage的先后顺序在内存中排列，从数据库的角度来看也就是不能进行有序迭代，那么博客这种明显两个文章之间有先后顺序的业务就没有办法直接查询存储了，所以一般来说都是直接在内存中维护一个索引来保持顺序。然后查询的时候通过索引直接把feed流查询出来。

  

**一个简单实现在这里：https://github.com/cybernonce/substrate-blog**

* * *

  

  

OK，那么我们就从这个方向来分析，Substrate怎么一步一步地把Pallet的接口暴露给Client使用的。在开始分析之前，我们记住一个结论: **Substrate de Runtime 会以一个Rust Struct 的形式存在，我们的所有Pallet都会作为这个Runtime Struct 中的一个 Field Struct 形式存在，然后Pallet内部的方法都会以Struct的静态方法的形式存在**（至于为什么是这个样子，留个悬念后面我们会专门出一个系列文章分析这个设计的原理和哲学，罪魁祸首就是WASM格式）。For Example:

  

  

![](https://i.imgur.com/h1KYCgX.png)

  

这个时候有些聪明的小伙伴已经有想法了，只要在Client的代码里面能拿到这个Runtime Struct就可以随便调用其中的Pallet的公开的静态方法了。  

  

![](https://i.imgur.com/7U1Z1KZ.png)

首先我们根据pallet_contract 这个模块入手来看如何实现runtime api，首先就是定义一个runtime api，为了简单，做了裁减  

![](https://i.imgur.com/YPjuttI.png)

其实就是一个普通的Trait定义被包裹在了一个Macro中

而实现部分长这个样子

![](https://i.imgur.com/mqFVPLu.png)

在实现中我们发现了一个非常有意思的线索：  

![](https://i.imgur.com/mWkYmb0.png)

这样的一个Trait Bound ，直接限定了Client这个类型，内部的关联类型需要实现这个Trait，那么我们看看，pallet_contract 的这个Runtime API 是怎么实现的。  

![](https://i.imgur.com/Pht0BkB.png)

在substrate-contract-node 这个项目中的runtime构建部分，我们看到，实现Runtime API 的本质就是，在Runtime 生产的过程中，加入对Pallet的调用。

那么实现一个Runtime API 我们需要完成几个事。

1.  完成Pallet 业务逻辑
    
2.  完成 Runtime API 的定义
    
3.  实现Runtime API 的定义
    
4.  在Client 侧调用 Runtime API 
    

  

在Client 调用 Runtime API 如何操作呢，显然RPC的实现告诉我们答案了，只要对Client加上一个Trait Bound 就可以完成对一个抽象的Generics直接调用Runtime API 方法。

  

  

* * *

  

分析完了Client怎么调用Runtime的，那么我们就去另外的一个方向，怎么从Runtime 调用Client。同样的我们也需要知道为什么Runtime需要调用Client，说起来，最直接的一个需求就是，我们的Pallet内部定义的那些存储变量，需要从数据库才能访问出来，而数据库在Substrate看起来是一个外部的东西，这也就是是Substrate所谓的Externalities。既然需要访问数据库，那么自然而然，Runtime访问Client就是必须的了，不过这个和Runtime开发离得稍微有点远，属于Substrate的内部能力，这个方向牵扯的东西有点多，所以干脆直接看他生成结果成什么样子就好了。

假设我们有这样的一个runtime interface

![](https://i.imgur.com/0sUGglm.png)

那么他生成了什么东西呢?  

  

![](https://i.imgur.com/v8AP9jA.png)

  

生成以后我们发现了一个非常有意思的实现，也就是在这个mod 内部我们为这个 &mut dyn sp\_externalities::Externalities实现了我们定义的Interface，而这个Interface内部调用的基本都是sp\_io的方法。也就是我们让我们的Runtime Interface 直接默认委托给了Externalities。  

那么就有观众好奇了，Externalities到底做了什么呢？这个详细的展开，我们放在后面来解释，现在只需要知道一个结论就好了，就是runtime interface 给了我们直接绕过 frame\_support 定义的那些存储直接访问底层ParityDB的能力，当然啦，frame\_support 内部的那四个存储，也只是在sp_io这个系列函数上包装出来的结构而已，本质也是利用了Externalities的能力直接访问存储。

  

那么在开发自己的链的时候，这两个东西的主要用法：  

1.   decl\_runtime\_api 这个东西，主要用来实现RPC，也就是我们自己定义的访问链上数据的方式。
    
2.  #\[runtime_interface\]就是一个让我们自己跨越各种复杂的抽象定义来直接访问ParityDB的能力，比如直接生成Paritial Key 去在ParityDB底层的BTree上完成Range Query。  
    

var first\_sceen\_\_time = (+new Date()); if ("" == 1 && document.getElementById('js\_content')) { document.getElementById('js\_content').addEventListener("selectstart",function(e){ e.preventDefault(); }); }