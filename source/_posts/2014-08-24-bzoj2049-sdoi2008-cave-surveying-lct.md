---
layout: post
title: 'bzoj2049 sdoi2008 洞穴勘测 LCT'
date: 2014-08-24 13:55
tags: [数据结构,省选题,bzoj]
---
LCT裸题。
注意点：
 1. `expose()`里的`splay(q)`操作是一定要做的。如果只用`q->sink()`会导致`q`的父亲的标记没有下传，导致问题。
 2. `cut()`里我经过思考，将`expose(u)`换成了`make_root(u)`。如果只用`expose(u)`可能会由于`v`是`u`的祖先节点而删出问题来。尽管测试数据并没有暴露出这样的问题，保险起见我还是改了过来。

<!--more-->
```c++
#include <cstdio>
#include <cstring>
#include <algorithm>

using namespace std;

const int MaxN = 10010;

struct Node{
	int id,size,rev;
	Node *lc,*rc,*fa,*path_fa;
	void update(){size=1+(lc ? lc->size : 0)+(rc ? rc->size : 0);}
	void flip(){rev^=1;swap(lc,rc);}
	void sink(){
		if(rev){
			if(lc) lc->flip();
			if(rc) rc->flip();
			rev=0;
		}
	}
	void zig(){
		Node *p=lc;
		p->rc=((lc=p->rc) ? lc->fa=this : this);
		if(p->fa=fa) fa->lc==this ? fa->lc=p : fa->rc=p;
		fa=p;swap(p->path_fa,path_fa);update();p->update();
	}
	void zag(){
		Node *p=rc;
		p->lc=((rc=p->lc) ? rc->fa=this : this);
		if(p->fa=fa) fa->lc==this ? fa->lc=p : fa->rc=p;
		fa=p;swap(p->path_fa,path_fa);update();p->update();
	}
}pool[MaxN],*tail=pool,*pos[MaxN];

inline Node *newNode(int id){
	tail->id=id;tail->size=1;tail->rev=0;
	return tail++;
}

void splay(Node *p){
	p->sink();
	while(p->fa){
		if(p->fa->fa){
			p->fa->fa->sink();
			p->fa->sink();p->sink();
			if(p->fa==p->fa->fa->lc){
				if(p==p->fa->lc) p->fa->fa->zig(),p->fa->zig();
				else p->fa->zag(),p->fa->zig();
			}
			else{
				if(p==p->fa->lc) p->fa->zig(),p->fa->zag();
				else p->fa->fa->zag(),p->fa->zag();
			}
		}
		else{
			p->fa->sink();p->sink();
			if(p==p->fa->lc) p->fa->zig();
			else p->fa->zag();
		}
	}
	p->update();
}

void expose(int u){
	Node *p=pos[u],*q;
	splay(p);
	if(q=p->rc) q->fa=0,q->path_fa=p;
	p->rc=0;
	while(q=p->path_fa){
		splay(q);
		if(q->rc) q->rc->fa=0,q->rc->path_fa=q;
		(q->rc=p)->fa=q;p->path_fa=0;
		q->update();splay(p);
	}
}

void make_root(int u){
	Node *p=pos[u];
	expose(u);
	if(p->lc) p->lc->fa=0,p->lc->path_fa=p,p->lc->flip();
	p->lc=0;p->update();
}

void cut(int u,int v){
	Node *p=pos[v];
	make_root(u);expose(v);
	if(p->lc) p->lc->fa=0;
	p->lc=0;p->update();
}

void join(int u,int v){
	expose(u);make_root(v);
	pos[v]->path_fa=pos[u];
	expose(v);
}

int getRoot(int u){
	Node *p=pos[u];
	expose(u);
	for(p->sink();p->lc;p=p->lc,p->sink());
	return p->id;
}

int query(int u,int v){
	return getRoot(u)==getRoot(v);
}

int n,m;
char op[10];

int main()
{
#ifdef __TEST
	freopen("cave.in","r",stdin);
	freopen("cave.out","w",stdout);
#endif
	scanf("%d%d",&n,&m);
	for(int i=1;i<=n;i++) pos[i]=newNode(i);
	for(int i=1;i<=m;i++){
		int u,v;
		scanf("%s%d%d",op,&u,&v);
		if(op[0]=='C') join(u,v);
		else if(op[0]=='D') cut(u,v);
		else puts(query(u,v) ? "Yes" : "No");
	}
	return 0;
}
```
