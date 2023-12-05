令f[i]=v表示长度为i的LIS的最后一个数为v，如果有多个长度为i的LIS，记录最小的v，易知f是单调序列（f[i + 1]一定大于f[i]）。

遍历待求解数列a，每遇到一个a[i]，对f进行二分，求比a[i]小且最大的f[i]，令f[i + 1] = min(f[i + 1], a[i]);

代码：

```c++
#include<bits/stdc++.h>
#define ll long long
#define debug(x) cout << #x << " = " << (x) << endl
#define lson(x) 2*(x)
#define rson(x) 2*(x)+1
#define INF 0x3f3f3f3f3f3f3f3f
using namespace std;
const ll N = 2e5 + 5;
ll n;
ll a[N], f[N];
ll lis[N];
void solve()
{
	cin >> n;
	for(ll i = 1; i <= n; i ++ )
	{
		cin >> a[i];
		lis[i] = 0;
		f[i] = INF;
	}

	
	for(ll i = 1; i <= n; i ++ )
	{
		ll l = 0, r = n;
		while(l < r)
		{
			ll mid = (l + r + 1) / 2;
			if(f[mid] < a[i])
			{
				l = mid;
			}
			else
			{
				r = mid - 1;
			}
		}
		if(a[i] > f[l])
		{
			f[l + 1] = min(f[l + 1], a[i]);
			lis[i] = l + 1;
		}
	}
	for(ll i = 1; i <= n; i ++ )
	{
		cout << lis[i] << ' ';
	}
	cout << endl;
	
}
int main()
{
	ll _;
	cin >> _;
	while(_ -- )
	{
		solve();
	}
}
```

