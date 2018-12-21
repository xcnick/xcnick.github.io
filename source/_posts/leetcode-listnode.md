title: LeetCode链表总结
date: 2018-11-23 15:26:43
tags: LeetCode
---

> 链表题目总结
<!-- more -->
# 删除链表中的节点

> 请编写一个函数，使其可以删除某个链表中给定的（非末尾）节点，你将只被给定要求被删除的节点。现有一个链表 -- head = [4,5,1,9]，它可以表示为: 4 -> 5 -> 1 -> 9

由于给定了node节点，所以无需再次遍历。另外，无法获取node前驱节点，所以只能将node的后续节点的值赋给node，然后删除node后续节点。

```Cpp
void deleteNode(ListNode* node) {
  node->val = node->next->val;
  ListNode *tmp = node->next;
  node->next = node->next->next;
  delete tmp;
  tmp = NULL;
}
```

# 删除链表的倒数第N个节点

> 给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。
给定一个链表: 1->2->3->4->5, 和 n = 2.
当删除了倒数第二个节点后，链表变为 1->2->3->5.

双指针问题，第一个指针走n步，第二个指针与第一个指针一起移动，需考虑删除节点为头节点的情况。也可通过添加一个空头，消除头节点的特殊判断。

```Cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
  ListNode *fast = head;
  ListNode *slow = head;
  for (int i = 0; i < n; ++i) {
    fast = fast->next;
  }
  if (fast == NULL) {
    head = head->next;
    return head;
  }
  while (fast->next != NULL) {
    slow = slow->next;
    fast = fast->next;
  }

  ListNode *nextNode = slow->next;
  slow->next = slow->next->next;
  delete nextNode;
  nextNode = NULL;
  return head;
}
```

# 反转链表

> 反转一个单链表。
示例:
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL

可以用递归和迭代两种方法来解决。
* 递归：
```Cpp
ListNode* reverseList(ListNode* head) {
  if (head == NULL || head->next == NULL) {
    return head;
  }
  ListNode *ans = reverseList(head->next);
  head->next->next = head;
  head->next = NULL;
  return ans;
}
```

* 迭代：从头逐一处理节点
```Cpp
ListNode* reverseList(ListNode* head) {
  ListNode *p = head;
  ListNode *newHead = NULL;
  while (p != NULL) {
    ListNode *tmp = p->next;  // 保存后继节点
    p->next = newHead;        // head指向前驱节点
    newHead = p;              // 前驱节点更新
    p = tmp;                  // 当前节点更新
  }
  return newHead;
}
```

# 合并两个有序链表

> 将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

可以用递归和迭代两种方法来解决。
* 递归：
```Cpp
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
  if (l1 == NULL) {
    return l2;
  }
  if (l2 == NULL) {
    return l1;
  }
  ListNode *head = NULL;
  if (l1->val <= l2->val) {
    head = l1;
    head->next = mergeTwoLists(l1->next, l2);
  } else {
    head = l2;
    head->next = mergeTwoLists(l1, l2->next);
  }
  return head;
}
```

* 迭代：
```Cpp
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
  if (l1 == NULL) {
    return l2;
  }
  if (l2 == NULL) {
    return l1;
  }
  ListNode *head = NULL, *p = NULL;
  if (l1->val <= l2->val) {
    head = l1;
    l1 = l1->next;
  } else {
    head = l2;
    l2 = l2->next;
  }
  p = head;
  while (l1 && l2) {
    if (l1->val <= l2->val) {
      p->next = l1;
      l1 = l1->next;
    } else {
      p->next = l2;
      l2 = l2->next;
    }
    p = p->next;
  }
  if (l1) {
    p->next = l1;
  } else if (l2) {
    p->next = l2;
  }
  return head;
}
```

# 回文链表

> 请判断一个链表是否为回文链表。

需要找到链表的中点，可以使用快慢指针实现。快指针每次走两步，慢指针每次走一步，当快指针走完时，慢指针的位置就是中点。
另外，还需要用到栈，每次慢指针走一步，将值存入栈中，当到达中点时，链表的前半段就存入栈中。再用栈后进先出的性质，可以和后半段链表按照回文对应比较。

hea```Cpp
bool isPalindrome(ListNode *head) {
  if (!head || !head->next) {
    return true;
  }
  ListNode *slow = head, *fast = head;
  stack<int> s;
  s.push(head->val);
  while (fast->next && fast->next->next) {
    slow = slow->next;
    fast = fast->next->next;
    s.push(slow->val);
  }
  if (!fast->next) {
    s.pop();
  }
  while (slow->next) {
    slow = slow->next;
    int tmp = s.top();
    s.pop();
    if (tmp != slow->val) {
      return false;
    }
  }
  return true;
}
```

如果要只能用O(1)的空间复杂度，则不能使用stack。使用stack是为了使用其后进先出的特点，可以倒着取出前半段元素。若不使用stack，则可将后半段链表翻转，即可按照回文比较。

```Cpp
bool isPalindrome(ListNode *head) {
  // ...
  // 前面快慢指针的运动与之前代码一致
  // ...

  // 反转后半部分链表
  ListNode *last = slow->next, *pre = head;
  while (last->next) {
      ListNode *tmp = last->next;
      last->next = tmp->next;
      tmp->next = slow->next;
      slow->next = tmp;
  }
  // 依次比较头节点pre和中间节点slow的值
  while (slow->next) {
      slow = slow->next;
      if (pre->val != slow->val) return false;
      pre = pre->next;
  }
  return true;
}
```

# 环形链表

> 给定一个链表，判断链表中是否有环。

可使用快慢指针的方法。设两个指针，一个每次走一步的慢指针和一个每次走两步的快指针，如果链表里有环的话，两个指针最终肯定会相遇。

```Cpp
bool hasCycle(ListNode *head) {
  ListNode *slow = head, *fast = head;
  while (fast && fast->next) {
    slow = slow->next;
    fast = fast->next->next;
    if (slow == fast) {
      return true;
    }
  }
  return false;
}
```