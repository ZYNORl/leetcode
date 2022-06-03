### 2022-6-1

- [473. 火柴拼正方形](https://leetcode.cn/problems/matchsticks-to-square/)

  - 回溯法

    - 代码

      ```java
      class Solution {
          public boolean makesquare(int[] matchsticks) {
              int totalLen = Arrays.stream(matchsticks).sum();
              if (totalLen % 4 != 0) {
                  return false;
              }
              Arrays.sort(matchsticks);
              for (int i = 0, j = matchsticks.length - 1; i < j; i++, j--) {
                  int temp = matchsticks[i];
                  matchsticks[i] = matchsticks[j];
                  matchsticks[j] = temp;
              }
      
              int[] edges = new int[4];
              return dfs(0, matchsticks, edges, totalLen / 4);
          }
      
          public boolean dfs(int index, int[] matchsticks, int[] edges, int len) {
              if (index == matchsticks.length) {
                  return true;
              }
              for (int i = 0; i < edges.length; i++) {
                  edges[i] += matchsticks[index];
                  if (edges[i] <= len && dfs(index + 1, matchsticks, edges, len)) {
                      return true;
                  }
                  edges[i] -= matchsticks[index];
              }
              return false;
          }
      }
      ```

    - 感悟与总结

      > 本回溯的关键是理清楚：在遍历火柴列表的过程中，对于当前火柴，他有4种选择，要么选1号边，要么选2号边，要么选3号边，要么选4号边，且同时只能选择一个。代码`22~28`行。
      >
      > 

      >通过火柴长度从大到小排序，来减少搜索的次数。对于第24行代码，每次选取火柴时，选取长度最大的火柴更容易使得条件：`edges[i] <= len`不满足，进而无法进行后面`dfs(index + 1, matchsticks, edges, len)`的这次搜索，起到了剪枝效果。

  - 状态压缩法

    - 代码

      ```java
      class Solution {
          public boolean makesquare(int[] matchsticks) {
              int totalLen = Arrays.stream(matchsticks).sum();
              if (totalLen % 4 != 0) {
                  return false;
              }
              int len = totalLen / 4, n = matchsticks.length;
              int[] dp = new int[1 << n];
              Arrays.fill(dp, -1);
              dp[0] = 0;
              for (int s = 1; s < (1 << n); s++) {
                  for (int k = 0; k < n; k++) {
                      if ((s & (1 << k)) == 0) {
                          continue;
                      }
                      int s1 = s & ~(1 << k);
                      if (dp[s1] >= 0 && dp[s1] + matchsticks[k] <= len) {
                          dp[s] = (dp[s1] + matchsticks[k]) % len;
                          break;
                      }
                  }
              }
              return dp[(1 << n) - 1] == 0;
          }
      }
      ```

    - 感悟与总结

      > dp[s]代表的含义是，选取s所对应的最后一个火柴后，要填充的边的大小。

      > 代码18行说明，填充边是按照边的顺序填充的，即先填充前面的边，前面的边填充完成后，再填充值为0的下一个边。
      >
      > 为啥要取余len,如果==len,那么填充下一个为0的边。

      > 状态转移方程为：
      >
      > ```java
      > int s1 = s & ~(1 << k);
      > if (dp[s1] >= 0 && dp[s1] + matchsticks[k] <= len) {
      > dp[s] = (dp[s1] + matchsticks[k]) % len;
      > break;
      > }
      > ```
      >
      > 该式表明，现在选取的火柴所对应的状态可以由现在去掉一个火柴后其所对应的前面的状态得到。
      >
      > 你可能疑惑，代码19行的`break`为什么当前状态可以由任意去掉一个火柴的前面的状态得到，这个任意怎么保证的。解释如下，比如我当前要选择1、4、8号火柴，那么其状态既可以由1、4号状态得到，也可以由1、8号状态得到。最终结果是一样的，因为虽然前面的状态不同，但是会加上一个缺失值（前者加8，后者加4），即最终都会在前面状态所对应的火柴中进化到相同的结果，即1，4，8。通过上面的分析可知，当前选取的火柴列表可以从删除任意一个火柴的前面的状态得到，且最初的状态为每个火柴在第一条边的状态。综上，即使当前的一步走错了，也不会影响上层的结果，因为上层的视野是整个当前火柴的所有排列组合。

      > `s = 2^n-1`所对应的二进制位都是1，也即当所有的火柴选取后，dp[s]的状态是怎样的，如果为0，表明所有火柴选取后把所有的边都填充满了，true。

      > `dp[s]`中的`s`的取值个数代表了`dp`的状态个数。`s`的取值为`2^n`,即对于任何一个火柴有选取与不选取之分，总共有`2^n`中组合。

      > 代码16行的~代表对该数的所有二进制位取反



### 2022-6-2

- #### [450. 删除二叉搜索树中的节点](https://leetcode.cn/problems/delete-node-in-a-bst/)

  - 递归法

    - 代码

      ```java
      class Solution {
          public TreeNode deleteNode(TreeNode root, int key) {
              if(root==null){
                  return null;
              }
              if(root.val<key){
                  root.right = deleteNode(root.right,key);
              }else if(root.val>key){
                  root.left = deleteNode(root.left,key);
              }else{
                  if(root.left==null&&root.right==null){
                      return null;
                  }else if(root.left==null){
                      return root.right;
                  }else if(root.right==null){
                      return root.left;
                  }else{
                      TreeNode right_min = root.right;
                      while(right_min.left!=null){
                          right_min = right_min.left;
                      }
                      root.val = right_min.val;
                      root.right = deleteNode(root.right,right_min.val);
                  }
              } 
              return root;
          }
      }
      ```

    - 感悟与总结

      >本题递归，关键是理清楚递归函数的含义 -- 从以root为根节点的子树上删除值为key的节点，然后用根节点==返回==修改后的子树。
      >
      >那么递归的思路就是，不断缩小递归函数的root所代表的子树的大小。

  - 迭代法

    - 代码

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
          public TreeNode deleteNode(TreeNode root, int key) {
              TreeNode cur_key = root;
              TreeNode parent_key = null;
              while(cur_key!=null&&cur_key.val!=key){
                  parent_key = cur_key;
                  if(key>cur_key.val){
                      cur_key = cur_key.right;
                  }else{
                      cur_key = cur_key.left;
                  }
              }
              if(cur_key==null){
                  return root;
              }
              if(cur_key.left==null&&cur_key.right==null){
                  cur_key = null;
                  if(parent_key==null){
                      return cur_key;
                  }else{
                       if(parent_key.left!=null&&parent_key.left.val == key){
                          parent_key.left = cur_key;
                      }else{
                          parent_key.right = cur_key;
                      }
                      return root;
                  }
              }else if(cur_key.left==null){
                  cur_key = cur_key.right;
                  if(parent_key==null){
                      return cur_key;
                  }else{
                       if(parent_key.left!=null&&parent_key.left.val == key){
                          parent_key.left = cur_key;
                      }else{
                          parent_key.right = cur_key;
                      }
                      return root;
                  }
              }else if(cur_key.right==null){
                  cur_key = cur_key.left;
                  if(parent_key==null){
                      return cur_key;
                  }else{
                       if(parent_key.left!=null&&parent_key.left.val == key){
                          parent_key.left = cur_key;
                      }else{
                          parent_key.right = cur_key;
                      }
                      return root;
                  }
              }else{
                  TreeNode right_min = cur_key.right;
                  TreeNode right_min_parent = cur_key;
                  while(right_min.left!=null){
                      right_min_parent = right_min;
                      right_min = right_min.left;
                  }
                  if(right_min_parent==cur_key){
                      right_min_parent.right = right_min.right;
                  }else{
                      right_min_parent.left = right_min.right;
                  }
      
                  cur_key.val = right_min.val;
              }
              return root;
          }
      }
      ```

    - 感悟与总结

      > java没有指针，这就导致每个节点的左右子节点是通过构造函数传参，传的是引用。
      >
      > 但是，我有一个疑问，就是为什么该引用所代表的整个对象改变了，其父节点必须重新指向。但是，如果仅仅是内部的值改变了，无需重新指向呢。

### 2022-6-3

- #### [829. 连续整数求和](https://leetcode.cn/problems/consecutive-numbers-sum/)

  - 数学

    - 代码

      ```java
      class Solution {
          public int consecutiveNumbersSum(int n) {
              int ans=0;
              for(int k = 1;k*(k+1)<=2*n;k++){
                  if(isKConsecutive(n,k)){
                      ans++;
                  }
              }
              return ans;
          }
          public boolean isKConsecutive(int n,int k){
              if(k%2==1){
                  return n%k==0;
              }else{
                  return n%k!=0 && 2*n%k==0;
              }
          }
      }
      ```

    - 感悟与总结

      > 每个情况所包含的连续元素的个数是不同的，即k不同。也就是k的不同取值的个数就是结果的个数。

      >又因为是连续的，那么连续元素的求和可以用第一项x和和个数k求得。

      > 分情况讨论k的奇偶性，和x是否为正整数（如果是，那么这个以x开始的连续情况就是合法的）。