B树
===

B树也是平衡搜索树，但与红黑树不同的是B树是为磁盘设计的一种搜索树。

##读写磁盘哪里耗时呢
都知道读写磁盘都是相对更耗时的，但是时间具体是花到哪了呢？
因此如果能减少读写磁盘的次数，效率必然更高。因此B树通过增大分支因子，从而大幅度地降底树的高度。