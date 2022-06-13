# 单调队列优化DP


[例题](https://www.acwing.com/problem/content/300/)

题目大意：

有$N$块木板从左到右排成一行，有$M$个工匠对这些木板进行粉刷，每块木板至多被粉刷一次。

第$i$个木匠要么不粉刷，要么粉刷包含木板$S_i$的，长度不超过$L_i$的连续的一段木板，每粉刷一块可以得到$P_i$的报酬。

不同工匠的$S_i$不同。

请问如何安排能使工匠们获得的总报酬最多。

题解：

令$DP[i][j]$表示考虑前$i$个工匠、粉刷不超过前$j$块木板（其中有的可以不粉刷）的最大报酬，可以得到转移方程为：

$$
dp[i][j] = max\{dp[i - 1][k] + P_i * (j - k))\},j \geq S_i,j - L_i \leq k \leq S_i - 1
$$


根据该式可以写出第一版代码：

```cpp
#include<bits/stdc++.h>
#define ll long long
#define endl '\n'
#define debug(x) cout << #x << " = " << (x) << endl;
#define IOS ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
using namespace std;
const ll N = 1e2 + 5;
const ll M = 16005;
ll n,m;
struct node{
	ll l,p,s;
}mem[N];
ll dp[N][M];
int main()
{
	IOS;
	cin >> n >> m; 
	for(ll i = 1; i <= m; i ++ )
	{
		cin >> mem[i].l >> mem[i].p >> mem[i].s;
	}
	sort(mem + 1, mem + 1 + m,[&](node a,node b){
		return a.s < b.s;
	});
	for(ll i = 1; i <= m; i ++ )
	{
		for(ll j = 1; j <= n; j ++ )
		{
			for(ll k = 0; k <= j; k ++ )
			{
				dp[i][j] = max(dp[i][j],dp[i - 1][k]);
			}
		}
		for(ll j = mem[i].s; j <= min(n,mem[i].s + mem[i].l - 1); j ++ )
		{
			for(ll k = max(0ll,j - mem[i].l); k < mem[i].s; k ++ )
			{
				dp[i][j] = max(dp[i][j], dp[i - 1][k] + mem[i].p * (j - k));
			}
		}
		for(ll j = 1; j <= n; j ++ )
		{
			for(ll k = 0; k <= j; k ++ )
			{
				dp[i][j] = max(dp[i][j],dp[i][k]);
			}
		}
	}
	ll ans = 0;
	for(ll j = 1; j <= n; j ++ )
	{
		ans = max(ans, dp[m][j]);
	}
	cout << ans << endl;
	
}
```

由于复杂度过高，需要进行优化。

首先，我们没必要每个$j$都遍历一遍$k$去计算，我们可以在$j$后推一步时，仅考虑新的$j$对已经计算好的状态的影响。

观察第二层循环，我们固定$i$，在计算当前的$dp[i][j]$时，考虑这样一种情况：

对于下标为$id_1$和$id_2$的两个元素，$id_1 < id_2$，如果有$dp[i - 1][id_1] + P_i * (j - id_1) \leq dp[i - 1][id_2]$，那么$j$在向后递推时，$id_1$的贡献永远不会大于$id_2$，也就是说，$id_1$是无用的，应该及时舍弃。

故而，我们维护一个单调队列，保证队列中的元素贡献单调递减，对每个$j$直接取队头维护当前的$dp$状态即可。

