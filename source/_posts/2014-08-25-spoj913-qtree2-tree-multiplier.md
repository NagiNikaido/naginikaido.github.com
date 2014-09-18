---
layout: post
title: 'spoj913 QTREE2 树上倍增'
date: 2014-08-25 11:45
tags: [spoj]
---
由于没有修改操作，直接上倍增。
如果有修改的话，应该是LCT写起来方便一些。
这里关键是KTH的询问。对于a,b，求出它们的lca，可以知道第k个点一定在lca的左侧或者右侧，这个通过计算就可以得到。之后再做一次倍增就可以求出这个点了。

<!--more-->
```c++
#include <cstdio>
#include <cstring>
#include <algorithm>

using namespace std;

const int MaxN = 10010;

struct Node{
	int v,c;
	Node *nxt;
}pool[MaxN << 1],*tail,*g[MaxN];

inline void make_edge(int u,int v,int c){
	tail->v=v;tail->c=c;tail->nxt=g[u];g[u]=tail++;
	tail->v=u;tail->c=c;tail->nxt=g[v];g[v]=tail++;
}

int n;
int u[MaxN],v[MaxN],c[MaxN],h[MaxN];
int fa[MaxN][20],d[MaxN][20];
int mapp[MaxN << 1],last;

void dfs(){
	static int q[MaxN],l,r,u;
	for(h[q[l=r=1]=1]=1;l<=r;l++)
		for(Node *p=g[u=q[l]];p;p=p->nxt) if(!h[p->v])
			h[p->v]=h[u]+1,fa[p->v][0]=u,d[p->v][0]=p->c,q[++r]=p->v;
	for(int j=1;j<20;j++)
		for(int i=1;i<=n;i++)
			fa[i][j]=fa[fa[i][j-1]][j-1],
			d[i][j]=d[i][j-1]+d[fa[i][j-1]][j-1];
}

char op[10];

int queryDist(int u,int v){
	int res=0;
	if(h[u]<h[v]) swap(u,v);
	for(int i=19;i>=0 && h[u]>h[v];i--) if(h[fa[u][i]]>=h[v]) res+=d[u][i],u=fa[u][i];
	if(u==v) return res;
	for(int i=19;i>=0;i--) if(fa[u][i]!=fa[v][i]) res+=d[u][i]+d[v][i],u=fa[u][i],v=fa[v][i];
	return res+d[u][0]+d[v][0];
}

int queryLca(int u,int v){
	if(h[u]<h[v]) swap(u,v);
	for(int i=19;i>=0 && h[u]>h[v];i--) if(h[fa[u][i]]>=h[v]) u=fa[u][i];
	if(u==v) return u;
	for(int i=19;i>=0;i--) if(fa[u][i]!=fa[v][i]) u=fa[u][i],v=fa[v][i];
	return fa[u][0];
}
int queryKth(int u,int v,int k){
	int lca=queryLca(u,v),len,lu;
	len=h[u]+h[v]-h[lca]-h[lca]+1;
	lu=h[u]-h[lca]+1;
	if(k>lu) swap(u,v),lu=len-k+1;
	else lu=k;
	lu--;
	for(int i=0;i<20;i++) if(lu&(1<<i)) u=fa[u][i];
	return mapp[u];
}
int main()
{
	int T;
	for(scanf("%d",&T);T--;){
		memset(g,0,sizeof(g));
		memset(h,0,sizeof(h));
		memset(fa,0,sizeof(fa));
		memset(d,0,sizeof(d));
		tail=pool;
		scanf("%d",&n);
		for(int i=1;i<n;i++)
			scanf("%d%d%d",&u[i],&v[i],&c[i]),
			mapp[++last]=u[i],mapp[++last]=v[i];
		sort(mapp+1,mapp+last+1);
		last=unique(mapp+1,mapp+last+1)-mapp-1;
		for(int i=1;i<n;i++)
			u[i]=lower_bound(mapp+1,mapp+last+1,u[i])-mapp,
			v[i]=lower_bound(mapp+1,mapp+last+1,v[i])-mapp,
			make_edge(u[i],v[i],c[i]);
		dfs();
		for(scanf("%s",op);op[1]!='O';scanf("%s",op)){
			int a,b,k;
			scanf("%d%d",&a,&b);
			a=lower_bound(mapp+1,mapp+last+1,a)-mapp;
			b=lower_bound(mapp+1,mapp+last+1,b)-mapp;
			if(op[0]=='D') printf("%d\n",queryDist(a,b));
			else{
				scanf("%d",&k);
				printf("%d\n",queryKth(a,b,k));
			}
		}
	}
	return 0;
}
```
