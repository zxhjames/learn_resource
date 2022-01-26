# 剑指offer第二版

## **数组**

### **[剑指 Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)**

解题思路:

这里最通用的方法是使用两个for循环去暴力求解，但是复杂度会很高，常规解法会达到O(N^2)，由于每一列按照由上而下递增的顺序排序，每一行按照由左到右递增的顺序排序，所有优化后的解法如下，即设定好边界条件后，可以从最右上角的位置开始往下走，如果当前位置的值比target要小，就往继续下走，如果比target要大，那就向左走，如果在没有到达边界之前可以找到与target相等的数，即返回True，否则返回False

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
       int rows = matrix.length;
       if (rows == 0) {
           return false;
       }
       int cols = matrix[0].length;
       if (cols == 0) {
           return false;
       }
       int r = 0;
       int c = cols - 1;
       while (r < rows && c >= 0) {
           if (matrix[r][c] < target) {
                r++;
           }else if (matrix[r][c] > target) {
                c--;
           }else{
               return true;
           }
       }
        return false;
    }
}
```

## **二叉树**

### **[剑指 Offer 07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)**

解题思路:

这道题可以通过递归来做，由于前序遍历的遍历顺序是“根左右”，中序遍历的顺序是“左根右”，那我们输入的两个前序和中序数组的整体表示情况就为**[   根 | 左 | 右 ]**和**[ 左 | 根 | 右 ]**，并且“根，左，右”的每一段序列长度都是一样长的，我们以前序遍历数组(*preorder*)的第一个位置值**"根"**为基准，然后在中序遍历数组(*inorder*)中寻找"根"位置的索引号，假设我们在inorder中找到位置的索引为index,那么很容易判断**在当前根结点**下的左子树有index个结点，右子树有preorder.length - index -1个结点，那么对于新的preorder而言，左子树的范围就是[1,i+1],右子树的范围是[i+1,preorder.length]，对于新的inorder而言，左子树的范围是[0,i],右子树是[i+1,inorder.length],这样就可以递归的去创建子树，代码如下

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        if (preorder.length == 0 || inorder.length == 0) {
            return null;
        }
        TreeNode root = new TreeNode(preorder[0]);
        for (int i = 0;i<inorder.length;++i) {
            if (preorder[0] == inorder[i]) {
                root.left=buildTree(Arrays.copyOfRange(preorder,1,i+1),Arrays.copyOfRange(inorder,0,i));
                root.right=buildTree(Arrays.copyOfRange(preorder,i+1,preorder.length),Arrays.copyOfRange(inorder,i+1,inorder.length));
                break;
            }
        }
        return root;
    }
}
```

### [剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

[](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

解题思路:

题目旨在判断两颗二叉树A，B，判断B是否是A的子结构，子结构意思就是二叉树B存在着与二叉树A同形的片段，我们假设都从两棵树的根结点出发，顺着左右两个字数搜索下去，如果二者当前结点都为空，那就都返回false，否则就尝试从A的左子树第一个根节点或者是右子树第一个根节点搜索搜索，在这里用于判断的函数我们使用一个递归来解决，需要满足的就是每一次匹配的结点值都要相同，当碰到当前A结点为空B不为空时，很显然已经不满足子数条件了，返回false，如果当前B结点为空A不为空时，空结点仍然属于子树，返回true

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
       public boolean isSubStructure(TreeNode A, TreeNode B) {
        if(A == null || B == null) return false;
        return dfs(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B);
    }
    public boolean dfs(TreeNode A, TreeNode B){
        if(B == null) return true;
        if(A == null) return false;
        return A.val == B.val && dfs(A.left, B.left) && dfs(A.right, B.right);
    }
}
```

### 剑指 Offer 36. 二叉搜索树与双向链表

[力扣](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

解题思路:

题目给出需要输入一棵二叉搜索树，将其转换为一个排序的循环双向链表，不能创建新的结点，只能调整树中的节点指针指向，链表中的每一个结点都有一个前驱指针和后继指针，在这里，我的方法是使用一个链表来存储中序遍历之后的结果，然后在这个结果里面，依次去调整链表里的前后指针

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val,Node _left,Node _right) {
        val = _val;
        left = _left;
        right = _right;
    }
};
*/
class Solution {
    public Node treeToDoublyList(Node root) {
        if (root == null) {
            return root;
        }

        // 左根右 中序遍历
        List<Node> nodes = new ArrayList<>();
        recursive(nodes,root);
        
        if (nodes.size() < 2) {
            nodes.get(0).left = nodes.get(0);
            nodes.get(0).right = nodes.get(0);
            return nodes.get(0);
        }
        // 依次转换
        for (int i = 0;i < nodes.size(); ++i){
            if(i == 0) {
                nodes.get(i).left = nodes.get(nodes.size() - 1);
                nodes.get(i).right = nodes.get(i + 1);
                continue;
            }
            if(i == nodes.size() - 1) {
                nodes.get(i).left = nodes.get(i - 1);
                nodes.get(i).right = nodes.get(0);
                continue;
            }else{
                nodes.get(i).left = nodes.get(i - 1);
                nodes.get(i).right = nodes.get(i + 1);
            }
        }
        return nodes.get(0);
    }

    public void recursive(List<Node> nodes,Node root) {
        if (root == null) {
            return;
        }
        recursive(nodes,root.left);
        nodes.add(root);
        recursive(nodes,root.right);
    }
}
```

