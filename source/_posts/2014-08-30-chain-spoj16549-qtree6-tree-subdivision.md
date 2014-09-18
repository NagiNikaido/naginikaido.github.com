---
layout: post
title: 'spoj16549 QTREE6 树链剖分'
date: 2014-08-30 13:31
tags: [数据结构,spoj]
---
 树链剖分+树状数组。5.62s。
 想法有些类似ch #51 falsita那题，也是在节点上存储子树的信息。
 树链剖分之后，在一条链中开三个`map<int,int>`，分别用来存储链上所有黑点及深度，所有白点及深度和所有点及深度，并各加入哨兵节点。之后用两个树状数组维护链中每个节点与子树中多少个黑色/白色节点相连。
 这里用`f[color][u]`来表示以u为根的子树中，与u相连的，颜色为color的点的个数。
 在变色之后，可以看到除了u本身以外，被影响到的点是从u的父亲开始，一直到u的最深异色祖先位置的一条链，并且是在这条链的每个点上加或减同一个值，这显然是可以用树状数组来做的。 注意，这里不只是到最深同色祖先，最深同色祖先的父亲也会被影响到。
 查询就是直接找到u的最深同色祖先v，输出`f[col[u]][v]`。
这里求最深同色异色祖先是在map里二分，因此单次修改和查询的时间复杂度都是$O(log_2^2n)$。总复杂度是$O(nlog_2n+mlog_2^2n)$。

<!--more-->
```c++
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <map>

using namespace std;

const int MaxN = 100010;

struct Node{
	int v;
	Node *nxt;
}pool[MaxN << 1],*tail=pool,*g[MaxN];

inline void make_edge(int u,int v){
	tail->v=v;tail->nxt=g[u];g[u]=tail++;
	tail->v=u;tail->nxt=g[v];g[v]=tail++;
}

int n,m;
map<int,int> s[MaxN][3];
int f[2][MaxN];

int top[MaxN],size[MaxN],fa[MaxN],h[MaxN],num[MaxN],nxt[MaxN],mapp[MaxN],cnt;
int col[MaxN];

void dfs(){
	static int q[MaxN],l,r,u;
	for(h[q[l=r=1]=1]=1;l<=r;l++)
		for(Node *p=g[u=q[l]];p;p=p->nxt) if(!h[p->v])
			h[q[++r]=p->v]=h[fa[p->v]=u]+1;
	for(int i=r;i;i--){
		size[u=q[i]]=1;
		for(Node *p=g[u];p;p=p->nxt) if(fa[p->v]==u)
			size[u]+=size[size[p->v]>size[nxt[u]] ? nxt[u]=p->v : p->v];
	}
	for(top[q[l=r=1]=1]=1;l<=r;l++){
		u=q[l];mapp[s[top[u]][0][h[u]]=s[top[u]][2][h[u]]=num[u]=++cnt]=u;
		if(top[u]==u) s[top[u]][0][h[u]-1]=s[top[u]][1][h[u]-1]=s[top[u]][2][h[u]-1]=0;
		if(!nxt[u]) continue;
		top[q[l--]=nxt[u]]=top[u];
		for(Node *p=g[u];p;p=p->nxt) if(fa[p->v]==u && p->v!=nxt[u])
			top[q[++r]=p->v]=p->v;
	}
}
inline void modify(int F,int p,int dt){for(;p && p<=n;p+=p&(-p)) f[F][p]+=dt;}
inline int query(int F,int p){int res=0;for(;p;p^=p&(-p)) res+=f[F][p];return res;}

#define MODIFY(COL,L,R,DT) modify(COL,L,DT),modify(COL,(R)+1,-(DT))

void Modify(int u){
	int f1=query(col[u],num[u]),f2=query(!col[u],num[u]),dt,cu=col[u];
	MODIFY(cu,num[u],num[u],-1);MODIFY(!cu,num[u],num[u],1);
	s[top[u]][cu].erase(h[u]);s[top[u]][!cu][h[u]]=num[u];
	col[u]=!col[u];if(!fa[u]) return;
	
	if(col[fa[u]]==cu) dt=-f1,MODIFY(!cu,num[fa[u]],num[fa[u]],f2+1);
	else dt=f2+1,MODIFY(cu,num[fa[u]],num[fa[u]],-f1),cu=!cu;
	for(u=fa[u];u;u=fa[top[u]]){
		map<int,int> :: iterator p=--s[top[u]][!cu].upper_bound(h[u]);
		int v=mapp[s[top[u]][2][p->first+1]],w=s[top[u]][2][p->first];
		if(h[v]<=h[u]) MODIFY(cu,num[v],num[u],dt);
		if(w) MODIFY(cu,w,w,dt);
		if(v!=top[u]) break;
	}
}
void getAns(int u){
	int res=u;
	for(;u;u=fa[top[u]]){
		map<int,int> :: iterator p=--s[top[u]][!col[res]].upper_bound(h[u]);
		int v=mapp[s[top[u]][2][p->first+1]];
		if(h[v]<=h[u]) res=v;
		if(v!=top[u]) break;
	}
	printf("%d\n",query(col[res],num[res]));
}
int main()
{
	scanf("%d",&n);
	for(int i=1;i<n;i++){
		int u,v;
		scanf("%d%d",&u,&v);
		make_edge(u,v);
	}
	dfs();
	for(int i=1;i<=n;i++) modify(0,num[i],size[i]-size[mapp[num[i]-1]]);
	scanf("%d",&m);
	for(int i=1;i<=m;i++){
		int op,u;scanf("%d%d",&op,&u);
		if(op) Modify(u);
		else getAns(u);
	}
	return 0;
}
```
