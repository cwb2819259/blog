## Golang 操作 etcd（上）

### 目录

[1. 连接 etcd](#1-连接-etcd)

[2. Put 写入 KV](#2-put-写入-kv)

[3. Get 读取 KV](#3-get-读取-kv)

[4. Get 读取目录下所有 KV](#4-get-读取目录下所有-kv)

[5. Delete 删除 KV](#5-delete-删除-kv)

--- 

[相关博文](#相关博文)


#### 1. 连接 etcd

<details>
<summary>查看源代码</summary>

```golang
package main

import (
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main() {

	var (
		config clientv3.Config
		client *clientv3.Client
		err    error
	)

	// 客户端配置
	config = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"}, // 集群列表
		DialTimeout: 5 * time.Second,
	}

	// 建立连接
	if client, err = clientv3.New(config); err != nil {
		fmt.Println(err)
		return
	}
    
    // 仅测试是否连接
	client = client

}

```

</details>


#### 2. Put 写入 KV

<details>
<summary>查看源代码</summary>

```golang
package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main() {
	var (
		config  clientv3.Config
		client  *clientv3.Client
		err     error
		kv      clientv3.KV
		putResp *clientv3.PutResponse
	)

	// 客户端配置
	config = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"}, // 集群列表
		DialTimeout: 5 * time.Second,
	}

	// 建立一个客户端
	if client, err = clientv3.New(config); err != nil {
		fmt.Println(err)
		return
	}

	// 用于读写etcd的键值对
	kv = clientv3.NewKV(client)

	if putResp, err = kv.Put(context.TODO(), "/cron/jobs/job1", "bye", clientv3.WithPrevKV()); err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("Revision", putResp.Header.Revision)
		fmt.Println("RaftTerm", putResp.Header.RaftTerm)
		if &putResp.PrevKv != nil {
			fmt.Println("PrevValue", string(putResp.PrevKv.Value))
		}

	}

}

```

</details>


#### 3. Get 读取 KV

<details>
<summary>查看源代码</summary>

```golang

package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main() {
	var (
		config  clientv3.Config
		client  *clientv3.Client
		err     error
		kv      clientv3.KV
		getResp *clientv3.GetResponse
	)

	// 客户端配置
	config = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"}, // 集群列表
		DialTimeout: 5 * time.Second,
	}

	// 建立一个客户端
	if client, err = clientv3.New(config); err != nil {
		fmt.Println(err)
		return
	}

	// 用于读写etcd的键值对
	kv = clientv3.NewKV(client)

	if getResp, err = kv.Get(context.TODO(), "/cron/jobs/job1", /*clientv3.WithCountOnly()*/); err != nil {
		fmt.Println(err)
	} else {
		fmt.Println(getResp.Kvs, getResp.Count)
	}

}

```

</details>


#### 4. Get 读取目录下所有 KV

<details>
<summary>查看源代码</summary>

```golang

package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main() {
	var (
		config  clientv3.Config
		client  *clientv3.Client
		err     error
		kv      clientv3.KV
		getResp *clientv3.GetResponse
	)

	// 客户端配置
	config = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"}, // 集群列表
		DialTimeout: 5 * time.Second,
	}

	// 建立一个客户端
	if client, err = clientv3.New(config); err != nil {
		fmt.Println(err)
		return
	}

	// 用于读写etcd的键值对
	kv = clientv3.NewKV(client)

	// 写入另外一个Job
	kv.Put(context.TODO(), "/cron/jobs/job2", "{...}")

	// 读取 /corn/jobs/ 为前缀的所有key
	if getResp, err = kv.Get(context.TODO(), "/cron/jobs", clientv3.WithPrefix()); err != nil {
		fmt.Println(err)
	} else { // 获取成功，遍历所有kvs
		fmt.Println(getResp.Kvs)
	}

}

```

</details>


#### 5. Delete 删除 KV

<details>
<summary>查看源代码</summary>

```golang

package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"go.etcd.io/etcd/mvcc/mvccpb"
	"time"
)

func main() {
	var (
		config  clientv3.Config
		client  *clientv3.Client
		err     error
		kv      clientv3.KV
		delResp *clientv3.DeleteResponse
		kvpair  *mvccpb.KeyValue
	)

	config = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"}, // 集群列表
		DialTimeout: 5 * time.Second,
	}

	// 建立一个客户端
	if client, err = clientv3.New(config); err != nil {
		fmt.Println(err)
		return
	}

	// 用于读写etcd的键值对
	kv = clientv3.NewKV(client)

	// 删除KV
	if delResp, err = kv.Delete(context.TODO(), "/cron/jobs/job1", clientv3.WithFromKey(), clientv3.WithLimit(2)); err != nil {
		fmt.Println(err)
		return
	}

	// 被删除之前的value是什么
	if len(delResp.PrevKvs) != 0 {
		for _, kvpair = range delResp.PrevKvs {
			fmt.Println("删除了:", string(kvpair.Key), string(kvpair.Value))
		}
	}
}


```

</details>

### 相关博文

- [什么是 etcd?](/tech/distributed/etcd/etcd_study_1_what_is_etcd.md)

- [etcd 功能与原理](/tech/distributed/etcd/etcd_function_and_principle.md)

- [Golang 操作 etcd（下）](/tech/distributed/etcd/etcd_usage_golang_2.md)


#### 感谢

[etcd 官网](https://etcd.io/)

[Golang - 开发分布式任务调度](/design/golang_crontab/golang_crontab.md)

