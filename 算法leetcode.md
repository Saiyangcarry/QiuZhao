# 动态规划

#### [5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

示例 1：

输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
示例 2：

输入: "cbbd"
输出: "bb"



中心扩展法：击败77%

```java
class Solution {
    private int n,count=0;
    private int first,second,length=0;
    public String longestPalindrome(String s) {
        if(s==null||s.length()<2)return s;
        this.n=s.length();
        for(int i=0;i<n;i++){
            
            moreLength(s,i,i);//字符串长度为奇数，以i为中心
            moreLength(s,i,i+1);//字符串长度为偶数，以i，i+1为中心
        }
        return s.substring(first,second);
    }
    private void moreLength(String s,int i,int j){
        count=0;
        while(i>=0&&j<=n-1&&s.charAt(i)==s.charAt(j)){
            i--;
            j++;
        }
        count=j-i-1;
        if(count>length){
            length=count;
            first=i+1;
            second=j;
        }
    }
}
```

#### [10. 正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

给你一个字符串 `s` 和一个字符规律 `p`，请你来实现一个支持 `'.'` 和 `'*'` 的正则表达式匹配。

```
'.' 匹配任意单个字符
'*' 匹配零个或多个前面的那一个元素
```

所谓匹配，是要涵盖 **整个** 字符串 `s`的，而不是部分字符串。

**说明:**

- `s` 可能为空，且只包含从 `a-z` 的小写字母。
- `p` 可能为空，且只包含从 `a-z` 的小写字母，以及字符 `.` 和 `*`。

**示例 1:**

```
输入:
s = "aa"
p = "a"
输出: false
解释: "a" 无法匹配 "aa" 整个字符串。
```

**示例 2:**

```
输入:
s = "aa"
p = "a*"
输出: true
解释: 因为 '*' 代表可以匹配零个或多个前面的那一个元素, 在这里前面的元素就是 'a'。因此，字符串 "aa" 可被视为 'a' 重复了一次。
```

**示例 3:**

```
输入:
s = "ab"
p = ".*"
输出: true
解释: ".*" 表示可匹配零个或多个（'*'）任意字符（'.'）。
```



暴力法：击败5%；

动态规划：我写的79%(不知道为什么有这个差距?)

动态规划：别人的91%；

```java
class Solution {
    public boolean isMatch(String s, String p) {
       int m=s.length(),n=p.length();
       return doBoolean(s,0,m,p,0,n); 
    }
    private boolean doBoolean(String s,int begin1,int end1,String p,int begin2,int end2){
        if(begin1==end1&&begin2==end2){
            return true;
        }
        if(begin1!=end1&&begin2==end2){
            return false;
        }
        if(begin2<end2-1&&p.charAt(begin2+1)=='*'){
            if(begin1!=end1&&s.charAt(begin1)==p.charAt(begin2)||p.charAt(begin2)=='.'&&begin1!=end1){
                return doBoolean(s,begin1,end1,p,begin2+2,end2)||doBoolean(s,begin1+1,end1,p,begin2+2,end2)||doBoolean(s,begin1+1,end1,p,begin2,end2);
            }else{
                return doBoolean(s,begin1,end1,p,begin2+2,end2);
            }
        }
        if(begin1!=end1&&s.charAt(begin1)==p.charAt(begin2)||p.charAt(begin2)=='.'&&begin1!=end1){
                return doBoolean(s,begin1+1,end1,p,begin2+1,end2);
        }
        
        return false;
    }
}
```

```java
class Solution {
    public boolean isMatch(String s, String p) {
       int m=s.length(),n=p.length();
       boolean[][] dp=new boolean[m+1][n+1];
       dp[0][0]=true;
       for(int j=1;j<=n;j++){
           if(p.charAt(j-1)=='*'&&dp[0][j-2]){
               dp[0][j]=true;
           }
       }
       for(int i=1;i<=m;i++){
           for(int j=1;j<=n;j++){
               if(s.charAt(i-1)==p.charAt(j-1)){
                   dp[i][j]=dp[i-1][j-1];
               }else{
                   if(p.charAt(j-1)=='.'){
                       dp[i][j]=dp[i-1][j-1];
                   }else if(p.charAt(j-1)=='*'&&j>=2){
                       if(p.charAt(j-2)==s.charAt(i-1)||p.charAt(j-2)=='.'){
                           dp[i][j]=dp[i][j-1]||dp[i-1][j];
                       }
                       dp[i][j]=dp[i][j-2]||dp[i][j];
                       
                   }else{
                       dp[i][j]=false;
                   }
               }
           }
       }
       return dp[m][n];
    }
   
}
```

```java
首先定义状态：
dp[i][j] ：s前i个字符[0,i)是否能匹配p的前j个字符[0,j)。要明确一点，这里是左闭右开的，因此此时是在比较s[i-1]与p[i-1]。

确定状态方程之前先明确都会出现什么情况。首先就是s[i-1]与p[j-1]之间的关系，可能相等，可能不等。不等的情况下有三种情况：（1）p[j-1]为'*'，（2）p[j-1]为'.'（3）p[j-1]就是普通的一个字符。接下来对这些情况进行具体的解释。

完整思路
1.s[i-1]==p[j-1]
dp[i][j] = dp[i-1][j-1];
2.s[i-1]!=p[j-1]
1)p[j-1]=='.'
'.'是万能字符，可以直接让'.'等于s[i]处的字符
dp[i][j] = dp[i-1][j-1];

2)p[j-1]=='*'
'*'可以匹配零个或多个前面的元素，而是否能取多个或1个字符要看j-2的字符是否和i-1的字符相同。因此首先要判断p[j-2]==s[i-1]

(1)p[j-2]!=s[i-1]
j-2的字符不等于i-1的字符，那就只能让*代表取0个字符。
dp[i][j] = dp[i][j-2] (相当于去掉p[j-1]和p[j-2])

(2)p[j-2]==s[i-1]||p[j-2]=='.'
可以让*代表0个字符或多个字符，如果p[j-2]为'.'就可以替换为s[i-1]的字符

'*'取0个字符
例子：s:aab,p:aabb*,虽然j-2和i-1相等，但是dp[i][j-2]已经匹配了，直接删去j-1和j-2即可（你来之前我们就已经是总冠军了）
dp[i][j] = dp[i][j-2] (取0个字符)
                                  
'*'取1个字符
例子：s:aab,p:aab*
dp[i][j] = dp[i][j-1] (取1个字符，相当于去掉p[j-1])

'*'取多个字符
例子：s：aabb,p：aab*,要判断的是s:aab和aab* 是否可以匹配，如果可以匹配的话，那么s后面再加上一个b也没关系，因为*可以变成多个b。
dp[i][j] = dp[i-1][j] (取多个字符)
                                  
3)else（j处就是个普通字符，dp[i][j]肯定不能匹配了,其实这里写不写都可以，只不过为了让大家看着思路清晰。）
dp[i][j] = false;
注意
最终要确定三点。
1.本题必须初始化第一行。也就是dp[0][j]。判断条件为：

//p[j-1]为*可以把j-2和j-1处的字符删去，只有[0,j-3]也为true才可以让dp[0,j-1]为true。
if(p.charAt(j-1)=='*'&&dp[0][j-2])
2.以理解角度两个都是空的字符串肯定互相匹配，以代码角度，当s="a",p="a"时，dp[1][1] = dp[0][0],因此必须dp[0][0]取true

dp[0][0] = true;
3.字符串为空长度为0，只需要判断他们是否为空即可。

/*
s和p可能为空。空的长度就是0，但是这些情况都已经判断过了，只需要判断是否为null即可
if(p.length()==0&&s.length()==0)
    return true;
 */
    if(s==null||p==null)
        return false;
写在最后（总结出现‘*’的情况）
这里最难理解的就是如果p[j-1]=='*'的情况，上述的描述是把整个情况完全描绘出来了，其实可以简略一些过程。
*的情况只有两种：
1.不论p[j-2]是否等于s[i-1]都可以删除掉j-1和j-2处字符：

dp[i][j] = dp[i][j]||dp[i][j-2];
2.只有p[j-2]==s[i-1]或p[j-2]==‘.’才可以让*取1个或者多个字符：

if(nowpLast==nows||nowpLast=='.')
    dp[i][j] = dp[i-1][j]||dp[i][j-1];
完整题解
class Solution {
    public boolean isMatch(String s, String p) {
       /*
       s和p可能为空。空的长度就是0，但是这些情况都已经判断过了，只需要判断是否为null即可
       if(p.length()==0&&s.length()==0)
            return true;
            */
        if(s==null||p==null)
            return false;
       int rows = s.length();
       int columns = p.length();
       boolean[][]dp = new boolean[rows+1][columns+1];
       //s和p两个都为空，肯定是可以匹配的，同时这里取true的原因是
       //当s=a，p=a，那么dp[1][1] = dp[0][0]。因此dp[0][0]必须为true。
       dp[0][0] = true;
        for(int j=1;j<=columns;j++)
        {   
            //p[j-1]为*可以把j-2和j-1处的字符删去，只有[0,j-3]都为true才可以
            //因此dp[j-2]也要为true，才可以说明前j个为true
            if(p.charAt(j-1)=='*'&&dp[0][j-2])
                dp[0][j] = true;
        }

        for(int i=1;i<=rows;i++)
        {
            for(int j=1;j<=columns;j++)
            {
                char nows = s.charAt(i-1);
                char nowp = p.charAt(j-1);
                if(nows==nowp)
                {
                    dp[i][j] = dp[i-1][j-1];
                }else{
                    if(nowp=='.')
                        dp[i][j] = dp[i-1][j-1];
                    else if(nowp=='*')
                    {
                        //p需要能前移1个。（当前p指向的是j-1，前移1位就是j-2，因此为j>=2）
                        if(j>=2){
                            char nowpLast = p.charAt(j-2);
                            //只有p[j-2]==s[i-1]或p[j-2]==‘.’才可以让*取1个或者多个字符：
                            if(nowpLast==nows||nowpLast=='.')
                                dp[i][j] = dp[i-1][j]||dp[i][j-1];
                            //不论p[j-2]是否等于s[i-1]都可以删除掉j-1和j-2处字符：
                            dp[i][j] = dp[i][j]||dp[i][j-2];
                        }
                    }
                    else
                        dp[i][j] = false;
                }
            }
        }
        return dp[rows][columns];
    }
}
```

#### [32. 最长有效括号](https://leetcode-cn.com/problems/longest-valid-parentheses/)

给定一个只包含 `'('` 和 `')'` 的字符串，找出最长的包含有效括号的子串的长度。

**示例 1:**

```
输入: "(()"
输出: 2
解释: 最长有效括号子串为 "()"
```

**示例 2:**

```
输入: ")()())"
输出: 4
解释: 最长有效括号子串为 "()()"
```

动态规划：时间O(N)，空间O(N);

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int maxans = 0;
        int dp[] = new int[s.length()];
        for (int i = 1; i < s.length(); i++) {
            if (s.charAt(i) == ')') {
                if (s.charAt(i - 1) == '(') {
                    dp[i] = (i >= 2 ? dp[i - 2] : 0) + 2;
                } else if (i - dp[i - 1] > 0 && s.charAt(i - dp[i - 1] - 1) == '(') {
                    dp[i] = dp[i - 1] + ((i - dp[i - 1]) >= 2 ? dp[i - dp[i - 1] - 2] : 0) + 2;
                }
                maxans = Math.max(maxans, dp[i]);
            }
        }
        return maxans;
    }
}
```

栈方法：时间O(N),空间O(N);

1、First of All，栈底永远保存着当前有效子串的前一个字符的下标，像个守门员一样守在那里，所以一开始要将-1放入栈中。
2、遇到左括号就入栈；
3、遇到右括号就将栈顶元素出栈。此时，如果栈内剩下的元素不为空，则说明弹出的这个栈顶元素是一个左括号，讲真，因为栈底有保险。
4、如果栈顶元素出栈后，栈内为空，则说明刚刚弹出的这个栈顶元素就是之前的“有效子串前一位的字符下标”，守门员都没了，所以此时应该使用当前的右括号的下标入栈，更新这个“有效子串前一位的字符下标”。

```java
public class Solution {

    public int longestValidParentheses(String s) {
        int maxans = 0;
        Stack<Integer> stack = new Stack<>();
        stack.push(-1);
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                stack.push(i);
            } else {
                stack.pop();
                if (stack.empty()) {
                    stack.push(i);
                } else {
                    maxans = Math.max(maxans, i - stack.peek());
                }
            }
        }
        return maxans;
    }
}
```

双指针：时间O(N),空间O(1)；

利用两个计数器 left 和 right。首先，从左到右遍历字符串，对于遇到的每个‘(’，增加 left 计算器，否则增加 right 计数器。每当 left 计数器与 right 计数器相等时，计算当前有效字符串的长度，并且记录最长匹配串的长度。如果 right 计数器比 left 计数器大，将 left 和 right 置零。然后从右往左，执行一次类似的操作。如果 left 计数器比 right 计数器大，将 left 和 right 置零。

**将整个字符串，按照“多余的右括号”和“多余的左括号”之间的分界线，分为左右两部分。然后从左到右和从右到左，分别统计这两部分中所有的有效括号串的长度。**

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int left = 0, right = 0, maxlength = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = Math.max(maxlength, 2 * right);
            } else if (right >= left) {
                left = right = 0;
            }
        }
        left = right = 0;
        for (int i = s.length() - 1; i >= 0; i--) {
            if (s.charAt(i) == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = Math.max(maxlength, 2 * left);
            } else if (left >= right) {
                left = right = 0;
            }
        }
        return maxlength;
    }
}
```

#### [322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/)

难度中等

给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 `-1`。

**示例 1:**

```
输入: coins = [1, 2, 5], amount = 11
输出: 3 
解释: 11 = 5 + 5 + 1
```

**示例 2:**

```
输入: coins = [2], amount = 3
输出: -1
```

 

**说明**:
你可以认为每种硬币的数量是无限的。



超过30%

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        Arrays.sort(coins);
        if(amount==0) return 0;
        if(coins==null||coins[0]>amount) return -1;
        int[] dp=new int[amount+1];
        Arrays.fill(dp,amount+1);
        dp[0]=0;
        for(int i=1;i<=amount;i++){
            for(int j=0;j<coins.length;j++){
                if(i<coins[j]){
                    break;
                }else if(i==coins[j]){
                    dp[i]=1;
                }else{
                    dp[i]=Math.min(dp[i],dp[i-coins[j]]+1);
                }
            }
        }
        return dp[amount]>amount?-1:dp[amount];
    }
}
```

#### [72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

给你两个单词 *word1* 和 *word2*，请你计算出将 *word1* 转换成 *word2* 所使用的最少操作数 。

你可以对一个单词进行如下三种操作：

1. 插入一个字符
2. 删除一个字符
3. 替换一个字符

 

**示例 1：**

```
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```

**示例 2：**

```
输入：word1 = "intention", word2 = "execution"
输出：5
解释：
intention -> inention (删除 't')
inention -> enention (将 'i' 替换为 'e')
enention -> exention (将 'n' 替换为 'x')
exention -> exection (将 'n' 替换为 'c')
exection -> execution (插入 'u')
```



击败40%

```java
class Solution {
    public int minDistance(String word1, String word2) {
       int[][] dp=new int[word1.length()+1][word2.length()+1];
       for(int i=0;i<word2.length()+1;i++){
           dp[0][i]=i;
       } 
       for(int i=0;i<word1.length()+1;i++){
           dp[i][0]=i;
       }
       for(int i=1;i<=word1.length();i++){
           for(int j=1;j<=word2.length();j++){
               if(word1.charAt(i-1)==word2.charAt(j-1)){
                   dp[i][j]=dp[i-1][j-1];
               }else{
                   dp[i][j]=Math.min(Math.min(dp[i][j-1],dp[i-1][j]),dp[i-1][j-1])+1;
               }
           }
       }
       return dp[word1.length()][word2.length()];
    }
}
```

#### [44. 通配符匹配](https://leetcode-cn.com/problems/wildcard-matching/)



给定一个字符串 (`s`) 和一个字符模式 (`p`) ，实现一个支持 `'?'` 和 `'*'` 的通配符匹配。

```
'?' 可以匹配任何单个字符。
'*' 可以匹配任意字符串（包括空字符串）。
```

两个字符串**完全匹配**才算匹配成功。

**说明:**

- `s` 可能为空，且只包含从 `a-z` 的小写字母。
- `p` 可能为空，且只包含从 `a-z` 的小写字母，以及字符 `?` 和 `*`。

**示例 1:**

```
输入:
s = "aa"
p = "a"
输出: false
解释: "a" 无法匹配 "aa" 整个字符串。
```

**示例 2:**

```
输入:
s = "aa"
p = "*"
输出: true
解释: '*' 可以匹配任意字符串。
```

**示例 3:**

```
输入:
s = "cb"
p = "?a"
输出: false
解释: '?' 可以匹配 'c', 但第二个 'a' 无法匹配 'b'。
```



击败28%：

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if(s==null&&p!=null||s!=null&&p==null)return false;
        int a=s.length(),b=p.length();
        boolean[][] dp=new boolean[b+1][a+1];
        dp[0][0]=true;
        for(int i=1;i<=b;i++){
            if(p.charAt(i-1)=='*'){
                int c=1;
                while(!dp[i-1][c-1]&&c<=a) c++;//判断p.substring(0,i)中是否能与任意s.substring(0,c)相匹配，如果匹配，则'*'可以充当0个活一个或任意个字符，即dp[i][c++]后面都为true，c<=a;
                dp[i][c-1]=dp[i-1][c-1];
                while(c<=a) dp[i][c++]=true;
            }else{
                for(int j=1;j<=a;j++){
                    if(s.charAt(j-1)==p.charAt(i-1)||p.charAt(i-1)=='?'){
                        dp[i][j]=dp[i-1][j-1];
                    }
                }
            }
        }
        return dp[b][a];
    }
}	
```

