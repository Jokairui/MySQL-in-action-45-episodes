### HashMap 源码解析

#### Put过程

1. 检查table数组是否需要初始化

2. 用key的hash值与table数组长度与运算，获取存放下标

3. 如果当前位置为空，新建node节点，并赋值给当前位置数组

4. Else，再检查新key与旧key是否相等（先通过hash值判断，再通过==号判断，最后通过新key不为空+equals方法判断），如果key相同，则更新旧的value值

5. else，判断当前节点是否为红黑树结点，如果是，走红黑树流程

6. else，开始遍历当前节点的链表，依次判断key指与之后节点的key值是否相同，直到链表尾部，此时，再判断当前链表长度是否到达了需要树化的临界值

7. 如果是当前key已存在的情况，更新旧的value值，之后调用子类的钩子函数afterNodeAccess，

   同时put函数return

8. 否则，增加modCount的值

9. 增加size的值并判断是否达到临界值，需要rehash

10. 最后调用钩子函数afterNodeInsertion

#### resize过程

