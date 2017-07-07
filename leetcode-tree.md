##　由遍历结果反求树
  分析：递归分治，第一层需要找到相应的遍历结果，对数组来说，问题转化为找下标问题
  对前序、中序遍历结果来说
  前序：[root,[左],[右]]
  中序:[[左],root,[右]]
  因此，中序中root的下标可求，为inorderPos
    对每一层来说，左子树的长度为leftLen = inorderPos，右子树的长度为rightLen = inorder.length - 1 - leftLen，
    左子树前序为preorder[1 至 leftLen]，中序为inorder[0 至 leftLen - 1]，可以使用System.arraycopy(preorder, 1, leftPre, 0, leftLen),
    System.arraycopy(inorder, 0, leftInorder, 0, leftLen);
    右子树前序为preorder[leftLen + 1 至 preorder.length - 1]，中序为inorder[leftLen + 1 至 inorder.lenth - 1]，可以使用System.arraycopy(preorder, leftLen + 1, rightPre, 0, rightLen)，System.arraycopy(inorder, leftLen + 1, rightInorder, 0, rightLen);

  对中序、后序来说，
  中序:[[左],root,[右]]
  后序：[[左],[右],root]
### Leetcode 105 由前序、中序构建树
```
public TreeNode buildTree(int[] preorder, int[] inorder) {
    if(preorder.length == 0 || inorder.length == 0 || preorder.length != inorder.length){
        return null;
    }

    int len = preorder.length;
    TreeNode root = new TreeNode(preorder[0]);
    int inorderPos = 0;
    for(; inorderPos < inorder.length; inorderPos++){
        if(inorder[inorderPos] == root.val){
            break;
        }
    }

    int leftLen = inorderPos;
    int rightLen = len - inorderPos - 1;

    int[] leftPre = new int[leftLen];
    int[] leftInorder = new int[leftLen];

    int[] rightPre = new int[rightLen];
    int[] rightInorder = new int[rightLen];

    for(int i = 0; i < leftLen; i++){
        leftPre[i] = preorder[i + 1];
        leftInorder[i] = inorder[i];
    }

    for(int i = 0; i < rightLen; i++){
        rightPre[i] = preorder[leftLen + 1 + i];
        rightInorder[i] = inorder[leftLen + 1 + i];
    }

    TreeNode left = buildTree(leftPre, leftInorder);
    TreeNode right = buildTree(rightPre, rightInorder);

    root.left = left;
    root.right = right;

    return root;
}
```

### Leetcode 106 由中序、后序构建树
```
public TreeNode buildTree(int[] inorder, int[] postorder) {
    if(inorder.length == 0 || postorder.length == 0){
        return null;
    }

    int len = postorder.length;
    TreeNode root = new TreeNode(postorder[len - 1]);

    int inorderPos = 0;

    for(; inorderPos < len; inorderPos++){
        if(inorder[inorderPos] == root.val){
            break;
        }
    }

    int leftLength = inorderPos;
    int rightLength = len - inorderPos - 1;

    int[] leftInorder = new int[leftLength];
    int[] leftPost = new int[leftLength];

    int[] rightInorder = new int[rightLength];
    int[] rightPost = new int[rightLength];

    for(int i = 0; i < leftLength; i++){
        leftInorder[i] = inorder[i];
        leftPost[i] = postorder[i];
    }

    for(int i = 0; i < rightLength; i++){
        rightInorder[i] = inorder[inorderPos + 1 + i];
        rightPost[i] = postorder[leftLength + i];
    }

    TreeNode left = buildTree(leftInorder, leftPost);
    TreeNode right = buildTree(rightInorder, rightPost);

    root.left = left;
    root.right = right;

    return root;

}
```

##　leetcode 124
  思路：
  分治：对于每一个结点来说，需要计算，当前值＋左结点＋右结点 与 最大值的比较，同时，左结点与右结点的值通过递归得到，因此，递归的返回值应是一条路径的和
```
public class Solution{
  int maxNum = Integer.MIN_VALUE;
  public int maxPathSum(TreeNode root){

      if(root == null){
        return 0;
      }
      count(root);
      return maxNum;
  }

  public int count(TreeNode root){
    int lval = Integer.MIN_VALUE, rval = Integer.MIN_VALUE;
    int val = root.val;

    if(root.left != null){
      lval = count(root.left);
    }

    if(root.right != null){
      rval = count(root.right);
    }

    val = val + Math.max(lval, 0) + Math.max(rval, 0);

    if(val > maxNum){
      maxNum = val;
    }

    return root.val + Math.max(Math.max(lval, rval), 0);
  }  
}
```

