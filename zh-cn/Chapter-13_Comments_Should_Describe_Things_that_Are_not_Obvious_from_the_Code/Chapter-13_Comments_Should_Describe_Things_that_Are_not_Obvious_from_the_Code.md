之所以编写注释，是因为开发者在编写代码时脑中的信息，无法通过编程语言的语句来完整地表述。注释能够记录这些信息，所以后续的开发者能够轻易地理解和修改代码。注释的指导原则是，注释应该描述那些代码中无法清晰表达的事情。

很多事情都无法从代码中清晰表述。有时可能是一些低层细节，例如，如果一对索引描述了一个范围，那索引自身所对应的元素，是否属于该范围则并不明显。有时某段代码的存在原因可能并不清晰，或为什么它由某种特殊的方式来实现。有时存在一些开发者所遵循的规则，比如 “总是在调用 b 之前调用 a”。你可能可以在查看所有代码后，猜测到规则，但这个过程痛苦且易错，注释可以让规则变得明晰。

抽象是编写注释的一大原因，注释能提供更多代码中隐含的信息，其目的是以更简单的方式来思考事物。代码过于详细，无法提取抽象信息，而注释却可以提供一种更简单，更高层次的视图，比如：“该函数被调用后，网络流量将会限制为 maxBandwidth 字节每秒”。即使某些信息能够通过阅读代码来推断得出，我们也不想强迫模块用户做这些工作，阅读代码非常耗时，并会迫使他们考虑大量与使用模块无关的信息。开发者应能在仅阅读外部可见声明的背景下，理解模块提供的抽象。而实现该目的的唯一方式，只有在定义时提供注释。

本章将探讨哪些信息需要在注释中记录，以及如何编写良好的注释。正如你所见，良好注释通常表述与代码不同层次的细节，在某些情况下更详细，而另外一些情况则更粗略（抽象）。

### 13.1 遵守规范

编写注释的第一步，需要确定注释的规范，比如注释内容和注释格式。如果编程语言具有文档编译工具，例如 Java 的 Javadoc、C++ 的 Doxygen、或 Go 的 godoc，那请遵守这些工具的规范，虽然它们都不够完美，但工具提供了足够的好处来加以弥补。如果编程语言没有可以遵守的规范，那可以尝试采用其它相似语言或相似项目的规范，这样其他开发者将会更易理解和遵守。

规范服务于两个目的。首先，确保一致性，这让注释更易阅读和理解。其次，确保你确实是在编写注释。如果对于将要注释的内容和方式没有明确想法，那最终很容易编写出毫无意义的注释。

大多数注释可以分为以下几类：

**接口：**紧邻组件（类、数据结构、方法和函数）声明前的注释块，描述组件接口。对于某类，注释描述该类提供的整体抽象。对于某方法或函数，注释描述它的整体行为、参数和返回值，还可能包括副作用、抛出异常，以及调用方在调用前必须知晓的其它信息。

**数据结构成员：**紧邻数据结构字段定义的注释，例如实例变量或类静态变量。

**实现注释：**方法或函数内部的注释，描述代码内部的工作方式。

**跨组件注释：**描述跨组件边界依赖的注释。

前面两个类型的注释最为重要。所有类和方法都应具有接口注释，每个类变量也应具有注释。有时变量或方法的声明非常清晰，以至于无需增加注释，getter 和 setter 方法有时就是这类情况，但这种情况很少见，相比于花精力担心是否需要注释，注释所有内容则容易得多。实现（Implementation）注释通常是多余的（见下面的 13.6 节）。跨组件注释最为最少见，并且最难编写，但当需要它们时，它们往往又是比较重要的，13.7 节将详细探讨它们。

### 13.2 不要复述代码

不幸的是，许多注释并不是特别有用。最常见的原因是注释复述了代码，注释中的所有信息，都能够轻易的从注释紧邻的代码中推断出来。下面是近期某篇研究论文中出现的代码示例：

