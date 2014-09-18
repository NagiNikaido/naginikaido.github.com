---
layout: post
title: 'noi2003 editor splay'
date: 2014-08-23 10:43
tags: [数据结构,字符串处理,noi,bzoj]
---
splay版本。没有使用getInterval操作，而是直接使用了split和join操作。这样可以避免很多的边界情况，更加好写好想好调。
<!--more-->
```c++
#include <cstdio>
#include <cstring>

using namespace std;

const int MaxN = 1<<22;

struct Node{
	char ch;
	int size;
	Node *lc,*rc,*fa;
	void update(){size=1+(lc ? lc->size : 0)+(rc ? rc->size : 0);}
	void zig(){
		Node *p=lc;
		p->rc=((lc=p->rc) ? lc->fa=this : this);
		if(p->fa=fa) fa->lc==this ? fa->lc=p : fa->rc=p;
		fa=p;update();p->update();
	}
	void zag(){
		Node *p=rc;
		p->lc=((rc=p->lc) ? rc->fa=this : this);
		if(p->fa=fa) fa->lc==this ? fa->lc=p : fa->rc=p;
		fa=p;update();p->update();
	}
}pool[MaxN],*tail=pool,*rt;

inline Node *newNode(char ch){
	tail->ch=ch;tail->size=1;
	tail->lc=tail->rc=tail->fa=0;
	return tail++;
}

inline void splay(Node *&rt,Node *p,Node *goal=0){
	while(p->fa!=goal){
		if(p->fa->fa!=goal){
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
			if(p==p->fa->lc) p->fa->zig();
			else p->fa->zag();
		}
	}
	if(!goal) rt=p;
}

Node *init(char *st,int l,int r){
	Node *p=newNode(st[l+r >> 1]);
	if(l<(l+r>>1)) (p->lc=init(st,l,(l+r>>1)-1))->fa=p;
	if(r>(l+r>>1)) (p->rc=init(st,(l+r>>1)+1,r))->fa=p;
	p->update();
	return p;
}

void __print(Node *rt){
	if(!rt) return;
	__print(rt->lc);
	putchar(rt->ch);
	__print(rt->rc);
}
void print(Node *rt){__print(rt);puts("");}

inline Node *select(Node *&rt,int size){
	if(!rt  || !size) return 0;
	if(size>rt->size) return (Node *)-1;
	Node *p=rt;
	while(1){
		if(p->lc){
			if(size<=p->lc->size) {p=p->lc;continue;}
			size-=p->lc->size;
		}
		if(!--size) return p;
		p=p->rc;
	}
}

inline Node *split(Node *&rt,int size){
	Node *p=select(rt,size),*q;
	if(!p) {p=rt,rt=0;return p;}
	if(p==(Node *)-1) return 0;
	splay(rt,p);
	if(q=rt->rc) q->fa=0;
	rt->rc=0;rt->update();
	return q;
}

inline Node *join(Node *p,Node *q){
	if(!p) return q;
	if(!q) return p;
	Node *t=select(p,p->size);
	splay(p,t);
	((p->rc=q)->fa=p)->update();
	return p;
}

void Insert(int pos,char *st,int len){
	Node *p=split(rt,pos);
	rt=join(rt,join(init(st,0,len-1),p));
}

void Remove(int pos,int len){
	Node *p=split(rt,pos);
	rt=join(rt,split(p,len));
}

void Get(int pos,int len){
	Node *p=split(rt,pos);
	Node *q=split(p,len);
	print(p);rt=join(join(rt,p),q);
}

char op[10],buf[1<<22];

inline void getInput(char *st,int cnt){
	while(cnt){
		*st=getchar();
		if(*st!='\n' && *st!='\r') st++,cnt--;
	}
	*st=0;
}
int main()
{
	int cur=0,t=0,k;
#ifdef __TEST
	freopen("editor.in","r",stdin);
	freopen("editor.out","w",stdout);
#endif
	scanf("%d",&t);
	while(t--){
		scanf("%s",op);
		switch(op[0]){
			case 'M':{scanf("%d",&k);cur=k;break;}
			case 'I':{scanf("%d",&k);getInput(buf,k);Insert(cur,buf,k);break;}
			case 'D':{scanf("%d",&k);Remove(cur,k);break;}
			case 'G':{scanf("%d",&k);Get(cur,k);break;}
			case 'P':{cur--;break;}
			case 'N':{cur++;break;}
			default:;
		}
	}
	return 0;
}
```
