# Map 和slice的底层数据结构

1. go中map 的扩容和java中有很大的不同，oldbucket不会像java中立马转移到新的bucket中，只有当访问到该bucket的时候才会使用growWork方法来进行迁移，随着访问 最终会完成所有的迁移，换言之，golang中的map是通过懒迁移的方式进行的
2. java 和 go 都是用拉链来处理的 hash冲突

## slice的数据结构

