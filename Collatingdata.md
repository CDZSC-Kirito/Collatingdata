# 字符串专题

## 字符串匹配算法

字符串模式匹配问题：给定一个子串，要求在某个字符串中找出与该子串相同的所有子串，这就是模式匹配。

### 哈希算法 (Hash)

关于字符串哈希，就一句话：把字符串有效地，**“唯一”**的映射(通过哈希函数)到每个**“不同”**的整数，我们就能很好的处理字符串。 判断两个字符串是否一致，直接用他们的哈希值判断即可，若哈希值一致，则**认为**字符串一致；

常见的哈希算法：BKDRHash，APHash，DJBHash，JSHash，RSHash，SDBMHash，PJWHash，ELFHash，多重Hash，高维Hash。

一个简单的哈希函数：$Hash[i]=Hash[i-1]*P+idx(S[i])\%MOD\left ( i\subseteq [1,|S|],S[i]\overset{Mapping}{\rightarrow}>idx(S[i])\right )$ 

这表示第i个前缀的哈希值，是一个哈希的前缀和，那么对于字符串S的子串$S[l\cdots r]$的哈希值：

$$Hash[l\cdots r]=(Hash[r]-Hash[l]*P^{^{r-l+1}})+MOD)\%MOD \left ( l\subseteq [1,|S|],r\subseteq [1,|S|],r\geq l \right )$$ 

哈希算法非常强大，能解决的问题：单模式匹配，树的同构，高维字符串匹配，回文串，LCP等。

**推荐阅读：**