#### [132. 分割回文串 II](https://leetcode-cn.com/problems/palindrome-partitioning-ii/)



给定一个字符串 *s*，将 *s* 分割成一些子串，使每个子串都是回文串。

返回符合要求的最少分割次数。

**示例:**

```
输入: "aab"
输出: 1
解释: 进行一次分割就可将 s 分割成 ["aa","b"] 这样两个回文子串。
```

步骤 1：思考状态
状态就尝试定义成题目问的那样，看看状态转移方程是否容易得到。

dp[i]：表示前缀子串 s[0:i] 分割成若干个回文子串所需要最小分割次数。

步骤 2：思考状态转移方程
思考的方向是：大问题的最优解怎么由小问题的最优解得到。

即 dp[i] 如何与 dp[i - 1]、dp[i - 2]、...、dp[0] 建立联系。

比较容易想到的是：如果 s[0:i] 本身就是一个回文串，那么不用分割，即 dp[i] = 0 ，这是首先可以判断的，否则就需要去遍历；

接下来枚举可能分割的位置：即如果 s[0:i] 本身不是一个回文串，就尝试分割，枚举分割的边界 j。

如果 s[j + 1, i] 不是回文串，尝试下一个分割边界。

如果 s[j + 1, i] 是回文串，则 dp[i] 就是在 dp[j] 的基础上多一个分割。

于是枚举 j 所有可能的位置，取所有 dp[j] 中最小的再加 1 ，就是 dp[i]。

得到状态转移方程如下：

```
dp[i] = min([dp[j] + 1 for j in range(i) if s[j + 1, i] 是回文])
```



击败47%

```java
class Solution {
    public int minCut(String s) {
       if(s==null||s.length()<2)return 0;
       int n=s.length();
       int[] dp=new int[n];
       for(int i=0;i<n;i++){
           dp[i]=i;
       }
       boolean[][] valid=new boolean[n][n];
       for(int i=0;i<n;i++){
           for(int j=0;j<=i;j++){
               if(s.charAt(i)==s.charAt(j)&&(i-j<=2||valid[j+1][i-1])){
                   valid[j][i]=true;
               }
           }
       }
       for(int i=0;i<n;i++){
           if(valid[0][i]){
               dp[i]=0;
               continue;
           }
           for(int j=0;j<i;j++){
               if(valid[j+1][i]){
                   dp[i]=Math.min(dp[j]+1,dp[i]);
               }
           }
       }
       return dp[n-1];
    }
}
```



#### [174. 地下城游戏](https://leetcode-cn.com/problems/dungeon-game/)

难度困难207收藏分享切换为英文关注反馈

一些恶魔抓住了公主（**P**）并将她关在了地下城的右下角。地下城是由 M x N 个房间组成的二维网格。我们英勇的骑士（**K**）最初被安置在左上角的房间里，他必须穿过地下城并通过对抗恶魔来拯救公主。

骑士的初始健康点数为一个正整数。如果他的健康点数在某一时刻降至 0 或以下，他会立即死亡。

有些房间由恶魔守卫，因此骑士在进入这些房间时会失去健康点数（若房间里的值为*负整数*，则表示骑士将损失健康点数）；其他房间要么是空的（房间里的值为 *0*），要么包含增加骑士健康点数的魔法球（若房间里的值为*正整数*，则表示骑士将增加健康点数）。

为了尽快到达公主，骑士决定每次只向右或向下移动一步。

 

**编写一个函数来计算确保骑士能够拯救到公主所需的最低初始健康点数。**

例如，考虑到如下布局的地下城，如果骑士遵循最佳路径 `右 -> 右 -> 下 -> 下`，则骑士的初始健康点数至少为 **7**。

| -2 (K) | -3   | 3      |
| ------ | ---- | ------ |
| -5     | -10  | 1      |
| 10     | 30   | -5 (P) |

 

**说明:**

- 骑士的健康点数没有上限。
- 任何房间都可能对骑士的健康点数造成威胁，也可能增加骑士的健康点数，包括骑士进入的左上角房间以及公主被监禁的右下角房间。

击败97%

我们先将题目的要求转述为
“在任意时刻，骑士的生命值x都大于等于1”

接下来，我们按题目要求定义，假设地牢的尺寸为{m x n}
**

DP[i][j]的含义是，使得 骑士从i,j出发走到右下角这一全部过程中血量都不小于1的 最初血量，最后求得DP[0][0]即可
**

那么是从左上到右下，还是从右下到左上呢？
我们先待定
我们假设骑士的初始生命值为x，并用a表示地牢二维矩阵

根据通常的二维数组移动的动态规划的经验
我们还是将问题先简化，分为以下三种情况

1、如果地牢的尺寸为{1x1}（仅有1个格子）
此时，我们不需要考虑子问题求解的方向
骑士只需要进入唯一房间中，并在生命值与这个房子发生交互后，还保持至少为1的血量
即

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/ab31f094d2e1885613b743a1ca9ea9f42dc427f47af2382bcdc1b1cebf2274cf-image.png)

解得

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/4dd253d52ff399f7e3825605a433f697ec85320d5fd034b4bc879702da19e86f-image.png)

由于我们需要求得初始生命可行的最低值，我们可以将max前面直接取等号

2、如果地牢的尺寸为{1 x n}或{m x 1}
此时，我们需要考虑子问题求解的方向！

我们以一个{1 x n}（1行n列）的地牢为例
这个地牢的a为

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/91ecd30136a7397a707de8039f6849f7ddc3de133a9506c91f68e14d68b63895-image.png)

假设我们从左上(a[0][0])到右下(a[0][n-1])
走过第一个格子，类似上面的”1、“
有

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/4c6481813c954edebd86f96f4a25a1dbfb3384194def4963b967328271f72207-image.png)

但此时，如果不知道a[0][1]~a[0][n-1]的值
我们便无推得dp[0][0]的值，因为我们不知道后面的格子，也就无法确定初始生命值

因此，我们必须考虑从右下往左上演化！
因此，我们先考虑若从最后一个格子a[0][n-1]出发，由于也引发一次生命值交互，
所以，类似”1、“，有

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/2e24ce2ddc914334de3fc4bdaf34c19479bedc9a3d92ec57ded6bcbc0984d1a4-image.png)

我们将起点左移至a[0][n-2]
若以生命值x从a[0][n-2]出发，则要求

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/61d8748674f33eefb7342ecd5a78fbc48c9c2734bb5459a31f4ed425621c8b91-image.png)

我们将上面的第三行移项，得到

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/73fb6762f17f45a944f8f0f1fa1c56e5efbbb99c2289b4ecee7683166b6f674d-image.png)

于是，我们可以将第二行式子和第三行式子合并，得到如下结果

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/4cbd512859459ee8377abf6e91da142d1c9283c6364bb6675097033188b10dc8-image.png)

这个时候，我们奇迹般地发现，合并后右边的式子，不就是DP[0][n-1]吗？？
因此，我们将DP[0][n-1]代入，得到

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/f1f3e8993ae19c41c348cf54757e72a0dd32e661b28f2cce209d0e4ac1b8f4c4-image.png)

我们再将第二行式子的a[0][n-2]移项，得到

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/8e1f7e493c4df37aac5bee0ae6ea7b0bebcd15d36c69ecf3110b859650347b1a-image.png)

易得

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/ccdcdabada7faf7468d70bfa06857e2754b5a76f952f29805d828ecbfb7d166a-image.png)

要想得到最低要求，取等号，得

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/1a5351c817d73e8b20c39ddc75d9fa678f963a491820d7731aa11ced81b1356c-image.png)

对于这个式子，我们推广到这种形状地牢的一般化结论

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/edf922e29d31e85f8fb47bee6ca88e5e182dac572ab987d9922aa71c1c938fa2-image.png)

同理，对于{m x 1}（m行1列）的地牢，我们能推得类似的结论

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/ecf6a00bb498f16dd5a9369acb853fa6ebc08666fc95a3b3c0a741c2a48be7e5-image.png)


这样，我们便解决了一行多列，或一列多行的情况

3、一般情况，即{m x n}(地牢有m行n列，二者都＞1)
由于我们在”2、“中已经得出结论：
在只能选择右移或者下移的前提下，
单行或单列的地牢中，每个格子的DP值只与这一行或这一列有关

即便是m行n列，
如果我们从最后一行出发，那么我们只有一条路线，即一直往右（与其它行的a的数值无关了），
如果我们从最后一列出发，那么我们只有一条路线，即一直往下（与其它列的a的数值无关了），
因此，我们将m行n列中最下面一行和最右面一列，按照”2、“中最后的两个式子初始化

也就是说，要填充DP矩阵，
我们可以根据上面得到的结论，先将下图中灰色部分初始化

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/4db527c70852f8749d01ce4a4f25ff184db6cf0682ef3631cff7fafbffd2410c-image.png)

接下来，我们从a[m-2][n-2]开始填充，可以先填充第m-2行，也可以先填充第n-2列

我们以先填充第m-2行为例，如下所示（此后，我们使用绿色表示已经初始化的格子）

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/66656b287ae4dcb38bc0b812c0b652567b0fed6119a5391485845587d501a842-image.png)

根据处理这类问题的经验，我们将子问题锁定在一个局部的2x2格子中
按照上图的顺序，我们每一次操作都能保证待求DP值的格子i,j的下方、右方和右下方的DP值都是已知
如下图所示

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/eb795c65fa526c1da8da317fd39238e019ff0d1477af52dadd1df05f85c0ed44-image.png)

则 从i,j到i+1,j+1，我们只有两种选择，分别是
（1）i,j —— 下移——> i+1,j—— 右移——>i+1,j+1
（2）i,j —— 右移——> i,j+1—— 下移——>i+1,j+1
若为（1），
假设初始生命为x,有

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/8eb59680483dfa2f86b9f6772e87fbf942e103fe5e62f6f89dce5b624c170f2b-image.png)

它的含义为，初始生命不低于1，经过i,j还不低于1且不低于i+1,j所需的最低要求值

因此

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/ce703351c519ee9ef9210f37fa0cf603619d1df377a7400de9f9fba4b2f9a05e-image.png)

考虑到上面的递推公式，能确保已经求得的DP均不小于1（也可理解为最初血量必定不小于1）
上式可简化为

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/74ba9bedb90b04452845f6f2ec1443bd8c2b27b4dc45d1a1a6edf64f25ceb32f-image.png)

若为(2)，同理，有

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/763e5679a8cabb957c24b192e867008d6b15b86e6ff94bcd329f2675f3800fd9-image.png)

接下来是关键，因为有两条道路可以走，我们要寻得的是最低要求值，
因此，只要有一条满足生命值大于等于1即可，因此，我们将上面两个不等式取或，
即让x大于等于两个max中较小的一个即可

即，当 0 ≤ i ≤ m - 2， 0 ≤ j ≤ n - 2时

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/79af71fe8372fbdbaad4d1f5d2fc25bea0f4cbf12a098a12d7e19e03133623d4-image.png)

当i = m - 1时

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/5c980f69f885b3bf76f33474c40267bbca7e4595f6940af55ec5a48e971cd932-image.png)

当j = n - 1时

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/ca54ce9b37c32c9dcb4cc40ad99f4f4d14e51691917704d41c9eefc35694e7a8-image.png)

这样，我们便得到了动态规划的状态转移方程

```java
class Solution {
    
    public int calculateMinimumHP(int[][] dungeon) {
    if(dungeon==null||dungeon.length==0||dungeon[0].length==0)return 0;
    int m=dungeon.length,n=dungeon[0].length;
    int[][] dp=new int[m][n];
    dp[m-1][n-1]=Math.max(1,-1*dungeon[m-1][n-1]+1);
    for(int i=m-2;i>=0;i--){
        dp[i][n-1]=Math.max(1,dp[i+1][n-1]-dungeon[i][n-1]);
    }
    for(int j=n-2;j>=0;j--){
        dp[m-1][j]=Math.max(1,dp[m-1][j+1]-dungeon[m-1][j]);
    }
    for(int i=m-2;i>=0;i--){
        for(int j=n-2;j>=0;j--){
            dp[i][j]=Math.max(1,Math.min(dp[i+1][j],dp[i][j+1])-dungeon[i][j]);
        }
    }
    return dp[0][0];
    }
}
```



#### [629. K个逆序对数组](https://leetcode-cn.com/problems/k-inverse-pairs-array/)

难度困难48收藏分享切换为英文关注反馈

给出两个整数 `n` 和 `k`，找出所有包含从 `1` 到 `n` 的数字，且恰好拥有 `k` 个逆序对的不同的数组的个数。

逆序对的定义如下：对于数组的第`i`个和第 `j`个元素，如果满`i` < `j`且 `a[i]` > `a[j]`，则其为一个逆序对；否则不是。

由于答案可能很大，只需要返回 答案 mod 109 + 7 的值。

**示例 1:**

```
输入: n = 3, k = 0
输出: 1
解释: 
只有数组 [1,2,3] 包含了从1到3的整数并且正好拥有 0 个逆序对。
```

**示例 2:**

```
输入: n = 3, k = 1
输出: 2
解释: 
数组 [1,3,2] 和 [2,1,3] 都有 1 个逆序对。
```

**说明:**

1.  `n` 的范围是 [1, 1000] 并且 `k` 的范围是 [0, 1000]。

击败36%

动态规划解法：
我们用 f(i, j) 表示数字 [1 .. i] 的排列中恰好包含 j 个逆序对的个数。在状态转移时，我们考虑数 i 放置的位置与逆序对个数的关系。我们在数字 [1 .. i - 1] 组成的排列中放入 i 时，有 i 种放置方法：如果将 i 放在最后，则逆序对数量不变；如果将 i 放在倒数第二个，则逆序对数量增加 1；如果将 i 放在第一个，则逆序对数量增加 i - 1。这是因为 i 是 [1 .. i] 中的最大值，因此将它放置在 [1 .. i - 1] 的排列中的任意一个位置，它都会与在它之后的那些数形成逆序对。如果它后面有 k 个数，则会形成 k 个逆序对。

因此我们可以写出状态转移方程：

```java
f(i, j) = f(i - 1, j) + f(i - 1, j - 1) + ... + f(i - 1, j - i + 1)
```

边界条件为:

```java
f(i, j0) = 0 if j0 < 0
f(0, 0) = 1
```