```auto
ptr_copy = get_copy(obj)             # 获取指针拷贝
if is_unlocked(ptr_copy):            # obj 是否没有锁？
  return obj                         # 返回当前 obj
if is_copy(ptr_copy):                # 是否为拷贝?
  return obj                         # 返回 obj
thread_id = get_thread_id(ptr_copy)
if thread_id == ctx.thread_id:       # 被当前 ctx 锁定
  return ptr_copy                    # 返回拷贝
```

除了 “被锁定” 注释以外，这些注释中没有任何有效信息，来提供某些关于线程的信息，无法从代码直接获得。可以发现，这些注释与代码的详细程度大致相同，每行代码都有一个注释，用以描述这行代码。类似这样的注释毫无用处。

这里是更多重复代码的注释样例：

```auto
// 增加一个水平滚动条
hScrollBar = new JScrollBar(JScrollBar.HORIZONTAL);
add(hScrollBar, BorderLayout.SOUTH);

// 增加一个竖直滚动条
vScrollBar = new JScrollBar(JScrollBar.VERTICAL);
add(vScrollBar, BorderLayout.EAST);

// 初始化插入符号位置的相关值
caretX = 0;
caretY = 0;
caretMemX = null;
```

这些注释没有提供任何价值。对于起始的两行注释，代码已经足够清晰，不再需要注释。在第三个情况下，注释可能有用，但当前注释没有提供足够有帮助的细节。

编写完注释后，询问自己下面这个问题：如果某人从未看过这些代码，他是否仅通过查看与注释相邻的代码，就可以编写出这些注释？如果答案是可以，那就像上面的示例一样，这些注释并没有让代码更易理解。像这样的注释，正是有些人认为注释毫无价值的原因。

另一个常见错误，是在注释中使用某些词语，而这些词语已在被注释实体名称中使用。

```auto
/*
* Obtain a normalized resource name from REQ.
*/
private static String[] getNormalizedResourceNames(HTTPRequest req) ...

/*
* Downcast PARAMETER to TYPE.
*/
private static Object downCastParameter(String parameter, String type)
...

/*
* The horizontal padding of each line in the text.
*/
private static final int textHorizontalPadding = 4;
```

这些注释只是将方法名或变量名中的单词提取出来，可能再增加一些参数名称和参数类型，并最终将它们拼成一个句子。例如，在第二个注释里，唯一未在代码中出现的词是 “to”。同样，只需查看声明，就可以完成这些注释的编写，而无需理解变量的含义，因此它们毫无价值。

**危险信号：注释复述代码**

如果注释中的信息，已经可以从相邻代码中明确获得，那这些注释就是无效的。如果注释使用了组成被描述事物名称的单词，那就是这种情况的一个例子。

同时，这些注释也遗漏了一些重要的信息，例如，什么是 “标准化的资源名称”，getNormalizedResourceNames 方法返回数组中的元素是什么？内边距的单位是什么？内边距是在每行的一侧还是两侧？在注释中记录这些信息，将会更有帮助。

编写良好注释的第一步，就是使用那些未在被描述实体名称中出现的词语。为注释选择能够提供有关实体含义额外信息的词语，而非仅仅重复其名称。例如，这是 textHorizontalPadding 方法优化后的注释：

```auto
/*
* The amount of blank space to leave on the left and
* right sides of each line of text, in pixels.
*/
private static final int textHorizontalPadding = 4;
```

这个注释提供了无法从声明本身直接获取的额外信息，例如单位（像素），以及内边距实际是对每行的两侧都生效。相比于直接使用术语 “内边距”，该注释解释了内边距是什么，以防读者不了解该术语。

### 13.3 低层注释提升准确度

现在已经知道了不应该做什么，接下来让我们讨论注释中应该提供哪些信息。注释通过提供与代码不同层次的细节来扩充代码。相比于代码，有些注释提供更低层、更细节的信息，通过明晰代码的准确含义，来提升准确度。另一些注释则提供更高层、更抽象的信息，以提供直观性，例如代码背后的推理，或通过更简单抽象的方式来思考代码。而与代码处于相同层级的注释，则更像是对代码的复述。本节会讨论更多低层方法的细节，下一节会讨论高层方法。

