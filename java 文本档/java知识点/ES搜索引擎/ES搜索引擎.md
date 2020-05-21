## ES分布式架构原理

ElasticSearch设计理念就是分布式搜搜引擎，底层其实还是基于Lucene的。核心思想就是在多台机器上启动多个es进程实例，组成一个es集群。

es中存储数据的基本单位是索引，比如说你现在要在es中存储一些订单数据，你就应该在es中创建一个索引order_idx，所有的订单数据就都写到这个索引里面去，一个索引差不多就是相当于是mysql里的一张表。

```
index -> type ->mapping ->document ->field
```

index相当于mysql中的一张表，而type无法跟mysql中的对比，一个index里可以有多个type，每个type的字段都是差不多的，但是有一些略微的差别。可以算是定义index的类型。

mapping就是type的表定义，你在mysql中创建一个表，index就代表多个type同属于的一个类型，实际上index里的一个type里面写一条数据，叫做一条document，一条document就代表了mysql中某一个表里的一行，每个document有多个filed，每个filed就代表了这个document中的一个字段的值。

![es-index-type-mapping-document-field](https://github.com/doocs/advanced-java/raw/master/images/es-index-type-mapping-document-field.png)

你创建一个索引，这个索引可以拆分成多个shard，每个shard存储部分数据。拆分多个shard是有好处的，一是支持横向扩展，比如数据量是3T，3个shard，每个可以占1T数据，若现在数据量增加到4T，就可以重新建一个有4个shard的索引，将数据导入进去；二是提高性能，数据分布在多个shard，即多台服务器，都会在多台服务器上并行分布式执行，提高吞吐量和性能。

接着就是这个shard的数据实际是有多个备份的，就是每个shard都有一个primary shard，负责写入数据，但是还有几个replica shard。primary shard写入数据之后，会将数据同步到replica shard上去。

![es-cluster](https://github.com/doocs/advanced-java/raw/master/images/es-cluster.png)

es集群有多个节点，会自动选举一个节点作为master节点，这个节点就是做一些管理的工作，比如维护索引元数据，负责去切换primary shard 和replica shard身份等，要是master宕机了，那么就会重新选举一个master出来。

如果非master节点宕机了，那么就会由master节点，让那个宕机节点上的primary shard的身份转移到其他机器上的replica shard。接着你要是恢复那个宕机机器，重启之后，master节点将会继续控制将缺失的replica shard分配过去，同步后续修改的数据之类的，让集群恢复正常。

说得更简单一点，就是说如果某个非 master 节点宕机了。那么此节点上的 primary shard 不就没了。那好，master 会让 primary shard 对应的 replica shard（在其他机器上）切换为 primary shard。如果宕机的机器修复了，修复后的节点也不再是 primary shard，而是 replica shard。

### es写入数据的工作原理？es查询数据工作原理？底层的lucene？倒排索引？

#### es写数据过程

- 客户端选择一个node发送请求过去，这个node就是coordinating node（协调节点）。

- coordinating node对document进行路由，将请求转发给对应的node（有primary shard）。

- 实际的node上的primary shard处理请求，然后将数据同步到replica node。

- coordinating node如果发现primary node和所有的replica node都搞定之后，就会返回响应结果给客户端。

  ![es-write](https://github.com/doocs/advanced-java/raw/master/images/es-write.png)

#### es读数据过程

可以通过doc_id来查询，会根据doc_id进行hash，判断出来当时把doc_id分配到哪个shard上面去，从那个shard去查询。

- 客户端发送请求到任意一个node，成为coordinating node。
- coordinating node对doc_id进行哈希路由，将请求转发到对应的node，此时会使用round_robin随机轮询算法，在primary shard以及其所有replica中随机选择一个，让请求负载均衡。
- 接收请求的node返回document给coordinating node。
- coordinating node返回给document给客户端。



#### es搜索数据过程

es最强大的就是做全文检索

- 客户端发送请求到一个coordinating node。
- 协调节点将搜索请求转发到**所有**的shard对应的primary shard或replica shard，都可以。
- query phase：每个shard将自己搜索结果（其实就是一些doc_id）返回给协调节点，由协调节点进行数据的合并，排序和分页等操作，产出最终结果。
- fetch phase：接着由协调节点根据doc_id区各节点**拉取实际**的document数据，最终返回到客户端。

```
写请求就是写入primary shard，然后同步给所有的replica shard；读取请求可以从primary shard或replica shard读取，采用的是随机轮询算法。
```

#### 写数据底层原理

![es-write-detail](https://github.com/doocs/advanced-java/raw/master/images/es-write-detail.png)

1. 先写入内存buffer，在buffer里的时候数据是搜索不到的；同时将数据写入translog日志文件。
2. 如果buffer快写满了，或者到一定时间，就会将内存buffer数据refresh到新的segment file中，但是此时数据不是直接进入segment file磁盘文件，而是先进入os cache。这个过程就是refresh。
3. 每隔1秒钟，es将buffer中的数据写入到一个新的segment file，每秒钟会产生一个新的磁盘文件segment file，这个segment file中就存储最近1秒内buffer中写入的数据。
4. 但是如果buffer里面此时没有数据，那当然不会执行refresh操作，如果buffer里面有数据，默认1秒钟执行一次refresh操作，刷入一个新的segment file中。
5. 操作系统里面，磁盘文件其实都有一个东西，叫做os cashe，即操作系统缓存，就是说数据写入磁盘文件之前，会先进入os cache，先进入操作系统级别的一个内存缓存中去，只有buffer中的数据被refresh操作刷入os cache中，这个数据就可以被搜索到了。

**为什么es是准实时的？NRT，全称near real_time。**默认是每隔1秒refresh一次的，所以es准实时的，因为写入的数据1秒之后才能被看到。可以通过es的restful api或者java api，手动执行一次refresh操作，就是手动将buffer中的数据刷入os cache中，让数据立马就可以搜索到。只要数据被输入os cache中，buffer就会被清空，数据在translog里面已经持久化一份到磁盘里了。

重复上面的步骤，新的数据不断进入buffer和translog，不断将buffer数据写入一个又一个新的segment file中去，每次refresh完buffer请空调，translog保留。随着这个过程推进，translog会变得越来越大。当translog达到一定长度的时候，就会触发commit操作。

commit操作发生的第一步，就是将buffer中现有的数据refresh到os cache中去，清空buffer。然后，将一个commit point写入磁盘文件，里面标识着这个commit point对应的所有segment file，同时强行将os cache中目前所有的数据都fsync到磁盘文件中去。最后清空现有的translog日志文件，重启一个translog，此时commit操作完成。

这个commit操作叫做flush。默认30分钟自动执行一次flush，如果translog过大，也会触发flush。flush操作就对应着commit的全过程，我们可以通过es api，手动执行flush操作，手动将os cache中的数据fsync强刷到磁盘中去。

translog日志文件的作用是什么？你执行commit操作之前，数据要么停留在buffer中，要么停留在os cache中，无论是buffer还是os cache都是内存，一旦这台机器死了，内存数据就会丢失，所以需要将数据对应的操作写入一个专门的日志文件translog中，一旦此时机器宕机，再次重启的时候，es会自动读取translog日志文件的数据，恢复到内存buffer和os cache中去。

translog其实也是先写入os cache的，默认每隔5秒刷一次到磁盘中去，所以默认的情况下，可能有5秒的数据会仅仅停留在buffer或者translog文件的os cache中，如果此时机器挂了，会丢失5秒钟的数据。但是这样性能比较好，最多丢失5秒的数据。也可以将translog设置成每次写操作必须直接fsync到磁盘，但是性能会差很多。

丢失数据的场景：有5秒的数据，停留在buffer，translog os cache segment file os cache中，而不在磁盘上，此时如果机器宕机，会导致丢失这5秒的数据。

**总结：**

数据是先写入内存buffer ------1秒后----->将数据refresh到os cache，到了这就可以搜索到了-------每隔5秒------>将数据写入translog文件（此时如果机器宕机则会丢失这5秒的数据），translog达到一定程度或者默认每隔30分钟，会触发commit操作，将缓冲区的数据flush到segment磁盘本地中去。

#### 删除/更新数据底层原理

如果是删除操作，commit的时候会生成一个.del文件，里面将某个doc标识为deleted状态，那么搜索的时候根据.del文件就知道这个doc文件是否被删除了。

如果是更新操作，就是将原来的doc标识为deleted状态，然后新写入一条数据。 

buffer每refresh一次，就会产生一个segment file，所以默认情况下是一秒钟一个segment file，这样下来segment file会越来越多，此时会定期执行merge。每次merge的时候，会将多个segment file合并成一个，同时这里会被标识为deleted的doc给物理删除掉，然后将新的segment file写入磁盘，这里会写一个commit point，标识所有新的segment file，然后打开segment file供搜索使用，同时删除旧的segment file。

#### 底层lucene

lucene就是一个里面包含封装好的各种建立倒排索引的算法代码的一个jar，我们所谓的引入lucene也就是对应着lucene的相关API进行操作数据。

#### 倒排索引

- 倒排索引中的所有词项对应一个或多个文档
- 倒排索引中的词项**根据字典顺序升序排列**