这个动态规划的时间复杂度为 O(N^2*K)，因此我们需要继续优化。可以发现，状态转移方程中的右侧是一段连续的和，我们将 j 变为 j - 1，有：

```java
f(i, j - 1) = f(i - 1, j - 1) + f(i - 1, j - 2) + ... + f(i - 1, j - i)
```

将 `f(i, j)` 与 `f(i, j - 1)` 相比较，可以得到：

```java
f(i, j) - f(i - 1, j) = f(i, j - 1) - f(i - 1, j - i)
==> f(i, j) = f(i, j - 1) + f(i - 1, j) - f(i - 1, j - i)
```

此时使用这个状态转移方程，动态规划的时间复杂度降低为 O(N*K)*。

![img](%E7%AE%97%E6%B3%95leetcode.assets/629_dp6Slide21.PNG)

```java
class Solution {
    public int kInversePairs(int n, int k) {
        int[][] dp=new int[n+1][k+1];
        int m=1000000007;
        for(int i=1;i<=n;i++){
            int top=(i-1)*i/2;
            for(int j=0;j<=k&&j<=top;j++){
                if(i==1&&j==0){
                    dp[i][j]=1;
                    break;
                }else if(j==0){
                    dp[i][j]=1;
                }else{
                    //我们的每个dp[i][j]里面存储的都是%m后的非负数，所以为了避免2个数相减会导致负数出现，我们加了m，再去取余数，然后再对2个数进行运算取余。
                    int val=(dp[i][j-1]+m-((j-i)>=0?dp[i-1][j-i]:0))%m;
                    dp[i][j]=(dp[i-1][j]+val)%m;
                }

            }
        }
    return dp[n][k];
    }
}
```



#### [730. 统计不同回文子序列](https://leetcode-cn.com/problems/count-different-palindromic-subsequences/)

难度困难49收藏分享切换为英文关注反馈

给定一个字符串 S，找出 S 中不同的非空回文子序列个数，并**返回该数字与 `10^9 + 7 `的模。**

通过从 S 中删除 0 个或多个字符来获得子序列。

如果一个字符序列与它反转后的字符序列一致，那么它是回文字符序列。

如果对于某个 `i`，`A_i != B_i`，那么 `A_1, A_2, ...` 和 `B_1, B_2, ...` 这两个字符序列是不同的。

 

**示例 1：**

```
输入：
S = 'bccb'
输出：6
解释：
6 个不同的非空回文子字符序列分别为：'b', 'c', 'bb', 'cc', 'bcb', 'bccb'。
注意：'bcb' 虽然出现两次但仅计数一次。
```

**示例 2：**

```
输入：
S = 'abcdabcdabcdabcdabcdabcdabcdabcddcbadcbadcbadcbadcbadcbadcbadcba'
输出：104860361
解释：
共有 3104860382 个不同的非空回文子序列，对 10^9 + 7 取模为 104860361。
```

 

**提示：**

- 字符串 `S` 的长度将在`[1, 1000]`范围内。
- 每个字符 `S[i]` 将会是集合 `{'a', 'b', 'c', 'd'}` 中的某一个。

击败56%



```java
//非常巧妙的DP题
    //dp[i][j]表示s[i...j]的不同回文子序列个数
    //怎么去掉重复部分?
    //如果s[i] != s[j], dp[i][j] = dp[i + 1][j] + dp[i][j - 1] - dp[i + 1][j - 1] (中间计算两遍的减掉)
    //如果s[i] == s[j], 找在i右边的S[i]相同的第一个位置left,在j右边的和S[j]相同的第一个位置right
    //通过left和right的关系,判断s[i +1] ~ s[j - 1]有多少个S[i]
    //如果有0个,那么dp[i][j] = (dp[i + 1][j - 1] * 2 + 2)  其中的2表示单个s[i]以及s[i]+s[i]这两个情况
    //如果有1个,那么dp[i][j] = (dp[i + 1][j - 1] * 2 + 1)  其中的1表示s[i]+s[i]
    //如果有多个,那么dp[i][j] = (dp[i + 1][j - 1] * 2 - dp[left + 1][right - 1])


//如果 S[i] == S[j]，这时我们需要判断[i, j]这一段中有多少字符与S[i]不相等
	//如果中间没有和S[i]相同的字母，例如"aba"这种情况，dp[i][j] = dp[i + 1][j - 1] * 2 + 2;（dp[i][j] = dp[i + 1][j - 1] * 2 代表的dp[i + 1[j - 1]这一段可以独立存在，也可在外层包裹S[i],S[j]，所有需要x2，而2是代表“aa”和“a”）
	//如果中间只有一个和S[i]相同的字母，就是"aaa"这种情况，dp[i][j] = dp[i + 1][j - 1] * 2 + 1;（x2与上面情况相同，加一单独计算"aa",而“a”在dp[i + 1][j - 1] 中计算过了）
	//否则中间至少有两个和S[i]相同的字母，就是"aabaa"这种情况，dp[i][j] = dp[i + 1][j - 1] * 2 - dp[left + 1][right - 1];（left、right请见代码注释，dp[left + 1][right - 1]这一段重复计算了）
//否则dp[i][j] = dp[i][j - 1] + dp[i + 1][j] - dp[i + 1][j - 1];

class Solution {
    public int countPalindromicSubsequences(String S) {
      int n=S.length();
      int m=1000000007;
      int[][] dp=new int[n][n];
      char[] s=S.toCharArray();
      for(int i=0;i<n;i++){
          dp[i][i]=1;
      }  
      for(int l=2;l<=n;l++){
          for(int i=0;i<n-1;i++){
              int j=i+l-1;
              if(j>=n){
                  break;
              }
              if(s[i]==s[j]){
                  int left=i+1,right=j-1;
                  while(left<j&&s[left]!=s[i]){
                      left++;
                  }
                  while(right>i&&s[right]!=s[j]){
                      right--;
                  }
                  if(left>right){
                      dp[i][j]=(dp[i+1][j-1]*2+2)%m;
                  }else if(left==right){
                      dp[i][j]=(dp[i+1][j-1]*2+1)%m;
                  }else{
                      dp[i][j]=(dp[i+1][j-1]*2%m+(m-dp[left+1][right-1])%m)%m;
                  }
              }else{
                  dp[i][j]=(dp[i+1][j]+dp[i][j-1])%m;
                  dp[i][j]=(dp[i][j]+(m-dp[i+1][j-1])%m)%m;
              }
          }
      }
      return dp[0][n-1];
    }
}
```



#### [546. 移除盒子](https://leetcode-cn.com/problems/remove-boxes/)

难度困难106收藏分享切换为英文关注反馈

给出一些不同颜色的盒子，盒子的颜色由数字表示，即不同的数字表示不同的颜色。
你将经过若干轮操作去去掉盒子，直到所有的盒子都去掉为止。每一轮你可以移除具有相同颜色的连续 k 个盒子（k >= 1），这样一轮之后你将得到 `k*k` 个积分。
当你将所有盒子都去掉之后，求你能获得的最大积分和。
输入:

```
[1, 3, 2, 2, 2, 3, 4, 3, 1]
```

输出:

```
23
```

解释:

```
[1, 3, 2, 2, 2, 3, 4, 3, 1] 
----> [1, 3, 3, 4, 3, 1] (3*3=9 分) 
----> [1, 3, 3, 3, 1] (1*1=1 分) 
----> [1, 1] (3*3=9 分) 
----> [] (2*2=4 分)
```

**提示：**盒子的总数 `n` 不会超过 100。

击败28%



```java
/*刷题老司机们都会优先考虑用DP来做吧。既然要玩子数组，肯定要限定子数组的范围，那么至少应该是个二维的dp数组，其中dp[i][j]表示在子数组[i, j]范围内所能得到的最高的分数，那么最后我们返回dp[0][n-1]就是要求的结果。

那么对于dp[i][j]我们想，如果我们移除boxes[i]这个数字，那么总得分应该是1 + dp[i+1][j]，但是通过分析题目中的例子，能够获得高积分的trick是，移除某个或某几个数字后，如果能使得原本不连续的相同数字变的连续是坠好的，因为同时移除的数字越多，那么所得的积分就越高。那么假如在[i, j]中间有个位置m，使得boxes[i]和boxes[m]相等，那么我们就不应该只是移除boxes[i]这个数字，而是还应该考虑直接移除[i+1, m-1]区间上的数，使得boxes[i]和boxes[m]直接相邻，那么我们获得的积分就是dp[i+1][m-1]，那么我们剩余了什么，boxes[i]和boxes[m, j]区间的数，此时我们无法处理子数组[m, j]，因为我们有些信息没有包括在我们的dp数组中，此类的题目归纳为不自己包含的子问题，其解法依赖于一些子问题以外的信息。这类问题通常没有定义好的重现关系，所以不太容易递归求解。为了解决这类问题，我们需要修改问题的定义，使得其包含一些外部信息，从而变成自包含子问题。

那么对于这道题来说，无法处理boxes[m, j]区间是因为其缺少了关键信息，我们不知道boxes[m]左边相同数字的个数k，只有知道了这个信息，那么m的位置才有意义，所以我们的dp数组应该是一个三维数组dp[i][j][k]，表示区间[i, j]中能获得的最大积分，当boxes[i]左边有k个数字跟其相等，那么我们的目标就是要求dp[0][n-1][0]了，而且我们也能推出dp[i][i][k] = (1+k) * (1+k)这个等式。那么我们来推导重现关系，对于dp[i][j][k]，如果我们移除boxes[i]，那么我们得到(1+k)*(1+k) + dp[i+1][j][0]。对于上面提到的那种情况，当某个位置m，有boxes[i] == boxes[m]时，我们也应该考虑先移除[i+1,m-1]这部分，我们得到积分dp[i+1][m-1][0]，然后再处理剩下的部分，得到积分dp[m][j][k+1]，这里k加1点原因是，移除了中间的部分后，原本和boxes[m]不相邻的boxes[i]现在相邻了，又因为二者值相同，所以k应该加1，因为k的定义就是左边相等的数字的个数。讲到这里，那么DP方法最难的递推公式也就得到了，那么代码就不难写了，而题目中说了数组长度不会超100，所以我们就用100来初始化，参见代码如下：*/
class Solution {
    public int removeBoxes(int[] boxes) {
        int n=boxes.length;
     if(n<=1)return n;
        int[][][] dp=new int[100][100][100];
        return help(boxes,0,n-1,0,dp);
    }
    private int help(int[] boxes,int i,int j,int k,int[][][] dp){
        if(i>j)return 0;
        if(dp[i][j][k]>0)return dp[i][j][k];
        while(i<j&&boxes[i]==boxes[i+1]){//直接筛选出连续的
            i=i+1;
            k=k+1;
        }
        int res=0;
        res=Math.max(res,(1+k)*(1+k)+help(boxes,i+1,j,0,dp));
        for(int m=i+1;m<=j;m++){
            if(boxes[m]==boxes[i]){
       //下面的help(boxes,i+1,m-1,0,dp)中k为0是因为把这段拿出来单独算，所以左边为0；         
            res=Math.max(res,help(boxes,i+1,m-1,0,dp)+help(boxes,m,j,k+1,dp));
            }
        }
        dp[i][j][k]=res;
        return res;
    }
}
```

#### [664. 奇怪的打印机](https://leetcode-cn.com/problems/strange-printer/)

难度困难56收藏分享切换为英文关注反馈

有台奇怪的打印机有以下两个特殊要求：

1. 打印机每次只能打印同一个字符序列。
2. 每次可以在任意起始和结束位置打印新字符，并且会覆盖掉原来已有的字符。

给定一个只包含小写英文字母的字符串，你的任务是计算这个打印机打印它需要的最少次数。

**示例 1:**

```
输入: "aaabbb"
输出: 2
解释: 首先打印 "aaa" 然后打印 "bbb"。
```

**示例 2:**

```
输入: "aba"
输出: 2
解释: 首先打印 "aaa" 然后在第二个位置打印 "b" 覆盖掉原来的字符 'a'。
```

**提示**: 输入字符串的长度不会超过 100。

击败86%

由于题目是包含相邻重复项的，但是相邻重复项其实没什么意义，如bbaacc和bac是同理的，所以……可以做事前工作将相邻相通项去除，当然由于去除了相邻相通项，所以距离为2的元素一定至少需要2步打印。

```java
//首先去除相邻的重复元素，得到去重后的字符串s；
//对该s进行动态规划。对于区间[i, j]，因为第一个字符s[i]肯定会首先打印，可以将dp[i][j]初始化为res = 1 + dp[i + 1][j]。然后遍历该区间内的字符，如果s[i] == s[k]，如果i到k位置的字符能够一次打印，则问题能分解成dp[i][k - 1] + dp[k + 1][j]。取最小值dp[i][j] = min(res, dp[i][k - 1] + dp[k + 1][j])。
class Solution {
    private int[][] dp;
    public int strangePrinter(String s) {
      if(s==null||s.length()==0)return 0;
      char pre=s.charAt(0);
      StringBuilder sb=new StringBuilder();
      sb.append(pre);
      int n=s.length();
      for(int i=1;i<n;i++){
          char cur=s.charAt(i);
          if(pre!=cur){
              sb.append(cur);
          }
          pre=cur;
      }
      String str=sb.toString();
      int len=str.length();
      dp=new int[len][len];
      return dfs(0,len-1,str);
    }
    private int dfs(int i,int j,String s){
        if(i>j)return 0;
        if(dp[i][j]==0){
            int res=1+dfs(i+1,j,s);
            for(int m=i+1;m<=j;m++){
                if(s.charAt(m)==s.charAt(i)){
                res=Math.min(res,dfs(i,m-1,s)+dfs(m+1,j,s));
                }
            }
            dp[i][j]=res;
        }
        return dp[i][j];
    }
}
```



# 分治

#### [241. 为运算表达式设计优先级](https://leetcode-cn.com/problems/different-ways-to-add-parentheses/)

给定一个含有数字和运算符的字符串，为表达式添加括号，改变其运算优先级以求出不同的结果。你需要给出所有可能的组合的结果。有效的运算符号包含 `+`, `-` 以及 `*` 。

**示例 1:**

```
输入: "2-1-1"
输出: [0, 2]
解释: 
((2-1)-1) = 0 
(2-(1-1)) = 2
```

**示例 2:**

```
输入: "2*3-4*5"
输出: [-34, -14, -10, -10, 10]
解释: 
(2*(3-(4*5))) = -34 
((2*3)-(4*5)) = -14 
((2*(3-4))*5) = -10 
(2*((3-4)*5)) = -10 
(((2*3)-4)*5) = 10
```



```java
class Solution {
    HashMap<String,List<Integer>> map=new HashMap<>();
    public List<Integer> diffWaysToCompute(String input) {
        List<Integer> list=new ArrayList<>();
        if(input.length()==0) return list;
        if(map.containsKey(input)) return map.get(input);
        for(int i=0;i<input.length();i++){
            char c=input.charAt(i);
            if(c=='+'||c=='-'||c=='*'){
                List<Integer> left=diffWaysToCompute(input.substring(0,i));
                List<Integer> right=diffWaysToCompute(input.substring(i+1));
                for(int l:left){
                    for(int r:right){
                        switch(c){
                            case '+':
                            list.add(l+r);
                            break;
                            case '-':
                            list.add(l-r);
                            break;
                            case '*':
                            list.add(l*r);
                            break;
                        }
                    }
                }
            }
            
        }
        if(list.size()==0){
                list.add(Integer.valueOf(input));
            }
        map.put(input,list);
        return list;
        
    }
}
```

#### [95. 不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

难度中等362收藏分享切换为英文关注反馈

给定一个整数 *n*，生成所有由 1 ... *n* 为节点所组成的**二叉搜索树**。

**示例:**

```
输入: 3
输出:
[
  [1,null,3,2],
  [3,2,null,1],
  [3,1,null,null,2],
  [2,1,3],
  [1,null,2,null,3]
]
解释:
以上的输出对应以下 5 种不同结构的二叉搜索树：

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```



