##[redies-data-structure](http://redisbook.com/index.html)

###1.**SDS**
---
`sds.h, sds.c`
```
struct sdshdr {

    // 记录 buf 数组中已使用字节的数量
    // 等于 SDS 所保存字符串的长度
    int len;

    // 记录 buf 数组中未使用字节的数量
    int free;

    // 字节数组，用于保存字符串
    char buf[];

};
```
C中字符串的升级版。优点：

 1. 常熟时间获取字符串长度
 2. 避免了原生C中对字符串处理溢出的可能
 3. 缓存和vector的机制差不多，避免频繁重新分配内存来满足字符串修改
 4. 二级制安全，也就是以len为长度。不仅仅以‘\0’为终结符
 5. 兼容部分C字符串函数，因为SDS也是以'\0'结尾

###2.**double list**
---
`adlist.h, adlist.c`
```
typedef struct listNode {

    // 前置节点
    struct listNode *prev;

    // 后置节点
    struct listNode *next;

    // 节点的值
    void *value;

} listNode;
```
```
typedef struct list {

    // 表头节点
    listNode *head;

    // 表尾节点
    listNode *tail;

    // 链表所包含的节点数量
    unsigned long len;

    // 节点值复制函数
    void *(*dup)(void *ptr);

    // 节点值释放函数
    void (*free)(void *ptr);

    // 节点值对比函数
    int (*match)(void *ptr, void *key);

} list;
```
C语言也能“多态”，主要是利用了void*，使得链表能保存多总类型的数据。
每一种数据都会有自己对应的复制、释放、对比函数。这里的设置是通过函数指针来实现的。

###**3.dict**
---
`dict.h, dict.c`
```
typedef struct dictht {

    // 哈希表数组
    dictEntry **table;

    // 哈希表大小
    unsigned long size;

    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;

    // 该哈希表已有节点的数量
    unsigned long used;

} dictht;
```
```
typedef struct dictEntry {

    // 键
    void *key;

    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;

    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;

} dictEntry;
```
```
typedef struct dict {

    // 类型特定函数
    dictType *type;

    // 私有数据
    void *privdata;

    // 哈希表
    dictht ht[2];

    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; // rehashing not in progress if rehashidx == -1 

} dict;
```
```
typedef struct dictType {

    // 计算哈希值的函数
    unsigned int (*hashFunction)(const void *key);

    // 复制键的函数
    void *(*keyDup)(void *privdata, const void *key);

    // 复制值的函数
    void *(*valDup)(void *privdata, const void *obj);

    // 对比键的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);

    // 销毁键的函数
    void (*keyDestructor)(void *privdata, void *key);

    // 销毁值的函数
    void (*valDestructor)(void *privdata, void *obj);

} dictType;
```
 - C语言的字典的实现，再次使用void*来实现“多态”，其中我们要提供dictType结构体来实现不同的类型。
 -  哈希表的rehash是渐进的，这是分摊的策略，避免了服务器会在rehash中会
 - 哈希函数是murmurhash2，对有规律的键都能产生很好的随机性



###**4.skiplist**
---
`redis.h/zskiplistNode`
```
typedef struct zskiplistNode {

    // 后退指针
    struct zskiplistNode *backward;

    // 分值
    double score;

    // 成员对象
    robj *obj;

    // 层
    struct zskiplistLevel {

        // 前进指针
        struct zskiplistNode *forward;

        // 跨度
        unsigned int span;

    } level[];

} zskiplistNode;
```
`redis.h/zskiplist`
```
typedef struct zskiplist {

    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;

    // 表中节点的数量
    unsigned long length;

    // 表中层数最大的节点的层数
    int level;

} zskiplist;
```
-  如果一个有序集合包含的元素数量比较多， 又或者有序集合中元素的成员（member）是比较长的字符串时， Redis 就会使用跳跃表来作为有序集合键的底层实现
- skiplist在redis中是有序集合的底层实现之一，skiplist是可以在概率上保证和平衡树有同样的性能。这里以前了解过，就不写了。

###**5.intset**
---
```
typedef struct intset {

    // 编码方式
    uint32_t encoding;

    // 集合包含的元素数量
    uint32_t length;

    // 保存元素的数组
    int8_t contents[];

} intset;
```

 - 整数集合（intset）是集合键的底层实现之一： 当一个集合只包含整数值元素， 并且这个集合的元素数量不多时， Redis 就会使用整数集合作为集合键的底层实现。
 - 整数集合的contents只是存放二进制数的一个结构，每个整数都是通过encoding的类型来编码的，例如：1byte，4byte，8byte。
 - 整数集合提供了upgrade功能来节约内存，同时提升了集合操作的灵活性。upgrade功能是在集合加入了一个超过了当前编码范围的数字的时候自动将所有的数字重新编码。整数集合但是不提供downgrade的功能
 

