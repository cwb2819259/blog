# 分布式id生成算法 - SnowFlake


#### 概述

分布式系统中，有一些需要使用全局唯一ID的场景，这种时候为了防止ID冲突可以使用36位的UUID，但是UUID有一些缺点，首先他相对比较长，另外UUID一般是无序的，
由于采用的是无意义的字符串，推测会在数据量增大时造成访问过慢，在基础互联网的系统设计中都不推荐采用。
有些时候我们希望能使用一种简单一些的ID，并且希望ID能够按照时间有序生成。

而twitter的snowflake解决了这种需求，最初Twitter把存储系统从MySQL迁移到Cassandra，因为Cassandra没有顺序ID生成机制，所以开发了这样一套全局唯一ID生成服务。

自增ID的方法虽然比较适合大数据量的场景，当时由于自增ID是按照顺序增加的，数据记录都是可以根据ID号进行推测出来，对于一些数据敏感的场景，不建议采用。


#### Snowflake 算法核心
把 **时间戳** ，**工作机器id** ，**序列号** 组合在一起。

snowflake的结构如下(每部分用-分开):

![Alt text](images/snowflake-64bit.jpg)


- `1位`，不用。二进制中最高位为1的都是负数，但是我们生成的id一般都使用整数，所以这个最高位固定是0

- `41位`，用来记录时间戳（毫秒）。
    - `41位`可以表示$2^{41}-1$个数字，
    
    - 如果只用来表示正整数（计算机中正数包含0），可以表示的数值范围是：0 至 $2^{41}-1$，减1是因为可表示的数值范围是从0开始算的，而不是1。
    
    - 也就是说41位可以表示$2^{41}-1$个毫秒的值，转化成单位年则是$(2^{41}-1) / (1000 * 60 * 60 * 24 * 365) = 69$年
    
- `10位`，用来记录工作机器id。
    - 可以部署在$2^{10} = 1024$个节点，包括5位datacenterId和5位workerId
    
    - `5位（bit）`可以表示的最大正整数是$2^{5}-1 = 31$，即可以用0、1、2、3、....31这32个数字，来表示不同的datecenterId或workerId
    
- `12位`，序列号，用来记录同毫秒内产生的不同id。
    - `12位（bit）`可以表示的最大正整数是$2^{12}-1 = 4095$，即可以用0、1、2、3、....4094这4095个数字，来表示同一机器同一时间截（毫秒)内产生的4095个ID序号

#### 感谢

[twitter开源项目 - Scala版](https://github.com/twitter/snowflake)

[pysnowflake - Python版](https://github.com/erans/pysnowflake)

[理解分布式id生成算法SnowFlake](https://segmentfault.com/a/1190000011282426)

[64位自增ID算法详解](https://www.lanindex.com/twitter-snowflake%EF%BC%8C64%E4%BD%8D%E8%87%AA%E5%A2%9Eid%E7%AE%97%E6%B3%95%E8%AF%A6%E8%A7%A3/)
