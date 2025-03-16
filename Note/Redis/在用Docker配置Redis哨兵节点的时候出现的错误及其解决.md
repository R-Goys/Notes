在用Docker配置Redis哨兵节点的时候，出现了以下的错误：

**WARNING: Sentinel was not able to save the new configuration on disk!!!: Device or resource busy**

**redis-sentinel throws error: " Can't resolve master instance hostname.**

这都是我踩过的坑，问AI也半天搞不定，现在来记录一下是怎么解决的。

首先是第一个问题

字面来看，就是Redis无法进行写入操作。

你的docker-compose.yml文件关于sentinel的配置可能是这样的

```c
    command: "redis-server /etc/redis-config/redis.conf --sentinel"
    volumes:
      - "./config/redis-sentinel/sentinel.conf:/etc/redis-config/redis.conf"
```

为了让redis能够进行写入，我们不能仅仅挂载一个配置文件，需要将文件夹一同挂载下来，所以解决办法是：

```c
    command: "redis-server /etc/redis-config/redis.conf --sentinel"
    volumes:
      - "./config/redis-sentinel:/etc/redis-config"
```

总体应该看起来是这样的：

```c
  sentinel:
    image: redis:latest
    container_name: redis-sentinel
    networks:
      - redis-net
    ports:
      - "26381:26379"
    command: "redis-server /etc/redis-config/redis.conf --sentinel"
    volumes:
      - "./config/redis-sentinel:/etc/redis-config"
    depends_on:
      - redis-master
      - redis-slave1
      - redis-slave2
```

那么回到第二个问题，

**redis-sentinel throws error: " Can't resolve master instance hostname.**

这里如果你的redis版本比较旧，需要在配置文件中将其修改成hostname修改为ip。

如果你已经确保你的redis版本在6.2之后，依旧出现这个问题，可以在挂载的配置文件中加上

```c
sentinel resolve-hostnames yes
```

这一行配置，就可以了。



**参考文献**

[WARNING: Sentinel was not able to save the new configuration on disk!!!: Device or resource busy](https://stackoverflow.com/questions/70384566/warning-sentinel-was-not-able-to-save-the-new-configuration-on-disk-device)

[redis-sentinel throws error: " Can't resolve master instance hostname."](https://stackoverflow.com/questions/57464443/redis-sentinel-throws-error-cant-resolve-master-instance-hostname)

**总结**

StackOverflow永远的神.
