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

- #### [526. 优美的排列](https://leetcode.cn/problems/beautiful-arrangement/)

  - 回溯

    - 代码

      ```java
      class Solution {
          List<Integer>[] match;
          boolean[] vis;
          int num;
      
          public int countArrangement(int n) {
              vis = new boolean[n + 1];
              match = new List[n + 1];
              for (int i = 0; i <= n; i++) {
                  match[i] = new ArrayList<Integer>();
              }
              for (int i = 1; i <= n; i++) {
                  for (int j = 1; j <= n; j++) {
                      if (i % j == 0 || j % i == 0) {
                          match[i].add(j);
                      }
                  }
              }
              backtrack(1, n);
              return num;
          }
      
          public void backtrack(int index, int n) {
              if (index == n + 1) {
                  num++;
                  return;
              }
              for (int x : match[index]) {
                  if (!vis[x]) {
                      vis[x] = true;
                      backtrack(index + 1, n);
                      vis[x] = false;
                  }
              }
          }
      }
      ```

    - 感悟与总结

      > 回溯法始终维护着一个列表的index，每当进行到index时，会有几种选择，选中一个继续深入返回后继续选中下一个。

      > 对于本题，当进行到index时，进行满足这个`(i % j == 0 || j % i == 0)`条件的选择。

      > 为了减少时间复杂度，提前为每个index进行了选择，记录在数组match中

      > 回溯的时间复杂度为：O(n!)

  - 状态压缩+动态规划

    - 代码

      ```java
      class Solution {
          public int countArrangement(int n) {
              int[] f = new int[1 << n];
              f[0] = 1;
              for (int mask = 1; mask < (1 << n); mask++) {
                  int num = Integer.bitCount(mask);
                  for (int i = 0; i < n; i++) {
                      if ((mask & (1 << i)) != 0 && ((num % (i + 1)) == 0 || (i + 1) % num == 0)) {
                          f[mask] += f[mask ^ (1 << i)];
                      }
                  }
              }
              return f[(1 << n) - 1];
          }
      }
      ```

    - 感悟与总结

      > 在状态压缩中，二进制的位数代表列表的长度；二进制位的选取代表列表中该位元素的选取；二进制整体的不同取值代表了不同的状态，总共有2^n

      > 状态压缩的递推过程同动态规划的转移方程，递推是该状态所对应的大二进制任意去掉一位的小二进制进化而来。

      > 最底层的二进制位，是n个只有一个1的二进制，是最初的二进制，上层的二进制都经过这层二进制位进化而来。我说的层，是二进制中1的个数。

      > 关键是，理清楚状态与实际列表对应关系和状态转移方程

      > 状态压缩的时间复杂度为 : O(2^n)

### 2022-6-4

- #### [929. 独特的电子邮件地址](https://leetcode.cn/problems/unique-email-addresses/)

  - 哈希表

    - 代码

      ```java
      class Solution {
          public int numUniqueEmails(String[] emails) {
              Set<String> emailSet = new HashSet<String>();
              for(String email : emails){
                  int i = email.indexOf('@');
                  String local = email.substring(0,i).split("\\+")[0];
                  local = local.replace(".","");
                  emailSet.add(local+email.substring(i));
              }
              return emailSet.size();
          }
      }
      ```

    - 感悟与总结

      >`str.substring(i)`返回一个新的字符串，该新的字符串以`i`开头直至结尾。

      > `str.substring(i,j)`返回一个新的字符串，该新的字符串以`i`开头,以`j`结尾，左闭右开。

      > split() 方法根据匹配给定的正则表达式来拆分字符串。
      >
      > **注意：** . 、 $、+、-、 | 和 * 等转义字符，必须得加 `\\`。
      >
      > **注意：**多个分隔符，可以用 | 作为连字符。
      >
      > ```java
      > public String[] split(String regex, int limit)
      > ```
      >
      > 参数
      >
      > - **regex** -- 正则表达式分隔符。
      > - **limit** -- 分割的份数。
      >
      > 返回值
      >
      > 字符串数组。