在注释变量（例如类实例变量、方法参数和返回值）时，准确度是最有用的。变量声明的名称和类型通常不够准确，注释可以补充这些缺失的细节，例如：

* 这个变量的单位是什么？
* 边界条件是开放还是关闭？
* 如果允许一个空值，它意味着什么？
* 如果变量引用一个最终必须被释放或关闭的资源，谁负责执行该操作？
* 变量（不变的）是否存在某些属性总是为真，比如 “这个列表总是至少包含一个条目”？

有些信息可以通过阅读使用该变量的代码获取，但这非常耗时且易错，声明的注释应足够清晰和完整，无需上述这些操作。当我说注释应描述代码未清晰表达的事情时，其中的 “代码” 是指紧邻注释（声明）的代码，而非 “应用的所有代码”。

注释过于含糊是变量注释的常见问题。这里有两个注释不够准确的样例：

```auto
// 响应缓冲区中的当前偏移量
uint32_t offset;

// 包含文档内的所有行宽和现实次数
private TreeMap<Integer, Integer> lineWidths;
```

第一个例子中，“当前” 意味着什么并不清楚。第二个例子中，TreeMap 的键是行宽，值是其出现的次数，而这些信息并不够清楚。同样，宽度是以像素衡量还是以字符衡量？下面修正后的注释提供了额外的细节：

```auto
// 未返回给客户端的首个对象，在此缓冲区中的位置
uint32_t offset;

// 持有关于行长度的数据，其格式为 <长度，数量>，
// 长度是一行中字符串的数量（包括换行符），
// 数量是包含相同数量字符的行数。
// 如果没有特定长度的行，则没有该长度的条目。
private TreeMap<Integer, Integer> numLinesWithLength;
```

第二个声明使用更长的命名来传达更多信息，并将单词 “width” 改为 “length”，这更容易让人认为值的单位是字符而非像素。需要注意的是，第二个注释不只记录了每个条目的细节，也记录了条目缺失的含义。

在注释变量时，考虑使用名词，而非动词。换句话说，关注变量代表什么，而非如何操作。考虑下面这个注释：

```auto
/* 跟随者变量: 指示器变量，让接收器和周期任务线程能够通信，
 * 以确认心跳是否在跟随者的选举超时时间窗口内被接收到。
 * 接收到有效心跳时，切换到 TRUE
 * 选举超时时间窗重置时，切换到 FALSE
 */
private boolean receivedValidHeartbeat
```

该注释描述了如何通过类中的几段代码修改变量，如果它能描述变量代表什么，而非只是反映代码结构，那它将更加简短有效。

```auto
/* True 意味着在上次选举计时器重置后，已经接收到心跳。
 * 用于在接收器和周期任务间通信。
 */
private boolean receivedValidHeartbeat;
```

通过这个注释，很容易就可以推断出，接收到心跳时，该变量一定会被设置为 true，而选举计时器重置后，该值将置为 false。

### 13.4 高层注释增强直觉

提供直觉是注释增强代码的第二种方式。相比于代码，这些注释在更高的层次上编写，省略了细节，以帮助读者理解整体的意图和代码的结构。通常用于方法的内部注释，以及接口注释。例如，考虑下面这些代码：

```auto
// 如果有一个 LOADING 状态的 readRpc，
// 使用与 assignPos 指向的 PKHash 相同的会话，
// 并且该 readRpc 中的最后一个 PKHash 小于当前分配的 PKHash，
// 那我们将分配 PKHash 放入该 readRpc 中。
int readActiveRpcId = RPC_ID_NOT_ASSIGNED;
  for (int i = 0; i < NUM_READ_RPC; i++) {
    if (session == readRpc[i].session
          && readRpc[i].status == LOADING
          && readRpc[i].maxPos < assignPos
          && readRpc[i].numHashes < MAX_PKHASHES_PERRPC) {
      readActiveRpcId = i;
      break;
  }
}
```

