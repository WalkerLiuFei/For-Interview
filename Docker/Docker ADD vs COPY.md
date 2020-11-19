

**ADD 和COPY的主要区别就在于 ADD 支持 Remote URL, 并且 如果`src` file 是一个压缩文件的话，ADD 操作会执行解压操作，需要注意的是如果tar file 是通过 remote url引用的话，其不会被解压**



在写 Docker file  时，除非需要ADD的特性，一般建议使用 COPY操作。

## ADD 

格式 ：

```
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

如果路径里面包含空格，则需要后面的那种格式

1. ADD 支持 src 是 HTTP的URL, 且download 后的文件权限为 ·600

2. ADD 的src 支持 regex 语法 ： 

   1. In the example below, `?` is replaced with any single character, e.g., “home.txt”.

   ```
   ADD hom?.txt /mydir/
   ```

3. 添加包含特殊字符（例如[和]）的文件或目录时，您需要按照Golang规则转义那些路径，以防止将它们视为匹配模式。

   1. 例如，要添加名为arr [0] .txt的文件，请使用以下命令： 添加`arr [[] 0] .txt / mydir /`

4. ADD 不支持 Authentication， 所以如果remote url 的文件需要权限验证，使用`wget` 或者`curl` alternatively.



### Rules 

1. <src>路径必须在构建上下文内；您不能添加`../something /something`，因为Docker构建的第一步是将上下文目录（和子目录）发送到`docker deamon`守护程序。
2. 如果<src>是URL，并且<dest>不以斜杠结尾，则从URL下载文件并将其复制到<dest>。如果带着slash 那么就会被拷贝为 `/dest/filename`
3. **如果 file 是个 tar / zip  文件, ADD 到 dest 会被解压， 但是如果这个 tar文件是 remote url 指定的tar 文件，则不会被执行解压操作**