击败77%

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
    public List<TreeNode> generateTrees(int n) {
      
      if(n<1)   return new LinkedList<TreeNode>();
      
      return help(1,n);
    }
    private List<TreeNode> help(int start,int end){
        List<TreeNode> all_treeNode=new LinkedList<TreeNode>();
        //此处如果不初始化LinkedList<>()为LinkedList<TreeNode>()，速度将会大大降低，只击败22%；待研究🐷
        if(start>end){
            all_treeNode.add(null);
            return all_treeNode;
        }
        for(int i=start;i<=end;i++){
            List<TreeNode> left=help(start,i-1);
            List<TreeNode> right=help(i+1,end);
            for(TreeNode l:left){
                for(TreeNode r:right){
                    TreeNode root=new TreeNode(i);
                    root.left=l;
                    root.right=r;
                    all_treeNode.add(root);
                }
            }
        }
        return all_treeNode;

        
    }
}
```

#### [23. 合并K个排序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)



合并 *k* 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。

**示例:**

```
输入:
[
  1->4->5,
  1->3->4,
  2->6
]
输出: 1->1->2->3->4->4->5->6
```



本人击败33%，思路对的，也是分治思想，但是ArrayList删除太浪费时间了

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
      if(lists.length==0) return null;
      if(lists.length==1) return lists[0];
      List<ListNode> list=new ArrayList<>();
      for(int i=0;i<lists.length;i++){
          list.add(lists[i]);
      }
      while(list.size()>1){
          help(list);
      }
      return list.get(0);  
    }
    private void help(List<ListNode> list){
        ListNode list1=list.get(0);
        ListNode list2=list.get(1);
        ListNode list33=new ListNode(-1);
        ListNode list3=list33;
        while(list1!=null||list2!=null){
            
            if(list1==null){
                list3.next=new ListNode(list2.val);
                list3=list3.next;
                list2=list2.next;
            }else if(list2==null){
                list3.next=new ListNode(list1.val);
                list3=list3.next;
                list1=list1.next;
            }else if(list1.val>list2.val){
                list3.next=new ListNode(list2.val);
                list3=list3.next;
                list2=list2.next;
            }else{
                list3.next=new ListNode(list1.val);
                list3=list3.next;
                list1=list1.next;
            }
        }
        list.remove(0);
        list.remove(0);
        list.add(list33.next);
    }
}
```



击败85%

```java
public ListNode mergeKLists(ListNode[] lists){
        if(lists.length == 0)
            return null;
        if(lists.length == 1)
            return lists[0];
        if(lists.length == 2){
           return mergeTwoLists(lists[0],lists[1]);
        }

        int mid = lists.length/2;
        ListNode[] l1 = new ListNode[mid];
        for(int i = 0; i < mid; i++){
            l1[i] = lists[i];
        }

        ListNode[] l2 = new ListNode[lists.length-mid];
        for(int i = mid,j=0; i < lists.length; i++,j++){
            l2[j] = lists[i];
        }

        return mergeTwoLists(mergeKLists(l1),mergeKLists(l2));

    }
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;

        ListNode head = null;
        if (l1.val <= l2.val){
            head = l1;
            head.next = mergeTwoLists(l1.next, l2);
        } else {
            head = l2;
            head.next = mergeTwoLists(l1, l2.next);
        }
        return head;
    }
```

最小堆实现击败64%

```java
 public ListNode mergeKLists(ListNode[] lists) {

        if (lists.length == 0) {
            return null;
        }

        ListNode dummyHead = new ListNode(0);
        ListNode curr = dummyHead;
        PriorityQueue<ListNode> pq = new PriorityQueue<>(new Comparator<ListNode>() {
            @Override
            public int compare(ListNode o1, ListNode o2) {
                return o1.val - o2.val;
            }
        });

        for (ListNode list : lists) {
            if (list == null) {
                continue;
            }
            pq.add(list);
        }

        while (!pq.isEmpty()) {
            ListNode nextNode = pq.poll();
            curr.next = nextNode;
            curr = curr.next;
            if (nextNode.next != null) {
                pq.add(nextNode.next);
            }
        }
        return dummyHead.next;
    }
```



# 回溯:

#### [282. 给表达式添加运算符](https://leetcode-cn.com/problems/expression-add-operators/)

难度困难92收藏分享切换为英文关注反馈

给定一个仅包含数字 `0-9` 的字符串和一个目标值，在数字之间添加**二元**运算符（不是一元）`+`、`-` 或 `*` ，返回所有能够得到目标值的表达式。

**示例 1:**

```
输入: num = "123", target = 6
输出: ["1+2+3", "1*2*3"] 
```

**示例 2:**

```
输入: num = "232", target = 8
输出: ["2*3+2", "2+3*2"]
```

**示例 3:**

```
输入: num = "105", target = 5
输出: ["1*0+5","10-5"]
```

**示例 4:**

```
输入: num = "00", target = 0
输出: ["0+0", "0-0", "0*0"]
```

**示例 5:**

```
输入: num = "3456237490", target = 9191
输出: []
```



击败69%：

```java
class Solution {

  public List<String> addOperators(String num, int target) {
      List<String> list=new ArrayList<>();
      if(num==null||num.length()==0) return list;
      help(num,target,list,new StringBuilder(),0,0,0);
      return list;
  }
  private void help(String num,int target,List<String> list,StringBuilder sb,int index,long res,long pre){
      if(index==num.length()){
          if(target==res){
              list.add(sb.toString());
          }
          return;
      }
      for(int i=index;i<num.length();i++){
          //去除0开头数字(多位)
          if(num.charAt(index)=='0'&&i>index){
              break;
          }
          long cur=Long.parseLong(num.substring(index,i+1));
          int length=sb.length();
          if(index==0){
              help(num,target,list,sb.append(cur),i+1,cur,cur);
              sb.setLength(length);
          }else{
              help(num,target,list,sb.append("+").append(cur),i+1,res+cur,cur);
              sb.setLength(length);
              help(num,target,list,sb.append("-").append(cur),i+1,res-cur,-cur);
              sb.setLength(length);
              help(num,target,list,sb.append("*").append(cur),i+1,res-pre+pre*cur,pre*cur);
              sb.setLength(length);
          }
      }
  }
}

```



#### [17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)



给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/original_images/17_telephone_keypad.png)

**示例:**

```
输入："23"
输出：["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
```

击败100%

```java
class Solution {
    private String[] matrix=new String[]{"","","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
    
    public List<String> letterCombinations(String digits) {
        List<String> list=new ArrayList<>();
        int n=digits.length();
        if(n<1)return list;
        doMatch(list,new StringBuilder(),digits,0,n);
        return list;
    }
    private void doMatch(List<String> list,StringBuilder sb,String digits,int begin,int end){
        if(sb.length()==end){
            list.add(sb.toString());
            return;
        }
        String s=matrix[digits.charAt(begin)-'0'];//注意此处digits.charAt(begin)-'0',这样才能把'2'转换为2，后面不减去'0'，就会把'2'转换成50，把'2'(char)转成对应的ASCII码值
        for(int i=0;i<s.length();i++){
            sb.append(s.charAt(i));
            doMatch(list,sb,digits,begin+1,end);
            sb.deleteCharAt(sb.length()-1);
        }
        return;
    }
}
```

#### [22. 括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

数字 *n* 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

**示例：**

```
输入：n = 3
输出：[
       "((()))",
       "(()())",
       "(())()",
       "()(())",
       "()()()"
     ]
```



击败97.5%

```java
class Solution {
    public List<String> generateParenthesis(int n) {
      List<String> list=new ArrayList<>();
      if(n<1)return list;
      help(list,new StringBuilder(),n,0,0);
      return list;
    }
    private void help(List<String> list,StringBuilder sb,int n,int leftIndex,int rightIndex){
        if(sb.length()==2*n){
            list.add(new String(sb));
            return;
        }
        if(leftIndex<n){
            
            sb.append("(");
            help(list,sb,n,leftIndex+1,rightIndex);
            sb.deleteCharAt(sb.length()-1);
        } //右括号出现的数量少于左括号时才可以增加右括号
        if(rightIndex<leftIndex){
            sb.append(")");
            help(list,sb,n,leftIndex,rightIndex+1);
            sb.deleteCharAt(sb.length()-1); 
            }
        

        return ;
    }
}
```

#### [78. 子集](https://leetcode-cn.com/problems/subsets/)



给定一组**不含重复元素**的整数数组 *nums*，返回该数组所有可能的子集（幂集）。

**说明：**解集不能包含重复的子集。

**示例:**

```
输入: nums = [1,2,3]
输出:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```

击败99.7%

```java
class Solution {
    private List<List<Integer>> list=new ArrayList<>();
    public List<List<Integer>> subsets(int[] nums) {
        Arrays.sort(nums);
        if(nums==null||nums.length==0)return list;
        for(int l=1;l<=nums.length;l++){
            help(nums,new ArrayList<>(),l,0);
        }
        list.add(new ArrayList<>());
        return list;
    }
    private void help(int[] nums,List<Integer> list1,int l,int start){
        if(list1.size()==l){
            list.add(new ArrayList<>(list1));
            return;
        }
        for(int i=start;i<nums.length;i++){
            list1.add(nums[i]);
            help(nums,list1,l,i+1);
            list1.remove(list1.size()-1);
        }
        return;
    }
}
```



# 双指针

#### [42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)



给定 *n* 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/rainwatertrap.png)

上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。

```
输入: [0,1,0,2,1,0,1,3,2,1,2,1]
输出: 6
```



击败99%

```java
class Solution { 
    public int trap(int[] height) {
        int left=0,right=height.length-1;
        int leftMax=0,rightMax=0;
        int ans=0;
        while(left<right){
            if(height[left]<height[right]){
                //进入此处时，左边的值小于右边，才有机会设置leftMax，此时leftMax小于右边的数，必然小于rightMax;此时他只要与左边的leftMax比较即可；
                if((height[left]>leftMax)){
                  leftMax=height[left]; 
                }else{
                    ans+=leftMax-height[left];
                }
                left++;
            }else{
                if(height[right]>rightMax){
                    rightMax=height[right];
                }else{
                    ans+=rightMax-height[right];
                }
                right--;
            }
        }
        return ans;
    }
}
```



#### [15. 三数之和](https://leetcode-cn.com/problems/3sum/)



给你一个包含 *n* 个整数的数组 `nums`，判断 `nums` 中是否存在三个元素 *a，b，c ，*使得 *a + b + c =* 0 ？请你找出所有满足条件且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

 

**示例：**

```
给定数组 nums = [-1, 0, 1, 2, -1, -4]，

满足要求的三元组集合为：
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```



击败50%

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> list=new ArrayList<>();
        int n=nums.length;
        if(n<3) return list;
        for(int i=0;i<n-2;i++){
            if(i==0||nums[i]!=nums[i-1]){//避免重复
            int l=i+1,r=n-1,target=-nums[i];
            while(l<r){
                if(nums[l]+nums[r]==target){
                    list.add(Arrays.asList(nums[i],nums[l],nums[r]));
                    while(l<r&&nums[l]==nums[l+1])l++;
                    while(l<r&&nums[r]==nums[r-1])r--;
                    l++;
                    r--;
                }else if(nums[l]+nums[r]>target){
                    while(l<r&&nums[r]==nums[r-1])r--;
                    r--;
                }else{
                    while(l<r&&nums[l]==nums[l+1])l++;
                    l++;
                }
            }
        }
        }
        return list;
    }
}
```

#### [142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)



给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 `null`。

为了表示给定链表中的环，我们使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 `pos` 是 `-1`，则在该链表中没有环。

**说明：**不允许修改给定的链表。

 

**示例 1：**

```
输入：head = [3,2,0,-4], pos = 1
输出：tail connects to node index 1
解释：链表中有一个环，其尾部连接到第二个节点。
```

![img](%E7%AE%97%E6%B3%95leetcode.assets/circularlinkedlist.png)

**示例 2：**

```
输入：head = [1,2], pos = 0
输出：tail connects to node index 0
解释：链表中有一个环，其尾部连接到第一个节点。
```

![img](%E7%AE%97%E6%B3%95leetcode.assets/circularlinkedlist_test2.png)

**示例 3：**

```
输入：head = [1], pos = -1
输出：no cycle
解释：链表中没有环。
```

![img](%E7%AE%97%E6%B3%95leetcode.assets/circularlinkedlist_test3.png)

击败100%

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode detectCycle(ListNode head) {
      if(head==null||head.next==null)return null;
      ListNode node1=head;
      ListNode node2=head;
      node1=node1.next;
      if(node1.next==null)return null;
      node2=node1.next;
      while(node2!=null&&node1!=node2&&node2.next!=null){
          node1=node1.next;
          node2=node2.next.next;
      }
      if(node2==null||node2.next==null)return null;
      node1=head;
      while(node1!=node2){
          node1=node1.next;
          node2=node2.next;
      }
      return node1;

    }
}
```

#### [76. 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

难度困难456收藏分享切换为英文关注反馈

给你一个字符串 S、一个字符串 T，请在字符串 S 里面找出：包含 T 所有字母的最小子串。

**示例：**

```
输入: S = "ADOBECODEBANC", T = "ABC"
输出: "BANC"
```

**说明：**

- 如果 S 中不存这样的子串，则返回空字符串 `""`。
- 如果 S 中存在这样的子串，我们保证它是唯一的答案。

初版击败51%

1. 初始，left 指针和 right 指针都指向 S 的第一个元素.
2. 将 right指针右移，扩张窗口，直到得到一个可行窗口，亦即包含 T的全部字母的窗口。
3. 得到可行的窗口后，将 left指针逐个右移，若得到的窗口依然可行，则更新最小窗口大小。
4. 若窗口不再可行，则跳转至 2 。

重复以上步骤，直到遍历完全部窗口。返回最小的窗口。

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/d3d70a1281ae22ade274cefdccd9226b3d9d4686412978e7e38a100099effeef-image.png)




```java
class Solution {
    public String minWindow(String s, String t) {
     if(s.length()==0||t.length()==0)return "";
     Map<Character,Integer> ditT=new HashMap<Character,Integer>();
     for(char c:t.toCharArray()){
         int count=ditT.getOrDefault(c,0);
         ditT.put(c,count+1);
     }
     int required=ditT.size();
     int formed=0,l=0,r=0;
     Map<Character,Integer> map=new HashMap<Character,Integer>();
     int[] res=new int[]{-1,0,0};
     while(r<s.length()){
         char c=s.charAt(r);
         map.put(c,map.getOrDefault(c,0)+1);
         if(ditT.containsKey(c)&&ditT.get(c).intValue()==map.get(c).intValue()){
             formed++;
         }
         while(l<=r&&formed==required){
             if(res[0]==-1||r-l+1<res[0]){
                 res[0]=r-l+1;
                 res[1]=l;
                 res[2]=r;
             }
             char c2=s.charAt(l);
             map.put(c2,map.getOrDefault(c2,0)-1);
             if(ditT.containsKey(c2)&&ditT.get(c2).intValue()>map.get(c2).intValue()){
                 formed--;
             }
            l++;
         }
         r++;
     } 
     return res[0]==-1?"":s.substring(res[1],res[2]+1);
    }
}
```

牺牲空间换时间击败90%

```java
class Solution {
    public static String minWindow(String s, String t) {
        if (s == null || s == "" || t == null || t == "" || s.length() < t.length()) {
            return "";
        }
        //用来统计t中每个字符出现次数
        int[] needs = new int[128];
        //用来统计滑动窗口中每个字符出现次数
        int[] window = new int[128];

        for (int i = 0; i < t.length(); i++) {
            needs[t.charAt(i)]++;
        }

        int left = 0;
        int right = 0;

        String res = "";

        //目前有多少个字符
        int count = 0;

        //用来记录最短需要多少个字符。
        int minLength = s.length() + 1;

        while (right < s.length()) {
            char ch = s.charAt(right);
            window[ch]++;
            if (needs[ch] > 0 && needs[ch] >= window[ch]) {
                count++;
            }

            //移动到不满足条件为止
            while (count == t.length()) {
                ch = s.charAt(left);
                if (needs[ch] > 0 && needs[ch] >= window[ch]) {
                    count--;
                }
                if (right - left + 1 < minLength) {
                    minLength = right - left + 1;
                    res = s.substring(left, right + 1);

                }
                window[ch]--;
                left++;

            }
            right++;

        }
        return res;
    }

}
```



