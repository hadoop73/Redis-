---
title: Redis 跳表 
tags: Redis,跳表
grammar_cjkRuby: true
---

[TOC]

[跳表（skiplist）的代码实现][1]

##  跳表结构

![跳表结构][2]

```cpp?linenums
// 跳表节点
typedef struct skiplistNode {
     double score;
     struct skiplistNode *backward;
     struct skiplistLevel {
         struct skiplistNode *forward;
     }level[];
}skiplistNode;

// 跳表结构
typedef struct skiplist {
    struct skiplistNode *header, *tail;
    unsigned long length;
    int level;
}skiplist;
```
每一个 `skiplist` 有一个 `header` 和 `tail`，用 `length` 记录跳表节点数，其中 `level` 表示节点中最大层数。

每个 `skiplistNode` 有一个递增的 `score`，`backward` 指向前一个节点，每一层的 `forward` 表示该层的下一个节点。

##  新建跳表

跳表头部的第一个节点不保存元素，一共有 `SKIPLIST_MAXLEVEL` 层。
```cpp?linenums
// 新建一个level层，score的节点
skiplistNode *slCreateNode(int level, double score) {
    skiplistNode * sn = malloc(sizeof(*sn) + level*sizeof(struct skiplistLevel));
    sn->score = score;
    return sn;
}

// 新建一个sliplist和一个SKIPLIST_MAXLEVEL层的节点
skiplist *slCreate(void) {
    int j;
    skiplist *sl;

    sl = malloc(sizeof(*sl));
    sl->level = 1;
    sl->length = 0;
    sl->header = slCreateNode(SKIPLIST_MAXLEVEL, 0);
    for(j = 0; j < SKIPLIST_MAXLEVEL; j++) {
        sl->header->level[j].forward = NULL;
    }
    sl->header->backward = NULL;
    sl->tail = NULL;
    return sl;
}

```
##  跳表搜索
对于一个完整的跳表，要查找一个 `score` 的节点，先从最高的 `level` 开始查找，能查找到距离最远的符合要求的节点，再降低 `level`，从已经查到的符合要求的节点出发，继续查找符合条件的节点，直到 `level` 为 `0`。
```cpp?linenums
skiplistNode *update[SKIPLIST_MAXLEVEL];
skiplistNode *node;

node = sl->header;
int i, level;
for ( i = sl->level-1; i >= 0; i--) {
    while(node->level[i].forward && node->level[i].forward->score < score) {
        node = node->level[i].forward;
    }
    update[i] = node;
}
```
`update` 数组用于记录每一层指向符合要求节点的节点

##  插入节点
在插入节点前，需要确定节点的层数，`level` 是通过随机函数产生，然后，查找节点，得到每层的前节点，最后调整节点指针
```cpp?linenums
level = slRandomLevel();
// level 大于 skipList 的 level，则需要调整
if (level > sl->level) {
    for (i = sl->level; i< level ;i++) {
        update[i] = sl->header;
    }
    sl->level = level;
}

// node 为前面搜索到的节点
// 调整指针，相当于链表的插入
node = slCreateNode(level, score);
for (i = 0; i < level; i++) {
    node->level[i].forward = update[i]->level[i].forward;
    update[i]->level[i].forward = node;
}
```


##  删除节点
先查找，调整指针，再删除
```cpp?linenums
void slDeleteNode(skiplist *sl, skiplistNode *x, skiplistNode **update){
    int i;
    for (i = 0; i < sl->level; i++) {
        if (update[i]->level[i].forward == x) {
            update[i]->level[i].forward = x->level[i].forward;
        }
    }
    if (x->level[0].forward) {
        x->level[0].forward->backward = x->backward;
    } else {
        sl->tail = x->backward;
    }
    while (sl->level > 1 && sl->header->level[sl->level-1].forward == NULL) 
        sl->level--;
    sl->length--;
}

int slDelete(skiplist *sl, double score) {
    skiplistNode *update[SKIPLIST_MAXLEVEL], *node;
    int i;

    node = sl->header;
    for(i = sl->level-1; i >= 0; i--) {
        while (node->level[i].forward && node->level[i].forward->score < score) {
            node = node->level[i].forward;
        }
        update[i] = node;
    }
    node = node->level[0].forward;
    if (node && score == node->score) {
        slDeleteNode(sl, node, update);
        slFreeNode(node);
        return 1;
    } else {
        return 0;
    }
    return 0;
}
```


  [1]: http://www.cnblogs.com/liuhao/archive/2012/07/26/2610218.html
  [2]: ./images/1461158199514.jpg "1461158199514.jpg"