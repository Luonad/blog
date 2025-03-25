# 20250324

> 背景是rocketmq灰度使用的代理模式流量劫持+影子topic/group

测试灰度节点消费消息是发现rocketmq客户端日志拉取消息时出现ERROR日志
`subscription group [%s] does not exist`。
查看相关源代码得出是消费组导致的，代码如下：
```java
        SubscriptionGroupConfig subscriptionGroupConfig =
            this.brokerController.getSubscriptionGroupManager().findSubscriptionGroupConfig(requestHeader.getConsumerGroup());
        if (null == subscriptionGroupConfig) {
            response.setCode(ResponseCode.SUBSCRIPTION_GROUP_NOT_EXIST);
            response.setRemark(String.format("subscription group [%s] does not exist, %s", requestHeader.getConsumerGroup(), FAQUrl.suggestTodo(FAQUrl.SUBSCRIPTION_GROUP_NOT_EXIST)));
            return response;
        }
```
一路追踪代码查看调用链路，发现是broker集群在配置文件关闭了自动创建消费组功能，只能通过管控端api创建消费组。生产环境为了管控才这么做的。
相关代码如下：
```java
    public SubscriptionGroupConfig findSubscriptionGroupConfig(final String group) {
        SubscriptionGroupConfig subscriptionGroupConfig = this.subscriptionGroupTable.get(group);
        if (null == subscriptionGroupConfig) {
            // 判断broker配置是否允许自动创建消费组或者系统消费组
            if (brokerController.getBrokerConfig().isAutoCreateSubscriptionGroup() || MixAll.isSysConsumerGroup(group)) {
                subscriptionGroupConfig = new SubscriptionGroupConfig();
                subscriptionGroupConfig.setGroupName(group);
                SubscriptionGroupConfig preConfig = this.subscriptionGroupTable.putIfAbsent(group, subscriptionGroupConfig);
                if (null == preConfig) {
                    log.info("auto create a subscription group, {}", subscriptionGroupConfig.toString());
                }
                this.dataVersion.nextVersion();
                this.persist();
            }
        }
        return subscriptionGroupConfig;
    }
```
# 解决办法
- 通过管控端API创建消费组即可。