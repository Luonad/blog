# 20240624

> 记一次RocketMQ生产环境broker异常问题

rocketmq集群配置如下，主从之间使用同步双写模式，rocketmq版本4.9：

- 10.107.36.42 主 10.107.36.43  从
- 10.107.36.44 主 10.107.36.45 从
- 10.107.36.46 主 10.107.36.47 从

异常信息：

> org.apache.rocketmq.client.exception.MQBrokerException:CODE: 2 DESC: [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 304ms, size of queue: 33 BROKER: 10.107.36.46:10911
> For more information, please visit the url, http://rocketmq.apache.org/docs/faq/

排查过程：

1. 查看broker状态，正常
2. 查看broker日志，正常
3. 查找rocketmq资料，找到相关线索：maxWaitTimeMillsInQueue
4. 检查broker.conf

```properties
brokerRole=ASYNC_MASTER
# 同步slave超时时间300ms
maxWaitTimeMillsInQueue=300
```

5. 修改maxWaitTimeMillsInQueue=400
6. 重启broker，异常消失

源码解读：

```java
void cleanExpiredRequestInQueue(final BlockingQueue<Runnable> blockingQueue, final long maxWaitTimeMillsInQueue) {
        while (true) {
            try {
                if (!blockingQueue.isEmpty()) {
                    final Runnable runnable = blockingQueue.peek();
                    if (null == runnable) {
                        break;
                    }
                    final RequestTask rt = castRunnable(runnable);
                    if (rt == null || rt.isStopRun()) {
                        break;
                    }

                    final long behind = System.currentTimeMillis() - rt.getCreateTimestamp();
                    // 同步写入SLAVE超时抛出异常
                    if (behind >= maxWaitTimeMillsInQueue) {
                        if (blockingQueue.remove(runnable)) {
                            rt.setStopRun(true);
                            rt.returnResponse(RemotingSysResponseCode.SYSTEM_BUSY, String.format("[TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: %sms, size of queue: %d", behind, blockingQueue.size()));
                        }
                    } else {
                        break;
                    }
                } else {
                    break;
                }
            } catch (Throwable ignored) {
            }
        }
    }
```

参考资料：

https://blog.csdn.net/guyue35/article/details/108101087

