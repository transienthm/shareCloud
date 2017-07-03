# 写在最前面
导师贪腐出逃美国，两年未归，可怜了我。拿了小米和美团的offer，要被延期，offer失效，工作重新找。把准备过程纪录下来，共勉。

# 一、 二叉树的基础

## 结点定义

    ```
    public class TreeNode{
        int val;
        TreeNode left;
        TreeNode right;

        public TreeNode(int val){
            this.val = val;
        }
    }
    ```
## 二叉树的遍历
### 前序遍历
- 前序遍历，递归法
    ```
    public static void preorderTraversalRec(TreeNode root) {
        if(root == null){
            return;
        }

        System.out.print(root.val + " ");
        preorderTraversalRec(root.left);
        preorderTraversalRec(root.right);
    }
    ```

- 前序遍历，迭代法
    思路：借助一个栈
    ```
    public static void preorderTraversal(TreeNode root) {
        if(null == root){
            return;
        }

        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);

        while(!stack.empty()){
            TreeNode cur = stack.pop();

            System.out.println(cur.val);

            //后入先出，因而先压右结点，再压左结点
            if(null != cur.right){
                stack.push(cur.right);
            }

            if(null != cur.left){
                stack.push(cur.left);
            }

        }
    }

    ```

    - 中序遍历，递归法
    ```
    public static void inorderTraversalRec(TreeNode root) {
        if(null == root){
            return;
        }

        inorderTraversalRec(root.left);
        System.out.print(root.val + " ");
        inorderTraversalRec(root.right);

    }

    ```

    - 中序遍历，迭代法
    ```
    public static void inorderTraversal(TreeNode root) {
        if(null == root){
            return;
        }

        Stack<TreeNode> stack = new Stack<>();
        TreeNode cur = root;

        while(true){
            while(cur != null){
                stack.push(cur);
                cur = cur.left;
            }

            if(stack.empty()){
                break;
            }

            cur = stack.pop();
            System.out.print(cur.val + " ");
            cur = cur.right;
        }

    }
    ```
    - 后序遍历，递归法
    ```
    public static void postorderTraversalRec(TreeNode root) {
        if(null == root){
            return;
        }

        postorderTraversalRec(root.left);
        postorderTraversalRec(root.right);
        System.out.print(root.val + " ");
    }
    ```
    - 后序遍历，迭代法
    ```
    public static void postorderTraversal(TreeNode root){
        if(null == root){
            return;
        }

        Stack<TreeNode> s = new Stack<TreeNode>();
        Stack<TreeNode> output = new Stack<>();

        s.push(root);
        while(!s.empty()){
            TreeNode cur = s.pop();
            output.push(cur);

            if(cur.left != null){
                s.push(cur.left);
            }

            if(cur.right != null){
                s.push(cur.right);
            }
        }

        while(!output.empty()){
            System.out.print(output.pop().val + " ");
        }
    }
    ```
- 分层遍历
    ```
    public static void levelTraversal(TreeNode root) {
        if(null == root){
            return;
        }

        Queue<TreeNode> queue = new LinkedList<>();
        queue.push(root);

        while(!queue.empty()){
            TreeNode cur = queue.removeFirst();
            System.out.print(cur.val + " ");

            if(cur.left != null){
                queue.add(cur.left);
            }

            if(cur.right != null){
                queue.add(cur.right);
            }
        }
    }
    ```

#### 求二叉树结点的个数
- 递归解法 时间复杂度O(n)
    ```
    public static int getNodeNumRec(TreeNode root){
        if(null != root){
            return 0;
        }

        return getNodeNumRec(root.left) + getNodeNumRec(root.right) + 1;
    }
    ```

- 迭代解法 时间复杂度O(n)
    思路：与层级遍历相同，遍历的过程中纪录结点数
    ```
    public static int getNodeNum(TreeNode root){
        if(null != root){
            return 0;
        }

        int count = 1;
        Queue<TreeNode> queue = LinkedList<>();
        queue.add(root);

        while(!queue.empty()){
            TreeNode cur = queue.remove();

            if(cur.left != null){
                queue.add(cur.left);
                count++;
            }

            if(cur.right != null){
                queue.add(cur.right);
                count++;
            }
        }

        return count;
    }
    ```

#### 求二叉树的深度（高度）
- 递归解法 时间复杂度O(n)
    ```
    public static int getDepthRec(TreeNode root) {
        if(null != root){
            return 0;
        }

        int leftDepth = getDepthRec(root.left);
        int rightDepth = getDepthRec(root.right);
        return Math.max(leftDepth, rightDepth) + 1;
    }
    ```

