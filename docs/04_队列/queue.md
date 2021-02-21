# 队列和栈

前面讲了线性和链式结构，如果你顺利掌握了，下边的队列和栈就小菜一碟了。因为我们会用前两章讲到的东西来实现队列和栈。
之所以放到一起讲是因为这两个东西很类似，队列是先进先出结构(FIFO, first in first out)，
栈是后进先出结构(LIFO, last in first out)。

生活中的数据结构：

- 队列。没错就是咱平常排队，第一个来的第一个走

本章我们详细讲讲常用的队列

# 队列 Queue

这里卖个关子，如果你熟悉了上两节讲的内容，这里你会选取哪个数据结构作为队列的底层存储？
还记得第一章讲的如何实现 ADT 吗？我视频了说了三个注意事项：

- 1.如何选用恰当的数据结构作为存储？
- 2.选取的数据结构能否满足 ADT 的功能需求
- 3.实现效率如何？

我们先来看看 list 可以不？对照这个三个需求，看看能否满足：

- 1.我们选择了 list
- 2.看起来队列需要从头删除，向尾部增加元素，也就是 list.insert(0, element) 和 list.append(element)
- 3.嗯，貌似 list.insert(0, element) 会导致所有list元素后移，O(n)复杂度。append 平均倒是O(1)，但是如果内存不够还要重新分配内存。

你看，使用了 list 的话频繁 insert(0, element) 和 append 都是非常低效的。

脑子再转转， 我们第二章实现了 链表 LinkedList，看看能否满足要求：
- 1.这里选择 LinkedList
- 2.删除头元素 LinkedList.popleft()，追加 append(element)。都可以满足
- 3.哇欧，这两个操作都是 O(1) 的，完美。

好， 就用 LinkedList 了，我们开始实现，具体看视频。这次实现我们还将演示自定义异常和测试异常。

```py
# -*- coding: utf-8 -*-

from collections import deque

# NOTE：注意这里是第三章 linked_list.py 里的内容，为了使文件自包含，我直接拷贝过来的


class Node(object):
    def __init__(self, value=None, next=None):   # 这里我们 root 节点默认都是 None，所以都给了默认值
        self.value = value
        self.next = next

    def __str__(self):
        """方便你打出来调试，复杂的代码可能需要断点调试"""
        return '<Node: value: {}, next={}>'.format(self.value, self.next)

    __repr__ = __str__


class LinkedList(object):
    """ 链接表 ADT
    [root] -> [node0] -> [node1] -> [node2]
    """

    def __init__(self, maxsize=None):
        """
        :param maxsize: int or None, 如果是 None，无限扩充
        """
        self.maxsize = maxsize
        self.root = Node()     # 默认 root 节点指向 None
        self.tailnode = None
        self.length = 0

    def __len__(self):
        return self.length

    def append(self, value):    # O(1)
        if self.maxsize is not None and len(self) >= self.maxsize:
            raise Exception('LinkedList is Full')
        node = Node(value)    # 构造节点
        tailnode = self.tailnode
        if tailnode is None:    # 还没有 append 过，length = 0， 追加到 root 后
            self.root.next = node
        else:     # 否则追加到最后一个节点的后边，并更新最后一个节点是 append 的节点
            tailnode.next = node
        self.tailnode = node
        self.length += 1

    def appendleft(self, value):
        headnode = self.root.next
        node = Node(value)
        self.root.next = node
        node.next = headnode
        self.length += 1

    def __iter__(self):
        for node in self.iter_node():
            yield node.value

    def iter_node(self):
        """遍历 从 head 节点到 tail 节点"""
        curnode = self.root.next
        while curnode is not self.tailnode:    # 从第一个节点开始遍历
            yield curnode
            curnode = curnode.next    # 移动到下一个节点
        yield curnode

    def remove(self, value):    # O(n)
        """ 删除包含值的一个节点，将其前一个节点的 next 指向被查询节点的下一个即可
        :param value:
        """
        prevnode = self.root    #
        curnode = self.root.next
        for curnode in self.iter_node():
            if curnode.value == value:
                prevnode.next = curnode.next
                del curnode
                self.length -= 1
                return 1  # 表明删除成功
            else:
                prevnode = curnode
        return -1  # 表明删除失败

    def find(self, value):    # O(n)
        """ 查找一个节点，返回序号，从 0 开始
        :param value:
        """
        index = 0
        for node in self.iter_node():   # 我们定义了 __iter__，这里就可以用 for 遍历它了
            if node.value == value:
                return index
            index += 1
        return -1    # 没找到

    def popleft(self):    # O(1)
        """ 删除第一个链表节点
        """
        if self.root.next is None:
            raise Exception('pop from empty LinkedList')
        headnode = self.root.next
        self.root.next = headnode.next
        self.length -= 1
        value = headnode.value
        del headnode
        return value

    def clear(self):
        for node in self.iter_node():
            del node
        self.root.next = None
        self.length = 0

######################################################
# 下边是 Queue 实现
######################################################


class EmptyError(Exception):
    """自定义异常"""
    pass


class Queue(object):
    def __init__(self, maxsize=None):
        self.maxsize = maxsize
        self._item_link_list = LinkedList()

    def __len__(self):
        return len(self._item_link_list)

    def push(self, value):    # O(1)
        """ 队尾添加元素 """
        return self._item_link_list.append(value)

    def pop(self):
        """队列头部删除元素"""
        if len(self) <= 0:
            raise EmptyError('empty queue')
        return self._item_link_list.popleft()


def test_queue():
    q = Queue()
    q.push(0)
    q.push(1)
    q.push(2)

    assert len(q) == 3

    assert q.pop() == 0
    assert q.pop() == 1
    assert q.pop() == 2

    import pytest    # pip install pytest
    with pytest.raises(EmptyError) as excinfo:   # 我们来测试是否真的抛出了异常
        q.pop()   # 继续调用会抛出异常
    assert 'empty queue' == str(excinfo.value)


class MyQueue:
    """
    使用 collections.deque 可以迅速实现一个队列
    """
    def __init__(self):
        self.items = deque()

    def append(self, val):
        return self.items.append(val)

    def pop(self):
        return self.items.popleft()

    def __len__(self):
        return len(self.items)

    def empty(self):
        return len(self.items) == 0

    def front(self):
        return self.items[0]
```