这里的注释过于低层和详细。一方面，它部分复述了代码 “如果这里存在 readRPC 的状态为 LOADING”，只是复述了判断 readRpc[i].status == LOADING。另一方面，该注释没有解释代码的整体意图，以及代码在方法中的作用。因此，它并不能帮助读者理解代码。

更好一些的注释：

```auto
// 尝试将当前密钥哈希，附加到尚未发送至所需服务器的现有 RPC 中
```

该注释没有包含任何细节，相反，它从更高层次上描述代码的整体功能。有了这些高层次信息，读者几乎可以解释代码发生的所有事情：循环必须遍历所有现存的远程过程调用（RPCs），session 判断可能用于确认特定 RPC 是否被指派给了正确的服务端，LOADING 判断表明 RPCs 可以很多状态，在某些状态下，添加很多哈希并不安全，MAX - PKHASHES_PERRPC 判断表明对于单个 RPC 可以发送多少个哈希，存在一定的限制。唯一无法通过注释解释的，就是 maxPos 判断。此外，新注释为读者评判代码提供了基础：代码是否执行了将密钥哈希添加到现有 RPC 时，需要完成的所有操作？原始注释没有描述代码的整体意图，所以读者很难确定代码的行为是否正确。

高层注释比低层注释更难编写，因为你必须以不同的方式来思考代码。问问自己：这段代码想要做什么？表述代码所有信息的最简方式是什么？这段代码最重要的是什么？

工程师往往非常注重细节，热爱细节，并善于管理大量细节，这也是成为一名优秀工程师的必要条件。但优秀的软件设计师也应能跳出细节，从更高的层次思考系统。这意味着需要确定系统最重要的方面，忽略底层细节，只考虑系统最基本的特征，这是抽象的本质（找到简单的方式来考虑复杂的实体），也是编写高层次注释的必要行为。良好的高层次注释，表述一个或几个提供概念框架的简单想法，例如 “附加到现有 RPC 中”。有了框架，就很容看出特定的代码语句，如何与整体的目标相关联。

这是另一个代码样例，具有良好的高层次注释：

```auto
if (numProcessedPKHashes < readRpc[i].numHashes) { 
  // 某些密钥哈希可能无法在该请求中被找到
  // （可能因为它们没有存储在服务器上，或服务崩溃，
  // 或响应消息中没有足够的空间）。
  // 标记未被加工的哈希值，这样它们能够在新的 RPCs 中被重新分配。
  for (size_t p = removePos; p < insertPos; p++) {
    if (activeRpcId[p] == i) {
      if (numProcessedPKHashes > 0) {
        numProcessedPKHashes--;
      } else {
        if (p < assignPos)
          assignPos = p;
        activeRpcId[p] = RPC_ID_NOT_ASSIGNED;
      }
    }
  }
}
```

这段注释做了两件事，第二句提供了代码功能的抽象描述，第一句则有些不同：它解释（从高层）了为什么代码会被执行。“我们是如何走到这一步的” 形式的注释，对于帮助读者理解代码非常有效。例如，注释方法时，描述最有可能调用该方法的条件，将会非常有用（尤其如果代码只在特殊的场景下被调用）。

### 13.5 接口文档

注释的重要作用之一是定义抽象（abstractions）。正如第 4 章所述，抽象是实体的简化视图，在保留重要信息的基础上，省略非必要的细节。代码不适合描述抽象，它太过于底层，并包含了在抽象中需要忽略的实现细节。使用注释是描述抽象的唯一方法，因此如果想要代码表现出良好的抽象，则必须使用注释来记录这些抽象。

描述抽象的第一步，就是区分接口注释（interface comments）和实现注释（implementation comments）。接口注释提供的信息，是某人想要使用类或方法时，需要知晓的信息，它定义了抽象。实现注释描述类或方法为了实现抽象，其内部的工作方式。区分两种注释非常重要，这样接口用户才不会暴露于实现的细节。此外，两种形式的注释最好不同。如果接口注释也必须描述实现，那类或方法就会变得浅薄。这意味编写注释的行为，会给高质量的设计提供材料，第 15 章将会继续讨论这个想法。

类的接口注释提供了高层次的描述，用以记录类提供的抽象。示例如下：

