---
layout: post
title: 'spoj2939 QTREE5 点分治'
date: 2014-08-29 15:50
tags: [数据结构,树分治,spoj]
---
树的点分治，1y，总时间16.22s。
由于这一题查询时的起始点已经给定，因此可以用树的点分治解决。
点分治的主要过程是在当前子树中用dfs求出重心，把重心挖掉，分成若干个子树递归进行。可以证明递归层数不会超过$O(log_2n)$。
对于每一个重心，记录父亲重心的编号和子树中的所有点到自己的距离。这里如果是黑点的话，距离为INF；如果是白点的话，距离可以通过之前的dfs预处理出来。这里对于距离的维护可以用堆或者`multiset<int>`来实现。
可以看到，这样单次修改和查询的时间复杂度都是$O(log_2^2 n)$的，因此总时间复杂度就是$O(n log_2 n+q log_2^2 n)$。
点分治比边分治好写多了。

<!--more-->

```c++
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <set>

using namespace std;

const int MaxN = 100010;
const int INF = 1000000;

struct Node{
	int v;
	Node *nxt;
}pool[MaxN << 1],*tail=pool,*g[MaxN];

int n,m;

struct Package{
	int u,dep,fa;
	multiset<int> q;
}t[MaxN];

int del[MaxN],q[MaxN],size[MaxN],fa[MaxN],flag[MaxN],cur,col[MaxN];
int stein[MaxN],gate;
int el[MaxN][30];

inline void make_edge(int u,int v){
	tail->v=v;tail->nxt=g[u];g[u]=tail++;
	tail->v=u;tail->nxt=g[v];g[v]=tail++;
}

int dfs(int u,int now,int S){
	int l,r,v,dep=t[now].dep-1;
	for(flag[q[l=r=1]=u]=++cur,el[u][dep]=1,fa[u]=0;l<=r;l++)
		for(Node *p=g[q[l]];p;p=p->nxt) if(!del[p->v] && flag[p->v]!=cur)
			flag[p->v]=cur,el[q[++r]=p->v][dep]=el[fa[p->v]=q[l]][dep]+1;
	for(int i=r;i;i--){
		size[q[i]]=1;stein[q[i]]=now;
		for(Node *p=g[q[i]];p;p=p->nxt) if(flag[p->v]==cur && fa[p->v]==q[i])
			size[q[i]]+=size[p->v];
	}
	for(;;){
		int F=0;
		for(Node *p=g[u];p;p=p->nxt) if(flag[p->v]==cur && fa[p->v]==u && size[p->v]>(S>>1)){
			F=1,u=p->v;break;
		}
		if(!F) break;
	}
	return u;
}

void split(int u,int dep,int S){
	int now=++gate;t[now].dep=dep;
	if(S==1){el[u][dep-1]=1;t[stein[u]=now].u=u;t[now].q.insert(INF);return ;}
	t[now].u=dfs(u,now,S);del[t[now].u]=1;
	for(int i=1;i<=S;i++) t[now].q.insert(INF);
	for(Node *p=g[t[now].u];p;p=p->nxt) if(!del[p->v])
		t[gate+1].fa=now,split(p->v,dep+1,size[p->v]>size[t[now].u] ? S-size[t[now].u] : size[p->v]);
}

void modify(int u){
	col[u]=!col[u];
	for(int p=stein[u];p;p=t[p].fa){
		t[p].q.erase(t[p].q.find(col[u] ? INF : el[u][t[p].dep]));
		t[p].q.insert(col[u] ? el[u][t[p].dep] : INF);
	}
}

void getAns(int u){
	int res=INF;
	for(int p=stein[u];p;p=t[p].fa){
		res=min(res,el[u][t[p].dep]+*t[p].q.begin());
	}
	printf("%d\n",res>=INF ? -1 : res);
}
int main()
{
	scanf("%d",&n);
	for(int i=1;i<n;i++){
		int u,v;scanf("%d%d",&u,&v);
		make_edge(u,v);
	}
	split(1,1,n);
	scanf("%d",&m);
	for(int i=1;i<=m;i++){
		int op,u;
		scanf("%d%d",&op,&u);
		if(!op) modify(u);
		else getAns(u);
	}
}
```
