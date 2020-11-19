# Page Cache

Page Cache 是linux系统在进行磁盘读写时在内存中做的缓存，可以通过 `free -m` 命令查看`page cache`使用情况

```
walker@walker:~$ free -m
              total        used        free      shared  buff/cache   available
Mem:          11879         118       11586           0         174       11548
Swap:          3072           0        3072
```

buff / cache 即为 Page Cache 占用的内存情况，可以看出现在 Page Cache占了 174mb的内存

##  Page Cache limit

限制Page Cache还是挺有意义的，，减轻内存管理的负担等等

[参考 Redhat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/tuning_and_optimizing_red_hat_enterprise_linux_for_oracle_9i_and_10g_databases/sect-oracle_9i_and_10g_tuning_guide-memory_usage_and_page_cache-tuning_the_page_cache)