### 98. 验证二叉搜索树

[力扣](https://leetcode-cn.com/problems/validate-binary-search-tree/)

题目解析:

题目给了一个二叉树让我们判断这棵树是严格遵循二叉搜索树的规范，所谓二叉搜索树，要满足以下三个点:1.节点的左子树必须包含小于当前结点的树，2.所有右子树必须包含大于当前结点的树，3.所有的左子树和右子树自身也必须是二叉搜索树

![src=http---www.icode9.com-i-ll-?i=img_convert-a74fc454bb8afbe43b950dd1ecf5047c.gif&refer=http---www.icode9.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b02d551b-b7fa-4375-835d-a5b9de0b3272/srchttp---www.icode9.com-i-ll-iimg_convert-a74fc454bb8afbe43b950dd1ecf5047c.gifreferhttp---www.icode9.comapp2002sizef999910000qa80n0g0nfmtjpeg.gif)

第一次做这题时，我连概念都没捋一遍就直接上手写递归代码了，错误的以为只要左右结点满足边界要求即可，却忘记了上面的第三个条件，所以真正的解决方法是首先要确定好一个很大的边界值，然后在递归的过程中逐渐缩小这个边界，如果出现不满足要求(大于或小于边界值)的结点，直接返回false，如果出现空结点，这种情况是可以的，返回true

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isValidBST(TreeNode root) {
        return recrusive(root,Long.MIN_VALUE,Long.MAX_VALUE);
    }

    public boolean recrusive(TreeNode root,long _min,long _max) {
        if (root == null) {
            return true;
        }
        if (root.val <= _min || root.val >= _max) {
            return false;
        }
        return recrusive(root.left,_min,root.val) && recrusive(root.right,root.val,_max);
    }
}
```

### 剑指 Offer 32 - I. 从上到下打印二叉树

[力扣](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

解题思路:

很基础的一个二叉树题，其实就是让我们写出层序遍历的算法,，使用一个队列去保存每个结点值，队列的头部输出节点值，队列的尾部输入节点的下一级子节点值

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] levelOrder(TreeNode root) {
        if (root == null) {
            return new int[]{};
        }

        if (root.left == null && root.right == null) {
            return new int[]{root.val};
        }
        List<Integer> res = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            res.add(node.val);
            if (node.left != null) queue.add(node.left);
            if (node.right != null) queue.add(node.right);
        }
        int[] result = new int[res.size()];
        for (int i =0;i<res.size();++i) {
            result[i] = res.get(i);
        }
        return result;
    }
}
```

### 剑指 Offer 32 - III. 从上到下打印二叉树 III

