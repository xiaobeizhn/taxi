﻿# 百度之星2023初赛第一场题解

​	*前言：2023年8月6日13:50，百度之星赛事授权平台码蹄集服务器遭到攻击，由于比赛所用服务器未与题库分割，导致服务器CPU占用飙升，随后服务器崩溃，届时微信公众号、考试客户端以及码蹄集网站全部瘫痪，比赛延迟到16:00并换题。虽然我自己没打，但是听说16:00还是很卡，比赛刚开始服务器比较卡很正常，不过服务器被攻击结果崩溃了的还是第一次见。附官方回应：*
![在这里插入图片描述](https://img-blog.csdnimg.cn/57f20b3c34c84c569f80b048aacfd294.png#pic_center)



[题目链接](https://www.matiji.net/exam/ojquestionlist?questionBankId=179CE77A7B772D15A8C00DD8198AAC74)	[官方讲解视频](https://www.bilibili.com/video/BV1p14y1z7sF/?t=77.2&vd_source=e1934b68ca16e40d94b9d1b1095d68b4)

### T1 公园（最短路）

​	数组`dis[0]`表示T起点的最短路，`dis[1]`表示F起点的最短路，`dis[2]`表示n为起点的最短路，枚举每一个他们相遇的点，然后计算两人分别到该点的最短路径和两人从该点一起到终点的最短路径。

```cpp
#include<bits/stdc++.h>
#define N 40005
#define LL long long
using namespace std;
int n,m,T,F,TE,FE,S;
struct node{
    int x,d;
    bool operator < (const node &_)const{
        return d>_.d;
    }
};
vector<int>G[N];
int dp[3][N];
bool mark[N];
priority_queue<node>q;
void dij(int s,int *dis){
    memset(mark,0,sizeof mark);
    memset(dis,0x3f,sizeof dp[0]);
    dis[s]=0;
    q.push((node)<%s,0%>);
    while(q.size()){
        int x=q.top().x;
        q.pop();
        if(mark[x])continue;
        mark[x]=1;
        for(int i=0;i<G[x].size();i++){
            int y=G[x][i];
            if(dis[y]>dis[x]+1){
                dis[y]=dis[x]+1;
                q.push((node)<%y,dis[y]%>);
            }
        }
    }
}
int main(){
    cin>>TE>>FE>>S;
    cin>>T>>F>>n>>m;
    for(int i=0;i<m;i++){
        int a,b;
        scanf("%d%d",&a,&b);
        G[a].push_back(b);
        G[b].push_back(a);
    }
    dij(T,dp[0]);
    dij(F,dp[1]);
    dij(n,dp[2]);
    LL Ans=0x7f7f7f7f7f7f7f7f;
    for(int i=1;i<=n;i++){
        if(1LL*dp[0][i]*TE+1LL*dp[1][i]*FE+1LL*dp[2][i]*(TE+FE-S)<Ans)Ans=1LL*dp[0][i]*TE+1LL*dp[1][i]*FE+1LL*dp[2][i]*(TE+FE-S);
    }
    if(dp[0][n]==0x3f3f3f3f||dp[1][n]==0x3f3f3f3f)puts("-1");
    else cout<<Ans<<endl;
    return 0;
}
```

​	memset函数是对n个字节进行赋值。而char类型占1个字节。但是int类型占4个字节，所以memset里赋值**0x3f**最后dp会等于**0x3f3f3f3f**。同理如果对long long数组进行memset会等于**0x3f3f3f3f3f3f3f3f**(8个3f)。同时最后不可以根据Ans有无变化来判断能否到达终点，因为int的定义的无限大比long long小，即使无法到达终点还是会更新Ans



### T2蛋糕划分（dp 或 二分+贪心）

​	首先考虑一行n列竖着切的情况，可以二分最大的蛋糕重量**mid**，然后通过不断地切小于等于**mid**的蛋糕看看最后有没有超过**K**刀来check **mid** 是否可行，复杂度为：
$$
O(log_2(n*W)*K)
$$
​	也可以通过动态规划，`dp[i][j]`表示在第i个后切第j刀后前**i**个中最大的蛋糕的质量的最小值，`dp[i][j] = Min(Max(dp[t][j-1], s[i] - s[t]])),(t=j-1..i-1)`复杂度为：
$$
O(n^2*K)
$$
​	接下来再考虑n行n列的情况，因为**n<=15**所以枚举横着的每一种切法是2^15也就是32768然后在这基础上竖着切就行，dp的话30000×15×15×30=2e8可以跑，附上dp的代码

```cpp
#include <bits/stdc++.h>
#define N 20
#define INF 0x3f3f3f3f
#define LL long long
using namespace std;
template <typename T>
const T& Max(const T& a, const T& b) {
    return a > b ? a : b;
}
template <typename T>
const T& Min(const T& a, const T& b) {
    return a < b ? a : b;
}
int n, K;
int A[N][N];
int s[N][N];
int ans = INF;
int dp[N][N];
void calc(int vis, int tot) {
    memset(dp, 0x3f, sizeof dp);
    dp[0][0] = 0;
    for (int i = 1; i < n; i++) {
        for (int j = 1; j <= Min(i, K - tot); j++) {
            for (int p = j - 1; p <= i - 1; p++) {
                if (dp[p][j - 1] == INF)
                    continue;
                int res = 0;
                int pre = 0;
                for (int t = 1; t <= n; t++) {
                    if (1 << (t - 1) & vis) {
                        res =
                            Max(res, s[t][i] + s[pre][p] - s[t][p] - s[pre][i]);
                        pre = t;
                    }
                }
                dp[i][j] = Min(dp[i][j], Max(dp[p][j - 1], res));
            }
        }
    }
    for (int p = K - tot; p < n; p++) {
        if (dp[p][K - tot] == INF)
            continue;
        int res = 0;
        int pre = 0;
        for (int t = 1; t <= n; t++) {
            if (1 << (t - 1) & vis) {
                res = Max(res, s[t][n] + s[pre][p] - s[t][p] - s[pre][n]);
                pre = t;
            }
        }
        ans = Min(ans, Max(dp[p][K - tot], res));
    }
}
void dfs(int vis, int now, int k) {
    if (now == n - 1 || k == K) {
        calc(vis | 1 << (n - 1), k);
        //在右边界补上一刀vis|1<<(n-1)方便计算最下面一行蛋糕的质量
        return;
    }//最后一个后面切一刀相当于切在右边界上不算一刀，故now==n-1就计算贡献
    if (k < K)
        dfs(vis | 1 << now, now + 1, k + 1);
    dfs(vis, now + 1, k);
    return;
}
int main() {
    scanf("%d%d", &n, &K);
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            scanf("%d", &A[i][j]);
        }
    }
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            s[i][j] = s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1] + A[i][j];
        }
    }
    dfs(0, 0, 0);
    cout << ans << endl;
    return 0;
}
```



### T3第五维度（二分）

​	第一种方法是枚举消失的科学家，然后二分。

​	注意到科学家消失时会把先前的贡献全部删除，因此科学家消失的时间不影响结果，可以开始直接消失；注意到这里已经用掉 **O(n)** 的复杂度了，若再用 **O(n)** 复杂度来计算当前科学家消失所需的时间明显超时，考虑二分时间**t**，计算到时间**t**时的理解力总数，但是不知道哪些科学家在**t**前已经开始思考，故考虑先将科学家按开始时间排序，接着预处理前k个开始思考的科学家的总理解速度**sv[k]**以及第**k**个科学家刚开始思考时的已经累积的理解力总数**ss[k]**。

​	至此先二分时间再二分**t**时正在思考的科学家 **logt logn** 仍然会超时，因此我们考虑能不能把**log(t)**去掉。

​	我们发现到第 **k+1** 个科学家开始思考前的总理解速度 **sv[k]** 是保持不变的，也就是说只要我们确定总理解力超过**m**前最后一个开始思考的科学家**ans**我们就能通过`A[ans].s + (m - ss[ans]) / sv[ans]`直接算出理解的时间，同时如果前**ans**个科学家里有消失的科学家需要在 **ss[ans]** 和 **sv[ans]** 里减掉，即

​				`A[ans].s + (m - (ss[ans] - A[i].v * (A[ans].s - A[i].s))) / (sv[ans] - A[i].v)` 。

​	总时间复杂度为：
$$
O(nlogn)
$$

```cpp
#include <bits/stdc++.h>
#define N 100005
#define LL long long
using namespace std;
template <typename T>
const T& Max(const T& a, const T& b) {
    return a > b ? a : b;
}
int n, m;
struct node {
    LL s, v;
} A[N];
LL sv[N], ss[N];
bool cmp(node a, node b) {
    return a.s < b.s;
}
int main() {
    scanf("%d%d", &n, &m);
    int flag = 0;
    for (int i = 1; i <= n; i++) {
        scanf("%lld%lld", &A[i].s, &A[i].v);
        if (A[i].v > 0) flag++;
    }
    if (flag < 2) return puts("-1"), 0;  // 只有1人思考或无人思考则永远无法理解
    sort(A + 1, A + 1 + n, cmp);
    for (int i = 1; i <= n; i++) {
        sv[i] = sv[i - 1] + A[i].v;
    }
    for (int i = 2; i <= n; i++) {
        ss[i] = ss[i - 1] + sv[i - 1] * (A[i].s - A[i - 1].s);
    }
    LL Ans = -1;
    for (int i = 1; i <= n; i++) {
        int ans = 0;
        int l = 1, r = n, mid;
        while (l <= r) {
            mid = l + r >> 1;
            if (i >= mid && ss[mid] <= m || i < mid && ss[mid] - A[i].v * (A[mid].s - A[i].s) <= m)
                ans = mid, l = mid + 1;
            else
                r = mid - 1return puts("-1"),0;
        }
        LL res = (ans >= i ? 
            A[ans].s + (m - (ss[ans] - A[i].v * (A[ans].s - A[i].s))) / (sv[ans] - A[i].v)
          : A[ans].s + (m - ss[ans]) / sv[ans]);
        Ans = Max(Ans, res + 1);  // 超过m才算理解，故加一
    }
    cout << Ans;
    return 0;
}
```

​	第二种方法也就是标答的方法，运用二分+贪心。先二分时间**t**，要想在**t**前的总理解力最少，只需要让在**t**时理解力最多的科学家消失，接着判断科学家消失后总理解力是否大于**t**。总时间复杂度为
$$
O(nlog(S+m))，S+m为最慢理解时间
$$
​	但是由于常数大，logn跑的并没有log(S+m)快（悲

```cpp
#include <iostream>
#define N 100005
#define LL long long
using namespace std;
LL s[N], v[N], n, m;
bool check(int t) {  // 二分答案,若时间为t,能理解,为真
    LL sum = 0, maxn = -1;
    for (int i = 1; i <= n; i++) {
        if (t - s[i] <= 0)
            continue;
        sum += (t - s[i]) * v[i];
        maxn = max(maxn, (t - s[i]) * v[i]);
    }
    return sum - maxn > m;
}
int main() {
    cin >> n >> m;
    int flag = 0;
    for (int i = 1; i <= n; i++) {
        scanf("%lld%lld",s+i,v+i);
        v[i]>0&&(flag++);
    }
    if (flag < 2) return puts("-1"), 0;
    int l = 0, r = 4e9+5;
    while (l < r) {
        int mid = (l + r) / 2;
        if (check(mid))
            r = mid;
        else
            l = mid + 1;
    }
    cout << l <<endl;
    return 0;
}
```



### T5 糖果促销（数论）

​		第一种方法是二分，复杂度**O(T log k log k)**，5秒随便跑

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
int K,P;
int check(int now){
    int t=now,m=now;
    while(m/P>0){
        t+=m/P;
        m=m/P+m%P;
    }
    return t;
}
int main(){
    int T;
    cin>>T;
    while(T--){
        scanf("%d%d",&P,&K);
        if(K==0){
            puts("0");
            continue;
        }
        if(P==1){
            puts("1");
            continue;
        }
        int l=1,r=K,ans;
        while(l<=r){
            int mid=l+r>>1;
            if(check(mid)>=K)ans=mid,r=mid-1;
            else l=mid+1;
        }
        printf("%d\n",ans);
    }
    return 0;
}
```

​		第二种方法是直接算，这种我是看了别人的才知道的（

```cpp
#include<bits/stdc++.h>
#define LL long long
using namespace std;
int K,P;
int main(){
    int T;
    cin>>T;
    while(T--){
        scanf("%d%d",&P,&K);
        if(K==0){
            puts("0");
            continue;
        }else{
            K-=(K-1)/P;
            printf("%d\n",K);
        }
    }
    return 0;
}
```

​		整个过程一共会有**k**个糖果纸，且最后手上一定至少有一个糖果纸，所以可以换到的糖果数量为**(k-1)/p**，所以**k-(k-1)/p**就是需要买的糖果数量。

### T7数字串（后缀数组）

​	首先考虑一个加号放在哪，每一位的数字用字母表示

​		①.a+bcdef	②.ab+cdef	③.abc+def	④.abcd+ef	⑤.abcde+f

​	明显①、⑤要比③大，因为①、⑤至少5位，③至多4位，那么②、④和①、⑤比呢？先比较①和②

​	①减②得 **(a + 10000b) - (10a + b) = 9999b - 9a > 0** ，由于字符串不包含0，所以①大，④和⑤同理

​	那么我们可以得到**①>②>③<④<⑤**，我们可以猜测，将加号放在两边得到的结果最大。具体的证明过程可以去看讲解视频。

​	此时考虑多个加号的情况，明显只需要继续把加号放在两边即可，一共有**k**个加号，因此我们只需要考虑如何使中间的连续**n - k**位最大即可。问题转化为寻找字符串的最大后缀，可以采用**SA-IS**算法，复杂度**O(n)**，这个算法非常精妙，复杂度也最低，我看了整整一天才大概看懂，已经放弃了；也可以采用**DC3**算法，它虽然是**O(n)**的算法，但是常数巨大，当有了**SA-IS**算法以后，它就不再有使用价值；本人采用倍增法求后缀数组，时间复杂度：
$$
O(nlogn)
$$
```cpp
#include <bits/stdc++.h>
#define N 1000005
#define LL long long
using namespace std;
char str[N];
int n, K;
int cnt[N], SA[N], subSA[N], RK[N][22];
struct Big {
    int length;
    int num[N];
    void operator+=(const LL a) {
        num[0] += a;
        for (int i = 0; i < length; i++) {
            num[i + 1] += num[i] / 10;
            num[i] %= 10;
        }
        while (num[length] > 0)
            length++;
        return;
    }
    void get(int begin) {
        length = n - K;
        for (int i = 0; i < length; i++)
            num[i] = str[begin + length - i - 1] - '0';
        return;
    }
    void Println() const {
        for (int i = length - 1; i >= 0; i--)
            printf("%d", num[i]);
        puts("");
        return;
    }
} Ans;
int main() {
    scanf("%d%d", &n, &K);
    scanf("%s", str + 1);
    for (int i = 1; i <= n; i++)
        cnt[(str[i] - '0')]++;
    for (int i = 1; i < 10; i++)
        cnt[i] += cnt[i - 1];
    for (int i = 1; i <= n; i++)
        RK[i][0] = cnt[str[i] - '0'];
    for (int i = n; i > 0; i--)
        SA[cnt[str[i] - '0']--] = i;
    int j, p;
    for (j = 1, p = 1; j < n; j <<= 1, p++) {
        int t = 0;
        for (int i = n - j + 1; i <= n; i++)
            subSA[++t] = i;
        for (int i = 1; i <= n; i++)
            if (SA[i] > j)
                subSA[++t] = SA[i] - j;
        memset(cnt, 0, sizeof cnt);
        for (int i = 1; i <= n; i++)
            cnt[RK[i][p - 1]]++;
        for (int i = 1; i <= n; i++)
            cnt[i] += cnt[i - 1];
        for (int i = n; i > 0; i--)
            SA[cnt[RK[subSA[i]][p - 1]]--] = subSA[i];
        t = 1, RK[SA[1]][p] = 1;
        for (int i = 2; i <= n; i++) {
            if (RK[SA[i]][p - 1] == RK[SA[i - 1]][p - 1] &&
                RK[SA[i] + j][p - 1] == RK[SA[i - 1] + j][p - 1])
                RK[SA[i]][p] = t;
            else
                RK[SA[i]][p] = ++t;
        }
        if (t == n) {
            p++;
            break;
        }
    }
    p--;
    int maxsuf = 1;
    for (int i = 2; i <= K + 1; i++)
        if (RK[maxsuf][p] < RK[i][p])
            maxsuf = i;
    Ans.get(maxsuf);
    LL res = 0;
    for (int i = 1; i < maxsuf; i++)
        res += 1LL * (str[i] - '0');
    for (int i = maxsuf + n - K; i <= n; i++)
        res += 1LL * (str[i] - '0');
    Ans += res;
    Ans.Println();
    return 0;
}
```

​	关于后缀排序，洛谷有一道模板题 [P3809 【模板】后缀排序](https://www.luogu.com.cn/problem/P3809) ，但是字符串包含大小写英文字母和0.

​	参考题解：[Rainy7の灯塔](https://www.luogu.com.cn/blog/Rainy7/post-hou-zhui-pai-xu-sa-xue-xi-bi-ji)

​	在讨论区还有一种Hash+二分的方法我觉得也挺好的。由于只需要求出最大的后缀，可以把n个后缀依次与当前最大后缀比较，通过Hash比较两个后缀的子串是否相同，用二分找出两个后缀的最长公共前缀，然后比较最长公共前缀的后一个字符就能比较大小。时间复杂度：
$$
O(nlogn)
$$
```cpp
#include <bits/stdc++.h>
#define N 1000005
#define LL long long
const LL P = 1e9 + 7;
const LL base = 17;
using namespace std;
char str[N];
int n, K;
struct Big {
    int length;
    int num[N];
    void operator+=(const LL a) {
        num[0] += a;
        for (int i = 0; i < length; i++) {
            num[i + 1] += num[i] / 10;
            num[i] %= 10;
        }
        while (num[length] > 0)
            length++;
        return;
    }
    void get(int begin) {
        length = n - K;
        for (int i = 0; i < length; i++)
            num[i] = str[begin + length - i - 1] - '0';
        return;
    }
    void Println() const {
        for (int i = length - 1; i >= 0; i--)
            printf("%d", num[i]);
        puts("");
        return;
    }
} Ans;
int maxsuf = 1;
LL Hash[N], Base[N];
LL get_hash(int a, int b) {
    return (P + Hash[b] - (Hash[a - 1] * Base[b - a + 1] % P)) % P;
}
int main() {
    scanf("%d%d", &n, &K);
    scanf("%s", str + 1);
    Base[0] = 1, Hash[0] = 0;
    for (int i = 1; i <= n; i++) {
        Hash[i] = (Hash[i - 1] * base % P + str[i] - '0') % P;
        Base[i] = Base[i - 1] * base % P;
    }
    for (int i = 2; i <= K + 1; i++) {
        int l = 1, r = n - K, mid, ans = 0;
        while (l <= r) {
            mid = l + r >> 1;
            if (get_hash(maxsuf, maxsuf + mid - 1) == get_hash(i, i + mid - 1))
                l = mid + 1, ans = mid;
            else
                r = mid - 1;
        }
        if (ans < n - K && str[maxsuf + ans] < str[i + ans])
            maxsuf = i;
    }
    Ans.get(maxsuf);
    LL res = 0;
    for (int i = 1; i < maxsuf; i++)
        res += 1LL * (str[i] - '0');
    for (int i = maxsuf + n - K; i <= n; i++)
        res += 1LL * (str[i] - '0');
    Ans += res;
    Ans.Println();
    return 0;
}
```
未完待续



