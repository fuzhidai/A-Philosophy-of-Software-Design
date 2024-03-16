翻译完成时间：2024.03.16

第一次校对时间：-

第二次校对时间：-

---

第 2.3 章中提过复杂度的两大诱因，模糊性正是其中之一。如果系统的重要信息对新开发者来说并不清晰，就会出现模糊性。通过编写更清晰的代码，将能够解决模糊性问题。本章将讨论的一些因素，它们会让代码变得更加清晰或模糊。

如果代码清晰，那意味着别人能在少量思考后，快速阅读代码，并对代码的行为和含义进行准确初估。且读者无需花费过多的时间和精力，来获取用于协作代码的信息。如果代码含糊不清，那读者就必须花费很多的时间和精力来理解。这不仅会降低效率，更会增加理解错误和产生漏洞的概率。清晰代码所需的注释少于含糊不清的代码。

“清晰” 是指读者脑中的感受，发现别人模糊的代码，比发现自己代码存在的问题更容易。因此，代码审核是判定代码清晰度的最佳方式。如果某人在看了你的代码后，认为不够清晰，那无论你认为它对你来说多清楚，它确实就是不清晰的。通过理解代码含糊不清的原因，将能够学到如何编写更好的代码。

### 18.1 让代码更加清晰的一些事

在前面的章节中，已经讨论过让代码更加清晰的两个关键技巧。第一个是选择好的名称（第 14 章）。准确且有意义的名称能够阐明代码的行为，并减少文档的需求。如果名称是模糊不清的，那读者需要通读代码才能够推断出命名实体的含义，这样非常的耗时并且容易出错。第二个技巧是一致性（第 17 章）。如果总是通过相同的方式来完成相似的事情，那读者就能意识到，他们可以从之前的看过的内容中推导模式，并在无需详细分析代码的情况下，立刻安全地得出结论。

这里是其它的一些通用技巧，能够让代码变得更加清晰：

**明智地使用空白符。**代码组织的形式会影响它理解起来的难度。考虑下面的参数文档，其空格已经被移除。

```auto
/**
* ...
* @param numThreads The number of threads that this manager should
* spin up in order to manage ongoing connections. The MessageManager
* spins up at least one thread for every open connection, so this
* should be at least equal to the number of connections you expect
* to be open at once. This should be a multiple of that number if
* you expect to send a lot of messages in a short amount of time.
* @param handler Used as a callback in order to handle incoming
* messages on this MessageManager's open connections. See
* {@code MessageHandler} and {@code handleMessage} for details.
*/
```

在该文档中，获取参数的开始点和结束点是非常困难的。甚至无法清晰的看出这里有多少个参数，以及它们的名称是什么。如果添加少量的空格，结构很快就会变的非常清晰，阅读起来也更加容易。

```auto
/**
* @param numThreads
* The number of threads that this manager should spin up in
* order to manage ongoing connections. The MessageManager
spins
* up at least one thread for every open connection, so this
* should be at least equal to the number of connections you
* expect to be open at once. This should be a multiple of
that
* number if you expect to send a lot of messages in a short
* amount of time.
* @param handler
* Used as a callback in order to handle incoming messages on
* this MessageManager's open connections. See
* {@code MessageHandler} and {@code handleMessage} for
details.
*/
```

使用空白行来分割代码中的主要模块也非常的有效，例如在下面的例子：

```auto
void* Buffer::allocAux(size_t numBytes)
{
  // Round up the length to a multiple of 8 bytes, to ensure alignment.
  uint32_t numBytes32 = (downCast<uint32_t>(numBytes) + 7) & ~0x7;
  assert(numBytes32 != 0);
  
  // If there is enough memory at firstAvailable, use that. Work down
  // from the top, because this memory is guaranteed to be aligned
  // (memory at the bottom may have been used for variable-size chunks).
  if (availableLength >= numBytes32) {
    availableLength -= numBytes32;
    return firstAvailable + availableLength;
  }
  
  // Next, see if there is extra space at the end of the last chunk.
  if (extraAppendBytes >= numBytes32) {
    extraAppendBytes -= numBytes32;
    return lastChunk->data + lastChunk->length + extraAppendBytes;
  }
  
  // Must create a new space allocation; allocate space within it.
  uint32_t allocatedLength;
  firstAvailable = getNewAllocation(numBytes32, &allocatedLength);
  availableLength = allocatedLength numBytes32;
  return firstAvailable + availableLength;
}
```

如果空白行后的首行为描述下一个代码块的注释，那这个方法将会更加的有效，空白行让注释更加清晰。

语句中的空格能够帮助划分语句的结构。比较下面的两个语句，一个有空格，而另一个则没有：

```auto
for(int pass=1;pass>=0&&!empty;pass--) {

for (int pass = 1; pass >= 0 && !empty; pass--) {
```