# 位运算

#### [29. 两数相除](https://leetcode-cn.com/problems/divide-two-integers/)

难度中等321收藏分享切换为英文关注反馈

给定两个整数，被除数 `dividend` 和除数 `divisor`。将两数相除，要求不使用乘法、除法和 mod 运算符。

返回被除数 `dividend` 除以除数 `divisor` 得到的商。

整数除法的结果应当截去（`truncate`）其小数部分，例如：`truncate(8.345) = 8` 以及 `truncate(-2.7335) = -2`

 

**示例 1:**

```
输入: dividend = 10, divisor = 3
输出: 3
解释: 10/3 = truncate(3.33333..) = truncate(3) = 3
```

**示例 2:**

```
输入: dividend = 7, divisor = -3
输出: -2
解释: 7/-3 = truncate(-2.33333..) = -2
```

 

**提示：**

- 被除数和除数均为 32 位有符号整数。

- 除数不为 0。

- 假设我们的环境只能存储 32 位有符号整数，其数值范围是 [−2*31, 2*31 − 1]

  。本题中，如果除法结果溢出，则返回 2*31 − 1。



击败100%

举个例子：11 除以 3 。
首先11比3大，结果至少是1， 然后我让3翻倍，就是6，发现11比3翻倍后还要大，那么结果就至少是2了，那我让这个6再翻倍，得12，11不比12大，吓死我了，差点让就让刚才的最小解2也翻倍得到4了。但是我知道最终结果肯定在2和4之间。也就是说2再加上某个数，这个数是多少呢？我让11减去刚才最后一次的结果6，剩下5，我们计算5是3的几倍，也就是除法，看，递归出现了。

```java
class Solution {
    public int divide(int dividend, int divisor) {
       if(dividend==0) return 0;
       if(divisor==1)return dividend;
       if(divisor==-1){//处理越界的情况
           return dividend==Integer.MIN_VALUE?Integer.MAX_VALUE:-dividend;
       }
       int sign=1;//标志正负
       long a=dividend;
       long b=divisor;
       if(a>0&&b<0||a<0&&b>0){
           sign=-1;
       }
       a=a>0?a:-a;
       b=b>0?b:-b;
       int res=0;
       res=help(a,b);
       if(sign==1) {
       //res=res>Integer.MAX_VALUE?Integer.MAX_VALUE:res;
       return res;
       }
       return -res;
    }
    private int help(long a,long b){
        if(a<b) return 0;
        int count=1;
        long b2=b;
        while(b2+b2<=a){
            count=count+count;
            b2=b2+b2;
        }
        return count+help(a-b2,b);
    }
}
```



# 递归

#### [236. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)



给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

[百度百科](https://baike.baidu.com/item/最近公共祖先/8918834?fr=aladdin)中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

例如，给定如下二叉树: root = [3,5,1,6,2,0,8,null,null,7,4]

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/15/binarytree.png)

 

**示例 1:**

```
输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
输出: 3
解释: 节点 5 和节点 1 的最近公共祖先是节点 3。
```



击败99%

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
    private TreeNode res;
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        this.res=null;
        if(root==null) return res;
        help(root,p,q);
        return res;
    }
    private boolean help(TreeNode root,TreeNode p,TreeNode q){
        if(root==null){
            return false;
        }
        int left=(help(root.left,p,q))?1:0;
        int right=(help(root.right,p,q))?1:0;
        int mid=(root==p||root==q)?1:0;
        if(mid+left+right>=2){
            this.res=root;
        }
        return (mid+left+right)>0;
    }
}
```



#### [33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)



假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 `[0,1,2,4,5,6,7]` 可能变为 `[4,5,6,7,0,1,2]` )。

搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 `-1` 。

你可以假设数组中不存在重复的元素。

你的算法时间复杂度必须是 *O*(log *n*) 级别。

**示例 1:**

```
输入: nums = [4,5,6,7,0,1,2], target = 0
输出: 4
```

**示例 2:**

```
输入: nums = [4,5,6,7,0,1,2], target = 3
输出: -1
```



击败100%

```java
class Solution {
    private int res;
    public int search(int[] nums, int target) {
       int n=nums.length;
       if(n==0) return -1;
        help(nums,0,n-1,target);
        if(nums[res]==target) return res;
        return -1;
       
    }
    private void help(int[] nums,int l,int r,int target){
        if(l>r) return;
        int mid=l+(r-l)/2;
        if(nums[mid]==target){
            res=mid;
            return;
        }else{
            help(nums,l,mid-1,target);
            help(nums,mid+1,r,target);
        }
        return;
    }
}
```



#### [230. 二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)



给定一个二叉搜索树，编写一个函数 `kthSmallest` 来查找其中第 **k** 个最小的元素。

**说明：**
你可以假设 k 总是有效的，1 ≤ k ≤ 二叉搜索树元素个数。

**示例 1:**

```
输入: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
输出: 1
```

**示例 2:**

```
输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 3
```

**进阶：**

如果二叉搜索树经常被修改（插入/删除操作）并且你需要频繁地查找第 k 小的值，你将如何优化 `kthSmallest` 函数？

击败66，如果改成迭代，再找到k时停止，会快一些，不用遍历整颗二叉搜索树

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
    private List<Integer> list=new ArrayList<>();
    public int kthSmallest(TreeNode root, int k) {
        inOrder(root);
        return list.get(k-1);
    }
    private void inOrder(TreeNode root){
        if(root==null) return;
        inOrder(root.left);
        list.add(root.val);
        inOrder(root.right);
    }
}
```

#### [124. 二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)



给定一个**非空**二叉树，返回其最大路径和。

本题中，路径被定义为一条从树中任意节点出发，达到任意节点的序列。该路径**至少包含一个**节点，且不一定经过根节点。

**示例 1:**

```
输入: [1,2,3]

       1
      / \
     2   3

输出: 6
```

**示例 2:**

```
输入: [-10,9,20,null,null,15,7]

   -10
   / \
  9  20
    /  \
   15   7

输出: 42
```



击败99%

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
    private int max=Integer.MIN_VALUE;
    public int maxPathSum(TreeNode root) {
        help(root);
        return max;
    }
    private int help(TreeNode root){
        if(root==null) return 0;
        int left=Math.max(help(root.left),0);
        int right=Math.max(help(root.right),0);
        max=Math.max(max,left+right+root.val);
        return root.val+Math.max(left,right);
    }
}
```



#### [59. 螺旋矩阵 II](https://leetcode-cn.com/problems/spiral-matrix-ii/)

难度中等180收藏分享切换为英文关注反馈

给定一个正整数 *n*，生成一个包含 1 到 *n*2 所有元素，且元素按顺时针顺序螺旋排列的正方形矩阵。

**示例:**

```
输入: 3
输出:
[
 [ 1, 2, 3 ],
 [ 8, 9, 4 ],
 [ 7, 6, 5 ]
]
```

 击败100%

```java
class Solution {
    public int[][] generateMatrix(int n) {
        int[][] matrix=new int[n][n];
        help(matrix,0,n-1,0,n-1,1);
        return matrix;
    }
    private void help(int[][] matrix,int rBegain,int rEnd,int cBegain,int cEnd,int number){
        if(rBegain<rEnd&&cBegain<cEnd){
            for(int i=cBegain;i<=cEnd;i++){
                matrix[rBegain][i]=number;
                number++;
            }
            for(int i=rBegain+1;i<=rEnd;i++){
                matrix[i][cEnd]=number;
                number++;
            }
            for(int i=cEnd-1;i>=cBegain;i--){
                matrix[rEnd][i]=number;
                number++;
            }
            for(int i=rEnd-1;i>rBegain;i--){
                matrix[i][cBegain]=number;
                number++;
            }
            help(matrix,rBegain+1,rEnd-1,cBegain+1,cEnd-1,number);
        }else if(rBegain==rEnd&&cBegain==cEnd){
            matrix[rBegain][cBegain]=number;
            return;
        }
    }
}
```



# 数学

#### [43. 字符串相乘](https://leetcode-cn.com/problems/multiply-strings/)



给定两个以字符串形式表示的非负整数 `num1` 和 `num2`，返回 `num1` 和 `num2` 的乘积，它们的乘积也表示为字符串形式。

**示例 1:**

```
输入: num1 = "2", num2 = "3"
输出: "6"
```

**示例 2:**

```
输入: num1 = "123", num2 = "456"
输出: "56088"
```

**说明：**

1. `num1` 和 `num2` 的长度小于110。
2. `num1` 和 `num2` 只包含数字 `0-9`。
3. `num1` 和 `num2` 均不以零开头，除非是数字 0 本身。
4. **不能使用任何标准库的大数类型（比如 BigInteger）**或**直接将输入转换为整数来处理**。
5. 

击败29%：

方法一：普通竖式
思路

竖式运算思想，以 num1 为 123，num2 为 456 为例分析：

遍历 num2 每一位与 num1 进行相乘，将每一步的结果进行累加。

注意：

num2 除了第一位的其他位与 num1 运算的结果需要 补0
计算字符串数字累加其实就是 415. 字符串相加

```java
class Solution {
    public String multiply(String num1, String num2) {
      if(num1.equals("0")||num2.equals("0"))return "0";
      String res="0";
        for(int i=num2.length()-1;i>=0;i--){
            int carray=0;
            StringBuilder sb=new StringBuilder();
            for(int l=0;l<num2.length()-1-i;l++){
                sb.append(0);
            }
            int a1=num2.charAt(i)-'0';
            for(int j=num1.length()-1;j>=0||carray!=0;j--){
                int a2=j<0?0:num1.charAt(j)-'0';
                int a3=a1*a2+carray;
                int a=a3%10;
                sb.append(a);
                carray=a3/10;
            }
            res=addString(res,sb.reverse().toString());
        }
        return res;
    }
    private String addString(String s1,String s2){
        int carray=0;
        StringBuilder ss=new StringBuilder();
        for(int i=s1.length()-1,j=s2.length()-1;i>=0||j>=0||carray!=0;i--,j--){
            int a1=i<0?0:s1.charAt(i)-'0';
            int a2=j<0?0:s2.charAt(j)-'0';
            int a3=(a1+a2+carray);
            int a=a3%10;
            ss.append(a);
            carray=a3/10;

        }
        return ss.reverse().toString();
    }
}
```



击败97%

方法二：优化竖式
该算法是通过两数相乘时，乘数某位与被乘数某位相乘，与产生结果的位置的规律来完成。具体规律如下：

乘数 num1 位数为 MM，被乘数 num2 位数为 NN， num1 x num2 结果 res 最大总位数为 M+N
num1[i] x num2[j] 的结果为 tmp(位数为两位，"0x","xy"的形式)，其第一位位于 res[i+j]，第二位位于 res[i+j+1]。
结合下图更容易理解

![img](https://pic.leetcode-cn.com/171cad48cd0c14f565f2a0e5aa5ccb130e4562906ee10a84289f12e4460fe164-image.png)

```java
class Solution {
    public String multiply(String num1, String num2) {
      if(num1.equals("0")||num2.equals("0"))return "0";
      int[] res=new int[num1.length()+num2.length()];
      for(int i=num1.length()-1;i>=0;i--){
          int a=num1.charAt(i)-'0';
          for(int j=num2.length()-1;j>=0;j--){
              
              int b=num2.charAt(j)-'0';
              int sum=a*b+res[i+j+1];
              res[i+j+1]=sum%10;
              res[i+j]+=sum/10;
          }
      }
      StringBuilder sb=new StringBuilder();
      for(int i=0;i<res.length;i++){
          if(i==0&&res[0]==0)continue;
            sb.append(res[i]);
      }
      return sb.toString();
    }
}
```



#### [89. 格雷编码](https://leetcode-cn.com/problems/gray-code/)



格雷编码是一个二进制数字系统，在该系统中，两个连续的数值仅有一个位数的差异。

给定一个代表编码总位数的非负整数 *n*，打印其格雷编码序列。即使有多个不同答案，你也只需要返回其中一种。

格雷编码序列必须以 0 开头。

 

**示例 1:**

```
输入: 2
输出: [0,1,3,2]
解释:
00 - 0
01 - 1
11 - 3
10 - 2

对于给定的 n，其格雷编码序列并不唯一。
例如，[0,2,3,1] 也是一个有效的格雷编码序列。

00 - 0
10 - 2
11 - 3
01 - 1
```

**示例 2:**

```
输入: 0
输出: [0]
解释: 我们定义格雷编码序列必须以 0 开头。
     给定编码总位数为 n 的格雷编码序列，其长度为 2n。当 n = 0 时，长度为 20 = 1。
     因此，当 n = 0 时，其格雷编码序列为 [0]。
```



击败86%：

思路：
设 n 阶格雷码集合为 G(n)，则 G(n+1) 阶格雷码为：
给 G(n) 阶格雷码每个元素二进制形式前面添加 0，得到 G'(n)；
设 G(n)集合倒序（镜像）为 R(n)，给 R(n) 每个元素二进制形式前面添加 1，得到 R'(n)；
G(n+1) = G'(n) ∪ R'(n)拼接两个集合即可得到下一阶格雷码。
根据以上规律，可从 0 阶格雷码推导致任何阶格雷码。
代码解析：
由于最高位前默认为 00，因此 G'(n) =G(n)，只需在 res(即 G(n))后添加 R'(n) 即可；
计算 R'(n)：执行 head = 1 << i 计算出对应位数，以给 R(n)前添加 1 得到对应 R'(n)；
倒序遍历 res(即 G(n))：依次求得 R'(n)各元素添加至 res 尾端，遍历完成后 res(即 G(n+1))。
图解：

![img](%E7%AE%97%E6%B3%95leetcode.assets/28acf6d5b1fae0fb2dddbedd7ac92ffeee8902cd28233bdfb08b52af411a9bb2-Picture4.png)

```java
class Solution {
    private List<Integer> list=new ArrayList<>();
    public List<Integer> grayCode(int n) {
        list.add(0);
        int head=1;
        for(int i=1;i<=n;i++){
            for(int j=list.size()-1;j>=0;j--){
                list.add(head|list.get(j));
            }
            head<<=1;
        }
        return list;
    }
}
```

#### [128. 最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

难度困难317收藏分享切换为英文关注反馈

给定一个未排序的整数数组，找出最长连续序列的长度。

要求算法的时间复杂度为 *O(n)*。

**示例:**

```
输入: [100, 4, 200, 1, 3, 2]
输出: 4
解释: 最长连续序列是 [1, 2, 3, 4]。它的长度为 4。
```

击败83%

利用set来达到O(1)查找，查找(num-1),如果不在，则以num为开始查找，

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> set=new HashSet<Integer>();
        for(int num:nums){
            set.add(num);
        }
        int longestCount=0;
        for(int num:nums){
            if(!set.contains(num-1)){
                int preCount=1;
                //只有当一个新的序列查找开始，while才会执行，while总共执行O(n)次，所以算法复杂度为O(n)+O(n)=O(n);
                while(set.contains(num+1)){
                    preCount+=1;
                    num+=1;
                }
                longestCount=Math.max(longestCount,preCount);
            }
        }
        return longestCount;
    }
}
```



# 链表

#### [61. 旋转链表](https://leetcode-cn.com/problems/rotate-list/)



给定一个链表，旋转链表，将链表每个节点向右移动 *k* 个位置，其中 *k* 是非负数。

**示例 1:**

```
输入: 1->2->3->4->5->NULL, k = 2
输出: 4->5->1->2->3->NULL
解释:
向右旋转 1 步: 5->1->2->3->4->NULL
向右旋转 2 步: 4->5->1->2->3->NULL
```

**示例 2:**

```
输入: 0->1->2->NULL, k = 4
输出: 2->0->1->NULL
解释:
向右旋转 1 步: 2->0->1->NULL
向右旋转 2 步: 1->2->0->NULL
向右旋转 3 步: 0->1->2->NULL
向右旋转 4 步: 2->0->1->NULL
```