```auto
/**
* This class implements a simple server-side interface to the HTTP
* protocol: by using this class, an application can receive HTTP
* requests, process them, and return responses. Each instance of
* this class corresponds to a particular socket used to receive
* requests. The current implementation is single-threaded and
* processes one request at a time.
*/
public class Http {...}
```

该注释描述了类的整体功能，不包含任何实现细节，甚至没有提到任何具体的方法。也记录了类实例的具体作用。最后，表述了类的限制（它不支持多线程并发访问），这对开发者衡量是否要使用它非常重要。

方法的接口注释，既包含用于抽象化的高层次信息，也包含用于准确化的低层次细节：

* 注释通常以一两句话开头，描述调用者所感知到的方法行为，这是高层次的抽象。
* 注释必须描述所有的参数和返回值（如果有的话）。这些注释必须非常准确，且必须描述参数的限制，以及参数之间的依赖关系。
* 如果方法存在任何副作用，必须在接口注释中记录。任何影响系统未来行为，但却并不属于最终结果的后果，都是副作用。例如，如果方法向内部数据结构添加了一个值，且该值会由后续方法调用获取，那这就是副作用，向文件系统写入内容，同样也是副作用。
* 方法的接口注释，必须描述方法可能产生的所有异常。
* 如果存在方法调用前需要满足的先决条件，那必须记录它们（可能其它的方法必须先被调用。对于二分检索法，待检索的列表必须已被存储）。最小化前置条件很棒，但所有剩下的前置条件也都必须记录。

这里是一个方法的接口注释，描述了从缓冲区对象里拷贝数据：

```auto
/**
* Copy a range of bytes from a buffer to an external location.
*
* \param offset
* Index within the buffer of the first byte to copy.
* \param length
* Number of bytes to copy.
* \param dest
* Where to copy the bytes: must have room for at least
* length bytes.
*
* \return
* The return value is the actual number of bytes copied,
* which may be less than length if the requested range of
* bytes extends past the end of the buffer. 0 is returned
* if there is no overlap between the requested range and
* the actual buffer.
*/
uint32_t
Buffer::copy(uint32_t offset, uint32_t length, void* dest)
...
```

这段注释的语法（例如 \return）遵循了 Doxygen 的规范，Doxygen 是提取 C/C++ 代码，并将其编译为网页的程序。该注释提供了开发者调用该方法所需的所有信息，包括特殊情况的处理方式（记录了该方法如何遵循第十章的建议，不定义任何关于范围规则的错误）。开发者无需为了调用该方法而查阅其实现，同时，该接口注释未提供任何与实现相关的信息，例如它如何扫描内部结构，以获取其所需的数据。

对于更广泛的例子，让我们考虑一个命名为 IndexLookup 的类，它是分布式存储系统的一部分。存储系统持有一个表集合，每个表都包含很多的对象。此外，每个表都可以具有一个或多个索引，每个索引根据对象的特定字段，提供对表中对象的高效访问。例如，某个索引可能用来基于对象的 name 字段进行检索，而另一个索引可能用来基于对象的 age 字段进行检索。使用这些索引，应用程序可以快速提取具有特定 name 的所有对象，或 age 在某个给定范围内的所有对象。

IndexLookup 类提供了便捷的接口来执行索引检索。这里是一些它可能在应用中使用的例子：

```auto
query = new IndexLookup(table, index, key1, key2);
while (true) {
  object = query.getNext();
  if (object == NULL) {
    break;
  }
  ... process object ...
}
```

应用程序首先通过提供查询表、索引以及索引范围参数（例如，如果索引是基于年龄字段，key1 和 key2 可能被指定为 21 和 65，以此检索所有年龄在该区间内的对象），构造 IndexLookup 的对象实例。然后应用程序多次调用 getNext 方法，每次调用将返回一个满足条件的对象，一旦所有匹配对象都已返回，getNext 将返回 NULL。由于存储系统是分布式的，所以该类的实现会有些复杂。表中的对象可能分布在多个服务器上，其索引也可能分布在一组不同的服务器上。因此，IndexLookup 类的代码必须先与所有相关索引服务器通信，收集满足范围的对象信息，然后必须与实际存储对象的服务器进行通信，获取对象值。

