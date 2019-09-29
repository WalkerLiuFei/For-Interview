# Buffer ,Reader ,Writer 

## 参考

https://medium.com/golangspec/introduction-to-bufio-package-in-golang-ad7d1877f762



Bufio.Writer 和 Bufio.Reader

1. 在持久化Write操作中，如果进行大量，小批量Write操作会加大CPU负担，通过Bufio缓冲池，等待缓存池满了以后才会进行持久化操作。这样可以减轻CPU的负担。

2. Bufio.Reader 类似Bufio.Writer，通过缓存的方式减轻CPU压力
3. ReadCloser和WriteCloser类似

