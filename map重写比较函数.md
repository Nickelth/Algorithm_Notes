# map重写比较函数

当我们使用结构体作为map的key时，需要重载<号.

比如：

```cpp
struct node{
	ll x,y,v;
	node(ll t,ll b,ll c)
	{
		x = t;
		y = b;
		v = c;
	}
	bool operator<(const node&b)const 
	{
		if(y == b.y && x == b.x)
		{
			return v < b.v;
		}
		else if(x == b.x)
		{
			return y < b.y;
		}
		else
		{
			return x < b.x;
		}
	}
};

```
声明时则可以直接：

```cpp
map<node,ll>mp;
```