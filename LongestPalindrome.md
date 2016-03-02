# 最长回文字符子串的解法(python版)
> 问题描述：求一个字符串S的最长回文字符子串

这个有趣的问题已经被著名的Greplin 编程挑战收录，并且是一个经常在面试中被问到的问题。之所以这么出名，因为这个问题可以有很多种解法。就我知道的就有5种完全不同的解法。准备好迎接挑战了么~？

> 提示：
> 首先你得清楚的了解什么是回文字符串。回文字符串就是是一个正反读起来都是一样的字符串。例如，"aba"就是回文字符串，"abc"则不是

## 一个常见的错误
有些人可能会想到下面这样一个快速的解法，但是很不幸的是它是错误的（但是还是可以在此基础上进行修改的）：

> 把字符串S反转变成S'。找到二者的最长公共子串，那就是最长回文子串。

看起来这个解法似乎是对的，下面我们用实际的例子来看看吧。

例如：S="caba" S'="abac"
最长公共子串是"aba"，也是正确的答案

让我们试试另一个例子：
S="abacdfgdcaba" S'="abacdgfdcaba"
最长公共子串则是"abacd".很明显，这并不是一个回文子串，也就不是正确的解。

我们可以发现上面的方法当字符存在一个非回文，反转的字符子串在S的其他部分时会出错。为了更正这个问题，每次我们找到一个最长公共子串，我们需要检查这个子串是不是回文子串。如果是，我们就记录下这个回文子串，否则就跳过它继续寻找。