- #### [638. 大礼包](https://leetcode.cn/problems/shopping-offers/)

  - 记忆化搜索

    - 代码

      ```java
      class Solution {
          //记忆化搜索，记忆那些已经保存过的'need'状态。以便直接被上层调用，减少复杂度。
          Map<List<Integer>,Integer> memo = new HashMap<List<Integer>,Integer>();
      
          public int shoppingOffers(List<Integer> price, List<List<Integer>> special, List<Integer> needs) {
              //记录有多少种可买的商品。
              int n = price.size();
      
              //过滤那些无法进行优惠的大礼包
              List<List<Integer>> filterSpecial = new ArrayList<List<Integer>>();
              for(List<Integer> sp : special){
                  int totalCount = 0,totalPrice = 0;
                  for(int i=0;i<n;i++){
                      totalCount+=sp.get(i);
                      totalPrice+=sp.get(i)*price.get(i);
                  }
                  if(totalCount>0 && totalPrice>sp.get(n)){
                      filterSpecial.add(sp);
                  }
              }
              return dfs(price,special,needs,filterSpecial,n);
          }
      
          //记忆化搜素计算满足购物清单的最低价格
          public int dfs(List<Integer> price,List<List<Integer>> special,List<Integer> curNeeds,List<List<Integer>> filterSpecial,int n){
              //只计算还没记忆化的购物清单状态
              if(!memo.containsKey(curNeeds)){
                  int minPrice = 0;
                  for(int i=0;i<n;i++){
                      minPrice+=curNeeds.get(i)*price.get(i);//在当前购物状态下，不购买任何大礼包的情况，原价购买购物清单中的所有物品。
                  }
                  for(List<Integer> curSpecial : filterSpecial){
                      int specialPrice = curSpecial.get(n);
                      List<Integer> nxtNeeds = new ArrayList<Integer>();
                      for(int i=0;i<n;i++){
                          if(curSpecial.get(i)>curNeeds.get(i)){ //不能够购买超出购物清单数量的物品。
                              break;
                          }
                          nxtNeeds.add(curNeeds.get(i)-curSpecial.get(i));
                      }
                      if(nxtNeeds.size()==n){ //该大礼包并没有超出限制，可以购买
                          minPrice = Math.min(minPrice,dfs(price,special,nxtNeeds,filterSpecial,n)+specialPrice);
                      }
                  }
                  memo.put(curNeeds,minPrice);
              }
              return memo.get(curNeeds);
          }
      }
      ```

    - 感悟与总结

      > 采用记忆化，对于一个特定的购物清单状态只需要计算一次, 下一次高层可以直接调用。

      > 不同于回溯，这个将上层的问题，递归的推迟到下层、下层层、、、，在返回时选取最优子结构。

      > 因为`1<=needs<<6 and 0<=needs[i]<=10`，所以最多只有 11^6 = 1771561 种不同的购物清单`needs`。我们可以将所有可能的购物清单作为状态，并考虑这些状态之间相互转移的方法。

      > 因为大礼包中可能包含多个物品，所以并不是所有状态都可以得到。

      > 我们也可以考虑使用状态压缩的方法来存储购物清单 `needs`。但是因为购物清单中每种物品都可能有 [0,10] 个，使用状态压缩需要设计一个相对复杂的方法来解决计算状态变化以及比较状态大小的问题，性价比较低。

### 2022-6-5

