### 给你一个链表数组，每个链表都已经按升序排列。请你将所有链表合并到一个升序链表中，返回合并后的链表。

![在这里插入图片描述](https://img-blog.csdnimg.cn/cf91811848194280bb1ce4717e973eba.png)

>小顶堆

```go
type ListNode struct {
	Val  int
	Next *ListNode
}

type ListNodes []*ListNode

func (l ListNodes) Len() int {
	return len(l)
}

func (l ListNodes) Less(i, j int) bool {
	return l[i].Val <= l[j].Val
}

func (l ListNodes) Swap(i, j int) {
	l[i], l[j] = l[j], l[i]
}

func (l *ListNodes) Pop() interface{} {
	n := len(*l)
	x := (*l)[n-1]
	*l = (*l)[:n-1]
	return x
}

func (l *ListNodes) Push(x interface{}) {
	*l = append(*l, x.(*ListNode))
}

var _ heap.Interface = (*ListNodes)(nil)

func mergeKLists(lists []*ListNode) *ListNode {
	dummy := new(ListNode)
	pre := dummy
	hp := &ListNodes{}
	for i := 0; i < len(lists); i++ {
		if lists[i] != nil {
			heap.Push(hp, lists[i])
		}
	}
	for len(*hp) > 0 {
		top := heap.Pop(hp).(*ListNode)
		pre.Next = top
		pre = pre.Next
		if top.Next != nil {
			heap.Push(hp, top.Next)
		}
	}
	return dummy.Next
}
```
