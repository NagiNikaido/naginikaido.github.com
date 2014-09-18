---
layout: post
title: 'noi2003 editor 函数式treap'
date: 2014-08-23 13:12
tags: [数据结构,字符串处理,noi,bzoj]
---
函数式treap的版本。不加gc就会MLE。

值得注意的是，这里为了能直接依据一个字符串获得一个Treap，需要使用一些手段。原因是treap需要在每个节点上附加一个随机权值weight，这为不使用旋转操作直接得到treap设下了障碍。而一个一个字符插入又是极不经济的，因此在`merge(Node *a,Node *b)`中使用下面的方法：

roll出一个在$[ 0 , size(a)+size(b) )$内的整数$p$。如果$p<size(a)$,就将b挂到a下面，否则将a挂到b下面。

这是CLJ提到过的方法，这样可以避免使用weight，就可以直接使用和splay一样的`init()`了。

如果不使用gc的话会爆空间。这里我使用了[引用计数](http://baike.baidu.com/view/4163962.htm?fr=aladdin)的方法，就是说对于每一个节点，用`cnt`记录它是多少个节点的后继。如果cnt=0，就说明这个点是可以被回收的。由于这里的`Insert()`和`Remove()`调用次数较少，一共最多只有4000次，因此我只在`Get()`里使用了`relax()`操作。

<!--more-->
下面是不使用gc的版本。尽管跑得比splay版本快，但由于会MLE只能用来看看。

```c++ editor_treap_without_gc.cpp
#include <cstdio>
#include <cstring>
#include <cstdlib>

using namespace std;

const int RANDSEED = 42;
const int MaxN = 1<<23;

struct Node{
	char ch;
	int size;
	Node *lc,*rc;
	inline int lc_size(){return lc ? lc->size : 0;}
	inline int rc_size(){return rc ? rc->size : 0;}
}pool[MaxN],*tail=pool,*rt;

inline Node *newNode(char ch,Node *lc=0,Node *rc=0){
	tail->ch=ch;tail->size=1+((tail->lc=lc) ? lc->size : 0)+((tail->rc=rc) ? rc->size : 0);
	return tail++;
}

Node *init(char *st,int l,int r){
	return newNode(st[l+r >> 1],
		l<(l+r>>1) ? init(st,l,(l+r>>1)-1) : 0
		,r>(l+r>>1) ? init(st,(l+r>>1)+1,r) : 0);
}

inline int getRandom(int first,int second){
	int s=first+second;
	return rand()%s<first;
}
int flag=0;

Node *join(Node *a,Node *b){
	if(!a) return b;
	if(!b) return a;
	if(getRandom(a->size,b->size)) return newNode(a->ch,a->lc,join(a->rc,b));
	else return newNode(b->ch,join(a,b->lc),b->rc);
}

Node *split_l(Node *a,int pos){
	if(!a || !pos) return 0;
	if(a->lc_size()>=pos) return split_l(a->lc,pos);
	else return newNode(a->ch,a->lc,split_l(a->rc,pos-(a->lc ? a->lc->size : 0)-1));
}

Node *split_r(Node *a,int pos){
	if(!a || pos>=a->size) return 0;
	if(a->lc_size()>=pos) return newNode(a->ch,split_r(a->lc,pos),a->rc);
	else return split_r(a->rc,pos-(a->lc ? a->lc->size : 0)-1);
}

void __print(Node *a){
	if(!a) return;
	__print(a->lc);
	putchar(a->ch);
	__print(a->rc);
}
void print(Node *a){__print(a);puts("");}

void debug(Node *a){
	if(!a) return;
	printf("%p : ch=%c size=%d lc=%p rc=%p\n",a,a->ch,a->size,a->lc,a->rc);
	debug(a->lc);
	debug(a->rc);
}

void Insert(int pos,char *st,int len){
	rt=join(join(split_l(rt,pos),init(st,0,len-1)),split_r(rt,pos));
}

void Remove(int pos,int len){
	rt=join(split_l(rt,pos),split_r(rt,pos+len));
}

void Get(int pos,int len){
	print(split_l(split_r(rt,pos),len));
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
	srand(RANDSEED);
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

下面是使用了gc的版本，这下跑得就比splay版本慢了。

```c++ editor_treap_with_gc.cpp
#include <cstdio>
#include <cstring>
#include <cstdlib>

using namespace std;

const int RANDSEED = 42;
const int MaxN = 1<<22;

struct Node{
	char ch;
	int size,cnt;
	Node *lc,*rc;
	inline int lc_size(){return lc ? lc->size : 0;}
	inline int rc_size(){return rc ? rc->size : 0;}
}pool[MaxN],*st[MaxN],*tail=pool,**upper=st,*rt;

inline Node *newNode(char ch,Node *lc=0,Node *rc=0){
	Node *p=(upper==st ? tail++ : *(upper--));
	p->ch=ch;p->size=1+((p->lc=lc) ? lc->size : 0)+((p->rc=rc) ? rc->size : 0);
	p->cnt=0;
	if(lc) lc->cnt++;
	if(rc) rc->cnt++;
	return p;
}
inline void recycle(Node *rt){*++upper=rt;}

void delTree(Node *rt){
	if(!rt) return;
	if(!--rt->cnt) delTree(rt->lc),delTree(rt->rc),recycle(rt);
}

void relax(Node *rt){
	if(!rt || rt->cnt) return;
	delTree(rt->lc);delTree(rt->rc);recycle(rt);
}

Node *init(char *st,int l,int r){
	return newNode(st[l+r >> 1],
		l<(l+r>>1) ? init(st,l,(l+r>>1)-1) : 0
		,r>(l+r>>1) ? init(st,(l+r>>1)+1,r) : 0);
}

inline int getRandom(int first,int second){
	return rand()%(first+second)<first;
}

Node *join(Node *a,Node *b){
	if(!a) return b;
	if(!b) return a;
	if(getRandom(a->size,b->size)) return newNode(a->ch,a->lc,join(a->rc,b));
	else return newNode(b->ch,join(a,b->lc),b->rc);
}

Node *split_l(Node *a,int pos){
	if(!a || !pos) return 0;
	if(a->lc_size()>=pos) return split_l(a->lc,pos);
	else return newNode(a->ch,a->lc,split_l(a->rc,pos-(a->lc ? a->lc->size : 0)-1));
}

Node *split_r(Node *a,int pos){
	if(!a || pos>=a->size) return 0;
	if(a->lc_size()>=pos) return newNode(a->ch,split_r(a->lc,pos),a->rc);
	else return split_r(a->rc,pos-(a->lc ? a->lc->size : 0)-1);
}

void __print(Node *a){
	if(!a) return;__print(a->lc);
	putchar(a->ch);__print(a->rc);
}
void print(Node *a){__print(a);puts("");}

void Insert(int pos,char *st,int len){
	rt=join(join(split_l(rt,pos),init(st,0,len-1)),split_r(rt,pos));
}

void Remove(int pos,int len){
	rt=join(split_l(rt,pos),split_r(rt,pos+len));
}

void Get(int pos,int len){
	Node *p=split_r(rt,pos),*q=split_l(p,len);
	print(q);relax(p);relax(q);
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
	srand(RANDSEED);
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