[力扣](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

结题思路:

这个题目与上题不同，需要我们按照之字型顺序打印二叉树，即奇数行按照从左到右遍历，偶数行与之相反，同时每行遍历的值要单独存在一个子列表里，在这里，由于每一轮遍历都要交换遍历方向，我们可以使用双向链表作为字列表存储使用，如果遇到奇数行，就要转为插入链表尾部的方向，否之则反，在这里内部我们使用一个循环去判断每一轮遍历队列里的元素是否清空，由于我们是从queue.size()开始，一直递减到0，所以不管整个过程中队列怎么变化，遍历的数量还是和初始的队列长度相同，所以一定能保证所有结点都能遍历完，最后只要某个时刻队列是空的，即程序停止

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    // 建立一个队列和一个双端列表
    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        if(root != null) queue.add(root);
        while(!queue.isEmpty()) {
            LinkedList<Integer> tmp = new LinkedList<>();
            // 每一层的结点数是固定的，在固定的范围内加入结点就行
            for(int i = queue.size(); i > 0; i--) {
                TreeNode node = queue.poll();
                if(res.size() % 2 == 0) tmp.addLast(node.val); // 偶数层 -> 转为插入链表头部
                else tmp.addFirst(node.val); // 奇数层 -> 转为插入链表尾部
                if(node.left != null) queue.add(node.left);
                if(node.right != null) queue.add(node.right);
            }
            res.add(tmp);
        }
        return res;
    }

}
```

## **字符串**

## **动态规划/搜索**

### **[剑指 Offer 12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)**

解题思路:

一道需要用到递归的搜索题，目的是在矩阵中找出一条符合条件(即由字符组成起来的单词等于word)的路径，在这里不仅要考虑边界条件，还要记录每个格子是否走过，**在这里要注意题目给出的路径可能不只有一条，所以从每一个格子出发去搜索都有可能出现符合条件的答案** ，在这里使用DFS去做，代码如下

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        if (board == null || board.length == 0 || board[0].length == 0) {
            return false;
        }
        // 单词专为字符数组
        char[] arr = word.toCharArray();
        int size = arr.length;
        // 防止有多个结果，从每一个格子开始搜索
        for (int i=0;i<board.length;++i) {
            for (int j=0;j<board[0].length;++j) {
                if (dfs(board,arr,i,j,0)) {
                    return true;
                }
            }
        }
        return false;
    }

   public boolean dfs(char[][] board,char[] word,int i,int j,int index){
        // 这里进行边界判断和剪枝(如果出现当前矩阵位置字符与结果字符数组当前索引的字符不一致情况，直接返回false)
        if(i<0 || i>=board.length || j<0 || j>=board[0].length || board[i][j]!=word[index]){
            return false;
        }

        // 如果达到了结果字符数组的长度 就返回
        if(index==word.length-1){
            return true;
        }

        // 初始化 标记已经走过
        board[i][j]='\0';
        // 往四个方向进行搜索
        boolean res=dfs(board,word,i+1,j,index+1) || dfs(board,word,i-1,j,index+1)
                    || dfs(board,word,i,j+1,index+1) || dfs(board,word,i,j-1,index+1);
        board[i][j]=word[index];
        return res;
    }
}
```

### **[剑指 Offer 13. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)**

解题思路:

搜索问题，在这里我的方法是首先对方格数组做一个转换，即满足题设条件的位置标注为0，代表当前位置可走，否则标为1，代表不可走，这样整个方格数组都会转化为一个非0即1的地图，我们只需要从(0,0)这个位置开始搜索，搜索到可以走的最大长度即可，在这里的搜索函数最后为什么我们不把(i,j)位置的值修改回来，这里有别于上一道题目**矩阵中的路径**，因为机器人不需要从地图中的每个位置开始，只需要一次遍历即可，所以不能修改回来，否则就会重复走了

