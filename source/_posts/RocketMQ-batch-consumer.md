---
title: RocketMQ批量消费消息
date: 2022-06-17 17:12:51
tags:
---

RocketMQ批量消费消息

<!-- more -->

### 默认消费方式

​	RocketMQ Consumer默认消费方式一条接着一条消费，先看以一个例子

**Producer**

​	发送20条消息

```java
public class Producer {


    public static void main(String[] args) throws MQClientException, MQBrokerException, RemotingException, InterruptedException 	{

        DefaultMQProducer defaultMQProducer = new DefaultMQProducer("ProducerGroup");
        defaultMQProducer.setNamesrvAddr("192.192.192.62:9876");
        defaultMQProducer.start();
        for (int i = 0; i < 20; i++) {
            Message message = new Message("SingleQueue", ("Hello").getBytes());
            defaultMQProducer.send(message);
        }

    }
}
```

**Consumer**

​	默认消费方式。consumeMessage的msgs，只有一条数据

```java
public class Consumer {

    public static void main(String[] args) throws MQClientException {

        DefaultMQPushConsumer defaultMQPushConsumer = new DefaultMQPushConsumer("ProducerGroup");

        defaultMQPushConsumer.setNamesrvAddr("192.192.192.62:9876");
        defaultMQPushConsumer.subscribe("SingleQueue", "*");
        defaultMQPushConsumer.setMessageModel(MessageModel.CLUSTERING);
      
        defaultMQPushConsumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                System.out.printf("%s 获取的消息 %s \n", LocalDateTime.now(), msgs);
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        defaultMQPushConsumer.start();

    }
}
```

**流程**

​	先启动Producer，然后再启动Consumer，打个断点观看。

如下截图

![](P1.png)



### 多条消息消费



RocketMQ Consumer 多条消息消费配置很简单，调用DefaultMQPushConsumer的setConsumeMessageBatchMaxSize方法设置每次能消费的消息数，代码如下

**Consumer**

```java
public class Consumer {

    public static void main(String[] args) throws MQClientException {


        DefaultMQPushConsumer defaultMQPushConsumer = new DefaultMQPushConsumer("ProducerGroup");

        defaultMQPushConsumer.setNamesrvAddr("192.192.192.62:9876");
        defaultMQPushConsumer.subscribe("SingleQueue", "*");
        defaultMQPushConsumer.setMessageModel(MessageModel.CLUSTERING);
        //执行每次消费多少条数据
        defaultMQPushConsumer.setConsumeMessageBatchMaxSize(10);

        defaultMQPushConsumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                System.out.printf("%s 获取的消息 %s \n", LocalDateTime.now(), msgs);
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        defaultMQPushConsumer.start();

    }
}
```



**流程**

​	重新启动之前Producer，然后再启动修改后的Consumer，打个断点观看。



如下截图

![](P2.png)





#### DefaultMQPushConsumer



查看DefaultMQPushConsumer代码，发现有个属性consumeMessageBatchMaxSize，设置每次能消费的消息数，默认值为1，这样就是默认消费的

```
public class DefaultMQPushConsumer extends ClientConfig implements MQPushConsumer {

    /**
     * Batch consumption size
     */
    private int consumeMessageBatchMaxSize = 1;

}
```



#### 注意：

如果多个消息消费，应该要保证本次的消费的消息数应该是具有原子性，要么都成功，要么都失败。





