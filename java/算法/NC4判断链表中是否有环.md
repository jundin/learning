>判断链表是否有环应该是老生常谈的一个话题了，最简单的一种方式就是快慢指针，慢指针针每次走一步，快指针每次走两步，如果相遇就说明有环，如果有一个为空说明没有环。

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null){
            return false;
        }
        // 快慢两个指针
        ListNode slow = head;
        ListNode fast = head;
        while(fast !=null && fast.next != null){
            //慢指针每次走一步
            slow = slow.next;
            //快指针每次走两步
            fast = fast.next.next;
            //如果相遇，说明有环，直接返回true
            if(slow ==fast){
                return true;
            }
        }
        //否则就是没环
        return false;
    }
}
```