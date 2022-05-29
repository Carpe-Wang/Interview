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
