# 动态规划

---

<!--sec data-title="最大乘积子串" data-id="dp_0" data-show=true ces-->

> 给一个浮点数序列，取最大乘积连续子串的值，例如 -2.5，4，0，3，0.5，8，-1，则取出的最大乘积连续子串为3，0.5，8。也就是说，上述数组中，3，0.5，8 这3个数的乘积 $$3 \times 0.5\times 8 = 12$$ 是最大的，而且是连续的。

### Analyze
由于乘法可能有正负情况，可以用两个变量记录以当前元素为末尾的连续元素乘积的最大和最小值：

* `maxend = max(minend*arr[i], maxend*arr[i], arr[i]);`
* `minend = min(minend*arr[i], maxend*arr[i], arr[i]);`

然后记录所有maxend中最大值即可。

### Answer
```c
double MaxMuply(double arr[], int len) {
    // maxend记录以当前元素为末尾的连续元素最大乘积
    // minend记录以当前元素为末尾的连续元素最小乘积
    // maxval记录当前最大乘积
    double maxend = arr[0], minend = arr[0], maxval = arr[0];
    for (int i = 1; i < len; i++) {
        maxend = max(max(minend*arr[i],maxend*arr[i]),arr[i]);
        minend = min(min(maxend*arr[i],minend*arr[i]),arr[i]);
        maxval = max(maxval,maxend);
    }
    return maxval;
}
```

<!--endsec-->

<!--sec data-title="最少变换问题" data-id="dp_1" data-show=true ces-->

> 给定一个源串和目标串，能够对源串进行如下操作： 

> * 在给定位置上插入一个字符 
> * 替换任意字符 
> * 删除任意字符

> 写一个程序，返回最小操作数，使得对源串进行这些操作后等于目标串，源串和目标串的长度都小于2000。

### Analyze
这题可以考虑使用DP方法，用`dp[i][j]`表示将源字符串中前`i`个元素变成目标字符串中前`j`个元素所需的最少步数。则当比较到`src[i-1]`和`tar[j-1]`时，分别考虑三种方案：

* 在`src[i-1]`后插入一个字符，并且要使得该字符为`tar[j-1]`，否则插入没有意义，此时 `dp[i][j] = dp[i][j-1] + 1`，即相当于将`src`的前`i`个字符变成`tar`的前`j-1`个字符后，插入`tar[j-1]`；
* 将`src[i-1]`替换成`tar[j-1]`，当 `src[i-1] == tar[j-1]` 时，`dp[i][j] = dp[i-1][j-1]`，否则需要增加一步替换过程，即 `dp[i][j] = dp[i-1][j-1] + 1`；
* 将`src[i-1]`删除，此时 `dp[i][j] = dp[i-1][j] + 1`，即相当于将`src`前`i-1`个字符变成`tar`前`j`个字符时，删除`src[i-1]`；

综合这三种操作，最小的操作步数即为`dp[i][j]`的结果。

### Answer
```c++
int MiniMove(char src[], int ls, char tar[], int lt) {
  int i, j, k, dp[100][100];
  // dp[i][j]表示src[0~i)与tar[0~j)的最少变化步数
  for (i = 0; i <= ls; i++) {
    dp[i][0] = i;
  }
  for (i = 1; i <= lt; i++) {
    dp[0][i] = i;
  }
  for (i = 1; i <= ls; i++) {
    for (j = 1; j <= lt; j++) {
      // dp[i][j] = dp[i-1][j]+1相当于把src[i]删除操作
      // dp[i][j] = dp[i][j-1]+1相当于在src[i]后插入tar[j]操作
      // dp[i][j] = dp[i-1][j-1]+1/0相当于把src[i]替换成tar[j]操作
      dp[i][j] = min(dp[i][j-1],dp[i-1][j]) + 1;
      if (src[i-1] == tar[j-1]) {
        dp[i][j] = min(dp[i-1][j-1], dp[i][j]);
      }
      else {
        dp[i][j] = min(dp[i-1][j-1] + 1, dp[i][j]);
      }
    }
  }
  return dp[ls][lt];
}
```


<!--endsec-->

<!--sec data-title="最大乘积子串" data-id="dp_2" data-show=true ces-->

> 有 $$n\times n$$ 个格子，每个格子里有正数或者0，从最左上角往最右下角走，只能向下和向右，一共走两次（即从左上角走到右下角走两趟），把所有经过的格子的数加起来，求最大值SUM，且两次如果经过同一个格子，则最后总和SUM中该格子的计数只加一次。