击败78%

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode rotateRight(ListNode head, int k) {
        if(k==0||head==null)return head;
        ListNode node=new ListNode(-1);
        ListNode point=head;
        int l=0;
        while(point!=null){
            l++;
            point=point.next;
        }
        int target=k%l;
        if(target==0)return head;
        point=head;
        int target2=l-target;
        while(target2>0){
            point=point.next;
            target2--;
        }
        ListNode node2=node;
        while(point!=null){
            node2.next=new ListNode(point.val);
            point=point.next;
            node2=node2.next;
        }
        point=head;
        target2=l-target;
        while(target2>0){
           node2.next=new ListNode(point.val);
            point=point.next;
            node2=node2.next;
            target2--; 
        }
        return node.next;
    }
}
```

#### [148. 排序链表](https://leetcode-cn.com/problems/sort-list/)



在 *O*(*n* log *n*) 时间复杂度和常数级空间复杂度下，对链表进行排序。

**示例 1:**

```
输入: 4->2->1->3
输出: 1->2->3->4
```

**示例 2:**

```
输入: -1->5->3->4->0
输出: -1->0->3->4->5
```

击败99%

**分割** cut 环节： 找到当前链表中点，并从中点将链表断开（以便在下次递归 cut 时，链表片段拥有正确边界）；
我们使用 fast,slow 快慢双指针法，奇数个节点找到中点，偶数个节点找到中心左边的节点。
找到中点 slow 后，执行 slow.next = None 将链表切断。
递归分割时，输入当前链表左端点 head 和中心节点 slow 的下一个节点 tmp(因为链表是从 slow 切断的)。
cut 递归终止条件： 当head.next == None时，说明只有一个节点了，直接返回此节点。
**合并** merge 环节： 将两个排序链表合并，转化为一个排序链表。
双指针法合并，建立辅助ListNode h 作为头部。
设置两指针 left, right 分别指向两链表头部，比较两指针处节点值大小，由小到大加入合并链表头部，指针交替前进，直至添加完两个链表。
返回辅助ListNode h 作为头部的下个节点 h.next。
时间复杂度 O(l + r)，l, r 分别代表两个链表长度。
当题目输入的 head == None 时，直接返回None。

![Picture2.png](%E7%AE%97%E6%B3%95leetcode.assets/8c47e58b6247676f3ef14e617a4686bc258cc573e36fcf67c1b0712fa7ed1699-Picture2.png)

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode sortList(ListNode head) {
       if(head==null||head.next==null)return head;
       
       ListNode slow=head;
       ListNode fast=head.next;//此处值得注意，不能是head，当长度为2时，想一下就明白了
       while(fast!=null&&fast.next!=null){
           slow=slow.next;
           fast=fast.next.next;
       }
       ListNode temp=slow.next;
       slow.next=null;
       ListNode left=sortList(head);
       ListNode right=sortList(temp);
       ListNode res=new ListNode(-1);
       ListNode node1=res;
       while(left!=null&&right!=null){
           if(left.val<right.val){
               node1.next=left;
               left=left.next;
           }else{
               node1.next=right;
               right=right.next;
           }
           node1=node1.next;
       }
       node1.next=left==null?right:left;
       return res.next;
    }
}
```



# 单调栈

#### [84. 柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

难度困难570收藏分享切换为英文关注反馈

给定 *n* 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。

求在该柱状图中，能够勾勒出来的矩形的最大面积。

 

![img](%E7%AE%97%E6%B3%95leetcode.assets/histogram.png)

以上是柱状图的示例，其中每个柱子的宽度为 1，给定的高度为 `[2,1,5,6,2,3]`。

 

![img](%E7%AE%97%E6%B3%95leetcode.assets/histogram_area.png)

图中阴影部分为所能勾勒出的最大矩形面积，其面积为 `10` 个单位。

 

**示例:**

```
输入: [2,1,5,6,2,3]
输出: 10
```



击败88%

我们就拿示例的数组 [2, 1, 5, 6, 2, 3] 为例：

一开始看到的柱形高度为 2 ，这个时候以这个 2 为高度的最大面积的矩形还不能确定，我们需要继续向右遍历，如下图。

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/414a06bad966eba25ef6c1281e5780381205d1c76c70b7a2561969bfe859eb01-image.png)

然后遍历到高度为 1 的柱形，这个时候以这个柱形为高度的矩形的最大面积还是不知道的。但是它之前的以 2 为高度的最大面积的矩形是可以确定的，这是因为这个 1 比 2 小 ，因为这个 1 卡在了这里 2 不能再向右边扩展了，如下图。

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/dcbed1d0cba33c059f3833a0da2e78e5e0a96370a415930acbdb126b44b398c8-image.png)

我们计算一下以 2 为高度的最大矩形的面积是 2。其实这个时候，求解这个问题的思路其实已经慢慢打开了。如果已经确定了一个柱形的高度，我们可以无视它，将它以虚框表示，如下图。

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/da72cf520eb05a42725a8982bb27e507f9213b257b6f32bd6ef4a49e1d416a1f-image.png)

遍历到高度为 5 的柱形，同样的以当前看到柱形为高度的矩形的最大面积也是不知道的，因为我们还要看右边高度的情况。那么它的左右有没有可以确定的柱形呢？没有，这是因为 5 比 1 大，我们看后面马上就出现了 6，不管是 1 这个柱形还是 5 这个柱形，都还可以向右边扩展；

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/1bc87193557ab8c2a0bc2319aef66a12c4514aa45e9aa823eefa0e536289f0ed-image.png)

接下来，遍历到高度为 6 的柱形，同样的，以柱形 1、5、6 为高度的最大矩形面积还是不能确定下来；

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/c34663bee2af70da626134a0426e7ac8e8e305a039abb43862a807fc1a7183a8-image.png)

再接下来，遍历到高度为 2 的柱形。

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/180d06d75a6480c649d0d4ad6c31e755a4e9c03294d23b7fb28bbac3fb0c006a-image.png)

发现了一件很神奇的事情，高度为 6 的柱形对应的最大矩形的面积的宽度可以确定下来，它就是夹在高度为 5 的柱形和高度为 2 的柱形之间的距离，它的高度是 6，宽度是 1。

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/5a60100c86cacd620424c807b6c062dd8f9990538da27420de81822557f8c782-image.png)

将可以确定的柱形设置为虚线。

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/5cc53fec12ab6a3c790c30c47ac183e5fd0426a50a7cd46d813cac6718f658cc-image.png)

接下来柱形 5 对应的最大面积的矩形的宽度也可以确定下来，它是夹在高度为 1 和高度为 2 的两个柱形之间的距离；

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/c4ae9e65efe6247d63b1ac2834015bf1be6270e0d323e6f02d7e1bb4ab6644ed-image.png)

确定好以后，我们将它标称虚线。

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/973788fa2d6407c9db41c100e9fa9b9ef00e4891343089773b716316c6f3e7f2-image.png)

我们发现了，只要是遇到了当前柱形的高度比它上一个柱形的高度严格小的时候，一定可以确定它之前的某些柱形的最大宽度，并且确定的柱形宽度的顺序是从右边向左边。
这个现象告诉我们，在遍历的时候需要记录的信息就是遍历到的柱形的下标，它一左一右的两个柱形的下标的差就是这个面积最大的矩形对应的最大宽度。

就在考虑如果遍历之后，栈不为空，那么还需要一步：即弹出栈中所有元素，分别计算最大面积。然而当加了两个0以后，在结束后，栈一定为空！ 

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int res = 0;
        Stack<Integer> stack = new Stack<>();
        int[] new_heights = new int[heights.length + 2];
        for (int i = 1; i < heights.length + 1; i++) new_heights[i] = heights[i - 1];
        
        for (int i = 0; i < new_heights.length; i++) {
            
            while (!stack.isEmpty() && new_heights[stack.peek()] > new_heights[i]) {
                int cur = stack.pop();
                res = Math.max(res, (i - stack.peek() -1) * new_heights[cur]);
                //此处减去的stack.peek()是因为stack已经pop过一次了，重点，已经删去了为高的元素
            }
            stack.push(i);
        }
        return res;  
    }


}
```



#### [85. 最大矩形](https://leetcode-cn.com/problems/maximal-rectangle/)

难度困难406收藏分享切换为英文关注反馈

给定一个仅包含 0 和 1 的二维二进制矩阵，找出只包含 1 的最大矩形，并返回其面积。

**示例:**

```
输入:
[
  ["1","0","1","0","0"],
  ["1","0","1","1","1"],
  ["1","1","1","1","1"],
  ["1","0","0","1","0"]
]
输出: 6
```

在上题基础上做：

![image.png](%E7%AE%97%E6%B3%95leetcode.assets/aabb1b287134cf950aa80526806ef4025e3920d57d237c0369ed34fae83e2690-image.png)

击败69%

```java
class Solution {
    public int maximalRectangle(char[][] matrix) {
       int maxArea=0;
       if(matrix==null||matrix.length==0||matrix[0].length==0)return maxArea;
       int[] matrix2=new int[matrix[0].length];
       for(int i=0;i<matrix.length;i++){
           for(int j=0;j<matrix[0].length;j++){
               if(matrix[i][j]=='1'){
                   matrix2[j]+=1;
               }else{
                   matrix2[j]=0;
               }
           }
           maxArea=Math.max(maxArea,help(matrix2));
       }
       return maxArea;
    }
    private int help(int[] matrix2){
        int[] matrix3=new int[matrix2.length+2];
        for(int i=1;i<matrix2.length+1;i++){
            matrix3[i]=matrix2[i-1];
        }
        int max=0;
        Stack<Integer> stack=new Stack<>();
        for(int i=0;i<matrix3.length;i++){
            while(!stack.isEmpty()&&matrix3[stack.peek()]>matrix3[i]){
                int count=stack.pop();
                max=Math.max(max,(i-stack.peek()-1)*matrix3[count]);
            }
            stack.push(i);
        }
        return max;
    }
}
```



# 双向队列

LinkedList和ArrayDeque都是实现Deque接口，所以，可以说他们俩都是双向队列。

#### [239. 滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum/)

难度困难347收藏分享切换为英文关注反馈

给定一个数组 *nums*，有一个大小为 *k* 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 *k* 个数字。滑动窗口每次只向右移动一位。

返回滑动窗口中的最大值。 

**进阶：**

你能在线性时间复杂度内解决此题吗？

**示例:**

```
输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
输出: [3,3,5,5,6,7] 
解释: 

  滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

击败64%

算法非常直截了当：

处理前 k 个元素，初始化双向队列。

遍历整个数组。在每一步 :

清理双向队列 :

1. 只保留当前滑动窗口中有的元素的索引。

2. 移除比当前元素小的所有元素，它们不可能是最大的。

3. 将当前元素添加到双向队列中。

4. 将 deque[0] 添加到输出中。

   返回输出数组。

```java
class Solution {
    private ArrayDeque<Integer> deque=new ArrayDeque<Integer>();
    private int[] nums;
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n=nums.length;
        if(n*k==0)return new int[0];
        if(k==1)return nums;
        this.deque=deque;
        this.nums=nums;
        int[] res=new int[n-k+1];
        int maxIndex=0;
        for(int i=0;i<k;i++){
            help(i,k);
            deque.add(i);
            if(nums[i]>nums[maxIndex])maxIndex=i;
        }
        res[0]=nums[maxIndex];
        for(int i=k;i<n;i++){
            help(i,k);
            deque.add(i);
            res[i-k+1]=nums[deque.getFirst()];
        }
        return res;
    }
    private void help(int i,int k){
        if(!deque.isEmpty()&&(i-k)==deque.getFirst()){
            deque.removeFirst();
        }
       while(!deque.isEmpty()&&nums[i]>nums[deque.getLast()]){
            deque.removeLast();
        }
    }
}
```

方法二(感觉是把上面的双向队列原理直接写出来了)

击败98%

思路：用head和tail控制一个窗口 第一次的时候遍历整个窗口的数据得到最大值 其他时间需要分情况，滑动窗口每次移动相当于除去头部数据，在尾部加入一个新数据（可以理解为长度固定的双端队列） 如果新加入的数据比移动前的滑动窗口中的最大值大或者两者相等，那么此滑块最大值就是新加入的尾部数据 如果新加入的数据比移动前的滑动窗口中的最大值还要小，那么需要判断之前去掉的头部数据是不是最大值，如果不是则此滑动窗口的最大值和移动前的滑动窗口的最大值相同，否则需要遍历求最大值

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if(k==0||nums.length==0){return new int[0];}
        int max=0;
        int result[] = new int [nums.length-k+1];
        int resultPos=0;
        int head=0;
        int tail=k-1;
        while(tail<nums.length){
            if(head==0){
             for(int i=0;i<k;i++){
                 if(max<nums[i]){max=nums[i];}
             }
             result[resultPos]=max;
            }
            else{
                if(nums[tail]>=max){
                    max=nums[tail];
                    result[resultPos]=max;
                }
                else{
                    if(max!=nums[head-1]){
                    result[resultPos]=max;

                    }
                    else{
                    max=nums[head];
                    for(int i=head+1;i<=tail;i++){
                        if(nums[i]>max){max=nums[i];}
                    }
                    result[resultPos]=max;
                    }
                }
            }
                    resultPos++;
                    head++;
                    tail++;
        }
        return result;
    }
}
```



# 树

#### [面试题 04.05. 合法二叉搜索树](https://leetcode-cn.com/problems/legal-binary-search-tree-lcci/)

难度中等12收藏分享切换为英文关注反馈

实现一个函数，检查一棵二叉树是否为二叉搜索树。

**示例 1:**

```
输入:
    2
   / \
  1   3
输出: true
```

**示例 2:**

```
输入:
    5
   / \
  1   4
     / \
    3   6
输出: false
解释: 输入为: [5,1,4,null,null,3,6]。
     根节点的值为 5 ，但是其右子节点值为 4 。
```

击败100%（双100%）

```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        if (root == null) return true;
        TreeNode maxLeft = root.left, minRight = root.right;
        // 找寻左子树中的最右（数值最大）节点
        while (maxLeft != null && maxLeft.right != null)
            maxLeft = maxLeft.right;
        // 找寻右子树中的最左（数值最小）节点
        while (minRight != null && minRight.left != null)
            minRight = minRight.left;
        // 当前层是否合法
        boolean ret = (maxLeft == null || maxLeft.val < root.val) && (minRight == null || root.val < minRight.val);
        // 进入左子树和右子树并判断是否合法
        return  ret && isValidBST(root.left) && isValidBST(root.right);
    }
}
```

#### [面试题 04.06. 后继者](https://leetcode-cn.com/problems/successor-lcci/)

难度中等12收藏分享切换为英文关注反馈

设计一个算法，找出二叉搜索树中指定节点的“下一个”节点（也即中序后继）。

如果指定节点没有对应的“下一个”节点，则返回`null`。

**示例 1:**

```
输入: root = [2,1,3], p = 1

  2
 / \
1   3

输出: 2
```

**示例 2:**

```
输入: root = [5,3,6,2,4,null,null,1], p = 6

      5
     / \
    3   6
   / \
  2   4
 /   
1

输出: null
```

方法一击败45%

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
    TreeNode res;
    public TreeNode inorderSuccessor(TreeNode root, TreeNode p) {
        if(root==null||p==null)return null;
        while(root.val!=p.val){
            if(root.val<p.val){
                root=root.right;
            }else{
                res=root;
                root=root.left;
            }
        }
        
        if(root.right!=null){
            root=root.right;
            while(root.left!=null){
                root=root.left;
            }
            return root;
        }else{
            return res;
        }
    }
}
```

方法二击败100%（双100%）

正常中序遍历，如果找到p对应的节点，那么下一个遍历节点必定是我们需要的后继节点。
这个时候，置flag开关为true。
下一个被遍历的节点就能被传出来（ainNode）啦，与此同时关闭flag开关。
总结：前序、中序、后序遍历搜索树都能用该方法。另外如果是一个存在多个p节点的普通树，也能用此方法，但连续两个节点是p的情况不能用此方法。

