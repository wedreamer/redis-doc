**注意：本文档由Redis的创建者Salvatore Sanfilippo撰写，在Redis开发的早期（约2010年）。自 Redis 2.6 以来，虚拟内存已被弃用，因此本文档
这里只是为了历史兴趣。**

Redis 字符串的实现包含在`sds.c`(`sds`代表
简单动态字符串）。该实现可作为独立库使用
在<https://github.com/antirez/sds>.

C 结构`sdshdr`声明于`sds.h`表示一个 Redis 字符串：

    struct sdshdr {
        long len;
        long free;
        char buf[];
    };

这`buf`字符数组存储实际字符串。

这`len`字段存储的长度`buf`.这使得获得长度
的 Redis 字符串的 O（1） 操作。

这`free`字段存储可供使用的其他字节数。

携手共进`len`和`free`字段可以被认为是保存`buf`字符数组。

## 创建 Redis 字符串

名为 的新数据类型，名为`sds`定义于`sds.h`成为字符指针的同义词：

    typedef char *sds;

`sdsnewlen`中定义的函数`sds.c`创建一个新的 Redis 字符串：

    sds sdsnewlen(const void *init, size_t initlen) {
        struct sdshdr *sh;

        sh = zmalloc(sizeof(struct sdshdr)+initlen+1);
    #ifdef SDS_ABORT_ON_OOM
        if (sh == NULL) sdsOomAbort();
    #else
        if (sh == NULL) return NULL;
    #endif
        sh->len = initlen;
        sh->free = 0;
        if (initlen) {
            if (init) memcpy(sh->buf, init, initlen);
            else memset(sh->buf,0,initlen);
        }
        sh->buf[initlen] = '\0';
        return (char*)sh->buf;
    }

请记住，Redis 字符串是类型的变量`struct sdshdr`.但`sdsnewlen`返回一个字符指针！！

这是一个技巧，需要一些解释。

假设我使用`sdsnewlen`如下图所示：

    sdsnewlen("redis", 5);

这将创建一个类型为的新变量`struct sdshdr`为`len`和`free`
字段以及`buf`字符数组。

    sh = zmalloc(sizeof(struct sdshdr)+initlen+1); // initlen is length of init argument.

后`sdsnewlen`成功创建 Redis 字符串，结果如下：

    -----------
    |5|0|redis|
    -----------
    ^   ^
    sh  sh->buf

`sdsnewlen`返回`sh->buf`给调用方。

如果您需要释放指向的 Redis 字符串，该怎么办`sh`?

您想要指针`sh`但你只有指针`sh->buf`.

你能得到指针吗？`sh`从`sh->buf`?

是的。指针算术。请注意，如果减去上述 ASCII 艺术
大小为两条龙头，从`sh->buf`你得到指针`sh`.

这`sizeof`两条多头恰好是`struct sdshdr`.

看`sdslen`函数，并在工作中看到这个技巧：

    size_t sdslen(const sds s) {
        struct sdshdr *sh = (void*) (s-(sizeof(struct sdshdr)));
        return sh->len;
    }

知道了这个技巧，你可以很容易地完成其余的功能`sds.c`.

Redis 字符串实现隐藏在仅接受字符指针的接口后面。Redis字符串的用户不需要关心它是如何实现的，可以将Redis字符串视为字符指针。
