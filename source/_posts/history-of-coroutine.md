---
title: 转载--计算机语言协程的历史、现在和未来
date: 2020-05-05 17:18:32
tags:
- coroutine
keywords:
- coroutine
- 协程
- categories:
- Coroutine
description: 本文原发于《程序员》2014年11月刊。简明介绍了协程的因何诞生，以及其兴衰史。
---

本文原发于《程序员》2014年11月刊。好文章备份一份

计算机科学是一门应用科学，几乎所有概念都是为了理解或解决实际问题而生的。协程 (Coroutine) 的出现也不例外。协程的概念，最早可以追溯到写作 COBOL 语言编译器中的技术难题。

## 从磁带到协程   

COBOL是最早的高级语言之一，编译器则是高级语言必不可少的一部分。现如今，我们对编译器的了解，已经到了可以把核心内容浓缩成一本教科书的程度。然而在二十世纪六十年代，如何写作高效的语言编译器还是绕不过的现实问题。例如，1960年夏天，D. E. Knuth就是利用开车横穿美国去加州理工读研究生的时间，对着[Burroughs 205](https://www.cs.virginia.edu/brochure/images/manuals/b205/central/central.html)机器指令集手写COBOL编译器。最早提出“协程”概念的Melvin Conway，也是从如何写一个只扫描一遍程序（one-pass）的COBOL编译器出发。众多的“高手”纷纷投入编译器开发，可见一门新科学发展之初也是筚路蓝缕。

以现代眼光来看，高级语言编译实际由多个步骤组合而成：词法解析、语法解析、语法树构建，以及优化和目标代码生成等。编译实质上就是从源程序出发，依次将这些步骤的输出作为下一步的输入，最终输出目标代码。在现代计算机上实现这种管道式的架构毫无困难：只需要依次运行，中间结果存为中间文件或放入内存即可。GCC和Clang编译器，以及[ANTLR](http://www.antlr.org/)构建的编译器，都遵循这种设计。

在Conway的设计里，词法和语法解析不再是独立运行的步骤，而是交织在一起。编译器的控制流在词法和语法解析之间来回切换：当词法模块读入足够多的token时，控制流交给语法分析；当语法分析消化完所有token后，控制流交给词法分析。词法和语法分别独立维护自身的运行状态。Conway构建的这种协同工作机制，需要参与者“让出（yield）”控制流时，记住自身状态，以便在控制流返回时能从上次让出的位置恢复（resume）执行。简言之，协程的全部精神就在于控制流的主动让出和恢复。我们熟悉的子过程调用可以看作在返回时让出控制流的一种特殊协程，其内部状态在返回时被丢弃了，因此不存在“恢复”这个操作。

以现在眼光来看，编译器的实现并非必需协程。然而，Conway用协程实现COBOL编译器在当时绝不是舍近求远。首先，从原理上，因为COBOL并不是[LL(1)](https://en.wikipedia.org/wiki/LL_parser)型语法，无法简单构建一个以词法分析为子过程的自动机。其次，当年计算机依赖于磁带存储设备，只支持顺序存储。也就是说，依次执行编译步骤并依靠中间文件通信的设计是不现实的，各步骤必须同步前进。正是这样的现实局限和设计需要，催生了协程的概念。

## 自顶向下，无需协同

虽然协程伴随着高级语言诞生，却没有能像子过程那样成为通用编程语言的基本元素。

从1963年首次提出到上世纪九十年代，我们在ALOGL、Pascal、C、FORTRAN等主流的命令式编程语言中都没有看到原生的协程支持。协程只稀疏地出现在Simula、Modular-2（Pascal升级版）和Smalltalk等相对小众的语言中。作为一个比子进程更加通用的概念，在实际编程中却没有取代子进程，不得不说出乎意外。但如果结合当时的程序设计思想看，又在意料之中：协程不符合那个时代所崇尚的“自顶向下”的程序设计思想，自然也就不会成为当时主流的命令式编程语言的一部分。

正如面向对象的语言是围绕面向对象的开发理念设计一样，命令式编程语言是围绕自顶向下的开发理念设计的。在这种理念的指导下，程序被切分为一个主程序和大大小小的子模块，每个子模块又可能调用更多子模块。C家族语言的main()函数就是这种自顶向下思想的体现。在这种理念指导下，各模块形成层次调用关系，而程序设计就是制作这些子过程。

在“自顶向下”这种层次化的理念下，具有鲜明层次的子过程调用成为软件系统最自然的组织方式，也是理所当然。相较之下，具有执行中让出和恢复功能的协程在这种架构下无用武之地。可以说，自顶向下的设计思想从一开始就排除了对协程的需求。其后的结构化编程思想，更进一步强化了“子过程调用作为唯一控制结构”的基本假设。在这样的指导思想下，协程没有成为当时编程语言的一等公民。

但作为一种易于理解的控制结构，协程的概念渗入到了软件设计的许多方面。在结构化编程思想一统天下之时，Knuth曾专门写过一篇[《Structured Programming with GOTO》](http://c2.com/cgi/wiki?StructuredProgrammingWithGoToStatements)来为GOTO语句辩护。在他列出的几条GOTO可以方便编程且不破坏程序结构的例子中，有一个（例子7b）就是用GOTO实现协程控制结构。相较之下，不用GOTO的“结构化”代码反而失去了良好的结构。当然，追求实际结果的工业界对于学界这场要不要剔除GOTO的争论并不感冒。当时许多语言都附带了不建议使用的GOTO语句，显得左右逢源。这方面一个最明显的例子就是Java——语言本身预留了goto关键字，而编译器却没提供任何支持，在这场争论中做足了中间派。

实践中，协程的思想频繁应用于任务调度和流处理。例如，Unix管道就可以看成是众多命令间的协同操作。当然，管道的现代实现都以pipe()系统调用和进程间的通信为基础，而非简单遵循协程的yield/resume语法。

许多协同式多任务操作系统，也可以看成协程运行系统。说到协同式多任务系统，一个常见的误区是认为协同式调度比抢占式调度“低级”，因为我们所熟悉的桌面操作系统，都是从协同式调度（如Windows 3.2、Mac OS 9等）过渡到抢占式多任务系统的。实际上，调度方式并无高下，完全取决于应用场景。抢占式系统允许操作系统剥夺进程执行权限，抢占控制流，因而天然适合服务器和图形操作系统，因为调度器可以优先保证对用户交互和网络事件的快速响应。当年Windows 95刚推出时，抢占式多任务就被作为一大卖点大加宣传。协同式调度则等到进程时间片用完或系统调用时转移执行权限，因此适合实时或分时等对运行时间有保障的系统。

另外，抢占式系统依赖于CPU的硬件支持。因为调度器需要“剥夺”进程的执行权，就意味着调度器需要运行在比普通进程高的权限上，否则任何“流氓（rogue）”进程都可以去剥夺其他进程了。只有CPU支持了执行权限后，抢占式调度才成为可能。x86系统从80386处理器开始引入Ring机制支持执行权限，这也是为何Windows 95和Linux其实只能运行在80386之后的x86处理器上的原因。而协同式多任务适用于那些没有处理器权限支持的场景，这些场景包括资源受限的嵌入式系统和实时系统。在这些系统中，程序均以协程的方式运行。调度器负责控制流的让出和恢复。通过协程的模型，无需硬件支持，我们就可以在一个“简陋”的处理器上实现多任务系统。许多常见的智能设备，如运动手环，受硬件所限，都采用协同调度架构。

## 协程的复兴和现代形式

编程思想能否普及开来，很大程度上在于应用场景。协程没有能在自顶向下的世界里立足，却在动态语言世界中大放光彩，这里最显著的例子莫过于Python的迭代器和生成器。
回想一下在C的世界里，循环的标准写法是
`for (i = 0; i < n; ++i) { ... } `
这行代码包含两个独立的逻辑：for循环控制了i的边界条件，++i控制了i的自增逻辑。这行代码适用于C世界里的数组即内存位移的范式，因此适合大多数访问场景。对于STL和复杂数据结构，因为往往只支持顺序访问，循环大多写成：`for (i = A.first(); i.hasNext(); i = i.next()) { ... } `

这种设计抽象出了一个独立于数据结构的迭代器，专门负责数据结构上元素的访问顺序。迭代器把访问逻辑从数据结构中分离出来，是一个常用的设计模式（GoF 23个设计模式之一），我们在STL和Java Collection中也常常看到迭代器的身影。在适当的时候，我们可以更进一步引入一个语法糖，将循环写成：```for i in A.Iterator() {func(i)}```

事实上，许多现代语言都支持类似的语法。这种语法抛弃了以i变量作为迭代指针的功能，要求迭代器自身能记住当前迭代位置，调用时返回下一个元素。读者不难看到，这就是我们在文章开始提到的语法分析器的架构。正因为如此，我们可以从协程的角度来理解迭代器：当控制流转换到迭代器时，迭代器负责生成和返回下一个元素。一旦准备就绪，迭代器就让出控制流。在Python中，这种特殊的迭代器实现又被成为生成器。以协程角度切入的好处在于设计大大精简。实际上，在Python中，生成器本身就是一个普通函数，和其他普通函数的唯一不同，在于它的返回语句是协程风格的yield。这里，yield一语双关，既是让出控制流，也是生成迭代器的返回值。

前文我们仅讨论了生成器最基本的特性。实际上，生成器的强大之处在于我们可以像Unix管道一样串联起来，组成所谓的生成器表达式。如果我们有一个可以生成1，2，3 …的生成器N，则square = (i **2 for i in N)就是一个生成平方数的生成器表达式。注意这里圆括号语法和[list comprehansion](http://en.wikipedia.org/wiki/List_comprehension)方括号语法的区别，square = [i **2 for i in N]是生成一个具体的列表。我们可以串联这些生成器表达式，最终的控制流会在这些串联的部分间转换，无需编写复杂的嵌套调用。当然，yield只是冰山的一角，现代的Python语言还充分利用了yield关键字构建[yield from](https://www.python.org/dev/peps/pep-0380/)语句、(yield)语法等，让我们毫无困难地将协程的思想融入到Python编程中，限于篇幅这里不再展开。

我们前面说过，协程的思想本质上就是控制流的主动让出和恢复机制。在现代语言中，可以实现协程思想的方法很多，这些实现间并无高下之分，所区别的就是是否适合应用场景。理解这一点，我们对于各种协程的分类，如半对称/对称协程、有栈与无栈协程等具体实现就能提纲挈领，无需在实现细节上纠结。

协程在实践中的实现方式千差万别，一个简单的原因是，协程本身可以通过许多基本元素构建。基本元素的选取方式不一样，构建出来的协程抽象也就有差别。例如，Lua语言选取了create、resume和yield作为基本构建元素，从调度器层面构建出所谓的“非对程”协程系统；而Julia语言绕过调度器，通过在协程内调用yieldto函数完成了同样的功能，构建出了一个所谓的对称协程系统。尽管这两种语言使用了同样的setjmp库，构造出来的原语却不一样。又如，许多C语言的协程库都使用了ucontext库实现，这是因为POSIX本身提供了ucontext库，不少协程实现是以ucontext为蓝本实现的。这些实现，都不可避免地带上了ucontext系统的一些基本假设，如协程间是平等的，一般带有调度器来协调协程等（如[libtask](http://swtch.com/libtask/)实现，以及云风的[coroutine](http://blog.codingnow.com/2012/07/c_coroutine.html)库）。Go语言的一个鲜明特色就是通道（channel）作为一级对象。因此，resume和yield等在其他语言里的原语在Go中都以通道方式构建。我们还可以举出诸多近似的例子。其风格差异往往和语言的历史、演化路径、要解决的问题相关，我们不必苛求其协程模型一定要如此这般。

总的来说，协程为协同任务提供了一种运行时抽象。这种抽象非常适合于协同多任务调度和数据流处理。在现代操作系统和编程语言中，因为用户态线程切换代价比内核态线程小，协程成为了一种轻量级的多任务模型。我们无法预测未来，但可以看到，协程已成为许多擅长数据处理语言的一级对象。随着计算机并行性能的提升，用户态任务调度已成为一种标准的多任务模型。在这样的大趋势下，协程这个简单且有效的模型就显得更加引人注目。