这给了我们一个: ![](http://latex.codecogs.com/png.latex?${O(N^{2})$)的DP算法，使用空间是: ![](http://latex.codecogs.com/png.latex?${O(N^{2})$).

## 暴力解决法——![](http://latex.codecogs.com/png.latex?${O(N^{3})$)：
最简单的暴力解决办法就是找到所有可能的字符子串然后确定它是不是回文字符串。总共有C(n,2)种字符子串(除了单个字符的字符串)

由于判断每个子串需要O(N)时间，这个算法的时间复杂度就是![](http://latex.codecogs.com/png.latex?${O(N^{3})$).

## 动态规划解法，![](http://latex.codecogs.com/png.latex?${O(N^{2})$)时间和![](http://latex.codecogs.com/png.latex?${O(N^{2})$)空间

为了获得明显提示，我们需要尽量避免在判断回文子串中不必要的重复计算。比如，"ababa"如果我们已经知道"bab"是一个回文字符串，明显的"ababa"一定是个回文字符串因为最左最右的字符是一样的。

更精确的描述如下：
> 1.  P[i,j]=true 表示字符子串S[i...j]是否是回文子串
> 2.  初始化：P[i,i]=true(0<=i<=n-1) P[i,i-1]=true(1<=i<=n-1)
> 3.  P[i,j]=(P[i+1,j-1] and S[i]=S[j])

在动态规划中保存最长回文长度和起点即可：

```python
def longestPalindromeDP(s):
    #st=time.time()
    n=len(s)
    if n<=1:
        return s
    mylist=[([0]*n) for i in range(n)]
    #生成N*N数组
    left,right=0,0
    mylist[0][0]=True
    for i in range(n):
        mylist[i][i],mylist[i][i-1]=True,True
    #初始化数组
    for k in range(2,n+1):
        #枚举子串长度
        for i in range(n-k):
            #枚举子串起始位置
            if s[i]==s[i+k-1] and mylist[i+1][i+k-2]:
                mylist[i][i+k-1]=True
                if right-left+1 < k:
                    left = i 
                    right = i+k-1
    #print(time.time()-st)
    return s[left:right+1]

```

## 简单的中心扩展方法![](http://latex.codecogs.com/png.latex?${O(N^{2})$)时间和O(1)空间

我们可以观察到回文字符串是中心对称的，因此，一个回文字符串可以从它的中心向外扩展，一共只有2N-1个这样的中心。

你可能会问为什么是2N-1个而不是N个，原因是偶数回文的字符中心是在2个字符之间的，例如"abba"这个字符串的中心是在两个'b'之间的。

因为扩展一个回文字符串需要花费O(N)的时间，总共的时间复杂度就是![](http://latex.codecogs.com/png.latex?${O(N^{2})$)。

```python
def longestPalindrome(s):
    #st=time.time()
    l=len(s)
    if l<=1:
        return s
    start=maxlen=0
    for i in range(l):
        low,high=i-1,i
        while low>=0 and high<l and s[low]==s[high]:
        #遍历偶数字符的子串
            low-=1
            high+=1
        if high-low-1>maxlen:
            maxlen=high-low-1
            start=low+1
        low,high=i-1,i+1
        while low >=0 and high<l and s[low]==s[high]:
        #遍历奇数字符的子串
            low-=1
            high+=1
        if high-low-1>maxlen:
            maxlen=high-low-1
            start=low+1
    #print(time.time()-st)
    return s[start:maxlen+start]
```

## 一个O(N)的解法(Manacher算法)

> 这是一个不可思议的解法，完全不能想象还有这种解法，这大概就是算法之美吧。

首先我们对字符串S做如下处理，在S相邻字符之间插入'#'。这么做的原因你很快就会知道。

例如：S="abaaba", T="#a#b#a#a#b#a#"

为了找到最长回文子串，我们需要扩展每个T[i],比如T[i-d]到T[i+d]就构成一个回文字符串。你应该会马上发现d就是中心在T[i]的回文字符串长度

我们把结果存在P数组中，P[i]代表了以T[i]为中心的回文字符串长度。最长回文字符子串就是P中最大的元素。

用上面的例子我们可以得到下面的结果：
*  T = # a # b # a # a # b # a #
*  P = 0 1 0 3 0 1 6 1 0 3 0 1 0

观察P数组我们可以马上得到最长子串是"abaaba"，因为P[6]=6

现在你应该注意到了我们插入的'#'字符的作用了，无论是奇数还是偶数个的回文字符串都可以按照奇数来处理了。

现在我们应该尝试一个更加复杂的例子了：
S="babcbabcbaccba"
![](http://articles.leetcode.com/wp-content/uploads/2011/11/palindrome_table10.png)
> 上面图片是经过处理之后的S，竖线C表示"abcbabcba"的中心。虚线L和R表示它的边缘。你现在所在位置是i，他关于C的镜像是i'。现在考虑下怎么样有效的计算P[i]

假定我们已经到了i=13这个点，并且我们需要计算P[13]。我们可以看见i关于C的镜像i'的下标是9.
![](http://articles.leetcode.com/wp-content/uploads/2011/11/palindrome_table11.png)

你可以从上图发现，P[i]=P[i']=1,这揭示了回文字符串中心对称的一个属性。事实上，C后面3个字符都是对称的，P[12]=P[10]=0,P[13]=[9]=1,P[14]=P[8]=0

![](http://articles.leetcode.com/wp-content/uploads/2011/11/palindrome_table4.png)

现在我们到了i=15的位置，P[i]的值又是多少呢？如果我们按照对称的属性，P[i]的值应该等于P[i']=7。但是明显这是不对的，我们以T[15]为中心向2边扩展，得到的回文字符串是"a#b#c#b#a"，比我们实际期望的要小，这是为什么呢？

![](http://articles.leetcode.com/wp-content/uploads/2011/11/palindrome_table5.png)

绿线覆盖的2区域明显的是匹配的。穿过中心区域的部分一定也是对称的。这里请注意P[i']是7因为它可以扩展超出L边缘（红线区域，这导致了它不再拥有我们推想的对称属性了）我们知道P[i]>=5，为了计算P[i]的值我们需要继续向两边匹配字符。在这个例子里，因为P[21]≠P[1],所以P[i]=5

现在我们来总结一下这个算法的核心部分：
> if P[i']<=R-i
> then P[i]=P[i']
> else P[i]>=P[i'](这意味着我们必须向右扩展来计算P[i])

最好一部分是确定什么时候我们需要把C和R向右移动：
> 如果回文中心在i需要扩展超过R,我们把C更新到i(新的回文中心)并且把R扩展到新的回文右边界。

```python
class Solution:
    #Manacher algorithm
    #http://en.wikipedia.org/wiki/Longest_palindromic_substring

    def longestPalindrome(self, s):
        # Transform S into T.
        # For example, S = "abba", T = "^#a#b#b#a#$".
        # ^ and $ signs are sentinels appended to each end to avoid bounds checking
        T = '#'.join('^{}$'.format(s))
        n = len(T)
        P = [0] * n
        C = R = 0
        for i in range (1, n-1):
            P[i] = (R > i) and min(R - i, P[2*C - i]) # equals to i' = C - (i-C)
            # Attempt to expand palindrome centered at i
            while T[i + 1 + P[i]] == T[i - 1 - P[i]]:
                P[i] += 1

            # If palindrome centered at i expand past R,
            # adjust center based on expanded palindrome.
            if i + P[i] > R:
                C, R = i, i + P[i]

        # Find the maximum element in P.
        maxLen, centerIndex = max((n, i) for i, n in enumerate(P))
        return s[(centerIndex  - maxLen)//2: (centerIndex  + maxLen)//2]
```

## 结语

最近开始刷leetcode来熟悉python语法，作为一个算法苦手简直是被虐的心疼。不过接触了解了更多算法的原理，这篇文章主要是翻译了leetcode上面的一篇文章，并且把算法都改为Python实现了。翻译的不是很信达雅，有问题的同学可以移步原帖。


> 参考：
> http://blog.csdn.net/suool/article/details/38383045
> http://articles.leetcode.com/longest-palindromic-substring-part-ii
> http://articles.leetcode.com/longest-palindromic-substring-part-i




