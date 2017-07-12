#写在最前面
导师贪腐出逃美国，两年未归，可怜了我。拿了小米和美团的offer，要被延期，offer失效，工作重新找。把准备过程纪录下来，共勉。


链表是最常考察的数据结构
    ```
    // 链表定义
    public class Node{
        int val;//数据域
        Node next;// 指针域

        public Node(int val){
            this.val = val;
        }
    }
    ```
## 基础题

### 求单链表的长度
考察链表遍历的方法，时间复杂度为o(n)
    ```
    //求单链表长度，起手判断链表是否为空，非常必要
    public static int getListLength(Node head){
        //如果链表为空，长度为0
        if(null == head){
            return 0;
        }
        // 开始着手办正事了
        int len = 0;
        Node cur = head;
        //如果循环判断条件不确定，可以先写逻辑，回头再定条件
        while(null != cur.next){
            len++;
            cur = cur.next;
        }
        return len;
    }
    ```
### 结点相对位置关系，结点与链表的相对位置关系
- **翻转链表**
    链表中经典解法1，引入两个指针，然后确定其相对位置关系
    + 迭代法

    解题思路：前后两个指针，遍历链表，每遍历一个结点，前面的指针将指向后面的指针，时间复杂度o(n)
    实际进行操作的是preCur这个指针，newHead与cur是两个游标，纪录位置
    ```
    public static Node getReverseList(Node head){
        if(null == head || null == head.next){
            return null;
        }

        Node cur = head;
        Node newHead = null;

        while(null != cur){
            Node preCur = cur;
            cur = cur.next;
            preCur.next = newHead;
            newHead = preCur;
        }

        return newHead;
    }
    ```

- **返回单链表倒数第k个结点**
    两个指针，前面的指针超前后面指针k个位置
    ```
    public static Node findLastKthNode(Node head,int k){
        if(k == 0 || null == head){
            return null;
        }

        Node p = head;
        Node q = head;

        while(k > 0 || p != null){
            p = p.next;
            k--;
        }

        if(k > 0 || p == null){
            return null;
        }

        while(p != null){
            q = q.next;
            p = p.next;
        }

        return q;
    }
    ```

- **查找中间结点**
    分析：两个指针可以解决，位置相对关系是：p指针向前移动一步，q指针向前移动两步
    ```
    public static Node getMiddleNode(Node head){
        if(null == head || head.next == null){
            return head;
        }

        Node p = head;
        Node q = head;

        while(p != null){
            p = p.next;
            q = q.next;
            if(p != null){
                p = p.next;
            }
        }

        return q;
    }
    ```

- **从尾到头打印单链表**
    ```
    public static void printList(Node head){
        Stack<Node> s = new Stack<>();

        Node cur = head;
        while(cur != null){
            s.push(cur);
            cur = cur.next;
        }

        while(!s.empty()){
            cur = s.pop();
            System.out.print(p.val + " ");
        }

        System.out.println();
    }
    ```

## 有环问题

- **判断一个单链表是否有环**
    思路：快慢两个指针，如果相遇，则有环
    ```
    public static boolean hasCycle(Node head){
        Node fast = head;
        Node slow = head;

        while(fast != null && fast.next != null){
            fast = fast.next.next;
            slow = slow.next;
            if(fast == slow){
                return true;
            }
        }

        return false;
    }
    ```
- **取出有环链表中，环的长度**
    思路：找到相遇的结点a后，让b结点从a位置向下遍历，回来后得到的结点数，即为环长度
    ```
    //方法：有环链表中，获取环的长度。参数node代表的是相遇的那个结点
    public int getCycleLength(Node node) {
        if(node == null){
            return 0;
        }

        int length = 0;
        Node cur = node;
        while(cur != null){
            cur = cur.next;
            length++;
            if(cur != node){
                return length;
            }
        }

        return length;
    }
    ```

- **判断两个链表相交**
    思路：如果两链接的尾结点相同，则有环。注意，有环的链表，此种方法失效。
    ```
    public static boolean isIntersect(Node head1, Node head2) {
        Node tail1 = head1;
        Node tail2 = head2;

        while(tail1 != null){
            tail1 = tail1.next;
        }

        while(tail2 != null){
            tail2 = tail2.next;
        }

        if(tail1 == tail2){
            return true;
        }

        return false;
    }
    ```



- **单链表中，取出环的起始点**
    ```
    public Node getCycleStart(Node head, int cycleLength) {
        Node first = head;
        Node second = head;

        for(int i = 0; i < cycleLength; i++){
            second = second.next;
        }

        while(first != null && second != null){
            if(first == second){
                return first;
            }
            first = first.next;
            second = second.next;
        }

        return null;
    }
    ```

- **求两个单链表相交的第一个节点**
    思路：分别求出a链表、b链表的长度len1、len2，如果a链表比b链表长，则a提前移动len1-len2距离，然后a、b一起向前移动，直到引用相同。
    ```
    public static Node getFirstCommonNode(Node head1, Node head2) {


        Node tail1 = head1;
        Node tail2 = head2;

        int len1 = 1;
        while(tail1 != null){
            tail1 = tail1.next;
            len1++;
        }

        int len2 = 1;
        while(tail2 != null){
            tail2 = tail2.next;
            len2++;
        }

        if(tail1 != tail2){
            return null;
        }

        Node n1 = head1;
        Node n2 = head2;

        if(len1 > len2){
            int k = len1 - len2;

            while(k != 0){
                n1 = n1.next;
                k--;
            }
        }

        if(len1 < len2){
            int k = len1 - len2;

            while(k != 0){
                n2 = n2.next;
                k--;
            }
        }

        while(n1 != n2){
            n1 = n1.next;
            n2 = n2.next;
        }

        return n1;
    }
    ```

- 已知两个单链表head1和head2各自有序，把它们合并成一个链表依然有序 

    ```
    public static Node mergeSortedList(Node head1, Node head2){
        if(head1 == null){
            return head2;
        }

        if(head2 == null){
            return head1;
        }

        Node mergeHead = null;
        if(head1.val < head2.val){
            mergeHead = head1;
            head1 = head1.next;
            mergeHead.next = null;
        }else{
            mergeHead = head2;
            head2 = head2.next;
            mergeHead.next = null;
        }

        Node mergeCur = mergeHead;
        while(head1 != null && head2 != null){
            if(head1.val < head2.val){
                mergeCur.next = head1;
                head1 = head1.next;
                mergeCur = mergeCur.next;
                mergeCur.next = null;
            }else{
                mergeCur.next = head2;
                head2 = head2.next;
                mergeCur = mergeCur.next;
                mergeCur.next = null;
            }
        }

        if(head1 != null){
            mergeCur.next = head1;
        }

        if(head2 != null){
            mergeCur.next = head2;
        }

        return mergeHead;
    }
    ```
