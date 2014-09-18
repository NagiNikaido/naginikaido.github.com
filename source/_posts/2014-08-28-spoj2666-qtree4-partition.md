---
layout: post
title: 'spoj2666 QTREE4 边分治'
date: 2014-08-28 15:08
tags: [数据结构,树分治,spoj]
---
边分治版本。总时长13.35秒（加了读入优化之后是11.65秒），是树链剖分+`priority_queue`的两倍。可见边分治常数之大。
最开始的版本是每条边开两个`priority_queue`+时间戳，TLE得飞起，空间200M+。
之后的版本中把每条边上的两个堆换成手写的，空间150M+，以30s+的总时限过了Hide。放到spoj上还是TLE得飞起。
之后参考了std，发现每块分治的区域中可以只开一个堆。
这里是将当前分治的部分视作一个“黑盒”。假如说这个黑盒与外部的“接口”，即与上一层被剖分的边相连的点为u，那么这个黑盒需要向外部提供的信息有：
    1. 当前区域中到u距离最大的白点；
    2. 当前区域中白点间的最大距离。
而边分治的每一层中，选定中心边以后，最多会分成两个区域，也就是说最多会有两个黑盒。
也就是说，边分治的过程，可以视作是在构造一棵静态的二叉树，并且这棵二叉树是平衡的——递归的层数不会超过$O(log_2n)$层。
那么，在分治的过程中，我们可以将这棵树构造出来。显然，这棵树的每个节点上只要挂一个堆就可以了。并且堆中只需要记编号，用不着记具体的数值，这样可以减掉堆操作一半的时空间常数。
边分治又长~~又臭~~又慢，比树链剖分短了12行，慢了一倍多（hide那一题则是2倍），唯一的优点是比树链剖分好想，比点分治好写。

<!--more-->
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <algorithm>

using namespace std;

const int MaxN = 200010;
const int INF = -(1<<29);

struct Edge{
	int v,c,num;
	Edge *nxt;
}pool[MaxN << 1],*tail=pool,*g[MaxN];

int pos[MaxN][31];
int POOL[MaxN*31],*TAIL=POOL;

int q[MaxN];
int v[MaxN][2],c[MaxN];
int n,m;
int last,deg[MaxN],st[MaxN];
int fa[MaxN],col[MaxN];
int del[MaxN],flag[MaxN],size[MaxN],cur;
int cnt[MaxN];

int stein[MaxN],gate;
int el[MaxN][31],psi[MaxN][31];

struct Heap{
	int *q;
	int n,ed;
	void build(int size){
		q=TAIL;TAIL+=size+10;
		n=size;ed=0;
	}
	void add(int x,int dep){q[pos[x][dep]=++ed]=x;up(ed,dep);}
	void modify(int x,int dep){x=pos[x][dep];up(x,dep);down(x,dep);}
	void up(int x,int dep){
		for(int p=x>>1;p;x=p,p=x>>1){
			if(el[q[p]][dep]>=el[q[x]][dep]) break;
			swap(q[p],q[x]);
			swap(pos[q[p]][dep],pos[q[x]][dep]);
		}
	}
	void down(int x,int dep){
		for(int p=x<<1;p<=ed;x=p,p=x<<1){
			if(p<ed && el[q[p+1]][dep]>el[q[p]][dep]) p++;
			if(el[q[p]][dep]<=el[q[x]][dep]) break;
			swap(q[p],q[x]);
			swap(pos[q[p]][dep],pos[q[x]][dep]);
		}
	}
};

struct Node{
	int l,r,fa,maxt,c,dep;
	Heap opt;
}con[MaxN<<1];

void make_edge(int _u,int _v,int _c,int num){
	tail->v=_v;tail->c=_c;tail->num=num;tail->nxt=g[_u];g[_u]=tail++;
	tail->v=_u;tail->c=_c;tail->num=num;tail->nxt=g[_v];g[_v]=tail++;
	v[num][0]=_u;v[num][1]=_v;c[num]=_c;
}

inline int ckmin(int &a,int b){return b<a ? a=b,1 : 0;}
inline int ckmax(int &a,int b){return b>a ? a=b,1 : 0;}

void update(int num){
	con[num].maxt=INF;
	ckmax(con[num].maxt,el[con[con[num].l].opt.q[1]][con[num].dep+1]+el[con[con[num].r].opt.q[1]][con[num].dep+1]+con[num].c);
	ckmax(con[num].maxt,max(con[con[num].l].maxt,con[con[num].r].maxt));
}

int dfs(int u,int num){
	int l,r,ret=0,mint=0x7fffffff,dep=con[num].dep;
	for(flag[q[l=r=1]=u]=++cur,fa[u]=el[u][dep]=0;l<=r;l++)
		for(Edge *p=g[q[l]];p;p=p->nxt) if(flag[p->v]!=cur && !del[p->num])
			flag[q[++r]=p->v]=cur,el[p->v][dep]=el[q[l]][dep]+p->c,fa[p->v]=q[l];
	for(int i=r;i;i--){
		u=q[i];size[u]=1;
		if(col[u]) el[u][dep]=INF;
		psi[u][dep]=INF;
		stein[u]=num;con[num].opt.add(u,dep);
		for(Edge *p=g[u];p;p=p->nxt) if(flag[p->v]==cur && fa[p->v]==u){
			size[u]+=size[p->v];
			if(ckmin(mint,abs(r-size[p->v]-size[p->v])))
				ret=p->num;
		}
	}
	return ret;
}

void split(int u,int dep,int ts){
	int now=0,ed=++gate;
	con[ed].dep=dep;con[ed].opt.build(ts);
	if(now=dfs(u,ed)){
		if(v[now][0]!=fa[v[now][1]]) swap(v[now][0],v[now][1]);
		del[now]=1;
		con[ed].c=c[now];
		con[con[ed].l=gate+1].fa=ed;split(v[now][0],dep+1,ts-size[v[now][1]]);
		con[con[ed].r=gate+1].fa=ed;split(v[now][1],dep+1,size[v[now][1]]);
		update(ed);
	}
}

void modify(int u){
	col[u]=!col[u];
	for(int p=stein[u];p;p=con[p].fa){
		swap(el[u][con[p].dep],psi[u][con[p].dep]);
		con[p].opt.modify(u,con[p].dep);
		if(p==stein[u]) con[p].maxt=el[u][con[p].dep]; else update(p);
	}
}

void getAns(){
	if(con[1].maxt==INF) puts("They have disappeared.");
	else printf("%d\n",con[1].maxt);
}
int main()
{
	scanf("%d",&n);
	for(int i=1;i<=n;i++) st[i]=i;
	last=n;
	for(int i=1;i<n;i++){
		int u,v,c,x,y;
		scanf("%d%d%d",&u,&v,&c);
		x=st[u],y=st[v];
		if(deg[x]==2) deg[st[u]=++last]=1,make_edge(x,last,0,last-1),col[x=last]=1;
		if(deg[y]==2) deg[st[v]=++last]=1,make_edge(y,last,0,last-1),col[y=last]=1;
		deg[x]++;deg[v]++;make_edge(x,y,c,i);
	}
	split(1,1,last);
	scanf("%d",&m);
	for(int i=1;i<=m;i++){
		char op[2];
		int a;
		scanf("%s",op);
		if(op[0]=='C') scanf("%d",&a),modify(a);
		else getAns();
	}
	return 0;
}
```