现在让我们考虑下，哪些信息需要包含在该类的方法注释中。对于下面给出的每条信息，问问自己，开发人员是否需要知道这些信息，才能使用这个类（我的答案在本章的结尾处）：

1. IndexLookup 类向持有索引和对象的服务器，所发送消息的格式。
2. 用于确定特定对象是否满足期望范围的对比方法（对比方法是使用整型、浮点型、数字、还是字符串？）。
3. 服务器存储索引的数据结构。
4. IndexLookup 是否是通过并发的方式，向多个不同的服务器发起请求。
5. 处理服务器崩溃的机制。

这里是 IndexLookup 类接口注释的原始版本，该摘录还包含了类定义中的几行，并在注释中引用了这些定义：

```auto
/*
* This class implements the client side framework for index range
* lookups. It manages a single LookupIndexKeys RPC and multiple
* IndexedRead RPCs. Client side just includes "IndexLookup.h" in
* its header to use IndexLookup class. Several parameters can be set
* in the config below:
* - The number of concurrent indexedRead RPCs
* - The max number of PKHashes a indexedRead RPC can hold at a time
* - The size of the active PKHashes
*
* To use IndexLookup, the client creates an object of this class by
* providing all necessary information. After construction of
* IndexLookup, client can call getNext() function to move to next
* available object. If getNext() returns NULL, it means we reached
* the last object. Client can use getKey, getKeyLength, getValue,
* and getValueLength to get object data of current object.
*/
class IndexLookup {
  ...
  private:
  /// Max number of concurrent indexedRead RPCs
  static const uint8_t NUM_READ_RPC = 10;
  /// Max number of PKHashes that can be sent in one
  /// indexedRead RPC
  static const uint32_t MAX_PKHASHES_PERRPC = 256;
  /// Max number of PKHashes that activeHashes can
  /// hold at once.

  static const size_t MAX_NUM_PK = (1 << LG_BUFFER_SIZE);
}
```

在继续阅读之前，看看自己是否能发现这个注释中的问题。这里是我发现的问题：

* 第一段的大部分关注实现，而非接口。例如，用户无需知道与服务器通信的特定远程调用名称。第一段后半部分提到的配置参数，都是私有变量，它们只与类的维护者相关，而与类的用户无关。注释中的这些实现信息都应该省略。
* 注释还包含了一些显而易见的事。例如，无需告知用户引入 IndexLookup.h：任何编写 C++ 代码的人都能猜到这是必要的。此外，文本 “通过提供所有必要信息” 什么也没说，所以可以省略。

对于这个类，一个简短的注释就够了（而且更可取）：

```auto
/*
* This class is used by client applications to make range queries
* using indexes. Each instance represents a single range query.
*
* To start a range query, a client creates an instance of this
* class. The client can then call getNext() to retrieve the objects
* in the desired range. For each object returned by getNext(), the
* caller can invoke getKey(), getKeyLength(), getValue(), and
* getValueLength() to get information about that object.
*/
```

该注释的最后一段并非绝对必要，因为它与方法注释存在大量的重合信息。但在类注释文档中提供相关示例，介绍其方法的协作方式可能会很有帮助，特别是对于使用模式并不明显的深层类。需要注意，新注释并没有提到 getNext 方法返回  NULL 的情况。该注释并不打算记录每个方法的所有细节，它只提供高层次的信息，帮助读者理解方法互相协作的方式，以及每个方法可能被调用的时机。有关详细信息，读者可以参考各个方法的接口注释。该注释也没有提到服务器崩溃，因为服务器崩溃对该类的用户并不可见（系统会自动恢复）。

**危险信号：实现文档污染接口**

当接口文档（例如方法接口文档）描述非使用所需的实现细节时，就是一个危险的信号。

现在考虑下面这段代码，展示了 IndexLookup 类中 isReady 方法的第一版文档注释：