- 迭代解法 时间复杂度O(n)
    ```
    public static int getDepth(TreeNode root){
        if(null != root){
            return 0;
        }

        int depth = 0;
        int curLevelNodes = 1;
        int nextLevelNodes = 0;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);

        while(!queue.empty()){
            TreeNode cur = queue.remove();
            curLevelNodes--;

            if(cur.left != null){
                nextLevelNodes++;
                queue.add(cur.left);
            }

            if(cur.right != null){
                nextLevelNodes++；
                queue.add(cur.right);
            }

            if(curLevelNodes == 0){
                depth++;
                curLevelNodes = nextLevelNodes;
                nextLevelNodes = 0;
            }
        }

        return depth；
    }
    ```

#### 求二叉树第K层的节点个数
- 递归解法
    思路：求以root为根的k层节点数目 等价于 求以root左孩子为根的k-1层（因为少了root那一层）节点数目 加上 以root右孩子为根的k-1层（因为少了root那一层）节点数目
    ```
    public static int getNodeNumKthLevelRec(TreeNode root, int k) {
        if(null != root || k < 0){
            return 0;
        }

        if(k == 1){
            return 1;
        }

        int leftNodeNumKth = getNodeNumKthLevelRec(root.left, k - 1);
        int rightNodeNumKth = getNodeNumKthLevelRec(root.right, k - 1);
        return leftNodeNumKth + rightNodeNumKth;
    }
    ```

- 迭代法
    思路：与求解树深度的解法相同，需要
    ```
    public static int getNodeNumKthLevel(TreeNode root, int k){
        if(root == null || k < 0){
            return 0;
        }

        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);

        int curLevelNodes = 1;
        int nextLevelNodes = 0;

        while(!queue.empty && k > 0){
            TreeNode cur = queue.remove();
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
                curLevelNodes = nextLevelNodes;
                nextLevelNodes = 0;
                k--;
            }
        }

        return curLevelNodes;
    }
    ```
#### 求二叉树中叶子节点的个数
- 迭代法
    ```
    public static int getNodeNumLeaf(TreeNode root) {
        if(root == null){
            return 0;
        }

        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);

        int leafNodeNum = 0;

        while(!queue.empty()){
        	TreeNode cur = queue.remove();

        	if(cur.left != null){
        		queue.add(cur.left);
        	}

        	if(cur.right != null){
        		queue.add(cur.right);
        	}

        	if(cur.left == null && cur.right = null){
        		leafNodeNum++;
        	}
        }
        return leafNodeNum;
    }
    ```

#### 两个二叉树之间的关系
##### 判断两棵二叉树是否相同的树。
- 递归法
	```
    public static boolean isSameRec(TreeNode r1, TreeNode r2) {
		if(r1 == null && r2 == null){
			return true;
		}

		if(r1 == null || r2 == null){
			return false;
		}

		if(r1.val != r2.val){
			return false;
		}

		boolean leftRes = isSameRec(r1.left, r2.left);
		boolean rightRes = isSameRec(r1.right, r2.right);

		return leftRes && rightRes;
	}
	```
- 迭代法
	思路：遍历一遍，比对即可
	```
    public static boolean isSame(TreeNode r1, TreeNode r2) {
		if(r1 == null && r2 == null){
			return true;
		}

		if(r1 == null || r2 == null){
			return false;
		}

		Stack<TreeNode> s1 = new Stack<>();
		Stack<TreeNode> s2 = new Stack<>();

		s1.push(r1);
		s2.push(r2);

		while(!s1.empty() && !s2.empty()){
			TreeNode n1 = s1.pop();
			TreeNode n2 = s2.pop();

			if(n1 == null && n2 == null){
				continue;
			}else if(n1 != null && n2 != null && n1.val == n2.val){
				s1.push(n1.right);
				s1.push(n1.left);
				s2.push(n2.right);
				s2.push(n2.left);
			}else{
				return false;
			}
		}
		return true;
 	}
	```