[各种字符串Hash函数比较](https://www.byvoid.com/zhs/blog/string-hash-compare) 

 [字符串hash以及7大问题(参见附件)](https://wenku.baidu.com/view/b7d3d1c6804d2b160a4ec090.html)  

[2005年国家集训队论文-杨弋《Hash在信息学竞赛中的一类应用》(参见附件)](https://wenku.baidu.com/view/6bfad1d8d15abe23482f4d7f.html) 

**模板代码**

```c++
struct StringHash {
	const int Speed = 1331; //基数，素数为优
	ULL Hash[MAX]; //前缀哈希值
	void GetHash(char *str) { //初始化前缀哈希值
		for (int i = 0,len=strlen(str); i < len; i++) {
			Hash[i] = i ? (Hash[i-1]*Speed+str[i]) : (str[i]);
		}
	}
	ULL PowMod(ULL a, ULL x) { 
		ULL result = 1;
		for (; x; x >>= 1, a *= a) {
			if (x & 1) result *= a;
		}
		return result;
	}
	ULL GetHashValue(int l, int r) { //求子串str[l...r]的哈希值
		return l ? Hash[r] - Hash[l - 1] * PowMod(Speed, r - l + 1) : Hash[r];
	}
};
```

**经典题目**

POJ-3461——[Oulipo](http://poj.org/problem?id=3461)

题意：给定2个字符p和t。问p在t中出现几次

分析：求出p整体的哈希值于t中每个长度为|p|的子串的哈希值比较即可。

代码：

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <functional>
using namespace std;
typedef long long int LL;
typedef unsigned long long int ULL; //unsigned自然溢出
const int MAX = 1e6+24; //字符串长度
struct StringHash {
	const int Speed = 1331; //基数，素数为优
	ULL Hash[MAX]; //前缀哈希值
	void GetHash(char *str) { //初始化前缀哈希值
		for (int i = 0,len=strlen(str); i < len; i++) {
			Hash[i] = i ? (Hash[i-1]*Speed+str[i]) : (str[i]);
		}
	}
	ULL PowMod(ULL a, ULL x) { 
		ULL result = 1;
		for (; x; x >>= 1, a *= a) {
			if (x & 1) result *= a;
		}
		return result;
	}
	ULL GetHashValue(int l, int r) { //求子串str[l...r]的哈希值
		return l ? Hash[r] - Hash[l - 1] * PowMod(Speed, r - l + 1) : Hash[r];
	}
};
char P[MAX], T[MAX];
StringHash shp, sht;
int Count(char *p, char *t) {
	shp.GetHash(p); sht.GetHash(t);
	int lenp = strlen(p), lent = strlen(t),result=0;
	ULL hashP = shp.GetHashValue(0, lenp - 1);
	for (int i = 0; i <= lent - lenp; i++) {
		if (hashP ==sht.GetHashValue(i,i+lenp-1)) {
			result++;
		}
	}
	return result;
}
int main() {
	int t;
	scanf("%d", &t);
	while (t--) {
		scanf("%s%s", P,T);
		printf("%d\n",Count(P,T));
	}
	return 0;
}
```



### BM算法(Boyer-Moore)

Boyer-Moore算法是一种基于后缀匹配的模式串匹配算法，后缀匹配就是模式串从右到左开始比较，但模式串的移动还是从左到右的。字符串匹配的关键就是模式串的如何移动才是最高效的，Boyer-Moore为了做到这点定义了两个规则：坏字符规则和好后缀规则，下面图解给出定义：

![BM-1](http://dl.iteye.com/upload/attachment/0075/1867/3b7f14ac-4282-34ed-821e-e68d2228e18f.png)

下面分别针对利用坏字符规则和好后缀规则移动模式串进行介绍：

**坏字符规则** 

1.如果坏字符没有出现在模式字符中，则直接将模式串移动到坏字符的下一个字符：

![BM-2](http://dl.iteye.com/upload/attachment/0075/1871/891ff664-4ad2-3687-85de-2f17d691e169.png)

 （坏字符c，没有出现模式串P中，直接将P移动c的下一个位置）

2.如果坏字符出现在模式串中，则将模式串最靠近好后缀的坏字符（当然这个实现就有点繁琐）与母串的坏字符对齐：

![BM-3](http://dl.iteye.com/upload/attachment/0075/1869/be2834b9-8a5c-32e1-86fe-6d78cb58bf66.png)

（注：如果模式串P是babababab，则是将第二个b与母串的b对齐）


**好后缀规则** 

好后缀规则分三种情况

1.模式串中有子串匹配上好后缀，此时移动模式串，让该子串和好后缀对齐即可，如果超过一个子串匹配上好后缀，则选择最靠靠近好后缀的子串对齐。

![BM-4](http://dl.iteye.com/upload/attachment/0075/1889/db8a0df0-a383-32bb-901c-b76185a7b0ff.png)

 2.模式串中没有子串匹配上后后缀，此时需要寻找模式串的一个最长前缀，并让该前缀等于好后缀的后缀，寻找到该前缀后，让该前缀和好后缀对齐即可。

![BM-5](http://dl.iteye.com/upload/attachment/0075/1909/89dbe34d-e78a-31d3-bcf2-82cf281eb641.png)


其实，1和2都可以看成模式串还含有好后缀串（好后缀子串也是好后缀）。

3.模式串中没有子串匹配上后后缀，并且在模式串中找不到最长前缀，让该前缀等于好后缀的后缀。此时，直接移动模式到好后缀的下一个字符。

![BM-6](http://dl.iteye.com/upload/attachment/0075/1891/d4ba8554-a7cb-332f-80de-fd317d4f078c.png)



**推荐阅读**

[字符串模式匹配算法](http://dsqiu.iteye.com/blog/1700312)

### Sunday算法

Sunday算法思想跟BM算法很相似，在匹配失败时关注的是文本串中参加匹配的最末位字符的下一位字符。如果该字符没有在匹配串中出现则直接跳过，即移动步长= 匹配串长度+1；否则，同BM算法一样其移动步长=匹配串中最右端的该字符到末尾的距离+1。

**推荐阅读：**

[Sunday算法详解](http://blog.csdn.net/laojiu_/article/details/50767615) 

### Shift-And/Shift-Or算法

**主要思想** 

Shift-And算法和KMP算法一样，是基于前缀来进行字符串匹配。但是它的算法思想要比KMP思路简单很多。它主要是维护了一个集合，该集合中保存的是所有既是模式串的前缀同时又是目标串的后缀的字符串。每次读入一个新的文本，本算法就利用位并行的方法更新该集合（神奇之处）。该集合用一个位掩码D进行表示：$d[m...1]$表示（m表示的是模式串的大小）

**算法介绍** 

D的第j位为1的时候（此时成$D[j]$是活动的），表示模式串前缀的$p[1...j]$同时也是目标串后缀$t[1...i]$.而当$d[m]$是1的时候，表示有一个匹配成功。当读入目标串的下一个字符$t[i+1]$的时候，需要对D进行更新为D’当且仅当D的第j位是活动的，并且$t[i+1]$和$p[j+1]$相同的时候，此时可以利用位并行的方法在常数时间内对D进行更新。

**构建辅助表B** 

集合B记录模式串中每一个字符位掩码$b[m…1]$。如果$p[j]=x$,则$B[x]$的第j为设为1.否则为0.

举例1：模式串announce共8位

![Shift-And](http://dl.iteye.com/upload/attachment/0083/0121/2d303eeb-4d51-382d-bfbc-34bbc9dae8dc.png)

![Shift-And1](http://dl.iteye.com/upload/attachment/0083/0123/aac9c1ef-507b-3656-b63c-615249472659.png)

同理可以得到B（其中模式串中不包含的字符*设为00000000）

![Shift-And2](http://dl.iteye.com/upload/attachment/0083/0152/bbb1dcea-499b-32e6-bf04-f76dc9d0e53b.png)

**容器创建和更新** 

对于容器D，初始化为00000000（$0^{m}$ :前m位全为0）.表示当前还没有即使模式串前缀又是目标串后缀。

当读入一个新的目标串字符$t[i+1]$，可以以如下公式进行更新。

![Shift-And3](http://dl.iteye.com/upload/attachment/0083/0126/87572151-567f-3800-8c1d-15665f196d8d.png)

**展示过程**

下面是整个过程：目标串是’annual_announce’ 模式串announce

![Shift-And4](http://dl.iteye.com/upload/attachment/0083/0128/d3dcced8-ba6b-3f6c-8399-57b3e91816fc.png)

**推荐阅读：** 

[令人惊叹的Shift-And/Shift-Or](http://www.iteye.com/topic/1130001) 

**算法模板**

```c++
#include <iostream>
#include <string>
#include <vector>
using namespace std;

void preprocess(unsigned int B[], string T, int n)
{
    unsigned int shift=1;
    for (int i=0; i<n; i++) {
        B[T[i]] |= shift;
        shift <<= 1;
    }
}

vector<int> and_match(string S, string T)
{ //求T在S中出现的位置
    int m=S.length(), n=T.length();
    unsigned int B[256], D=0, mask;
    for (int i=0; i<256; i++)
        B[i] = 0;
    preprocess(B, T, n);
    vector<int> res;

    mask  = 1 << (n - 1);
    for (int i=0; i<m; i++) {
        D = (D << 1 | 1) & B[S[i]];
        if (D & mask)
            res.push_back(i-n+1);
    }

    return res;
}

int main()
{
    string S, T;
    cin >> S >> T;
    vector<int> res=and_match(S,T);
    for (vector<int>::iterator it=res.begin(); it!=res.end(); ++it)
        cout << *it << endl;
    return 0;
}
```

```c++
#include <iostream>
#include <string>
#include <vector>
using namespace std;

void preprocess(unsigned int B[], string T, int n)
{
    unsigned int shift=1;
    for (int i=0; i<n; i++) {
        B[T[i]] &= ~shift; // right bit is set to "0"
        shift <<= 1;
    }
}

vector<int> or_match(string S, string T)
{ //求T在S中出现的位置
    int m=S.length(), n=T.length();
    unsigned int B[256], D=~0, mask;
    for (int i=0; i<256; i++)
        B[i] = ~0; // every bit is set to "1"
    preprocess(B, T, n);
    vector<int> res;

    mask  = ~(1 << (n - 1));
    for (int i=0; i<m; i++) {
        D = (D << 1) | B[S[i]];
        if (~(D | mask))
            res.push_back(i-n+1);
    }

    return res;
}

int main()
{
    string S, T;
    cin >> S >> T;
    vector<int> res=or_match(S,T);
    for (vector<int>::iterator it=res.begin(); it!=res.end(); ++it)
        cout << *it << endl;
    return 0;
}
```



### KMP算法(Knuth-Morris-Pratt)

**问题模型：**

给定一个主字符串（以$S$代替）和模式串（以$P$代替），要求找出$P$在$S$中出现的位置，即串的模式匹配问题。今天来介绍解决这一问题的常用算法之一，$Knuth-Morris-Pratt$算法（简称$KMP$）。该算法还能求最小循环节等问题。并且是后续学习自动机算法的基铺垫。

两个概念：**真前缀**和**真后缀**。

![KMP-1](https://61mon.com/images/illustrations/KMP/1.png)

由上图所得， **"真前缀"** 指除了自身以外，一个字符串的全部头部组合；**"真后缀"** 指除了自身以外，一个字符串的全部尾部组合。

**算法流程：**

1. 首先，主串 "$BBC ABCDAB ABCDABCDABDE$" 的第一个字符与模式串 "$ABCDABD$" 的第一个字符，进行比较。因为$B$与$A$不匹配，所以模式串后移一位。

   ![KMP-2](https://61mon.com/images/illustrations/KMP/2.png)

2. 因为$B$与$A$又不匹配，模式串再往后移。

   ![KMP-3](https://61mon.com/images/illustrations/KMP/3.png)

3. 就这样，直到主串有一个字符，与模式串的第一个字符相同为止。

   ![KMP-4](https://61mon.com/images/illustrations/KMP/4.png)

4. 接着比较主串和模式串的下一个字符，还是相同。

   ![KMP-5](https://61mon.com/images/illustrations/KMP/5.png)

5. 直到主串有一个字符，与模式串对应的字符不相同为止。

   ![KMP-6](https://61mon.com/images/illustrations/KMP/6.png)

6. 这时，最自然的反应是，将模式串整个后移一位，再从头逐个比较。这样做虽然可行，但是效率很差，因为你要把 "搜索位置" 移到已经比较过的位置，重比一遍。

   ![KMP-7](https://61mon.com/images/illustrations/KMP/7.png)

7. 一个基本事实是，当空格与 D 不匹配时，你其实知道前面六个字符是 "ABCDAB"。KMP 算法的想法是，设法利用这个已知信息，不要把 "搜索位置" 移回已经比较过的位置，而是继续把它向后移，这样就提高了效率。

   ![KMP-8](https://61mon.com/images/illustrations/KMP/8.png)

8. ​

   | $i$       | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    |
   | --------- | ---- | ---- | ---- | ---- | ---- | ---- | :--- | ---- |
   | 模式串       | A    | B    | C    | D    | A    | B    | D    | '\0' |
   | $next[i]$ | -1   | 0    | 0    | 0    | 0    | 1    | 2    | 0    |

   怎么做到这一点呢？可以针对模式串，设置一个跳转数组$int next[]$，这个数组是怎么计算出来的，后面再介绍，这里只要会用就可以了。

9. 已知空格与 D 不匹配时，前面六个字符 "$ABCDAB$" 是匹配的。根据跳转数组可知，不匹配处$D$的$next$值为 2，因此接下来从模式串下标为 2 的位置开始匹配。

   ![KMP-9](https://61mon.com/images/illustrations/KMP/9.png)

10. 因为空格与$Ｃ$不匹配，$C$ 处的$next$值为0，因此接下来模式串从下标为 0 处开始匹配。

  ![KMP-10](https://61mon.com/images/illustrations/KMP/10.png)

11. 因为空格与$A$不匹配，此处$next$值为 -1，表示模式串的第一个字符就不匹配，那么直接往后移一位。

    ![KMP-11](https://61mon.com/images/illustrations/KMP/11.png)

12. 逐位比较，直到发现 C 与 D 不匹配。于是，下一步从下标为 2 的地方开始匹配。

    ![KMP-12](https://61mon.com/images/illustrations/KMP/12.png)

13. 逐位比较，直到模式串的最后一位，发现完全匹配，于是搜索完成。

    ![KMP-13](https://61mon.com/images/illustrations/KMP/13.png)

**next数组是如何求出来的**

next 数组的求解基于 **“真前缀”** 和**“真后缀”**，即$next[i]$等于$P[0...(i - 1)]$最长的相同真前后缀的长度（请暂时忽视 i 等于 0 时的情况，下面会有解释）。我们依旧以上述的表格为例。

| $i$       | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    |
| --------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 模式串       | A    | B    | C    | D    | A    | B    | D    | ‘\0’ |
| $next[i]$ | -1   | 0    | 0    | 0    | 0    | 1    | 2    | 0    |

（1）：i = 0，对于模式串的首字符，我们统一为$next[0]$ = -1；
（2）：i = 1，前面的字符串为$A$，其最长相同真前后缀长度为 0，即$next[1]$ = 0；
（3）：i = 2，前面的字符串为$AB$，其最长相同真前后缀长度为 0，即$next[2]$ = 0；
（4）：i = 3，前面的字符串为$ABC$，其最长相同真前后缀长度为 0，即$next[3]$ = 0；
（5）：i = 4，前面的字符串为$ABCD$，其最长相同真前后缀长度为 0，即$next[4]$ = 0；
（6）：i = 5，前面的字符串为$ABCDA$，其最长相同真前后缀为$A$，即$next[5]$ = 1；
（7）：i = 6，前面的字符串为$ABCDAB$，其最长相同真前后缀为$AB$，即$next[6]$ = 2；
（8）：i = 7，前面的字符串为$ABCDABD$，其最长相同真前后缀长度为 0，即$next[7]$ = 0。

**推荐阅读：**

[KMP算法(1)](https://www.61mon.com/index.php/archives/183/)

[KMP算法(2)](https://61mon.com/index.php/archives/192/)

**模板代码**

```c++
namespace KMP {
	int Next[MAX];
	void GetNext(char *str) {
		Next[0] = -1;
		for (int i = 0, j = -1, len = strlen(str); i < len; ) {
			if (j == -1 || str[i] == str[j]) { Next[++i] = ++j; }
			else { j = Next[j]; }
		}
	}
};
```

**经典例题**

POJ -3461——[Oulipo](http://poj.org/problem?id=3461)

题意：给定2个字符p和t。问p在t中出现几次

分析：求出p的next数组，然后与t匹配。

代码：

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <functional>
using namespace std;
typedef long long int LL;
const int MAX = 1e6+24; //字符串长度
namespace KMP {
	int Next[MAX];
	void Init(char *str) {
		Next[0] = -1;
		for (int i = 0, j = -1, len = strlen(str); i < len; ) {
			if (j == -1 || str[i] == str[j]) { Next[++i] = ++j; }
			else { j = Next[j]; }
		}
	}
	int kmp(char *p, char *t) {
		Init(p);
		int result = 0;
		for (int i = 0, j = 0, lenp = strlen(p), lent = strlen(t); i < lent;) {
			if (j == -1 || p[j] == t[i]) { j++; i++; }
			else { j = Next[j]; }
			if (j == lenp) { result++; j = Next[j]; }
		}
		return result;
	}
};
char P[MAX], T[MAX];
int main() {
	int t;
	scanf("%d", &t);
	while (t--) {
		scanf("%s%s", P,T);
		printf("%d\n",KMP::kmp(P,T));
	}
	return 0;
}
```

### Extend-KMP算法

**问题模型** 

给定两个字符串S和T（长度分别为n和m），下标从0开始，定义$extend[i]$等于$S[i...(n-1)] (Suff(i))$与T的最长公共前缀的长度，求出所有的$extend[i]$。举个例子，看下表：

| i           | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    |
| ----------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| S           | a    | a    | a    | a    | a    | b    | b    | b    |
| $extend[i]$ | 5    | 4    | 3    | 2    | 1    | 0    | 0    | 0    |
| T           | a    | a    | a    | a    | a    | c    |      |      |

显然，如果在S的某个位置i有extend[i]等于m，则可知在S中找到了匹配串T，并且匹配的首位置是i。而且，扩展KMP算法可以找到S中所有T的匹配。

**算法流程**

![Extend-KMP1](https://segmentfault.com/img/remote/1460000008663860?w=760&h=310)

如上图，假设当前遍历到S串位置i，即$extend[0...(i-1)]$这i个位置的值已经计算得到。算法在遍历过程中记录了匹配成功的字符的最远位置p，及这次匹配的起始位置a。相较于字符串T得出，$S[a...p]$等于$T[0...(p-a)]$。

再定义一个辅助数组$int next[]$，其中$next[i]$含义为：$T[i...(m-1)]$与T的最长公共前缀长度，m为串T的长度。

![Extend-KMP2](https://segmentfault.com/img/remote/1460000008663861?w=760&h=310)

椭圆的长度为$next[i-a]$，对比S和T，很容易发现，三个椭圆完全相同。如上图，此时$i+next[i-a]<p$，根据$next$数组的定义，此时$extend[i]=next[i-a]$。

![Extend-KMP](https://segmentfault.com/img/remote/1460000008663862?w=760&h=310)

如果$i+next[i-a]>=p$呢？仔细观察上图，很容易发现$i+next[i-a]$是不可能大于p的，如果可以大于p，那么以a为起始位置的最远匹配位置就不是p了，而是到了红线位置。因此$i+next[i-a]$只可以小于等于p，小于的情况已经讨论过了，下面讨论下等于的情况，见下图：

![Extend-KMP3](https://segmentfault.com/img/remote/1460000008663863?w=760&h=310)

三个椭圆都是完全相同的，此时我们可以直接从$S[p]与T[next[i-a]-1]$开始往后匹配，加快了速度。

（4）最后，就是求解$next$数组。我们再来看下$next$与$extend$的定义：
$next[i]$： $T[i...(m-1)]$与T的最长公共前缀长度；
$extend[i]$： $S[i...(n-1)]$与T的最长公共前缀的长度。
恍然大悟，求解$next$的过程不就是T和自己的一个匹配过程嘛。

**推荐阅读：**

[扩展KMP算法](https://segmentfault.com/a/1190000008663857) 

**模板代码**

```c++
namespace ExtendKMP {
	int Next[MAX],ExNext[MAX];
	void GetNext(char *str) {
		int lenstr = strlen(str);
		Next[0] = lenstr;
		for (int i = 1, j = -1, a, p; i < lenstr; i++, j--) {
			if (j < 0 || i + Next[i - a] >= p) {
				if (j < 0) p = i, j = 0;
				while (p < lenstr&&str[p] == str[j])  p++, j++;
				Next[i] = j;
				a = i;
			}
			else
				Next[i] = Next[i - a];
		}
	}
	void GetExtendNext(char *t, char *s) {
		GetNext(t);
		for (int i = 0, j = -1, lent = strlen(t), lens = strlen(s), p, a; i < lens; i++, j--) {
			if (j < 0 || i + Next[i - a] >= p) {                                  
				if (j < 0) p = i, j = 0; 
				while (p < lens&&j < lent&&s[p] == t[j]) p++, j++;
				ExNext[i] = j;
				a = i;
			}
			else
				ExNext[i] = Next[i - a];
		}
	}
};
```

**经典问题**

POJ-3461——[Oulipo](http://poj.org/problem?id=3461)

题意：给定2个字符p和t。问p在t中出现几次

分析：exnext[i]表示t[i...|t|-1]与p[0...|p|-1]的最长公共前缀，如果公共前缀长度>=|p|，说明t[i...i+|p|-1]==p[0...|p|-1]。

代码：

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <functional>
using namespace std;
typedef long long int LL;
const int MAX = 1e6+24; //字符串长度
namespace ExtendKMP {
	int Next[MAX],ExNext[MAX];
	void GetNext(char *str) {
		int lenstr = strlen(str);
		Next[0] = lenstr;
		for (int i = 1, j = -1, a, p; i < lenstr; i++, j--) {
			if (j < 0 || i + Next[i - a] >= p) {
				if (j < 0) p = i, j = 0;
				while (p < lenstr&&str[p] == str[j])  p++, j++;
				Next[i] = j;
				a = i;
			}
			else
				Next[i] = Next[i - a];
		}
	}
	void GetExtendNext(char *t, char *s) {
		GetNext(t);
		for (int i = 0, j = -1, lent = strlen(t), lens = strlen(s), p, a; i < lens; i++, j--) {
			if (j < 0 || i + Next[i - a] >= p) {                                  
				if (j < 0) p = i, j = 0; 
				while (p < lens&&j < lent&&s[p] == t[j]) p++, j++;
				ExNext[i] = j;
				a = i;
			}
			else
				ExNext[i] = Next[i - a];
		}
	}
	int Solve(char *p, char *t) {
		GetExtendNext(p, t);
		int result = 0;
		for (int i = 0, lenp = strlen(p), lent = strlen(t); i < lent; i++) {
			if (ExNext[i] >= lenp) { 
				result++;
			}
		}
		return result;
	}
};
char P[MAX], T[MAX];
int main() {
	int t;
	scanf("%d", &t);
	while (t--) {
		scanf("%s%s", P,T);
		printf("%d\n", ExtendKMP::Solve(P,T));
	}
	return 0;
}
```

### 字典树算法 (Trie)

**问题模型**



**算法模板**

```c++
namespace Trie {
	const int SIZE = 256;
	int child[MAX][SIZE], value[MAX*SIZE], L, root;
	int newNode() {
		memset(child[L], -1, sizeof(child[L]));
		value[L] = 0;
		return L++;
	}
	void Init() {
		root = newNode();
	}
	void Insert(char *str) {
		int now = root;
		for (int i = 0, len = strlen(str); i < len; i++) {
			if (child[now][str[i]] == -1) {
				child[now][str[i]] = newNode();
			}
			now = child[now][str[i]];
		}
		value[now]++;
	}
	int Search(char *str) {
		int now = root;
		for (int i = 0, len = strlen(str); i < len; i++) {
			if (child[now][str[i]] == -1) {
				return 0;
			}
			now = child[now][str[i]];
		}
		return value[now];
	}
};
```

**经典例题**



### AC自动机算法 (Aho-Corasick automaton)



## 回文串问题

“回文串”是一个正读和反读都一样的字符串

### 马拉车算法 (Manacher)

**问题模型**

给定一个字符串，求出其最长回文子串。

**算法过程** 

由于回文分为偶回文（比如 $bccb$）和奇回文（比如 $bcacb$），而在处理奇偶问题上会比较繁琐，所以这里我们使用一个技巧，具体做法是，在字符串首尾，及字符间各插入一个字符（前提这个字符未出现在串里）。

举个例子：$s="abbahopxpo"$，转换为$s_{new}="\$\#a\#b\#b\#a\#h\#o\#p\#x\#p\#o\#"$（这里的字符 $\$$ 只是为了防止越界，下面代码会有说明），如此，s 里起初有一个偶回文$abba$和一个奇回文$opxpo$，被转换为$\#a\#b\#b\#a\#$和$\#o\#p\#x\#p\#o\#$，长度都转换成了奇数。

定义一个辅助数组$int p[]$，其中$p[i]$表示以 $i$ 为中心的最长回文的半径，例如：

| i         | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   | 11   | 12   | 13   | 14   | 15   | 16   | 17   | 18   | 19   |
| --------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| $s_{new}$ | \$   | \#   | a    | \#   | b    | \#   | b    | \#   | a    | \#   | h    | \#   | o    | \#   | p    | \#   | x    | \#   | p    | \#   |
| $p$       |      | 1    | 2    | 1    | 2    | 5    | 2    | 1    | 2    | 1    | 2    | 1    | 2    | 1    | 2    | 1    | 4    | 1    | 2    | 1    |

可以看出，$p[i] - 1$正好是原字符串中最长回文串的长度。

接下来的重点就是求解 $p$ 数组，如下图：

![Manacher](https://61mon.com/images/illustrations/Manacher/1.png)

设置两个变量，$mx$ 和 $id$ 。$mx$ 代表以 $id$ 为中心的最长回文的右边界，也就是$mx = id + p[id]$。

假设我们现在求$p[i]$，也就是以$i$ 为中心的最长回文半径，如果$i < mx$，如上图，那么：

```C++
if (i < mx)  
    p[i] = min(p[2 * id - i], mx - i); 
```

$2*id-i$为$i$关于$id$的对称点，即上图的$j$点，而$p[j]$表示以j为中心的最长回文半径，因此我们可以利用$p[j]$来加快查找

当 $mx - i > p[j]$ 的时候，以$S[j]$为中心的回文子串包含在以$s[id]$为中心的回文子串中，由于 $i$ 和 $j$ 对称，以$s[i]$为中心的回文子串必然包含在以$s[id]$为中心的回文子串中，所以必有 $p[i] = p[j]$，见下图。

![Manacher1](http://pic002.cnblogs.com/images/2012/426620/2012100415402843.png)

当 $p[j] > mx - i$ 的时候，以$s[j]$为中心的回文子串不完全包含于以$s[id]$为中心的回文子串中，但是基于对称性可知，下图中两个绿框所包围的部分是相同的，也就是说以$s[i]$为中心的回文子串，其向右至少会扩张到$mx$的位置，也就是说 $p[i] >= mx - i$。至于$mx$之后的部分是否对称，就只能一个一个匹配了。

![Manacher2](http://pic002.cnblogs.com/images/2012/426620/2012100415431789.png)

对于 $mx <= i$ 的情况，无法对 $p[i]$做更多的假设，只能$p[i] = 1$，然后再去匹配了

**线性复杂的解析**

![Manacher4](http://pic002.cnblogs.com/images/2012/426620/2012100415453056.png)

**推荐阅读：**

[Manacher算法](https://www.61mon.com/index.php/archives/181/)

**算法模板**

```c++
namespace Manacher {
	char dstr[MAX * 3];
	int p[MAX * 3],lendstr;
	void init(char *str) {
		dstr[0] = '$'; dstr[1] = '#';
		int lenstr = strlen(str);
		for (int i = 0; i<lenstr; i++) {
			dstr[i * 2 + 2] = str[i]; dstr[i * 2 + 3] = '#';
		}
		lendstr = lenstr * 2 + 2;
		dstr[lendstr] = '*';
	}
	int manacher() {
		memset(p, 0, sizeof(p));
		int id = 0, mx = 0;
		for (int i = 1; i<lendstr; i++) {
			if (mx>i) {
				p[i] = min(p[2 * id - i], mx - i);
			}
			else {
				p[i] = 1;
			}
			while (dstr[i - p[i]] == dstr[i + p[i]]) {
				p[i]++;
			}
			if (p[i] + i>mx) {
				mx = p[i] + i;
				id = i;
			}
		}
		int result = 0;
		for (int i = 0; i<lendstr; i++) {
			result = max(result, p[i]);
		}
		return result - 1;
	}
};
```

**经典问题**

HDU-3068——[最长回文](http://acm.hdu.edu.cn/showproblem.php?pid=3068)

题意：给定一个字符串，求该字符串的最长回文子串长度。

分析：模板题

代码：

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <functional>
#include <algorithm>
#include <cmath>
#include <vector>
#include <string>
using namespace std;
typedef long long int LL;
const int MAX = 110000 +24; //字符串长度
namespace Manacher {
	char dstr[MAX * 3];
	int p[MAX * 3],lendstr;
	void init(char *str) {
		dstr[0] = '$'; dstr[1] = '#';
		int lenstr = strlen(str);
		for (int i = 0; i<lenstr; i++) {
			dstr[i * 2 + 2] = str[i]; dstr[i * 2 + 3] = '#';
		}
		lendstr = lenstr * 2 + 2;
		dstr[lendstr] = '*';
	}
	int manacher() {
		memset(p, 0, sizeof(p));
		int id = 0, mx = 0;
		for (int i = 1; i<lendstr; i++) {
			if (mx>i) {
				p[i] = min(p[2 * id - i], mx - i);
			}
			else {
				p[i] = 1;
			}
			while (dstr[i - p[i]] == dstr[i + p[i]]) {
				p[i]++;
			}
			if (p[i] + i>mx) {
				mx = p[i] + i;
				id = i;
			}
		}
		int result = 0;
		for (int i = 0; i<lendstr; i++) {
			result = max(result, p[i]);
		}
		return result - 1;
	}
};
char str[MAX];
int main() {
	while (~scanf("%s", str)) {
		Manacher::init(str);
		printf("%d\n",Manacher::manacher());
	}
	return 0;
}
```



### 回文树(Palindromic Tree)

**前言：**

回文树是由Mikhail Rubinchik大神发明的，在Petrozavodsk Summer Camp 2014上首次提出来，是一个很新的数据结构。正如其名，就是用来解决回文相关的题目的。应该说，是manacher的一个特殊化，所以他跟manacher有很多相似之处。

**问题模型：**

一切关于回文相关的题目

**结点结构：**

就像线段树、平衡树等其它树结构一样，回文树由若干个节点组成，每个节点代表一个回文串(palindrome)。

《结点》

![Ptree-1](http://img.blog.csdn.net/20150925215913929)

《边》

节点之间通过有向边连接起来，回文树中有两种类型的边，第一种类型的边上同时有字符做标记，比如：$u$和$v$通过带字符$X$的边连接起来，表示节点$u$所表示的回文串两边添加$X$字符可以得到$v$节点所表示的回文串。以下是一个例子： 

![Ptree-2](http://img.blog.csdn.net/20150925220527427)

回文树中另一种类型的边是后缀链接边(suffix link)。从$u$到$w$存在一条后缀链接边，当且仅当$w$节点所代表的回文串是$u$的后缀中最长的回文串，但$w$和$u$不能相同（后缀：包含最后一个字符的子串，$bcd$是$abcd$的后缀，但$bc$不是$abcd$的后缀）。

下面是一个例子：(**后缀链接：虚线表示从aba到a的后缀链接边，因为a是aba最长的后缀回文串**)

![Ptree-3](http://img.blog.csdn.net/20150925221459116)

“回文树”这个名字可能会让人产生疑惑，因为回文树这个数据结构并不是一棵普通的树，它有两个根，一个根表示长度为-1的回文串(奇长度回文)，是我们为了方便操作加进去的，长度为1的回文串可以通过它左右两侧各添加一个字符得到。另一个根表示长度为0的回文串(偶长度回文)，即空串。

注意，我们并不在每个节点中实际存储它所表示的回文串，否则很容易爆内存，节点中仅仅包含如下信息：1.回文串长度；2.通过所有字符连接的边（即第一种类型的边）；3.后缀链接边（即第二种类型的边）。还有其它根据实际问题需要添加的边。

**算法过程：**

对于一个给定的字符串$s$，它所对应的回文树就包含了$s$所有的回文子串，由于一个长度为$n$的字符串最多只有$n$个本质不同的回文子串（可以尝试自己证明这个结论，并不难，提示：考虑新加一个字符最多会贡献多少个新的回文子串），因此回文树的节点数目不会超过字符串的长度+2，另外两个是前面提到的两个虚拟的根。

从空串开始，每次添加一个字符，并更新回文树。假设我们已经处理了字符串的某个**前缀$p$**，接下来要**添加**的字符是$x$。

![Ptree-5](http://img.blog.csdn.net/20150925223352213)

同时需要维护**前缀$p$**的**最长后缀回文串**，不妨设为$t$。 

![Ptree-6](http://img.blog.csdn.net/20150925224559202)

由于$t$已经处于某个**已经处理的前缀**中，因此它必定对应于回文树上的某个节点，这个节点会有后缀链接边指向其他节点，然后这个节点再指向其他节点，形成一个链。下面是的图示： 

![Ptree-7](http://img.blog.csdn.net/20150925224628627)

现在我们来找**新前缀$p+x$的后缀回文串**，这个回文串肯定是$xAx$的形式，其中$A$是某个回文串（注意A可能为空，或者对应于长度为-1的根，此时的后缀回文串就是$x$这一个字符啦）。同时注意到，$A$是$p$的后缀，因此一定可以从$t$出发通过后缀链接边到达$A$所对应的节点。

![Ptree-8](http://img.blog.csdn.net/20150925224645466)

字符串$xAx$是唯一一个有可能在$p+x$中出现却从来没有在**前缀$p$**中出现的回文串。原因也很简单，因为所有可能的新回文串都必须以$x$为结尾，因此必定是$p+x$的后缀回文串。由于$xAx$是$p+x$的最长后缀回文串，因此其它更短的回文串必定是在$xAx$的前缀中出现了，也就是在前缀$p$中出现过。证毕。

所以，为了处理这个新添加的字符$x$，我们需要沿着后缀链接边走，直到找到一个合适的A(也有可能一直回溯到根)。然后我们检查与$A$相对应的节点是否与一条标记为$x$的边，如果没有的话，就添加一条边指向新的节点$xAx$。(有的话就什么都不用做了。。)

接下来还需要更新$xAx$的后缀链接边，如果后缀链接边已经存在，那就不需要做任何事情了。否则，我们就找到$xAx$的最长后缀回文串，必定是有$xBx$的形式，其中$B$有可能是空串。按照前面的逻辑，$B$是前缀$p$的后缀回文串并且从$t$通过边可达。

![Ptree-9](http://img.blog.csdn.net/20150925224721709)

总结一下回文树的构造过程。从左到右一个字符一个字符地处理，始终维护着当前已处理前缀的最长后缀回文串(初始时为空串)。每次扫描一个新的字符$x​$是，我们就沿着后缀链接边找到一个回文串$A​$，它的两边可以同时添加字符$x，得​$到一个合法的后缀回文串。$xAx​$是新节点的唯一候选，为了得到它的后缀链接边，我们需要继续沿着链接走，直到找到另一个回文串$B​$，它的两边添加字符$x​$可以得到$xAx​$的合法后缀回文串，于是添加一条从$xAx​$到$xBx​$的边（当然，如果这条边已经存在就不用了）。

**推荐阅读：**

Victor Wonder《Palindromic tree.》(参见附件)

2017国家集训队论文- 翁文涛《回文树及其应用》(参见附件)

[Palindromic Tree——回文树【处理一类回文串问题的强力工具】](http://blog.csdn.net/u013368721/article/details/42100363)

**模板代码**

```c++
const int MAXN = 100005 ;
const int N = 26 ;

struct Palindromic_Tree {
	int next[MAXN][N] ;//next指针，next指针和字典树类似，指向的串为当前串两端加上同一个字符构成
	int fail[MAXN] ;//fail指针，失配后跳转到fail指针指向的节点
	int cnt[MAXN] ;
	int num[MAXN] ;
	int len[MAXN] ;//len[i]表示节点i表示的回文串的长度
	int S[MAXN] ;//存放添加的字符
	int last ;//指向上一个字符所在的节点，方便下一次add
	int n ;//字符数组指针
	int p ;//节点指针

	int newnode ( int l ) {//新建节点
		for ( int i = 0 ; i < N ; ++ i ) next[p][i] = 0 ;
		cnt[p] = 0 ;
		num[p] = 0 ;
		len[p] = l ;
		return p ++ ;
	}

	void init () {//初始化
		p = 0 ;
		newnode (  0 ) ;
		newnode ( -1 ) ;
		last = 0 ;
		n = 0 ;
		S[n] = -1 ;//开头放一个字符集中没有的字符，减少特判
		fail[0] = 1 ;
	}

	int get_fail ( int x ) {//和KMP一样，失配后找一个尽量最长的
		while ( S[n - len[x] - 1] != S[n] ) x = fail[x] ;
		return x ;
	}

	void add ( int c ) {
		c -= 'a' ;
		S[++ n] = c ;
		int cur = get_fail ( last ) ;//通过上一个回文串找这个回文串的匹配位置
		if ( !next[cur][c] ) {//如果这个回文串没有出现过，说明出现了一个新的本质不同的回文串
			int now = newnode ( len[cur] + 2 ) ;//新建节点
			fail[now] = next[get_fail ( fail[cur] )][c] ;//和AC自动机一样建立fail指针，以便失配后跳转
			next[cur][c] = now ;
			num[now] = num[fail[now]] + 1 ;
		}
		last = next[cur][c] ;
		cnt[last] ++ ;
	}

	void count () {
		for ( int i = p - 1 ; i >= 0 ; -- i ) cnt[fail[i]] += cnt[i] ;
		//父亲累加儿子的cnt，因为如果fail[v]=u，则u一定是v的子回文串！
	}
} ;
```

## 后缀利器 
### 后缀数组(Suffix Array)
### 后缀树(Suffix Tree)
### 后缀自动机(Suffix Automaton)
### 后缀平衡树(Suffix Balanced Tree)

### 后缀仙人掌(Suffix Cactus)



## 其他问题
### 最小/大表示法(Minimum Representation)

**前言：**

“最小表示法”比起动态规划、贪心等思想，在当今竞赛中似乎并不是很常见。但是在解决判断**“同构”**一类问题中却起着重要的作用。

**问题模型：**

有两条环状的项链，每条项链上各有N个多种颜色的珍珠，相同颜色的珍珠，被视为相同。由于项链是环状的，因此循环以后的项链被视为相同的,判断两条项链是否相同。如下图

![最小表示法-1](Image\最小表示法-1.PNG)

**算法流程：**

《定义》

设有事物集合$T=\left \{ t1,t2,...,tn \right \}$，映射集合$F= \left \{ f1，f2，…，fm \right \}$。任意$f\subseteq F$均为$T$到$T$的映射，$f[i]=T\rightarrow T$,如果两个事物$s,t \subseteq T$，有一系列F的映射的连接$f_{i1}•f_{i2}•…•f_{ik}(s)=t$，则说$s$和$t$是$F$本质相同的。

《过程》

设函数$M(s)$返回值意义为：从$s$的第$M(s)$个字符引起的$s$的一个循环表示是$s$的最小表示。若有多个值，则返回最小的一个。

![最小表示法-2](Image\最小表示法-2.png)

1. 设$u=s1+s1$，$w=s2+s2$并设指针$i$,$j$指向$u$,$w$第一个字符

   ![最小表示法-3](Image\最小表示法-3.png)

   ​

2. 如果$s1$和$s2$是循环同构的，那么当$i$,$j$分别指向$M(s1)$,$M(s2)$时，一定可以得到$u[i→i+|s1|-1]=w[j→j+|s2|-1]$，迅速输出正确解。

   ![最小表示法-4](Image\最小表示法-4.PNG)

   ​

3. 同样$s1$和$s2$循环同构时，当$i$,$j$分别满足$i≤M(s1),j≤M(s2)$时，两指针仍有机会达到$i=M(s1)$,$j=M(s2)$这个状态。

   问题转化成，两指针分别向后滑动比较，如果比较失败，如何正确的滑动指针，新指针$i^{`}$,$j^{`}$仍然满足
   $i’≤M(s1),j’≤M(s2)$。

   ​

4. 设指针$i$,$j$分别向后滑动$k$个位置后比较失败($k≥0$)，即有$u[i+k] \neq w[j+k]$。

   设$u[i+k] \gt w[j+k]$，同理可以讨论$u[i+k] \lt w[j+k]$的情况。

   当$i \leqslant x\leqslant i+k$时，我们来研究$s1^{(x-1)}$。

   ![最小表示法-5](Image\最小表示法-5.PNG)

   ​

5. 因为$u[x]$在$u[i]$后($x-i$)个位置，对应的可以找到在$w[j]$后($x-i$)个位置的$w[j+(x-i)]$，同样对应的有$u[x+1]$和$w[j+(x+1-i)]$，$u[x+2]$和$w[j+(x+2)-i]$，直到$u[i+k-1]$和$w[j+k-1]$。它们都是相等的，即有$u[x→i+k-1]=w[j+(x-i)→j+k-1]$。

   ![最小表示法-6](Image\最小表示法-6.PNG)

   ​

6. 很容易就得到$u[x→i+k] \gt w[j+(x-i)→j+k]$。所以$s1^{(x-1)}$不可能是$s1$的最小表示！因此$M(s1) \gt i+k$，

   指针$i​$滑到$u[i+k+1]​$处仍可以保证**小于等于**$M(s1)​$！

   ![最小表示法-7](.\Image\最小表示法-7.PNG)

   ​

7. 同理，当$u[i+k]<w[j+k]$的时候，可以将指针$j$滑到$w[j+k+1]$处！也就是说，两指针向后滑动比较失败以后，

   指向较大字符的指针向后滑动$k+1$个位置。

8. 同理，可以推出最大表示法的过程。

**推荐阅读：**

IOI2003冬令营演示文稿-周源《浅析“最小表示法”思想》(参见附件)

[字符串最小表示法O(n)算法](http://blog.csdn.net/zy691357966/article/details/39854359#comments)

**模板代码**

```c++
namespace MinRepresentation {
	int MinPosition(char *str) { //返回最小表示法的起点下标
		int i, j, k, len = strlen(str);
		for (i = 0, j = 1, k = 0 ; i < len&&j < len&&k < len; ) {
			int li, lj;
			li = (i + k) >= len ? i + k - len : i + k;
			lj = (j + k) >= len ? j + k - len : j + k;
			if (str[li] == str[lj]) { k++; }
			else {
				if (str[li]>str[lj]) { i = i + k + 1; }
				else { j = j + k + 1; }
				if (i == j) { j++; }
				k = 0;
			}
		}
		return i < j ? i : j;
	}
};
```

**经典例题**

HDU-2609——[How many](http://acm.hdu.edu.cn/showproblem.php?pid=2609)

题意：给定n个字符串，求在循环同构的情况下不同的字符串数量。

分析：模板题

代码：

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <functional>
#include <algorithm>
#include <cmath>
#include <vector>
#include <set>
#include <string>
using namespace std;
typedef long long int LL;
const int MAX = 10000 +24; //字符串长度
namespace MinRepresentation {
	int MinPosition(char *str) {
		int i, j, k, len = strlen(str);
		for (i = 0, j = 1, k = 0 ; i < len&&j < len&&k < len; ) {
			int li, lj;
			li = (i + k) >= len ? i + k - len : i + k;
			lj = (j + k) >= len ? j + k - len : j + k;
			if (str[li] == str[lj]) { k++; }
			else {
				if (str[li]>str[lj]) { i = i + k + 1; }
				else { j = j + k + 1; }
				if (i == j) { j++; }
				k = 0;
			}
		}
		return i < j ? i : j;
	}
};
char str[MAX][105];
set<string>se;
int main() {
	int n;
	while (~scanf("%d", &n)) {
		se.clear();
		for (int i = 0; i < n; i++) {
			scanf("%s", str[i]);
			int start = MinRepresentation::MinPosition(str[i]);
			string minstr;
			for (int j = 0,len=strlen(str[i]); j < len; j++) {
				minstr += str[i][(j + start) % len];
			}
			if (!se.count(minstr)) {
				se.insert(minstr);
			}
		}
		printf("%d\n", se.size());
	}
	return 0;
}
```

### Trie图(DFA)

**前言：**

$DFA$确定性有限状态自动机是一种图结构的数据结构，可以由($Q$, $q0$, $A$, $Sigma$, $Delta$)来描述，其中$Q$为状态集，$q0$为初始状态，$A$为终态集合，$Sigma$为字母表，$Delta$为转移函数。它表示从唯一一个起始状态$q0$开始，经过有限步的$Delta$转移，转移是根据字母表$Sigma$中的元素来进行，最终到达终态集合$A$中的某个状态的状态移动。 

下图所示是一个终态集合为{$"nano"$}的$DFA$。$DFA$只能有一个起点而可以有多个终点。每个节点都有字符集大小数条有向边，并且任一节点，都不会存在相同字符的有向边指向不同的节点。

![DFA-1](http://7xkwr3.com1.z0.glb.clouddn.com/dfa.PNG)

**算法思想：**

1. $Trie$图结构 

       Trie图为以Trie树为基础构造出来的一种DFA。对于插入的每个模式串，其插入过程使用的最后一个节点都作为DFA的一个“终止”节点。 
       如果要求一个母串包含哪些模式串（即该母串的某个子串恰好等于预先给定的某个模式串），以母串作为DFA的输入，
       在DFA上行走，走到“终止”节点就意味着匹配了相应的模式串。

2. 失配处理 
   ​    在行走的过程中，如果出现母串中的下一个字符在$Trie$图中当前位置处没有一个子节点与之对应，或者$Trie$图中的当前位置正好匹配了一个模式串，那么需要调整母串重新匹配的位置。一般，可以调整母串上开始匹配的位置，使之加1，再尝试从$Trie$图的根节点位置开始匹配。这样显然效率很低。可以**参考$KMP$算法的最长相同前后缀的方法，来避免回溯。**

3. **前缀指针** 
   ​    为了避免回溯，参考$KMP$的$next$数组，在$Trie$图中定义“前缀指针”：

   ```
   从根节点到节点P可以得到一个字符串S，节点P的前缀指针定义为 指向树中出现过的S的最长后缀（不能等于S）
   ```

4. **高效的构造前缀指针** 
   ​    根据深度一一求出每一个节点的前缀指针。对于当前节点，设它的父节点与它的边上的字符为$ch$，如果它的父节点的前缀指针所指向的节点的儿子节点中，有通过$ch$字符指向的儿子，那么当前节点的前缀指针指向该儿子节点，否则通过当前节点父节点的前缀指针指向节点的前缀指针，继续向上查找，直到到达根为止。 

   ![DFA-2](http://7xkwr3.com1.z0.glb.clouddn.com/dfa2.PNG)

   ​

5. **危险节点** 
   （1）“终止”节点是危险节点 
   （2）如果一个节点的前缀指针指向危险节点，则该节点为危险节点

6. **在建立好的$Trie$图上遍历** 
   从$root$出发，按照当前串的下一个字符$ch$进行在树上的移动。若当前点$P$不存在通过$ch$连接的儿子，那么将$P$的前缀指针所指向的节点$Q$作为当前节点； 如果还无法找到通过$ch$连接的儿子节点，再考虑$Q$的前缀指针指向的节点（将之作为当前节点）....; 直到找到通过$ch$连接的儿子，再继续遍历； 
   **如果遍历的过程中经过了某个非终止节点的危险节点，则可以断定$S$包含某个模式串，要找出是哪个模式串，沿着危险节点的前缀指针走，碰到终止节点即可。**

   ​

7. **$Trie$图的时间复杂度**

   **在$Trie$图上遍历母串S的时间复杂度为$len(S)$。**

   （1）母串每过掉一个字符，不论该字符是匹配上还是没匹配上，在$trie$图上最多向下走一层

   （2）一个节点的前缀指针总是指向更高层次的节点，所以每次沿着前缀指针走一步，节点的层次就会向上一层

   （3）母串$S$最终被过掉了$len(S)$个字符，所以最多向下走了$len(S)$次。

   （4）最多向下走了$len(S)$次，那么就不可能向上走超过$len(S)$次，因此沿着前缀指针走的次数，做多不超过$len(S)$

   ​

8. **前缀指针思想**

   $Trie$通过前缀指针来避免母串的回溯，其思想和$KMP$算法非常相似。 $KMP$算法是通过确定子串中失配点之前的子串的最长相同前后缀，失配时，母串当前点不回溯，而是直接和最长相同前后缀的前缀处继续进行匹配。 

   ![DFA-3](http://7xkwr3.com1.z0.glb.clouddn.com/kmp-kmp.PNG)

   **（kmp 避免母串指针回溯）**

   和$KMP$类似，$Trie$图中的每个节点都对应一个模板串（节点为终止节点）或者模板串的子串（节点不是终止节点），记为$S$。$S$可以确定$len(S)-1$个后缀（从$S$中的第$2$到第$len(S)-1$个位置到$S$的末尾确定），其中有些后缀串$Si$可能正好对应该$Trie$图中从$root$节点出发的到某个节点$Pi$确定的串。

   ![DFA-4](http://7xkwr3.com1.z0.glb.clouddn.com/dfa3.PNG)

   如上图所示，绿色方块区域为从母串上一个开始匹配点到失配点之前的匹配区域，红色为失配点，该绿色匹配区域中有两个后缀子串$sub1[S1,A]$区域和$sub2[S2,A]$区域，分别对应$Trie$图中从$root$出发到$P1,P2$点确定的串。且母串中$[S1,E1]$和$[S2,E2]$分别对应一个模式串。

   母串不回溯，$Trie$图上当前点的移动，可以匹配母串中存在的所有的模式串

   以上图为例此时，需要确定母串指针之后的移动可以找到$[S1,E1]$和$[S2,E2]$两个模式串，策略是先匹配起点靠前的那个串，即$[S1,E1]$。 

   **case 1 E1 > E2** 

   ![DFA-5](http://7xkwr3.com1.z0.glb.clouddn.com/dfa3.PNG)

   母串指针不回溯，$Trie$图的当前点转移到$P1$（从$root$到$P1$对应$[S1,A]$),然后尝试匹配。匹配成功，到达$E1$点，此时将$Trie$图中点从$E1$移动到$E1$的前缀指针，由于$[S2,E1]$为$[S1,E1]$的最长后缀，即移动到点$P$，使得$root$到$P$为$[S2,E1]$。因为$Trie$图中$[S2,E2]$对应从$root$出发到某个点$Q$的串，那么$root$到$Q$的路径必然经过点$P$,此时从点$P$继续匹配，必然能够到达$Q$； 这样，就得到了$[S1,E1]$和$[S2,E2]$两个模式串。

   **case 2 E1 < E2** 

   ![DFA-7](http://7xkwr3.com1.z0.glb.clouddn.com/dfa4.PNG)

   母串指针不回溯，$Trie$图的当前点转移到$P1$（从$root$到$P1$对应$[S1,A]$),然后尝试匹配。由于$[S1,E1]$对应一个模式串，即对应$Trie$图中的某个终止节点，从$P1$点开始会一直匹配到达$P$（从$root$到$P$对应$[S1,E1]$)。在匹配的过程中，会碰到某个点危险节点$M$，$M$指向节点$Q$（从$root$到$Q$对应$[S2,E2]$）(这是在设置$Trie$图中各个节点的前缀指针的时候确定的），根据$Trie$图的遍历规则，会得到$[S2,E2]$的模式串。这样，就得到了$[S1,E1]$和$[S2,E2]$两个模式串。

**推荐阅读：**

2006年全国信息学冬令营讲座-Maigo《Trie图的构建、活用与改进》(参见附件)

[Trie图](http://www.cnblogs.com/gtarcoder/p/4820560.html)

 [Trie图&AC自动机初学](http://blog.csdn.net/qq_32036091/article/details/51344346)



### Fail树

**前言：**

$Fail$树是$AC$自动机的扩展，在学习$Fail$树前需要具备的前置技能：$KMP$，$Trie$树，$AC$自动机

**定义：**

把AC自动机中所有fail指针逆向,这样就得到了一棵树

(因为每个节点的出度都为1,所以逆向后每个节点入度为1,所以得到的是一棵树)

**算法描述：**

假设我们有了一个AC自动机，然后在上面进行字符串匹配。

![Fail-1](http://images2015.cnblogs.com/blog/903652/201701/903652-20170123220923456-528570908.png)

上面是一个有四个字符串的AC自动机（abcde、aacdf、cdf、cde），虚线是fail指针，实线是转移。

出题人嘿嘿一笑，给了你一个“aaaaaaaaaaaaaaaaaaa”。这样的字符串fail链长度为O(n)的，这就很尴尬了。

我们发现，如果我们把每个x与fail[x]连边，好像形成了一个树结构（fail[x]是x的父节点）。

![Fail-2](http://images2015.cnblogs.com/blog/903652/201701/903652-20170123220924284-1448248561.png)

我们就是要查询一个点到根路径上的cnt之和！我们只要dfs一下预处理出来就行了。

我们来分析一下这个树结构是什么东西。

首先，这棵树是在一个trie的基础上产生的，所以这棵树上的每个点都是一个字符串的前缀，而且每个字符串的每个前缀在这棵树上都对应着一个点。

其次，由于fail指针，每个点父节点的字符串都是这个点字符串的后缀，并且树上没有更长的它的后缀。

**推荐阅读：**

[fail树](http://www.cnblogs.com/zzqsblog/p/6227545.html)

[AC自动机相关Fail树和Trie图相关基础知识 ](http://blog.csdn.net/txl199106/article/details/45315703)


# 数据结构专题

## STL库

## 树状数组(Fenwick Tree)

## 线段树(Segment Tree)

## ZKW线段树(ZKW Segment Tree)

## 主席树

## 李超树

## 分块

## ST表

## 并查集

## 虚树

## 笛卡尔树(Cartesian Tree)

## 左偏树/可并堆(Leftist Tree)

## K-D Tree

## 哈夫曼树(Huffman Tree)

## 跳跃表(SkipList)

## 块状数组/块状链表

## 四分树

## 平衡树

### 树堆(Treap)

### 伸展树(Splay Tree)

### 替罪羊树(Scapegoat Tree)

### 朝鲜树(DPRK Tree)

### 动态树(Link-Cut-Tree)

### 平衡二叉树(Size Balanced Tree)

## 可持久化数据结构

### 可持久化线段树

### 可持久化Trie

### 可持久化AC自动机

### 可持久化主席树

### 可持久化队列

### 可持久化栈

###可持久化并查集

### 可持久化平衡树

### 可持久化块状链表



## 树上的问题

### DFS序

### 树链剖分

### 树分块

### 树分治

#### 点分治

#### 边分治

#### 链分治

## 离线技巧

### 莫队算法

### CDQ分治

### 整体二分



## 其他技巧

### 离散化

### 启发式合并

### 树套树



















