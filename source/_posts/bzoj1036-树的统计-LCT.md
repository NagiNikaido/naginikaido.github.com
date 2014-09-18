title: bzoj1036 树的统计 LCT
date: 2014-09-16 22:38:15
tags: [省选题,数据结构]
---

1y的LCT。对make_root操作进行了改进。

<!--more-->
```c++
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <vector>

using namespace std;

const int MaxN = 30010;

inline int ckmax(int &a,int b){return a<b ? a=b,1 : 0;}

struct Node{
	int w,maxw,sumw,rev;
	Node *lc,*rc,*fa,*path_fa;
	void update(){
		maxw=w;sumw=w;
		lc ? ckmax(maxw,lc->maxw),sumw+=lc->sumw : 0;
		rc ? ckmax(maxw,rc->maxw),sumw+=rc->sumw : 0;
	}
	void sink(){if(rev) lc ? lc->flip(),0 : 0, rc ? rc->flip(),0 : 0, rev=0;}
	void flip(){swap(lc,rc);rev^=1;}
	void modify(int t){w=t;update();}
	#define ROT(a,b)\
		Node *p=a;\
		p->b=((a=p->b) ? a->fa=this : this);\
		(p->fa=fa) ? (fa->lc==this ? fa->lc=p : fa->rc=p) : 0;\
		fa=p;swap(path_fa,p->path_fa);update();
	void zig(){ROT(lc,rc);}
	void zag(){ROT(rc,lc);}
}pool[MaxN],*pos[MaxN],*tail=pool;

int n,m,weight[MaxN];
vector<int> g[MaxN];

void splay(Node *rt){
	rt->sink();
	while(rt->fa){
		if(rt->fa->fa){
			rt->fa->fa->sink();rt->fa->sink();rt->sink();
			if(rt->fa==rt->fa->fa->lc){
				if(rt==rt->fa->lc) rt->fa->fa->zig(),rt->fa->zig();
				else rt->fa->zag(),rt->fa->zig();
			}
			else{
				if(rt==rt->fa->lc) rt->fa->zig(),rt->fa->zag();
				else rt->fa->fa->zag(),rt->fa->zag();
			}
		}
		else{
			rt->fa->sink();rt->sink();
			if(rt==rt->fa->lc) rt->fa->zig();
			else rt->fa->zag();
		}
	}
	rt->update();
}

void expose(int u){
	Node *p=pos[u],*q;
	splay(p);
	if(q=p->rc) q->fa=0,q->path_fa=p;
	p->rc=0;p->update();
	while(q=p->path_fa){
		splay(q);
		if(q->rc) q->rc->fa=0,q->rc->path_fa=q;
		(q->rc=p)->fa=q;q->update();
		splay(p);
	}
}

void make_root(int u){
	Node *p=pos[u];
	expose(u);p->flip();expose(u);
}

int query(int u,int v,int type){
	make_root(u);expose(v);
	return type ? pos[v]->sumw : pos[v]->maxw;
}

void modify(int u,int t){
	expose(u);pos[u]->modify(t);
}

void dfs(int u){
	(pos[u]=tail++)->w=weight[u];pos[u]->update();
	for(int i=0;i<g[u].size();i++) if(!pos[g[u][i]])
		dfs(g[u][i]),pos[g[u][i]]->path_fa=pos[u];
}

char opt[10];

int main()
{
	scanf("%d",&n);
	for(int i=1;i<n;i++){
		int u,v;scanf("%d%d",&u,&v);
		g[u].push_back(v);
		g[v].push_back(u);
	}
	for(int i=1;i<=n;i++) scanf("%d",&weight[i]);
	dfs(1);scanf("%d",&m);
	for(int i=1;i<=m;i++){
		int x,y;
		scanf("%s%d%d",opt,&x,&y);
		if(opt[1]=='H') modify(x,y);
		else printf("%d\n",query(x,y,opt[1]=='S'));
	}
	return 0;
}
```
