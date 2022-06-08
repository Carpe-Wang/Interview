### 给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。
![在这里插入图片描述](https://img-blog.csdnimg.cn/53acb964620541cc90a3167aa38e0997.png)

```go
type ListNode struct {
	Val  int
	Next *ListNode
}

func removeNthFromEnd(head *ListNode, n int) *ListNode {

	// 方便删除头节点
	var dummy *ListNode = &ListNode{Next:head}

	slow, fast := dummy,dummy//快慢指针
	for i := 0; i < n; i++ {//找到要删除的节点。
		fast = fast.Next
	}

	for fast.Next != nil {
		slow, fast = slow.Next, fast.Next //然后 slow 和 fast 一起走，当 fast 到链表尾部时，此时 slow.Next 就是我们想要删除的节点。
	}

	slow.Next = slow.Next.Next //删除
	return dummy.Next
}
```

>刚才突然又刷到了，这次写一个java的快慢指针，和go的栈。
* java快慢指针
```java
class Solution {
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode temp = new ListNode(-1);
    temp.next = head;
    ListNode fast = temp;
    ListNode slow = temp;
    while(n > 0) {
        fast = fast.next;
        n--;
    }
    ListNode pre = null;
    while(fast != null) {
        pre = slow;
        fast = fast.next;
        slow = slow.next;
    }
    pre.next = slow.next;
    return temp.next;
    }
}
```


* go 栈
```go
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    nodes := []*ListNode{}
    dummy := &ListNode{0, head}
    for node := dummy; node != nil; node = node.Next {
        nodes = append(nodes, node)
    }
    prev := nodes[len(nodes)-1-n]
    prev.Next = prev.Next.Next
    return dummy.Next
}
```
