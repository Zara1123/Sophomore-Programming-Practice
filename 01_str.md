# 交替合并字符串

题目：给你两个字符串 word1 和 word2 。请你从 word1 开始，通过交替添加字母来合并字符串。如果一个字符串比另一个字符串长，就将多出来的字母追加到合并后字符串的末尾。返回 合并后的字符串 。

解答：
```
class Solution:
    def mergeAlternately(self, word1: str, word2: str) -> str:
        ls = [];
        for(int i=0,i<word1.length or i<word2.length,i++){
            ls = ls.append(word1[i]);
            ls = ls.append(word2[i]);
        }
        if (word1.length > word2.length){
            for(i=word1.length,i<word1.length,i++){ls = ls.append(word2[i]);}
        }
        else if(word1.length < word2.length)
        {
            for(i=word2.length,i<word2.length,i++){ls = ls.apend(word1[i]);}
        }
        return "".jion(ls)
        ```
        
1.为什么新建空列表而不是字符串？
核心原因：Python 中字符串是不可变的（immutable），而列表是可变的（mutable）。
```
result = ""
for ch in word1:
    result += ch  # 每次都创建了一个全新的字符串对象！
```

2.列表的增添操作
```
ls.append("a")   # append 是原地修改，返回值是 None
ls = ls.append("a")  # 等于把 None 赋给了 ls！列表直接丢了！
```
3.拼写错误
```
ls.apend(...) → 应为 append（少了个 p）
"".jion(ls) → 应为 join（字母顺序反了
```
4.python与JAVA/C语言混淆
正确写法：
python版
```
class Solution:
    def mergeAlternately(self, word1: str, word2: str) -> str:
        ls = []
        for i in range(max(len(word1),len(word2))):#取较长的为循环次数
            if i < len(word1):
                ls.append(word1[i])
            if i <len(word2):
                ls.append(word2[i])
        return "".join(ls)

分析：
-这一段循环执行了两次判断，不会导致时间复杂度变高吗
不会，时间复杂度依然是 O(n)。每次循环做的操作：2 次比较（i < len(...)）->最多2次 append；设 n = max(len(word1), len(word2))，总操作数 ≈ 4n 次（常数个操作 × n 次循环）。而$O(4n)=O(2n)=O(n+100)=O(n)$，真正让复杂度升级的是嵌套和递归。

一个消掉if的更优雅写法
```
# Python 版思路
def mergeAlternately(self, word1, word2):
    ls = []
    for i in range(min(len(word1), len(word2))):  # 先走共同长度，绝对安全
        ls.append(word1[i])
        ls.append(word2[i])
    ls.append(word1[min(len(word1),len(word2)):]) # 一次性接上长串的尾巴
    ls.append(word2[min(len(word1),len(word2)):])
    return "".join(ls)

```

```
java版
```
class Solution {
    public String mergeAlternately(String word1, String word2) {
        StringBuilder sb = new StringBuilder();          // ≈ Python 的 ls = []
        int n = Math.max(word1.length(), word2.length()); // ≈ max(len(...))
        for (int i = 0; i < n; i++) {
            if (i < word1.length()) {
                sb.append(word1.charAt(i));              // ≈ ls.append(word1[i])
            }
            if (i < word2.length()) {
                sb.append(word2.charAt(i));
            }
    }
}

```

C 版（最硬核，让你看到内存的真面目）

char * mergeAlternately(char * word1, char * word2){
    int len1 = strlen(word1);
    int len2 = strlen(word2);
    int n = len1 > len2 ? len1 : len2;      // 三目运算符，≈ max()

    // 关键区别：C 没有动态数组，必须提前算好总大小，一次性申请
    char *result = (char *)malloc(len1 + len2 + 1);  // +1 是给结束符 '\0' 的
    int idx = 0;                             // C 要自己维护"写到第几个格子了"

    for (int i = 0; i < n; i++) {
        if (i < len1) {
            result[idx++] = word1[i];        // idx++ ：先赋值，再自增
        }
        if (i < len2) {
            result[idx++] = word2[i];
        }
    }
    result[idx] = '\0';                      // 手动补上字符串结束标志！
    return result;
}
C 版和前两个的三大本质区别
① 没有 append，因为“提前知道答案有多大”

Python/Java 的 append 之所以需要动态扩容，是因为写代码时不知道最终长度。但这道题 len1 + len2 一眼就算出来了——所以 C 直接 malloc 一块刚好大的内存，一次到位，零次搬家。