**注释。**有时无法完全避免代码变得模糊。此时，最重要的是使用注释来补充提供缺失的信息。为了做好注释，你必须将自己置于读者的位置，并指出哪些内容会让他们感到疑惑，以及哪些信息能够让迷惑消失。下一节将会展示几个例子。

### 18.2 让代码变得模糊的一些事

有些事情会让代码变得模糊，本节将会提供一些示例。部分示例，如事件驱动编程，在某些场景下是有效的，所以你可能会在任何情况下都适用该方式。如果发生了这样的情况，额外的文档能够尽可能减少读者的疑惑。

**事件驱动编程。**在事件驱动编程中，应用需要响应外部事件，例如网络包到达或鼠标点击。某个模块会负责报告到来的时间，另一些部分会通过让事件模块在某个事件到达时，调用某个指定方法，来注册其感兴趣的某个具体事件，

事件驱动编程让追踪控制流更加困难。事件处理器方法从不会被直接调用，而是被事件模块间接调用，通常是使用方法指针或接口。即使找到了事件模块中的调用位置，也无法确定哪个具体的方法会被调用，其由运行时注册的处理器所决定。因此，推理事件驱动代码，确保其正常工作通常是很难完成的。

消除这个模糊的行为，可以为每个处理器方法增加接口注解，指明其被调用的时机，如下例所示：

```auto
  /**
* This method is invoked in the dispatch thread by a transport if a
* transport-level error prevents an RPC from completing.
*/
void
Transport::RpcNotifier::failed() {
  ...
}
```

危险信号：模糊不清的代码

如果代码的含义和行为无法通过快速的阅读所理解，这是个危险的信号。这通常意味着当某人阅读代码时，某些重要信息没有及时清晰的反馈给他。

**通用容器。**许多语言为将两个或多个对象分组存储到一个对象中，提供了通用类，例如 Java 中的 Pair 或 C++ 中的 std::pair。这些类提供了将多个对象通过单个变量进行传递的简单方式，所以充满了吸引力。一个常见的用法是用它传递方法的多个返回值，例如这个 Java 示例：

```auto
return new Pair<Integer, Boolean>(currentTerm, false);
```

不幸的是，通用的容器会导致代码变得模糊，因为分组元素的通用名称掩盖了它们的真实含义。在上面这个例子中，调用方必须通过 result.getKey() 和 result.getValue() 来获取响应中的两个值，但这并没有提供任何关于两个值具体含义的线索。

因此，最好不要使用通用容器。如果你需要容器，定义一个新的类或结构，并具体指明其用途。可以为它的元素选择有意义的名称，并在声明上提供额外的文档，而这在通用容器中是无法完成的。

这个例子说明了这样一个通用规则：软件设计应易于阅读，而非易于编写。通用容器对代码编写者来说非常方便，但却给所有的阅读者造成了困惑。对于代码编写者来说，最好多花额外的几分钟来定义一个具体的容器结构，这会让代码更加清晰。

**声明和分配的类型不同。**考虑下面这个 Java 示例：

```auto
private List<Message> incomingMessageList;
...
incomingMessageList = new ArrayList<Message>();
```

变量被声明为 List 类型，但实际值却是一个 ArrayList。这份代码没有错误，因为 List 是 ArrayList 的超类，但却可能会误导读者，因为声明的类型并非实际分配的类型。实际类型可能会影响到变量的使用（ArrayList 与 List 的其它子类相比，具有不同的性能和线程安全属性），所以最好让声明与其定义一致。

**出乎读者意料的代码。**考虑下面这段代码，它是一个 Java 应用的主程序：

```auto
public static void main(String[] args) {
  ...
  new RaftClient(myAddress, serverAddresses);
}
```

大多数应用程序会在主程序返回后退出，所以读者可能会推测当前程序也会如此。但是事实却并非如此。RaftClient 构造器创建了额外的线程，在主线程结束后继续执行其它操作。这个行为应该在 RaftClient 构造器的接口注释中被记录，但即使这样也不够明显，应当在 main 函数结尾处也增加简短的注释，该注释应指明应用程序会在其它的线程中继续执行。如果代码符合读者预期的惯例，那它就是清晰的，若非如此，则重要的是记录这些行为，以确保读者不会感到困惑。

### 18.3 结论

思考清晰度的另一种方式，从信息的角度来考虑。如果代码不够清晰，那通常意味着代码的重要信息没有传达给读者，在 RaftClient 示例中，读者可能并不知道 RaftClient 构造器创建了其它线程，在 Pair 示例中，读者可能并不知道 result.getKey() 会返回当前学期的数量。

要让代码足够清晰，必须确保读者总能获取足够的信息来理解代码。你可以通过三种方式来实现。最好的方法是使用设计技巧，例如抽象和消除特例，来减少所需的信息数量。其次，可以利用读者已在其它上下文中获取到的信息（例如，遵循惯例，以符合期望），读者无需用你的代码中学习新的知识。最后，可以使用技巧如良好命名和战略注解，在代码中展示重要的信息。