```cpp
#include<bits/stdc++.h>
#define ll long long
#define endl '\n'
#define debug(x) cout << #x << " = " << (x) << endl;
#define IOS ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
using namespace std;
const ll N = 1e2 + 5;
const ll M = 16005;
ll n,m;
struct node{
	ll l,p,s;
}mem[N];
ll dp[N][M];
int main()
{
	//IOS;
	cin >> n >> m; 
	for(ll i = 1; i <= m; i ++ )
	{
		cin >> mem[i].l >> mem[i].p >> mem[i].s;
	}
	sort(mem + 1, mem + 1 + m,[&](node a,node b){
		return a.s < b.s;
	});

	for(ll i = 1; i <= m; i ++ )
	{		
		deque<ll>q;
		for(ll j = 1; j <= n; j ++ )
		{
			dp[i][j] = dp[i - 1][j];
		}
		for(ll j = max(0ll, mem[i].s - mem[i].l) ; j <= min(n, mem[i].s + mem[i].l - 1); j ++ )
		{
			while(!q.empty() && q.front() < j - mem[i].l){
			
				q.pop_front();
			}
			if(j >= mem[i].s && q.front() < mem[i].s)
				dp[i][j] = max(dp[i][j],dp[i - 1][q.front()] + mem[i].p * (j - q.front()));
			while(!q.empty() && dp[i - 1][q.back()] + mem[i].p * (j - q.back()) <= dp[i - 1][j] && j < mem[i].s)
			{
				q.pop_back();
			}
			q.push_back(j);
		}
		for(ll j = 1; j <= n; j ++ )
		{
			dp[i][j] = max(dp[i][j], dp[i][j - 1]);
		}
		
	}
	cout << dp[m][n] << endl;
	
}


```

还有另外一种维护的角度，来自《算法竞赛进阶指南》，比我的简洁清晰很多：



将转移方程写成如下形式：

$$ 
dp[i][j] = P_i * j + max\{dp[i - 1][k] - P_i * k\},j \geq S_i,j - L_i \leq k \leq S_i - 1 
$$

这时假设有两个元素$k_1,k_2,k_1 < k_2$，那么当$dp[i - 1][k_1] - P_i * k_1 \leq dp[i - 1][k_2] - P_i * k_2$时，$k_1$是一个没有贡献的答案，可以直接舍弃。

代码也是类似的：

```cpp
#include<bits/stdc++.h>
#define ll long long
#define endl '\n'
#define debug(x) cout << #x << " = " << (x) << endl;
#define IOS ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
using namespace std;
const ll N = 1e2 + 5;
const ll M = 16005;
ll n,m;
struct node{
	ll l,p,s;
}mem[N];
ll dp[N][M];
ll q[M];
ll cal(ll i,ll k)
{
	return dp[i - 1][k] - mem[i].p * k;
}
int main()
{
	//IOS;
	cin >> n >> m; 
	for(ll i = 1; i <= m; i ++ )
	{
		cin >> mem[i].l >> mem[i].p >> mem[i].s;
	}
	sort(mem + 1, mem + 1 + m,[&](node a,node b){
		return a.s < b.s;
	});

	for(ll i = 1; i <= m; i ++ )
	{	
		//初始化单调队列 
		ll l = 1,r = 0;
		//把最初的筛选集合插入队列 
		for(ll k = max(0ll,mem[i].s -mem[i].l); k <= mem[i].s - 1; k ++ )
		{
			//插入新决策，维护队尾单调性 
			while(l <= r && cal(i,q[r]) <= cal(i, k))r --;
			q[++ r] = k;
		}
		for(ll j = 1; j <= n; j ++ )
		{
			//不粉刷时的情况，第i个人可以不粉刷，此时dp[i][j] = dp[i - 1][j];第j块木板可以不粉刷，此时dp[i][j] = dp[i][j - 1] 
			dp[i][j] = max(dp[i - 1][j],dp[i][j - 1]);
			//粉刷时的情况 
			if(j >= mem[i].s)
			{
				//排除队头不合法决策 
				while(l <= r && q[l] < j - mem[i].l)l ++ ;
				//队列非空时，取队头进行转移 
				if(l <= r)
				{
					dp[i][j] = max(dp[i][j], cal(i,q[l]) + mem[i].p * j);
				}
			}
		}
	}
	
	cout << dp[m][n] << endl;
	
}


```











