



# LeetCode-剑指offer第二版



## 数组

#### [剑指 Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

解题思路:

这里最通用的方法是使用两个for循环去暴力求解，但是复杂度会很高，常规解法会达到$O(N^2)$，由于每一列按照由上而下递增的顺序排序，每一行按照由左到右递增的顺序排序，所有优化后的解法如下，即设定好边界条件后，可以从最右上角的位置开始往下走，如果当前位置的值比target要小，就往继续下走，如果比target要大，那就向左走，如果在没有到达边界之前可以找到与target相等的数，即返回True，否则返回False

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
























## 二叉树

## 字符串

## 动态规划/搜索

### [剑指 Offer 12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

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




