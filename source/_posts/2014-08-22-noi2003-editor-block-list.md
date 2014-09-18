---
layout: post
title: 'noi2003 editor 块状链表'
date: 2014-08-22 14:56
tags: [数据结构,字符串处理,noi,bzoj]
---

正确性没有问题，但T得飞起的块链版本。
<!--more-->
```c++
 #include <cstdio>
 #include <cstring>

 using namespace std;

 const int maxBlockCnt = 4096;
 const int maxBlockSize = 6000;
 const int blockSize = maxBlockSize >> 1;

 struct Block{
 	char st[maxBlockSize+1];
 	int size;
	Block *nxt;
}pool[maxBlockCnt],*st[maxBlockCnt],**upper=st,*tail=pool;
Block none,*None=&none,*rt=None;

inline Block *newBlock(){return upper==st ? tail++ : *--upper;}
inline void recycle(Block *t){t->size=0;*(upper++)=t;}

void init(char *str,Block *&op,Block *&ed){
	int i,j;
	ed=op=newBlock();
	for(i=0,j=0;str[i];i++,j++){
		if(j==blockSize) ed->st[ed->size=j]=0,j=0,ed->nxt=newBlock(),ed=ed->nxt;
		ed->st[j]=str[i];
	}
	ed->st[ed->size=j]=0;
}

inline Block *split(int pos){
	Block *p=rt,*q;
	for(;p->nxt && pos>p->size;p=p->nxt) pos-=p->size;
	if(pos>=p->size) return p;
	q=newBlock();
	q->st[q->size=p->size-pos]=0;q->nxt=p->nxt;p->nxt=q;
	for(int i=0;i<q->size;i++) q->st[i]=p->st[i+pos];
	p->st[p->size=pos]=0;
	return p;
}

inline void merge(Block *a,Block *b){
	for(int i=0;i<b->size;i++) a->st[a->size+i]=b->st[i];
	a->st[a->size+=b->size]=0;a->nxt=b->nxt;recycle(b);
}

inline void maintain(Block *st){
	for(;st && st->nxt;st=st->nxt) if(st->size+st->nxt->size<maxBlockSize) merge(st,st->nxt);
}

inline void insert(char *str,int pos){
	Block *p=split(pos),*op,*ed;
	init(str,op,ed);
	ed->nxt=p->nxt;p->nxt=op;
	maintain(p==None ? p->nxt : p);
}

inline void remove(int l,int r){
	Block *op=split(l),*ed=split(r),*p;
	p=op->nxt;op->nxt=ed->nxt;
	for(;p!=ed->nxt;p=p->nxt) recycle(p);
	maintain(op);
}

inline void subseq(int l,int r){
	Block *op=split(l),*ed=split(r);
	for(Block *p=op->nxt;p!=ed->nxt;p=p->nxt) printf("%s",p->st);
	puts("");
	maintain(op);
}
void print(){
	for(Block *p=rt;p;p=p->nxt){
		printf("%s %d\n",p->st,p->size);
	}
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
			case 'I':{scanf("%d",&k);getInput(buf,k);insert(buf,cur);break;}
			case 'D':{scanf("%d",&k);remove(cur,cur+k);break;}
			case 'G':{scanf("%d",&k);subseq(cur,cur+k);break;}
			case 'P':{cur--;break;}
			case 'N':{cur++;break;}
			default:;
		}
	}
	return 0;
}
```
