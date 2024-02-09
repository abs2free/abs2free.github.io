---
title: 布隆过滤器
author: 
date: 2024-02-09 10:55:00 +0800
categories: [后端]
tags: [redis]
---

## 背景

1.去重问题

我们在使用新闻客户端的时候，它会不停的给我们推荐心得内容，而它每次推荐时都要去重，去掉我们已经看过的内容，那么新闻客户端推荐系统如何实现去重的？

你可能会想到：服务器记录了用户已经看过的所有历史记录，当推荐系统推荐新闻时可以从历史记录中筛选，过滤掉那些已经存在的记录。但是当用户量很大，每个用户看过很多新闻时，这种情况下去重工作的性能能跟得上吗？

实际上，如果存储在关系型数据库中，去重需要频繁地对数据库进行exists查询，当并发量很高时，数据库很难扛住压力。

如果将历史数据全部缓存起来，那得浪费多少存储空间？而且这个空间随着时间线性增长，能撑多长时间？如果不缓存，性能跟不上又怎么办？

2.缓存问题

在高并发请求时，业务数据一般会对数据进行缓存，提高系统并发量，因为磁盘IO和网络IO相对于内存IO的成百上千倍的性能劣势。
做个简单计算，如果我们需要某个数据，该数据从数据库磁盘读出来需要0.1s，从交换机传过来需要0.05s，那么每个请求完成最少0.15s（当然，事实上磁盘和网络IO也没有这么慢，这里只是举例），该数据库服务器每秒只能响应67个请求；而如果该数据存在于本机内存里，读出来只需要10us，那么每秒钟能够响应100，000个请求。
通过将高频使用的数据存在离cpu更近的位置，以减少数据传输时间，从而提高处理效率，这就是缓存的意义。

缓存会存在击穿，穿透等问题，对于此我们可以引入Bloom Filter (布隆过滤器)，防止数据库压力过大导致系统异常。

## 布隆过滤器是什么
Bloom Filter是一种空间效率很高的随机数据结构，它利用位数组很简洁地表示一个集合，并能判断一个元素是否属于这个集合。Bloom Filter的这种高效是有一定代价的：在判断一个元素是否属于某个集合时，有可能会把不属于这个集合的元素误认为属于这个集合（false positive）。因此，Bloom Filter不适合那些“零错误”的应用场合。而在能容忍低错误率的应用场合下，Bloom Filter通过极少的错误换取了存储空间的极大节省。

### 结构

bloom过滤器底层是一个位数组，初始时每一位都是0

![alt text](/assets/image.png)

当插入值x后，分别利用k个哈希函数（图中为3）利用x的值进行散列，并将散列得到的值与bloom过滤器的容量进行取余，将取余结果所代表的那一位值置为1。

![alt text](/assets/image-1.png)

一次查找过程与一次插入过程类似，同样利用k个哈希函数对所需要查找的值进行散列，只有散列得到的每一个位的值均为1，才表示该值“有可能”真正存在；反之若有任意一位的值为0，则表示该值一定不存在。例如y1一定不存在；而y2可能存在。

![alt text](/assets/image-2.png)

简而言之：**当布隆过滤器说某个值存在时，这个值可能不存在；当它说某个值不存在时，那就肯定不存在**。

## 如何选择哈希函数个数和布隆过滤器长度

如果布隆过滤器的长度太小，所有的 bit 位很快就会被用完，此时任何查询都会返回“可能存在”；
如果布隆过滤器的长度太大，那么误判的概率会很小，但是内存空间浪费严重。
类似的，哈希函数的个数越多，则布隆过滤器的 bit 位被占用的速度越快；哈希函数的个数越少，
则误判的概率又会上升。因此，布隆过滤器的长度和哈希函数的个数需要根据业务场景来权衡。
三个参数

哈希函数的个数k；
布隆过滤器位数组的容量m;
布隆过滤器插入的数据数量n;

关于三个值的设定，有位大佬十年前的文章已经做了总结，需要深厚的数学功底，这里不再详细赘述。
[文章链接](http://blog.csdn.net/jiaomeng/article/details/1495500)

主要的数学结论有：

为了获得最优的准确率，当k = ln2 * (m/n)时，布隆过滤器获得最优的准确性；

## 优缺点

优点：优点很明显，二进制组成的数组，占用内存极少，并且插入和查询速度都足够快。

缺点：随着数据的增加，误判率会增加；还有无法判断数据一定存在；另外还有一个重要缺点，无法删除数据

## Redis中的布隆过滤器
Redis官方提供的布隆过滤器到了4.0提供了插件功能。布隆过滤器作为插件加载的Redis Server中，给Redis提供了强大的布隆去重功能。

```sh
docker pull redislabs/rebloom    # 拉取镜像
docker run -p6379:6379 redislabs/rebloom # 运行容器
redis-cli  # 连接redis服务
```
### 基本用法
- bf.add code user1 // 添加元素
- bf.exists // 查询元素是否存在
- bf.madd // 添加多个元素
- bf.mexists // 查询多个元素

### 参数设置
Redis提供了自定义参数的布隆过滤器，需要我们在add之前使用`bf.reserve`指令现实创建。
`bf.reserve`有三个参数，分别是`key`,`error_rate`（错误率），和initial_size.
- `error_rate`越低，需要的空间越大。
- `initial_size`表示预计放入的元素数量，当实际数量超过这个值时，错误率会上升，所以需要提前设置一个比较大的数值避免超出导致误判率上升。
如果我们不设置`bf.reserve`，默认的`error_rate`是0.01，默认的`initial_size`是100。 

### 示例
```go
package main

import (
	"context"
	"fmt"

	"github.com/redis/go-redis/v9"
)

func main() {
	// 创建Redis客户端
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379", // Redis服务器地址
		Password: "",               // Redis密码，如果没有密码则为空
		DB:       0,                // Redis数据库索引，默认为0
	})

	ctx := context.Background()

	// 添加元素到布隆过滤器
	add, err := client.BFAdd(ctx, "myBloomFilter", "element1").Result()
	if err != nil {
		fmt.Println("添加元素失败:", err)
		return
	}
	fmt.Println("add success %t", add)

	// 检查元素是否存在于布隆过滤器中
	exists, err := client.BFExists(ctx, "myBloomFilter", "element1").Result()
	if err != nil {
		fmt.Println("查询元素失败:", err)
		return
	}
	if exists {
		fmt.Println("元素存在于布隆过滤器中")
	} else {
		fmt.Println("元素不存在于布隆过滤器中")
	}

	// 删除布隆过滤器
	del, err := client.Del(ctx, "myBloomFilter").Result()
	if err != nil {
		fmt.Println("删除布隆过滤器失败:", err)
		return
	}

	fmt.Println("del success %t", del)
}

```

### 注意事项
布隆过滤器的initial_size设置得过大，会浪费存储空间，设置的过小，就会影响准确率，用户在使用之前尽可能地精确估计元素数量，还要加上一定的冗余空间以避免实际元素可能会意外高出估计值很多。

布隆过滤器的error_rate越小，需要的空间越大，对于一些不需要过于精确的场合，error_rate设置的稍大一些也无伤大雅。比如在新闻客户端的去重，误判率高一点只会让小部分文章不能被合适的人看到，文章的阅读量不会因为这点误判率就带来巨大的改变。

## 免费的在线布隆计算器
[https://krisives.github.io/bloom-calculator/](https://krisives.github.io/bloom-calculator/)