```java
class Solution {
    public int movingCount(int m, int n, int k) {
        // 生成数组
        int[][] map = new int[m][n];
        // 首先将数组转化为地图 “0” 代表该位置可以走，“1”代表不能走
        for (int i=0;i<m;++i){
            for(int j=0;j<n;++j){
                // 计算索引的和
                int sum = i / 10 + i % 10 + j / 10 + j % 10;
                if (sum <= k){
                    map[i][j] = 0;
                }else{
                    map[i][j] = 1;
                }
            }
        }
        // dfs
        return dfs(map,0,0);
    }

    public int dfs(int[][] map,int i,int j) {
        // 边界条件即返回
        if (i < 0 || i >= map.length || j < 0 || j >= map[0].length || map[i][j]!=0) {
            return 0;
        }
        // 修改当前位置不可走
        map[i][j] = -1;
        // 返回四个位置进行搜索后求和的结果
        return dfs(map,i+1,j) + dfs(map,i-1,j) + dfs(map,i,j+1) + dfs(map,i,j-1) + 1;
    }
}
```

## 字典树

### LeetCode 221.设计一个添加与搜索单词的数据结构

[力扣](https://leetcode-cn.com/problems/design-add-and-search-words-data-structure/)

超时的解法:

```java
class WordDictionary {

    // 这个代码超时了
    List<String> words = new ArrayList<>();
    public WordDictionary() {
        words = new ArrayList<>();
    }
    
    public void addWord(String word) {
        words.add(word);
    }
    
    public boolean search(String word) {
        int size = 0;
        char[] arr = word.toCharArray();
        List<Integer> index = new ArrayList<>();
        if (word.indexOf(".") != -1) {
            size = word.length();
            for (int i = 0;i<arr.length;++i){
                if (arr[i] == '.') {
                    index.add(i);
                }
            }
            for (int i =0;i<words.size();++i) {
                String tmp = words.get(i);
                if (tmp.length() == size) {
                    boolean f = true;
                    for (int k=0;k<size;++k) {
                        if (index.indexOf(k) == -1 && tmp.charAt(k) != arr[k]) {
                            f =  false;
                            break;
                        } 
                    }
                    if (f) {
                        return true;
                    }
                }
            }
        }else{
            return words.indexOf(word) == -1 ? false : true;
        }
        return false;
    }
}

/**
 * Your WordDictionary object will be instantiated and called as such:
 * WordDictionary obj = new WordDictionary();
 * obj.addWord(word);
 * boolean param_2 = obj.search(word);
 */
```

正确的解决思路(字典树)

本文需要我们设计一个数据结构，能够支持添加新单词和寻找字符换是否与任何先前添加的字符串匹配

```java
class WordDictionary {
    private Trie root;

    public WordDictionary() {
        root = new Trie();
    }
    
    public void addWord(String word) {
        root.insert(word);
    }
    
    public boolean search(String word) {
        return dfs(word, 0, root);
    }

    private boolean dfs(String word, int index, Trie node) {
        if (index == word.length()) {
            return node.isEnd();
        }
        char ch = word.charAt(index);
        if (Character.isLetter(ch)) {
            int childIndex = ch - 'a';
            Trie child = node.getChildren()[childIndex];
            if (child != null && dfs(word, index + 1, child)) {
                return true;
            }
        } else {
            for (int i = 0; i < 26; i++) {
                Trie child = node.getChildren()[i];
                if (child != null && dfs(word, index + 1, child)) {
                    return true;
                }
            }
        }
        return false;
    }
}

class Trie {
    // 在本题中，每一个字典树后面都有26个字数，代表26个字母
    private Trie[] children;
    // 在本题中，判断是够构成一个单词
    private boolean isEnd;

    public Trie() {
        children = new Trie[26];
        isEnd = false;
    }

    // 在这里插入一个单词，
    public void insert(String word) {
        Trie node = this;
        for (int i = 0; i < word.length(); i++) {
            char ch = word.charAt(i);
            int index = ch - 'a';
            if (node.children[index] == null) {
                node.children[index] = new Trie();
            }
            node = node.children[index];
        }
        node.isEnd = true;
    }

    public Trie[] getChildren() {
        return children;
    }

    public boolean isEnd() {
        return isEnd;
    }
}

/**
 * Your WordDictionary object will be instantiated and called as such:
 * WordDictionary obj = new WordDictionary();
 * obj.addWord(word);
 * boolean param_2 = obj.search(word);
 */
```
