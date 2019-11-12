[江西省ccpc省赛](http://acm.hdu.edu.cn/search.php?field=problem&key=2019CCPC-%BD%AD%CE%F7%CA%A1%C8%FC%A3%A8%D6%D8%CF%D6%C8%FC%A3%A9-+%B8%D0%D0%BB%C4%CF%B2%FD%B4%F3%D1%A7&source=1&searchmode=source)

[A](#A - Cotree) [D](#D - Wave)
## A - Cotree

题意：给你两颗树，要求你在两颗树分别找一个点连成一条边，使得这两颗树连起来的新树结点两两距离之和最小

思路：先求出两个树的重心，然后把两个重心相连，求一下新树的两两结点距离之和，这个和就是最小值。

具体过程：

### 怎么区分出两颗树？
可以通过并查集把给的数据分成两部分，也就是两个树。在并查集完毕之后，`fa[i] = i,fa[j] = j,i!=j` 此时可以把i，j认为是两个树的根节点

### 怎么求树的重心
一个树去掉某个点之后，会剩下若干个子树，若去掉某点，剩下的最大的子树结点数最小，那么这个去掉的点就是`重心`
1. 首先对树跑一遍DFS，算出每棵子树的结点个数，存放在`sz[maxn]`数组中
ve是存图的链接表，sz[i]:以i为根结点的树的结点个数
```cpp
ll DFS_coun(ll u,ll fa = -1){
    ll sum = 1;
    for(ll i = 0;i<ve[u].size();i++){
        if(ve[u][i] == fa) continue;
        sum += DFS_coun(ve[u][i],u);
    }
    return sz[u] = sum;
}
```
2. 再对树一遍DFS，计算去掉某个点最大的树结点个数，并更新最大值，使得最大的树结点个数最小的结点，就是重心
u:当前结点 sum：这个树的总结点数 h：存放重心 fa：u结点的父结点
```cpp
void DFS_find(ll u,ll sum,ll &h,ll fa = -1){
    ll m = max(sum-sz[u],0LL);
    for(ll i = 0;i<ve[u].size();i++)
        if(ve[u][i]!=fa) m = max(m,sz[ve[u][i]]);
    if(m<mx) h = u,mx = m;
    for(ll i = 0;i<ve[u].size();i++){
        int v = ve[u][i];
        if(v != fa) DFS_find(v,sum,h,u);
    }
}
```
### 怎么计算一个树两两结点距离之和？
对于一条边，每一个左边的结点都需要和右边的一个结点相连形成一条路，所以这条边经过的次数是`左边结点数*右边结点数`
只要遍历完每一条边就进行累加即可
```cpp
ll DFS_dis(ll u,ll fa = -1){
    ll sum = 0;
    for(ll i = 0;i<ve[u].size();i++){
        ll v = ve[u][i];
        if(v == fa) continue;
        sum += sz[v]*(sumT3-sz[v]);
        sum += DFS_dis(v,u);
    }
    return sum;
}
```
### 为什么连接两个树的重心，形成的新树就能保证两两结点距离之和是最小的？
两个重心连接起来的边就像让是两个树联通的一座桥梁，我们的目的就是在两个树之间建一个桥，要保证桥两边的结点到桥的端点距离之和是最近的，形成的新树两两结点距离之和就是最小的。而这个端点正是重心。重心就是保证，树的所有结点到此点的距离之和是最小的。

完整代码：
```cpp
#include <iostream>
#include <vector>
using namespace std;
typedef long long ll;

const ll maxn = 1e5+10;

ll N,sumT1,sumT2,sumT3,H1,H2,mx;//结点总数，树1的结点数，树2的结点数，新树的结点数，树1的重心，树2的重心
ll fa[maxn];
ll sz[maxn];//sz[i]:以i为根结点的树的结点数
vector<ll> ve[maxn];

ll find(ll x){
    return x == fa[x]? x:fa[x] = find(fa[x]);
}
void join(ll x,ll y){
    ll fx = find(x),fy = find(y);
    if(fx != fy) fa[fx] = fy;
}

ll DFS_coun(ll u,ll fa = -1){
    ll sum = 1;
    for(ll i = 0;i<ve[u].size();i++){
        if(ve[u][i] == fa) continue;
        sum += DFS_coun(ve[u][i],u);
    }
    return sz[u] = sum;
}

void DFS_find(ll u,ll sum,ll &h,ll fa = -1){
    ll m = max(sum-sz[u],0LL);
    for(ll i = 0;i<ve[u].size();i++)
        if(ve[u][i]!=fa) m = max(m,sz[ve[u][i]]);
    if(m<mx) h = u,mx = m;
    for(ll i = 0;i<ve[u].size();i++){
        int v = ve[u][i];
        if(v != fa) DFS_find(v,sum,h,u);
    }
}

ll DFS_dis(ll u,ll fa = -1){
    ll sum = 0;
    for(ll i = 0;i<ve[u].size();i++){
        ll v = ve[u][i];
        if(v == fa) continue;
        sum += sz[v]*(sumT3-sz[v]);
        sum += DFS_dis(v,u);
    }
    return sum;
}

int main(){
    cin>>N;
    for(ll i = 1;i<=N;i++) fa[i] = i;
    ll a,b;
    for(ll i = 1;i<=N-2;i++){
        scanf("%lld%lld",&a,&b);
        ve[a].push_back(b);
        ve[b].push_back(a);
        join(a,b);
    }
    ll root1 = -1,root2 = -1;
    for(ll i = 1;i<=N;i++) find(i);
    for(ll i  = 1;i<=N;i++){
        if(fa[i] == i){
            if(root1 == -1) root1 = i;
            else root2 = i;
        }
    }
    sumT1 = DFS_coun(root1);
    sumT2 = DFS_coun(root2);
    H1 = root1,H2 = root2;
    sumT3 = sumT1+sumT2;
    mx = 0x3f3f3f3f;
    DFS_find(root1,sumT1,H1);
    mx = 0x3f3f3f3f;
    DFS_find(root2,sumT2,H2);
    ve[H1].push_back(H2);
    ve[H2].push_back(H1);

    DFS_coun(H1);
    ll res = DFS_dis(H1);
    cout<<res<<endl;
    return 0;
}
```
## D - Wave

`题意`：给一个序列，让找一个最大的子序列，这个子序列的奇数位置元素相同，偶数位置相同，奇数位置元素不同于偶数位置元素，输出满足这个条件的最大子序列的长度

`思路`：先用vector记录每种元素出现的下标，然后对两两元素对应的vector求组成的子序列的长度，并更新

### 如何计算两两元素对应的vector求组成的子序列的长度？
其实就和打牌一样，一方出一张牌，另一方出一张比它大的牌，反复执行这个过程，直到有一方没有牌出就结束，注意：为了形成的序列最大，开始时牌小的先出

```cpp
#include <iostream>
#include <vector>
using namespace std;

vector<int> ve[110];
int arr[1000010];
int N,C;

int getlong(int a,int b){
    int res = 0,len1 = ve[a].size(),len2 = ve[b].size();
    if(len1 == 0 || len2 == 0) res = 0;
    else if(ve[a][0] < ve[b][0]){
        int i = 0,j = 0;
        res++;
        while(true){
            while(j<len2 && ve[b][j]<ve[a][i]) j++;
            if(j == len2) break;
            res++;
            while(i<len1 && ve[a][i]<ve[b][j]) i++;
            if(i == len1) break;
            res++;
        }
    }else{
        int i = 0,j = 0;
        res++;
        while(true){
            while(i<len1 && ve[a][i]<ve[b][j]) i++;
            if(i == len1) break;
            res++;
            while(j<len2 && ve[b][j]<ve[a][i]) j++;
            if(j == len2) break;
            res++;
        }
    }
    return res>1? res:0;
}
int main(){
    cin>>N>>C;
    for(int i = 0;i<N;i++) scanf("%d",&arr[i]);
    for(int i = 0;i<N;i++){
        ve[arr[i]].push_back(i);
    }
    int res = 0;
    for(int i = 1;i<=C;i++){
        for(int j = i+1;j<=C;j++){
            int lent = getlong(i,j);
            res = max(res,lent);
        }
    }
    cout<<res<<endl;

    return 0;
}
```
## F - String

题意：给一个字符串，在字符串中任意挑一个字母（循环4次），求挑出来的字母能组成`avin`的概率是多大，将概率分数约分后输出
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

string s;
int main(){
	int len;
	while(scanf("%d",&len)!=EOF){
		cin>>s;
		map<char,int> mp;
		for(const auto &c: s) mp[c]++;
		ll son = mp['a']*mp['v']*mp['i']*mp['n'];
		ll mu = len*len*len*len;
		ll g = __gcd(son,mu);
		son/=g,mu/=g;
		printf("%lld/%lld\n",son,mu);
		
	}
	return 0;
}
```
## G - Traffic

题意：在一个十字路口，有东西方向行驶的车，有南北方向行驶的车，为了不相撞，让南北方向行驶的车都等待相同的时间，求这个时间的最小值

分析：将问题转换成，有两个数组，把一个数组都统一加一个值，使得两个数组没有交集，所以枚举等待时间，直接暴力即可

```cpp
#include <bits/stdc++.h>
#include <sstream>
using namespace std;
typedef long long ll;

int arr1[1100],arr2[1100],arr3[2200],N1,N2;
map<int,int> mp;


int main(){
	int N1,N2;
	while(scanf("%d%d",&N1,&N2)!=EOF){
		int mx =0 ,mi = 0;
		for(int i = 0;i<N1;i++) scanf("%d",&arr1[i]),mx = max(mx,arr1[i]);
		for(int i = 0;i<N2;i++) scanf("%d",&arr2[i]),mi = min(mi,arr2[i]);
		
		mp.clear();
		for(int i = 0;i<N1;i++) mp[arr1[i]]++;
		for(int i = 0;i<= mx-mi+1;i++){
			int ok = true;
			for(int j = 0;j<N2;j++){
				if(mp[arr2[j]+i]!=0){
					ok = false;
					break;
				}
			}
			if(ok){
				cout<<i<<endl;
				break;
			}
		}
	}
	
	
	return 0;
}
```
## H - Rng

题意: 给一个区间[1,n],分别两次在[1,n] 选两个子区间[l,r]，求两个子区间不相交的概率

分析：可以将问题转换成`1 - 两个子区间不相交的概率`。对于两个区间，左边的区间为[l1,r1],右边的区间为[l2,r2],只要满足r1<l2,两个子区间就是不相交的。
所以只要枚举r1<l2的所有情况数，然后除以r1，l2任意选择情况数就是两个子区间不相交的概率

因为运算要进行取模，所以除法要变成乘以其逆元，求逆元可以用费马小定理，a的逆元为a^(mod-2)

```cpp
#include <iostream>
#include <vector>
using namespace std;
typedef long long ll;

const ll mod = 1000000007;
ll N;
ll ksm(ll a,ll b){
    ll res = 1;
    while(b){
        if(b%2) res = res*a%mod;
        b/=2;
        a = a*a%mod;
    }
    return res;
}

int main(){

    cin>>N;
    ll res = (N+1)*ksm(2*N,mod-2)%mod;
    cout<<res<<endl;
    return 0;
}
```
## I - Budget

题意：现在银行要将小数点保留3位改成保留2位，求修改之后的总值的变化值

分析：按字符串读入，只需要判断最后一位，如果它小于'5',就会舍去，否则就多加上'10'-'5'

```cpp
#include <bits/stdc++.h>
#include <sstream>
using namespace std;
typedef long long ll;


int main(){

	int N;
	while(scanf("%d",&N)!=EOF){
		double ans = 0;string s;
		while(N--){
			cin>>s;
			int t = s[s.size()-1]-'0';
			if(t<5) ans -= t;
			else ans += (10-t);
		}
		ans /= 1000;
		printf("%.3f\n",ans);
	}	
	return 0;
}
```

## J - Worker

题意：一个车间一个人产x个零件，有n个车间，m个人，每个车间都分配一定的人数，人数需要分配完，并且使得每个车间产的零件数一样多

分析：假如A车间一人产1个零件，B车间一人产3个零件，如果要让两个车间产量一样，那么必须A车间分配的人数是B车间的3*k倍。所以只需要求所有车间一人产量的最小公倍数p，只要m是p的倍数就可以实现每个车间的产量一样多

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

ll N,M;
ll W[1100];

ll lcm(ll a,ll b){
	return (a*b)/(__gcd(a,b));
}
int main(){
	
	while(scanf("%lld%lld",&N,&M)!=EOF){
		for(int i = 0;i<N;i++) scanf("%lld",&W[i]);
		ll L = W[0],sum = 0;
		for(int i = 0;i<N;i++){
			L = lcm(L,W[i]);
		}
		for(int i = 0;i<N;i++){
			sum += L/W[i];
		}
		if(M%sum!=0) cout<<"No"<<endl;
		else{
			cout<<"Yes"<<endl;
			ll t = M/sum;
			for(int i = 0;i<N;i++){
				printf("%lld",L/W[i]*t);
				if(i<N-1) putchar(' ');
			}
			cout<<endl;
		}
	}
	
	return 0;
}
```

## K - Class

题意：解方程

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;


ll x,y;
int main(){
	cin>>x>>y;
	ll a = (x+y)/2,b = (x-y)/2;
	cout<<a*b<<endl;
	return 0;
}
```





