```auto
/**
* Check if the next object is RESULT_READY. This function is
* implemented in a DCFT module, each execution of isReady() tries
* to make small progress, and getNext() invokes isReady() in a
* while loop, until isReady() returns true.
*
* isReady() is implemented in a rule-based approach. We check
* different rules by following a particular order, and perform
* certain actions if some rule is satisfied.
*
* \return
* True means the next Object is available. Otherwise, return
* false.
*/
bool IndexLookup::isReady() { ... }
```

再一次，该注释的大部分都不属于这里，例如对于 DCFT 的引用，以及关于实现的整个第二段，这是接口文档注释的常见错误。有些实现文档很有用，但它应位于方法内部，这样将能与接口文档更加清晰的区分开来。此外，文档的第一段比较模糊（RESULT_READY 是什么意思？），缺失了一些重要信息。最后，这里没有必要描述 getNext 方法的实现。下面是更好一些的版本：

```auto
/*
* Indicates whether an indexed read has made enough progress for
* getNext to return immediately without blocking. In addition, this
* method does most of the real work for indexed reads, so it must
* be invoked (either directly, or indirectly by calling getNext) in
* order for the indexed read to make progress.
*
* \return
* True means that the next invocation of getNext will not block
* (at least one object is available to return, or the end of
the
* lookup has been reached); false means getNext may block.
*/
```

这个版本的注释对 “就绪” 的含义，提供了更加准确的信息，并补充了一些重要的信息，即如果要继续基于索引进行检索，则必须调用该方法。

### 13.6 实现注解：是什么和为什么，而非如何做

实现（Implementation）注释是出现在方法内部的注释，用来帮助读者理解方法的内部工作方式。大多数的方法都很简短，所以它们并不需要任何的实现注释：有了代码和接口注释，很容易就可以理解方法的工作原理。

实现注释的主要目的是帮助读者理解代码在做什么（而非如何做），一旦读者知道代码要做什么，通常就很容易理解代码的工作方式。对于简短的方法，代码只做一件事，并且已经记录在了接口的注释中，所以并不需要实现注释。较长的方法具有多个代码块，每个代码块会执行方法整体任务的不同部分。在每个主要代码块之前增加注释，能够为代码块的行为，提供高层次（更抽象）的描述。下面是一个例子：

```auto
// 阶段 1: 扫描活跃的 RPCs 来检查是否某些已经完成。
```

对于循环来说，在循环之前提供注释，描述每个迭代发生了什么，会很有帮助：

```auto
// 下面循环的每个迭代提取请求消息中的一个请求，
// 增加相应的对象，并追加响应到响应消息中。
```

注意该注释如何从更抽象且易懂的层次上来描述循环，它没有陷入任何细节，比如请求如何从请求消息中提取出来，或对象是如何递增的。只有较长且较复杂的循环才需要注释，因为此时循环的行为可能并不明显，但很多循环都已足够简短，它们的行为已经非常清晰。

除了描述代码在做什么之外，实现注释对于解释原因也很有用。如果代码的某些方面比较棘手，无法仅通过阅读代码来理解，那应将其记录下来。例如，如果漏洞修复需要额外的代码，但其行为并不总是非常明确，可以添加注释来说明为什么需要这段代码。对于漏洞修复，有些写得很好的问题描述漏洞报告，注释可以引用这些漏洞追踪数据库中的报告，而非重复它们的细节（“修复 RAM-436，关于 Linux 2.4.x 设备驱动崩溃”）。开发人员可以通过查询漏洞数据库，获取更多的详细信息（这是防止注释重复的例子，将会在第十六章中讨论）。

对于较长的方法，为一些重要的本地变量编写注释，可能非常有帮助。可是，如果大多数本地变量具有良好的命名，那它们就不再需要注释。如果变量的所有用法都在上下几行内可见，那通常无需注解，也能轻松理解变量的用途。在这个场景下，可以让读者阅读代码来理解变量的含义。可是，如果变量的用法跨越大段代码，那应考虑增加注释来描述该变量。当注释变量时，关注变量代表什么，而非它如何在代码中被使用。