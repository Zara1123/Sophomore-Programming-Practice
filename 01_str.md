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
        if str1 + str2 != str2 + str1:
            return ""
        return str1[: math.gcd(len(str1), len(str2))]
```

str1 + str2 == str2 + str1 是"存在公共除数字"的充要条件。

1.若两者都能被 x 除尽，设 str1 = x*k1、str2 = x*k2，则 str1+str2 = x*(k1+k2) = str2+str1，两种拼接必然相等。
2.若两种拼接相等，可以证明两者必由同一个基串重复构成（可用归纳法或辗转相除法
核心等式：gcd(a, b) = gcd(b, a mod b)，直到 b = 0 时，a 就是答案
3.答案长度：公共除数字 x 的长度必须同时整除 len(str1) 和 len(str2)，且要最长，所以取 gcd(len(str1), len(str2))。
4.返回值：直接取 str1 的前 gcd 个字符（str2 的前缀完全相同）。