```java
class Solution {
    //传root值的开关
    boolean flag = false;
    //存储后继节点
    TreeNode aimNode;
    public TreeNode inorderSuccessor(TreeNode root, TreeNode p) {
        if(root==null){
            return aimNode;
        }
        //下面所有代码是正常中序顺序的步骤
       inorderSuccessor(root.left,p);
       if(root==p){
           //找到p节点，打开开关
           flag = true;
       }else if(flag){
           //root不为p，且开关被打开时，才能进入此处
           //传出这个后继节点
           aimNode = root;
           flag =false;
       }
       inorderSuccessor(root.right,p);
       return aimNode;
    }

}

```



#### [297. 二叉树的序列化与反序列化](https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/)

难度困难180收藏分享切换为英文关注反馈

序列化是将一个数据结构或者对象转换为连续的比特位的操作，进而可以将转换后的数据存储在一个文件或者内存中，同时也可以通过网络传输到另一个计算机环境，采取相反方式重构得到原数据。

请设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，你只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。

**示例:** 

```
你可以将以下二叉树：

    1
   / \
  2   3
     / \
    4   5

序列化为 "[1,2,3,null,null,4,5]"
```

**提示:** 这与 LeetCode 目前使用的方式一致，详情请参阅 [LeetCode 序列化二叉树的格式](https://leetcode-cn.com/faq/#binary-tree)。你并非必须采取这种方式，你也可以采用其他的方法解决这个问题。

**说明:** 不要使用类的成员 / 全局 / 静态变量来存储状态，你的序列化和反序列化算法应该是无状态的。

击败62%

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
public class Codec {
    
    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
       StringBuilder sb=new StringBuilder();
       sb.append("[");
        if(root==null) return sb.append("null").append("]").toString();
        List<TreeNode> list=new LinkedList<>();
        list.add(root);
        help(sb,list);
        return sb.delete(sb.length()-1,sb.length()).append("]").toString();
    }
    private void help(StringBuilder sb,List<TreeNode> list){
        
        while(!list.isEmpty()){
            TreeNode node=list.remove(0);
            if(node==null){
                sb.append("null").append(",");
            }else{
                sb.append(node.val).append(",");
                list.add(node.left);
                list.add(node.right);
            }
        
        }
        return;
    }
    

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        if(data==null||data.length()==0)return null;
        String[] matrix=data.substring(1,data.length()-1).split(",");
        if(matrix==null||matrix.length==0||matrix[0].equals("null"))return null;
        List<TreeNode> list=new LinkedList<>();
        list.add(new TreeNode(Integer.valueOf(matrix[0])));
        return help2(matrix,list); 
    }
    private TreeNode help2(String[] matrix,List<TreeNode> list){
        int i=0;
        int n=matrix.length;
        TreeNode root=list.get(0);//此处为get，切记，我写成了remove导致浪费4个小时在这题上。
        while(!list.isEmpty()&&i<n-1){
            TreeNode node=list.remove(0);
                i++;
                if(!matrix[i].equals("null")){
                    node.left=new TreeNode(Integer.valueOf(matrix[i]));
                    list.add(node.left);
                }
                i++;
                if(!matrix[i].equals("null")){
                    node.right=new TreeNode(Integer.valueOf(matrix[i]));
                    list.add(node.right);
                }
        }
        return root;
    }
}

// Your Codec object will be instantiated and called as such:
// Codec codec = new Codec();
// codec.deserialize(codec.serialize(root));
```

#### [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

难度中等645

给定一个二叉树，返回它的*中序* 遍历。

**示例:**

```
输入: [1,null,2,3]
   1
    \
     2
    /
   3

输出: [1,3,2]
```

**进阶:** 递归算法很简单，你可以通过迭代算法完成吗？

执行用时：0 ms, 在所有 Java 提交中击败了100.00%的用户

内存消耗：37.8 MB, 在所有 Java 提交中击败了78.87%的用户

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
//递归
class Solution {
    List<Integer> list=new LinkedList<>();
    public List<Integer> inorderTraversal(TreeNode root) {
        help(root);
        return list;
    }
    private void help(TreeNode root){
        if(root==null)return;
        help(root.left);
        list.add(root.val);
        help(root.right);
    }
}
//迭代，栈

class Solution {
    
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> list=new LinkedList<>();
        Stack<TreeNode> stack=new Stack<>(); 
       
        while(root!=null||!stack.isEmpty()){
            while(root!=null){
                stack.push(root);
                root=root.left;
            }
            TreeNode node=stack.pop();
            list.add(node.val);
            root=node.right;
        }
        return list;
    }
    
}
```

#### [144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

难度中等354

给定一个二叉树，返回它的 *前序* 遍历。

 **示例:**

```
输入: [1,null,2,3]  
   1
    \
     2
    /
   3 

输出: [1,2,3]
```

**进阶:** 递归算法很简单，你可以通过迭代算法完成吗？

执行用时：0 ms, 在所有 Java 提交中击败了100.00%的用户

内存消耗：38.1 MB, 在所有 Java 提交中击败了15.14%的用户

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
//递归
class Solution {
    List<Integer> list=new LinkedList<>();
    public List<Integer> preorderTraversal(TreeNode root) {
        help(root);
        return list;
    }
    private void help(TreeNode root){
        if(root==null)return;
        list.add(root.val);
        help(root.left);
        help(root.right);
    }
}
//迭代，栈
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
    
    public List<Integer> preorderTraversal(TreeNode root) {
        
        List<Integer> list=new LinkedList<>();
        if(root==null)return list;
        Stack<TreeNode> stack=new Stack<>();
        stack.push(root);
        while(!stack.isEmpty()){
            TreeNode node=stack.pop();
            list.add(node.val);
            if(node.right!=null){
                stack.push(node.right);
            }
            if(node.left!=null){
                stack.push(node.left);
            }
        }
        return list;
    }
    
}

```

#### [145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

难度困难385

给定一个二叉树，返回它的 *后序* 遍历。

**示例:**

```
输入: [1,null,2,3]  
   1
    \
     2
    /
   3 

输出: [3,2,1]
```

**进阶:** 递归算法很简单，你可以通过迭代算法完成吗？

执行用时：0 ms, 在所有 Java 提交中击败了100.00%的用户

内存消耗：38 MB, 在所有 Java 提交中击败了47.01%的用户

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
//递归
class Solution {
    List<Integer> list=new LinkedList<>();
    public List<Integer> postorderTraversal(TreeNode root) {
        help(root);
        return list;
    }
    private void help(TreeNode root){
        if(root==null)return;
        help(root.left);
        help(root.right);
        list.add(root.val);

    }
}
//迭代，栈
class Solution { 
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> list=new LinkedList<>();
        if(root==null)return list;
        Stack<TreeNode> stack=new Stack<>();
        stack.push(root);
        while(!stack.isEmpty()){
            TreeNode node=stack.pop();
            list.add(0,node.val);
            if(node.left!=null){
                stack.push(node.left);
            }
            if(node.right!=null){
                stack.push(node.right);
            }
        }
        return list;
    }
    
}
```



# BFS

#### [542. 01 矩阵](https://leetcode-cn.com/problems/01-matrix/)

难度中等261收藏分享切换为英文关注反馈

给定一个由 0 和 1 组成的矩阵，找出每个元素到最近的 0 的距离。

两个相邻元素间的距离为 1 。

**示例 :**
输入:

```
0 0 0
0 1 0
1 1 1
```

输出:

```
0 0 0
0 1 0
1 2 1
```

**注意:**

1. 给定矩阵的元素个数不超过 10000。
2. 给定矩阵中至少有一个元素是 0。
3. 矩阵中的元素只在四个方向上相邻: 上、下、左、右。

击败47%

```java
class Solution {
    private int[][] grand=new int[][]{{-1,0},{0,1},{1,0},{0,-1}};
    private int[][] res;
    public int[][] updateMatrix(int[][] matrix) {
        if(matrix==null||matrix.length==0||matrix[0].length==0)return new int[0][0];
        int m=matrix.length,n=matrix[0].length;
        res=new int[m][n];
        Queue<int[]> queue=new LinkedList<>();
        boolean[][] visited=new boolean[m][n];
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(matrix[i][j]==0){
                    visited[i][j]=true;
                    res[i][j]=0;
                    queue.add(new int[]{i,j});
                }
            }
        }
        while(!queue.isEmpty()){
            int[] tem=queue.poll();
            int i=tem[0];
            int j=tem[1];
            for(int[] num:grand){
                int di=i+num[0],dj=j+num[1];
                if(di>=0&&di<m&&dj>=0&&dj<n&&!visited[di][dj]){
                    res[di][dj]=res[i][j]+1;
                    visited[di][dj]=true;
                    queue.add(new int[]{di,dj});
                }
                
            }
        }
        return res;
    }
}
```



#### [863. 二叉树中所有距离为 K 的结点](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/)

难度中等94收藏分享切换为英文关注反馈

给定一个二叉树（具有根结点 `root`）， 一个目标结点 `target` ，和一个整数值 `K` 。

返回到目标结点 `target` 距离为 `K` 的所有结点的值的列表。 答案可以以任何顺序返回。

**示例 1：**

```
输入：root = [3,5,1,6,2,0,8,null,null,7,4], target = 5, K = 2

输出：[7,4,1]

解释：
所求结点为与目标结点（值为 5）距离为 2 的结点，
值分别为 7，4，以及 1



注意，输入的 "root" 和 "target" 实际上是树上的结点。
上面的输入仅仅是对这些对象进行了序列化描述。
```

击败33%~94%同样的代码，不稳定

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
    // 用map记录每个节点的父节点
    private Map<TreeNode, TreeNode> parents = new HashMap<>();

    private Set<TreeNode> used = new HashSet<>();

    private TreeNode targetNode;

    // 找到目标节点后以目标节点为开始位置向三个方向蔓延
    public List<Integer> distanceK(TreeNode root, TreeNode target, int K) {
        find(root, null, target);
        List<Integer> res = new LinkedList<>();
        dfs(targetNode, res, K);
        return res;
    }

    private void find(TreeNode root, TreeNode parent, TreeNode target) {
        if (null == root) {
            return;
        }
        if (root.val == target.val) {
            targetNode = root;
        }
        parents.put(root, parent);
        find(root.left, root, target);
        find(root.right, root, target);
    }

    private void dfs(TreeNode root, List<Integer> collector, int distance) {
        if (root != null && !used.contains(root)) {
            // 标记为已访问
            used.add(root);
            if (distance <= 0) {
                collector.add(root.val);
                return;
            }
            dfs(root.left, collector, distance - 1);
            dfs(root.right, collector, distance - 1);
            dfs(parents.get(root), collector, distance - 1);
        }
    }
}
```



#### [847. 访问所有节点的最短路径](https://leetcode-cn.com/problems/shortest-path-visiting-all-nodes/)

难度困难60收藏分享切换为英文关注反馈

给出 `graph` 为有 N 个节点（编号为 `0, 1, 2, ..., N-1`）的无向连通图。 

`graph.length = N`，且只有节点 `i` 和 `j` 连通时，`j != i` 在列表 `graph[i]` 中恰好出现一次。

返回能够访问所有节点的最短路径的长度。你可以在任一节点开始和停止，也可以多次重访节点，并且可以重用边。

**示例 1：**

```
输入：[[1,2,3],[0],[0],[0]]
输出：4
解释：一个可能的路径为 [1,0,2,0,3]
```

**示例 2：**

```
输入：[[1],[0,2,4],[1,3,4],[2],[1,2]]
输出：4
解释：一个可能的路径为 [0,1,4,2,3]
```

**提示：**

1. `1 <= graph.length <= 12`
2. `0 <= graph[i].length < graph.length`

```java
class Solution {
    public int shortestPathLength(int[][] graph) {
        int len=graph.length;
        if(graph==null || graph.length==0){
            return 0;
        }
        boolean[][] visited=new boolean[len][1<<len]; // 标记是否访问过,用于避免重复访问
        int finishState=(1<<len)-1;  // 用于检查是否访问完所有的节点,每个位代表一个节点的状态,形如1111
        Queue<int[]> queue=new LinkedList<>(); // 队列里的数组,第一个记录的是标号,第二个是状态
        for(int i=0; i<len; i++){
            queue.offer(new int[]{i,1<<i});
        }
        int step=0;
        while(!queue.isEmpty()){
            for(int i=queue.size(); i>0; i--){
                int[] node=queue.poll();
                if(finishState==node[1]){ // 如果标记的节点访问状态是结束,那么返回步长
                    return step;
                }
                for(int next:graph[node[0]]){
                    int nextState=node[1]|(1<<next); // 2个节点相或,标记着访问了这条边的2个点
                    if(visited[next][nextState]){//此处判断对方有没有搜到过这个状态（同样的搜寻过的点连在一起，如果为真，表面，此状态下有更短的搜寻路径，因为BFS
                        continue;
                    }
                    visited[next][nextState]=true;
                    queue.offer(new int[]{next,nextState}); // 将该节点和边的信息加入bfs对列
                }
            }
            step++;
        }
        return step;
    }
}
```



# DFS

#### [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

难度中等577收藏分享切换为英文关注反馈

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

**示例 1:**

```
输入:
11110
11010
11000
00000
输出: 1
```

**示例 2:**

```
输入:
11000
11000
00100
00011
输出: 3
解释: 每座岛屿只能由水平和/或竖直方向上相邻的陆地连接而成。
```

击败97%

```java
class Solution {
    int[][] change={{0,1},{0,-1},{-1,0},{1,0}};
    public int numIslands(char[][] grid) {
        if(grid==null||grid.length==0||grid[0].length==0)return 0;
        int m=grid.length,n=grid[0].length;
        boolean[][] visited=new boolean[m][n];
        int count=0;
        
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(grid[i][j]=='1'&&!visited[i][j]){
                help(grid,visited,i,m,j,n);
                count++;
                }
            }
        }
        return count; 
    }
    private void help(char[][] grid,boolean[][] visited,int i,int m,int j,int n){
        visited[i][j]=true;
        for(int[] num:change){
            int i2=num[0]+i;
            int j2=num[1]+j;
            if(i2>=0&&i2<m&&j2>=0&&j2<n&&grid[i2][j2]=='1'&&!visited[i2][j2]){
                help(grid,visited,i2,m,j2,n);
            }
        }
        return;
    }
}
```

#### [329. 矩阵中的最长递增路径](https://leetcode-cn.com/problems/longest-increasing-path-in-a-matrix/)

难度困难188收藏分享切换为英文关注反馈

给定一个整数矩阵，找出最长递增路径的长度。

对于每个单元格，你可以往上，下，左，右四个方向移动。 你不能在对角线方向上移动或移动到边界外（即不允许环绕）。

**示例 1:**

```
输入: nums = 
[
  [9,9,4],
  [6,6,8],
  [2,1,1]
] 
输出: 4 
解释: 最长递增路径为 [1, 2, 6, 9]。
```

**示例 2:**

```
输入: nums = 
[
  [3,4,5],
  [3,2,6],
  [2,2,1]
] 
输出: 4 
解释: 最长递增路径是 [3, 4, 5, 6]。注意不允许在对角线方向上移动。
```

击败75%

DFS加记忆

```java
class Solution {
    private int[][] direct={{-1,0},{1,0},{0,-1},{0,1}};
    private int[][] cash;
    private int len=1;
    private int m,n;
    public int longestIncreasingPath(int[][] matrix) {
      if(matrix==null||matrix.length==0||matrix[0].length==0){
          return 0;
      }
      m=matrix.length;
      n=matrix[0].length;
      cash=new int[m][n];
      for(int i=0;i<m;i++){
          for(int j=0;j<n;j++){
              len=Math.max(len,dfs(matrix,i,j));
          }
      }
      return len;  
    }
    private int dfs(int[][] matrix,int i,int j){
        if(cash[i][j]>0)return cash[i][j];
        for(int[] num:direct){
            int di=i+num[0];
            int dj=j+num[1];
            if(di<0||di>=m||dj<0||dj>=n||matrix[di][dj]<=matrix[i][j]){
                continue;
            }
            
            cash[i][j]=Math.max(cash[i][j],dfs(matrix,di,dj));
        }
         return ++cash[i][j];
    }
}
```



#### [1203. 项目管理](https://leetcode-cn.com/problems/sort-items-by-groups-respecting-dependencies/)

难度困难26收藏分享切换为英文关注反馈

公司共有 `n` 个项目和  `m` 个小组，每个项目要不没有归属，要不就由其中的一个小组负责。

我们用 `group[i]` 代表第 `i` 个项目所属的小组，如果这个项目目前无人接手，那么 `group[i]` 就等于 `-1`。（项目和小组都是从零开始编号的）

请你帮忙按要求安排这些项目的进度，并返回排序后的项目列表：

- 同一小组的项目，排序后在列表中彼此相邻。
- 项目之间存在一定的依赖关系，我们用一个列表 `beforeItems` 来表示，其中 `beforeItems[i]` 表示在进行第 `i` 个项目前（位于第 `i` 个项目左侧）应该完成的所有项目。

**结果要求：**

如果存在多个解决方案，只需要返回其中任意一个即可。

如果没有合适的解决方案，就请返回一个 **空列表**。

 

**示例 1：**

**![img](%E7%AE%97%E6%B3%95leetcode.assets/1359_ex1.png)**

```
输入：n = 8, m = 2, group = [-1,-1,1,0,0,1,0,-1], beforeItems = [[],[6],[5],[6],[3,6],[],[],[]]
输出：[6,3,4,1,5,2,0,7]
```

**示例 2：**

```
输入：n = 8, m = 2, group = [-1,-1,1,0,0,1,0,-1], beforeItems = [[],[6],[5],[6],[3],[],[4],[]]
输出：[]
解释：与示例 1 大致相同，但是在排序后的列表中，4 必须放在 6 的前面。
```

 

**提示：**

- `1 <= m <= n <= 3*10^4`
- `group.length == beforeItems.length == n`
- `-1 <= group[i] <= m-1`
- `0 <= beforeItems[i].length <= n-1`
- `0 <= beforeItems[i][j] <= n-1`
- `i != beforeItems[i][j]`

击败96%

**看不太懂**

```java
class Solution {
    // 组依赖图及访问标识（1正在访问，0未访问，-1已访问且无环）
    private Node[] grpGraph;
    private int[] grpVisited;
    // 项目依赖图及访问标识
    private Node[] prdGraph;
    private int[] prdVisited;