###**6.ziplist**
---
![enter image description here](http://redisbook.com/_images/graphviz-fe42f343a3f32f477efb5e895da547d476a7c97d.png)

| 属性    | 类型     | 长度   | 用途                                                                                                                                                                                              |
|---------|----------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| zlbytes | uint32_t | 4字节 | 记录整个压缩列表占用的内存字节数：在对压缩列表进行内存重分配， 或者计算 zlend的位置时使用。                                                                                                       |
| zltail  | uint32_t | 4字节 | 记录压缩列表表尾节点距离压缩列表的起始地址有多少字节： 通过这个偏移量，程序无须遍历整个压缩列表就可以确定表尾节点的地址。                                                                         |
| zllen   | uint16_t  | 2字节 | 记录了压缩列表包含的节点数量： 当这个属性的值小于 UINT16_MAX （65535）时， 这个属性的值就是压缩列表包含节点的数量； 当这个值等于 UINT16_MAX 时， 节点的真实数量需要遍历整个压缩列表才能计算得出。 |
| entryX  | 列表节点 | 不定   | 压缩列表包含的各个节点，节点的长度由节点保存的内容决定。                                                                                                                                          |
| zlend   | uint8_t  | 1字节 | 特殊值 0xFF （十进制 255 ），用于标记压缩列表的末端。                                                                                                                                             |

![enter image description here](http://redisbook.com/_images/graphviz-cc6b40e182bfc142c12ac0518819a2d949eafa4a.png)
其中压缩表的节点encoding有多种，因为要对字符串和整数都编码，它们的大小又可能不一样

 - 当一个列表键只包含少量列表项， 并且每个列表项要么就是小整数值， 要么就是长度比较短的字符串， 那么 Redis 就会使用压缩列表来做列表键的底层实现。
 - 压缩列表是一种为节约内存而开发的顺序型数据结构。
 - 压缩列表被用作列表键和哈希键的底层实现之一。
 - 压缩列表可以包含多个节点，每个节点可以保存一个字节数组或者整数值。
 - 添加新节点到压缩列表， 或者从压缩列表中删除节点， 可能会引发连锁更新操作， 但这种操作出现的几率并不高。

###**7.Object**
---
```
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码
    unsigned encoding:4;

    // 指向底层实现数据结构的指针
    void *ptr;

    // ...

} robj;
```
Object 的5中类型：
| 对象         | 对象 type 属性的值 | TYPE 命令的输出 |
|--------------|--------------------|-----------------|
| 字符串对象   | REDIS_STRING       | string          |
| 列表对象     | REDIS_LIST         | list            |
| 哈希对象     | REDIS_HASH         | hash            |
| 集合对象     | REDIS_SET          | set             |
| 有序集合对象 | REDIS_ZSET         | zset            |

每种类型有2种或以上的编码方式：

| 类型         | 编码                      | 对象                                               |
|--------------|---------------------------|----------------------------------------------------|
| REDIS_STRING | REDIS_ENCODING_INT        | 使用整数值实现的字符串对象。                       |
| REDIS_STRING | REDIS_ENCODING_EMBSTR     | 使用 embstr 编码的简单动态字符串实现的字符串对象。 |
| REDIS_STRING | REDIS_ENCODING_RAW        | 使用简单动态字符串实现的字符串对象。               |
| REDIS_LIST   | REDIS_ENCODING_ZIPLIST    | 使用压缩列表实现的列表对象。                       |
| REDIS_LIST   | REDIS_ENCODING_LINKEDLIST | 使用双端链表实现的列表对象。                       |
| REDIS_HASH   | REDIS_ENCODING_ZIPLIST    | 使用压缩列表实现的哈希对象。                       |
| REDIS_HASH   | REDIS_ENCODING_HT         | 使用字典实现的哈希对象。                           |
| REDIS_SET    | REDIS_ENCODING_INTSET     | 使用整数集合实现的集合对象。                       |
| REDIS_SET    | REDIS_ENCODING_HT         | 使用字典实现的集合对象。                           |
| REDIS_ZSET   | REDIS_ENCODING_ZIPLIST    | 使用压缩列表实现的有序集合对象。                   |
| REDIS_ZSET   | REDIS_ENCODING_SKIPLIST   | 使用跳跃表和字典实现的有序集合对象。               |

 
 - 这样的灵活性是的redis能够在不同的场景给出不同的优化
 - Redis用引用计数来实现对象共享和内存回收。其中共享的只有0 到 9999 的字符串对。因为其他对象的共享需要判断时间复杂度较高。当然这可以按需求配置。
 - Redis的对象都会有lru的属性值来记录最后一次访问时间，由名字看出是用于lru算法之类的。

