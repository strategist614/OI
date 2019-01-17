####<center> Can you answer on these queries III<center>
* **题目**
[CH4301](http://contest-hunter.org:83/contest/0x40%E3%80%8C%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E8%BF%9B%E9%98%B6%E3%80%8D%E4%BE%8B%E9%A2%98/4301%20Can%20you%20answer%20on%20these%20queries%20III)
* **题意**
  * 1 操作 查询区间$[x,y]$最大连续子段和
  * 0 操作 单点修改
* **分析**
  $0$操作单点修改板子
  $1$操作区间操作，首先分析区间可加性，最大连续子段和满足区间可加性。与普通求区间和不同，要对区间和进行判断，找到最大的和。直接维护区间最大连续子段和是不行的，数据中出现负数，在两个区间合并时，不只是简单的$tree[rt].dat = max(tree[rt<<1].dat,tree[rt<<1|1].dat)$,而是两个区间交界的部分形成的子段可能的和会最大,那么引入两个还需要维护的值，紧靠左端的最大连续和，紧靠右段的最大连续和。现在的答案就是$tree[rt].dat = max(max(tree[rt<<1].dat,tree[rt<<1|1].dat),tree[rt<<1].rmax+tree[rt<<1|1].lmax)$.想要维护这两个值，同样还是由这个点的左右两个子节点的$lmax$贡献答案，但是右子节点的$lmax$要加上左子节点的$sum$(即区间和)，因为区间是连续的。所以还要在维护一个$sum$区间和.
  在执行$ask$操作时要注意：需要先更新左右子节点区间记为$a,b$，答案记为$ans$,那么先更新左子节点区间和$ans.sum = ask(rt<<1,L,R)$,右子节点同理。再累计左右子节点的$lmax,rmax$,以$lamx$为例，当$L > mid$ 时，只递归右子节点，答案更新为$ans.lmax = b.lmax$,当$L {\leq} mid$ 左右都会递归，答案更新为$ans.lmax = max(a.lmax,a.sum+b.lmax)$.最后$ans.dat = max(max(a.dat,b.dat),a.rmax+b.lmax)$,就是最终答案。
```C++{.line-numbers}
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 5e5+5;
const int INF = 0x3fffffff;
int a[N];
struct segment_tree
{
  	int l;
  	int r;
    int dat;//区间最大连续子段和
    int lmax;//紧靠左端的最大连续和
    int rmax;//紧靠右段的最大连续和
    int sum;//区间和
}tree[N<<2];
void pushup(int rt)
{
   tree[rt].sum = tree[rt<<1].sum + tree[rt<<1|1].sum;
   tree[rt].lmax = max(tree[rt<<1].lmax,tree[rt<<1].sum+tree[rt<<1|1].lmax);
   tree[rt].rmax = max(tree[rt<<1|1].rmax,tree[rt<<1|1].sum+tree[rt<<1].rmax);
   tree[rt].dat = max(max(tree[rt<<1].dat,tree[rt<<1|1].dat),tree[rt<<1].rmax+tree[rt<<1|1].lmax);
}
void build(int rt,int l,int r)
{
	tree[rt].l = l;
	tree[rt].r = r;
	if(l == r)
	{
		tree[rt].dat = tree[rt].rmax = tree[rt].lmax = tree[rt].sum = a[l];
		return ;
	}
	int mid = (l+r)>>1;
	build(rt<<1,l,mid);
	build(rt<<1|1,mid+1,r);
	pushup(rt);
}
void update(int rt,int x,int val)
{
	int l = tree[rt].l;
	int r = tree[rt].r;
	if(l == r) {tree[rt].dat = tree[rt].sum = tree[rt].lmax = tree[rt].rmax = val;return;}
	int mid = (l+r)>>1;
	if(x <= mid) update(rt<<1,x,val);
	else update(rt<<1|1,x,val);
	pushup(rt);
}
segment_tree ask(int rt,int L,int R)
{
	int l = tree[rt].l;
	int r = tree[rt].r;
	if(L <= l && r <= R) return tree[rt];
	segment_tree a,b,ans;
	a.sum = a.dat = a.lmax = a.rmax = b.sum = b.dat = b.lmax = b.rmax = -INF;
    ans.sum = 0;
	int mid = (l+r)>>1;
	if(L <= mid)
	{
		a = ask(rt<<1,L,R);
		ans.sum += a.sum;
	}
	if(R > mid)
	{
		b = ask(rt<<1|1,L,R);
		ans.sum += b.sum;
	}
	if(L > mid)
		ans.lmax = b.lmax;
	else ans.lmax = max(a.lmax,a.sum+b.lmax);
	if(R <= mid)
		ans.rmax = a.rmax;
	else ans.rmax = max(b.rmax,b.sum+a.rmax);
	ans.dat = max(max(a.dat,b.dat),a.rmax+b.lmax);
	return ans;
}
int main ()
{
    int n,m;
    cin>>n>>m;
    for(int i = 1;i <= n;i++) cin>>a[i];
    build(1,1,n);
    while(m--)
    {
    	int op,x,y;
    	cin>>op>>x>>y;
    	if(op == 1)
    	{
    	   if(x > y) swap(x,y);
           cout<<ask(1,x,y).dat<<endl;
    	}
    	else
    	{
          update(1,x,y);
    	}
    }
	return 0 ;
}
```