这就是为什么我说均摊分析重要：动态数组是“为未知长度付出的保险费”。一旦长度已知（数仓里建表指定字段长度、C 里 malloc 精确大小），就不需要这份保险。这也是数仓开发中“建表时合理预估字段长度”的底层逻辑。

② idx 要自己数

Python 的 append 帮你记着“该塞第几个格子了”，C 没这服务，idx++ 就是你在手动维护动态数组的 size 字段。

③ '\0' 结束符

C 的字符串没有 length 属性，全靠结尾的 '\0' 标记“到这结束了”。忘写这行，strlen 读返回字符串时会一直往后读到内存 garbage 为止——这就是无数黑客攻击的源头（缓冲区溢出）。

# 字符串的最大公因子

题目：对于字符串 s 和 t，只有在 s = t + t + t + ... + t + t（t 自身连接 1 次或多次）时，我们才认定 “t 能除尽 s”。给定两个字符串 str1 和 str2 。返回 最长字符串 x，要求满足 x 能除尽 str1 且 x 能除尽 str2 。

```
import math
class Solution:
    def gcdOfStrings(self, str1: str, str2: str) -> str:
        if str1 + str2 != str2 + str1:#拼接相等 ⟺ 存在公共基串
            return ""
        return str1[: math.gcd(len(str1), len(str2))]#基串长度 = gcd(m,n)，且必是 str1 的前缀
```


| 重要定理：str1 + str2 == str2 + str1 是"存在公共基串"的充要条件。

gcd = 最大公因数（最大公约数）
gcd 是 greatest common divisor 的缩写，就是数学里的“最大公因数”。从头说起：

第一步：什么是“因数”（约数）
如果 a 能被 b 整除（没有余数），就说 b 是 a 的因数。
第二步：什么是“公因数”
两个数共同拥有的因数。
第三步：什么是“最大”公因数
公因数里最大的那个。
gcd(12, 18) = 6，6是12和18的最大公因数
math.gcd
就是 Python 自带的求最大公因数函数：


【拓展】
1.暴力枚举法(遍历-检查长度-检查字符)
## 
```
        for L in range(min(len(str1), len(str2)), 0, -1):
        #假设x的长度为L，从较小字符串的长度开始由大到小遍历
        #第一步检查：选择较小的字符串长度
            if len(str1) % L == 0 and len(str2) % L == 0:
            #第二步检查：长度。L 能整除 str1 的长度——str1 的总长度能被恰好切成若干个 L 宽的块，str2同理。
                cand = str1[:L]
                #cand即candidate“候选者”，cand为取出str1的前L个字符。
                if cand * (len(str1) // L) == str1 and cand * (len(str2) // L) == str2:
                #第三步检查：字符。len(str1)//L为L整除str1的长度取商，k1。cand*k1：将k1个候选者拼接。总：k1个cand拼接是否得到str1。str2同理。
                    return cand
        return ""
```
2.规律
由`if cand * (len(str1) // L) == str1 and cand * (len(str2) // L) == str2:`得：
（str1 + str2 == str2 + str1的必要性）
$$
str1 + str2 = x×k1 + x×k2 = x×(k1+k2)
str2 + str1 = x×k2 + x×k1 = x×(k2+k1)
$$
拼接可以交换顺序

# 得到最多糖果的孩子
暴力求解版
初稿：
class Solution:
    def kidsWithCandies(self, candies: List[int], extraCandies: int) -> List[bool]:
        result = []
        for i in range(len(candies)){
            candies[i] = candies[i] + entraCandies
            for j in range(i+1,len(candies))
                if  candies[i] > candies[j]
                    result[i] = True
                else result[i] = False
        }
        return result
改错：
1.python划分代码块不用{},用缩进for 和 if 行尾缺冒号
2. 原地修改 candies，污染后续判断
eg.candies = [2, 1], extraCandies = 3
我的代码：i=0 时 candies[0] 被改成 5；
         i=1 时 1+3=4，跟被污染的 5 比 → 4 < 5 → False，错
3.只和后面的孩子比，漏掉了前面的

改错后：