- [478. 在圆内随机生成点](https://leetcode.cn/problems/generate-random-point-in-a-circle/)

  - 数学——拒绝采样

    - 代码

      ```java
      class Solution {
          Random random;
          double xc,yc,r;
          public Solution(double radius, double x_center, double y_center) {
              random = new Random();
              xc = x_center;
              yc = y_center;
              r = radius;
          }
          
          public double[] randPoint() {
              while(true){
                  double x = random.nextDouble()*(2*r)-r;
                  double y = random.nextDouble()*(2*r)-r;
                  if(x*x+y*y<r*r){
                      return new double[]{xc+x,yc+y};
                  }
              }
          }
      }
      ```

    - 感悟与总结

      > 不失一般性，我们只考虑在原点且半径为 11 的单位圆。对于非一般性的情况，我们只需要把生成的点的坐标根据半径等比例放大，再根据圆心坐标进行平移即可。

      > `public double nextDouble()`
      > Returns a pseudorandom double value between zero (inclusive) and one (exclusive).

      > `public double nextDouble(double bound)`
      > Returns a pseudorandom double value between 0.0 (inclusive) and the specified bound (exclusive).


### 2022-6-6

- #### [729. 我的日程安排表 I](https://leetcode.cn/problems/my-calendar-i/)

  - 暴力法

    - 代码

      ```java
      public class MyCalendar {
          List<int[]> calendar;
          MyCalendar() {
              calendar = new ArrayList();
          }
          public boolean book(int start, int end) {
              for (int[] iv: calendar) {
                  if (iv[0] < end && start < iv[1]) return false;
              }
              calendar.add(new int[]{start, end});
              return true;
          }
      }
      ```

    - 感悟与总结

      > 我们将维护一个日程安排列表（不一定要排序）。当且仅当其中一个日程安排在另一个日程安排结束后开始时，两个日程安排 [s1，e1) 和 [s2，e2) 不冲突：e1<=s2 或 e2<=s1。这意味着当 s1<e2 和 s2<e1 时，日程安排发生冲突。

      > 运算规则：$\frac{}{AUB} == \frac{}{A}n\frac{}{B}$

  - 差分数组，插旗法

    - 代码

      ```java
      class MyCalendar {
          TreeMap<Integer, Integer> delta;
          public MyCalendar() {
              delta = new TreeMap();
          }
          public boolean book(int start, int end) {
              delta.put(start, delta.getOrDefault(start, 0) + 1);
              delta.put(end, delta.getOrDefault(end, 0) - 1);
              int active = 0;
              for (int d: delta.values()) {
                  active += d;
                  if (active >= 2) {
                      delta.put(start, delta.get(start) - 1);
                      delta.put(end, delta.get(end) + 1);
                      return false;
                  }
              }
              return true;
          }
      }
      ```

    - 感悟与总结

      > `TreeMap`是底层用红黑树实现，可以对`map`中的`key`进行排序（一种平衡二叉排序树），对当个元素的插入、查询、删除可以实现`O(lg(n))`的时间复杂度。通过`TreeMap`，我们按时间顺序维护日程安排。

      > 先来介绍一下插旗法：进入一个区间的时候将该点坐标对应的值+1，代表插上一面进入的🚩，离开时将该点坐标值-1，代表插上一面离开的🚩，在同一个点可以同时插进入的旗或离开的旗，因为这样并不形成区间重叠。

      > 插旗法理解起来就像是左右括号一样，进入🚩就是左括号，出去🚩就是右括号，左右🚩可以抵消，就像左右括号可以抵消一样，题目这样就变成了如果你在两个以上的==括号嵌套==内就返回false，其余返回true。这个括号嵌套的层数，就是时间段重叠的个数。



- #### [731. 我的日程安排表 II](https://leetcode.cn/problems/my-calendar-ii/)

  - 暴力法

    - 代码

      ```java
      class MyCalendarTwo {
          List<int[]> calendar;
          List<int[]> overlaps;
          public MyCalendarTwo() {
              calendar = new ArrayList();
              overlaps = new ArrayList();
          }
          public boolean book(int start, int end) {
              for(int[] iv:overlaps){
                  if(iv[0]<end && start<iv[1]) return false;
              }
              for(int[] iv : calendar){
                  if(iv[0]<end&&start<iv[1]){
                      overlaps.add(new int[]{Math.max(start,iv[0]),Math.min(end,iv[1])});
                  }
              }
              calendar.add(new int[]{start,end});
              return true;
          }
      }
      ```

    - 感悟与总结

      > 维护一重预订列表和双重预订列表。当预订一个新的日程安排 `[start, end)` 时，如果它与双重预订列表冲突，则会产生三重预定。

      > - 如果新的日程安排与双重预订冲突，则返回 `false`。否则，我们会将与一重预定列表冲突的时间添加到双重预订列表中，并将该预定添加到一重预定列表中。

  - 差分数组，插旗法

    - 代码

      ```java
      class MyCalendarTwo {
          TreeMap<Integer,Integer> delta;
          public MyCalendarTwo() {
              delta = new TreeMap();
          }
          public boolean book(int start, int end) {
              delta.put(start,delta.getOrDefault(start,0)+1);
              delta.put(end,delta.getOrDefault(end,0)-1);
              int active = 0;
              for(int d:delta.values()){
                  active+=d;
                  if(active>=3){
                      delta.put(start,delta.get(start)-1);
                      delta.put(end,delta.get(end)+1);
                      return false;
                  }
              }
              return true;
          }
      }
      ```

    - 感悟与总结

      > 模板代码：
      >
      > > 这种方法非常适合解最大的区间重叠数量 (或最大的并行数量) 的题目，能够将时间复杂度控制在 O(nlog{n})*O*(*n**l**o**g**n*)，而且代码可以说是八九不离十。
      >
      > ```java
      > class MyCalendarTwo {
      >     TreeMap<Integer,Integer> delta;
      >     public MyCalendarTwo() {
      >         delta = new TreeMap();
      >     }
      >     public boolean book(int start, int end) {
      >         delta.put(start,delta.getOrDefault(start,0)+1);
      >         delta.put(end,delta.getOrDefault(end,0)-1);
      >         int active = 0;
      >         for(int d:delta.values()){
      >             active+=d;
      >             if(active>=???){ //???,代表几重
      >                 delta.put(start,delta.get(start)-1);
      >                 delta.put(end,delta.get(end)+1);
      >                 return false;
      >             }
      >         }
      >         return true;
      >     }
      > }
      > ```

- #### [732. 我的日程安排表 III](https://leetcode.cn/problems/my-calendar-iii/)

  - 差分数组，插旗法

    - 代码

      ```java
      class MyCalendarThree {
          private TreeMap<Integer, Integer> delta;
          public MyCalendarThree() {
              delta = new TreeMap();
          }
          public int book(int start, int end) {
              int ans = 0;
              int active = 0;
              delta.put(start, delta.getOrDefault(start, 0) + 1);
              delta.put(end, delta.getOrDefault(end, 0) - 1);
              for(int d:delta.values()){
                  active+=d;
                  ans = Math.max(active,ans);
              }
              return ans;
          }
      }
      ```

    - 总结与感悟

      > 同上

  - 线段树

    - 代码

      ```java
      class MyCalendarThree{
          private Map<Integer,Integer> tree;
          private Map<Integer,Integer> lazy;
          public MyCalendarThree(){
              tree = new HashMap<Integer,Integer>();
              lazy = new HashMap<Integer,Integer>();
          }
          public int book(int start,int end){
              update(start,end-1,0,1000000000,1);
              return tree.getOrDefault(1,0);
          }
          public void update(int start,int end,int l,int r,int idx){
              if(r<start || end<l){
                  return;
              }
              if(start<=l && r<=end){
                  tree.put(idx,tree.getOrDefault(idx,0)+1);
                  lazy.put(idx,lazy.getOrDefault(idx,0)+1);
              }else{
                  int mid = (l+r)>>1;
                  update(start,end,l,mid,2*idx);
                  update(start,end,mid+1,r,2*idx+1);
                  tree.put(idx,lazy.getOrDefault(idx,0)+Math.max(tree.getOrDefault(2*idx,0),tree.getOrDefault(2*idx+1,0)));
              }
          }
      }
      ```

    - 感悟与总结

      > `tree`是整体线段树；`lazy`是每个分段。它们中的线段都有`idx`进行标识。

      > 题意限制：
      >
      > - `0 <= start < end <= 109`
      > - 给你一些日程安排 `[start, end)`
      >
      > - `MyCalendarThree()` 初始化对象。
      > - `int book(int start, int end)` 返回一个整数 k ，表示日历中存在的 k 次预订的最大值。
      >
      > 根据题意，线段树的区间为`[0,1000000000]`，且每个点都是整数。对于代码第九行，为什么end要减一，是因为end是开区间取不到，且是整数，所以为了各个线段区间不重复，才这样考虑。

      > 对于递归：
      >
      > - 如果无交叉则返回，说明不更新线段树的该分段；
      > - 如果覆盖，级分段区间小于给定区间，则直接加一，有点类似叶子节点。
      > - 如果交叉，则对该分段区间进行分解，可以把该分段区间理解为父节点，分解后递归子节点后更新父节点。

      > 线段树更详细的解释可以看下面🔗：
      >
      > [线段树](https://leetcode.cn/problems/my-calendar-iii/solution/xian-duan-shu-by-xiaohu9527-rfzj/ )

### 2022-6-7

- #### [875. 爱吃香蕉的珂珂](https://leetcode.cn/problems/koko-eating-bananas/)

  - 二分法

    - 代码

      ```java
      class Solution {
          public int minEatingSpeed(int[] piles, int h) {
              int low = 1;
              int high = 1;
              for(int pile:piles){
                  high = Math.max(high,pile);
              }
              int ans = high;
              while(low<high){
                  int speed = (high+low)>>1;
                  long time = getTime(piles,speed);
                  if(time<=h){
                      high = speed;
                      ans = high;
                  }else{
                      low = speed+1;
                  }
              }
              return ans;
          }
          public long getTime(int[] piles,int speed){
              long time = 0;
              for(int pile:piles){
                  time+=(pile+speed-1)/speed;
              }
              return time;
          }
      }
      ```

    - 感悟与总结

      > $⌈speed/pile⌉$等价于$⌊(speed+pile-1)/speed⌋$

      > 由于吃香蕉的速度和是否可以在规定时间内吃掉所有香蕉之间存在单调性，因此可以使用二分查找的方法得到最小速度 k。	

      > 初始时，speed左边界为1，右边界为所有堆里面的最大值。

      > 在二分查找的过程中，根据是否能在规定时间内吃完来调整mid

- #### [698. 划分为k个相等的子集](https://leetcode.cn/problems/partition-to-k-equal-sum-subsets/)

  - 状态压缩

    - 代码

      ```java
      class Solution {
          public boolean canPartitionKSubsets(int[] nums, int k) {
              if(k==1){
                  return true;
              }
              int len = nums.length;
              Arrays.sort(nums);
              int sum = 0;
              for(int num : nums){
                  sum+=num;
              }
              if(sum%k!=0){
                  return false;
              }
              int target = sum/k;
              if(nums[len-1]>target){
                  return false;
              }
             
              int size = 1<<len;
              boolean[] dp = new boolean[size];
              dp[0] = true;
              int[] currentSum = new int[size];
              for(int i=0;i<size;i++){
                  //z
                  if(dp[i]!=true){
                      continue;
                  }
                  //基于当前状态，添加一个数以后
                  for(int j=0;j<len;j++){
                      if((i&(1<<j))!=0){
                          continue;
                      }
                      int next = i|(1<<j);
                      if(dp[next]==true){
                          continue;
                      }
                      if((currentSum[i]%target)+nums[j]<=target){
                          currentSum[next] = currentSum[i]+nums[j];
                          dp[next] = true;
                      }else{
                          //由于数组已经排好序了，剩下的就没必要枚举了
                          break;
                      }
      
                  }
              }
              return dp[size-1];
          }
      }
      ```

    - 感悟与总结

      > 如果把k个相同的子集看作K个相同的桶，把子集里面的元素当成球。那么最大的球一定比桶小，否则就放不下了。
      >
      > 这个条件也就决定了代码38行在每次只选取一个球的正确性，因为要么更小，要么恰好接着下一个，不能更大。

      > 那为什么size-1时会有更小和恰好两种情况下都是true呢。因为，在这两种情况和size-1下，就只有一个桶没满了，其它都满了，那么这个桶肯定会满。他是通过%==0来开始下一个桶的。

  - 回溯

    - 代码--球视角

      ```java
      public boolean canPartitionKSubsets(int[] nums, int k) {
          int sum = 0;
          for (int i = 0; i < nums.length; i++) sum += nums[i];
          if (sum % k != 0) return false;
          int target = sum / k;
          int[] bucket = new int[k + 1];
          return backtrack(nums, 0, bucket, k, target);
      }
      // index : 第 index 个球开始做选择
      // bucket : 桶
      private boolean backtrack(int[] nums, int index, int[] bucket, int k, int target) {
      
          // 结束条件：已经处理完所有球
          if (index == nums.length) {
              // 判断现在桶中的球是否符合要求 -> 路径是否满足要求
              for (int i = 0; i < k; i++) {
                  if (bucket[i] != target) return false;
              }
              return true;
          }
      
          // nums[index] 开始做选择
          for (int i = 0; i < k; i++) {
              // 剪枝：放入球后超过 target 的值，选择一下桶
              if (bucket[i] + nums[index] > target) continue;
              // 做选择：放入 i 号桶
              bucket[i] += nums[index];
      
              // 处理下一个球，即 nums[index + 1]
              if (backtrack(nums, index + 1, bucket, k, target)) return true;
      
              // 撤销选择：挪出 i 号桶
              bucket[i] -= nums[index];
          }
      
          // k 个桶都不满足要求
          return false;
      }
      ```

    - 代码---桶视角

      ```java
      // 备忘录，存储 used 的状态
      private HashMap<Integer, Boolean> memo = new HashMap<>();
      
      public boolean canPartitionKSubsets(int[] nums, int k) {
          int sum = 0;
          for (int i = 0; i < nums.length; i++) sum += nums[i];
          if (sum % k != 0) return false;
          int target = sum / k;
          // 使用位图技巧
          int used = 0;
          int[] bucket = new int[k + 1];
          return backtrack(nums, 0, bucket, k, target, used);
      }
      private boolean backtrack(int[] nums, int start, int[] bucket, int k, int target, int used) {
          // k 个桶均装满
          if (k == 0) return true;
      
          // 当前桶装满了，开始装下一个桶
          if (bucket[k] == target) {
              // 注意：桶从下一个开始；球从第一个开始
              boolean res = backtrack(nums, 0, bucket, k - 1, target, used);
              memo.put(used, res);
              return res;
          }
      
          if (memo.containsKey(used)) {
              // 如果当前状态曾今计算过，就直接返回，不要再递归穷举了
              return memo.get(used);
          }
      
          // 第 k 个桶开始对每一个球选择进行选择是否装入
          for (int i = start; i < nums.length; i++) {
              // 如果当前球已经被装入，则跳过
              if (((used >> i) & 1) == 1) continue;
              // 如果装入当前球，桶溢出，则跳过
              if (bucket[k] + nums[i] > target) continue;
      
              // 装入 && 标记已使用
              bucket[k] += nums[i];
              // 将第 i 位标记为 1
              used |= 1 << i;
      
              // 开始判断是否选择下一个球
              // 注意：桶依旧是当前桶；球是下一个球
              // 注意：是 i + 1
              if (backtrack(nums, i + 1, bucket, k, target, used)) return true;
      
              // 拿出 && 标记未使用
              bucket[k] -= nums[i];
              // 将第 i 位标记为 0
              used ^= 1 << i;
          }
          // 如果所有球均不能使所有桶刚好装满
          return false;
      }
      ```

    - 感悟与总结

      > 对列表排序一般都可以提高回溯

      > 回溯有三大要素：
      >
      > - 路径：已经做出的选择
      > - 选择列表：当前可以做的选择
      > - 结束条件：到达决策树底层，无法再做选择的条件

      > 根据选择的主人不同，可以有不同的视角。如：上面的桶视角和球视角。回溯一般都是指数函数，指数函数的底的大小就是最初最大可选择的个数。要根据视角选择指数函数底更小的时间复杂度。

      > 可以使用位运算来当作`hashmap`的key，状态压缩也是这样的应用。

      > [经典回溯算法：集合划分问题「重要更新 🔥🔥🔥」](https://leetcode.cn/problems/partition-to-k-equal-sum-subsets/solution/by-lfool-d9o7/)

### 2022-6-8

- #### [1037. 有效的回旋镖](https://leetcode.cn/problems/valid-boomerang/)

  - 简单模拟

    - 代码

      ```java
      class Solution {
          public boolean isBoomerang(int[][] points) {
              if((points[0][0]==points[1][0]&&points[0][1]==points[1][1])||(points[0][0]==points[2][0]&&points[0][1]==points[2][1])||(points[2][0]==points[1][0]&&points[2][1]==points[1][1])){
                  return false;
              }
      
              if((points[0][0]==points[1][0]&&points[1][0]==points[2][0])||(points[0][1]==points[1][1]&&points[1][1]==points[2][1])){
                  return false;
              }
      
              if(points[0][0]==points[1][0]||points[1][0]==points[2][0]||points[0][0]==points[2][0]||points[0][1]==points[1][1]||points[1][1]==points[2][1]||points[0][1]==points[2][1]){
                  return true;
              }
      
              if(1.0*(points[1][1]-points[0][1])/(points[1][0]-points[0][0])==1.0*(points[2][1]-points[1][1])/(points[2][0]-points[1][0])){
                  return false;
              }
      
              return true;
          }
      }
      ```

    - 感悟与总结

      > 横坐标或纵坐标，如果三值相等，那么共线。
      >
      > 在此基础上，如果只要二值共线，那么一定不共线（因为三坐标互补相同）。
      >
      > 在上述基础上，只存在没两值相同的了，所有就可以用斜率计算 (已经排除了分母为0了)。

      > 整数`/`是取模。那么分子乘上`1.0`就变成浮点数，`/`就是除以。

  - 数学 -- 向量交叉积

    - 代码

      ```java
      class Solution {
          public boolean isBoomerang(int[][] points) {
              int[] v1 = {points[1][0]-points[0][0],points[1][1]-points[0][1]};
              int[] v2 = {points[2][0]-points[0][0],points[2][1]-points[0][1]};
              return v1[0]*v2[1]-v1[1]*v2[0]!=0;
          }
      }
      ```

    - 感悟与总结

      > 判断向量垂直用点乘
      >
      > 判断向量平行用叉乘

      > 点乘0是个数，用叉乘0是个向量。

  - #### [136. 只出现一次的数字](https://leetcode.cn/problems/single-number/)

    - 位运算

      - 代码

        ```java
        class Solution {
            public int singleNumber(int[] nums) {
                int ans = 0;
                for(int num:nums){
                    ans = ans^num;
                }
                return ans;
            }
        }
        ```

      - 感悟与总结

        > 不需要额外空间的方法，就往位运算上想

### 2022-6-9

- #### [497. 非重叠矩形中的随机点](https://leetcode.cn/problems/random-point-in-non-overlapping-rectangles/)

  - #### 前缀和 + 二分查找

    - 代码

      ```java
      class Solution {
          List<Integer> sumAreaList;
          Random random;
          int[][] rects;
          public Solution(int[][] rects) {
              this.rects = rects;
              sumAreaList = new ArrayList();
              random = new Random();
              sumAreaList.add(0);
              for(int[] rect : rects){
                  int area = (rect[2]-rect[0]+1)*(rect[3]-rect[1]+1);
                  int lastSum = sumAreaList.get(sumAreaList.size()-1);
                  sumAreaList.add(lastSum+area);
              }
          }
          
          public int[] pick() {
              int value = random.nextInt(sumAreaList.get(sumAreaList.size()-1))+1;
              int left = 0;
              int right = sumAreaList.size()-1;
              while(left<right){
                  int mid = (left+right)>>1;
                  if(sumAreaList.get(mid)>=value){
                      right = mid;
                  }else{
                      left = mid+1;
                  }
              }
              int[] curRect = rects[right-1]; //因为前缀多了一个0首位
              int x = random.nextInt(curRect[2]-curRect[0]+1)+curRect[0];
              int y = random.nextInt(curRect[3]-curRect[1]+1)+curRect[1];
              return new int[]{x,y};
          }
      }
      ```

    - 感悟与总结

      > 总体思路：
      >
      > 「先随机使用哪个矩形，再随机该矩形内的点」，其中后者是极其容易的，根据矩形特质，只需在该矩形的 XY 坐标范围内随机即可确保等概率，而前者（随机使用哪个矩形）为了确保是等概率，我们不能简单随机坐标，而需要结合面积来做。

      > 具体的，我们可以预处理前缀和`sumAreaList`（前缀和下标默认从 1 开始），其中 `sumAreaList[i]`代表前 `i` 个矩形的面积之和，最终 `sumAreaList[n]` 为所有矩形的总面积，我们可以在 `[1,sumAreaList[n]]` 范围内随机，假定随机到的值为 `value`，然后利用`sumAreaList`的具有单调性，进行「二分」，找到 valval 所在的矩形。然后在该矩形中进行随机，得到最终的随机点。

      > 注意二分的使用，尤其代码`23`行。

- #### [191. 位1的个数](https://leetcode.cn/problems/number-of-1-bits/)

  - 位运算

    - 代码

      ```java
      public class Solution {
          // you need to treat n as an unsigned value
          public int hammingWeight(int n) {
              int ans = 0;
              for(int i=0;i<32;i++){
                  if((n&(1<<i))!=0){
                      ans++;
                  }
              }
              return ans;
          }
      }
      ```

    - 感悟与总结

      > 循环检查二进制位
      >
      > 我们可以直接循环检查给定整数 n 的二进制位的每一位是否为 1。
      >
      > 当检查第 i 位时，我们可以让 n 与 $2^i$ 进行`&`运算，当且仅当 n 的第 i 位为 1 时，运算结果不为 0。

  - 位运算优化

    - 代码

      ```java
      public class Solution {
          public int hammingWeight(int n) {
              int ret = 0;
              while (n != 0) {
                  n &= n - 1;
                  ret++;
              }
              return ret;
          }
      }
      ```

    - 感悟与总结

      > `n & (n−1)`，其运算结果恰为把 n*n* 的二进制位中的最低位的 1 变为 0 之后的结果。

- #### [67. 二进制求和](https://leetcode.cn/problems/add-binary/)

  - 模拟

    - 代码

      ```java
      class Solution {
          public String addBinary(String a, String b) {
              int carryVal = 0;
              int len = Math.max(a.length(),b.length());
              String str = "";
              int oneIndex = a.length()-1;
              int twoIndex = b.length()-1;
              while(oneIndex>=0&&twoIndex>=0){
                  str=((a.charAt(oneIndex)-'0')^(b.charAt(twoIndex)-'0')^carryVal)+str;
                  // System.out.println(str);
                  if(a.charAt(oneIndex)-'0'+b.charAt(twoIndex)-'0'+carryVal>1){
                      carryVal = 1;
                  }else{
                      carryVal = 0;
                  }
                  oneIndex--;
                  twoIndex--;
      
              }
              while(oneIndex>=0){
                  str=((a.charAt(oneIndex)-'0')^carryVal)+str;
                  if(a.charAt(oneIndex)-'0'+carryVal>1){
                      carryVal = 1;
                  }else{
                      carryVal = 0;
                  }
                  oneIndex--;
              }
              while(twoIndex>=0){
                  str=((b.charAt(twoIndex)-'0')^carryVal)+str;
                  if(b.charAt(twoIndex)-'0'+carryVal>1){
                      carryVal = 1;
                  }else{
                      carryVal = 0;
                  }
                  twoIndex--;
              }
              if(carryVal==1){
                  str = carryVal+str;
              }
              return  str;
          }
      }
      ```

    - 感悟与总结

      > 数字转字符串
      >
      > - `String str = 123+"";`
      >
      > - `String str = String.valueOf(123);`
      > - `String str = Integer.toString(123);`
      >
      > 字符串转数字
      >
      > - `int i = Integer.parseInt(str);`
      >
      > 数字转字符，用字符串做桥梁：
      >
      > `char a = str.charAt(i);`
      >
      > 字符转数字
      >
      > `char a = '2';`
      >
      > `int i = a-'0'`

### 2022-6-10

- #### [730. 统计不同回文子序列](https://leetcode.cn/problems/count-different-palindromic-subsequences/)

  - 动态规划

    - 代码

      ```java
      class Solution {
          public int countPalindromicSubsequences(String s) {
              int mod = 1000000007;
              int n = s.length();
              int[][] dp = new int[n][n];
              //一个单字符是一个回文子序列
              for(int i=0;i<n;i++){
                  dp[i][i] = 1;
              }
              //从长度为2的子串开始计算
              for(int len = 2;len<=n;len++){
                  //挨个计算长度为len的子串的回文子序列的个数
                  for(int i=0;i+len<=n;i++){
                      int j = i+len-1;
                      //该串的左右边界值相同
                      if(s.charAt(i)==s.charAt(j)){
                          int left = i+1;
                          int right = j-1;
                          //找到第一个和s[i]相同的字符
                          while(left<=right&&s.charAt(left)!=s.charAt(i)){
                              left++;
                          }
                          //找到第一个和s[j]相同的字符
                          while(left<=right&&s.charAt(right)!=s.charAt(j)){
                              right--;
                          }
                          if(left>right){
                              dp[i][j] = 2*dp[i+1][j-1]+2;
                          }else if(left==right){
                              dp[i][j] = 2*dp[i+1][j-1]+1;
                          }else{
                              dp[i][j] = 2*dp[i+1][j-1]-dp[left+1][right-1];
                          }
                      }else{
                          //该串左右边界值不相同
                          dp[i][j] = dp[i][j-1]+dp[i+1][j]-dp[i+1][j-1];
                      }
                      //处理超范围的结果
                     //由于MOD的存在，可能出现在和前面计算出的dp[left + 1][right - 1]相减值为负，因此需要做个判断！
                      dp[i][j] = (dp[i][j]>=0)?dp[i][j]%mod:dp[i][j]+mod;
                  }
                  return dp[0][n-1];
              }
          }
      }
      ```

    - 感悟与总结

      > 突破点：如果字符串是回文，那么左右边界值相同，去掉边界值后还是回文串。根据这个条件进行递推。

      > [统计不同回文子序列【动态规划】🎉🎉🎉](https://leetcode.cn/problems/count-different-palindromic-subsequences/solution/tong-ji-butong-by-jiang-hui-4-q5xf/)

### 2022-6-11

- #### [926. 将字符串翻转到单调递增](https://leetcode.cn/problems/flip-string-to-monotone-increasing/)

  - 动态规划

    - 代码

      ```java
      class Solution {
          public int minFlipsMonoIncr(String s) {
              int n = s.length();
              int dp0 = 0,dp1 = 0;
              for(int i=0;i<n;i++){
                  char c = s.charAt(i);
                  int dp0New = dp0;
                  int dp1New = Math.min(dp0,dp1);
                  if(c=='1'){
                      dp0New++;
                  }else{
                      dp1New++;
                  }
                  dp0 = dp0New;
                  dp1 = dp1New;
              }
              return Math.min(dp0,dp1);
          }
      }
      ```

    - 感悟与总结

      > `0,1`代表双标志位状态转移
      
      > ![2022-6-11](https://raw.githubusercontent.com/ZYNORl/leetcode/main/picture/2022-6-11.jpg)
  
  - 前缀和
  
    - 代码
  
      ```java
      class Solution {
          public int minFlipsMonoIncr(String s) {
              int n = s.length();
              int[] PreSum = new int[n+1];
              for(int i=1;i<=n;i++){
                  PreSum[i]=PreSum[i-1]+(s.charAt(i-1)-'0');
              }
              int ans = n;
              for(int i=0;i<=n;i++){
                  ans = Math.min(ans,PreSum[i]+n-i-(PreSum[n]-PreSum[i]));
              }
              return ans;
          }
      }
      ```
  
    - 感悟与总结
  
      >递增序列想到前缀和
  
      > 该递增序列的特点是：前面全是0，后面全是1
  
      > 如果把`i`当作最后一个`0`的坐标，那么前缀和在该处的值，就是前面`0`中有多少`1`需要反转。同理`n-i-(PreSum[n]-PreSum[i]))`是后面1中有多少0需要反转。在代码`10`行。