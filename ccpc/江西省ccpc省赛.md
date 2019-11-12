[江西省ccpc省赛](http://acm.hdu.edu.cn/search.php?field=problem&key=2019CCPC-%BD%AD%CE%F7%CA%A1%C8%FC%A3%A8%D6%D8%CF%D6%C8%FC%A3%A9-+%B8%D0%D0%BB%C4%CF%B2%FD%B4%F3%D1%A7&source=1&searchmode=source)
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
对于一条边，每一个左边的结点都需要和右边的一个结点相连形成一条路，所以这条边经过
