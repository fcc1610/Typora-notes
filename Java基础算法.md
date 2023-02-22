[TOC]

### 时间复杂度（Time Complexity）

算法的 *“执行时间”* 和 *输入问题规模* 之间的关系

- 执行时间：不是实际的时间

- 输入问题规模：具体问题具体分析

  ![image-20230213202534336](C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230213202534336.png)

### 链表（Linked List）

- 由结点构成的列表

- 线性的数据结构

  java自带的链表LinkedList不常用。

```java
class ListNode(){
	public ElementType val;
    public ListNode next;
}
```

#### 链表的操作

- 遍历(traverse)

- 插入(insert)

- 查找(find)

- 删除(delete)

- 更新(update)



#### 遍历(traverse)

```java
//初始化：
ListNode node1 = new ListNode(1);
ListNode node2 = new ListNode(3);
ListNode node3 = new ListNode(5);
ListNode node4 = new ListNode(7);

node1.next = node2;
node2.next = node3;
node3.next = node4;

ListNode cur = node1;
while(cur != null){
    System.out.print(cur.val+" ");
    cur = cur.next;
}
```

#### 插入(insert)

```java
public void add(int location,int val){
    ListNode node = new ListNode(val);
    if(location > 0){
        ListNode pre = head;
        for(int i = 0;i < location - 1;i++){
            pre = pre.next;
        }
        node.next = pre.next;
        pre.next = node;
    }else if(location == 0){
        node.next = head;
        head = node;
    }
}
```

#### 查找(find)

```java
public ListNode find(int location){
    ListNode node = head;
    for(int i = 0;i < location;i++){
        node = node.next;
    }
    return node;
}
```

#### 删除(delete)

```java
public int remove(int location){
    ListNode pre = head;
	if(location > 0){
        for(int i = 0;i < location - 1;i++){
        	pre = pre.next;
    	}
        pre.next = pre.next.next;
    }else if(location == 0){
        head = head.next;
    }
}
```

#### 更新(update)

```java
public void update(int location,int value){
    ListNode node = head;
    for(int i = 0;i < location;i++){
        node = node.next;
    }
    node.val = value;
}
```

#### 哑结点(Dummy Node)

- 在整个链表前插一个哨兵节点
- 使得对于链表头结点 和 链表中间结点 操作逻辑相同

```java
ListNode dummy = new ListNode();
dummy.next = head;
```





