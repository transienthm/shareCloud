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