### Analyze
这题有两点值得注意：

* 由于是单调向右下角走，因此总步数一定，为`2n-2`步；
* 由于是 $$n\times n$$ 的格子，因此知道步数`n`和行数`i`就能推算出列数`n-i`；

根据这两点，可以设计DP方程如下`dp[m][i][j]`表示第`m`步时，两条路径分别走到`(i,m-i)`和`(j,m-j)`时的最大和。由于`i`和`j`不分先后，因此在查找时只需要查找`j>=i`的情况。而当`j==i`时，对应的`mat[i][m-i]`只能加一次。最后，还需要考虑`dp[m][i][j]`是否合法的问题，主要就是判断`(i,m-i)`和`(j,m-j)`是否在格子中。

而状态转移方程则比较简单，`(j,n-j)`只可能从上一步的`(j-1,n-j)`或`(j,n-j-1)`转移而来，`(i,n-i)`只可能从上一步的`(i-1,n-i)`或`(i,n-i-1)`转移而来。即：
$$
dp[m][i][j] = \max
\begin{cases}
dp[m-1][i-1][j-1]\\
dp[m-1][i-1][j]\\
dp[m-1][i][j-1]\\
dp[m-1][i][j]
\end{cases}+mat[i][n-i]+mat[j][n-j].
$$
当 `i==j` 时，再减去一个`mat[i][n-i]`即可。

### Answer
```c++
const int MAXN = 100;
int dp[MAXN+MAXN][MAXN][MAXN];
int mat[MAXN][MAXN];
// 判断当前位置是否合法
bool isValid(int n, int i, int j, int k) {
  return j >= 0 && k >= 0 && j < n && k < n && i-j >= 0
      && i-j < n && i-k >= 0 && i-k < n;
}
int getValue(int i, int j, int k, int n) {
  if (isValid(n, i, j, k)) {
    return dp[i][j][k];
  }
  return 0;
}
int MaxSum(int step) {
  int i, j, k, n = step/2+1;
  // dp[i][j][k]表示第i步时两条路径分别到达点mat[j][i-j]和mat[k][i-k]时的最大值
  dp[0][0][0] = mat[0][0];
  for (i = 1; i <= step; i++) {
    for (j = 0; j < n; j++) {
    // j和k其实只需要算一半，另一半是等价的
      for (k = j; k < n; k++) {
        if (isValid(n,i,j,k)) {
          dp[i][j][k] = max(getValue(i-1,j-1,k-1,n),getValue(i-1,j-1,k,n));
          dp[i][j][k] = max(getValue(i-1,j,k-1,n),dp[i][j][k]);
          dp[i][j][k] = max(getValue(i-1,j,k,n),dp[i][j][k]);
          dp[i][j][k] += mat[j][i-j];
          if (j != k) {
            dp[i][j][k] += mat[k][i-k];
          }
        }
      }
    }
  }
  return dp[step][n-1][n-1];
}
```

<!--endsec-->

<!--sec data-title="判断字符串是否交错" data-id="dp_3" data-show=true ces-->

> 输入三个字符串s1、s2和s3，判断第三个字符串s3是否由前两个字符串s1和s2交错而成，即不改变s1和s2中各个字符原有的相对顺序，例如当s1 = “aabcc”，s2 = “dbbca”，s3 = “aadbbcbcac”时，则输出true，但如果s3=“accabdbbca”，则输出false。

### Analyze
用`dp[i][j]`表示`c[0~i+j-1]`是否由`a[0~i-1]`和`b[0~j-1]`组成。则
`dp[i][j]`为true当且仅当`dp[i-1][j]`为true且 `a[i-1]==c[i+j-1]` 或 `dp[i][j-1]`为true且 `b[j-1]==c[i+j-1]`。

### Answer
```c++
bool APBEC(char a[], int la, char b[], int lb, char c[], int lc) {
    if (la + lb != lc) {
        return false;
    }
    int i, j;
    // dp[i][j]表示c[0~i+j-1]是否由a[0~i-1]和b[0~j-1]组成
    bool dp[100][100];
    dp[0][1] = b[0] == c[0] ? true : false;
    dp[1][0] = a[0] == c[0] ? true : false;
    for (i = 1; i <= la; i++) {
        for (j = 1; j <= lb; j++) {
            dp[i][j] = a[i-1] == c[i+j-1] ? dp[i-1][j] : false;
            dp[i][j] |= b[j-1] == c[i+j-1] ? dp[i][j-1] : false;
        }
    }
    return dp[la][lb];
}
```

<!--endsec-->