## 最小深度与最大深度
### leetcode 111 最小深度
- 递归法：
思路：
  退出条件
  1. root == null,直接返回0，但是！如果root.left或root.right其中一个为null，不能退出递归，两种解决方法
    方法一：使用新的递归函数规避
    ```
    public int minDepth(TreeNode root){
      if(root == null){
        return 0;
      }
      return getMin(root);
    }
    public int getMin(TreeNode root){
      //规避左右子树某一个为null
      if(root == null){
        return Integer.MAX_VALUE;//排除此条路径
      }

      if(root.left == null && root.right == null){
        return 1;
      }
      int left = Integer.MAX_VALUE;
      int right = Integer.MAX_VALUE;

      if(root.left != null){
        left = getMin(root.left);
      }
      if(root.right != null){
        right = getMin(root.right);
      }

      return Math.min(left, right) + 1;
    }
    ```
    方法二：给当前方法打补丁
    ```
    public int minDepth(TreeNode root) {
          if(root == null){
            return 0;
          }

          if(root.left == null && root.right == null){
            return 1;
          }

          if(root.left == null){
              return minDepth(root.right) + 1;
          }else if(root.right == null){
              return minDepth(root.left) + 1;
          }else{
              return Math.min(minDepth(root.left), minDepth(root.right)) + 1;
          }
    }  
    ```
  2. root.left == null && root.right == null 说明为叶子结点，返回1
  3. 当前层数加 左右子树的最小深度
- 迭代法
思路：层级遍历，一旦在当前层发现叶子结点，返回层数
```
  public int minDepth(TreeNode root){
    if(root == null){
      return 0;
    }
    if(root.left == null && root.right == null){
      return 1;
    }

    int depth = 0;
    int curLevelNodes = 1;
    int nextLevelNodes = 0;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root);

    while(!queue.isEmpty()){
      TreeNode cur = queue.poll();
      curLevelNodes--;

      if(cur.left == null && cur.right == null){
        return depth + 1;
      }

      if(cur.left != null){
        queue.add(cur.left);
        nextLevelNodes++;
      }

      if(cur.right != null){
        queue.add(cur.right);
        nextLevelNodes++;
      }

      if(curLevelNodes == 0){
        depth++;
        curLevelNodes = nextLevelNodes;
        nextLevelNodes = 0;
      }
    }

    return depth;
  }
```

### leetcode 104 最大深度
- 递归法
思路：递归时逻辑是一贯的
```
  public int getMaxDepth(TreeNode root){
    if(root == null){
      return 0;
    }

    return Math.max(getMaxDepth(root.left), getMaxDepth(root.right)) + 1;
  }
```
- 迭代法
思路：层级遍历求最大深度

```
  public int maxDepth(TreeNode root){
    if(root == null){
      return 0;
    }
    if(root.left == null && root.right == null){
      return 1;
    }

    int depth = 0;
    int curLevelNodes = 1;
    int nextLevelNodes = 0;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root);

    while(!queue.isEmpty()){
      TreeNode cur = queue.poll();
      curLevelNodes--;

      if(cur.left != null){
        queue.add(cur.left);
        nextLevelNodes++;
      }

      if(cur.right != null){
        queue.add(cur.right);
        nextLevelNodes++;
      }

      if(curLevelNodes == 0){
        depth++;
        curLevelNodes = nextLevelNodes;
        nextLevelNodes = 0;
      }
    }

    return depth;
  }
```

## leetcode 110 树是否平衡
树平衡要求对所有结点来说，其左右子树的深度差不超过1
```
public boolean isBalanced(TreeNode root){
  if(root == null){
    return true;
  }

  int leftDepth = getMaxDepth(root.left);
  int rightDepth = getMaxDepth(root.right);

  if(Math.abs(leftDepth - rightDepth) > 1){
    return false;
  }

  return isBalanced(root.left) && isBalanced(root.right);
}
```

