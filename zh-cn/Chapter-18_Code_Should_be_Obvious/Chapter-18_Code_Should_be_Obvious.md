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