# 用数组实现队列

难道用数组就不能实现队列了吗？其实还是可以的。只不过数组是预先分配固定内存的，所以如果你知道了队列的最大长度，也是
可以用数组来实现的。

想象一下，队列就俩操作，进进出出，一进一出，pop 和 push 操作。
似乎只要两个下标 head, tail 就可以了。 当我们 push 的时候赋值并且前移 head，pop 的时候前移 tail 就可以了。你可以在纸上
模拟下试试。列队的长度就是 head-pop，这个长度必须不能大于初始化的最大程度。

如果 head 先到了数组末尾咋办？重头来呗，只要我们保证 tail 不会超过 head 就行。

head = 0,1,2,3,4 ... 0,1,2,3,4 ...

重头再来，循环往复，仿佛一个轮回。。。。
怎么重头来呢？看上边数组的规律你如果还想不起来用取模，估计小学数学是体育老师教的。

```py
maxsize = 5
for i in range(100):
    print(i % maxsize)
```

![](./array_queue.png)

我们来实现一个空间有限的循环队列。ArrayQueue，它的实现很简单，但是缺点是需要预先知道队列的长度来分配内存。

```py
# -*- coding: utf-8 -*-


# NOTE: 从 array_and_list 第一章拷贝的代码
class Array(object):

    def __init__(self, size=32):
        self._size = size
        self._items = [None] * size

    def __getitem__(self, index):
        return self._items[index]

    def __setitem__(self, index, value):
        self._items[index] = value

    def __len__(self):
        return self._size

    def clear(self, value=None):
        for i in range(len(self._items)):
            self._items[i] = value

    def __iter__(self):
        for item in self._items:
            yield item


class FullError(Exception):
    pass


class ArrayQueue(object):
    def __init__(self, maxsize):
        self.maxsize = maxsize
        self.array = Array(maxsize)
        self.head = 0
        self.tail = 0

    def push(self, value):
        if len(self) >= self.maxsize:
            raise FullError('queue full')
        self.array[self.head % self.maxsize] = value
        self.head += 1

    def pop(self):
        value = self.array[self.tail % self.maxsize]
        self.tail += 1
        return value

    def __len__(self):
        return self.head - self.tail


def test_queue():
    import pytest    # pip install pytest
    size = 5
    q = ArrayQueue(size)
    for i in range(size):
        q.push(i)

    with pytest.raises(FullError) as excinfo:   # 我们来测试是否真的抛出了异常
        q.push(size)
    assert 'full' in str(excinfo.value)

    assert len(q) == 5

    assert q.pop() == 0
    assert q.pop() == 1

    q.push(5)

    assert len(q) == 4

    assert q.pop() == 2
    assert q.pop() == 3
    assert q.pop() == 4
    assert q.pop() == 5

    assert len(q) == 0
```

# 双端队列 Double ended Queue
看了视频相信你已经会实现队列了，你可能还听过双端队列。上边讲到的队列 队头出，尾尾进，我们如果想头部和尾巴都能进能出呢？
这就是双端队列了，如果你用过 collections.deque 模块，就是这个东西。他能高效在两头操作。

假如让你实现你能想起来嘛？
似乎我们需要一个能 append()  appendleft() popleft() pop() 都是 O(1) 的数据结构。

上边我们实现 队列的 LinkedList 可以吗？貌似就差一个 pop() 最后边的元素无法实现了。
对，我们还有双端链表。它有这几个方法：

- append
- appendleft
- headnode()
- tailnode()
- remove(node)    # O(1)

啊哈，似乎删除头尾都可以啦，而且都是 O(1) 的，完美。
交给你一个艰巨的任务，实现双端队列 Deque（） ADT。你可以参考前几章的任何代码，挑战一下这个任务，别忘记写单元测试呦。当然如果没想出来也没关系，后边我们实现栈的时候还会用到它，那里我们会实现这个代码。


# 思考题
- 你能用 python 的 deque 来实现 queue ADT 吗？
- 哪些经典算法里用到了队列呢？