## leetcode 100 判断两棵树是否相同
分析：树的相同，首先结构相同，其次结点值相同
两种判断结构是否相同的写法，逻辑一样
方法一
```
  public boolean isSame(TreeNode r1, TreeNode r2){
    if(r1 == null && r2 == null){
      return true;
    }
    if(r1 == null || r2 == null){
      return false;
    }
    //else 结构相同
  }
```
方法二
```
  public boolean isSame(TreeNode r1, TreeNode r2){
    if(r1 == null){
      return r2 == null;
    }

    if(r2 == null){
      return false;
    }
    //else 结构相同
  }
```
完整逻辑
```
  public boolean isSame(TreeNode r1, TreeNode r2){
    if(r1 == null){
      return r2 == null;
    }

    if(r2 == null){
      return false;
    }

    return r1.val == r2.val && isSame(r1.left, r2.left) && isSame(r1.right, r2.right);
  }
```

## leetcode 101 判断对称
左右子树，结构相同，对称位置值相同
```
public boolean isSymmetric(TreeNode root) {
  if(root == null){
    return true;
  }
  return help(root.left, root.right);
}

public boolean help(TreeNode p, TreeNode q){
  if(p == null){
    return q == null;
  }
  if(q == null){
    return false;
  }

  return p.val == q.val && help(p.left, q.right) && help(p.right, q.left);
}
```

## leetcode 98 判断二叉搜索树
- 迭代法
思路：中序遍历 前一个结点值小于后面的结点值
```
public boolean isValidBST(TreeNode root){
  if(root == null){
    return true;
  }

  Stack<TreeNode> stack = new Stack<>();
  TreeNode cur = root;

  TreeNode preCur = null;
  while(true){
    while(cur != null){
        stack.push(cur);
        cur = cur.left;
      }

      if(stack.isEmpty()){
        break;
      }

      cur = stack.pop();

      if(preCur != null){
        if(preCur.val >= cur.val){
          return false;
        }
      }
      preCur = cur;
      cur = cur.right;
  }  
  return true;
}
```
- 递归法
思路：同样是中序遍历
思考 pre结点在哪赋值，赋值前如何处理
```
    TreeNode pre = null;
    public boolean isValidBST(TreeNode root) {
        if(root == null){
          return true;
        }

        if(root.left == null && root.right == null){
            return true;
        }

        return help(root);
    }

    public boolean help(TreeNode root){
        if(root == null){
            return true;
        }

        boolean left = help(root.left);
        if(pre != null && pre.val >= root.val){
            return false;
        }
        pre = root;
        boolean right = help(root.right);
        return left && right;
    }
```

## 链表与树
### leetcode 114 二叉树转链表
思路：断开每一个结点，从用一个指针递归地向下指，每次都只更新右结点，递归顺序为先左子树，后右子树
```
  TreeNode pointer = new TreeNode(-1);
  public void flatten(TreeNode root){
    if(root == null){
      return;
    }
    TreeNode left = root.left;
    TreeNode right = root.right;
    root.left = null;
    root.right = null;
    pointer.right = root;
    pointer = root;

    flatten(root.left);
    flatten(root.right);
  }
```

### 链表转二叉树
O(nlogn)解法
```
public TreeNode sortedListToBST(ListNode head){
       if(head == null){
         return null;
       }
       if(head.next == null){
           return new TreeNode(head.val);
       }
       int length = 0;
       ListNode cur = head;
       while(cur != null){
         cur = cur.next;
         length++;
       }
       return help(head, length);
 }
   public TreeNode help(ListNode head, int length){
     if(length == 0){
       return null;
     }

     ListNode now = head;
     for(int i = 0; i < (length - 1) >> 1; i++){
       now = now.next;
     }
     TreeNode root = new TreeNode(now.val);

     TreeNode left = help(head, (length - 1) >> 1);
     TreeNode right = help(now.next, length >> 1);

     root.left = left;
     root.right = right;

     return root;
   }
```
O(n)解法
将链表先转成数组

### leetcode 108 数组转平衡二叉树
```
public TreeNode sortedArrayToBST(int[] nums) {
        if(nums.length == 0){
            return null;
        }

        if(nums.length == 1){
            return new TreeNode(nums[0]);
        }

        int length = nums.length;
        int now = nums[(length - 1) >> 1];
        TreeNode root = new TreeNode(now);
        int leftLen = (length - 1) >> 1;
        int rightLen = length >> 1;

        int[] leftArr = new int[leftLen];
        int[] rightArr = new int[rightLen];
        System.arraycopy(nums, 0, leftArr, 0, leftLen);
        System.arraycopy(nums, leftLen + 1, rightArr, 0, rightLen);

        root.left = sortedArrayToBST(leftArr);
        root.right = sortedArrayToBST(rightArr);

        return root;
    }
```