#### 判断二叉树是不是平衡二叉树
- 递归解法
	思路：
	(1）如果二叉树为空，返回真
    (2）如果二叉树不为空，如果左子树和右子树都是AVL树并且左子树和右子树高度相差不大于1，返回真，其他返回假
     ```
     public static boolean isAVLRec(TreeNode root) {
     	if(root == null){
     		return true;
     	}

     	if(Math.abs(getDepthRec(root.left) - getDepthRec(root.right)) > 1){
     		return false;
     	}

     	return isAVLRec(root.left) && isAVLRec(root.right);
     }
     ```
#### 树的镜像
##### 判断两个树是否互相镜像
 	```
    public static boolean isMirrorRec(TreeNode r1, TreeNode r2){
    	if(r1 == null && r2 == null){
    		return true;
    	}

    	if(r1 == null || r2 == null){
    		return false;
    	}

    	if(r1.val != r2.val){
    		return false;
    	}

    	return isMirrorRec(r1.left, r2.right) && isMirrorRec(r1.right, r2.left);
    }
 	```
##### 求树的镜像
- 递归解法
	 （1）如果二叉树为空，返回空
     （2）如果二叉树不为空，求左子树和右子树的镜像，然后交换左子树和右子树
     1. 破坏原来的树

     ```
         public static TreeNode mirrorRec(TreeNode root) {
     	if(root == null){
     		return null;
     	}

     	TreeNode left = mirrorRec(root.left);
     	TreeNode right = mirrorRec(root.right);

     	root.left = right;
     	root.right = left;
     	return root;
     }
     ```
     2.保存原来的树
     ```
     public static TreeNode mirrorCopyRec(TreeNode root) {
     	if(root == null){
     		return null;
     	}

     	TreeNode newRoot = new TreeNode(root.val);
     	newRoot.left = mirrorCopyRec(root.right);
     	newRoot.right = mirrorCopyRec(root.left);

     	return newRoot;

     }
     ```
- 迭代解法
    1. 破坏原来的树
    ```
    public static void mirror(TreeNode root) {
        if(root == null){
            return;
        }

        Stack<TreeNode> stack = new Stack<TreeNode>();
        stack.push(root);

        while(!stack.empty()){
            TreeNode cur = stack.pop();

            TreeNode tmp = cur.left;
            cur.left = cur.right;
            cur.right = tmp;

            if(cur.left != null){
                stack.push(cur.left);
            }

            if(cur.right != null){
                stack.push(cur.right);
            }
        }
    }
    ```

    2. 不能破坏原来的树，返回一个新的镜像树
    ```
    public static TreeNode mirrorCopy(TreeNode root){
        if(root == null){
            return null;
        }

        Stack<TreeNode> stack = new Stack<>();
        Stack<TreeNode> newStack = new Stack<>();
        stack.push(root);
        TreeNode newRoot = new TreeNode(root.val);
        newStack.push(newRoot);

        while(!stack.empty()){
            TreeNode cur = stack.pop();
            TreeNode newCur = newStack.pop();

            if(cur.left != null){
                stack.push(cur.left);
                newCur.right = new TreeNode(cur.left.val);
                newStack.push(newCur.right);
            }

            if(cur.right != null){
                stack.push(cur.right);
                newCur.left = new TreeNode(cur.right.val);
                newStack.push(newCur.left);
            }
        }
        return newRoot;
    }
    ```
##### 求二叉树中两个节点的最低公共祖先节点
- 递归法
思路：1. 如果其中一个结点为根结点，则返回根结点
     2. 如果一个左子树找到，一个在右子树找到，则说明root是唯一可能的最低公共祖先
     3. 其他情况是要不然在左子树要不然在右子树
    ```
    public static TreeNode getLastCommonParentRec(TreeNode root, TreeNode n1, TreeNode n2) {
        if (root == null || n1 == null || n2 == null) {
            return null;
        }

        if(root.equals(n1) || root.equals(n2)){
            return root;
        }

        TreeNode commonInLeft = getLastCommonParentRec(root.left, n1, n2);
        TreeNode commonInRight = getLastCommonParentRec(root.right, n1, n2);
        if(commonInLeft != null && commonInRight != null){
            return root;
        }

        if(commonInLeft != null){
            return commonInLeft;
        }

        if(commonInRight != null){
            return commonInRight;
        }
    }
    ```
- 迭代法
    ```
    public static TreeNode getLastCommonParent(TreeNode root, TreeNode n1, TreeNode n2){
        if(root == null || n1 == null || n2 == null){
            return null;
        }

        List<TreeNode> list1= new ArrayList<>();
        List<TreeNode> list2 = new ArrayList<>();

        boolean res1 = getNodePath(root, n1, list1);
        boolean res2 = getNodePath(root, n2, list2);

        if(!res1 || !res2){
            return null;
        }

        Iterator<TreeNode> iter1 = list1.iterator();
        Iterator<TreeNode> iter2 = list2.iterator();
        TreeNode last = null;

        while(iter1.hasNext() && iter2.hasNext()){
            TreeNode tmp1 = iter1.next();
            TreeNode tmp2 = iter2.next();

            if(tmp1 == tmp2){
                last = tmp1;
            }else{
                break;
            }
        }
        return last;
    }

    private static boolean getNodePath(TreeNode root, TreeNode n, List<TreeNode> path){
        if(root == null){
            return false;
        }

        path.add(root);
        if(root == n){
            return true;
        }

        boolean found = false;
        found = getNodePath(root.left, n, path);

        if(!found){
            found = getNodePath(root.right, n, path);
        }

        if(!found){
            path.remove(root);
        }

        return found;
    }
    ```

    由中序、后序构建树
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

> 本章参考http://blog.csdn.net/fightforyourdream/article/details/16843303