    // 项目与组映射关系
    private int[] prdToGrp;
    // 组与项目映射关系
    private Map<Integer, Set<Integer>> grpToPrd;
    // 节省时间
    int[] res;
    int idx = 0;

    public int[] sortItems(int n, int m, int[] group, List<List<Integer>> beforeItems) {
        if (m < 1 || n < 1 || n < m || n != group.length || n != beforeItems.size()) throw new IllegalArgumentException("invalid param");

        prdToGrp = group;
        grpToPrd = new HashMap<>();
        // 构建项目依赖图和组依赖图
        grpGraph = new Node[m + 1]; // 多一个存放-1，无组情况
        prdGraph = new Node[n];
        grpVisited = new int[m + 1];
        prdVisited = new int[n];

        // 遍历每个项目
        for (int i = 0; i < n; i++) {
            // 当前项目的组，如果没有组则分配组的下标为m
            int curGrp = group[i] == -1 ? m : group[i];
            if (grpToPrd.get(curGrp) == null) grpToPrd.put(curGrp, new HashSet<>());
            grpToPrd.get(curGrp).add(i);

            // 当前项目依赖的项目
            for (Integer item : beforeItems.get(i)) {
                prdGraph[i] = new Node(item, prdGraph[i]);

                int itemGrp = group[item] == -1 ? m : group[item];
                // 维护组依赖，杜绝自环情况
                if (curGrp != itemGrp) {
                    // 存在重复边，不影响拓扑排序
                    grpGraph[curGrp] = new Node(itemGrp, grpGraph[curGrp]);
                }
            }
        }

        // 根据组依赖拓扑排序，深度搜索
        res = new int[n];
        for (int i = 0; i <= m; i++) {
            // 存在环，返回空数组
            if (grpVisited[i] == 0 && !dfsParent(i)) {
                return new int[]{};
            }
        }
        
        return res;
    }

    private boolean dfsParent(int start) {
        // 正在遍历或已经遍历过，返回（1为有环，-1则不必遍历）
        if (grpVisited[start] != 0) return grpVisited[start] == -1;
        // 标记为正在遍历
        grpVisited[start] = 1;
        // 拓扑排序当前组
        Node temp = grpGraph[start];
        while (temp != null) {
            // 后继存在环，则返回失败
            if(!dfsParent(temp.ver)) return false;
            temp = temp.next;
        }
        Set<Integer> childVers = grpToPrd.getOrDefault(start, new HashSet<>());
        for (Integer cur : childVers) {
            // 组内项目循环依赖
            if (prdVisited[cur] == 0 && !dfsChild(cur)) return false;
        }
        // 遍历结束，设置标识
        grpVisited[start] = -1;
        return true;
    }

    private boolean dfsChild(int start) {
        // 组内存在环则是1，已遍历则是-1
        if (prdVisited[start] != 0) return prdVisited[start] == -1;

        // 标记为正在访问
        prdVisited[start] = 1; 
        Node temp = prdGraph[start];
        while (temp != null) {
            // 遍历与start同一分组的后继结点，如果后续路径不满足条件存在环，返回
            if (prdToGrp[start] == prdToGrp[temp.ver] && !dfsChild(temp.ver)) {
                return false;
            } 
            temp = temp.next;
        }
        // 子路径已遍历完
        res[idx++] = start;
        prdVisited[start] = -1; 
        return true;
    }
}

class Node {
    int ver;
    Node next;

    Node(int ver, Node next) {
        this.ver = ver;
        this.next = next;
    }
}

```





# 排序

#### [147. 对链表进行插入排序](https://leetcode-cn.com/problems/insertion-sort-list/)

难度中等154收藏分享切换为英文关注反馈

对链表进行插入排序。

![img](https://upload.wikimedia.org/wikipedia/commons/0/0f/Insertion-sort-example-300px.gif)
插入排序的动画演示如上。从第一个元素开始，该链表可以被认为已经部分排序（用黑色表示）。
每次迭代时，从输入数据中移除一个元素（用红色表示），并原地将其插入到已排好序的链表中。

 

**插入排序算法：**

1. 插入排序是迭代的，每次只移动一个元素，直到所有元素可以形成一个有序的输出列表。
2. 每次迭代中，插入排序只从输入数据中移除一个待排序的元素，找到它在序列中适当的位置，并将其插入。
3. 重复直到所有输入数据插入完为止。

 

**示例 1：**

```
输入: 4->2->1->3
输出: 1->2->3->4
```

**示例 2：**

```
输入: -1->5->3->4->0
输出: -1->0->3->4->5
```

击败98%

- 想要排序块，就要尽可能少的做比较
- 需要一个指针指向当前已排序的最后一个位置，这里用的是head指针
- 需要另外一个指针pre,每次从表头循环，这里用的是dummy表头指针。
- 每次拿出未排序的节点，先和前驱比较，如果大于或者等于前驱，就不用排序了，直接进入下一次循环
- 如果前驱小，则进入内层循环，依次和pre指针比较，插入对应位置即可。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    
    public ListNode insertionSortList(ListNode head) {
       if(head==null||head.next==null)return head;
       ListNode dummy=new ListNode(-1),pre;
       dummy.next=head;
       while(head!=null&&head.next!=null){
           if(head.val<=head.next.val){
               head=head.next;
               continue;
           }
           pre=dummy;
           while(pre.next.val<head.next.val)pre=pre.next;
           ListNode cur=head.next;
           head.next=cur.next;;
           cur.next=pre.next;
           pre.next=cur;
           
       }
       return dummy.next;
    }
}
```



#### [220. 存在重复元素 III](https://leetcode-cn.com/problems/contains-duplicate-iii/)

难度中等169收藏分享切换为英文关注反馈

给定一个整数数组，判断数组中是否有两个不同的索引 *i* 和 *j*，使得 **nums [i]** 和 **nums [j]** 的差的绝对值最大为 *t*，并且 *i* 和 *j* 之间的差的绝对值最大为 *ķ*。

**示例 1:**

```
输入: nums = [1,2,3,1], k = 3, t = 0
输出: true
```

**示例 2:**

```
输入: nums = [1,0,1,1], k = 1, t = 2
输出: true
```

**示例 3:**

```
输入: nums = [1,5,9,1,5,9], k = 2, t = 3
输出: false
```

**注意数值越界**

击败87%

桶排序是一种把元素分散到不同桶中的排序算法。接着把每个桶再独立地用不同的排序算法进行排序。桶排序的概览如下所示：

在上面的例子中，我们有 8 个未排序的整数。我们首先来创建五个桶，这五个桶分别包含 [0,9], [10,19], [20, 29], [30, 39],[40,49] 这几个区间。这 8 个元素中的任何一个元素都在一个桶里面。对于值为 x 的元素来说，它所属桶的标签为 x/w，在这里我们让 w = 10。对于每个桶我们单独用其他排序算法进行排序，最后按照桶的顺序收集所有的元素就可以得到一个有序的数组了。

回到这个问题，我们尝试去解决的最大的问题在于：

对于给定的元素 x, 在窗口中是否有存在区间 [x-t, x+t][x−t,x+t] 内的元素？
我们能在常量时间内完成以上判断嘛？
我们不妨把把每个元素当做一个人的生日来考虑一下吧。假设你是班上新来的一位学生，你的生日在 三月 的某一天，你想知道班上是否有人生日跟你生日在 t=30 天以内。在这里我们先假设每个月都是30天，很明显，我们只需要检查所有生日在 二月，三月，四月 的同学就可以了。

之所以能这么做的原因在于，我们知道每个人的生日都属于一个桶，我们把这个桶称作月份！每个桶所包含的区间范围都是 t，这能极大的简化我们的问题。很显然，任何不在同一个桶或相邻桶的两个元素之间的距离一定是大于 t 的。

我们把上面提到的桶的思想应用到这个问题里面来，我们设计一些桶，让他们分别包含区间 ..., [0,t], [t+1, 2t+1], ......。我们把桶来当做窗口，于是每次我们只需要检查 x 所属的那个桶和相邻桶中的元素就可以了。终于，我们可以在常量时间解决在窗口中搜索的问题了。

还有一件值得注意的事，这个问题和桶排序的不同之处在于每次我们的桶里只需要包含最多一个元素就可以了，因为如果任意一个桶中包含了两个元素，那么这也就是意味着这两个元素是 足够接近的 了，这时候我们就直接得到答案了。因此，我们只需使用一个标签为桶序号的散列表就可以了。

```java
public class Solution {
    // Get the ID of the bucket from element value x and bucket width w
    // In Java, `-3 / 5 = 0` and but we need `-3 / 5 = -1`.
    private long getID(long x, long w) {
        return x < 0 ? (x + 1) / w - 1 : x / w;
    }

    public boolean containsNearbyAlmostDuplicate(int[] nums, int k, int t) {
        if (t < 0) return false;
        Map<Long, Long> d = new HashMap<>();
        long w = (long)t + 1;
        for (int i = 0; i < nums.length; ++i) {
            long m = getID(nums[i], w);
            // check if bucket m is empty, each bucket may contain at most one element
            if (d.containsKey(m))
                return true;
            // check the nei***or buckets for almost duplicate
            if (d.containsKey(m - 1) && Math.abs(nums[i] - d.get(m - 1)) < w)
                return true;
            if (d.containsKey(m + 1) && Math.abs(nums[i] - d.get(m + 1)) < w)
                return true;
            // now bucket m is empty and no almost duplicate in nei***or buckets
            d.put(m, (long)nums[i]);
            if (i >= k) d.remove(getID(nums[i - k], w));
        }
        return false;
    }
}
```



#### [1366. 通过投票对团队排名](https://leetcode-cn.com/problems/rank-teams-by-votes/)

难度中等20收藏分享切换为英文关注反馈

现在有一个特殊的排名系统，依据参赛团队在投票人心中的次序进行排名，每个投票者都需要按从高到低的顺序对参与排名的所有团队进行排位。

排名规则如下：

- 参赛团队的排名次序依照其所获「排位第一」的票的多少决定。如果存在多个团队并列的情况，将继续考虑其「排位第二」的票的数量。以此类推，直到不再存在并列的情况。
- 如果在考虑完所有投票情况后仍然出现并列现象，则根据团队字母的字母顺序进行排名。

给你一个字符串数组 `votes` 代表全体投票者给出的排位情况，请你根据上述排名规则对所有参赛团队进行排名。

请你返回能表示按排名系统 **排序后** 的所有团队排名的字符串。

 

**示例 1：**

```
输入：votes = ["ABC","ACB","ABC","ACB","ACB"]
输出："ACB"
解释：A 队获得五票「排位第一」，没有其他队获得「排位第一」，所以 A 队排名第一。
B 队获得两票「排位第二」，三票「排位第三」。
C 队获得三票「排位第二」，两票「排位第三」。
由于 C 队「排位第二」的票数较多，所以 C 队排第二，B 队排第三。
```

**示例 2：**

```
输入：votes = ["WXYZ","XYZW"]
输出："XWYZ"
解释：X 队在并列僵局打破后成为排名第一的团队。X 队和 W 队的「排位第一」票数一样，但是 X 队有一票「排位第二」，而 W 没有获得「排位第二」。 
```

**示例 3：**

```
输入：votes = ["ZMNAGUEDSJYLBOPHRQICWFXTVK"]
输出："ZMNAGUEDSJYLBOPHRQICWFXTVK"
解释：只有一个投票者，所以排名完全按照他的意愿。
```

**示例 4：**

```
输入：votes = ["BCA","CAB","CBA","ABC","ACB","BAC"]
输出："ABC"
解释： 
A 队获得两票「排位第一」，两票「排位第二」，两票「排位第三」。
B 队获得两票「排位第一」，两票「排位第二」，两票「排位第三」。
C 队获得两票「排位第一」，两票「排位第二」，两票「排位第三」。
完全并列，所以我们需要按照字母升序排名。
```

**示例 5：**

```
输入：votes = ["M","M","M","M"]
输出："M"
解释：只有 M 队参赛，所以它排名第一。
```

 

**提示：**

- `1 <= votes.length <= 1000`
- `1 <= votes[i].length <= 26`
- `votes[i].length == votes[j].length` for `0 <= i, j < votes.length`
- `votes[i][j]` 是英文 **大写** 字母
- `votes[i]` 中的所有字母都是唯一的
- `votes[0]` 中出现的所有字母 **同样也** 出现在 `votes[j]` 中，其中 `1 <= j < votes.length`

击败54%

**List<Map.Entry<Character,int[]>> list=new ArrayList<>(map.entrySet())**

方法不知道，需切记；

利用map集合Map<Character,int[]> map=new HashMap<>();得到每一个团队的每个名次的投票数的数组，然后对这个数组按照投票数排序，有相等的，再按照字母顺序排序，最后根据这个输出团队名称即可；

```java
class Solution {
    public String rankTeams(String[] votes) {
     if(votes==null||votes.length==0||votes[0].length()==0)return null;
     if(votes.length==1)return votes[0];
     int n=votes.length;
     int m=votes[0].length();
     Map<Character,int[]> map=new HashMap<>();
     String ss=votes[0];
     for(int i=0;i<m;i++){
         char c=ss.charAt(i);
         map.put(c,new int[m]);
     }
     for(String s:votes){
         for(int i=0;i<m;i++){
             char c=s.charAt(i);
             int[] matrix=map.get(c);
             matrix[i]=matrix[i]+1;
             map.put(c,matrix);
         }
     }
     List<Map.Entry<Character,int[]>> list=new ArrayList<>(map.entrySet());
     Collections.sort(list,(tem1,tem2)->{
         int[] matrix1=tem1.getValue();
         int[] matrix2=tem2.getValue();
         //根据投票排序
         for(int i=0;i<m;i++){
             if(matrix1[i]!=matrix2[i]){
                 return matrix2[i]-matrix1[i];//降序
             }
         }
         //字母顺序排序
         return tem1.getKey()-tem2.getKey();//升序
     });
        StringBuilder sb=new StringBuilder();
        for(int i=0;i<m;i++){
            sb.append(list.get(i).getKey());
        }
     return sb.toString();
    }
}
```

