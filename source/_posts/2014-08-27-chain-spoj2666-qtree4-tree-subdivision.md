---
layout: post
title: 'spoj2666 QTREE4 树链剖分'
date: 2014-08-27 09:48
tags: [数据结构,spoj]
---
T了五六发之后发现了问题。我把树链剖分之后的总重链条数当成 $O(log_2 n)$ 了，实际上是 $O(n)$ 的，而我原来的做法是，对于每次询问 $O(n)$ 扫一遍。这样显然会T得不成样子。至于zjoi2007 hide那题，只能认为是数据出弱了。
为了方便写出了每条重链一棵线段树的形式，这样就用不着额外的合并答案了，单点修改也可以自底向上地修改了。
以及这里用`priority_queue`+时间戳的方式完全没有问题，根本不必使用蛋疼的`multiset<int>`。
~~TODO：用边分治A之。~~
<!--more-->

```c++
#include <cstdio>
#include <cstring>
#include <cctype>
#include <algorithm>
#include <queue>
#include <utility>

using namespace std;

const int MaxN = 200010;
const int INF = -500000000;

typedef pair<int,int> PII;
typedef pair<PII,int> PPI;

struct Edge{
	int v,c;
	Edge *nxt;
}pool_e[MaxN],*g[MaxN],*tail_e=pool_e;


int n,m;
int num[MaxN],top[MaxN],fa[MaxN],h[MaxN],d[MaxN],col[MaxN];
int nxt[MaxN],size[MaxN],cnt,inv[MaxN],L[MaxN],R[MaxN];
int Time[MaxN],mapp[MaxN];
int f[MaxN][2];

priority_queue<PPI> heaps[MaxN],Ans;

void make_edge(int u,int v,int c){
	tail_e->v=v;tail_e->c=c;tail_e->nxt=g[u];g[u]=tail_e++;
	tail_e->v=u;tail_e->c=c;tail_e->nxt=g[v];g[v]=tail_e++;
}

void update(int u){
	priority_queue<PPI> &q=heaps[u];
	PPI a;
	f[u][0]=f[u][1]=INF;
	while(!q.empty() && q.top().second!=Time[q.top().first.second]) q.pop();
	if(q.empty()) return ;
	f[u][0]=(a=q.top()).first.first;q.pop();
	while(!q.empty() && q.top().second!=Time[q.top().first.second]) q.pop();
	if(!q.empty()) f[u][1]=q.top().first.first;
	q.push(a);
}

void dfs(){
	static int q[MaxN],l,r,u;
	for(h[q[l=r=1]=1]=1;l<=r;l++)
		for(Edge *p=g[u=q[l]];p;p=p->nxt) if(!h[p->v])
			fa[p->v]=u,d[p->v]=d[u]+p->c,h[q[++r]=p->v]=h[u]+1;
	for(int i=r;i;i--){
		size[u=q[i]]=1;
		for(Edge *p=g[u];p;p=p->nxt) if(fa[p->v]==u)
			size[u]+=size[size[p->v]>size[nxt[u]] ? nxt[u]=p->v : p->v];
	}
	for(top[q[l=r=1]=1]=1;l<=r;l++){
		inv[num[u=q[l]]=++cnt]=u;
		if(nxt[u]){
			top[q[l--]=nxt[u]]=top[u];
			for(Edge *p=g[u];p;p=p->nxt) if(fa[p->v]==u && nxt[u]!=p->v)
				top[q[++r]=p->v]=p->v;
		}
	}
	for(int i=1;i<=n;i++) if(top[i]==i) L[mapp[i]=++cnt]=num[i];
	for(int i=1;i<=n;i++){
		R[mapp[top[i]]]=max(num[i],R[mapp[top[i]]]);
		update(i);
	}
}

struct Node{
	int l,r,mid;
	int maxl,maxr,maxans;
	Node *lc,*rc,*fa;
	void update(){
		if(l==r){
			maxl=maxr=f[inv[l]][0];maxans=col[inv[l]] ? f[inv[l]][0] : INF;
			if(f[inv[l]][1]!=INF) maxans=max(maxans,f[inv[l]][0]+f[inv[l]][1]);
		}
		else{
			maxl=max(lc->maxl,rc->maxl!=INF ? d[inv[mid+1]]-d[inv[l]]+rc->maxl : INF);
			maxr=max(rc->maxr,lc->maxr!=INF ? d[inv[r]]-d[inv[mid]]+lc->maxr : INF);
			maxans=max(lc->maxans,rc->maxans);
			if(lc->maxr!=INF && rc->maxr!=INF) maxans=max(maxans,lc->maxr+rc->maxl+d[inv[mid+1]]-d[inv[mid]]);
		}
	}
}pool[MaxN<<1],*tail=pool,*pos[MaxN],*rt[MaxN];

void build(int l,int r,Node *&rt){
	(rt=tail++)->l=l;rt->r=r;rt->mid=l+r >> 1;
	if(l==r) pos[l]=rt;
	else{
		build(l,rt->mid,rt->lc);rt->lc->fa=rt;
		build(rt->mid+1,r,rt->rc);rt->rc->fa=rt;
	}
	rt->update();
}

void getAns(){
	int res=INF;
	while(!Ans.empty() && Ans.top().second!=Time[Ans.top().first.second]) Ans.pop();
	if(!Ans.empty()) res=Ans.top().first.first;
	if(res==INF) puts("They have disappeared.");
	else printf("%d\n",res);
}

void modify(int u){
	Time[u]++;
	if(col[u]=!col[u]) heaps[u].push(PPI(PII(0,u),Time[u]));
	for(int v;u;u=fa[top[u]]){
		update(u);
		for(Node *p=pos[num[u]];p;p=p->fa) p->update();
		++Time[mapp[top[u]]];
		Ans.push(PPI(PII(rt[mapp[top[u]]]->maxans,mapp[top[u]]),Time[mapp[top[u]]]));
		if((v=fa[top[u]]) && rt[mapp[top[u]]]->maxl!=INF)
			heaps[v].push(PPI(PII(rt[mapp[top[u]]]->maxl+d[top[u]]-d[v],mapp[top[u]]),Time[mapp[top[u]]]));
	}
}

char buf[3500000],*c_ptr;
void initInput(){
	fread(buf,sizeof(char),sizeof(buf),stdin);
	c_ptr=buf;
}
int getInt(){
	int res=0,flag=1;
	for(;*c_ptr && !isdigit(*c_ptr) && *c_ptr!='-';c_ptr++);
	if(*c_ptr=='-') flag=-1,c_ptr++;
	for(;isdigit(*c_ptr);c_ptr++) res=res*10+*c_ptr-'0';
	return res*flag;
}
int getAlpha(){
	for(;*c_ptr && !isalpha(*c_ptr);c_ptr++);
	return *(c_ptr++);
}

int main()
{
	initInput();
	n=getInt();
	for(int i=1;i<n;i++){
		int u,v,c;
		u=getInt(),v=getInt(),c=getInt();
		make_edge(u,v,c);
	}
	dfs();
	for(int i=n+1;i<=cnt;i++) build(L[i],R[i],rt[i]);
	for(int i=1;i<=n;i++) modify(i);
	m=getInt();
	for(int i=1;i<=m;i++){
		int a=getAlpha(),b;
		if(a=='A') getAns();
		else modify(b=getInt());
	}
	return 0;
}
```
