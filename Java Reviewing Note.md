##### 面向对象

- 封装，隐藏具体的实现，让调用者无法触及不应当触及的部分，而且类内部的修改对外部透明，外部的调用不受影响
- 继承，创建了一个类之后，如果需要一个新类，而且功能类似，也就是说以现有类为基础，复制一份，然后修改或添加复制的类，继承就能达到这样的效果。一般来说理想的继承是存在`is a`关系，可以用派生类完全替代一个基类，而如果派生类需要增加一些自己的方法，通过基类无法访问，那么这种关系成为`is like a`
- 多态，调用同名方法而执行不同的功能，具体表现为重载和重写
  - 重载，同名但是参数列表不同，编译器通过参数的静态类型来决定使用哪个版本，然而当实参为字面量时，则编译器只能确定更为合适的调用版本，因为字面量没有静态类型
  - 重写，将基类的实现覆盖，两个不同的派生类可以对同一个方法重写，从而达到调用该方法而执行的具体内容不同，这是因为在指令执行阶段的第一步就会去查找调用者也可以成为接收者的实际类型，也就是在`class`字节码的解析过程中就会解析到不同的直接引用上，那么也就会调用其重写后的方法

##### ArrayList

- 构造
  - 默认构造方法，`elementData`（存值的一个`Object`数组）指向一个默认的空数组（成员变量）
  - `int`参数的构造方法，判断参数是否大于0，大于零直接创建一个该大小的Object数组，等于0则指向名为`EMPTY_ELEMENTDATA的Object空数组`，小于0抛异常
  - 利用其他`Collection`构造，将参数转成`array`赋值给`elementData`，`size`同样赋值成对应的长度，如果长度为0，那赋值成`Object空数组`（成员变量）
- 增
  - 判断`size + 1`是否需要扩容（`size`为有数据的长度），如果是默认构造函数创建的`arraylist`，扩容到默认容量（10），否则`(size+1) > arr.length ? 那要扩容，默认扩容原来长度的一般，如果不够，那就扩容到size+1的长度，如果新扩容的长度大于MAX_ARRAY_SIZE(Integer.MAX_VAUE - 8) 就 判断 size + 1 > MAX_ARRAY_SIZE ? 长度为 Integer.MAX_VALUE : MAX_ARRAY_SIZE ： 否则就不扩容` 不管是否扩容modCount都++
- 删除
  - 首先对要删除的`index rangeCeck()`不通过会抛异常，modCount++，删除一定会修改数组结构，用 `size - index - 1`算出要移动的`element的长度`，用`System.arraycopy`将这段往前移，删除最后一个size--
  - 如果是删除一个object，分两种情况，对象为非空则调用其equals方法，找到一个相等的调用fastRemove，若为空则判断到第一个为空的然后 fastRemove
  - 如果是调用 `removeAll`，用一个index记录保留的数量，循环`elementData`判断是否在`参数Collection中出现`，如果未出现则`elementData[index++]`，循环结束后然后将index到size的其余位置全部置空，如果在循环中出现了异常，则从出现异常的位置开始，全部保留，其余同样也置空

##### ConcurrentHashMap

- `ConcurrentHashMap`使用锁分段提升并发访问效率

- `HashTable`使用`synchronized`保证线程安全，在竞争较激烈的时候效率非常低，某线程在执行`put`操作时，其他线程不仅不能`put`，`get`也不能使用
- `HashMap`多线程时线程不安全，在并发执行`put`操作的时候，会导致某个`index`上的`Entry`链表形成环，那其的`next`节点用不为空，就产生了死循环

##### 阻塞队列

- `ArrayBlockingQueue` 数组结构的有界阻塞队列，按照先进先出的原则对数组排序
- `LinkedBlockQueue` 链表结构的有界阻塞队列，也是先进先出的
- `LinkedTransferQueue`链表结构的无界阻塞队列
- `LinkedBlockingDeque`链表结构的双向阻塞队列，可以在两头拿任务，可以运用在工作窃取模式
- `PriorityBlockingQueue`支持优先级的无界阻塞队列，默认是自然升序，可以实现自定义类的`compareTo`方法，或者在构造函数中传入`comparator`比较器
- `DelayQueue`支持延时获取元素的无界阻塞队列，可以运用于缓存系统的设计，如果能获取到元素的值，说明有效期已到
- `SynchronousQueue`不存储元素的无界阻塞队列，一个`put`后必须等待一个`take`，否则不能添加元素了

##### 异常

- 基类是`Throwable`，下分为`Error`以及`Exception`异常
- `Error`是指程序中出现的严重错误，一旦出现只能终止运行，如`OutOfMemoryError, StackOverflowError`
- `Exception`分为受检查异常和运行时异常
  - 受检查异常也就是编译器异常，需要在代码中`try catch`，如`IOException, ClassNotFoundException`，其中`IOException 又可细分为 FileNotFoundException, EOFException`
  - 运行时异常`RuntimeException`，这个异常出现的原因是代码没写好，我们写代码的时候应该尽量去避免这种异常的出现而不是去捕获它然后做挽救措施，有`NPE, IndexOutOfBoundsException, NoSuchElementException, NoSuchMethodException, ClassCastException`

