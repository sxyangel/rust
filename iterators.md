## 迭代器

下面我们来探讨一下循环问题。

还记得 Rust 的 **for** 循环吗？下面有一个例子：

    for x in 0..10 {
    	println!("{}", x);
    }

现在你已经知道了更多的 Rust，我们可以详细谈谈它是如何工作的。范围 **(0 . . 10)** 是一个迭代器。我们可以使用 **.next()** 方法反复调用迭代器，它给出了事情的一个序列。

如下所示：

    let mut range = 0..10;
    
    loop {
    	match range.next() {
    		Some(x) => {
    			println!("{}", x);
   			 },
   			None => { break }
    	}
    }

我们针对范围给出一个可变的绑定，这就是迭代器。然后用一个内在的 **match** 进行 **loop** 。用这个 **match** 操作 **range.next()** 的结果，这就给出了到迭代器的下一个值的一个引用。**next** 返回一个 **Option<i32>**，在这种情况下，一旦循环运行完毕，我们会得到一个值：  **Some(i32)** 或者 **None**。如果我们得到 **Some(i32)**，就打印出来，如果我们得到 **None**，就跳出循环。

这个代码示例和我们的 **for** 循环版本基本上是一样的。**for** 循环仅仅是编写**loop/match/break** 构造的一个方便的方式。

然而，**for** 循环不是唯一使用迭代器的情况。编写自己的迭代器包括实现**迭代器**的特征。虽然这种操作不是在本指南的范围之内，Rust 提供了许多有用的迭代器来完成各种任务。在我们谈论这些之前，我们应该谈论一下 Rust 反模式。这就是范围的使用方式。

是的，我们刚刚谈到范围很有用。但范围也很原始。例如，如果你需要遍历一个向量的内容，你可能会这样写：

    let nums = vec![1, 2, 3];
    
    for num in &nums {
    	println!("{}", num);
    }

这比使用一个真正的迭代器严格的多。你可以直接迭代向量，像下面写的这样：
    
    let nums = vec![1, 2, 3];
    
    for num in &nums {
    	println!("{}", num);
    }

这样做有两个原因。首先，这可以更直接的表达我们的意思。我们遍历整个向量，而不是遍历索引，然后通过索引访问向量。第二，这个版本是更高效的：第一个版本将有额外的边界检查，因为它使用了索引、**nums[i]**。但是因为我们使用迭代器依次针对向量的每个元素产生一个引用，在第二个示例中不涉及边界检查。这对于迭代器是很常见的：我们可以忽略不必要的检查范围，但仍知道我们是安全的。

关于 **println!** 怎样工作，这里还有一个细节不是 100% 的清楚。**num** 实际上是 **&i32** 类型的数字。也就是说，它是对 **i32** 的一个引用，而不是本身就是 **i32**，**println!** 为我们处理非关联化的事物，所以我们不能看到它的细节。下面这段代码同样工作正常：

    let nums = vec![1, 2, 3];
    
    for num in &nums {
    	println!("{}", *num);
    }

现在我们来明确非关联化 **num**。为什么 **&num** 可以给我们引用？首先，因为我们明确地使用 **&** 调用。其次，如果给我们数据本身，我们必须是它的拥有者，它将制造一个数据的副本并将副本给我们。与引用相比，我们只是借用了引用数据，所以它只是传递一个引用,而不需要做移动。

所以，既然我们已经认定范围往往不是你想要的,让我们来谈谈你想要的东西。

这里主要有3个类，它们是彼此相关的事物：迭代器（iterators），迭代器适配器（iterator adapters），和消费者（consumers）。下面给出一些定义：

- 迭代器给出序列值。
- 迭代器适配器使用一个迭代器，产生一个新的的迭代器，它拥有不同的输出序列。
- 消费者使用一个迭代器，产生一些值最后的设置。

首先让我们来谈谈消费者，因为你已经看到一个迭代器。

### 消费者

consumer 操作一个迭代器，返回某些类型的值。最常见的 consumer 是 **collect()**。下面的代码并没有完全编译，但它已经显示了意图：

    let one_to_one_hundred = (1..101).collect();

正如你所看到的，我们可以在我们的迭代器上调用 **collect()**。**collect()** 尽可能多地接收迭代器给它的值，并返回结果的集合。为什么这不能编译呢？Rust 不能确定你想收集什么类型的值，所以你需要让它知道。下边是编译的版本：
    
    let one_to_one_hundred = (1..101).collect::<Vec<i32>>();

如果你还记得，**::<>** 语法允许我们给出一个类型提示，所以我们可以告诉它，我们需要一个整数向量。尽管你并不总是需要使用整个类型。使用一个 **_** 可以允许你提供部分提示：

    let one_to_one_hundred = (1..101).collect::<Vec<_>>();

即：“请收集 **Vec<T>**，但为我推断出T是什么。”因为这个原因 **_** 有时被称为一种“占位符”。

**collect()** 是最常见的 consumer，但也存在其他的消费者。find()就是其中之一：

    let greater_than_forty_two = (0..100)
     							.find(|x| *x > 42);
    
    match greater_than_forty_two {
    	Some(_) => println!("We got some numbers!"),
    	None => println!("No numbers found :("),
    }

find 消耗一个闭包，针对一个迭代器的每个元素的引用操作。如果元素是我们要找的元素，这个闭包返回 **true**，否则,则返回 **false**。因为我们可能找不到一个匹配的元素，**find** 返回一个 **Option** 而不是元素本身。

另一个重要的消费者是 **fold**。下面就是 fold 的示例：

    let sum = (1..4).fold(0, |sum, x| sum + x);

