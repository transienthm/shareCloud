

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

415. Add Strings
```
public String addStrings(String num1, String num2) {
    if(num1 == null && num2 == null){
        return null;
    }

    int carry = 0;
    int len1 = num1.length();
    int len2 = num2.length();
    int i = len1 - 1;
    int j = len2 - 1;
    StringBuilder sb = new StringBuilder();
    while(i >= 0 && j >= 0){
        int val1 = convertCharToInt(num1.charAt(i));
        int val2 = convertCharToInt(num2.charAt(j));
        int sum = val1 + val2 + carry;
        sb.insert(0, sum % 10);
        carry = sum / 10;
        i--;
        j--;
    }

    while(i >= 0){
        int val1 = convertCharToInt(num1.charAt(i));
        int sum = val1 + carry;
        sb.insert(0, sum % 10);
        carry = sum / 10;
        i--;
    }

    while(j >= 0){
        int val2 = convertCharToInt(num2.charAt(j));
        int sum = val2 + carry;
        sb.insert(0, sum % 10);
        carry = sum / 10;
        j--;
    }

    if(carry != 0){
        sb.insert(0, carry);
    }

    return sb.toString();
}

public int convertCharToInt(char c){
    return c - '0';
}
```
