
 
2. Add Two Numbers
```
	public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
		int carry = 0;
		ListNode n = new ListNode(0);
		ListNode t = n;
		ListNode p1 = l1;
		ListNode p2 = l2;
		while(p1 != null || p2 != null){
			if(p1 != null){
				carry += p1.val;
				p1 = p1.next;
			}

			if(p2 != null){
				carry += p2.val;
				p2 = p2.next;
			}

			t.next = new ListNode(carry % 10);
			t = t.next;
			carry /= 10;
		}

		if(carry == 1){
			t.next = new ListNode(1);
		}

		return n.next;
	}
```

