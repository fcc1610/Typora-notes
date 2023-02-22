[TOC]



### 时间复杂度（Time Complexity）

算法的 *“执行时间”* 和 *输入问题规模* 之间的关系

- 执行时间：不是实际的时间

- 输入问题规模：具体问题具体分析

  ![image-20230213202534336](C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230213202534336.png)

------



### *链表(Linked List)*

| 遍历 |                   |
| ---- | ----------------- |
| 插入 | add(location,val) |
| 查找 | get(location)     |
| 更新 | set(location,val) |
| 删除 | remove(location)  |



#### 遍历(traverse)

```python
cur = node_1
while cur is not None:
    	print(cur.val,end=' ')
        cur = cur.next
```

##### quiz: 

改变结点指向，观察遍历输出结果是否改变

```
node_1 = ListNode(1)
node_2 = ListNode(3)
node_3 = ListNode(5)
node_4 = ListNode(7)

node1.next = node_2
node2.next = node_3
node3.next = node_4

node2 = node3

cur = node1
while cur is not None:
	print(cur.val,end=‘ ’)
    cur = cur.next
```

![image-20230213152850883](C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230213152850883.png)

输出结果不变，为1 3 5 7

#### 插入(insert)

```python
def add(self,location,val):
	if location > 0
		for i in range(location - i):
			pre = pre.next
        new_node = ListNode(val)
        new_node.next = pre.next
        pre.next = new_node
	elif location == 0
		new_node = ListNode(val)
        new_node.next = self.head
        self.head = new_node
```

#### 查找

```python
def get(self,location):
	cur = self.head
	for i in range(location):
		cur = cur.next
	return cur.val
```

#### 修改

```python
def set(self,location,val):
	cur = self.head
	for i in range(location):
		cur = cur.next
	cur.val = val;
```

#### 删除

```python
def remove(self,location):
	if(location > 0):
		pre = self.head
		for i in range (location - 1):
			pre = pre.next
			
			pre.next = pre.next.next
			
	elif(location == 0):
		self.head = self.head.next;
```

#### 判空

```python
def if_empty(self):
	return self.head is None
```

#### 练习

删除链表中倒数第n个结点

```python
def remove_nth_from_end(self, head: ListNode, n: int) -> ListNode:
        # write your code here   
        node2 = node1 = head
        for i in range(n):
            node1 = node1.next
        # 1 -> null , 1 的情况
        if (node2 == head and node1 is None):
            return head.next

        while node1.next:
            node1 = node1.next
            node2 = node2.next
            
        node2.next = node2.next.next
        return head
```

为克服要删除的结点是第一个结点的情况，在head前添加一个哑结点dummy，指向head。

```python
    def remove_nth_from_end(self, head: ListNode, n: int) -> ListNode:
        # write your code here
        dummy = ListNode(0, head)
        first = head
        second = dummy
        for i in range (n):
            first = first.next
        while first:
            first = first.next
            second = second.next
        second.next = second.next.next
        return dummy.next
```





------

### *栈（Stack）*

#### 栈的操作

- push(val)

- pop

- peek() ：查看栈顶值

- is_empty()

  

#### 栈的实现

- 基于list实现stack
- list的尾部相当于stack的栈顶

<img src="C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230222162154228.png" alt="image-20230222162154228" style="zoom:80%;" /> 

![image-20230222162917417](C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230222162917417.png) 



#### quiz 423

whether the string is a valid parentheses

<img src="C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230222164136895.png" alt="image-20230222164136895" style="zoom:150%;" />



#### 调用栈 （Call Stack）

- 栈的应用
  - 操作系统用来保存函数的状态





------

### *队列（Queue）*

#### 队列的操作

- enqueque(val): 进队列
- dequeue():   出队列
- size(): 队列中元素个数
- is_empty(): 队列是否为空

<img src="C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230222171129373.png" alt="image-20230222171129373" style="zoom: 80%;" /> 



#### 队列的实现

- 使用LinkedList实现队列（ 为什么不用List? ）（ 因为List在头部操作需要O(n) ）
- 使用queue模块

<img src="C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230222171607934.png" alt="image-20230222171607934" style="zoom:80%;" /> 

![image-20230222183434735](C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230222183434735.png)