**fold()** 是一个消费者，语法：**fold(base, |accumulator, element| ...)**。它需要两个参数：第一个是一个称为 **基** 的元素 。第二个是一个闭包，本身有两个参数：第一个被称为累加器，第二个是一个元素。在每次迭代中，调用闭包，结果是在下一次迭代中累加器的值。在第一次迭代中，base 是累加器的值。

好吧，这有点令人困惑。让我们看看在这个迭代器中所有事物的值：

<table>
<tr>
<th>基</th>
<th>累加器</th>
<th>元素</th>
<th>闭包结果</th>
</tr>
<tr>
<td>0</td>
<td>0</td>
<td>1</td>
<td>1</td>
</tr>
<tr>
<td>0</td>
<td>1</td>
<td>2</td>
<td>3</td>
</tr>
<tr>
<td>0</td>
<td>3</td>
<td>3</td>
<td>6</td>
</tr>
</table>

我们使用这些参数来调用 fold() 函数:

    .fold(0, |sum, x| sum + x);

所以，**0** 是基，**sum** 是累加器，**x** 是我们的元素。在第一次迭代中，我们将 **sum** 设置为 **0**，**x** 是 **nums** 的第一个元素 **1**。然后将 **sum** 和 **x** 相加，即 **0 + 1 = 1**。第二次迭代中，和值成为我们的累加器 **sum**，元素是数组的第二个元素 **2**，相加，即 **1 + 2 = 3** ，这样就得到了最后一次迭代的累加器的值。在这次迭代中，**x** 是最后一个元素 **3** ，相加，即 **3 + 3 = 6**，这就是求和最后的结果。**1 + 2 + 3 = 6**，这就是我们最后得到的结果。

对于 **fold** 如果你刚开始接触它，可能会觉得这种语法有点奇怪，但一旦开始使用它，您会发现它的使用范围很广，几乎到处都可以使用。任何时候，如果你有一系列的事物，而你想要一个单一的结果，**fold** 都是最适当的。

由于迭代器存在另一个我们还没有谈到属性:懒惰，消费者就变得尤为重要。让我们多谈论一些关于迭代器的问题，你就会明白消费者为什么如此重要。

### 迭代器

正如之前说过的，我们可以使用 **.next()** 方法反复调用一个迭代器，它给出了一个事情的序列。因为你需要调用这个方法，这意味着迭代器可以偷懒，而不是预先生成的所有值。例如，在这段代码中，实际上并没有生成 1-100的数字，它仅仅代表了一个序列，而不是产生一个值：

    let nums = 1..100;

因为我们没有针对范围做任何事情，它不会生成序列。下面让我们加入消费者:

    let nums = (1..100).collect::<Vec<i32>>();

消费者 **collect()** 要求范围给它一些数字，这样它才会做生成序列的工作。

您将看到范围是两个基本的迭代器之一。另一种是 **iter()**。**iter()** 可以把一个向量变成一个简单的迭代器，反过来这个迭代器给出每个元素：

    let nums = vec![1, 2, 3];
    
    for num in nums.iter() {
    	println!("{}", num);
    }

这两种基本迭代器应该能很好地为你服务。有一些更高级的迭代器，包括那些不限范围的。

这里谈论到的迭代器已经足够你使用。迭代器适配器是我们需要谈论的关于迭代器最后的概念。

## 迭代器适配器

迭代器适配器获得一个迭代器，以某种方式对其进行修改，产生一个新的迭代器。最简单的一个叫做 **map**：

    (1..100).map(|x| x + 1);

**map** 被另一个迭代器调用，产生一个新的迭代器，在新的迭代器中，每个元素引用迭代器给出的关闭作为调用它的参数。这将打印出 2-100 的数字。如果你编译这个示例，您会得到一个警告：

    warning: unused result which must be used: iterator adaptors are lazy and
     		do nothing unless consumed, #[warn(unused_must_use)] on by default
    (1..100).map(|x| x + 1);
     ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

迭代器的懒惰又起作用了！这个闭包永远不会执行。这个例子也不会打印任何数字:

    (1..100).map(|x| println!("{}", x));

如果你只是为了测试其副作用，而在一个迭代器上执行一个闭包，那只需要使用 **for** 就好了。

有很多有趣的迭代器适配器。**take(n)** 将返回一个新的迭代器，在原来的迭代器的 n 个元素的基础上。注意，这个对于原迭代器没有副作用。让我们使用一下之前提到的无限迭代器：

    for i in (1..).step_by(5).take(5) {
    	println!("{}", i);
    }

这将打印出

    1
    6
    11
    16
    21

**filter()**是一个以一个闭包作为参数的适配器。这个闭包返回 **true** 或者 **false**。新的迭代器 **filter()** 产生唯一的元素，闭包返回true：

    for i in (1..100).filter(|&x| x % 2 == 0) {
    	println!("{}", i);
    }

这将打印 1 到 100 之间的所有的偶数。(注意：因为 **filter** 不消耗将遍历的元素，它只是传递每一个元素的引用，从而可以使用 **&x** 模式过滤谓词来提取整数本身。)

现在你可以将三件事放在一起考虑：首先是一个迭代器，经过几次调整，然后消耗这个结果。检查一下:

    (1..1000)
    	.filter(|&x| x % 2 == 0)
    	.filter(|&x| x % 3 == 0)
    	.take(5)
    	.collect::<Vec<i32>>();

这将给你一个包含 6、12、18、24 和 30 的向量。

这只是一个关于迭代器，迭代器适配器，和消费者的小的尝试。这有很多非常有用的迭代器，您也可以编写您自己的迭代器。迭代器提供一个安全、有效的方式来操纵各种列表。起初你会觉得它们有点不同寻常，但一旦你开始操作它们，你就会迷上它们。关于迭代器和消费者的不同点的完整列表，你可以查阅[迭代器模块文档。](https://doc.rust-lang.org/stable/std/iter/)