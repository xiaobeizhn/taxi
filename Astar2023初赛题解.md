# 百度之星2023初赛题解

​	*前言：2023年8月6日13:50，百度之星赛事授权平台码蹄集服务器遭到攻击，由于比赛所用服务器未与题库分割，导致服务器CPU占用飙升，随后服务器崩溃，比赛延迟到16:00并换题。虽然我自己没打，但是听说16:00还是很卡，打竞赛这么多年什么垃圾服务器没见过，第一次见到服务器被攻击结果崩溃了的。附截图和官方回应*

## 第一场

官方讲解视频 [【2023百度之星第一场题解】嘉宾：NOI、IOI金牌周航锐](https://www.bilibili.com/video/BV1p14y1z7sF/?t=77.2&vd_source=e1934b68ca16e40d94b9d1b1095d68b4)

### T1 公园（最短路）

​	**dis[0]**表示T起点的最短路，**dis[1]**表示F起点的最短路，**dis[2]**表示n为起点的最短路，枚举每一个他们相遇的点，然后计算两人分别到该点的最短路径和两人从该点一起到终点的最短路径。

```c++
#include<bits/stdc++.h>
#define N 40005
#define LL long long
using namespace std;
template<typename T>
const T & Max(const T &a,const T &b){
    return a>b?a:b;
}
template<typename T>
const T & Min(const T &a,const T &b){
    return a<b?a:b;
}
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

​	memset函数是对n个字节进行赋值。而char类型占1个字节。但是int类型占4个字节，所以memset里赋值0x3f最后dp会等于0x3f3f3f3f。同理如果对long long数组进行memset会等于0x3f3f3f3f3f3f3f3f(8个3f)。同时最后不可以根据Ans有无变化来判断能否到达终点，因为int的定义的无限大比long long小，即使无法到达终点还是会更新Ans