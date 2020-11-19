## Sync.Pool

The `sync` package provides a powerful instance pool that can be re-used in order to reduce the pressure of reuse garbage collector.

```
type Small struct {
   a int
}

var pool = sync.Pool{
   New: func() interface{} { return new(Small) },
}

//go:noinline
func inc(s *Small) { s.a++ }

func BenchmarkWithoutPool(b *testing.B) {
   var s *Small
   for i := 0; i < b.N; i++ {
      for j := 0; j < 10000; j++ {
         s = &Small{ a: 1, }
         b.StopTimer(); inc(s); b.StartTimer()
      }
   }
}

func BenchmarkWithPool(b *testing.B) {
   var s *Small
   for i := 0; i < b.N; i++ {
      for j := 0; j < 10000; j++ {
         s = pool.Get().(*Small)
         s.a = 1
         b.StopTimer(); inc(s); b.StartTimer()
         pool.Put(s)
      }
   }
}
```



```
name           time/op        alloc/op        allocs/op
WithoutPool-8  3.02ms ± 1%    160kB ± 0%      1.05kB ± 1%
WithPool-8     1.36ms ± 6%   1.05kB ± 0%        3.00 ± 0%
```

### survival 区

sync.pool 中的对象在一次GC后会被存放在survivor区，再一次GC后才会被被清理，正因为如此如果业务中存在频繁GC的应用并不一定完全就适合使用sync.pool。 他会调整sync.pool的随便，占用大量CPU。

## Warning 

But, in real world, when you using pool, your application will do lots of new instance operation in heap. In this situaition, when heap size increase, it will trigger the garbage collector. And garbage collector will trigger `sync.pool` to clear all its cache instances. And, your application will be slower with the use of `sync.pool` than without it.  So, you have to balance the it.

但是，在现实世界中，当您使用池时，您的应用程序将对堆进行很多新分配。在这种情况下，当内存增加时，它将触发垃圾回收器。