```
class Solution:
    def kidsWithCandies(self, candies: List[int], extraCandies: int) -> List[bool]:
        n = len(candies)
        result = [False] * n                # ① 先建好结果盒子
        for i in range(n):
            give = candies[i] + extraCandies  # ② 用临时变量，不动原数组
            b = True
            for j in range(n):                # ③ 和"所有"孩子比（0 到 n-1）
                if give < candies[j]:         # ④ 只要有一个人比他高
                    b = False                 #    就不算最多
                    break                     #    break 只跳出内层循环，函数还活着
            result[i] = b                     # ⑤ 记录，不 return
        return result                         # ⑥ 全部判完，整体返回

```
简化算法
```
class Solution:
    def kidsWithCandies(self, candies: List[int], extraCandies: int) -> List[bool]:
        m = max(candies)                                  # 只找一次最大值
        return [c + extraCandies >= m for c in candies]   # 每人够不够得着
```
1.回顾列表推导式
[  c + extraCandies >= m   for c in candies  ]
   └──────┬──────────┘     └───────┬───────┘
     每轮要算出的元素          循环本身（从 candies 逐个取 c，c是糖果数）

2.列表循环的用法
|想要的|正确写法|
|:---:|:---:|
|只要每个元素的值|for c in candies:|
|只要位置|for i in range(len(candies)):|
|位置和值都要|for i, c in enumerate(candies):|

# 种花问题（贪心算法）

题目：
假设有一个很长的花坛，一部分地块种植了花，另一部分却没有。可是，花不能种植在相邻的地块上，它们会争夺水源，两者都会死去。

给你一个整数数组 flowerbed 表示花坛，由若干 0 和 1 组成，其中 0 表示没种植花，1 表示种植了花。另有一个数 n ，能否在不打破种植规则的情况下种入 n 朵花？能则返回 true ，不能则返回 false 。

示例 1：

输入：flowerbed = [1,0,0,0,1], n = 1
输出：true
示例 2：

输入：flowerbed = [1,0,0,0,1], n = 2
输出：false
 

提示constraints(题目对输入数据的保证)：

1 <= flowerbed.length <= 2 * 104
flowerbed[i] 为 0 或 1
flowerbed 中不存在相邻的两朵花
0 <= n <= flowerbed.length

1.n 最大 2 万，说明 O(n) 一次遍历绰绰有余，不需要也不该写更复杂的解法。反之如果题目给 10⁹，就是在暗示你必须用 O(log n) 或数学解。
2.注意 n=0 的坑：n=0 时不管花坛什么样都应返回 True（一朵花都不种当然"不打破规则"），要先处理 n==0 再谈合法性检查。

解答：（边界检查版）
```
class Solution:
    def canPlaceFlowers(self, flowerbed: list[int], n: int) -> bool:
        count = 0
        m = len(flowerbed)
        if n = 0:return True
        for i in range(l):
            if flowerbed[i] == 0:
                left_ok  = (i == 0) or (flowerbed[i-1] == 0)
                right_ok = (i == m - 1) or (flowerbed[i+1] == 0) 
                if left_ok and right_ok:
                    flowerbed[i] = 1 
                    count += 1
                    if count >= n:
                        return True
        return count >= n           
```

改错：判断语句应用“=”，`if n = 0:`应该为`if n == 0:`
优化：(补0版)
1.列表的拼接操作——把三个列表首尾相连，串成一个新的长列表：
'flowerbed = [0] + [flowerbed] + [0]'
2.每一块真实的地，左右都有了邻居，因此无需特别的边界检查，可以统一检查有无种花：
`if flowerbed[i] == 0 and flowerbed[i-1] == 0 and flowerbed[i+1] == 0:`
3.真实的地在 1 到 len(flowerbed)-2 之间，所以循环要写成：
```
for i in range(1,len(flowerbed)-1):
    if flowerbed == 0 and flowerbed[i-1] == 0 and flowerbed[i+1] == 0:
        flowerbed[i] = 1
        count += 1
```
4.可以删掉if n==0的特判
考虑n=0，n=0时不管种没种，都有 count >= 0 恒成立 → 返回 True ✅return count >= n

最终代码：
```
class Solution:
    def canPlaceFlowers(self, flowerbed: list[int], n: int) -> bool:
        count = 0
        flowerbed = [0] + flowerbed + [0]
        if n == 0:
            return True
        for i in range(1,len(flowerbed)-1):
            if flowerbed[i] == 0 and flowerbed[i-1] == 0 and flowerbed[i+1] == 0:
                flowerbed[i] = 1
                count += 1
                if count >= n:
                    return True
        return count >= n
```

算法思想：贪心算法（在每一步都做“当下看起来最划算”的选择，并且做完绝不反悔、不回头重来。）
“遇到能种的地方就马上种”就是贪心的选择。
