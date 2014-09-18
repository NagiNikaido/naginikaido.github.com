title: bzoj1015 火星人prefix splay 字符串hash
date: 2014-09-07 23:34:17
tags: [数据结构,字符串处理,省选题,bzoj]
---

用splay维护字符串和hash值。对于Q操作二分lcp长度即可。时间复杂度$O(n log_2^2 len)$，len是字符串的长度。
由于我的写法常数极大，跑了10s左右，差点T掉。

<!-- more -->

```c++
#include <cstdio>
#include <cstring>

using namespace std;

typedef long long LL;

const int MaxN =  100010;
const LL BASE  =  29;
const LL MOD   =  1000000007LL;

LL pow[MaxN];

struct Node{
	char a;
	int size;
	LL hsh;
	Node *lc,*rc,*fa;
	#define SIZE(p) (p ? p->size : 0)
	#define HSH(p) (p ? p->hsh : 0)
	void update(){
		hsh=(HSH(lc)+(a-'a'+1)*pow[SIZE(lc)]+HSH(rc)*pow[SIZE(lc)+1])%MOD;
		size=1+SIZE(lc)+SIZE(rc);
	}
	#define ROT(a,b)\
		Node *p=a;\
		p->b=((a=p->b) ? a->fa=this : this);\
		if(p->fa=fa) fa->lc==this ? fa->lc=p : fa->rc=p;\
		fa=p;update();p->update();
	void zig(){ROT(lc,rc);}
	void zag(){ROT(rc,lc);}
}pool[MaxN],*tail=pool,*rt;

void splay(Node *p,Node *goal=0){
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
	p->update();
}

inline Node *newNode(char a,Node *lc=0,Node *rc=0){
	(tail->lc=lc) ? lc->fa=tail : 0;
	(tail->rc=rc) ? rc->fa=tail : 0;
	tail->a=a;tail->update();
	return tail++;
}

Node *init(char *a,int l,int r){
	return l>r ? 0 : newNode(a[l+r >> 1],init(a,l,(l+r >>1)-1),init(a,(l+r >> 1)+1,r));
}

Node *select(Node *rt,int s){
	if(!s || !rt) return 0;
	if(s>rt->size) return (Node *)-1;
	while(1){
		if(rt->lc){
			if(s<=rt->lc->size) {rt=rt->lc;continue;}
			s-=rt->lc->size;
		}
		if(!--s) return rt;
		rt=rt->rc;
	}
}

Node *split(Node *&p,int pos){
	Node *q=select(p,pos);
	if(!q) {q=p,p=0;return q;}
	if(q==(Node *)-1) return 0;
	splay(p=q);if(q=p->rc) q->fa=0;
	p->rc=0;p->update();
	return q;
}

Node *join(Node *p,Node *q){
	if(!p) return q;
	if(!q) return p;
	splay(p=select(p,p->size));
	((p->rc=q)->fa=p)->update();
	return p;
}

void Replace(int pos,char t){
	Node *p=select(rt,pos);
	splay(p);p->a=t;p->update();
	rt=p;
}

void Insert(int pos,char t){
	Node *p=split(rt,pos);
	rt=join(join(rt,newNode(t)),p);
}

LL query(int l,int len){
	Node *p=split(rt,l-1);
	Node *q=split(p,len);
	LL res=p->hsh;
	rt=join(join(rt,p),q);
	return res;
}

int Query(int pos1,int pos2){
	int res=0,l=1,r=rt->size-(pos1>pos2 ? pos1 : pos2)+1;
	while(l<=r){
		int mid=l+r >> 1;
		if(query(pos1,mid)==query(pos2,mid)) l=mid+1,res=mid;
		else r=mid-1;
	}
	return res;
}

char buf[100010];
int n;

int main()
{
	pow[0]=1;
	for(int i=1;i<=100000;i++) pow[i]=pow[i-1]*BASE%MOD;
	scanf("%s%d",buf+1,&n);rt=init(buf,1,strlen(buf+1));
	for(int x,y;n--;){
		scanf("%s",buf);
		switch(buf[0]){
			case 'Q':{
				scanf("%d%d",&x,&y);
				printf("%d\n",Query(x,y));
				break;
			}
			case 'R':{
				scanf("%d%s",&x,buf);
				Replace(x,buf[0]);
				break;
			}
			case 'I':{
				scanf("%d%s",&x,buf);
				Insert(x,buf[0]);
				break;
			}
		}
	}
	return 0;
}
```
